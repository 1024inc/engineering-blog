---
title: Tools to backfill large PostgreSQL tables (part 1)
author: Tamas Simak
date: 2022-06-01 08:00:00 +0800
tags: []
comments: true
---


At Beyond we are mainly using PostgreSQL databases for our services, consisting of over a hundred tables. With some of the largest ones we’ve already had a few close calls as they were reaching the maximum supported 32TB table size - but that’s a story for another day. Backfilling tables that large is not a trivial task, assuming we want to perform it safely, and within a reasonable timeframe. In this post, we are going to outline a few methods and tools similar to what we have successfully used to make this process scalable, and a little less painful.

## What’s backfilling and why can it be a problem?

Backfilling simply means modifying or adding new data to existing database records, i.e. running `UPDATE` statements, but with an emphasis on performing it on all or most of the records in a table.

### When would it be needed?

* We want to modify a column’s value to fix faulty data
* We have to add a new field, i.e. insert a new column to the table, with either a default value or with data computed by arbitrary business logic.

In any case, the main issue is that if we want to perform a backfill on a seriously large table, it just simply takes a very long time. And by that I mean if we just naively issue an `UPDATE table SET field=value WHERE id=n` for each and every row, it will finish in weeks, months, or even years, depending on the use case’s context. In a specific case, we had to update a 22TB table with hundreds of billions of rows where the estimated time to completion was over 2 years.

With tables of this size, just creating a copy of the whole table and doing the update on it to avoid certain problems is not always a viable option, so we’ll focus on the use case where we really want to update the live table.
The first challenge is to complete this process as quickly as possible, naturally. The second one is to accept that it won’t be quick enough, and come up with tools that make this task more controlled and manageable, while making sure our application can still operate on the table without major hiccups.

## What can we do?

Our fictional scenario will be performed on a table named users. We’ll act like we have billions of users in our system. We want to backfill this table, inserting values into our newly created `reversed_last_name` column where we want to store our users’ last name, but, wait for it: reversed. And we want to do this because of, well, reasons. We will also pretend this is something that needs some calculations beforehand in a script and we couldn’t possibly do the update with just a snappy SQL expression (you know, like `REVERSE()`).

### Let’s perform the update in batches

Updating rows one by one puts a massive overhead on the process, and is the primary culprit for slowness. The solution seems simple enough: use as few individual `UPDATE` statements as possible. To do this, we want to write our data migration script so that it will output SQL which uses PostgreSQL’s `CASE` expression:

```sql
UPDATE users 
SET reversed_last_name = (
  CASE
    WHEN id = 1 THEN 'htulB'
    WHEN id = 2 THEN 'nasemraP'
    WHEN id = 3 THEN 'walboL'
    [...]
    ELSE last_name
  END
)
WHERE id >= 1 AND id <= 500
```

As you can see from the `WHERE` clause, we want to build update statements only for a subset of data. We might be tempted to build one huge `UPDATE` on all the rows, but there are a couple of issues with that. You’ll find that PostgreSQL puts an entire table lock on this huge table until the statement execution is completed, which unfortunately still won’t happen fast enough. Besides our application being denied write access to any rows, it would very likely cause a myriad of other issues. What works best is finding an equilibrium, and executing the updates in batches, balancing the number of rows affected in one batch, being aware that they will be locked while being updated, but processed and released from the lock in a reasonable amount of time.

To help us compose these batch operations, we’ll create a simple iterator class that will receive a database connection, the name of the table we want to backfill, the desired batch size, and an optional `start_batch` indicator that will come in handy later, when we implement a pause/resume feature and parallel processing. The class retrieves the max id of the table, and will yield us `id` boundaries according to the batch size, with a `begin_id` and an `end_id` which will help us build our select queries and batched update statements.

*This is a simplified example and it assumes we have integer primary keys called id and we have a data set starting from id=1.*

```python
import math

class TableBatchIterator:
    def __init__(self, conn, table, batch_size, start_batch=1):
        self.batch_size = batch_size
        self.start_batch = start_batch
        self.max_id = self.table_max_id(conn, table)
        self.total_batches = math.ceil(self.max_id / batch_size)

    def __iter__(self):
        self.current_batch = self.start_batch
        return self

    def __next__(self):
        if self.current_batch > self.total_batches:
            raise StopIteration

        begin_id = (self.current_batch - 1) * self.batch_size + 1
        end_id = min(self.current_batch * self.batch_size, self.max_id)

        self.current_batch += 1
        return (begin_id, end_id)
		
    @staticmethod
    def table_max_id(conn, table):
        with self.conn.cursor() as cursor:
            cursor.execute(f'SELECT max(id) FROM {table};')
            row = cursor.fetchone()

        return row[0]
```

To build the update statements shown above, we’ll also need something that takes the begin and end id values, the field name, and the new value. We’ll use a simple to use class for this.

The `BatchUpdateBuilder` class exposes two public methods: `update()`, which will be responsible for accumulating all the new values we want to persist for a given id and column name, and `build_sql()`, which will return the corresponding `UPDATE` SQL statement string with the `CASE/WHEN` expressions built from the values stored by `update()`. It will also append a `WHERE` clause to the statement to narrow down the scope to rows concerned in the given batch.

```python
import json

class BatchUpdateBuilder:
    def __init__(self, table, begin_id, end_id):
        self.table = table
	  self.begin_id = begin_id
        self.end_id = end_id
        self.updates = {}

    def update(self, _id, field, new_value):
        field_updates = self.updates.setdefault(field, {})
        field_updates[_id] = new_value

    def build_sql(self, begin_id, end_id):
        if not self.updates:
            return None

        sql = (
            f'UPDATE {self.table}\\n'
            f'SET\\n{self._build_cases()}\\n'
            f'WHERE {self._build_where(begin_id, end_id)}'
        )

        return sql

    def _build_cases(self):
        cases = []
        for field, field_updates in self.updates.items():
            cases.append(self._build_field_case(field, field_updates))
        return ',\\n'.join(cases)

    def _build_field_case(self, field, field_updates):
        cases = []
        for _id, new_value in field_updates.items():
            cases.append(f'WHEN id = {_id} THEN {self._to_sql_value(new_value)}')

        cases = '\\n          '.join(cases)
        return (
            f'  {field} = (\\n'
            '       CASE\\n'
            f'          {cases}\\n'
            f'          ELSE {field}\\n'
            '       END\\n'
            '  )'
        )

    def _build_where(self):
        return f'id >= {self.begin_id} AND id <= {self.end_id}'

    @staticmethod
    def _to_sql_value(value):
        if type(value) in (dict, list):
            return f'\\'{json.dumps(value)}\\'::jsonb'
        elif type(value) is str:
            return f'\\'{value}\\''
        elif value is None:
            return 'null'
        else:
            return str(value)
```

With these in place, we can do something like this, demonstrating with a Django example:

```python
from django.db import connection
from . import BatchUpdateBuilder, TableBatchIterator

BATCH_SIZE = 500
TABLE_NAME = 'users'

table_batch_iterator = TableBatchIterator(
    conn=connection, table=TABLE_NAME, batch_size=BATCH_SIZE
)

cursor = connection.cursor()

for boundaries in table_batch_iterator:
    begin_id, end_id = boundaries
    batch_update_builder = BatchUpdateBuilder(TABLE_NAME, begin_id, end_id)
    users = User.objects.select_for_update().filter(id__gte=begin_id, id__lte=end_id)
    for user in users:
        reversed_last_name = user.last_name[::-1]
        batch_update_builder.update(user.id, 'reversed_last_name', reversed_last_name)

    sql = batch_update_builder.build_sql()

    cursor.execute(sql)

cursor.close()
```

Pretty straightforward. We create an instance of the `TableBatchIterator` instrumented to perform the updates in batches of 500 (at most, actual batch size may vary if rows with certain id values are missing) on our users table, then iterate through it to build and execute a bunch of `UPDATE` statements generated by the BatchUpdateBuilder.

## Moving forward

In the second part of this blog post we’ll be going further, adding a **progress calculator** to help us monitor the status of the process, and as a last step we will create a lightweight **queue system** that makes it possible to run the task in parallel processes while also allowing us to pause and resume the process on demand. 

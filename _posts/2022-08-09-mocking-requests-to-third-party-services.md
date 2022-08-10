---
title: Mocking requests to third party services
author: Yiorgos Krypotos
date: 2022-08-10 08:00:00 +0800
tags: []
comments: true
---


Third party services are frequently used to extend the functionality of our product/service, especially in cases where what is needed grows beyond the limits of our industry or purpose. Once an integration is set up and running, naturally this must be included in our automation tests, right? (right?) 

It is a widely accepted practice to always mock requests made to third party services while testing, integration tests included. The reasoning behind this lies in 2 main areas:
1. We don’t control the response we get. This may lead to flaky tests, our testing suite failing because of service unavailability etc. The negative impact is even greater when a CI/CD process is used to deploy to our production and staging environments. Not to mention hotfixes...
2. Tests may take a while, since a connection is in the middle of it. If our suite consists of hundreds or thousands of tests, this may add up to a horrible amount of time to complete the suite.

So what are the best ways to mock those requests in our automated tests?

## Sample case

Let’s say that in our product, we deal with international customers who pay in different currencies and we want to convert everything to let’s say USD. To do so we can use the OpenExchange API. Here is a sample code to implement this:

```python
import requests

class ExchangeRatesClient:
    def __init__(self):
        self.base_url = "https://open.er-api.com/v6/latest/"

    def get_exchange_rates(self, currency_code):
        url = self.base_url + currency_code
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
```

## Good ol’ monkeypatch

Monkeypatch is a fixture from [pytest](https://docs.pytest.org/) which allows us to easily patch attributes. In this case the `get()` function of `requests`.

Example:

```python
class MockResponse:
    def __init__(self, json_data, status_code):
        self.json_data = json_data
        self.status_code = status_code
    
    def json(self)
        return self.json
    
    def raise_for_status(self)
        return

class TestExchangeRatesClient:
    def test_get_exchange_rates(self, monkeypatch):
        #assign
        exchange_rates_client = ExchangeRatesClient()
        expected_value = {
            "result": "success",
            "base_code": "USD",
            "rates": {
                "USD": 1,
                "CAD": 1.26,
                "CHF": 0.941,
                "EUR": 0.923,
                "GBP": 0.765,
                "JPY": 125.9,
            },
        }
        mock_requests = Mock(
            return_value = MockResponse(json_data = expected_value, status_code = 200)
        )
        monkeypatch.setattr(requests, "get", mock_requests)
        #act
        result = exchange_rates_client.get_exchange_rates(currency_code="USD")
        #assert
        assert result == expected_value
```


## How it works:

When requesting data, we get a [Response](https://requests.readthedocs.io/en/latest/api/#requests.Response) object so we have to mock its behaviour. This is what the `class MockResponse` is doing. This class may change depending on how we use the response object. Eg. we may need to add a `text` attribute, or add logic to the `raise_for_error()` to mock its behaviour and raise an HTTPError in case of an invalid status_code (4xx-5xx)

PROS:
- Doesn’t depend on 3rd party libraries
- Flexibility on implementing exotic cases when mocking

CONS:
- In real life examples, the response object can be very complex (headers, huge reponses, cookies etc) which can make the MockResponse class a hairy mess
- Monkeypatching has a global effect. If there were more requests we had to use side_effect and things could break.
- If instead of requests.get we use requests.Session().get(), then monkeypatching goes bust.


## Requests-mock

From the solution above we see that there is a lot of logic inside the MockResponse which isn’t always pleasant and we may introduce more bugs. And it feels a little too custom of a solution for such a generic problem to solve. So yeah… you guessed it. There is a library for that. One of the mostly used libraries in the python world is [requests-mock](https://requests-mock.readthedocs.io/en/latest/) 

Example

```python
def test_get_exchange_rates_with_requests_mock(self):
    # assign
    exchange_rates_client = ExchangeRatesClient()
    expected_value = {
        "result": "success",
        "base_code": "USD",
        "rates": {
            "USD": 1,
            "CAD": 1.26,
            "CHF": 0.941,
            "EUR": 0.923,
            "GBP": 0.765,
            "JPY": 125.9,
        },
    }
    # act
    with requests_mock.Mocker() as m:
        m.get("https://open.er-api.com/v6/latest/USD", json=expected_value)
        result = exchange_rates_client.get_exchange_rates("USD")
    assert result == expected_value
```

## How it works

Within the context manager, every request is being mocked by declaring it using the method and url. Regex can be used for more dynamic matching. If a request is not matched, then requests_mock will throw an exception and the test will fail, thus preventing the call to an external endpoint. Here, there are arguments as well to specify status_code, json, text etc depending on the desired response. 

PROS
- Easy to use
- Flexibility
- Less code
- Mock response objects have the desired behaviour by default

CONS
- You need to explicitly take care of every request in terms of expected values
- Hand written expected values

## VCR.py

Storming from Ruby's VCR library, VCR.py ~~brings all the 80s nostalgia that we secretly crave for~~ is a neat and powerful library that takes away most of the pain. 
What it does is that it records every http request done inside a test and “writes” them in a (by default .yaml) file. This file is then reused every time a request needs to be mocked. If there are more than one requests, then it records them all and are exposed in the cassette.responses list.

Example

```python
def test_get_exchange_rates_with_vcr(self):
    # assign
    exchange_rates_client = ExchangeRatesClient()

    # act
    with vcr.use_cassette(
        "exchange_rates.yaml", serializer="yaml", decode_compressed_response=True
    ) as cassette:
        result = exchange_rates_client.get_exchange_rates("USD")
    # assert
    assert result == json.loads(cassette.responses[0]["body"]["string"])
```

## How it works

This library is very flexible and can be configured in many different ways. 

### Decorator vs context manager
First it can be used as a decorator to mock all requests within a test, or as a context manager to mock requests in a specific area inside the test, allowing to have different VCR objects per request with different configurations. The `vcr.use_cassette()` can be used to specify the file to write and/or the path of that file. Ex. `with vcr.use_cassette(“path/to/folder/exchange_rates.yaml”)`

### Record modes
once (default)
When a request is about to happen, it checks if the file exists. If not, it makes an actual request, and writes it into the file. IF the file exists already, it uses that file instead of the actual request. If the file exists but the request is different than the one that it is recorded in, then it raises an error. 
This is the most used record mode. Usually we make a real request (with api_keys etc) and then we remove the sensitive code (we can also filter headers, query_parameters as described here)
If something goes wrong, then we can simply delete the .yaml file and run the test again.

none
Never make a real http request. It is mostly used when we want to be absolutely sure that an http endpoint is never called because that endpoint is sensitive. 

all
Always make an http request. This mode is not for mocking but when we want to override an existing cassette without deleting the file or when we want to simply log requests (rare)

new_episodes
Like the `once` mode, but in case there is a file with a similar request, it overrides it instead of raising an exception

### Sample Configuration

A very common blocker when first using this library is to create a generic configuration that all subsequent tests can easily use. Here is a personal favourite: a double wrapped func. 
Config file:

```python
# here more params can be added
def vcr_cassette(cassette_path):
    def inner(func):
        @wraps(func)
        def conf_vcr(*args, **kwargs):
            # default config
            vcr = VCR(
                record_mode=record_mode.RecordMode.ONCE,
                decode_compressed_response=True,
                serializer='yaml',
                cassette_library_dir=os.path.join(
                    os.path.dirname(os.path.realpath(__file__)) + '/cassettes/'
                ),
            )
            with vcr.use_cassette(cassette_path):
                return func(*args, **kwargs)
        return conf_vcr
    return inner
```

Usage:

```python
@vcr_cassette("exchange_rates.yaml")
def test_get_exchange_rates_with_vcr(self):
    # assign
    exchange_rates_client = ExchangeRatesClient()

    # act
    result = exchange_rates_client.get_exchange_rates("USD")

    # assert
    assert result == json.loads(cassette.responses[0]["body"]["string"])
```

PROS
- Easy to use
- Flexibility
- Stores real responses and is easy to record the complex ones that would be a nightmare to mock them manually
- Ability to view those responses
- Ability to hide/not record sensitive information like tokens/api keys etc
- Easily record multiple calls within a test, which in normal cases would be hard to mock.

CONS
- Every time there is a change in a request, that means that a response needs to be generated. But what happens if that response was time sensitive and changes in time? (Trick is to - manually tweak the request).
- A live request is needed if the record mode is set to ONCE. Sometimes this is impossible. 
- Configuration can be daunting at first


## Conclusion
So there you have it. When dealing with third parties there is a range of strategies that can fit to a variety of cases. Here at Beyond we prefer the feature rich and production-ready solution of the VCR library, since we are doing a heavy use of external requests. The other solutions proposed can also be an ideal approach, especially for simpler cases.

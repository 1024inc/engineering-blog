# Beyond Blog
## Running Locally
`docker-compose up`
Every change will livereload the server. you can see the changes in the browser at http://localhost:4000.

*Note: changes to config.yml will not trigger a restart of the server, you will need to restart the server manually.

## Writing a Post
### Quick start
2. Copy the example post from the `_draft` folder to the `_posts` folder.
3. Add a date to the filename, e.g. `2022-01-01-my-first-post.md`.
4. Edit the post.
5. Publish the post by committing to master, it will automatically be published for you.

### Static files
You can add static files to your posts by adding a `POST_DATE` folder to `/assets/images/posts/`

### Engineering paths
If you are working on updating the engineering paths, you will need to work from [this speadsheet](https://docs.google.com/spreadsheets/d/1PR2SKx3wIFfOOjx8XbzAGxlG4BmA3YWlhNol5HvxuTU/edit#gid=0), clone `https://github.com/1024inc/eng-paths-generator` **at the same level as this repo**, and use `deploy.sh` to deploy your changes here, including the index found at `/eng-paths.md` in the `eng-paths-generator` repo.

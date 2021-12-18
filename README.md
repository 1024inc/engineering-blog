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

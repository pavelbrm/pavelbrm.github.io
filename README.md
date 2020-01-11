## Blog

This repo contains source files for the blog.


### Running Locally

Using Docker:

```bash
docker run --rm -it --volume="$PWD:/srv/jekyll" --volume="$PWD/vendor/bundle:/usr/local/bundle" --publish 4000:4000 jekyll/jekyll:3.8 jekyll serve --watch
```

Using `docker-compose`:

```bash
docker-compose up
```

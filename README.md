Blue Yonder Tech Blog
=====================


## Run jekyll build in docker

```bash
export JEKYLL_VERSION=3.8
docker run --rm --volume="$PWD:/srv/jekyll" --volume="$PWD/vendor/bundle:/usr/local/bundle" -it jekyll/jekyll:$JEKYLL_VERSION  jekyll build
```

 ## Serving an existing site

```bash
export JEKYLL_VERSION=3.8
docker run --name jda-tech-blog --volume="$PWD:/srv/jekyll" --volume="$PWD/vendor/bundle:/usr/local/bundle" -p 4000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch --drafts
```

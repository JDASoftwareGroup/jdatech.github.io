# Blue Yonder Tech Blog

## Usage

### Run Jekyll build in Docker

```bash
export JEKYLL_VERSION=3.8
docker run \
    --rm \
    --volume="$PWD:/srv/jekyll" \
    --volume="$PWD/vendor/bundle:/usr/local/bundle" \
    --interactive \
    --tty \
    "jekyll/jekyll:$JEKYLL_VERSION" \
    jekyll build
```

### Serving an existing site

```bash
export JEKYLL_VERSION=3.8
docker run \
    --rm \
    --name jda-tech-blog \
    --volume="$PWD:/srv/jekyll" \
    --volume="$PWD/vendor/bundle:/usr/local/bundle" \
    --publish 4000:4000 \
    --interactive \
    --tty \
    "jekyll/jekyll:$JEKYLL_VERSION" \
    jekyll serve --watch --drafts
```

## Contribution

### Pre-requisites

* Use your personal GitHub account instead of the manage BY enterprise user account 
  (s. [Managing multiple accounts](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-your-personal-account/managing-multiple-accounts)).
* Ask [Gabriel Kohen](https://github.com/gkohen) for write access to the repository.

### How To: Add a new post

Create a new markdown file `_posts/<YYYY>-<MM>-<DD>-<some-title>.markdown`:

```markdown
---
layout: single
title: "<Some Title>"
date: <YYYY>-<MM>-<DD> <HH>:00:00 +0100
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg  # can also be a different asset
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: <Author Name>  # as used in `authors.yml`
author_profile: true
---
# Some Title
Foo bar.

![some picture](/assets/images/<YYYY>-<MM>-<DD>-<some-title>/picture1.svg)

## Section 1
...
```

Please test your post locally before publishing it. You can use the `jekyll serve` command shown in the
"Serving an existing site" section to have the page hosted locally and automatically reload when you make changes.

#### Formatting
See [GitHub Flavored Markdown Spec](https://github.github.com/gfm/).

#### Time
Please make sure the post filename and the date in the [YAML](https://en.wikipedia.org/wiki/YAML) front matter are
in-sync. For may choose your local time zone and some appropriate time (like 10AM).

#### Authors
Every author must be registered, see "How To: Add a new author". If the post is written by multiple authors, a
"multi-author" must be registered there.

#### Assets
If you want to use pictures, add them to `assets/images/<YYYY>-<MM>-<DD>-<some-title>/` (or as
`assets/images/<YYYY>-<MM>-<DD>-<some-title>.png` if you only have a single file). They can also be used as a header
image.

In general, prefer vector images ([SVG](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics)) for rasterized images
and uncompressed data ([PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics)) for compressed images. There are
exceptions though (like photos).

#### Linking to other posts
Example:

Global URL: http://0.0.0.0:4000/introducing-kartothek/

local URL: introducing-kartothek

To link other posts (like above example) in your current post, please use the following local URL syntax instead of a global URL:

```markdown
[Another Post](../local URL/)
```


### How To: Add a new author

In `_data/authors.yml`, add a new author:

```yaml
Author Name:
  name: "Author Name"
  avatar: "assets/images/authors/<author_name>.jpeg"  # optional
  links:  # optional, you can also add a subset
    - label: "Twitter"
      icon: "fab fa-fw fa-twitter-square"
      url: "https://twitter.com/<twitter_handle>"
    - label: "Github"
      icon: "fab fa-fw fa-github-square"
      url: "https://github.com/<github_handle>"
    - label: "LinkedIn"
      icon: "fab fa-linkedin"
      url: "https://www.linkedin.com/in/<linkedin_id>"
```

The author avatar (if used) must be added as `assets/images/authors/<author_name>.jpeg`.

For "multi-authors" (e.g. if a blog post was written by multiple people), a special entry must be added:

```yaml
Author1 Name1 & Author2 Name2:
  name: "Author1 Name1 & Author2 Name2"
  # no avatar used, or you find a picture of all authors
  links:  # optional, you can also add a subset
    - label: "Author1s' Twitter"
      icon: "fab fa-fw fa-twitter-square"
      url: "https://twitter.com/<twitter_handle1>"
    - label: "Author2s' Twitter"
      icon: "fab fa-fw fa-twitter-square"
      url: "https://twitter.com/<twitter_handle2>"
    # ...
```

## Hints

### Escaping curly brackets

Curly brackets in documented code snippets are conflicting with Jekyll's Liquid templating engine.\
For instance, when you want to document GitHub Actions workflows.
```yaml
 e2e-test:
    needs: build-deploy
    uses: ./.github/workflows/e2e_test.execution.yml
    secrets: inherit
    with:
      app-url: ${{ needs.build-deploy.outputs.app-url }}
```
The line `app-url: ${{ needs.build-deploy.outputs.app-url }}` will cause a Liquid syntax error because the curly 
brackets are interpreted as Liquid syntax.

To escape them, treat the opening brackets as a Liquid variable that outputs them as a string: `{{ '{{' }}`.\
The closing brackets don't need to be escaped.
```yaml
 e2e-test:
    needs: build-deploy
    uses: ./.github/workflows/e2e_test.execution.yml
    secrets: inherit
    with:
      app-url: ${{ '{{' }} needs.build-deploy.outputs.app-url }}
```

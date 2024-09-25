---
author: thunderbong
aliases:
  - I Like Makefiles
tags:
  - readwise/articles
url: https://switowski.com/blog/i-like-makefiles/
date: 2024-09-25
---
# I Like Makefiles

![rw-book-cover](https://news.ycombinator.com/favicon.ico)

## Highlights


The reason I like makefiles is that they often follow an unwritten convention of implementing the same set of commands to get you up and running. When I find a project I know nothing about, and I see a `Makefile` file inside, chances are that I can run `make` or `make build` followed by `make install`, and I will get this project built and set up on my computer.
> [View Highlight](https://read.readwise.io/read/01j8ja6b0w864q90nebv6q5q50)



With makefiles, when I come back to a project I haven't touched for months (or years), I don't have to remember the command to start a dev server with, let's say, Jekyll. I just run `make dev`, and this, in turn, fires up the corresponding Bundler commands.
> [View Highlight](https://read.readwise.io/read/01j8ja7xhq9k69pjh36ytmf3ck)



My makefiles are simple. I don't use conditional statements, flags or any other fancy features. Most of the tasks (they are technically called *targets*, but I always call them *tasks* in my head) consist of one or more shell commands.
> [View Highlight](https://read.readwise.io/read/01j8ja8ejdrfdk8mj4gearsp9z)



Some common tasks that most of my personal projects[[1]](https://switowski.com/blog/i-like-makefiles/#fn1) contain include:
 • `dev` to start the development server
 • `build` to build the project (if a build step is necessary)
 • `deploy` to deploy/publish the project
> [View Highlight](https://read.readwise.io/read/01j8ja8p3fpcb8cpnakbkqbd9c)



And a more advanced project of mine uses the following makefile to run the dev server, watch for changes, build, encrypt and deploy the website:
 ```Makefile
# Run dev server
dev:
	bundle exec jekyll serve --unpublished -w --config _config.yml,_config-dev.yml --livereload

# Build assets
build:
	npm run gulp build

# Watch a specific folder and process assets
watch:
	npm run gulp watch -- --wip

# Build the website locally, encrypt and deploy to Netlify server
deploy:
	JEKYLL_ENV=production bundle exec jekyll build; \
	make encrypt; \
	netlify deploy --prod

# Encrypt the "_site" folder
encrypt:
	npx staticrypt _site/*.html -r -d _site
```
> [View Highlight](https://read.readwise.io/read/01j8ja9g667mn0qz98cgj1k5s9)


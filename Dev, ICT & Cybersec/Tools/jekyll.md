---
Description: Jekyll is a static site generator. It takes text written in your favorite markup language and uses layouts to create a static website. You can tweak the site’s look and feel, URLs, the data displayed on the page, and more.
URL: https://jekyllrb.com/
---

## Start with jekyll

Install the jekyll and bundler gems:

```bash
gem install jekyll bundler
```

Create a new Jekyll site at `./myblog`:

```bash
jekyll new myblog
```

Build the site and make it available on a local server:

```bash
bundle exec jekyll serve
bundle exec jekyll serve --livereload	# reload changes without refreshing the page
bundle exec jekyll serve --drafts		  # include drafts folder
```

Browse to [http://localhost:4000](http://localhost:4000/)

Build the site for production:

```bash
bundle exec jekyll build
```
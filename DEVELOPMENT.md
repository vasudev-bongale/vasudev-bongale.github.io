# Local Development

## Prerequisites

- Ruby (2.7+)
- Bundler (`gem install bundler`)

## Setup

```
bundle install
```

## Run locally

```
bundle exec jekyll serve
```

Site will be available at http://localhost:4000.

Use `--livereload` for auto-refresh on changes:

```
bundle exec jekyll serve --livereload
```

## Build without serving

```
bundle exec jekyll build
```

Output goes to `_site/`.

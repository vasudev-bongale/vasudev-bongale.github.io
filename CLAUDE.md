# CLAUDE.md

Personal website for Vasudev Bongale — vasudevb.com.

## Stack

- Jekyll 4.4 static site, hosted on GitHub Pages
- Custom theme (no gem theme) — all CSS is inline in `_includes/head.html`
- Fonts: Literata (serif), Tiro Devanagari (for Sanskrit text), SF Mono (monospace)
- Colors: white background (#ffffff), dark text (#2a2a2a), tan accent (#c4b08a)

## Structure

- `_layouts/` — `home.html` (landing page), `default.html` (wrapper), `page.html`, `post.html`
- `_includes/head.html` — all meta tags and CSS (single file, no external stylesheets)
- `_posts/` — blog posts in markdown
- `writing.html` — post listing page
- `assets/images/` — avatar, background pattern SVG

## Local dev

```
/opt/homebrew/opt/ruby/bin/bundle exec jekyll serve --livereload
```

System Ruby is too old; use Homebrew Ruby at `/opt/homebrew/opt/ruby/bin/`.

## Conventions

- Keep the site minimal — avoid adding dependencies, external CSS, or JavaScript
- All styling lives in `_includes/head.html` as inline `<style>`
- Posts use `layout: post`, pages use `layout: page`
- Exclude non-site files (like this one) in `_config.yml`

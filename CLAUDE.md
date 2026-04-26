# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Branch Layout

This repo follows the standard Hexo two-branch pattern:

- **`source` branch** — Hexo source files (Markdown posts, theme, `_config.yml`, `package.json`)
- **`master` branch** — generated HTML output, deployed via GitHub Pages to **intwoplacesatonce.com**

Content editing and development happen on `source`. The `master` branch is managed by Hexo's deploy step and should not be edited by hand.

## Common Commands (run on `source` branch)

```bash
npm install          # install dependencies
npx hexo server      # local dev server at http://localhost:4000
npx hexo generate    # build to ./public/
npx hexo deploy      # generate + push output to master branch
npx hexo new "Title" # scaffold a new post in source/_posts/
```

## Source Structure

- `source/_posts/` — blog posts as Markdown files
- `themes/next/` — NexT v7.8.0 theme (Mist scheme); theme config lives in `themes/next/_config.yml`
- `_config.yml` — site-level Hexo config (URL, permalink pattern `:year/:month/:title/`, deploy target)
- `scaffolds/` — post/page/draft templates

## Key Configuration

- Permalink format: `YYYY/MM/title/`
- Deploy target: `git@github.com:davcamer/davcamer.github.io.git`, branch `master`
- Theme: `next` (Mist scheme), comments via Disqus (`in-two-places-at-once`)
- The repo uses **Jujutsu (jj)** alongside Git (`.jj/` present on master); prefer `jj` commands if available

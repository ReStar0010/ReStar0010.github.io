# ReStar0010.github.io

Personal site of Ming-Lung (Stanley) Tsai — writing on systems, security, and
the structures of thought and society. Built with the
[Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) Jekyll theme.

**Live:** <https://restar0010.github.io>

## Writing a post

Add a Markdown file to `_posts/` named `YYYY-MM-DD-title.md` with front matter:

```yaml
---
title: Your title
date: 2026-07-11 10:00:00 +0800
categories: [Security]
tags: [kernel, isolation]
---
```

Push to `master`; GitHub Actions builds and deploys automatically.

## Local preview (optional)

Requires **Ruby 3.1+** (system Ruby on macOS is too old). With `rbenv`:

```bash
rbenv install 3.4.0 && rbenv local 3.4.0
bundle install
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000>. A `.devcontainer` is also included for a
zero-setup container environment.

## Structure

- `_posts/` — blog articles
- `_tabs/` — sidebar pages (Portfolio, About, Categories, Tags, Archives)
- `_config.yml` — site identity and settings
- `assets/img/` — avatar and images

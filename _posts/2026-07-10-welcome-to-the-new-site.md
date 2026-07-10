---
title: A new home for the writing
date: 2026-07-10 09:00:00 +0800
categories: [Meta]
tags: [site]
pin: true
---

The site moved to a new foundation. The plan is simple: **write regularly, and
let it accumulate.** Two threads, one place —

- **Systems & security** — low-level machinery, isolation and kernel security,
  reverse-engineering notes.
- **Thought & society** — continental philosophy, and the people a system's
  "edge cases" tend to overlook.

If either of those is your thing, subscribe via the RSS link in the sidebar.

> This is a placeholder post so the homepage isn't empty. Delete it once your
> first real piece is up.
{: .prompt-tip }

## How posts work here

Every post is a Markdown file in `_posts/` named `YYYY-MM-DD-title.md`, with a
small header:

```yaml
---
title: Your title
date: 2026-07-11 10:00:00 +0800
categories: [Security]
tags: [kernel, isolation]
---
```

Code blocks, callouts, and a table of contents all come for free:

```c
// syntax highlighting works out of the box
int main(void) { return 0; }
```

That's it. Front matter, prose, publish.

---
layout: post
title: "Bringing a Legacy WordPress Update Down to Three Minutes"
description: "In a small public-sector part-time role, I used AI to map REST, FTP, and browser operations, then turned one-off exploration into a repeatable, verifiable workflow."
date: 2026-07-24 01:06:00 +0800
permalink: /en/projects/legacy-wordpress-ai-operations/
categories: [Project]
tags: [Project, WordPress, Automation, AI Assisted]
lang: en
translation_key: legacy-wordpress-ai-operations
published: true
---

[繁體中文](/projects/legacy-wordpress-ai-operations/)

> I used AI to move quickly through this first batch of project articles. Each one was expanded from reports, code, or notes I already had, then checked against those sources before publishing.
>
> 這一批專案文章是我用 AI 快速補寫的。內容都從既有報告、程式碼或筆記展開，發布前再逐篇對回原始資料。

My task was ordinary: update a legacy WordPress site with material I received.

Once I started, it turned out to involve more than logging in, editing some
text, and clicking Update. Some content was readable through REST, but writes
still had to go through a browser. Some public files lived in a separate FTP
structure. The admin interface also mixed editors and plugins from different
eras. It looked like one website on the surface, but underneath it exposed
several different interfaces.

This was a small public-sector part-time job. I do not identify the unit,
workplace, actual content, filenames, internal URLs, or how requests reached me.
What I can publish is how I approached this kind of administrative work: using
AI to map the system, preserving a repeatable procedure, and checking that the
site had actually changed as intended.

## Letting AI map the operational boundaries

I did not begin with one script that tried to handle everything. First, I had
AI follow the real task and help investigate:

- what the public REST interface could read;
- which writes required an authenticated browser;
- which static files were available only through FTP;
- where to read the result back after an update.

Those boundaries determined how the tools divided the work. REST was useful for
reading public structure. The browser retained login state and handled admin
operations. FTP dealt only with the site's existing static files. AI helped me
read page structure, test selectors, compare requests, and record failed paths.
The useful result was an interface map that I could still understand the next
time.

## Turning one conversation into a repeatable procedure

The early work resembled a series of improvised exchanges: open a page, find a
field, try an update, discover that a plugin had not synchronized, and switch
paths. If the only record after solving the problem was a chat log, I would
still have to guess again next time.

I turned the working procedures into runbooks. Each answered a few practical
questions:

1. Should this task use REST, FTP, or the browser?
2. Which inputs need checking before a write?
3. Which steps can be rerun, and which failures require cleanup first?
4. After a successful write, where should the result be read back and compared?

I later moved repetitive work into adapters, staging payloads, and batch
scripts. AI no longer had to rediscover the entire site. It could fill in inputs
within known boundaries, execute fixed steps, and stop at a clear checkpoint
when something went wrong.

```text
receive material
  -> normalize it into fixed fields
  -> run preflight checks
  -> choose the REST / browser / FTP path
  -> write
  -> read the result back from the site
  -> stop on a mismatch instead of continuing
```

## A successful write does not finish the task

The most important rule in this workflow is `No write without roundtrip`.

An API success response, an admin notification, or a completed FTP upload proves
only that one layer accepted the operation. The public page may still contain
old content, a file may be cached, or a plugin may have written the field
somewhere else. After each write, the workflow reads from the place that will
actually serve the data and checks the title, content state, file hash, or
public page.

Test data also belongs in a recognizable sandbox and is removed afterward.
Login details stay in local configuration outside the repository. They do not
enter scripts, runbooks, or execution results.

## Where the three-minute figure came from

One runbook records the timing for this kind of task. The first exploration
took about 90 minutes. Once the path had stabilized, one item, including final
verification, took about three minutes. A later measurement recorded about 40
seconds for machine execution alone. Batch records also show that retries and
cleanup can take much longer than the successful execution itself.

These numbers describe runs of that particular workflow. There was no manual
baseline and no error-rate measurement, so they cannot support a claim about
hours saved or improved administrative efficiency. What the records do show is
that knowledge once scattered across browser actions, FTP paths, and temporary
judgments now exists in a form that can be inspected, rerun, and handed over.

## What still requires a person

The batch scripts do not decide whether source material is complete, how public
copy should be written, whether an image is suitable, or whether dates and
categories make sense. The workflow stops at those points and waits for a
person's confirmation.

I also did not send internal material directly to an external model. AI mainly
helped map interfaces, generate and correct operational scripts, organize
failure records, and turn working procedures into runbooks. Site credentials,
work content, and original submissions remained inside the existing work
environment.

This article stops at the engineering record. Faster execution raises a
different set of questions about working hours, compensation, and what the
organization knows. Those questions need different evidence and belong in a
separate article.

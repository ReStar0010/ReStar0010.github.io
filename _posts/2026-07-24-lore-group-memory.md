---
layout: post
title: "Lore: Building Memory and Follow-Through for LINE Groups"
description: "A LINE bot prototype that turns scattered chat into retrievable facts, digests, and reminders. The code works in replay tests; real-group use has not validated the idea."
date: 2026-07-24 01:03:00 +0800
permalink: /projects/lore-group-memory/
lang: en
translation_key: lore-group-memory
categories: [Project]
tags: [Project, Agents, LINE, AI Assisted]
published: true
---

[中文版](/zh/projects/lore-group-memory/)

> I used AI to move quickly through this first batch of project articles. Each one was expanded from reports, code, or notes I already had, then checked against those sources before publishing.
>
> 這一批專案文章是我用 AI 快速補寫的。內容都從既有報告、程式碼或筆記展開，發布前再逐篇對回原始資料。

Group chats are good at producing context and bad at preserving it. A decision,
event, or promise can be clear at 10 p.m. and nearly impossible to recover a
week later.

Lore is a private prototype that treats a LINE group as a stream of possible
facts and actions. It can ingest messages, answer questions from stored group
memory, produce a digest, and create reminders. There is no verified real-group
adoption yet; the current evidence is the implementation and its replay tests.

## From message history to small facts

The memory layer does not store every generated summary as permanent truth. It
extracts candidate facts, checks them for sensitive patterns, and lets newer
facts supersede older ones. Similar entries can be consolidated, and facts can
decay rather than remain equally prominent forever.

The retrieval path deliberately began with a simple bigram-based index. That
choice kept the first version inspectable: a failed answer could be traced to
stored facts and matching terms without debugging an embedding pipeline at the
same time.

The result is a compact loop:

```text
LINE webhook
-> persist the message
-> extract and filter candidate facts
-> consolidate group memory
-> retrieve facts for Q&A, digest, or reminders
```

## Tool use instead of one large prompt

Later commits rebuilt Lore as a tool-using agent. The model selects among
bounded operations such as answering a question, changing digest time, or
setting a reminder. A registry parses the available skills and renders their
results back into LINE.

The same codebase includes event extraction from images, LINE Flex cards, and
iCalendar export. Those paths turn an event poster into something a group
member can save instead of another image that disappears into chat history.

## Testing conversations as workflows

My implementation work followed a red/green loop. Replay scripts feed a known
conversation through the webhook and inspect the resulting facts or actions.
Phase gates keep a feature from being treated as complete until the replay
reaches the expected state.

This does not replace live-user evaluation. It does make failures repeatable,
which is necessary before asking a real group to trust reminders or recovered
context.

## Current boundary

My implementation work covered the LINE webhook, message store, fact lifecycle,
PII gate, retrieval, skill registry, digests, Q&A, reminders, and replay
tooling.

The repository remains private, and no private chat transcript is used on this
page. The next useful test is a real group using Lore long enough to reveal
which memories are useful, which are intrusive, and which reminders people
actually follow.

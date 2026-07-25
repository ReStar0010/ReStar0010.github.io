---
layout: post
title: "Adventure Cube: From a Live Story App to a Batch Story and TTS Pipeline"
description: "We used hands-on testing and feedback to revise a children's story prototype. The repository now centers on a batch story-and-narration pipeline."
date: 2026-07-24 01:04:00 +0800
permalink: /projects/adventure-cube/
lang: en
translation_key: adventure-cube
categories: [Project]
tags: [Project, Generative AI, TTS, AI Assisted]
published: true
---

[中文版](/zh/projects/adventure-cube/)

> I used AI to move quickly through this first batch of project articles. Each one was expanded from reports, code, or notes I already had, then checked against those sources before publishing.
>
> 這一批專案文章是我用 AI 快速補寫的。內容都從既有報告、程式碼或筆記展開，發布前再逐篇對回原始資料。

Adventure Cube began as an Expo and Django application that generated
children's stories on demand. The part that remains active is different: a
batch pipeline that generates a story corpus, cleans the text, normalizes
Taiwanese wording, and produces narration files.

That distinction is the most important status fact in the repository. The live
runtime exists, but it is dormant. The batch content pipeline is the working
surface.

## The original product path

The mobile application let a user choose story ingredients such as characters,
backgrounds, themes, and key items. A Django backend handled story records and
generation. The team later split generation into stages and experimented with
several speech providers.

As the project changed, the codebase accumulated two different systems:

- an Expo/Django runtime for interactive story generation;
- scripts for generating many stories and narration files ahead of time.

Current onboarding notes warn that older architecture documents still describe
the first system as active. They should be read as history, not current
deployment documentation.

## The batch workflow that remains

The active path combines prompt fragments for story logic, characters, and
themes. A batch script generates raw stories, filters setup markers and
formatting, then converts Mainland Chinese terms where needed for a Taiwanese
audience.

The TTS scripts read a directory of cleaned stories and render WAV files with
Gemini's preview speech models. Earlier Azure, Vertex AI, and other experiments
remain as comparison material, but they are not the canonical runtime.

```text
prompt fragments
-> batch story generation
-> filtering and zh-TW normalization
-> directory-based TTS generation
-> story and audio corpus
```

## My contribution boundary

I designed the overall system architecture and worked alongside teammates
during hands-on testing, using what we observed and the feedback we received
to revise the product. I also contributed:

- early story-tab implementation;
- context-engineering templates and original scripts;
- an audio playback UI change and a TTS-service conflict fix;
- TTS experiments and tuning scripts;
- generated-story uploads;
- the bilingual onboarding and handover document.

That testing informed the product direction, but it was not a public launch or
evidence of sustained external adoption.

I worked alongside teammates who implemented substantial parts of the Expo
interface, Django backend, authentication, generation pipeline, filtering, and
content.

## Status before any revival

There is no retained evidence that the full application shipped to users. The
on-demand runtime has known integration gaps, including unfinished
authentication use and a mismatch between the preferred batch TTS provider and
the providers wired into Django.

The private repository also requires credential rotation and history cleanup
before it should be shared or linked. For that reason, this page records the
product and my bounded contribution but does not point readers to the source.

A future version should choose one direction: keep the batch content system,
revive the live app, or use a smaller hybrid that serves pre-generated stories
without the dormant Django runtime.

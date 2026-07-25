---
layout: post
title: "FurnitureStyle: Turning an Image into a Searchable Furniture Description"
description: "Instead of asking one prompt to recognize furniture and rewrite its color at once, this prototype split the job into two inspectable steps before searching."
date: 2026-07-24 01:02:00 +0800
permalink: /projects/furniturestyle/
lang: en
translation_key: furniturestyle
categories: [Project]
tags: [Project, Web, AI Assisted]
published: true
---

[中文版](/zh/projects/furniturestyle/)

> I used AI to move quickly through this first batch of project articles. Each one was expanded from reports, code, or notes I already had, then checked against those sources before publishing.
>
> 這一批專案文章是我用 AI 快速補寫的。內容都從既有報告、程式碼或筆記展開，發布前再逐篇對回原始資料。

FurnitureStyle was a Django prototype for searching furniture from an
image or text description. Its central path converted an input into a small
structured description, then used that description to query shopping results.

The prototype is associated with an NTU Web Application Programming course
submission, although the current course artifact does not identify the project
by name or record a final evaluation. This page therefore describes the code
that exists, not a deployment or course outcome.

## Why the classifier used two calls

An earlier implementation asked one multimodal prompt to identify the furniture
and transform its color description at the same time. The revised path split
those jobs:

```text
image or text
-> identify furniture type, style, and original dominant color
-> if requested, transform only the color description
-> search shopping results with the structured type and style
```

The first model response had to be JSON with `type` and `style`. If the user
selected the original color mode, the application could use that result
directly. Complementary, monochrome, and analogous modes triggered a second
request that changed the color portion while keeping the furniture identity.

This made the intermediate interpretation visible to the application instead
of asking one prompt to perform recognition and transformation in one step.

## The application around the classifier

The repository contains Django models, views, serializers, templates, and a
shopping-search integration. User profiles stored room, color, and style
preferences. A favorites path let users save and remove items.

![FurnitureStyle 的初步系統架構與搜尋流程](/assets/img/furniturestyle/system-flow.webp)
_期末資料中的初步流程圖。前端原始碼可對上 Vue 登入、搜尋、結果、收藏與歷史紀錄；OpenAI、購物搜尋、MySQL 與部署關係仍需後端材料才能完整驗證。_

The supplied Vue 3 front end makes the user path concrete. A user could sign
in with a username or Google account, submit either text or an image, choose an
original, analogous, or complementary color mode, then move to a result page.
The result cards linked to products and supported favorites; the account view
also exposed search history.

My implementation work covered changes to the profile and furniture data
models, favorites and user-profile features, and the two-stage classifier with
its color-mode handling. I collaborated with the member responsible for the front-end flow,
but the author identity recorded in the supplied source is still unresolved,
so I do not list that front-end work as mine.

The repository is private, so this article does not link or reproduce its
source.

The retained final-project archive contains a report, slides, a demo, and the
system-flow diagram above. A separately supplied RAR contains the front-end
source and a built `dist` directory. Its companion backend ZIP is empty, so it
does not add evidence for the Django internals, deployment, or testing.

## What this record does not claim

There is no retained evaluation of classifier accuracy, recommendation
quality, user adoption, or production deployment. The course-to-repository
identity is also inferred from timing and context rather than stated on the
submission page.

The prototype is useful evidence of a working product path and a specific
prompt-architecture revision. It is not evidence that the search results were
good enough for real purchasing decisions.

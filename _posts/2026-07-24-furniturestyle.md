---
layout: post
title: "FurnitureStyle: Turning an Image into a Searchable Furniture Description"
date: 2026-07-24 01:02:00 +0800
permalink: /projects/furniturestyle/
categories: [專案紀錄]
tags: [Project, Web, AI Assisted]
published: true
---

FurnitureStyle was a Django group prototype for searching furniture from an
image or text description. Its central path converted an input into a small
structured description, then used that description to query shopping results.

The prototype is associated with an NTU Web Application Programming group
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

The commit history attributes four changes to me:

- the initial project import;
- profile and furniture-model changes;
- favorites and user-profile functionality;
- the two-stage classifier and color-mode handling.

A collaborator's commits cover authentication changes and favorite-item
removal. The repository is private, so this article does not link or reproduce
its source.

## What this record does not claim

There is no retained evaluation of classifier accuracy, recommendation
quality, user adoption, or production deployment. The course-to-repository
identity is also inferred from timing and context rather than stated on the
submission page.

The prototype is useful evidence of a working product path and a specific
prompt-architecture revision. It is not evidence that the search results were
good enough for real purchasing decisions.

---

**Context:** Group web-application prototype<br>
**My evidenced work:** profiles, favorites, data-model changes, and two-stage furniture classification<br>
**Status:** Prototype; evaluation and course identity still need confirmation

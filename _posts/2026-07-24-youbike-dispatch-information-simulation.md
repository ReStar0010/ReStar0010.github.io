---
layout: post
title: "What Is the Time Value of Showing Users Where YouBike Rebalancing Happens?"
description: "A 76-station simulation asks a narrow question: if bikes are already being rebalanced, how much time can users save simply by seeing where?"
date: 2026-07-24 01:01:00 +0800
permalink: /projects/youbike-dispatch-information-simulation/
lang: en
translation_key: youbike-dispatch-information-simulation
categories: [Research]
tags: [Project, Simulation, YouBike, AI Assisted]
published: true
---

[閱讀中文版](/zh/projects/youbike-dispatch-information-simulation/)

The simulation held the number and location of rebalanced bikes fixed. The only
thing that changed was whether users knew where those bikes were.

Under the model's measured NTU shortage level of about 32%, showing that
information reduced mean arrival time by 1.8 to 2.4 minutes across the report's
physically plausible dispatch range. That is a simulated 15% to 22% reduction,
not a field measurement.

We developed the study in NTU's Computer Networks Laboratory. The repository
contains the processed data, simulation, maps, figures, and report, but it is
not publicly accessible, so this page does not link it.

## Isolating the value of information

Most bike-sharing work asks how the operator should predict shortages or move
bikes. This project asked a different question: if the operator is already
rebalancing the system, does exposing the current dispatch locations help a
user who has arrived at an empty station?

The comparison used two scenarios:

```text
without information:
  try the nearest station, then walk if no bike is available

with information:
  walk to a known replenished station within the detour tolerance,
  then ride to the destination
```

Both scenarios received the same dispatch footprint and capacity. This kept
the result focused on information disclosure rather than additional supply.

## Turning public data into a campus model

The model covered 76 stations around National Taiwan University. It combined
monthly origin-destination statistics with 43.9 GB of weekday station-status
data from September 2025.

The dynamic data was used to estimate departure timing, shortage rates,
dock-fullness, and the likely dispatch footprint. A jump of at least ten
available bikes between observations was treated as a replenishment event.
Because that proxy can confuse a burst of normal returns with a truck delivery,
the team did not treat the detected total as ground truth. It swept dispatch
capacity across a range instead.

The simulation compared the same agents under the same clock and supply, ran
30 Monte Carlo trials per parameter setting, and reported confidence
intervals. It also varied the detour-tolerance parameter rather than choosing
one value and hiding its influence.

## What the model found

At an empirically estimated shortage level near 32%, revealing dispatch
locations saved about 1.8 to 2.4 minutes within the report's plausible range of
50 to 200 replenished bikes per peak window. At 10% to 20% shortage, the
estimated reduction was 2.4 to 3.2 minutes.

The spatial analysis also placed residual unmet demand inside the eastern
campus area, around the men's first dormitory and Ming-Da Hall, while much of
the observed replenishment footprint was concentrated near Gongguan at the
campus edge.

These figures are outputs of the model. They do not show that a deployed app
would produce the same behavior.

## The assumptions that still matter

The dispatch quantity was the largest uncertainty. There was no official
dispatch log against which to validate the inferred replenishment events. The
origin-destination data and station dynamics also came from different months,
and the user model did not include waiting, repeated station searches,
ride-sharing, or other transport choices.

The study's useful contribution is therefore conditional: if dispatch supply
exists where the dynamic data suggests, and if users respond according to the
modeled detour rule, location information has measurable value without adding
bikes.

I proposed the topic and then worked in a PM-like role as the team shaped it
into a tractable comparison. I helped formalize the research question and the
two simulation conditions, and collaborated with teammates around that
framing while the implementation, figures, report, and reported numbers came
together through our shared work.

---

**Context:** Computer Networks Laboratory, National Taiwan University<br>
**My contribution:** Topic proposal, PM-like formalization, and team collaboration<br>
**Status:** Completed simulation study

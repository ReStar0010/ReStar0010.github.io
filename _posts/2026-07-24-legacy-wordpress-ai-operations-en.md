---
layout: post
title: "From WordPress to Agentic AI"
description: "How I handed a tedious public-sector WordPress update job to an AI agent, then turned each runbook into context for a reusable skill."
date: 2026-07-24 01:06:00 +0800
permalink: /en/projects/legacy-wordpress-ai-operations/
categories: [Project]
tags: [Project, WordPress, Automation, AI Assisted]
lang: en
translation_key: legacy-wordpress-ai-operations
published: true
---

[繁體中文](/projects/legacy-wordpress-ai-operations/)

## A simple but difficult task

This happened during a part-time job at a public-sector agency. My job was simple: take the material I received and update a WordPress site with it. I will leave out the agency, the actual content, and the internal URLs.

Honestly, this kind of work is not difficult. It is more like a "simple but difficult task." The goal is simple. The material is simple. I could copy the template from previous years. But putting everything onto the site was a pain. It took a lot of patience and energy to figure out a horrible system, find the right pages and formats, and place the content correctly. The system often froze. Every new component or editing screen took forever to load, and one mistake meant spending even more time undoing it.

So when Agentic AI took off in 2026, I started wondering: could I hand this whole thing to an AI Agent? It was not an impressive technical problem. It was just the most annoying, time-consuming part of my job, and the part I least wanted to keep repeating.

## How I judge whether Agentic AI can enter a workflow

Whenever I think about bringing Agentic AI into an existing workflow, I focus on two things: its level of symbolization and its level of interface access. This is how I currently judge how difficult the integration will be. I may write out the full definitions and argument another time.

First, symbolization. Put simply: how describable is the task? Did the people before me leave any specs, docs, examples, or historical records? Can the judgment that used to exist only in someone's head be turned into text, rules, or conditions that can be checked?

The quality of that description depends on two things. One is structure: how the content is organized and how different pieces connect. The other is precision: how closely the wording matches the context of the actual work. If the workflow is sufficiently symbolized and the quality is stable, an LLM finally has enough context to understand what it is dealing with instead of guessing from a vague instruction.

But symbolization only solves half the problem. It replaces the part where a person reads and understands the material. The other half is the Agentic part of Agentic AI: interface access.

Then comes a very practical question: once it understands the task, can it actually do anything? Is the work carried out entirely on paper, through a website, in some custom App, or through a simple CLI? Is there an API, MCP, or another interface an AI can use directly? This determines whether an LLM can enter the workflow and operate in place of a person.

Put the two together and a closed loop becomes possible:

```text
understand + execute -> result -> (next round) understand + execute -> result
```

If the AI can understand the material but a person still has to carry out every operation, it is closer to an assistant. If it keeps clicking through steps but never reads the result back, it is just automation. Agentic AI becomes useful when it can look at the result of an operation and decide what to do next.

## Back to this WordPress job

How symbolized was this workflow? Honestly, not very. The people before me left the basic materials and a simple operating spec. I had to figure out the rest myself. There was no complete document explaining where to find each type of page, how the components differed, or how to recover from a mistake.

The interface access was not much better. This old WordPress site was probably not going to magically grow an MCP server, and its rendering logic was complicated. Many buttons and components appeared only after a long wait. In this situation, using a dynamic browser tool like Playwright made more sense than hoping for a clean, usable API.

I ended up using `playwright-cli`, with a skill that constrained and explained how it should operate. About half a year earlier, I had started noticing this direction in the community: compared with using Playwright MCP directly, skill + CLI could return only the information needed at that moment and take up less of the context window. It also made it easier for me to see how far the Agent had gotten, what information it had, and what it planned to do next.

The harder problem was not the tool. At the beginning, we simply did not have enough context. When I first set the goal and problem boundary, the Agentic AI often misunderstood the work context and therefore misunderstood my intent. It might find a page that looked right but should not be edited, or copy the wrong historical format.

That meant a person still had to monitor and assist it at first. I split the task into checkpoints, each one a smaller goal. If I had ten pages to update, I would not tell it to run all ten on its first attempt. I would check the first page, and maybe the first three if needed. Only after the page, formatting, write location, and final result all looked right would I let it work through the remaining pages in a batch.

After every write, it also had to return to the actual page and read the result back. A success message in the admin panel only meant the system had received the operation. It did not mean the web page was correct. I kept that check at every checkpoint.

Once the task was complete, I asked the AI to turn the experience into a runbook: the mistakes we ran into, the tools that worked, the correct path, and where to check the final result. The next time a similar task appeared, it did not have to guess all over again.

My rough number is three to five runbooks. By then, I can start turning the repeated parts into a more abstract skill. This is how I fill in the symbolization that was missing at the beginning. Every completed task leaves behind a little more context, so the next one can be done more precisely.

## From more than half an hour to three minutes

Updating these two WordPress pages by hand could easily take me more than half an hour. I had to locate the right elements, wait for this awful old system to render every time I entered a component or started editing, and spend even more time fixing anything I got wrong. Just thinking about it was irritating.

With the Loop above, I now copy the request to the Agent and give it the screenshots it needs. It can finish both page updates in three minutes. Without network latency and the old system's loading time, it could be even faster. More importantly, I do not have to sit there operating the site for those three minutes. I can do something else and come back when it finishes or reaches a checkpoint.

This made the part-time job much easier. I still have to set the goal, provide context, and verify the final result, but I no longer have to perform all the most annoying and repetitive operations myself.

## The first time Agentic AI directly made me money

I am a heavy AI user, and this was not the first time I had used Agentic AI to solve a problem. But it was the first time I clearly felt that I had directly made money through Agentic AI. My pay was fixed. When the time needed to complete the same work fell from more than half an hour to three minutes, I spent less time while receiving the same pay.

There was no fancy or sexy Idea behind it, and no new Tool that I absolutely had to use. I only needed to understand the tedious workflow and know where Agentic AI could fit into it. That was enough to produce a huge change in productivity. I was not merely doing the same work faster. I could step out of most of the repeated operations completely.

This was only a tiny test inside a much larger system. Still, it was when I finally felt that the challenges facing white-collar, knowledge, and clerical work under capitalism were not coming out of nowhere. New production relations may turn the current system upside down. But the change may not arrive in some dramatic form. It may begin with someone realizing that a task that used to take half an hour now takes three minutes.

If the execution time for many tasks inside an office can be cut by 90%, will we reduce only the hours, or the people too? Is this what is already happening in Silicon Valley?

I want to return to this from an economic or sociological angle. When a job still pays a fixed wage for fixed hours, but the actual time required to finish the work suddenly falls, what choices will individuals and organizations make? And how could those rational choices eventually produce a result that is completely reasonable and completely absurd at the same time?

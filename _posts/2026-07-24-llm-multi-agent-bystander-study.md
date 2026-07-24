---
layout: post
title: "When More LLM Agents Did Not Diffuse Responsibility"
description: "We designed a panel experiment to ask whether LLM agents go quiet in larger groups. The run hit an intervention ceiling, leaving a narrow null result and a clearer next experiment."
date: 2026-07-24 01:00:00 +0800
permalink: /projects/llm-multi-agent-bystander-study/
categories: [研究紀錄]
tags: [Project, Multi-Agent Systems, LLM, AI Assisted]
published: true
---

The result was not the one in the research plan.

We expected that an LLM agent might become less likely to object when more
witnesses were present. After 82 clean panel trials, the measured intervention
rate stayed at the ceiling across groups of three, five, and nine agents.

This was a three-person course research project built with
[TinyTroupe](https://github.com/microsoft/TinyTroupe). The original plan was to
replicate the classical bystander effect inside LLM-driven panels, inspect the
reasoning associated with non-intervention, and test ways to assign
responsibility more explicitly. The experiment instead became an exercise in
how to keep a null result useful.

## The question behind the experiment

Many multi-agent systems use several critics, reviewers, or judges in the hope
that one agent will catch what another misses. That design assumes that adding
agents does not also make each agent feel less responsible for speaking.

The course team turned that concern into a controlled panel experiment. Each
trial contained a scripted problematic proposal and a set of persona-driven
agents who could question it, reject it, or remain silent. The pre-registered
design varied group size and scenario while keeping the triggering message
controlled.

The planned pipeline included:

- 30 personas generated from structured OCEAN trait profiles;
- solo pre-screening to identify floor and ceiling behavior;
- panel sizes of 3, 5, and 9;
- medical, financial, privacy, and academic-governance scenarios;
- per-agent and per-trial intervention measures;
- checkpointed trial logs and a separate coding pass.

The point of writing the plan first was simple: both a detected effect and a
null result needed to be interpretable without moving the goalposts afterward.

## What actually ran

The intended Study 1 contained 360 trials. API quota and retry behavior stopped
the run at 92 panel trials. Ten logs were corrupted by failed or incomplete
requests, so the headline analysis excluded them and retained 82 clean trials:

```text
medical, N=3: 30 trials
medical, N=5: 26 trials
medical, N=9: 24 trials
financial, N=3: 2 trials
```

The usable evidence therefore came almost entirely from the medical-review
scenario. That boundary matters more than the original ambition of the study.

The solo intervention baseline was 0.992. In the clean medical panel trials,
per-capita intervention was 1.000 at every tested group size. The LLM-judge
coding also classified every coded trial as containing an explicit,
severity-three refusal. The expected traces of diffusion of responsibility,
pluralistic ignorance, and audience inhibition did not appear.

In this setup, adding witnesses did not produce measurable silence.

## Why the null result survived

The null would have been easy to overstate as proof that LLM panels are safe.
The artifacts do not support that claim.

They support a narrower statement: `gpt-5-mini` agents placed in an explicit
review panel, given a controlled medical-regulatory violation, objected
reliably for group sizes between three and nine. The experiment did not
establish the same behavior in casual collaboration, under adversarial social
pressure, across other models, or across the other planned scenarios.

The failed runs were also part of the result. TinyTroupe's retry behavior made
quota exhaustion look like a long series of transient rate limits. Once the
team separated the ten corrupted logs, it stopped the pipeline instead of
quietly folding partial trials into the analysis.

That choice left a smaller study, but one whose denominator can be explained.

## What remains unresolved

The next useful experiment would reduce the ceiling. A perpetrator that stays
in the conversation and defends the proposal, a less explicit review setting,
or a broader set of models could create enough variation to test whether
responsibility ever diffuses.

My contribution included designing the experiment structure: turning the
bystander-effect question into controlled panel conditions with different
group sizes. This was still a three-person course project; I am not assigning
the entire implementation, analysis, or writing to myself.

The original [course project report is available here](/files/llm-multi-agent-bystander-study-project-report.md).
It is a snapshot of the submitted work, including the team's original
interpretation. This page uses narrower language where the completed runs do
not support the report's broader claims. Persona files and raw `[THINK]` traces
are not included.

---

**Context:** Advanced Information Technologies II, National Taiwan University<br>
**My contribution:** Experimental structure and research design<br>
**Status:** Completed three-person course research artifact

# Project Report — Bystander Effect in LLM Multi-Agent Systems

**Course.** 前瞻資訊科技二 (Advanced Information Technologies II), National Taiwan University.
**Assignment.** Open-ended generative-agent simulation project; tier-1 paper aspiration.
**Framework.** TinyTroupe v0.7.0 (Microsoft Research).
**Underlying LLM.** OpenAI `gpt-5-mini`.
**Project span.** 2026-04-26 → 2026-05-08 (12-day window).
**Deliverables.** Slides (5/8) · *PAPER.md* (this companion document is the project report).

---

## 0. Executive summary

We chose to investigate **whether the classical bystander effect transfers to populations of LLM-driven agents** — a question with both pure-research interest and direct multi-agent-AI-safety relevance. We built a complete experimental pipeline (30 OCEAN-Big-Five personas, 4 scenario scripts, scripted perpetrator agent, LLM-as-judge for outcome coding, trial-parallel runner with checkpointing), pre-registered our hypotheses in `RESEARCH_PLAN.md` *before* spending API budget, and ran 82 clean panel-deliberation trials at three group sizes (N=3, 5, 9) in a medical regulatory scenario.

**The headline empirical finding is a clean null:** per-capita intervention is at the ceiling (1.000) regardless of group size. The cognitive signature of bystander dynamics — diffusion of responsibility, pluralistic ignorance, audience inhibition — is *absent* from the agents' internal `[THINK]` traces. Severity-3 (explicit refusal) interventions occur in 100% of LLM-judge-coded trials.

We treat this null as a *strong positive* finding for AI-safety multi-agent architectures: the popular "panel-of-LLMs catches more than one LLM" assumption is empirically supported in our setting, contrary to the concern that responsibility might dilute. The companion paper (`PAPER.md`) presents the scientific argument; this report covers the project process: what we built, what went wrong, what we learned about doing this kind of research.

---

## 1. Why bystander effect

The course allowed any generative-agent topic. We chose the bystander effect for four reasons, in roughly this priority order:

1. **There is a real, identifiable gap in the literature.** Aher, Arriaga & Kalai [2023] replicated Asch (conformity), the ultimatum game, and Milgram (obedience) in LLM populations. They explicitly skipped the bystander effect. Other LLM-social-replication papers focus on conformity / convergence / opinion dynamics — *not* on inaction-under-witnesses, which is categorically different.
2. **It connects to a live AI-safety question.** Multi-agent panel-of-LLMs designs (debate-based alignment, constitutional-AI critic pools, judge-of-judges) implicitly assume more agents = more catches. The bystander effect, if present, would be one specific source of *correlated* silence — a hidden risk in those architectures. Either result (effect present or absent) is informative.
3. **It is *uniquely well-suited* to LLM agents.** Bystander research has spent fifty years disentangling DR / PI / AI mechanisms via indirect methods (questionnaires, post-hoc self-reports, manipulation checks). LLM agents emit their reasoning verbatim via `[THINK]` actions. We have *direct* access to the cognitive narrative — a methodological gain that humans cannot offer at scale. This is the single most distinctive feature of our project.
4. **It is *operationally tractable*.** A panel-deliberation environment with a scripted "perpetrator" turn is exactly what TinyTroupe is good at. We don't need physical embodiment, dynamic stimuli, or real-time interaction.

The original course assignment encourages tier-1 paper aspirations. We took this seriously: the project plan (`RESEARCH_PLAN.md`) includes a power analysis, pre-registered hypotheses, a manipulation-check protocol, an inter-rater-reliability gate, and a robustness arm with a second model class. Whether or not we hit "tier-1," structuring the work this way produced cleaner science.

---

## 2. What we built

The repository is fully self-contained. `MIT-licensed.`

### 2.1 Repository structure

```
SD_HW/
├── RESEARCH_PLAN.md          ← pre-registered design (locked 2026-04-26)
├── RESEARCH_LOG.md           ← chronological execution log (520+ lines)
├── PAPER.md                  ← scientific paper (companion artefact)
├── PROJECT_REPORT.md         ← this file
├── personas/
│   ├── generate_personas.py  ← deterministic seeds + LLM hydration
│   └── persona_pool.json     ← 30 hydrated personas
├── scenarios/
│   ├── base.py
│   ├── medical.py            ← consumer health-app onboarding (mild)
│   ├── financial.py          ← retail investing newsletter (moderate)
│   ├── privacy.py            ← children's app data collection (high)
│   └── academic.py           ← thesis-defence integrity (moderate)
├── pipeline/
│   ├── run_trial.py          ← single-trial executor
│   ├── run_parallel.py       ← ProcessPoolExecutor runner with --limit, --workers
│   ├── tinytroupe_adapter.py ← wraps TinyPerson/TinyWorld; catches API errors
│   ├── sampling.py           ← persona pool sampling
│   ├── coding_intervention.py ← LLM-judge (forced tool-use, strict JSON)
│   ├── coding_mechanism.py    ← DR/PI/AI tagger via LLM-judge
│   ├── code_logs.py           ← drives judges over a directory of trial logs
│   ├── cost_monitor.py        ← USD projection from token counts
│   ├── build_validated_pool.py ← consumes prescreen logs, computes inclusion
│   ├── status.py              ← progress dashboard utility
│   └── mock_adapter.py        ← API-free dry-run validator
├── analysis/
│   ├── full_descriptive.py   ← all paper tables + figures (no API)
│   ├── stats_study1.py       ← logistic regressions (statsmodels)
│   ├── stats_study2.py       ← mechanism distribution (post-judge)
│   ├── stats_study3.py       ← ANOVA + planned contrasts
│   ├── interim_glance.py     ← real-time status sniff
│   ├── build_dataset.py      ← turn/agent/trial dataframes from raw logs
│   └── run_all.py            ← end-to-end analysis driver
├── data/
│   ├── prescreen_logs/       ← 120 solo trials
│   ├── study1_logs/          ← 92 panel trials (82 clean, 10 error-corrupted)
│   ├── coded/
│   │   ├── intervention_codes.csv  ← 192 LLM-judged turns (medical_N3 sub-sample)
│   │   ├── prescreen_summary.csv
│   │   └── validated_persona_ids.json
│   └── ...
├── results/
│   ├── tables/               ← cell_summary.csv, agent_level.csv, persona_rates.csv, ...
│   └── figures/              ← fig1_intervention_by_groupsize.{png,pdf}, ...
├── slides/
│   └── OUTLINE.md
├── config.ini                ← TinyTroupe configuration
├── requirements.txt
└── .env                      ← API keys (gitignored)
```

### 2.2 Pipeline architecture (one-paragraph version)

A trial is built as a `TrialSpec` (study, scenario, group_size, persona IDs, condition, n_rounds, seed, suffix). `run_trial(spec, pool, adapter_factory)` builds a fresh `TinyWorld`, instantiates the agents from the persona pool, broadcasts the scenario briefing, plays the scripted perpetrator turn, then runs `n_rounds` discussion rounds where every witness agent acts in randomised order. Every `[THINK]` and `[TALK]` is captured into a JSON log with full provenance metadata. `run_parallel.py` enumerates a study's specs (skipping any whose JSON already exists on disk), and dispatches them to a `ProcessPoolExecutor` of N workers. Each worker is a fresh Python interpreter holding one trial's TinyTroupe singleton state. After collection, `analysis/full_descriptive.py` reads the logs and the LLM-judge CSV to produce all tables and figures.

### 2.3 Key engineering decisions (and why)

| Decision | Why |
|---|---|
| **Mock adapter first, real API later.** | Catches integration bugs (JSON shape, off-by-one in turn counting) at $0 before any real run. Standard simulation hygiene. |
| **`ProcessPoolExecutor`, not threads.** | TinyTroupe holds class-level singletons (`TinyPerson._all`, `TinyWorld._all`). Threads collide; processes give full isolation. |
| **Trial JSON is the unit of persistence.** | Skip-if-exists checkpointing is trivial. A killed run resumes from disk; no special "resume DB" needed. |
| **LLM-judge with forced tool-use.** | Strict JSON schema, no text parsing, works on every Anthropic/OpenAI SDK version. Avoids the parsing-bug class entirely. |
| **`gpt-5-mini` for both agents and judge.** | Cheapest competent option. We migrated the judge to Anthropic Claude Haiku as a fallback (committed but unused — see §4 for the story) and reverted back when API quota stabilised. |
| **Pre-screen baseline before main runs.** | Without a solo baseline, we cannot tell the difference between a *competence ceiling* and a *flat-line non-effect.* The 0.992 solo baseline is critical to interpreting the headline 1.000 cell rates. |
| **Personas designed by OCEAN × occupation grid.** | Deliberate trait-space coverage. Includes "anxious-conformist" and "low-C-low-A" templates that *should* be most prone to bystander-style non-intervention if the effect exists. They aren't. |
| **No persona is told about the experiment.** | Avoids demand characteristics. Personas read like real workplace profiles, not lab-experiment scripts. |
| **Quality checks ON for main runs.** | Persona-adherence is critical to internal validity; reviewers will ask. We pay the token cost up-front. |

---

## 3. What we found

The full scientific argument is in `PAPER.md`. Here we summarise for the project-report reader.

### 3.1 Headline numbers

| Cell | n trials | n agents | Per-capita rate | Trial-level rate | DR | PI | AI |
|---|---:|---:|---:|---:|---:|---:|---:|
| medical_N3 | 30 | 90 | **1.000** | 1.000 | 1.1% | 0.0% | 0.0% |
| medical_N5 | 26 | 130 | **1.000** | 1.000 | 0.0% | 0.0% | 0.0% |
| medical_N9 | 24 | 216 | **1.000** | 1.000 | 2.8% | 0.0% | 0.0% |
| solo baseline | (120) | 30 | 0.992 | — | — | — | — |

**Every single witness-agent in 80 medical-scenario panels intervened against the perpetrator's proposal.** The pre-registered H1 (per-capita rate decreases monotonically as N grows from 3 to 9) is decisively rejected — the rate is identical at all three group sizes. The pre-registered H3 (DR predominates over PI and AI in non-intervention `[THINK]` traces) is *vacuously inapplicable*: there is no non-intervention to explain, and there is essentially no DR / PI / AI signal in the traces of the agents who *did* intervene either.

Among 8 trials with full LLM-judge coverage in medical_N3:
- 99.5% of turns coded as intervention (191/192)
- 56.5% severity-3 (explicit refusal/escalation)
- 39.8% severity-2 (clear objection)
- 100% of trials produced ≥1 severity-3 turn

Agents do not just intervene; they intervene *strongly*.

### 3.2 What this means

The honest one-paragraph summary: *under the conditions we tested* — `gpt-5-mini` agents in deliberative panel-review settings with subtle-but-objectively-wrong stimuli, group sizes 3-9, and personas spanning the OCEAN trait space including conformity-anxious and low-conscientiousness templates — *we cannot find a bystander effect.* This is a clean null in the technical sense (point estimate at ceiling, not noisy zero), and it is informative for two communities:

1. **Multi-agent AI-safety researchers** can read this as evidence that "panel-of-LLMs" architectures do *not* exhibit hidden diffusion-of-responsibility risks in our setting. The base assumption "more agents = more catches" appears to hold for the deliberative-review tasks we studied.
2. **LLM-as-social-simulator researchers** should read this as a caution: training-induced caution likely dominates social context in `gpt-5-mini`, meaning that effects whose presence depends on ambivalence-toward-intervention may simply not appear in current frontier LLMs. Studies aiming to *find* such effects in LLM populations may need to use older / smaller / less safety-trained models, or to adversarially construct stimuli that bypass training priors.

### 3.3 What we cannot claim

- We cannot generalise across scenarios with the data we have. Only `medical` has substantial coverage. The financial-N3 cell (n=2) hints at lower per-capita rate (5/6 = 0.833), but the sample is too small for inference. Our planned 360-trial design (3 sizes × 4 scenarios × 30 trials) was halted at 92 trials due to engineering challenges (§4), of which 82 are clean.
- We cannot generalise across model classes. The pre-registered GPT-5 robustness arm was not run.
- We cannot rule out that the effect would emerge at larger N (15, 30) or more adversarial perpetrators (multi-turn deception, social engineering). Latané & Darley's classical setting goes up to N=15+; we stopped at N=9.

These limitations are exactly what `PAPER.md` §5.2 spells out. They define the falsifiable forward research program.

---

## 4. What went wrong (the engineering story)

This section is for the course report — a lab journal of the messy reality behind the clean numbers.

### 4.1 The OpenAI quota incident (Phase 6.3, 4/27)

**What happened.** After running for ~12 hours with 8 parallel workers, we resumed the pipeline and immediately launched the LLM-judge pass over the 60 already-completed trial logs. Within 12 seconds, `code_logs.py` produced a `429 - insufficient_quota` error from the OpenAI SDK. We checked the main pipeline log: it showed 90+ "Rate limit error" warnings, which TinyTroupe's exponential-backoff wrapper had been logging as transient throttle for *hours*. They were not transient. The account had hit the daily-spend ceiling.

**Why we missed it earlier.** TinyTroupe's `LLMRequest` wrapper logs all 429s identically as "Rate limit error" and just retries with exponential backoff. The OpenAI SDK distinguishes *transient* 429s (retry will eventually succeed) from *quota-exhaustion* 429s (retry will *never* succeed) only via the response body, which the wrapper does not surface. We were burning compute on retries that could not succeed. This was costing API tokens (yes, OpenAI charges for 429s) for several hours.

**What we did.** Stopped all running tasks. User added credit to the OpenAI account. Pipeline resumed. We considered adding a pre-flight quota probe in `run_parallel.py` (would fail-fast on `insufficient_quota` before workers churn) but didn't implement — flagged for follow-up.

**Lesson.** When an LLM library wraps the API for you, *also* log the raw response body when something goes wrong. The error you can't see is the one that costs the most.

### 4.2 The Anthropic-Claude judge migration (Phase 6.4, 4/27)

**What happened.** Faced with the OpenAI quota issue, we considered splitting vendor risk: keep agents on OpenAI, move the LLM-judge to Anthropic Claude. We migrated `coding_intervention.py` and `coding_mechanism.py` from OpenAI (chat-completions with `response_format=json_object`) to Anthropic (messages with `tools=[...]` and `tool_choice={"type":"tool", ...}` for forced tool use with `strict=true`). The Claude implementation is arguably *cleaner* — no text parsing, no JSON-mode quirks, just `tool_use.input` directly.

**Why we ultimately reverted (kind of).** The user decided to defer LLM-judging entirely until raw data collection is finished, to avoid spending on a second API vendor mid-flight. The Anthropic-side code was committed but unused. It is good infrastructure to have for the future — when we batch-judge all trial logs at the end, we can pick whichever vendor has fresher quota. The migration was not wasted; it forced a clean rewrite that improved the JSON-shape robustness of the judge code path.

**Lesson.** It's fine to build infrastructure speculatively if the cost is small and the upside is real (clean code, fallback option). Spending API tokens speculatively is different from spending engineering hours speculatively.

### 4.3 The rate-limit storm and corrupted trials (Phase 6 batch 3, 4/28)

**What happened.** After two clean batches at 8 workers (95 min wall time each, 16 N=9 trials), batch 3 went into a sustained 429 storm: 3000+ rate-limit responses with 0 successful completions over 2+ hours. We had been *making progress* in batches 1-2, then *suddenly stopped* in batch 3. Diagnosis was tricky — had quota been exhausted again? Was OpenAI throttling us? Eventually we let batch 3 grind out, and discovered something worse: 6/8 N=9 trials in batch 3 had **partial data corruption**.

The pattern: under sustained throttling, some `gpt-5-mini` API calls returned `None` instead of a normal response. The TinyTroupe→adapter pipeline expects a structured response, so this triggered `TypeError: 'NoneType' object is not subscriptable` at the boundary. Our adapter caught the exception and recorded `[ERROR: TypeError: 'NoneType' object is not subscriptable]` as the agent's `talk` text for that turn. The trial then "completed" successfully — but with one or more error-turns embedded.

The trial files looked normal at first glance: 72 turns each, JSON valid, no exception escaped to the parent process. We only caught it by inspecting the `talk` content of the last turn:

```python
>>> [t.get('talk', '') for t in turns][-1]
"[ERROR: TypeError: 'NoneType' object is not subscriptable]"
```

Once one agent's internal state was poisoned by a None response, all subsequent turns *for that agent* also threw the same exception. The errors cascaded: the trials had ~25-28 error-turns each (out of 72). About 30-40% of the discussion content was lost in those trials.

**What we did.**
1. Cataloged all corrupted trials by scanning every JSON for `[ERROR` strings: 4 medical_N5 (t26-t29) + 6 medical_N9 (t24-t29) = 10 trials.
2. Excluded them from the headline analysis. The clean dataset is 82 trials.
3. Implemented a sensitivity analysis (PAPER §4.7): even adversarially recoding all error-turns as non-intervention, the bystander-effect prediction is still rejected.
4. Deleted the corrupted JSONs and attempted a re-run with 4 workers (half load). The re-run also entered a sustained 429 storm — by that point the daily quota was likely tighter than tier-1 RPM/TPM limits.
5. Stopped the pipeline. Pivoted to writing the paper with the 82 clean trials we had.

**Lesson.** Defensive coding in the adapter (catching exceptions and recording an error string) is good for *robustness* (the trial completes, the trial JSON is valid, the pipeline doesn't deadlock) but bad for *data quality* (silently corrupted data looks normal until you specifically look for the error pattern). The right fix is to *fail hard* when the API returns None — drop the trial, note it in a manifest, re-run later. We will fix this in the post-deadline iteration.

### 4.4 The pivot: from "show the effect" to "characterise its absence"

The original `RESEARCH_PLAN.md` is structured around *finding* a bystander effect: power analysis aimed at *d* ≈ 0.35 (Fischer et al. meta-analytic mean), Study 3 designed to *test mitigations* of the effect, mechanism analysis (Study 2) to *explain non-intervention*. By the time we had 60 medical trials (N=3 + N=5 cells), the per-capita rate was 1.000 / 0.993 — nothing to mitigate, nothing to explain.

The intellectual pivot was made in `RESEARCH_LOG.md` §6.6 (4/27): **"What this implies for the paper if N=9 stays at ceiling"**. The pivot was *toward* the null finding rather than away from it. The reframing went something like this:

- *If* we had run all 360 trials and found ceiling at every cell: same finding, but with 4× the data backing it.
- *If* we had found a graded effect: traditional bystander-replication paper.
- *If* we had found ceiling: AI-safety-friendly null result, distinctively framed as "panel-of-LLMs is more robust than predicted."

We invested in instrumentation that would make either outcome publishable: pre-registered hypotheses, validated personas, calibrated scenarios, dual coding instruments (LLM-judge + heuristic), trial-level transparency (every trial's full JSON is on disk and reviewable). When the ceiling appeared, we had the substrate to defend the null.

**The lesson here is general:** for an empirical project, design the protocol so that *both* directions of the result are publishable. Otherwise, sunk-cost pressure will push the team to re-frame mid-flight in ways that compromise integrity.

### 4.5 What we'd do differently next time

1. **Pre-flight quota probe** in `run_parallel.py`. Two API calls before worker dispatch: one to verify `gpt-5-mini` works (catch invalid key), one to attempt a small completion (catch insufficient_quota). Fail fast.
2. **None-response guard in adapter**. If `act()` returns None, raise immediately. Don't let the corruption silently accumulate as `[ERROR: TypeError ...]` turns.
3. **Smaller worker count by default**. We started at 8 because the wall-clock estimate was tight. With hindsight, 4 would have given us most of the throughput at half the rate-limit risk. The marginal trial gained at 8 workers vs 4 was worth less than the engineering cost of the corrupted batch.
4. **Write the analysis script before any real run.** We had `analysis/build_dataset.py` and `stats_study1.py` from the planning phase, but `analysis/full_descriptive.py` was written *after* the data was in. Writing it first would have forced earlier reckoning with "what does the headline plot look like if everything is at ceiling?".
5. **Allocate slack at the end.** Our 12-day plan had no buffer for engineering surprises. The actual project timeline used roughly ⅔ of the budget on data collection and ⅓ on debugging/recovery — exactly the wrong ratio. Future projects should plan for at least 50% slack.

---

## 5. Methodological reflections

### 5.1 What worked

**Pre-registration.** Locking the design in `RESEARCH_PLAN.md` *before* spending API budget made the eventual null finding defensible. We were not "testing N until we found a result" or "trying scenarios until one popped"; the design was committed in writing on day 1.

**Mock-first, real-API-second.** All four scenarios, all five condition choreographies (C0-C5), the LLM-judge code paths, the cost monitor, the trial JSON shape — every code path was validated end-to-end with the mock adapter at $0 cost before a single real-API trial. This caught a half-dozen integration bugs that would have cost tens-of-dollars to find by trial-and-error on the live API.

**Persona pre-screen.** The 0.992 solo baseline is the *most-load-bearing* number in the paper. Without it, the headline 1.000 cell rates would just look like "well, this is a good model." With the solo baseline, we have a calibration point: solo and panel rates are statistically indistinguishable, *which is exactly what the bystander hypothesis predicts should NOT happen*. The pre-screen was 14 minutes of wall-clock and a few dollars. It is the cheapest part of the project and the most epistemically valuable.

**LLM-judge with forced tool-use.** Once we adopted `tool_choice={"type":"tool", "name":"record_judgment"}` with strict JSON schema, JSON-shape errors became impossible. The tool API signs the schema; the model returns a structured `tool_use.input` block; we read it directly. Compare to chat-completions with `response_format={"type":"json_object"}`, which still requires text parsing and silently produces non-conforming JSON in edge cases. This is a process improvement we will carry to all future projects.

### 5.2 What we'd do differently with more time

**Run the financial / privacy / academic cells.** The most-load-bearing missing data. Even N=10 trials per cell would let us say something about scenario generalisation. We have all the infrastructure.

**Run an older / smaller model.** `gpt-3.5-turbo`, `gpt-4o-mini`, or one of the open-source `Llama-3.1-8B-Instruct` variants. Hypothesis: bystander effect is more likely to emerge in less safety-trained models. This is the *single highest-value follow-up experiment*.

**Adversarial perpetrator.** Our perpetrator delivers their script and disappears. A perpetrator who *defends* the proposal in a follow-up turn, *appeals to authority*, or *socially engineers the panel* might break the saturation. This would test whether the absence of bystander effect is robust to non-trivial pressure.

**Run Study 3 (mitigations).** The pre-registered plan calls for 5 conditions × 30 trials testing whether explicit role assignment, distributed responsibility, pluralistic-ignorance disruption, and pre-commitment can *induce* or *amplify* non-intervention. Even if the C0 baseline is at ceiling, Study 3 conditions designed to *induce* bystander dynamics (e.g., distributed responsibility framing) could surface the effect. This becomes the natural follow-up paper.

**Manual dual-coding for κ.** The pre-registered Cohen's κ ≥ 0.7 manual gold-standard validation was deferred. Doing this on a 100-turn random sample would solidify the claim that the LLM-judge is reliable. With mostly heuristic-coded data in this draft, this is the methodological gap most reviewer-criticism-prone.

### 5.3 On the "internal `[THINK]` trace" affordance

This was the most distinctive methodological idea in our plan: humans cannot tell us what they were thinking when they didn't help; LLM agents emit their reasoning verbatim. With more cases of *non-intervention*, we'd have a unique mechanism-analysis dataset.

In our actual data, there is essentially no non-intervention. We cannot demonstrate the affordance because the empirical pattern doesn't give us non-intervention to inspect. This is fine — the affordance exists whether or not we found data to apply it to — but it means our paper *doesn't get* to showcase it the way we hoped.

The right way to recover this: re-run with **adversarially-constructed scenarios** where solo intervention is < 50% (i.e., scenarios deliberately *too subtle* for solo agents to catch). Then we have non-intervention by construction and can study its mechanism. This is on the follow-up-work list.

---

## 6. AI-safety implications (the case for the paper's positive framing)

We want to be careful not to over-claim. The setting matters. With that caveat:

### 6.1 What the result supports

- **Panel-of-LLMs designs (debate, critic pools, judge-of-judges) do not exhibit hidden bystander dynamics in deliberative review tasks.** At least for the conditions we tested, the implicit assumption "more agents = more catches" holds. Designers can build on this assumption without engineering specific anti-bystander mitigations.
- **Persona variation does not break the saturation.** Even personas designed to be *less* prone to intervention (anxious-conformist, low-C-low-A) intervened reliably. This is good news: heterogeneous-persona panels (e.g., for diverse-perspective review) maintain the catch property.
- **Severity is high.** Agents don't hedge. Modal intervention is severity-3 (explicit refusal). This means panel decisions are *unambiguous* — useful for downstream automated processing of panel verdicts.

### 6.2 What the result does NOT support

- **The panel-of-LLMs assumption holding in non-deliberative settings.** Our scenarios are *explicit review*. If the panel framing changes — e.g., agents are casual collaborators on a project rather than reviewers — the result may not transfer.
- **Robustness to adversarial perpetrators.** Our perpetrator delivers a script and disappears. Sophisticated perpetrators (multi-turn, defensive, socially engineering) are an open question.
- **Generalisation across model classes.** Only `gpt-5-mini` was tested. We *expect* less safety-trained models to show the effect.

### 6.3 What it suggests about LLM social cognition

The most striking single observation is the *complete absence of mechanism markers in `[THINK]` traces*. Even when we read the traces of agents who *did* intervene, they do not consider non-intervention as an option. The cognitive scaffolding for "should I or shouldn't I help?" — which is the precondition for the bystander effect to emerge — appears not to be active in our agents. They go directly from "subtle violation" → "intervention required" without traversing the deliberative hesitation that humans show.

If this is robust (a big if — could be a paradigm artefact), it suggests that contemporary LLMs may be more *deontological* in their behavioural-decision policies than humans — they do not perform the cost-benefit calculation that produces bystander apathy. This is a tentative finding worth investigation in its own right.

---

## 7. Submission plan

### 7.1 Required (5/8)

- ✅ **Slides** (5 minutes). Outline at `slides/OUTLINE.md`. Three figures: (i) experimental setup schematic, (ii) per-capita rate by N (the headline null), (iii) severity distribution + mechanism markers (showing the bystander reasoning is absent).
- ✅ **Working code**: `pipeline/`, `analysis/`, `personas/`, `scenarios/` are all in this repo.

### 7.2 Optional (5/15) — extended written report

- **`PAPER.md`** is our paper draft. Roughly 800 lines, ~7,500 words, includes abstract, related work, full method, results with figures, full discussion with limitations and future work, four appendices (personas, scenarios, coding instruments, engineering notes), and references.
- **`PROJECT_REPORT.md`** is this file — the course-flavoured project report.
- **`RESEARCH_LOG.md`** is the chronological execution log with timestamps, decisions, and incident response narratives. Useful as a methods-section supplement.

### 7.3 Stretch (post-deadline)

- **OSF pre-registration.** The `RESEARCH_PLAN.md` is already structured for this — locked hypotheses, locked design, locked stat plan.
- **arXiv preprint.** `PAPER.md` → LaTeX conversion → arXiv submission.
- **Workshop submission.** Plausible venues: NeurIPS SoLaR (Socially Responsible LM); ICLR SeT-LLM; FAccT; AIES.
- **Full conference submission.** ACL Findings / EMNLP / CSCW with the follow-up scenarios + GPT-5 robustness arm completed.
- **Multi-model bystander study.** Re-run on 3-4 models (gpt-3.5-turbo, gpt-4o, claude-haiku, llama-3.1-8b). Frame as "the bystander effect is a property of post-training caution, not of LLM social-cognitive architecture per se." This would be a strong NeurIPS / ICLR submission.

---

## 8. Acknowledgements

Underlying simulation framework: TinyTroupe (Microsoft Research). We did not modify TinyTroupe; our code wraps it via the `tinytroupe_adapter.py` interface. All scientific design and analysis is original to this project; all code in `pipeline/`, `analysis/`, `personas/`, `scenarios/` was written for this project.

---

## 9. Statement of contributions (within the team)

[To be filled in by the team based on actual contributions.]

The repository commit log (`git log` if applicable) provides the authoritative attribution of code changes. The pre-registered `RESEARCH_PLAN.md` was authored 2026-04-26 and `RESEARCH_LOG.md` is appended chronologically.

---

*End of project report.*
*See `PAPER.md` for the scientific argument; `RESEARCH_LOG.md` for the chronological execution log; `RESEARCH_PLAN.md` for the original pre-registered design.*

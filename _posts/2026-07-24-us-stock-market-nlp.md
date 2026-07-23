---
layout: post
title: "Predicting Financial Direction from 342 Annual Reports"
date: 2026-07-24 01:02:30 +0800
permalink: /projects/us-stock-market-nlp/
categories: [研究紀錄]
tags: [Project, NLP, Machine Learning, AI Assisted]
published: true
---

This five-person project tested whether the language in a company's annual
report could help predict the direction of next year's gross margin or return
on assets.

The dataset contained 342 U.S. listed companies. For each company, the team
extracted roughly 2,000 to 5,000 words from the Management Discussion and
Analysis section of its annual report, then labeled the following year's
financial change as growth, unchanged, or decline.

The [report, code, and
data](https://github.com/AAAA-source/US-Stock-Market-Analysis-and-Prediction)
are public.

## Turning MD&A language into labels

The study used two targets:

- gross-margin change;
- return-on-assets change.

A change greater than 1% was labeled growth, a change below -1% was labeled
decline, and values between those thresholds were treated as unchanged.

Most models received TF-IDF, term-frequency, or multi-hot text features after
stop-word removal and document-frequency filtering. The repository contains
Naive Bayes variants, SVM, Rocchio, KNN, clustering, dimensionality reduction,
a simple neural network, and RNN experiments.

The project did not try to predict a stock price. It predicted the direction of
two accounting ratios from MD&A text.

## Why the report introduced "non-fail rate"

Precision alone treated all wrong classes equally. The team wanted a second
measure for an investor who mainly wished to avoid predicting growth when the
company later declined, or predicting decline when it later grew.

For a growth prediction, the "non-fail" count included actual growth and
unchanged cases. For a decline prediction, it included actual decline and
unchanged cases. Several individual and merged models reached rates around
80%, with some class-specific results higher.

That metric needs a careful boundary. It is not portfolio return, trading
accuracy, or evidence that a strategy beats a market benchmark. Counting an
unchanged company as non-failure also makes it less strict than three-class
accuracy. The name describes the team's chosen risk framing, not a standard
financial metric.

## Combining models

The final experiments combined SVM and Naive Bayes outputs. The merge used each
model's observed performance, confidence, and thresholds to choose a more
conservative class. The report says this improved the balance between precision
and non-fail rate, but the small dataset limits how much can be inferred from
the result.

The presentation assigns my work to hierarchical agglomerative clustering,
RNN experiments, the proposal, and shared merge-model work. The GitHub history
is a bulk upload under another teammate's account, so the presentation is the
source for that division rather than per-file commit attribution.

## What the experiment did not establish

The sample was small for the number of model variants, and the class
distribution was uneven. The report does not document a market benchmark,
transaction costs, time-split backtest, or out-of-sample trading simulation.
The approximately 80% figure should therefore remain attached to the project's
own non-fail definition.

The useful record is narrower: a team assembled an MD&A dataset, compared
traditional and neural text models, inspected the error direction through
confusion matrices, and designed an ensemble around the errors it cared about.

---

**Context:** Five-person text-mining team project<br>
**My documented work:** HAC, RNN experiments, proposal, and shared merge-model work<br>
**Status:** Completed public course artifact; investment usefulness not validated

# Product Experimentation Framework

> A lightweight A/B testing and feature experimentation framework built for fintech product teams. Enables data-driven product decisions with statistical rigor — adopted company-wide at E&M Technology House Uganda.

---

## Table of Contents
- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Core Concepts](#core-concepts)
- [Example Experiments](#example-experiments)
- [Key Results](#key-results)
- [Running Locally](#running-locally)
- [What I Learned](#what-i-learned)

---

## Overview

Before this framework, product decisions at E&M Tech House were made on intuition and anecdote. A new UI flow might ship because someone on the team "felt" it would work better. This framework replaced that with a structured, statistically sound process for validating hypotheses before full rollout.

It handles the full experiment lifecycle:
- Define a hypothesis and success metric
- Split users into control/treatment groups (deterministically, so the same user always sees the same variant)
- Collect event data during the experiment window
- Run statistical significance tests
- Produce a clear yes/no recommendation with confidence intervals

---

## Problem Statement

Product and engineering teams needed answers to questions like:
- "Does the new payment confirmation screen reduce drop-offs?"
- "Does showing transaction history increase mobile money usage?"
- "Which onboarding flow leads to more account activations?"

Without a framework, answering these required either guesswork or ad-hoc SQL queries that varied by analyst. The framework standardizes the entire process.

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    Experiment lifecycle                         │
│                                                                 │
│  1. DEFINE       2. ASSIGN        3. COLLECT     4. ANALYSE    │
│  ─────────       ─────────        ─────────      ────────      │
│  Hypothesis →    User hash →      Events log →   Stats test →  │
│  Metric     →    Control or       Conversion      Significant? │
│  Duration        Treatment        rates           Recommend    │
└─────────────────────────────────────────────────────────────────┘
```

**User assignment is deterministic:**  
Given `hash(user_id + experiment_id) % 100`, a user is always assigned to the same group. This prevents the "flickering" problem where users see different variants on repeated visits.

**Statistical test:**  
For binary outcomes (converted / didn't convert), we use a two-proportion z-test. For continuous outcomes (transaction amount), we use a t-test. Both report a p-value and 95% confidence interval.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Core logic | Python |
| Statistical tests | SciPy |
| Data storage | PostgreSQL |
| Analysis & reporting | Pandas, Matplotlib |
| Notebook walkthrough | Jupyter |

---

## Project Structure

```
experimentation-framework/
│
├── framework/
│   ├── experiment.py     # Experiment definition and user assignment
│   ├── tracker.py        # Event logging (conversions, metrics)
│   ├── analyser.py       # Statistical tests and result interpretation
│   └── report.py         # Summary report generation
│
├── experiments/
│   ├── payment_flow_v2.py          # Example: payment screen A/B test
│   └── onboarding_cta_test.py      # Example: button colour test
│
├── notebooks/
│   └── experimentation_framework.ipynb   # Full walkthrough (start here)
│
├── requirements.txt
└── README.md
```

---

## Core Concepts

### Hypothesis format
Every experiment starts with a structured hypothesis:
```
IF we [change],
THEN [metric] will [increase/decrease] by [amount],
BECAUSE [reasoning].
```

Example:
> IF we show the transaction summary before the confirmation button,  
> THEN payment completion rate will increase by ≥ 5%,  
> BECAUSE users are more confident when they can review details.

### Minimum Detectable Effect (MDE)
Before running an experiment, we calculate the minimum sample size needed to detect a meaningful difference. Running under-powered experiments is one of the most common mistakes in A/B testing — you get a "not significant" result and can't tell if the change genuinely didn't work, or if you just didn't have enough data.

### Statistical significance vs. practical significance
A result can be statistically significant (p < 0.05) but practically meaningless (e.g. +0.1% conversion). The framework reports both the p-value and the absolute effect size so teams make decisions on business impact, not just math.

---

## Example Experiments

### Experiment 1: Payment confirmation screen
- **Hypothesis:** Showing a transaction summary before confirm button increases completion
- **Metric:** Payment completion rate
- **Result:** +7.3% lift, p=0.003 → Shipped to 100% of users

### Experiment 2: Onboarding CTA button colour
- **Hypothesis:** Green CTA button increases account activations vs. grey
- **Metric:** Account activation rate (7-day)
- **Result:** +1.1% lift, p=0.21 → Not significant, did not ship

### Experiment 3: Transaction history visibility
- **Hypothesis:** Showing last 5 transactions on home screen increases repeat usage
- **Metric:** DAU/MAU ratio (stickiness)
- **Result:** +4.8% lift, p=0.018 → Shipped

---

## Key Results

| Metric | Value |
|--------|-------|
| Experiments run | 12+ |
| Decisions informed | All major product releases (Q1–Q2 2026) |
| Average experiment duration | 14 days |
| Adoption | Used by product, engineering, and data teams |

---

## Running Locally

```bash
# 1. Clone the repo
git clone https://github.com/your-username/experimentation-framework.git
cd experimentation-framework

# 2. Install dependencies
pip install -r requirements.txt

# 3. Open the notebook
jupyter notebook notebooks/experimentation_framework.ipynb
```

---

## What I Learned

- **Deterministic assignment matters** — random assignment per session causes users to flip between variants, corrupting your data
- **Run power calculations before, not after** — deciding sample size after seeing results is p-hacking
- **Novelty effect is real** — new UI changes often get a short-term lift just because they're new; running experiments for at least 1–2 weeks filters this out
- **Shipping ≠ success** — the most valuable output of some experiments was the decision *not* to ship a feature, saving engineering time

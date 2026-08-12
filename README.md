# SaaS Customer Churn — Root-Cause Analysis

Diagnostic analysis of subscription churn for a fictional productivity
SaaS tool, built to answer a growth-team question: which customers are
actually at risk of churning, and why — before spending on a broad
retention campaign that treats every customer the same way.

> **Disclaimer:** Fictional SaaS company, synthetically generated
> dataset. Built specifically because the two most important hypotheses
> (gradual usage decline vs. usage-spike-then-churn) required real
> day-by-day usage history that no single public snapshot dataset
> provides — see Methodology below.

## Business Context

> "Churn isn't stable month to month, and the Customer Success team says
> some heavily-engaged customers cancel out of nowhere, while some
> barely-active customers stay subscribed for a long time. We need to
> understand who's actually at risk before investing in an expensive
> retention campaign aimed at everyone."

## Methodology

The request was split into a neutral problem statement and 5 competing,
independently-tested hypotheses:

| # | Hypothesis | Result |
|---|---|---|
| H1 | Seasonality (usage & billing-cycle effects) | ✅ **Confirmed**, with a twist — usage drops in December as expected, but churn spikes *in* December, not the following January as originally assumed |
| H2 | Price shock (a plan price increase) | ✅ **Confirmed, directionally** — Business-plan churn rose 5x after the price increase vs. 2.3x for the unaffected Pro-plan control group. Small sample (7 customers in the affected pattern) — a real signal, not a certainty |
| H3 | Gradual, unmonitored usage decline | ✅ **Confirmed** — 106 customers, usage fades steadily well before cancellation |
| H4 | Usage-spike-then-churn ("goal achievement") | ✅ **Confirmed — the opposite pattern of H3** — 54 customers ramp usage sharply in their final ~20 days, then cancel |
| H5 | Support ticket severity & timing | ✅ **Confirmed, partially** — high-severity tickets predict *whether* a customer churns (3.8x higher rate) but not *when* — timing is similar regardless of severity |

## Key Findings

- **Two opposite usage patterns drive churn, and a single "usage
  dropped" alert would only catch one of them.** Gradual-decline
  customers show a long, low, fading usage signal — ideal for automated
  early-warning. Goal-achievement customers stay flat for months, then
  spike hard right before cancelling — that's not disengagement, it's a
  customer who got what they needed and is a better upsell candidate than
  a retention target.
- **Seasonality operates faster than assumed.** December's usage dip
  translates into a same-month churn spike, not a delayed one — the data
  didn't support the "people cancel after the holidays" theory that
  seemed intuitive going in.
- **Support tickets give a genuinely useful window for intervention.**
  The median ticket arrives 57 days before cancellation, and severity
  doesn't compress that window — meaning any complaint, not just a
  severe one, is worth treating as an early signal.
- **A price increase drove excess churn on the affected plan** — but the
  affected sample is small enough that this is treated as directional
  evidence for negotiation/monitoring, not grounds for reversing the
  price change outright.

## A Modeling Note Worth Reading

Testing H3/H4/H5 required pulling values across multiple fact tables
(e.g. "how many days before this customer's churn date did this usage
event happen?"). The direct approach hits a real Power BI modeling
constraint — relationship loops — documented with the full fix in
[`dashboard/DAX_measures.md`](dashboard/DAX_measures.md), including every
place it had to be applied and why.

## Repository Structure

```
data-generation/
└── generate_saas_data.py     # Synthetic dataset generator (seeded, reproducible)

dashboard/
└── DAX_measures.md           # Every DAX formula, including the relationship-loop fix

EXECUTIVE_SUMMARY.md          # Stakeholder-facing summary and recommendations
```

## Reproducing the data

```bash
cd data-generation
python generate_saas_data.py
```

## Tools

Python (pandas, numpy) · Power BI (DAX, `USERELATIONSHIP`, Decomposition
Tree, data modeling)

# Replacing a Fixed Video-Length Rule with a Visual Risk Score

## Product Analytics Project

### Overview

A video reconstruction pipeline was using a simple duration-based routing rule:

- **≤ 5 seconds → automated reconstruction**
- **> 5 seconds → manual review**

The problem: **duration is only a proxy for visual complexity**. Some longer clips can be visually simple, while some shorter clips can contain substantial visual change.

The goal was to explore whether visual characteristics could provide a more useful, transparent screening rule and safely expand automated eligibility toward the **5–7 second band**, which represented costly manual routing.

## The Question

> **Can we quantify visual characteristics associated with reconstruction risk and use them to decide which clips are safer to automate?**

Rather than building a black-box prediction model, I built a small, interpretable screening heuristic from three video-level signals.

## Dataset

I analyzed **100 video clips** and compared quantitative signals with manual observations of scene changes and visual behavior.

The analysis was performed in **Python/Pandas**.

## The Three Signals

### 1. Mean Frame Difference
Measures the overall amount of frame-to-frame visual change across the clip.

### 2. Max Frame Difference
Captures the largest instantaneous frame-to-frame change.

### 3. Max Histogram Difference
Captures the largest change in the overall visual/compositional distribution.

These signals were selected because they capture different aspects of visual change rather than relying on a single metric.

## What the Data Showed

Pairwise correlations across the 100 clips:

| Signal Pair | Correlation |
|---|---:|
| Mean Frame ↔ Max Frame | 0.67 |
| Max Frame ↔ Max Histogram | 0.72 |
| Mean Frame ↔ Max Histogram | 0.35 |

The signals are related, but not identical. This supported keeping all three rather than selecting a single “best” metric.

## From Three Signals to One Risk Score

Because the three measurements operate on different scales, I normalized them and combined them into a single transparent score.

### Weighting

```text
Risk Score =
    40% × Mean Frame Signal
  + 30% × Max Frame Signal
  + 30% × Max Histogram Signal
```

The resulting score provides a single operational signal for routing:

```text
Lower visual-change risk → Eligible
Higher visual-change risk → Exclude
```

This was intentionally designed as a **screening heuristic**, not a trained machine-learning model.

## Example

For **Block Blast.mp4**, the normalized signals were:

| Signal | Normalized Value |
|---|---:|
| Mean Frame Difference | 0.95 |
| Max Frame Difference | 0.79 |
| Max Histogram Difference | 0.67 |

The resulting risk score was:

**81.8 → EXCLUDE**

This shows how three separate visual-change measurements can be converted into one traceable routing decision.

## Model vs. Manual Review

Across the 100 clips:

- **68 → Model: ELIGIBLE**
- **32 → Model: EXCLUDE**

Comparison with manual routing:

| | Manual: ELIGIBLE | Manual: EXCLUDE |
|---|---:|---:|
| **Model: ELIGIBLE** | 54 | 18 |
| **Model: EXCLUDE** | 14 | 14 |

The important takeaway was not that the heuristic perfectly reproduced manual judgment.

> **The score doesn't eliminate judgment — it concentrates it where the signals are ambiguous.**

## Why This Approach?

I deliberately kept the approach:

- **Interpretable** — each component has a clear meaning.
- **Transparent** — the final score can be decomposed into its inputs.
- **Lightweight** — no complex model training required.
- **Operational** — produces a simple Eligible/Exclude routing decision.
- **Adjustable** — weights and thresholds can be tested rather than treated as fixed truths.

## Limitations

This is an exploratory screening framework rather than a validated predictor of reconstruction failure.

The 100-clip dataset and manual observations provide a basis for exploration, but stronger validation would require:

1. Actual reconstruction outcomes for each clip.
2. A larger labeled dataset.
3. A clearly defined ground-truth failure criterion.
4. Out-of-sample validation.
5. Threshold selection based on the business costs of false inclusion vs. false exclusion.

The next step would be to test whether the visual risk score predicts **actual reconstruction failures**, rather than only matching manual assessments.

## Key Takeaway

A fixed duration cutoff treats all clips of the same length as equally risky.

This project explored a different idea:

> **Use measurable visual behavior to make routing more context-aware.**

The result is a simple, explainable framework that turns three visual-change signals into a single screening score and creates a basis for evaluating whether the **5–7 second band** can be handled more intelligently.

## Tools

- Python
- Pandas
- NumPy
- Statistical analysis
- Video-level frame and histogram difference metrics

## Note

This project focuses on the **analytics and decision framework**. The risk score is a heuristic for screening and should not be described as a production-grade machine-learning model without outcome-based validation.

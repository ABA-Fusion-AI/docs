---
node_id: "t-test"
title: "t-test"
description: "Performs one-sample, two-sample (Welch), or paired t-tests with approximate p-values."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - t-test
  - statistics
  - hypothesis-testing
  - welch-t-test
  - paired-t-test
  - p-value
  - data-analysis
related_nodes:
  - anova
  - chi-square
  - ks-test
  - normality
  - confidence-interval
---

<!-- SECTION: header -->
# t-test

> **Category:** Mathematical & Statistical Analysis | **Subcategory:** Statistics & Experiments | **Type:** Action Node

Perform One-Sample, Two-Sample (Welch's), or Paired Student's t-tests with p-values and effect sizes.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **t-test** node performs statistical hypothesis testing on numerical datasets to determine if sample means differ significantly from a known baseline, from each other, or across paired repeated measurements.

It supports three test types:

1. **One-Sample t-test (`one_sample`):** Compares a sample dataset against a hypothesized baseline mean ($\mu_0$).
2. **Two-Sample Welch's t-test (`two_sample`):** Compares two independent sample groups ($A$ and $B$) without assuming equal population variances (using the Welch–Satterthwaite approximation), and computes Cohen's $d$ and Hedges' $g$ effect sizes.
3. **Paired t-test (`paired`):** Compares repeated observations from the same subjects (e.g., pre-treatment vs. post-treatment measurements).

### Key Features

- **Three Supported Test Types:** Easily switch between One-Sample, Two-Sample Welch, and Paired t-tests.
- **Directional Hypotheses:** Support for two-sided ($H_1: \mu \ne \mu_0$), right-tailed / greater ($H_1: \mu > \mu_0$), and left-tailed / less ($H_1: \mu < \mu_0$) alternative hypotheses.
- **Welch's Variance Correction:** Avoids false positives in two-sample tests by dynamically calculating adjusted degrees of freedom without assuming homoscedasticity.
- **Effect Size Metrics:** Reports standardized effect sizes including Cohen's $d$ and small-sample-corrected Hedges' $g$.
- **Instant In-Memory Computation:** Evaluates statistical formulas synchronously with zero external latency.

### Processing Flow

```text
Incoming Dataset / Trigger
  ↓
Validate Parameters (Test kind, minLength ≥ 2, equal paired lengths)
  ↓
Select Statistical Model:
  ├─ one_sample → Calculate sample mean, SE, t-statistic, and p-value against mu0
  ├─ two_sample → Calculate Welch SE, Welch df, t-statistic, Cohen's d, Hedges' g
  └─ paired     → Compute differences d = a - b, evaluate as one-sample against 0
  ↓
Emit Output Object { test, t, df, pValueApprox, ... }
```

### Use Cases

- **A/B Testing & Feature Evaluation:** Compare conversion rates or user engagement scores between control and variant groups.
- **Pre / Post Experiment Analysis:** Measure the effectiveness of a medical treatment, marketing campaign, or system optimization by running a paired t-test on before/after metrics.
- **Quality Control & SLA Benchmarking:** Test whether manufacturing measurements or API response times deviate significantly from target thresholds ($\mu_0$).
- **Academic & Scientific Automation:** Automate statistical data validation within data science and bioinformatics pipelines.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `kind` | `enum` | Yes | `one_sample` | The type of t-test to perform: `one_sample`, `two_sample`, or `paired`. |
| `alternative` | `enum` | No | `two-sided` | The alternative hypothesis direction: `two-sided` ($\ne$), `greater` ($>$), or `less` ($<$). |
| `series` | `array<number>` | Conditional | — | Sample dataset used when `kind` is `one_sample` (minimum 2 values). |
| `mu0` | `number` | No | `0` | Hypothesized population mean $\mu_0$ used when `kind` is `one_sample`. |
| `a` | `array<number>` | Conditional | — | First sample dataset used when `kind` is `two_sample` (minimum 2 values). |
| `b` | `array<number>` | Conditional | — | Second sample dataset used when `kind` is `two_sample` (minimum 2 values). |
| `aPaired` | `array<number>` | Conditional | — | Pre-treatment or baseline measurements used when `kind` is `paired` (minimum 2 values). |
| `bPaired` | `array<number>` | Conditional | — | Post-treatment or follow-up measurements used when `kind` is `paired` (must have the same length as `aPaired`). |

### Test Types Reference

| Kind | Required Fields | Formula Summary |
|------|-----------------|-----------------|
| `one_sample` | `series`, `mu0` (optional) | $t = \frac{\bar{x} - \mu_0}{s / \sqrt{n}}$, $df = n - 1$ |
| `two_sample` | `a`, `b` | Welch $t = \frac{\bar{x}_1 - \bar{x}_2}{\sqrt{s_1^2/n_1 + s_2^2/n_2}}$, adjusted $df$ |
| `paired` | `aPaired`, `bPaired` | $d_i = a_i - b_i$, $t = \frac{\bar{d} - 0}{s_d / \sqrt{n}}$, $df = n - 1$ |

### Default Configuration

```json
{
  "kind": "one_sample",
  "alternative": "two-sided",
  "mu0": 10,
  "series": [12, 15, 14, 16, 18, 20, 19]
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or data passed from previous analysis nodes. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful calculation, containing test statistics ($t$, $df$, $p$-value, effect sizes). |
| `error` | `object` | Returned if arrays contain fewer than 2 elements or if paired arrays have mismatched lengths. |

### Output Schemas

#### One-Sample & Paired Output Shape

```typescript
{
  test: string;           // "one_sample_t_approx"
  n: number;              // Sample size (or number of pairs)
  mean: number;           // Calculated sample mean (or mean difference)
  mu0: number;            // Hypothesized baseline mean
  t: number;              // t-statistic value
  df: number;             // Degrees of freedom (n - 1)
  pValueApprox: number;   // Approximate p-value
}
```

#### Two-Sample Welch Output Shape

```typescript
{
  test: string;           // "two_sample_welch_t_approx"
  meanA: number;          // Mean of sample group A
  meanB: number;          // Mean of sample group B
  t: number;              // Welch t-statistic value
  df: number;             // Welch–Satterthwaite degrees of freedom
  pValueApprox: number;   // Approximate p-value
  cohenD: number;         // Cohen's d standardized effect size
  hedgesG: number;        // Hedges' g small-sample corrected effect size
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: One-Sample Two-Sided Test

Test whether a dataset's mean is significantly different from $\mu_0 = 10$:

```json
{
  "kind": "one_sample",
  "alternative": "two-sided",
  "mu0": 10,
  "series": [12, 15, 14, 16, 18, 20, 19]
}
```

**Output:**
```json
{
  "test": "one_sample_t_approx",
  "n": 7,
  "mean": 16.285714,
  "mu0": 10,
  "t": 5.753303,
  "df": 6,
  "pValueApprox": 0.000000087
}
```

### Example 2: One-Sample Greater (One-Tailed)

Test if average system throughput exceeds baseline SLA ($\mu_0 = 100$):

```json
{
  "kind": "one_sample",
  "alternative": "greater",
  "mu0": 100,
  "series": [105, 110, 108, 112, 115]
}
```

**Output:**
```json
{
  "test": "one_sample_t_approx",
  "n": 5,
  "mean": 110,
  "mu0": 100,
  "t": 5.700877,
  "df": 4,
  "pValueApprox": 0.000000006
}
```

### Example 3: Two-Sample Welch Test (A/B Test)

Compare two independent groups ($A$ vs. $B$) without assuming equal variance:

```json
{
  "kind": "two_sample",
  "alternative": "two-sided",
  "a": [25, 28, 30, 29, 27, 31],
  "b": [18, 20, 19, 22, 21, 10]
}
```

**Output:**
```json
{
  "test": "two_sample_welch_t_approx",
  "meanA": 28.333333,
  "meanB": 18.333333,
  "t": 4.902903,
  "df": 6.133207,
  "pValueApprox": 0.000000944,
  "cohenD": 2.830694,
  "hedgesG": 2.594803
}
```

### Example 4: Paired t-test (Pre vs. Post Treatment)

Measure the change across subjects before ($A$) and after ($B$) an intervention:

```json
{
  "kind": "paired",
  "alternative": "two-sided",
  "aPaired": [140, 145, 150, 142, 148],
  "bPaired": [130, 135, 138, 132, 126]
}
```

**Output:**
```json
{
  "test": "one_sample_t_approx",
  "n": 5,
  "mean": 12.8,
  "mu0": 0,
  "t": 5.485705,
  "df": 4,
  "pValueApprox": 0.000000041
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Run one-sample, Welch two-sample, and paired t-tests
```

### Common Workflow Patterns

- **Automated A/B Significance Check:** Trigger → t-test (`kind: "two_sample"`) → If/Else (`condition: {{ $node["t-test"].pValueApprox < 0.05 }}`) → Slack / Email Alert ("Statistically significant improvement").
- **Quality Benchmark Alerting:** Scheduled Trigger → Fetch Production Metrics → t-test (`kind: "one_sample"`, `mu0: 100`, `alternative: "less"`) → Incident Response.
- **Medical & Clinical Follow-ups:** Database Query (Pre/Post Patient Scores) → t-test (`kind: "paired"`) → Google Sheets (Append Study Results).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "a and b must have same length (paired)"

**Cause:** In a paired t-test (`kind: "paired"`), the array `aPaired` and `aPaired` have different numbers of elements.

**Solution:** Ensure both arrays contain the exact same number of paired observations (each index $i$ corresponds to the same subject).

### Error: Sample size is less than 2

**Cause:** An input array contains 0 or 1 number, which is insufficient to compute sample variance or degrees of freedom ($df = n - 1$).

**Solution:** Provide at least 2 numerical observations in each sample array.

### Interpreting the p-value (`pValueApprox`)

- If `pValueApprox < 0.05`, reject the null hypothesis $H_0$ (the difference is statistically significant at the 95% confidence level).
- If `pValueApprox >= 0.05`, fail to reject $H_0$ (there is insufficient evidence to conclude a significant difference).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [ANOVA](../anova/en.md) - Compare means across three or more sample groups
- [KS Test](../ks-test/en.md) - Test distribution shapes and two-sample distribution equality
- [Normality](../normality/en.md) - Verify whether sample data follows a normal distribution before running parametric t-tests
- [Confidence Interval](../confidence-interval/en.md) - Calculate upper and lower confidence bounds for sample means

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for t-test node |

<!-- /SECTION: changelog -->

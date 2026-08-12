---
node_id: "anova"
title: "ANOVA One-Way"
description: "Performs one-way ANOVA (Analysis of Variance) with F-statistic and approximate p-value."
category: "utilities"
subcategory: "statistics"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:
  - statistics
  - anova
  - analysis-of-variance
  - one-way-anova
  - f-test
  - hypothesis-testing
  - p-value
related_nodes:
  - bootstrap
  - function
  - log
---

<!-- SECTION: header -->
# ANOVA One-Way

> **Category:** Utilities | **Type:** Action Node

Performs one-way ANOVA (Analysis of Variance) with F-statistic and approximate p-value.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **ANOVA One-Way** node performs a One-Way Analysis of Variance (ANOVA) to compare the means of two or more independent categorical groups (treatment conditions, product variants, algorithms, or operational methods) to determine whether there is statistically significant evidence that their population means differ.

One-Way ANOVA tests the null hypothesis (\(H_0\)) that all group means are equal against the alternative hypothesis (\(H_1\)) that at least one group mean is significantly different from the others.

### Key Features

- **Multi-Group Comparison:** Analyzes two or more numeric sample groups simultaneously.
- **F-Statistic & P-Value Computation:** Calculates between-group variance, within-group variance, degrees of freedom, F-ratio, and p-value.
- **Significance Flagging:** Automatically returns a `significant` boolean flag based on standard alpha levels (\(\alpha = 0.05\)).
- **Group-Level Summaries:** Computes descriptive statistics (sample size \(n\), sum, mean, variance, standard deviation) for each individual group.
- **Flexible Data Input:** Supports array-based configuration in UI forms or dynamic upstream JSON inputs via expressions.

### How It Works

1. **Calculate Group Means & Grand Mean:** Computes the arithmetic mean for each group (\(\bar{X}_i\)) and the overall grand mean (\(\bar{X}\)) across all sample observations.
2. **Sum of Squares Between Groups (\(SS_{\text{between}}\)):** Measures variation between group means and the grand mean:
   $$SS_{\text{between}} = \sum_{i=1}^{k} n_i (\bar{X}_i - \bar{X})^2$$
3. **Sum of Squares Within Groups (\(SS_{\text{within}}\)):** Measures variation within individual groups (error / residual variance):
   $$SS_{\text{within}} = \sum_{i=1}^{k} \sum_{j=1}^{n_i} (X_{ij} - \bar{X}_i)^2$$
4. **Mean Squares & F-Ratio:** Computes mean squares by dividing sums of squares by their respective degrees of freedom (\(df_{\text{between}} = k - 1\), \(df_{\text{within}} = N - k\)):
   $$MS_{\text{between}} = \frac{SS_{\text{between}}}{k - 1}, \quad MS_{\text{within}} = \frac{SS_{\text{within}}}{N - k}$$
   $$F = \frac{MS_{\text{between}}}{MS_{\text{within}}}$$
5. **P-Value Derivation:** Approximates the p-value from the F-distribution with degrees of freedom \((df_1, df_2)\).

### Use Cases

- **A/B/C Experimentation:** Compare user engagement or conversion rates across 3 or more design variants or marketing strategies.
- **Machine Learning Benchmark:** Determine if performance metrics (accuracy, latency, F1-score) differ significantly across multiple models or hyperparameter sets.
- **Manufacturing & Quality Control:** Evaluate defect rates or product metrics across different factory production lines or batches.
- **Healthcare & Clinical Trials:** Assess treatment efficacy across control, low-dose, and high-dose patient cohorts.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `groups` | `array` | Yes | `[]` | List of group objects to compare. Each group object contains a `name` (optional) and an array of numeric `values`. |

### Group Object Structure

Each entry in the `groups` array accepts the following properties:

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | `string` | No | `"Group N"` | Descriptive label for the group (e.g. `"Method A"`, `"Variant B"`). |
| `values` | `number[]` | Yes | `[]` | List of numeric sample observations for the group. Must contain at least 2 numbers per group. |

### Parameter Details

- **Minimum Groups:** At least **2 groups** must be supplied.
- **Minimum Observations:** Each group must contain at least **2 numeric data points**.
- **Variance Requirements:** At least one group must have non-zero variance, and within-group variance must not be zero across all groups.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data passed from preceding nodes (used to dynamically map `groups` via expressions). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when ANOVA analysis completes successfully. Contains F-statistic, p-value, sums of squares, degrees of freedom, and group summaries. |
| `error` | `Error` | Emitted if less than 2 groups are provided, groups contain insufficient data, or calculation fails. |

### Output Structure

```json
{
  "fStatistic": 74.9231,
  "pValue": 0.000000154,
  "significant": true,
  "alpha": 0.05,
  "dfBetween": 2,
  "dfWithin": 12,
  "dfTotal": 14,
  "ssBetween": 194.8,
  "ssWithin": 15.6,
  "ssTotal": 210.4,
  "msBetween": 97.4,
  "msWithin": 1.3,
  "groups": [
    {
      "name": "Method A",
      "count": 5,
      "sum": 58,
      "mean": 11.6,
      "variance": 1.3,
      "stdDev": 1.1402
    },
    {
      "name": "Method B",
      "count": 5,
      "sum": 102,
      "mean": 20.4,
      "variance": 1.3,
      "stdDev": 1.1402
    },
    {
      "name": "Method C",
      "count": 5,
      "sum": 77,
      "mean": 15.4,
      "variance": 1.3,
      "stdDev": 1.1402
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `fStatistic` | `number` | Calculated F-ratio (\(MS_{\text{between}} / MS_{\text{within}}\)). Higher values indicate larger differences relative to group noise. |
| `pValue` | `number` | Approximate p-value corresponding to the F-statistic. |
| `significant` | `boolean` | `true` if `pValue < alpha` (statistically significant difference among groups). |
| `alpha` | `number` | Significance threshold used (default `0.05`). |
| `dfBetween` | `number` | Degrees of freedom between groups (\(k - 1\)). |
| `dfWithin` | `number` | Degrees of freedom within groups (\(N - k\)). |
| `dfTotal` | `number` | Total degrees of freedom (\(N - 1\)). |
| `ssBetween` | `number` | Sum of squares between groups. |
| `ssWithin` | `number` | Sum of squares within groups (error). |
| `ssTotal` | `number` | Total sum of squares (\(SS_{\text{between}} + SS_{\text{within}}\)). |
| `msBetween` | `number` | Mean square between groups (\(SS_{\text{between}} / df_{\text{between}}\)). |
| `msWithin` | `number` | Mean square within groups (\(SS_{\text{within}} / df_{\text{within}}\)). |
| `groups` | `array` | Array of per-group statistical summaries (`name`, `count`, `sum`, `mean`, `variance`, `stdDev`). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Comparing 3 Process Methods (A/B/C Test)

Compare performance scores across three operational methods to determine if any method produces significantly higher yield.

**Configuration:**

```json
{
  "groups": [
    {
      "name": "Method A",
      "values": [10, 12, 11, 13, 12]
    },
    {
      "name": "Method B",
      "values": [20, 19, 21, 22, 20]
    },
    {
      "name": "Method C",
      "values": [15, 14, 16, 15, 17]
    }
  ]
}
```

**Output:**

```json
{
  "fStatistic": 74.9231,
  "pValue": 0.000000154,
  "significant": true,
  "alpha": 0.05,
  "dfBetween": 2,
  "dfWithin": 12,
  "dfTotal": 14,
  "ssBetween": 194.8,
  "ssWithin": 15.6,
  "ssTotal": 210.4,
  "msBetween": 97.4,
  "msWithin": 1.3,
  "groups": [
    { "name": "Method A", "count": 5, "sum": 58, "mean": 11.6, "variance": 1.3, "stdDev": 1.1402 },
    { "name": "Method B", "count": 5, "sum": 102, "mean": 20.4, "variance": 1.3, "stdDev": 1.1402 },
    { "name": "Method C", "count": 5, "sum": 77, "mean": 15.4, "variance": 1.3, "stdDev": 1.1402 }
  ]
}
```

---

### Example 2: Dynamic Upstream Data Mapping

Map group data dynamically from an upstream API payload or database query result.

**Expression Configuration:**

```json
{
  "groups": "{{ $input.experimentResults.groupData }}"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Perform One-Way ANOVA on three experimental groups
```

### Common Patterns

- **Experiment Analysis Pipeline:** HTTP Request (Fetch Experiment Metrics) → ANOVA One-Way → Function (Evaluate Significance) → Email / Slack Alert
- **Automated Quality Assurance:** Database Query → ANOVA One-Way → Log (Output F-ratio & P-value)
- **Model Evaluation:** AI Agent → Evaluate Model Scores → ANOVA One-Way → Select Best Performing Model

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Less than 2 groups provided

**Cause:** The `groups` parameter contains only 1 group or is empty.

**Solution:** Ensure at least 2 distinct sample groups are provided for comparison.

---

#### Insufficient values in group

**Cause:** One or more groups contain fewer than 2 numeric values.

**Solution:** Each group must have at least 2 sample observations to calculate variance.

---

#### Division by zero / Zero within-group variance

**Cause:** All values within every group are identical, resulting in `ssWithin = 0` and `msWithin = 0`.

**Solution:** Check the input dataset to ensure observations contain variation.

---

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Invalid groups configuration` | Parameter is missing or not an array | Pass a valid array of group objects |
| `At least 2 groups required` | Fewer than 2 groups provided | Add more groups to compare |
| `Insufficient sample size` | A group has fewer than 2 numbers | Provide at least 2 numbers per group |
| `Non-numeric data detected` | String or non-numeric values present | Clean dataset to contain numbers only |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Bootstrap](../bootstrap/en.md) — Calculate non-parametric bootstrap confidence intervals
- [Function](../function/en.md) — Pre-process group data or post-process ANOVA results
- [Log](../log/en.md) — Output ANOVA results to console logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-12 | Initial release |

<!-- /SECTION: changelog -->

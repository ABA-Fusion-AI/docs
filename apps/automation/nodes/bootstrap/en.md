---
node_id: "bootstrap"
title: "Bootstrap CI"
description: "Calculates bootstrap confidence intervals for mean or median using resampling."
category: "utilities"
subcategory: "statistics"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - statistics
  - bootstrap
  - confidence-interval
  - resampling
  - mean
  - median
related_nodes:
  - function
  - log
  - correlation
  - dispersion
---

<!-- SECTION: header -->
# Bootstrap CI

> **Category:** Utilities | **Type:** Action Node

Calculates bootstrap confidence intervals for a mean or median statistic using resampling.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Bootstrap CI** node estimates a confidence interval for a given statistic (mean or median) by repeatedly resampling the input data with replacement — a technique known as **bootstrapping**.

Unlike parametric methods, bootstrapping makes no assumptions about the underlying data distribution. It is well-suited for small samples, non-normal distributions, and robust statistical estimation.

### Key Features

- **Non-parametric:** No distributional assumptions required
- **Two statistics:** Supports `mean` and `median`
- **Configurable iterations:** Control the number of bootstrap resamples
- **Configurable alpha:** Set the significance level for the confidence interval
- **Numeric array input:** Accepts a list of numeric values directly as parameters

### How It Works

1. The node takes the input `series` of numbers.
2. It draws `iterations` random samples (with replacement) of the same size as the input.
3. For each sample, it computes the chosen statistic (`mean` or `median`).
4. It sorts the resulting distribution and extracts the lower and upper percentile bounds based on `alpha`.

### Use Cases

- Estimate confidence intervals for survey or sensor data
- Validate statistical estimates without assuming a normal distribution
- Perform robust data analysis in small-sample experiments
- Build automated data quality checks in analytics workflows
- Complement other statistical nodes (correlation, dispersion) in a pipeline

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter    | Type       | Required | Default  | Description |
|--------------|------------|----------|----------|-------------|
| `series`     | `number[]` | Yes      | —        | The numeric data series to analyze. Add each value as an item in the list. |
| `stat`       | `enum`     | Yes      | `mean`   | The statistic to estimate. Accepted values: `mean` or `median`. |
| `iterations` | `number`   | No       | `1000`   | Number of bootstrap resamples to perform. Higher values yield more stable estimates. |
| `alpha`      | `number`   | No       | `0.05`   | Significance level for the confidence interval. `0.05` produces a 95% CI; `0.01` produces a 99% CI. |

### Stat Options

| Value    | Description |
|----------|-------------|
| `mean`   | Computes the arithmetic mean of each resample. Use for normally distributed or symmetric data. |
| `median` | Computes the median of each resample. More robust to outliers and skewed distributions. |

### Alpha / Confidence Level

The `alpha` parameter controls the width of the confidence interval:

| Alpha | Confidence Level |
|-------|-----------------|
| 0.10  | 90%             |
| 0.05  | 95% (default)   |
| 0.01  | 99%             |

### Iterations

The `iterations` parameter sets how many bootstrap samples are drawn:

- **1000** (default) — suitable for most use cases
- **5000–10000** — recommended when high precision is needed
- **Lower values** — faster execution but less stable interval estimates

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input   | Type  | Description |
|---------|-------|-------------|
| `input` | `any` | Incoming workflow data passed from the preceding node. |

### Outputs

| Output    | Type     | Description |
|-----------|----------|-------------|
| `success` | `object` | The bootstrap result containing the estimated statistic and confidence interval bounds. |
| `error`   | `Error`  | Emitted when the input series is invalid or computation fails. |

### Output Structure

```json
{
  "stat": "mean",
  "observed": 12.14,
  "lower": 10.57,
  "upper": 13.71,
  "alpha": 0.05,
  "iterations": 1000,
  "confidenceLevel": 0.95
}
```

| Field             | Type     | Description |
|-------------------|----------|-------------|
| `stat`            | `string` | The statistic that was estimated (`mean` or `median`) |
| `observed`        | `number` | The statistic computed on the original input series |
| `lower`           | `number` | Lower bound of the confidence interval |
| `upper`           | `number` | Upper bound of the confidence interval |
| `alpha`           | `number` | The significance level used |
| `iterations`      | `number` | The number of bootstrap resamples performed |
| `confidenceLevel` | `number` | Expressed as `1 - alpha` (e.g., `0.95` for a 95% CI) |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: 95% Confidence Interval for the Mean

Estimate a 95% CI for the mean of a small numeric dataset.

**Configuration:**

```text
Series:     [10, 12, 11, 14, 15, 16, 7]
Stat:       mean
Iterations: 1000
Alpha:      0.05
```

**Output:**

```json
{
  "stat": "mean",
  "observed": 12.14,
  "lower": 10.14,
  "upper": 14.14,
  "alpha": 0.05,
  "iterations": 1000,
  "confidenceLevel": 0.95
}
```

---

### Example 2: 99% Confidence Interval for the Median

Use the median statistic and a stricter significance level.

**Configuration:**

```text
Series:     [10, 12, 11, 14, 15, 16, 7]
Stat:       median
Iterations: 5000
Alpha:      0.01
```

**Output:**

```json
{
  "stat": "median",
  "observed": 12,
  "lower": 10,
  "upper": 15,
  "alpha": 0.01,
  "iterations": 5000,
  "confidenceLevel": 0.99
}
```

---

### Example 3: Dynamic Series from Upstream Data

Pass a dynamic series from a previous node using an expression on the `series` parameter.

**Workflow pattern:**

```text
HTTP Request (fetch data array)
  → Bootstrap CI (series: {{ $input.data.values }}, stat: mean)
  → Function (check if 0 is within [lower, upper])
  → Log
```

---

### Example 4: Alerting on Wide Intervals

Flag results where the confidence interval is unexpectedly wide (indicating high variance or insufficient data).

**Workflow pattern:**

```text
Manual Trigger
  → Bootstrap CI (series: [...], stat: mean)
  → Function (if output.upper - output.lower > threshold → alert)
  → Email Send
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Calculate a bootstrap confidence interval for the mean
```

### Common Patterns

- **Data QA:** Fetch data → Bootstrap CI → Function (validate interval) → Alert
- **Report Generation:** Collect metrics → Bootstrap CI → Format → Email
- **Experiment Analysis:** Load results → Bootstrap CI (mean + median) → Log
- **Dashboard:** Cron Trigger → Bootstrap CI → HTTP Request (POST to dashboard API)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty or invalid series

**Cause:** The `series` parameter was left empty or contains non-numeric values.

**Solution:** Ensure every item in the `series` list is a valid number. Remove any `null`, empty string, or text values.

#### Confidence interval is very wide

**Cause:** The input series is small or has high variance. Bootstrap intervals naturally reflect data spread.

**Solution:**
- Increase `iterations` (e.g., to 5000 or 10000) for more stable bounds.
- Consider collecting more data points before running the analysis.

#### Lower and upper bounds are the same

**Cause:** All values in the series are identical, so every resample returns the same statistic.

**Solution:** Verify that the input series contains meaningful variation.

#### Slow execution

**Cause:** A very large `series` combined with a high `iterations` value can be computationally intensive.

**Solution:** Reduce `iterations` to 1000 (the default) for exploratory analysis, and increase only when high precision is required.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid series | Non-numeric or empty array | Provide a non-empty array of numbers |
| Unknown stat | Unsupported `stat` value | Use `mean` or `median` |
| Computation error | Internal resampling failure | Check that `iterations` and `alpha` are positive numbers |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Correlation](../correlation/en.md) — Compute correlation coefficients between two series
- [Dispersion](../dispersion/en.md) — Measure spread and variability of a dataset
- [Function](../function/en.md) — Post-process or validate the CI output
- [Log](../log/en.md) — Inspect the bootstrap result

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date       | Changes         |
|---------|------------|-----------------|
| 1.0.0   | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->

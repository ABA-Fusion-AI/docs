---
node_id: "ks-test"
title: "KS Test"
description: "Kolmogorov-Smirnov test: tests if data follows a normal distribution or compares two samples."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - kolmogorov-smirnov
  - ks-test
  - normality
  - two-sample
  - statistics
related_nodes:
  - normality
  - chi-square
  - correlation
---

<!-- SECTION: header -->
# KS Test

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Perform a Kolmogorov-Smirnov test against a normal distribution or compare two numeric samples.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **KS Test** node performs a Kolmogorov-Smirnov test in one of two modes:

- `normal` — compares a numeric series with a normal distribution;
- `two_sample` — compares the empirical distributions of two numeric samples.

In normal mode, the node can use an explicitly configured mean and standard deviation or estimate them directly from the input data.

In two-sample mode, the node compares the empirical cumulative distribution functions of two samples.

### Key Features

- Supports one-sample KS testing against a normal distribution.
- Supports two-sample KS testing.
- Estimates `mu` from the data when not provided.
- Estimates population `sigma` from the data when not provided.
- Calculates the maximum empirical CDF difference.
- Calculates a scaled KS statistic.
- Calculates a simplified approximate p-value.
- Returns sample sizes.
- Rounds the KS distance and statistic to six decimal places.
- Rounds the approximate p-value to nine decimal places.

### Processing Flow

```text
Select test kind
  ↓
normal or two_sample
  ↓
Validate required parameters
  ↓
Sort sample data
  ↓
Calculate empirical CDF values
  ↓
Calculate maximum CDF difference D
  ↓
Calculate scaled KS statistic
  ↓
Calculate approximate p-value
  ↓
Round results
  ↓
Return result
```

### Use Cases

- Comparing observed data with a normal distribution.
- Checking distribution similarity.
- Comparing two independent numeric samples.
- Measuring empirical distribution differences.
- Performing distribution checks before downstream statistical analysis.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `kind` | `string` | No | `normal` | KS test mode: `normal` or `two_sample`. |
| `series` | `array<number>` | Conditional | — | Numeric series for normal-distribution testing. Required when `kind` is `normal`. Must contain at least two values. |
| `mu` | `number` | No | Estimated | Normal-distribution mean. Estimated from `series` when omitted. |
| `sigma` | `number` | No | Estimated | Normal-distribution standard deviation. Estimated from `series` when omitted. Must be greater than `0` at execution time. |
| `a` | `array<number>` | Conditional | — | First sample for two-sample testing. Required when `kind` is `two_sample`. Must contain at least two values. |
| `b` | `array<number>` | Conditional | — | Second sample for two-sample testing. Required when `kind` is `two_sample`. Must contain at least two values. |

### Kind

Select the KS test mode.

Supported values:

```text
normal
two_sample
```

### Normal Mode

When:

```text
kind: normal
```

the node compares `series` with a normal distribution.

Required parameter:

```text
series
```

Optional parameters:

```text
mu
sigma
```

### Series

Provide at least two numeric values.

Example:

```text
1
2
3
4
5
```

### Mu

`mu` is the mean of the normal distribution used for comparison.

If `mu` is omitted, the node calculates:

```text
mean(series)
```

For:

```text
series: [1, 2, 3, 4, 5]
```

the estimated value is:

```text
3
```

If `mu` is explicitly provided, that value is used instead.

### Sigma

`sigma` is the standard deviation of the normal distribution used for comparison.

If `sigma` is omitted, the node calculates the population standard deviation:

```text
stddev(series, false)
```

For:

```text
series: [1, 2, 3, 4, 5]
```

the estimated value is approximately:

```text
1.4142135623730951
```

Although the schema accepts values greater than or equal to `0`, the runtime requires:

```text
sigma > 0
```

If `sigma` is `0` or lower, the node throws:

```text
sigma must be > 0
```

### Two-Sample Mode

When:

```text
kind: two_sample
```

the node compares two numeric samples:

```text
a
b
```

Both arrays must contain at least two numeric values.

Example:

```text
a: [1, 2, 3, 4, 5]
b: [1, 2, 3, 4, 5]
```

### Empirical CDF

For each comparison value, the node calculates the empirical cumulative distribution function as:

```text
count(values <= x) / sample length
```

### KS Distance

The KS distance `D` is the maximum absolute difference between the compared cumulative distribution functions.

In normal mode:

```text
D = max(|ECDF(x) - NormalCDF(x)|)
```

In two-sample mode:

```text
D = max(|ECDF_A(x) - ECDF_B(x)|)
```

### KS Statistic

In normal mode:

```text
ksStat = D × sqrt(n)
```

In two-sample mode:

```text
ksStat = D × sqrt((n1 × n2) / (n1 + n2))
```

### Approximate P-Value

The implementation calculates a simplified approximate p-value using:

```text
pValueApprox = 2 × exp(-2 × ksStat²)
```

The result is then clamped to the range:

```text
0 to 1
```

This is a simplified approximation and is not the exact Kolmogorov-Smirnov distribution.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The parameters depend on the selected `kind`.

For:

```text
kind: normal
```

the node uses:

- `series`
- `mu`
- `sigma`

For:

```text
kind: two_sample
```

the node uses:

- `a`
- `b`

Incoming workflow data is not used for the calculation.

### Normal Mode Output

Example:

```json
{
  "test": "ks_normal",
  "D": 0.16025,
  "ksStat": 0.35833,
  "pValueApprox": 1,
  "mu": 3,
  "sigma": 1.4142135623730951,
  "n": 5
}
```

The output contains:

- test identifier;
- KS distance;
- scaled KS statistic;
- approximate p-value;
- normal mean;
- normal standard deviation;
- number of observations.

### Two-Sample Output

Example:

```json
{
  "test": "ks_two_sample",
  "D": 0,
  "ksStat": 0,
  "pValueApprox": 1,
  "n1": 5,
  "n2": 5
}
```

The output contains:

- test identifier;
- KS distance;
- scaled KS statistic;
- approximate p-value;
- first sample size;
- second sample size.

### Test

Normal mode returns:

```text
ks_normal
```

Two-sample mode returns:

```text
ks_two_sample
```

### D

The `D` field contains the maximum absolute CDF difference.

It is rounded to six decimal places.

### KS Statistic

The `ksStat` field contains the scaled KS statistic.

It is rounded to six decimal places.

### Approximate P-Value

The `pValueApprox` field contains the simplified approximate p-value.

The value is clamped to:

```text
0 <= pValueApprox <= 1
```

and rounded to nine decimal places.

### Mu and Sigma

These fields are returned only in `normal` mode.

If they were not explicitly configured, the returned values are the values estimated from the data.

### N

`n` is returned in normal mode and contains the number of values in `series`.

### N1 and N2

In two-sample mode:

```text
n1 = a.length
n2 = b.length
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Normal Test With Estimated Parameters

**Configuration**

```text
kind: normal
series:
  - 1
  - 2
  - 3
  - 4
  - 5
mu: omitted
sigma: omitted
```

**Output**

```json
{
  "test": "ks_normal",
  "D": 0.16025,
  "ksStat": 0.35833,
  "pValueApprox": 1,
  "mu": 3,
  "sigma": 1.4142135623730951,
  "n": 5
}
```

### Example 2: Normal Test With Explicit Parameters

**Configuration**

```text
kind: normal
series:
  - 1
  - 2
  - 3
  - 4
  - 5
mu: 3
sigma: 1
```

**Output**

```json
{
  "test": "ks_normal",
  "D": 0.241345,
  "ksStat": 0.539663,
  "pValueApprox": 1,
  "mu": 3,
  "sigma": 1,
  "n": 5
}
```

### Example 3: Two Identical Samples

**Configuration**

```text
kind: two_sample
a:
  - 1
  - 2
  - 3
  - 4
  - 5
b:
  - 1
  - 2
  - 3
  - 4
  - 5
```

**Output**

```json
{
  "test": "ks_two_sample",
  "D": 0,
  "ksStat": 0,
  "pValueApprox": 1,
  "n1": 5,
  "n2": 5
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: KS Test Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Series Is Invalid

**Cause:** `series` contains fewer than two numeric values when `kind` is `normal`.

**Solution:** Provide at least two numeric values.

### Sample A Is Invalid

**Cause:** `a` contains fewer than two numeric values when `kind` is `two_sample`.

**Solution:** Provide at least two numeric values.

### Sample B Is Invalid

**Cause:** `b` contains fewer than two numeric values when `kind` is `two_sample`.

**Solution:** Provide at least two numeric values.

### Sigma Is Zero

The schema accepts:

```text
sigma >= 0
```

but the runtime explicitly requires:

```text
sigma > 0
```

If `sigma` is `0`, the node throws:

```text
sigma must be > 0
```

**Solution:** Use a positive standard deviation or leave `sigma` empty so the node can estimate it from the input series.

### Estimated Sigma Is Zero

If all values in `series` are identical and `sigma` is omitted, the estimated population standard deviation is `0`.

The node then throws:

```text
sigma must be > 0
```

**Solution:** Provide a non-constant series or explicitly configure a positive `sigma`.

### Unexpected P-Value

The implementation uses the simplified approximation:

```text
2 × exp(-2 × ksStat²)
```

and clamps the result between `0` and `1`.

It does not use the exact Kolmogorov-Smirnov distribution.

Therefore, results may differ from statistical software that calculates an exact or more advanced KS p-value.

### P-Value Equals One

The unbounded approximation may produce a value greater than `1`.

The node clamps the calculated result to:

```text
1
```

This is why some tested cases return:

```text
pValueApprox: 1
```

### Unexpected Normal Test Result

Verify:

- `series`;
- whether `mu` was explicitly provided;
- whether `sigma` was explicitly provided.

When omitted, both values are estimated from the input data.

### Unexpected Two-Sample Result

The node calculates the maximum difference between the two empirical cumulative distribution functions.

Identical samples produce:

```text
D: 0
ksStat: 0
pValueApprox: 1
```

### Rounding Differences

The node rounds:

```text
D
ksStat
```

to six decimal places.

The approximate p-value is rounded to nine decimal places.

`mu` and `sigma` themselves are returned without additional rounding by this node.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Normality Check (Jarque-Bera)** — Test a numeric series for normality using skewness and kurtosis.
- **Chi-Square** — Perform chi-square statistical analysis.
- **Correlation Analyzer** — Measure Pearson or Spearman correlation between numeric variables.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the KS Test node. |

<!-- /SECTION: changelog -->
---
node_id: "normality"
title: "Normality Check (Jarque-Bera)"
description: "Tests for normality using the Jarque-Bera test (based on skewness and kurtosis)."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - normality
  - jarque-bera
  - skewness
  - kurtosis
  - statistics
related_nodes:
  - dispersion
  - chi-square
  - ks-test
---

<!-- SECTION: header -->
# Normality Check (Jarque-Bera)

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Test a numeric data series for normality using the Jarque-Bera test based on skewness and kurtosis.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Normality Check (Jarque-Bera)** node evaluates whether a numeric data series is consistent with a normal distribution using the Jarque-Bera test.

The test uses the skewness and kurtosis of the input series to calculate the Jarque-Bera statistic.

The node calculates:

- skewness;
- kurtosis;
- Jarque-Bera statistic;
- degrees of freedom;
- approximate p-value;
- number of observations for non-constant series.

The input series must contain at least three numeric values.

### Key Features

- Performs the Jarque-Bera normality test.
- Calculates skewness.
- Calculates kurtosis.
- Calculates the Jarque-Bera statistic.
- Uses `2` degrees of freedom.
- Calculates an approximate right-tail chi-square p-value.
- Supports numeric arrays with at least three values.
- Handles constant series as a special case.
- Rounds skewness, kurtosis, and the Jarque-Bera statistic to six decimal places.
- Rounds the approximate p-value to nine decimal places.

### Processing Flow

```text
Numeric series
  ↓
Calculate number of observations
  ↓
Calculate mean
  ↓
Calculate population standard deviation
  ↓
Check for zero standard deviation
  ↓
Calculate standardized values
  ↓
Calculate skewness
  ↓
Calculate kurtosis
  ↓
Calculate Jarque-Bera statistic
  ↓
Calculate approximate p-value
  ↓
Round results
  ↓
Return result
```

### Use Cases

- Testing numeric data for normality.
- Measuring skewness and kurtosis.
- Checking distribution assumptions before statistical analysis.
- Evaluating datasets before applying parametric statistical methods.
- Analyzing measurement or experimental data.
- Preparing normality metrics for downstream statistical workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `series` | `array<number>` | Yes | — | Numeric data series to test for normality. Must contain at least three values. |

### Series

Provide the numeric data series to analyze.

Example:

```text
1
2
3
4
5
```

The series must contain at least three numeric values.

### Mean

The node first calculates the arithmetic mean of the input series.

For:

```text
series: [1, 2, 3, 4, 5]
```

the mean is:

```text
3
```

### Standard Deviation

The node calculates the population standard deviation of the series.

The implementation uses population statistics when calculating the standard deviation used for the standardized moments.

Each value is then standardized using:

```text
z = (value - mean) / standardDeviation
```

### Skewness

Skewness is calculated from the third standardized moment:

```text
skewness = sum(z³) / n
```

A skewness value near:

```text
0
```

indicates a relatively symmetric distribution.

Positive values indicate greater asymmetry toward larger values, while negative values indicate greater asymmetry toward smaller values.

### Kurtosis

Kurtosis is calculated from the fourth standardized moment:

```text
kurtosis = sum(z⁴) / n
```

The implementation uses raw kurtosis rather than excess kurtosis.

For a normal distribution, the reference kurtosis is:

```text
3
```

### Jarque-Bera Statistic

The Jarque-Bera statistic is calculated using:

```text
JB = (n / 6) × (skewness² + (kurtosis - 3)² / 4)
```

The statistic combines deviations in skewness and kurtosis from the values expected under normality.

### Degrees of Freedom

The Jarque-Bera test uses:

```text
df = 2
```

The returned `df` value is therefore always `2`.

### Approximate P-Value

The node calculates the approximate p-value using the right tail of a chi-square distribution:

```text
pValueApprox = chiSquarePValueRight(JB, 2)
```

A larger p-value indicates less evidence against the normality assumption according to the implemented Jarque-Bera test.

A smaller p-value indicates stronger evidence against the normality assumption.

The node returns the calculated p-value but does not automatically classify the result as normal or non-normal and does not apply a significance threshold.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses the configured value from:

- `series`

Incoming workflow data is not used for the calculation.

### Output

For a non-constant series, the node returns an object containing:

- test identifier;
- number of observations;
- skewness;
- kurtosis;
- Jarque-Bera statistic;
- degrees of freedom;
- approximate p-value.

Example:

```json
{
  "test": "jarque_bera",
  "n": 5,
  "skewness": 1.5,
  "kurtosis": 3.25,
  "JB": 1.888021,
  "df": 2,
  "pValueApprox": 0.391172318
}
```

### Test

The `test` field identifies the statistical test.

The node returns:

```text
jarque_bera
```

### N

For non-constant series, the `n` field contains the number of observations in the input series.

For:

```text
series: [1, 1, 1, 1, 10]
```

the result is:

```text
5
```

### Skewness

The `skewness` field contains the calculated third standardized moment.

The value is rounded to six decimal places.

### Kurtosis

The `kurtosis` field contains the calculated fourth standardized moment.

The value is rounded to six decimal places.

### JB

The `JB` field contains the calculated Jarque-Bera statistic.

The value is rounded to six decimal places.

### Degrees of Freedom

The `df` field contains:

```text
2
```

### Approximate P-Value

The `pValueApprox` field contains the approximate right-tail chi-square p-value.

The value is rounded to nine decimal places using:

```text
Math.round(pValueApprox × 1000000000) / 1000000000
```

### Rounding

The node rounds `skewness`, `kurtosis`, and `JB` to six decimal places:

```text
Math.round(value × 1000000) / 1000000
```

The approximate p-value is rounded to nine decimal places.

### Constant Series Output

When the population standard deviation is exactly `0`, the node returns a special result:

```json
{
  "test": "jarque_bera",
  "JB": 0,
  "df": 2,
  "pValueApprox": 1,
  "skewness": 0,
  "kurtosis": 0
}
```

In this special branch, the current implementation does not include the `n` field.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Symmetric Series

**Configuration**

```text
series:
  - 1
  - 2
  - 3
  - 4
  - 5
```

**Output**

```json
{
  "test": "jarque_bera",
  "n": 5,
  "skewness": 0,
  "kurtosis": 1.7,
  "JB": 0.352083,
  "df": 2,
  "pValueApprox": 0.837764185
}
```

### Example 2: Asymmetric Series

**Configuration**

```text
series:
  - 1
  - 1
  - 1
  - 1
  - 10
```

**Output**

```json
{
  "test": "jarque_bera",
  "n": 5,
  "skewness": 1.5,
  "kurtosis": 3.25,
  "JB": 1.888021,
  "df": 2,
  "pValueApprox": 0.391172318
}
```

### Example 3: Constant Series

**Configuration**

```text
series:
  - 10
  - 10
  - 10
  - 10
  - 10
```

**Output**

```json
{
  "test": "jarque_bera",
  "JB": 0,
  "df": 2,
  "pValueApprox": 1,
  "skewness": 0,
  "kurtosis": 0
}
```

Because every value is identical, the population standard deviation is `0`.

The node handles this condition directly instead of calculating standardized moments.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Normality Check Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Series Is Invalid

**Cause:** `series` contains fewer than three values or contains invalid values.

**Solution:** Provide an array containing at least three numeric values.

### Constant Series

When every value in the series is identical, the population standard deviation is:

```text
0
```

The node handles this condition explicitly and returns:

```json
{
  "test": "jarque_bera",
  "JB": 0,
  "df": 2,
  "pValueApprox": 1,
  "skewness": 0,
  "kurtosis": 0
}
```

The current implementation does not include `n` in this special output.

### Unexpected Skewness

Skewness is calculated using the third standardized moment:

```text
sum(z³) / n
```

Verify that the input `series` contains the intended numeric values.

### Unexpected Kurtosis

The implementation returns raw kurtosis.

A normal distribution has a reference kurtosis of:

```text
3
```

The returned value is not excess kurtosis, so a normal distribution is not expected to have a kurtosis value of `0`.

### Unexpected Jarque-Bera Statistic

The Jarque-Bera statistic depends on:

- the number of observations;
- skewness;
- kurtosis.

The implementation calculates:

```text
JB = (n / 6) × (skewness² + (kurtosis - 3)² / 4)
```

Verify the input series and the calculated skewness and kurtosis when investigating unexpected results.

### Unexpected P-Value

The `pValueApprox` field is calculated from the right tail of a chi-square distribution with:

```text
df = 2
```

The node reports the approximate p-value directly.

It does not apply a predefined significance level such as `0.05` and does not return an automatic normal or non-normal decision.

### Small Sample Size

The schema allows the Jarque-Bera calculation when the series contains at least three values.

Statistical interpretation of a normality test can depend on sample size, so the returned result should be interpreted according to the requirements of the workflow using it.

### Missing N Field

If the result does not contain `n`, verify whether the input series is constant.

The current implementation returns early when the population standard deviation is `0`, before adding the `n` field to the output.

### Rounding Differences

The node rounds:

```text
skewness
kurtosis
JB
```

to six decimal places.

It rounds:

```text
pValueApprox
```

to nine decimal places.

Small differences from external statistical tools may therefore appear beyond these decimal places.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Std Dev & Variance** — Calculate variance and standard deviation for numeric data.
- **Chi-Square** — Perform chi-square statistical analysis.
- **Kolmogorov-Smirnov Test** — Perform distribution comparison using the Kolmogorov-Smirnov test.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Normality Check (Jarque-Bera) node. |

<!-- /SECTION: changelog -->
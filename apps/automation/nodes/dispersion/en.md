---
node_id: "dispersion"
title: "Std Dev & Variance"
description: "Calculates standard deviation and variance for a series of numbers (sample or population)."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - standard-deviation
  - variance
  - dispersion
  - statistics
  - sample
related_nodes:
  - moving-average
  - correlation
  - regression
---

<!-- SECTION: header -->
# Std Dev & Variance

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate mean, variance, and standard deviation for a numeric series using sample or population statistics.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Std Dev & Variance** node calculates the mean, variance, and standard deviation of a numeric series.

The node supports both:

- sample statistics, using `n - 1`;
- population statistics, using `n`.

The selected mode is controlled by the `sample` parameter.

### Key Features

- Calculates the arithmetic mean.
- Calculates variance.
- Calculates standard deviation.
- Supports sample statistics.
- Supports population statistics.
- Returns the number of input values.
- Rounds calculated values to six decimal places.
- Accepts numeric arrays as input.

### Processing Flow

```text
Numeric series
  ↓
Sample or population mode
  ↓
Calculate mean
  ↓
Calculate variance
  ↓
Calculate standard deviation
  ↓
Round values to six decimals
  ↓
Return result
```

### Use Cases

- Measuring variability in numeric data.
- Comparing sample and population dispersion.
- Calculating statistical spread.
- Analyzing measurement or sensor data.
- Preparing descriptive statistics for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `series` | `array<number>` | Yes | — | Numeric values to analyze. Must contain at least one value. |
| `sample` | `boolean` | No | `true` | Use sample statistics when `true`, or population statistics when `false`. |

### Series

Provide the numeric values to analyze.

Example:

```text
10
20
30
40
50
```

The series must contain at least one numeric value.

### Sample

Enable or disable sample statistics.

When:

```text
sample: true
```

the node uses sample variance and standard deviation based on `n - 1`.

When:

```text
sample: false
```

the node uses population variance and standard deviation based on `n`.

### Mean

The arithmetic mean is calculated from the input series.

For:

```text
series: [10, 20, 30, 40, 50]
```

the mean is:

```text
30
```

### Sample Variance

For:

```text
series: [10, 20, 30, 40, 50]
sample: true
```

the sample variance is:

```text
250
```

### Sample Standard Deviation

The sample standard deviation is the square root of the sample variance.

For:

```text
variance: 250
```

the result is:

```text
15.811388
```

### Population Variance

For:

```text
series: [10, 20, 30, 40, 50]
sample: false
```

the population variance is:

```text
200
```

### Population Standard Deviation

For:

```text
variance: 200
```

the result is:

```text
14.142136
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `series`
- `sample`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- mean;
- variance;
- standard deviation;
- selected sample mode;
- number of input values.

Example:

```json
{
  "mean": 30,
  "variance": 200,
  "stddev": 14.142136,
  "sample": false,
  "n": 5
}
```

### Mean

The `mean` field contains the arithmetic mean of the input series.

### Variance

The `variance` field contains either sample variance or population variance depending on `sample`.

### Standard Deviation

The `stddev` field contains the standard deviation corresponding to the selected variance mode.

### Sample

The `sample` field indicates which mode was used:

```text
true  → sample statistics
false → population statistics
```

### N

The `n` field contains the number of values in the input series.

For:

```text
series: [10, 20, 30, 40, 50]
```

the result is:

```text
5
```

### Rounding

The node rounds `mean`, `variance`, and `stddev` to six decimal places.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Sample Statistics

**Configuration**

```text
series:
  - 10
  - 20
  - 30
  - 40
  - 50
sample: true
```

**Output**

```json
{
  "mean": 30,
  "variance": 250,
  "stddev": 15.811388,
  "sample": true,
  "n": 5
}
```

### Example 2: Population Statistics

**Configuration**

```text
series:
  - 10
  - 20
  - 30
  - 40
  - 50
sample: false
```

**Output**

```json
{
  "mean": 30,
  "variance": 200,
  "stddev": 14.142136,
  "sample": false,
  "n": 5
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Dispersion Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Series Is Invalid

**Cause:** `series` is empty or contains invalid values.

**Solution:** Provide an array containing at least one number.

### Unexpected Variance

**Cause:** The `sample` mode may not match the intended statistical method.

**Solution:** Use:

```text
sample: true
```

for sample statistics, or:

```text
sample: false
```

for population statistics.

### Unexpected Standard Deviation

The standard deviation is calculated from the variance using the selected sample or population mode.

Verify both `series` and `sample`.

### Single Value With Sample Statistics

The schema allows a series containing a single numeric value.

Sample statistics normally require more than one observation because the variance calculation uses `n - 1`.

If `sample` is `true`, use at least two values when meaningful sample variance and standard deviation are required.

### Rounding Differences

The node rounds calculated values to six decimal places.

Small differences from external statistical tools may therefore appear beyond six decimal places.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Moving Average** — Calculate SMA, EMA, and WMA values.
- **Correlation** — Measure statistical relationships between variables.
- **Regression** — Perform regression analysis.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Std Dev & Variance node. |

<!-- /SECTION: changelog -->
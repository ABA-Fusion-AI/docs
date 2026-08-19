---
node_id: "correlation"
title: "Correlation Analyzer"
description: "Calculates Pearson (linear) or Spearman (rank-based) correlation coefficient between two variables."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - correlation
  - pearson
  - spearman
  - statistics
  - analysis
related_nodes:
  - moving-average
  - dispersion
  - regression
---

<!-- SECTION: header -->
# Correlation Analyzer

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate Pearson or Spearman correlation coefficients between two numeric variables.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Correlation Analyzer** node measures the relationship between two numeric variables using either Pearson or Spearman correlation.

Pearson correlation measures linear association between two numeric series.

Spearman correlation measures rank-based association between two numeric series.

Both input arrays must contain at least two numeric values and must have the same length.

### Key Features

- Calculates Pearson correlation.
- Calculates Spearman correlation.
- Supports numeric arrays for both variables.
- Validates that both arrays have the same length.
- Requires at least two values in each array.
- Rounds the correlation coefficient to six decimal places.
- Returns the selected correlation type and number of observations.

### Processing Flow

```text
Correlation type
  ↓
X values
  ↓
Y values
  ↓
Validate array lengths
  ↓
Select Pearson or Spearman
  ↓
Calculate correlation coefficient
  ↓
Round result to six decimals
  ↓
Return result
```

### Use Cases

- Measuring linear relationships between variables.
- Measuring rank-based relationships.
- Comparing two numeric series.
- Exploring relationships in statistical datasets.
- Preparing correlation metrics for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `type` | `string` | No | `pearson` | Correlation type: `pearson` or `spearman`. |
| `x` | `array<number>` | Yes | — | First numeric variable. Must contain at least two values. |
| `y` | `array<number>` | Yes | — | Second numeric variable. Must contain at least two values and have the same length as `x`. |

### Type

Select the correlation method.

Supported values:

```text
pearson
spearman
```

### X

Provide the first numeric variable.

Example:

```text
1
2
3
4
5
```

The array must contain at least two numeric values.

### Y

Provide the second numeric variable.

Example:

```text
10
20
30
40
50
```

The array must contain at least two numeric values.

The number of values in `y` must match the number of values in `x`.

### Pearson Correlation

Pearson correlation measures linear association between the two variables.

For:

```text
x: [10, 20, 30, 40, 50]
y: [20, 40, 60, 80, 100]
```

the result is:

```text
1
```

This represents a perfect positive linear correlation.

For:

```text
x: [10, 20, 30, 40, 50]
y: [50, 40, 30, 20, 10]
```

the result is:

```text
-1
```

This represents a perfect negative linear correlation.

### Spearman Correlation

Spearman correlation measures rank-based association between the variables.

For:

```text
x: [1, 2, 3, 4, 5]
y: [10, 20, 30, 40, 50]
```

the result is:

```text
1
```

This represents a perfect positive rank correlation.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `type`
- `x`
- `y`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- selected correlation type;
- correlation coefficient;
- number of observations.

Example:

```json
{
  "type": "spearman",
  "value": 1,
  "n": 5
}
```

### Type

The `type` field contains the selected correlation method:

```text
pearson
```

or:

```text
spearman
```

### Value

The `value` field contains the calculated correlation coefficient.

The node rounds the result to six decimal places using:

```text
Math.round(value × 1000000) / 1000000
```

### N

The `n` field contains the number of observations.

For:

```text
x: [1, 2, 3, 4, 5]
```

the result is:

```text
5
```

Because the node requires equal array lengths, `n` also represents the number of values in `y`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Positive Pearson Correlation

**Configuration**

```text
type: pearson
x:
  - 10
  - 20
  - 30
  - 40
  - 50
y:
  - 20
  - 40
  - 60
  - 80
  - 100
```

**Output**

```json
{
  "type": "pearson",
  "value": 1,
  "n": 5
}
```

### Example 2: Negative Pearson Correlation

**Configuration**

```text
type: pearson
x:
  - 10
  - 20
  - 30
  - 40
  - 50
y:
  - 50
  - 40
  - 30
  - 20
  - 10
```

**Output**

```json
{
  "type": "pearson",
  "value": -1,
  "n": 5
}
```

### Example 3: Spearman Correlation

**Configuration**

```text
type: spearman
x:
  - 1
  - 2
  - 3
  - 4
  - 5
y:
  - 10
  - 20
  - 30
  - 40
  - 50
```

**Output**

```json
{
  "type": "spearman",
  "value": 1,
  "n": 5
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Correlation Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### X Is Invalid

**Cause:** `x` contains fewer than two values or contains invalid values.

**Solution:** Provide an array containing at least two numbers.

### Y Is Invalid

**Cause:** `y` contains fewer than two values or contains invalid values.

**Solution:** Provide an array containing at least two numbers.

### X and Y Have Different Lengths

**Cause:** The number of values in `x` does not match the number of values in `y`.

The node throws:

```text
x and y must have the same length
```

**Solution:** Use arrays with the same number of values.

### Unexpected Pearson Result

Pearson correlation measures linear association.

A value close to:

```text
1
```

indicates a strong positive linear relationship.

A value close to:

```text
-1
```

indicates a strong negative linear relationship.

A value near:

```text
0
```

indicates little or no linear relationship.

### Unexpected Spearman Result

Spearman correlation is rank-based rather than directly based on the numeric distances between values.

Use `spearman` when the rank relationship between the two variables is the intended measure.

### Rounding Differences

The node rounds the calculated correlation coefficient to six decimal places.

Small differences from external statistical tools may therefore appear beyond six decimal places.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Moving Average** — Calculate SMA, EMA, and WMA values.
- **Dispersion** — Calculate variance and standard deviation.
- **Regression** — Perform regression analysis.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Correlation Analyzer node. |

<!-- /SECTION: changelog -->
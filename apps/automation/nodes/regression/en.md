---
node_id: "regression"
title: "Regression"
description: "Performs linear or polynomial regression and calculates R² (coefficient of determination)."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - regression
  - linear-regression
  - polynomial-regression
  - r-squared
  - statistics
related_nodes:
  - correlation
  - dispersion
  - probability
---

<!-- SECTION: header -->
# Regression

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Perform linear or polynomial regression and calculate the coefficient of determination.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Regression** node performs polynomial regression on two numeric variables.

A degree of `1` performs linear regression, while larger degree values perform polynomial regression.

The node calculates:

- regression coefficients;
- predicted values;
- R² coefficient of determination;
- number of observations.

The input arrays `x` and `y` must contain at least two numeric values and must have the same length.

### Key Features

- Performs linear regression.
- Performs polynomial regression.
- Supports configurable polynomial degree.
- Calculates regression coefficients.
- Calculates predicted values.
- Calculates R².
- Validates that `x` and `y` have the same length.
- Rounds returned numeric results to six decimal places.

### Processing Flow

```text
X values
  ↓
Y values
  ↓
Polynomial degree
  ↓
Validate array lengths
  ↓
Build polynomial regression system
  ↓
Solve coefficients
  ↓
Calculate predictions
  ↓
Calculate R²
  ↓
Round results
  ↓
Return result
```

### Use Cases

- Fitting linear trends to numeric data.
- Fitting quadratic or higher-degree polynomial curves.
- Measuring regression goodness of fit.
- Generating fitted predictions.
- Exploring relationships between dependent and independent variables.
- Preparing regression metrics for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `x` | `array<number>` | Yes | — | Independent variable values. Must contain at least two numbers. |
| `y` | `array<number>` | Yes | — | Dependent variable values. Must contain at least two numbers and have the same length as `x`. |
| `degree` | `number` | No | `1` | Polynomial degree. Must be greater than or equal to `1`. |

### X

Provide the independent variable values.

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

Provide the dependent variable values.

Example:

```text
2
4
6
8
10
```

The array must contain at least two numeric values and must have the same length as `x`.

### Degree

Provide the polynomial degree.

Example:

```text
1
```

A degree of:

```text
1
```

performs linear regression.

A degree of:

```text
2
```

performs quadratic regression.

The current implementation applies:

```text
Math.floor(degree)
```

before performing the regression.

Therefore, a decimal value such as:

```text
2.8
```

is processed using an effective degree of:

```text
2
```

### Regression Coefficients

The returned coefficient array is ordered from the constant term upward.

For example:

```text
coeffs: [a0, a1, a2]
```

represents:

```text
y = a0 + a1 × x + a2 × x²
```

For linear regression:

```text
coeffs: [0, 2]
```

represents:

```text
y = 0 + 2x
```

For quadratic regression:

```text
coeffs: [0, 0, 1]
```

represents:

```text
y = x²
```

### R²

The node calculates the coefficient of determination using:

```text
R² = 1 - SSres / SStot
```

where:

```text
SSres = residual sum of squares
SStot = total sum of squares
```

A value close to:

```text
1
```

indicates that the fitted regression explains a large proportion of the observed variation.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `x`
- `y`
- `degree`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- effective polynomial degree;
- regression coefficients;
- R²;
- predicted values;
- number of observations.

Example:

```json
{
  "degree": 1,
  "coeffs": [
    -0.2,
    2.2
  ],
  "rSquared": 0.945313,
  "predictions": [
    2,
    4.2,
    6.4,
    8.6,
    10.8
  ],
  "n": 5
}
```

### Degree

The `degree` field contains the effective integer polynomial degree used by the calculation.

The node returns:

```text
Math.floor(degree)
```

### Coefficients

The `coeffs` field contains the fitted polynomial coefficients.

Each coefficient is rounded to six decimal places.

### R Squared

The `rSquared` field contains the calculated coefficient of determination.

The returned value is rounded to six decimal places.

### Predictions

The `predictions` field contains the fitted value for each input `x`.

Each prediction is rounded to six decimal places.

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

Because `x` and `y` must have equal lengths, this is also the number of values in `y`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Perfect Linear Regression

**Configuration**

```text
x:
  - 1
  - 2
  - 3
  - 4
  - 5
y:
  - 2
  - 4
  - 6
  - 8
  - 10
degree: 1
```

**Output**

```json
{
  "degree": 1,
  "coeffs": [
    0,
    2
  ],
  "rSquared": 1,
  "predictions": [
    2,
    4,
    6,
    8,
    10
  ],
  "n": 5
}
```

### Example 2: Quadratic Regression

**Configuration**

```text
x:
  - 1
  - 2
  - 3
  - 4
  - 5
y:
  - 1
  - 4
  - 9
  - 16
  - 25
degree: 2
```

**Output**

```json
{
  "degree": 2,
  "coeffs": [
    0,
    0,
    1
  ],
  "rSquared": 1,
  "predictions": [
    1,
    4,
    9,
    16,
    25
  ],
  "n": 5
}
```

### Example 3: Non-Perfect Linear Regression

**Configuration**

```text
x:
  - 1
  - 2
  - 3
  - 4
  - 5
y:
  - 2
  - 5
  - 5
  - 9
  - 11
degree: 1
```

**Output**

```json
{
  "degree": 1,
  "coeffs": [
    -0.2,
    2.2
  ],
  "rSquared": 0.945313,
  "predictions": [
    2,
    4.2,
    6.4,
    8.6,
    10.8
  ],
  "n": 5
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Regression Example
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

**Cause:** `x` and `y` contain different numbers of values.

The node throws:

```text
x and y must have the same length
```

**Solution:** Provide arrays with the same number of values.

### Degree Is Invalid

**Cause:** `degree` is below `1`.

**Solution:** Use a degree greater than or equal to `1`.

### Decimal Degree

The `degree` parameter accepts numeric values, including decimals.

The node applies:

```text
Math.floor(degree)
```

before fitting the polynomial.

For example:

```text
degree: 2.8
```

is processed as:

```text
degree: 2
```

### Singular Matrix

Some combinations of input values and polynomial degree may produce a singular regression system.

The node throws:

```text
Singular matrix (regression impossible)
```

This can occur when the regression system cannot be uniquely solved, for example when the input data does not provide enough independent information for the requested polynomial degree.

**Solution:** Use more suitable `x` values, reduce the polynomial degree, or provide additional distinct observations.

### Unexpected R²

R² measures how closely the fitted predictions match the observed `y` values.

A perfect fit returns:

```text
1
```

Values closer to `1` indicate a better fit according to the implemented regression model. Lower values indicate that the fitted predictions explain less of the observed variation.

### Constant Y Values

When all `y` values are identical, the implementation returns:

```text
rSquared: 1
```

because the total sum of squares is zero.

### Rounding Differences

Coefficients, predictions, and R² are rounded to six decimal places.

Small differences from external statistical software may appear beyond six decimal places.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Correlation Analyzer** — Measure Pearson or Spearman correlation.
- **Std Dev & Variance** — Calculate variance and standard deviation.
- **Probability** — Perform probability-related calculations.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Regression node. |

<!-- /SECTION: changelog -->
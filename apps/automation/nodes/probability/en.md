---
node_id: "probability"
title: "Probability"
description: "Calculates probability mass/density functions (PMF/PDF) and cumulative distribution functions (CDF) for Binomial, Poisson, and Normal distributions."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - probability
  - binomial
  - poisson
  - normal-distribution
  - pmf
  - pdf
  - cdf
related_nodes:
  - regression
  - normality
  - quantiles
---

<!-- SECTION: header -->
# Probability

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate PMF/PDF and CDF values for Binomial, Poisson, and Normal probability distributions.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Probability** node calculates probability values for three supported distributions:

- Binomial;
- Poisson;
- Normal.

For discrete distributions, the node returns a probability mass function (`pmf`) and cumulative distribution function (`cdf`).

For the Normal distribution, it returns a probability density function (`pdf`) and cumulative distribution function (`cdf`).

The returned numeric values are rounded to nine decimal places.

### Key Features

- Supports Binomial distribution calculations.
- Supports Poisson distribution calculations.
- Supports Normal distribution calculations.
- Calculates Binomial PMF and CDF.
- Calculates Poisson PMF and CDF.
- Calculates Normal PDF and CDF.
- Uses distribution-specific parameters.
- Validates the Binomial success probability `p` between `0` and `1`.
- Applies default Normal parameters for `mu` and `sigma`.
- Rounds returned probability values to nine decimal places.

### Processing Flow

```text
Select distribution
  ↓
Load distribution parameters
  ↓
Validate parameters
  ↓
Distribution type?
  ├─ Binomial → Calculate PMF + CDF
  ├─ Poisson  → Calculate PMF + CDF
  └─ Normal   → Calculate PDF + CDF
                    ↓
              Round numeric values
                    ↓
                Return result
```

### Use Cases

- Calculating probabilities for repeated independent trials.
- Modeling event counts using a Poisson distribution.
- Evaluating values under a Normal distribution.
- Calculating cumulative probabilities.
- Preparing probability metrics for statistical workflows.
- Comparing values across common probability distributions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

The available parameters depend on the selected distribution.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `dist` | `string` | Yes | `binomial` | Probability distribution: `binomial`, `poisson`, or `normal`. |
| `k` | `number` | Binomial | — | Number of successes. Must be greater than or equal to `0`. |
| `n` | `number` | Binomial | — | Number of trials. Must be greater than or equal to `0`. |
| `p` | `number` | Binomial | — | Probability of success. Must be between `0` and `1`. |
| `kPoisson` | `number` | Poisson | — | Number of observed events. Must be greater than or equal to `0`. |
| `lambda` | `number` | Poisson | — | Poisson rate parameter. Must be greater than or equal to `0`. |
| `x` | `number` | Normal | — | Value at which the PDF and CDF are evaluated. |
| `mu` | `number` | No | `0` | Mean of the Normal distribution. |
| `sigma` | `number` | No | `1` | Standard deviation of the Normal distribution. |

### Distribution

Supported values:

```text
binomial
poisson
normal
```

The selected distribution controls which parameters are displayed and used.

### Binomial Distribution

For:

```text
dist: binomial
```

provide:

- `k`
- `n`
- `p`

Example:

```text
k: 2
n: 5
p: 0.5
```

The node calculates:

```text
pmf
cdf
```

For this example, the output contains:

```text
pmf: 0.3125
cdf: 0.5
```

### K

`k` represents the number of successes.

It must be greater than or equal to:

```text
0
```

### N

`n` represents the number of trials.

It must be greater than or equal to:

```text
0
```

### P

`p` represents the probability of success for each trial.

The validator requires:

```text
0 <= p <= 1
```

For example:

```text
p: 1.5
```

is rejected during validation.

### Poisson Distribution

For:

```text
dist: poisson
```

provide:

```text
kPoisson
lambda
```

Example:

```text
kPoisson: 3
lambda: 2
```

The node returns approximately:

```text
pmf: 0.180447044
cdf: 0.85712346
```

### K Poisson

`kPoisson` represents the number of observed events.

It must be greater than or equal to:

```text
0
```

The returned parameter is named:

```text
k
```

in the output object.

### Lambda

`lambda` is the rate parameter of the Poisson distribution.

It must be greater than or equal to:

```text
0
```

### Normal Distribution

For:

```text
dist: normal
```

provide:

```text
x
mu
sigma
```

`mu` and `sigma` are optional and default to:

```text
mu: 0
sigma: 1
```

Example:

```text
x: 0
mu: 0
sigma: 1
```

returns approximately:

```text
pdf: 0.39894228
cdf: 0.500000001
```

### X

`x` is the numeric value where the Normal probability density and cumulative probability are evaluated.

### Mu

`mu` is the mean of the Normal distribution.

Default:

```text
0
```

### Sigma

`sigma` is the standard deviation of the Normal distribution.

Default:

```text
1
```

The schema accepts numeric values greater than or equal to `0`, but the probability calculation requires a strictly positive standard deviation.

Therefore:

```text
sigma: 0
```

results in:

```text
sigma must be > 0
```

Use:

```text
sigma > 0
```

for Normal distribution calculations.

### Numeric Rounding

The node rounds returned PMF, PDF, and CDF values to nine decimal places using:

```text
Math.round(value * 1000000000) / 1000000000
```

Small approximation differences can therefore occur in calculated distribution values.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured parameters only.

The parameters used depend on `dist`.

For Binomial:

- `k`
- `n`
- `p`

For Poisson:

- `kPoisson`
- `lambda`

For Normal:

- `x`
- `mu`
- `sigma`

Incoming workflow data is not used for the probability calculation.

### Binomial Output

Example:

```json
{
  "dist": "binomial",
  "pmf": 0.3125,
  "cdf": 0.5,
  "parameters": {
    "k": 2,
    "n": 5,
    "p": 0.5
  }
}
```

### Poisson Output

Example:

```json
{
  "dist": "poisson",
  "pmf": 0.180447044,
  "cdf": 0.85712346,
  "parameters": {
    "k": 3,
    "lambda": 2
  }
}
```

### Normal Output

Example:

```json
{
  "dist": "normal",
  "pdf": 0.39894228,
  "cdf": 0.500000001,
  "parameters": {
    "x": 0,
    "mu": 0,
    "sigma": 1
  }
}
```

### Dist

The `dist` field contains the selected probability distribution.

Possible values:

```text
binomial
poisson
normal
```

### PMF

The `pmf` field is returned for:

- Binomial;
- Poisson.

It represents the probability mass at the configured discrete value.

### PDF

The `pdf` field is returned for the Normal distribution.

It represents the probability density at `x`.

### CDF

The `cdf` field contains the cumulative probability calculated by the selected distribution helper.

### Parameters

The `parameters` object contains the values used in the calculation.

For Poisson, the configured `kPoisson` parameter is returned as:

```text
k
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Binomial Distribution

**Configuration**

```text
dist: binomial
k: 2
n: 5
p: 0.5
```

**Output**

```json
{
  "dist": "binomial",
  "pmf": 0.3125,
  "cdf": 0.5,
  "parameters": {
    "k": 2,
    "n": 5,
    "p": 0.5
  }
}
```

### Example 2: Poisson Distribution

**Configuration**

```text
dist: poisson
kPoisson: 3
lambda: 2
```

**Output**

```json
{
  "dist": "poisson",
  "pmf": 0.180447044,
  "cdf": 0.85712346,
  "parameters": {
    "k": 3,
    "lambda": 2
  }
}
```

### Example 3: Standard Normal Distribution

**Configuration**

```text
dist: normal
x: 0
mu: 0
sigma: 1
```

**Output**

```json
{
  "dist": "normal",
  "pdf": 0.39894228,
  "cdf": 0.500000001,
  "parameters": {
    "x": 0,
    "mu": 0,
    "sigma": 1
  }
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Probability Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Probability Is Outside the Valid Range

For a Binomial distribution, `p` must be between:

```text
0
```

and:

```text
1
```

For example:

```text
p: 1.5
```

results in a validation error similar to:

```text
p: Number must be less than or equal to 1
```

**Solution:** Provide a value satisfying:

```text
0 <= p <= 1
```

### Sigma Is Zero

For a Normal distribution:

```text
sigma: 0
```

passes the schema minimum but is rejected by the calculation.

The node throws:

```text
sigma must be > 0
```

**Solution:** Use a strictly positive standard deviation.

### Unexpected Normal CDF Precision

For the standard Normal distribution at:

```text
x: 0
mu: 0
sigma: 1
```

the implementation can return:

```text
cdf: 0.500000001
```

instead of exactly:

```text
0.5
```

This is a small numerical approximation difference and is expected from the implemented Normal CDF helper.

### Missing Distribution Parameters

Each distribution requires its own parameters.

For Binomial, provide:

```text
k
n
p
```

For Poisson, provide:

```text
kPoisson
lambda
```

For Normal, provide:

```text
x
```

with optional:

```text
mu
sigma
```

### Unknown Distribution

The schema restricts `dist` to the supported distribution values.

The implementation also contains a fallback error:

```text
Unknown distribution: <dist>
```

if an unsupported distribution reaches the calculation.

### Rounding Differences

PMF, PDF, and CDF values are rounded to nine decimal places.

Small differences from external statistical software can appear beyond the returned precision.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Regression** — Perform linear or polynomial regression.
- **Normality Check (Jarque-Bera)** — Test a numeric series for normality.
- **Quantiles** — Calculate distribution quantiles and percentiles.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation for the Probability node. |

<!-- /SECTION: changelog -->
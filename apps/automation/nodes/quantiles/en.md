---
node_id: "quantiles"
title: "Quantiles / Percentiles / IQR"
description: "Calculates quantiles, percentiles, and IQR (Interquartile Range) for a data series."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - quantiles
  - percentiles
  - iqr
  - median
  - quartiles
  - statistics
related_nodes:
  - probability
  - dispersion
  - outlier-detection
---

<!-- SECTION: header -->
# Quantiles / Percentiles / IQR

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate quantiles, percentiles, quartiles, median, and interquartile range (IQR) for a numerical data series.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Quantiles / Percentiles / IQR** node calculates quantile-based statistical measures for a numerical data series.

The node automatically calculates the first quartile (Q1), median, third quartile (Q3), and interquartile range (IQR).

It can also calculate additional quantiles through the `qs` parameter.

The input series does not need to be sorted. The node creates a sorted copy internally before calculating Q1, median, Q3, and IQR.

### Key Features

- Calculates Q1, median, and Q3.
- Calculates the interquartile range (IQR).
- Supports custom quantile positions through `qs` when the workflow editor serializes the field as a numeric array.
- Accepts unsorted numerical data.
- Sorts a copy of the series before quartile calculations.
- Supports interpolation when a quantile falls between data points.
- Returns the number of observations.
- Validates that the input series contains at least one value.
- Validates configured quantile positions between `0` and `1`.
- Rounds calculated values to six decimal places.

### Processing Flow

```text
Receive numerical series
        ↓
Validate series and qs
        ↓
Calculate requested quantiles
        ↓
Sort a copy of the series
        ↓
Calculate Q1
        ↓
Calculate median
        ↓
Calculate Q3
        ↓
IQR = Q3 - Q1
        ↓
Round calculated values
        ↓
Return statistics
```

### Use Cases

- Calculating quartiles for numerical datasets.
- Finding the median of a data series.
- Measuring spread using IQR.
- Preparing data for outlier detection.
- Calculating percentile-based statistics.
- Summarizing distributions in statistical workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `series` | `number[]` | Yes | — | Numerical data series to analyze. Must contain at least one value. |
| `qs` | `number[]` | No | `[0.25, 0.5, 0.75]` | Quantile positions to calculate. Each value must be between `0` and `1`. |

### Series

`series` contains the numerical values to analyze.

Example:

```json
[1, 2, 3, 4, 6, 8]
```

The schema requires at least one item:

```text
series.length >= 1
```

The series does not need to be sorted before being passed to the node.

For quartile calculations, the implementation creates a sorted copy using ascending numeric order.

For example:

```json
[5, 1, 4, 2, 3]
```

is internally treated as:

```json
[1, 2, 3, 4, 5]
```

for Q1, median, and Q3 calculations.

### Qs

`qs` defines additional quantile positions to calculate.

The schema default is:

```json
[0.25, 0.5, 0.75]
```

Each value must satisfy:

```text
0 <= q <= 1
```

Common values include:

| Value | Meaning |
|-------|---------|
| `0` | Minimum / 0th percentile |
| `0.25` | First quartile / 25th percentile |
| `0.5` | Median / 50th percentile |
| `0.75` | Third quartile / 75th percentile |
| `1` | Maximum / 100th percentile |

When `qs` is successfully provided as a numeric array, the requested quantiles are returned through the `values` object.

### Empty Qs

If the workflow editor explicitly provides:

```json
[]
```

then `qs` is an existing empty array.

In that case, the schema default is not applied and the node returns:

```json
{
  "qs": [],
  "values": {}
}
```

Q1, median, Q3, and IQR are still calculated independently.

### Default Qs Behavior

The schema defines:

```json
[0.25, 0.5, 0.75]
```

as the default for `qs`.

This default applies when `qs` is absent and the schema resolves the optional field to its default value.

An explicitly supplied empty array is different from an omitted value.

### Numerical Precision

The node rounds calculated values to six decimal places using:

```text
Math.round(value * 1000000) / 1000000
```

This rounding is applied to:

- requested quantile values;
- Q1;
- median;
- Q3;
- IQR.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured parameters:

- `series`
- `qs`

The calculation uses the configured `series` and `qs` parameters.

### Output

The node returns:

```json
{
  "qs": [],
  "values": {},
  "q1": 2.25,
  "median": 3.5,
  "q3": 5.5,
  "iqr": 3.25,
  "n": 6
}
```

### Qs

`qs` contains the quantile positions used for the `values` calculation.

### Values

`values` contains the calculated values for the quantile positions supplied through `qs`.

When:

```json
"qs": []
```

the result is:

```json
"values": {}
```

### Q1

`q1` contains the first quartile calculated at:

```text
q = 0.25
```

### Median

`median` contains the median calculated at:

```text
q = 0.5
```

### Q3

`q3` contains the third quartile calculated at:

```text
q = 0.75
```

### IQR

`iqr` contains the interquartile range:

```text
IQR = Q3 - Q1
```

For example:

```text
Q1 = 2.25
Q3 = 5.5
```

produces:

```text
IQR = 3.25
```

### N

`n` contains the number of observations in the original series.

For:

```json
[1, 2, 3, 4, 6, 8]
```

the returned value is:

```text
n = 6
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Quartiles and IQR

**Configuration**

```json
{
  "series": [1, 2, 3, 4, 6, 8],
  "qs": []
}
```

**Output**

```json
{
  "qs": [],
  "values": {},
  "q1": 2.25,
  "median": 3.5,
  "q3": 5.5,
  "iqr": 3.25,
  "n": 6
}
```

This example demonstrates interpolation when quartile positions fall between values.

### Example 2: Unsorted Series

**Configuration**

```json
{
  "series": [5, 1, 4, 2, 3],
  "qs": []
}
```

The node sorts a copy of the series internally for quartile calculations.

**Output**

```json
{
  "qs": [],
  "values": {},
  "q1": 2,
  "median": 3,
  "q3": 4,
  "iqr": 2,
  "n": 5
}
```

### Example 3: Odd-Length Series

**Configuration**

```json
{
  "series": [1, 2, 3, 4, 5],
  "qs": []
}
```

**Output**

```json
{
  "qs": [],
  "values": {},
  "q1": 2,
  "median": 3,
  "q3": 4,
  "iqr": 2,
  "n": 5
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Quantiles Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Series Is Empty

The schema requires `series` to contain at least one numerical value.

An empty series is rejected with a validation error similar to:

```text
series: Array must have at least 1 items
```

**Solution:** Add at least one numerical value to `series`.

### Qs Must Be an Array

The `qs` parameter is defined as an array of numbers.

If the workflow editor or configuration sends a non-array value, validation can fail with:

```text
qs: Expected an array
```

**Solution:** Ensure `qs` is serialized as an array.

### Empty Qs Does Not Use the Schema Default

If the editor sends:

```json
"qs": []
```

the field is present and therefore the default:

```json
[0.25, 0.5, 0.75]
```

is not substituted.

The node consequently returns:

```json
{
  "qs": [],
  "values": {}
}
```

This does not prevent Q1, median, Q3, or IQR from being calculated.

### Custom Qs from the Workflow Editor

During editor configuration, custom `qs` entries must be serialized as a numeric array expected by the schema.

If the editor serializes those entries in another form, validation can report:

```text
qs: Expected an array
```

This concerns the configuration representation passed to the validator rather than the quartile calculations themselves.

### Quantile Is Outside the Valid Range

Every configured `qs` value must be between:

```text
0
```

and:

```text
1
```

Values outside that range are rejected by schema validation.

### Input Series Is Not Sorted

No manual sorting is required.

The implementation creates a copy:

```text
[...series]
```

and sorts it numerically before calculating Q1, median, and Q3.

The original configured series is not modified.

### Decimal Results

For even-sized datasets or quantile positions between observations, the quantile helper can return interpolated decimal values.

For example:

```json
[1, 2, 3, 4, 6, 8]
```

produces:

```text
Q1     = 2.25
Median = 3.5
Q3     = 5.5
IQR    = 3.25
```

This is expected behavior.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Probability** — Calculate probability distribution functions.
- **Dispersion** — Calculate statistical measures of dispersion.
- **Outlier Detection** — Detect unusual observations in numerical datasets.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation for the Quantiles / Percentiles / IQR node. |

<!-- /SECTION: changelog -->
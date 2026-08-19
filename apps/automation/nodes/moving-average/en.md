---
node_id: "moving-average"
title: "Moving Average"
description: "Calculates Simple Moving Average (SMA), Exponential Moving Average (EMA), or Weighted Moving Average (WMA)."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - moving-average
  - sma
  - ema
  - wma
  - statistics
related_nodes:
  - correlation
  - dispersion
  - regression
---

<!-- SECTION: header -->
# Moving Average

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate Simple Moving Average (SMA), Exponential Moving Average (EMA), or Weighted Moving Average (WMA) values from a numeric time series.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Moving Average** node calculates moving averages over a numeric time series using one of three supported methods:

- `sma` — Simple Moving Average
- `ema` — Exponential Moving Average
- `wma` — Weighted Moving Average

The node uses the configured series and window size to generate a reduced output series containing the calculated moving-average values.

### Key Features

- Calculates Simple Moving Average (SMA).
- Calculates Exponential Moving Average (EMA).
- Calculates Weighted Moving Average (WMA).
- Supports configurable window sizes.
- Accepts numeric time-series data.
- Rounds calculated values to six decimal places.
- Returns input and output lengths.
- Validates that the window does not exceed the series length.

### Processing Flow

```text
Moving average type
  ↓
Numeric series
  ↓
Window size
  ↓
Validate window
  ↓
Select SMA, EMA, or WMA
  ↓
Calculate moving-average values
  ↓
Round results to six decimals
  ↓
Return result
```

### Use Cases

- Smoothing time-series data.
- Analyzing short-term trends.
- Calculating financial moving averages.
- Processing sensor or measurement data.
- Comparing different moving-average methods.
- Preparing smoothed data for downstream statistical workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `type` | `string` | No | `sma` | Moving-average type: `sma`, `ema`, or `wma`. |
| `series` | `array<number>` | Yes | — | Numeric time-series data. Must contain at least one value. |
| `window` | `number` | Yes | — | Moving-average window size. Must be greater than or equal to `1`. |

### Type

Select the moving-average calculation method.

Supported values:

```text
sma
ema
wma
```

### Series

Provide the numeric time-series data.

Example:

```text
10
20
30
40
50
```

The series must contain at least one numeric value.

### Window

Provide the moving-average window size.

Example:

```text
3
```

The configured window must be greater than or equal to `1`.

Before calculating the moving average, the node verifies that the configured `window` is not greater than the series length.

After this validation, the current implementation applies:

```text
Math.floor(window)
```

to determine the effective window used by the calculation.

For example:

```text
window: 3.8
```

with a series containing at least `4` values is processed using an effective window of:

```text
3
```

### SMA Calculation

The Simple Moving Average calculates the arithmetic mean of each consecutive window.

For:

```text
series: [10, 20, 30, 40, 50]
window: 3
```

the calculations are:

```text
(10 + 20 + 30) / 3 = 20
(20 + 30 + 40) / 3 = 30
(30 + 40 + 50) / 3 = 40
```

Result:

```text
[20, 30, 40]
```

### EMA Calculation

The Exponential Moving Average uses the following smoothing factor:

```text
k = 2 / (window + 1)
```

The first EMA value is initialized using the mean of the first window.

For:

```text
series: [10, 20, 30, 40, 50]
window: 3
```

the initial value is:

```text
(10 + 20 + 30) / 3 = 20
```

The smoothing factor is:

```text
2 / (3 + 1) = 0.5
```

The remaining values are calculated as:

```text
40 × 0.5 + 20 × 0.5 = 30
50 × 0.5 + 30 × 0.5 = 40
```

Result:

```text
[20, 30, 40]
```

### WMA Calculation

The Weighted Moving Average assigns increasing weights from the oldest to the newest value.

For a window of `3`, the weights are:

```text
1, 2, 3
```

The denominator is:

```text
3 × (3 + 1) / 2 = 6
```

For:

```text
series: [10, 20, 30, 40, 50]
window: 3
```

the calculations are:

```text
(10 × 1 + 20 × 2 + 30 × 3) / 6 = 23.333333
(20 × 1 + 30 × 2 + 40 × 3) / 6 = 33.333333
(30 × 1 + 40 × 2 + 50 × 3) / 6 = 43.333333
```

Result:

```text
[23.333333, 33.333333, 43.333333]
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `type`
- `series`
- `window`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- moving-average type;
- calculated values;
- effective window size;
- input series length;
- output series length.

Example:

```json
{
  "type": "wma",
  "values": [
    23.333333,
    33.333333,
    43.333333
  ],
  "window": 3,
  "inputLength": 5,
  "outputLength": 3
}
```

### Values

The `values` field contains the calculated moving-average values.

Each value is rounded to six decimal places using:

```text
Math.round(value × 1000000) / 1000000
```

### Window

The returned `window` field contains the effective integer window used for the calculation:

```text
Math.floor(window)
```

### Input Length

The `inputLength` field contains the number of values in the original series.

For:

```text
series: [10, 20, 30, 40, 50]
```

the result is:

```text
5
```

### Output Length

The `outputLength` field contains the number of calculated moving-average values.

For:

```text
series: [10, 20, 30, 40, 50]
window: 3
```

the result is:

```text
3
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Simple Moving Average

**Configuration**

```text
type: sma
series:
  - 10
  - 20
  - 30
  - 40
  - 50
window: 3
```

**Output**

```json
{
  "type": "sma",
  "values": [
    20,
    30,
    40
  ],
  "window": 3,
  "inputLength": 5,
  "outputLength": 3
}
```

### Example 2: Exponential Moving Average

**Configuration**

```text
type: ema
series:
  - 10
  - 20
  - 30
  - 40
  - 50
window: 3
```

**Output**

```json
{
  "type": "ema",
  "values": [
    20,
    30,
    40
  ],
  "window": 3,
  "inputLength": 5,
  "outputLength": 3
}
```

### Example 3: Weighted Moving Average

**Configuration**

```text
type: wma
series:
  - 10
  - 20
  - 30
  - 40
  - 50
window: 3
```

**Output**

```json
{
  "type": "wma",
  "values": [
    23.333333,
    33.333333,
    43.333333
  ],
  "window": 3,
  "inputLength": 5,
  "outputLength": 3
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Moving Average Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Series Is Invalid

**Cause:** `series` is empty or does not contain valid numeric values.

**Solution:** Provide an array containing at least one number.

### Window Is Invalid

**Cause:** `window` is below `1`.

**Solution:** Use a value greater than or equal to `1`.

### Window Is Greater Than Series Length

**Cause:** The configured `window` is greater than the number of values in `series`.

The node throws:

```text
window cannot be greater than series length
```

**Solution:** Use a window value less than or equal to the number of values in the series.

### Decimal Window Value

The `window` parameter accepts numeric values, including decimals.

The node first validates the original configured value against the series length. It then applies:

```text
Math.floor(window)
```

for the actual moving-average calculation.

For example:

```text
window: 3.8
```

with a sufficiently long series is calculated using:

```text
window: 3
```

If the original decimal value is greater than the series length, the node throws:

```text
window cannot be greater than series length
```

before applying `Math.floor()`.

### Unexpected Output Length

The number of output values depends on the series length and effective window size.

For a series containing `5` values with a window of `3`, the node returns `3` moving-average values.

### Unexpected Moving Average Values

**Cause:** The selected moving-average method or configured values may not match the intended calculation.

**Solution:** Verify `type`, `series`, and `window`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Correlation** — Measure statistical relationships between variables.
- **Dispersion** — Analyze statistical dispersion.
- **Regression** — Perform regression analysis.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Moving Average node. |

<!-- /SECTION: changelog -->
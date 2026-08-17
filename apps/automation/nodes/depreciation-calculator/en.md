---
node_id: "depreciation-calculator"
title: "Depreciation Calculator"
description: "Calculate asset depreciation using straight-line or declining balance methods"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - depreciation
  - finance
  - asset
  - straight-line
  - declining-balance
related_nodes:
  - loan-calculator
  - mortgage-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Depreciation Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate asset depreciation schedules using straight-line or declining balance methods.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Depreciation Calculator** node calculates asset depreciation over a specified useful life.

It supports two depreciation methods:

- `straight-line`
- `declining-balance`

The node returns a complete yearly depreciation schedule including depreciation amount, accumulated depreciation, and book value.

### Key Features

- Calculates straight-line depreciation.
- Calculates double declining balance depreciation.
- Supports configurable asset cost.
- Supports configurable salvage value.
- Supports useful life from 1 to 50 years.
- Generates a yearly depreciation schedule.
- Tracks accumulated depreciation.
- Tracks remaining book value.
- Prevents declining-balance book value from falling below the salvage value.

### Processing Flow

```text
Asset cost
  ↓
Salvage value
  ↓
Useful life
  ↓
Select depreciation method
  ↓
Calculate yearly depreciation
  ↓
Build depreciation schedule
  ↓
Return result
```

### Use Cases

- Calculating fixed asset depreciation.
- Generating depreciation schedules.
- Estimating yearly asset expenses.
- Tracking accumulated depreciation.
- Calculating remaining book value.
- Preparing financial data for downstream processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `cost` | `number` | No | `100000` | Initial cost of the asset. Must be greater than or equal to `0`. |
| `salvageValue` | `number` | No | `10000` | Expected residual value of the asset. Must be greater than or equal to `0`. |
| `usefulLife` | `number` | No | `10` | Useful life of the asset in years. Must be between `1` and `50`. |
| `method` | `string` | No | `straight-line` | Depreciation method: `straight-line` or `declining-balance`. |

### Cost

Provide the initial cost of the asset.

Example:

```text
100000
```

### Salvage Value

Provide the expected value of the asset at the end of its useful life.

Example:

```text
10000
```

### Useful Life

Provide the number of years over which depreciation should be calculated.

Example:

```text
10
```

### Method

The node supports two depreciation methods.

#### straight-line

The depreciable amount is calculated as:

```text
cost - salvageValue
```

Annual depreciation is then:

```text
(cost - salvageValue) / usefulLife
```

For:

```text
cost: 100000
salvageValue: 10000
usefulLife: 10
```

the annual depreciation is:

```text
9000
```

#### declining-balance

The node uses the double declining balance method.

The depreciation rate is:

```text
2 / usefulLife
```

For a useful life of `10` years, the rate is:

```text
20%
```

The yearly depreciation is capped so the calculated book value does not fall below the configured salvage value.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses the configured values:

- `cost`
- `salvageValue`
- `usefulLife`
- `method`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- cost;
- salvage value;
- useful life;
- selected method;
- yearly depreciation schedule.

Example structure:

```json
{
  "success": true,
  "cost": 100000,
  "salvageValue": 10000,
  "usefulLife": 10,
  "method": "straight-line",
  "schedule": [
    {
      "year": 1,
      "depreciation": 9000,
      "accumulated": 9000,
      "bookValue": 91000
    }
  ]
}
```

### Schedule

Each schedule entry contains:

| Field | Description |
|-------|-------------|
| `year` | Depreciation year. |
| `depreciation` | Depreciation amount for the year. |
| `accumulated` | Total accumulated depreciation. |
| `bookValue` | Remaining book value of the asset. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Straight-Line Depreciation

**Configuration**

```text
cost: 100000
salvageValue: 10000
usefulLife: 10
method: straight-line
```

The depreciable amount is:

```text
90000
```

The yearly depreciation is:

```text
9000
```

The first schedule entry is:

```json
{
  "year": 1,
  "depreciation": 9000,
  "accumulated": 9000,
  "bookValue": 91000
}
```

The final book value after 10 years is:

```text
10000
```

### Example 2: Declining Balance Depreciation

**Configuration**

```text
cost: 100000
salvageValue: 10000
usefulLife: 10
method: declining-balance
```

The depreciation rate is:

```text
20%
```

The first year is:

```json
{
  "year": 1,
  "depreciation": 20000,
  "accumulated": 20000,
  "bookValue": 80000
}
```

The second year is:

```json
{
  "year": 2,
  "depreciation": 16000,
  "accumulated": 36000,
  "bookValue": 64000
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Depreciation Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Useful Life Is Invalid

**Cause:** `usefulLife` is outside the supported range.

**Solution:** Use a value between `1` and `50`.

### Cost or Salvage Value Is Invalid

**Cause:** `cost` or `salvageValue` is below `0`.

**Solution:** Use values greater than or equal to `0`.

### Unexpected Depreciation Schedule

**Cause:** The selected depreciation method may not match the intended calculation.

**Solution:** Verify whether `straight-line` or `declining-balance` is selected.

### Book Value Stops at Salvage Value

This is expected behavior for the declining balance method.

The calculation prevents depreciation from reducing the asset below its configured salvage value.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Loan Calculator** — Calculate loan payment information.
- **Mortgage Calculator** — Calculate mortgage payment information.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Depreciation Calculator node. |

<!-- /SECTION: changelog -->
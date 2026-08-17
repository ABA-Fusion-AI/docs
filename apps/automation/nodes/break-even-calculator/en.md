---
node_id: "break-even-calculator"
title: "Break-Even Calculator"
description: "Calculate break-even point, contribution margin, and margin of safety"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - break-even
  - finance
  - contribution-margin
  - revenue
  - margin-of-safety
related_nodes:
  - budget-variance
  - profit-margin-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Break-Even Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate break-even point, contribution margin, break-even revenue, and margin of safety.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Break-Even Calculator** node calculates the break-even point for a business or product based on fixed costs, variable cost per unit, and selling price per unit.

It can also calculate the contribution margin, contribution margin ratio, break-even revenue, and margin of safety when the number of units is provided.

### Key Features

- Calculates contribution margin per unit.
- Calculates contribution margin ratio.
- Calculates break-even units.
- Calculates break-even revenue.
- Calculates margin of safety when units are provided.
- Rounds break-even units up to the next whole unit.
- Returns structured financial calculation results.

### Processing Flow

```text
Fixed costs
  ↓
Variable cost per unit
  ↓
Price per unit
  ↓
Calculate contribution margin
  ↓
Calculate break-even point
  ↓
Calculate margin of safety when units are provided
  ↓
Return result
```

### Use Cases

- Calculating the sales volume required to break even.
- Evaluating product profitability.
- Measuring contribution margin.
- Estimating break-even revenue.
- Calculating margin of safety.
- Preparing financial data for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fixedCosts` | `number` | No | `10000` | Total fixed costs. Must be greater than or equal to `0`. |
| `variableCostPerUnit` | `number` | No | `5` | Variable cost for each unit. Must be greater than or equal to `0`. |
| `pricePerUnit` | `number` | No | `10` | Selling price for each unit. The schema accepts values greater than or equal to `0`; use a value greater than `0` for meaningful break-even calculations. |
| `units` | `number` | No | `0` | Number of units used to calculate margin of safety. |

### Fixed Costs

Provide the total fixed costs.

Example:

```text
10000
```

### Variable Cost Per Unit

Provide the variable cost associated with producing or selling one unit.

Example:

```text
5
```

### Price Per Unit

Provide the selling price for one unit.

Example:

```text
10
```

### Units

Provide the current or expected number of units when calculating the margin of safety.

Example:

```text
2500
```

When `units` is `0` or not greater than `0`, the node returns `null` for the margin of safety.

### Calculation Formulas

Contribution margin:

```text
contributionMargin = pricePerUnit - variableCostPerUnit
```

Break-even units:

```text
breakEvenUnits = fixedCosts / contributionMargin
```

Break-even revenue:

```text
breakEvenRevenue = breakEvenUnits × pricePerUnit
```

Contribution margin ratio:

```text
contributionMarginRatio = contributionMargin / pricePerUnit × 100
```

Margin of safety:

```text
marginOfSafety = (units - breakEvenUnits) / units × 100
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `fixedCosts`
- `variableCostPerUnit`
- `pricePerUnit`
- `units`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- fixed costs;
- variable cost per unit;
- price per unit;
- contribution margin;
- contribution margin ratio;
- break-even units;
- break-even revenue;
- margin of safety;
- units.

Example:

```json
{
  "success": true,
  "fixedCosts": 10000,
  "variableCostPerUnit": 5,
  "pricePerUnit": 10,
  "contributionMargin": 5,
  "contributionMarginRatio": 50,
  "breakEvenUnits": 2000,
  "breakEvenRevenue": 20000,
  "marginOfSafety": 20,
  "units": 2500
}
```

### Break-Even Units

The calculated break-even value is rounded up using `Math.ceil()`.

For example, if the raw break-even calculation is:

```text
2000.4
```

the returned value is:

```text
2001
```

### Margin of Safety

When `units` is greater than `0`, the node calculates the margin of safety as a percentage.

When `units` is `0`, the returned values are:

```json
{
  "marginOfSafety": null,
  "units": null
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic Break-Even Calculation

**Configuration**

```text
fixedCosts: 10000
variableCostPerUnit: 5
pricePerUnit: 10
units: 0
```

**Output**

```json
{
  "success": true,
  "fixedCosts": 10000,
  "variableCostPerUnit": 5,
  "pricePerUnit": 10,
  "contributionMargin": 5,
  "contributionMarginRatio": 50,
  "breakEvenUnits": 2000,
  "breakEvenRevenue": 20000,
  "marginOfSafety": null,
  "units": null
}
```

### Example 2: Calculate Margin of Safety

**Configuration**

```text
fixedCosts: 10000
variableCostPerUnit: 5
pricePerUnit: 10
units: 2500
```

**Output**

```json
{
  "success": true,
  "fixedCosts": 10000,
  "variableCostPerUnit": 5,
  "pricePerUnit": 10,
  "contributionMargin": 5,
  "contributionMarginRatio": 50,
  "breakEvenUnits": 2000,
  "breakEvenRevenue": 20000,
  "marginOfSafety": 20,
  "units": 2500
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Break-Even Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Fixed Costs Are Invalid

**Cause:** `fixedCosts` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Variable Cost Per Unit Is Invalid

**Cause:** `variableCostPerUnit` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Price Per Unit Is Invalid

**Cause:** `pricePerUnit` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Contribution Margin Is Zero or Negative

**Cause:** `pricePerUnit` is less than or equal to `variableCostPerUnit`.

**Solution:** Use a `pricePerUnit` greater than `variableCostPerUnit` to calculate a meaningful break-even point.

### Margin of Safety Is Null

**Cause:** `units` is `0` or not greater than `0`.

**Solution:** Provide a positive number of units to calculate the margin of safety.

### Unexpected Break-Even Result

**Cause:** One or more financial parameters may not match the intended calculation.

**Solution:** Verify `fixedCosts`, `variableCostPerUnit`, `pricePerUnit`, and `units`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Budget Variance** — Compare budgeted and actual financial values.
- **Profit Margin Calculator** — Calculate profit margin values.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Break-Even Calculator node. |

<!-- /SECTION: changelog -->
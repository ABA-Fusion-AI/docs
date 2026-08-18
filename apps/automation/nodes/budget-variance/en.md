---
node_id: "budget-variance"
title: "Budget Variance Analysis"
description: "Analyze budget vs actual spending with variance calculations"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - budget
  - variance
  - finance
  - spending
  - analysis
related_nodes:
  - break-even-calculator
  - profit-margin-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Budget Variance Analysis

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Compare budgeted values with actual spending and calculate variance by category and in total.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Budget Variance Analysis** node compares budgeted amounts with actual spending for each configured category.

It calculates the variance amount, variance percentage, and budget status for every category defined in the budget object.

The node also returns overall budget, actual spending, total variance, and total variance percentage.

### Key Features

- Compares budgeted and actual values.
- Calculates variance by category.
- Calculates variance percentage by category.
- Classifies each category as `Over Budget`, `Under Budget`, or `On Budget`.
- Calculates total budget.
- Calculates total actual spending.
- Calculates total variance.
- Calculates total variance percentage.
- Treats missing actual values as `0`.

### Processing Flow

```text
Budget object
  ↓
Actual object
  ↓
Read budget categories
  ↓
Compare budget vs actual
  ↓
Calculate variance and percentage
  ↓
Assign budget status
  ↓
Calculate totals
  ↓
Return analysis
```

### Use Cases

- Comparing planned and actual spending.
- Tracking departmental budgets.
- Identifying overspending.
- Identifying underspending.
- Monitoring financial performance.
- Preparing variance reports for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `budget` | `object` | No | `{}` | Key-value object containing budgeted amounts by category. |
| `actual` | `object` | No | `{}` | Key-value object containing actual amounts by category. |

### Budget

Provide budget values as category and amount pairs.

Example:

```json
{
  "marketing": 10000,
  "operations": 20000
}
```

The node uses the keys from `budget` as the categories to analyze.

### Actual

Provide actual spending values using matching category names.

Example:

```json
{
  "marketing": 12000,
  "operations": 18000
}
```

If an actual value is missing for a budget category, the node uses `0`.

### Variance Calculation

Variance is calculated as:

```text
variance = actual - budget
```

Variance percentage is calculated as:

```text
variancePercent = variance / budget × 100
```

### Status

The node assigns the following status:

```text
variance > 0  → Over Budget
variance < 0  → Under Budget
variance = 0  → On Budget
```

### Category Handling

Only categories defined in `budget` are analyzed.

If `actual` contains additional categories that do not exist in `budget`, those categories are not included in the analysis.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `budget`
- `actual`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- category-level analysis;
- overall totals.

Example:

```json
{
  "success": true,
  "categories": {
    "marketing": {
      "budget": 10000,
      "actual": 12000,
      "variance": 2000,
      "variancePercent": 20,
      "status": "Over Budget"
    },
    "operations": {
      "budget": 20000,
      "actual": 18000,
      "variance": -2000,
      "variancePercent": -10,
      "status": "Under Budget"
    }
  },
  "totals": {
    "budget": 30000,
    "actual": 30000,
    "variance": 0,
    "variancePercent": 0
  }
}
```

### Category Analysis

Each category contains:

| Field | Description |
|-------|-------------|
| `budget` | Budgeted amount. |
| `actual` | Actual amount. |
| `variance` | Difference between actual and budget. |
| `variancePercent` | Variance expressed as a percentage of budget. |
| `status` | `Over Budget`, `Under Budget`, or `On Budget`. |

### Totals

The `totals` object contains:

- total budget;
- total actual spending;
- total variance;
- total variance percentage.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Budget Variance Analysis

**Budget**

```json
{
  "marketing": 10000,
  "operations": 20000
}
```

**Actual**

```json
{
  "marketing": 12000,
  "operations": 18000
}
```

**Output**

```json
{
  "success": true,
  "categories": {
    "marketing": {
      "budget": 10000,
      "actual": 12000,
      "variance": 2000,
      "variancePercent": 20,
      "status": "Over Budget"
    },
    "operations": {
      "budget": 20000,
      "actual": 18000,
      "variance": -2000,
      "variancePercent": -10,
      "status": "Under Budget"
    }
  },
  "totals": {
    "budget": 30000,
    "actual": 30000,
    "variance": 0,
    "variancePercent": 0
  }
}
```

### Example 2: Missing Actual Value

For:

```json
{
  "budget": {
    "marketing": 10000
  },
  "actual": {}
}
```

the node treats the missing actual value as `0`.

The category result is:

```json
{
  "marketing": {
    "budget": 10000,
    "actual": 0,
    "variance": -10000,
    "variancePercent": -100,
    "status": "Under Budget"
  }
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Budget Variance Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Budget or Actual Is Invalid

**Cause:** One of the configured values is not a key-value object containing numeric values.

**Solution:** Provide objects where each category maps to a number.

### Missing Actual Category

This is expected behavior.

If a budget category is missing from `actual`, the node uses `0` as the actual value.

### Additional Actual Categories Are Ignored

This is expected behavior.

Only categories defined in `budget` are analyzed.

### Budget Value Is Zero

**Cause:** A category has a budget value of `0`, which prevents a meaningful variance percentage from being calculated.

**Solution:** Use a non-zero budget value when a meaningful variance percentage is required.

### Total Budget Is Zero

**Cause:** The total budget is `0`, which prevents a meaningful total variance percentage from being calculated.

**Solution:** Use a non-zero total budget when a meaningful total variance percentage is required.

### Unexpected Variance Result

**Cause:** One or more budget or actual values may not match the intended calculation.

**Solution:** Verify the category names and numeric values in `budget` and `actual`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Break-Even Calculator** — Calculate break-even point and contribution margin.
- **Profit Margin Calculator** — Calculate profit margin values.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Budget Variance Analysis node. |

<!-- /SECTION: changelog -->
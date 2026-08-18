---
node_id: "escrow-calculator"
title: "Escrow Account Calculator"
description: "Calculate monthly escrow payments for property taxes and insurance"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - escrow
  - property-tax
  - insurance
  - real-estate
  - finance
related_nodes:
  - mortgage-calculator
  - loan-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Escrow Account Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate monthly escrow payments for property taxes and insurance.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Escrow Account Calculator** node calculates estimated monthly escrow payments based on a home's price, property tax rate, and annual insurance cost.

It calculates annual property tax, monthly property tax, monthly insurance, total monthly escrow, and a two-month escrow cushion.

### Key Features

- Calculates annual property tax.
- Calculates monthly property tax.
- Calculates monthly insurance.
- Calculates total monthly escrow.
- Calculates a two-month escrow cushion.
- Returns the two-month cushion as the total required amount.
- Supports configurable home price, tax rate, and annual insurance.
- Returns structured financial calculation results.

### Processing Flow

```text
Home price
  ↓
Property tax rate
  ↓
Annual insurance
  ↓
Calculate annual property tax
  ↓
Calculate monthly property tax
  ↓
Calculate monthly insurance
  ↓
Calculate monthly escrow
  ↓
Calculate two-month cushion
  ↓
Return result
```

### Use Cases

- Estimating monthly escrow payments.
- Calculating monthly property tax allocations.
- Calculating monthly insurance allocations.
- Estimating an escrow cushion.
- Supporting mortgage-related calculations.
- Preparing escrow information for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `homePrice` | `number` | No | `300000` | Home price used to calculate property tax. Must be greater than or equal to `0`. |
| `propertyTaxRate` | `number` | No | `1.5` | Annual property tax rate as a percentage. Must be between `0` and `100`. |
| `insuranceAnnual` | `number` | No | `1200` | Annual insurance amount. Must be greater than or equal to `0`. |

### Home Price

Provide the home price used as the basis for the property tax calculation.

Example:

```text
300000
```

### Property Tax Rate

Provide the annual property tax rate as a percentage.

Example:

```text
1.5
```

For a home price of `300000` and a property tax rate of `1.5`, the annual property tax is:

```text
4500
```

### Annual Insurance

Provide the annual insurance amount.

Example:

```text
1200
```

The node divides this value by `12` to calculate the monthly insurance amount.

### Calculation Details

Annual property tax:

```text
annualPropertyTax = homePrice × propertyTaxRate / 100
```

Monthly property tax:

```text
monthlyTax = annualPropertyTax / 12
```

Monthly insurance:

```text
monthlyInsurance = insuranceAnnual / 12
```

Monthly escrow:

```text
monthlyEscrow = monthlyTax + monthlyInsurance
```

Escrow cushion:

```text
cushion = monthlyEscrow × 2
```

The current implementation returns the cushion value as `totalRequired`:

```text
totalRequired = cushion
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `homePrice`
- `propertyTaxRate`
- `insuranceAnnual`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- home price;
- annual property tax;
- annual insurance;
- monthly property tax;
- monthly insurance;
- monthly escrow;
- two-month cushion;
- total required amount.

Example:

```json
{
  "success": true,
  "homePrice": 300000,
  "annualPropertyTax": 4500,
  "annualInsurance": 1200,
  "monthlyTax": 375,
  "monthlyInsurance": 100,
  "monthlyEscrow": 475,
  "cushion": 950,
  "totalRequired": 950
}
```

### Monthly Escrow

The `monthlyEscrow` field is the sum of monthly property tax and monthly insurance.

For the default configuration:

```text
monthlyTax: 375
monthlyInsurance: 100
monthlyEscrow: 475
```

### Cushion

The `cushion` field represents two months of calculated escrow payments.

For:

```text
monthlyEscrow: 475
```

the cushion is:

```text
950
```

### Total Required

In the current implementation, `totalRequired` contains the same value as `cushion`.

For the default configuration:

```text
totalRequired: 950
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Default Escrow Calculation

**Configuration**

```text
homePrice: 300000
propertyTaxRate: 1.5
insuranceAnnual: 1200
```

**Output**

```json
{
  "success": true,
  "homePrice": 300000,
  "annualPropertyTax": 4500,
  "annualInsurance": 1200,
  "monthlyTax": 375,
  "monthlyInsurance": 100,
  "monthlyEscrow": 475,
  "cushion": 950,
  "totalRequired": 950
}
```

### Example 2: Higher Home Price

**Configuration**

```text
homePrice: 500000
propertyTaxRate: 2
insuranceAnnual: 2400
```

**Output**

```json
{
  "success": true,
  "homePrice": 500000,
  "annualPropertyTax": 10000,
  "annualInsurance": 2400,
  "monthlyTax": 833.3333333333334,
  "monthlyInsurance": 200,
  "monthlyEscrow": 1033.3333333333335,
  "cushion": 2066.666666666667,
  "totalRequired": 2066.666666666667
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Escrow Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Home Price Is Invalid

**Cause:** `homePrice` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Property Tax Rate Is Invalid

**Cause:** `propertyTaxRate` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Annual Insurance Is Invalid

**Cause:** `insuranceAnnual` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Unexpected Monthly Escrow

**Cause:** One or more configured values may not match the intended calculation.

**Solution:** Verify `homePrice`, `propertyTaxRate`, and `insuranceAnnual`.

### Total Required Matches Cushion

This is expected behavior in the current implementation.

The node assigns the two-month `cushion` directly to `totalRequired`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Mortgage Calculator** — Calculate mortgage payment information.
- **Loan Calculator** — Calculate loan payments and interest.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial documentation for the Escrow Account Calculator node. |

<!-- /SECTION: changelog -->
---
node_id: "loan-calculator"
title: "Loan Calculator"
description: "Calculate loan payments, interest, and generate amortization schedules"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - loan
  - payment
  - interest
  - amortization
  - finance
related_nodes:
  - mortgage-calculator
  - depreciation-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Loan Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate loan payments, interest, and amortization schedules using multiple financial calculation modes.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Loan Calculator** node performs common loan and interest calculations.

It supports four operations:

- `monthlyPayment`
- `amortizationSchedule`
- `simpleInterest`
- `compoundInterest`

The node uses configured financial parameters and returns structured calculation results.

### Key Features

- Calculates monthly loan payments.
- Generates full amortization schedules.
- Calculates simple interest.
- Calculates compound interest.
- Calculates total payment amounts.
- Calculates total interest.
- Supports configurable loan duration.
- Supports configurable compounding frequency.
- Returns detailed structured results.

### Processing Flow

```text
Financial parameters
  ↓
Select operation
  ↓
Apply calculation formula
  ↓
Generate result
  ↓
Return structured output
```

### Use Cases

- Calculating monthly loan payments.
- Generating amortization schedules.
- Calculating simple interest.
- Calculating compound interest.
- Estimating total loan repayment.
- Preparing financial calculations for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `string` | No | `monthlyPayment` | Calculation mode: `monthlyPayment`, `amortizationSchedule`, `simpleInterest`, or `compoundInterest`. |
| `principal` | `number` | No | `100000` | Principal amount used in the calculation. Must be greater than or equal to `0`. |
| `annualRate` | `number` | No | `4.5` | Annual interest rate used for loan payment calculations. Must be between `0` and `100`. |
| `years` | `number` | No | `30` | Loan duration in years. The schema accepts values greater than or equal to `0`; use a value greater than `0` for payment and amortization calculations. |
| `rate` | `number` | No | `5` | Interest rate used for simple and compound interest calculations. Must be between `0` and `100`. |
| `time` | `number` | No | `1` | Time period used for simple and compound interest calculations. Must be greater than or equal to `0`. |
| `frequency` | `number` | No | `12` | Number of compounding periods per time unit. Must be at least `1`. |

### Operation

The node supports four operations.

#### monthlyPayment

Calculates the monthly payment for a loan using the configured principal, annual interest rate, and loan duration.

For:

```text
principal: 100000
annualRate: 4.5
years: 30
```

the monthly payment is approximately:

```text
506.6853098258858
```

#### amortizationSchedule

Generates a monthly amortization schedule.

Each schedule entry contains:

- month;
- payment;
- principal payment;
- interest payment;
- remaining balance;
- accumulated interest.

#### simpleInterest

Calculates simple interest using:

```text
(principal × rate × time) / 100
```

For:

```text
principal: 100000
rate: 5
time: 1
```

the result is:

```text
interest: 5000
total: 105000
```

#### compoundInterest

Calculates compound interest using the configured principal, rate, time, and compounding frequency.

For:

```text
principal: 100000
rate: 5
time: 1
frequency: 12
```

the total amount is approximately:

```text
105116.1897881733
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `operation`
- `principal`
- `annualRate`
- `years`
- `rate`
- `time`
- `frequency`

Incoming workflow data is not used for the financial calculations.

### Monthly Payment Output

Example:

```json
{
  "success": true,
  "monthlyPayment": 506.6853098258858,
  "principal": 100000,
  "annualRate": 4.5,
  "years": 30,
  "totalPayment": 182406.7115373189
}
```

### Amortization Schedule Output

The `amortizationSchedule` operation returns loan summary information and a monthly schedule.

Example structure:

```json
{
  "success": true,
  "principal": 100000,
  "annualRate": 4.5,
  "years": 30,
  "monthlyPayment": 506.6853098258858,
  "totalPayment": 182406.7115373189,
  "totalInterest": 82406.71153731487,
  "schedule": [
    {
      "month": 1,
      "payment": 506.6853098258858,
      "principal": 131.68530982588578,
      "interest": 375,
      "balance": 99868.31469017411,
      "totalInterest": 375
    }
  ]
}
```

### Simple Interest Output

Example:

```json
{
  "success": true,
  "principal": 100000,
  "rate": 5,
  "time": 1,
  "interest": 5000,
  "total": 105000
}
```

### Compound Interest Output

Example:

```json
{
  "success": true,
  "principal": 100000,
  "rate": 5,
  "time": 1,
  "frequency": 12,
  "interest": 5116.189788173302,
  "totalAmount": 105116.1897881733,
  "effectiveRate": 5.116189788173298
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Calculate Monthly Payment

**Configuration**

```text
operation: monthlyPayment
principal: 100000
annualRate: 4.5
years: 30
```

**Output**

```json
{
  "success": true,
  "monthlyPayment": 506.6853098258858,
  "principal": 100000,
  "annualRate": 4.5,
  "years": 30,
  "totalPayment": 182406.7115373189
}
```

### Example 2: Generate an Amortization Schedule

**Configuration**

```text
operation: amortizationSchedule
principal: 100000
annualRate: 4.5
years: 30
```

The node generates `360` monthly schedule entries.

### Example 3: Calculate Simple Interest

**Configuration**

```text
operation: simpleInterest
principal: 100000
rate: 5
time: 1
```

**Output**

```json
{
  "success": true,
  "principal": 100000,
  "rate": 5,
  "time": 1,
  "interest": 5000,
  "total": 105000
}
```

### Example 4: Calculate Compound Interest

**Configuration**

```text
operation: compoundInterest
principal: 100000
rate: 5
time: 1
frequency: 12
```

**Output**

```json
{
  "success": true,
  "principal": 100000,
  "rate": 5,
  "time": 1,
  "frequency": 12,
  "interest": 5116.189788173302,
  "totalAmount": 105116.1897881733,
  "effectiveRate": 5.116189788173298
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Loan Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Principal or Rate Is Invalid

**Cause:** A numeric value is outside the allowed range.

**Solution:** Verify that principal and time values are not negative and that interest rates are between `0` and `100`.

### Frequency Is Invalid

**Cause:** `frequency` is below `1`.

**Solution:** Use a compounding frequency of at least `1`.

### Loan Duration Is Zero

**Cause:** `years` is set to `0` for `monthlyPayment` or `amortizationSchedule`.

**Solution:** Use a loan duration greater than `0` for payment and amortization calculations.

### Unexpected Monthly Payment

**Cause:** One or more loan parameters may not match the intended calculation.

**Solution:** Verify `principal`, `annualRate`, and `years`.

### Unexpected Interest Result

**Cause:** The selected operation or interest parameters may not match the intended calculation.

**Solution:** Verify whether `simpleInterest` or `compoundInterest` is selected and check `principal`, `rate`, `time`, and `frequency`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Mortgage Calculator** — Calculate mortgage payment information.
- **Depreciation Calculator** — Calculate asset depreciation schedules.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Loan Calculator node. |

<!-- /SECTION: changelog -->
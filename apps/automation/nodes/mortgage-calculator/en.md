---
node_id: "mortgage-calculator"
title: "Mortgage Calculator"
description: "Calculate mortgage payments with extra payment support and amortization schedule"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - mortgage
  - loan
  - amortization
  - interest
  - finance
related_nodes:
  - loan-calculator
  - escrow-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Mortgage Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate mortgage payments, amortization details, payoff time, and savings from extra payments.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Mortgage Calculator** node calculates a standard monthly mortgage payment and simulates mortgage repayment over time.

It supports optional extra monthly payments and returns payoff duration, total interest, interest saved, time saved, and an amortization schedule for the first 12 months.

### Key Features

- Calculates the standard monthly mortgage payment.
- Supports optional extra monthly payments.
- Calculates total payoff time.
- Calculates payoff duration in years.
- Calculates total interest paid.
- Calculates estimated interest saved from extra payments.
- Calculates months saved compared with the original term.
- Returns the first 12 months of the amortization schedule.

### Processing Flow

```text
Principal
  ↓
Annual interest rate
  ↓
Loan term
  ↓
Calculate standard monthly payment
  ↓
Apply optional extra payment
  ↓
Simulate monthly repayment
  ↓
Calculate interest and time savings
  ↓
Build first 12 schedule entries
  ↓
Return result
```

### Use Cases

- Estimating monthly mortgage payments.
- Evaluating the impact of extra payments.
- Estimating mortgage payoff time.
- Comparing interest costs with and without extra payments.
- Reviewing early amortization details.
- Preparing mortgage calculations for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `principal` | `number` | No | `300000` | Mortgage principal. Must be greater than or equal to `0`. |
| `annualRate` | `number` | No | `4.5` | Annual interest rate as a percentage. The schema accepts values between `0` and `100`; use a value greater than `0` for the current mortgage payment calculation. |
| `years` | `number` | No | `30` | Mortgage term in years. Must be between `1` and `50`. |
| `extraPayment` | `number` | No | `0` | Additional amount applied to principal each month. Must be greater than or equal to `0`. |
| `frequency` | `string` | No | `monthly` | Configurable value: `monthly` or `yearly`. The current calculation uses monthly periods regardless of this value. |

### Principal

Provide the mortgage principal.

Example:

```text
300000
```

### Annual Rate

Provide the annual interest rate as a percentage.

Example:

```text
4.5
```

The node converts this to a monthly rate:

```text
monthlyRate = annualRate / 100 / 12
```

### Years

Provide the mortgage term in years.

Example:

```text
30
```

The total scheduled number of monthly payments is:

```text
numPayments = years × 12
```

### Extra Payment

Provide an optional extra amount to apply to principal each month.

Example:

```text
500
```

The extra payment does not change the calculated standard `monthlyPayment`. It is added during repayment simulation.

### Frequency

Supported configuration values are:

```text
monthly
yearly
```

The current implementation performs the calculation using monthly periods and does not use `frequency` to change the calculation.

### Monthly Payment Formula

The standard monthly payment is calculated as:

```text
monthlyPayment =
principal × (monthlyRate × (1 + monthlyRate)^numPayments)
/
((1 + monthlyRate)^numPayments - 1)
```

### Repayment Simulation

For each month:

```text
interest = balance × monthlyRate
principalPaid = monthlyPayment - interest + extraPayment
balance = balance - principalPaid
```

If the calculated principal payment is greater than the remaining balance, the node limits it to the remaining balance.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `principal`
- `annualRate`
- `years`
- `extraPayment`
- `frequency`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- standard monthly mortgage payment;
- extra monthly payment;
- total months required for payoff;
- payoff duration in years;
- total interest;
- estimated interest saved;
- months saved;
- first 12 amortization schedule entries.

Example with an extra payment:

```json
{
  "success": true,
  "monthlyPayment": 1520.0559294776574,
  "extraPayment": 500,
  "totalMonths": 218,
  "yearsToPayoff": "18.2",
  "totalInterest": 139306.30698645883,
  "interestSaved": 107913.82762549783,
  "timeSaved": 142,
  "schedule": [
    {
      "month": 1,
      "payment": 2020.0559294776574,
      "principal": 895.0559294776574,
      "interest": 1125,
      "balance": 299104.9440705223
    }
  ]
}
```

### Monthly Payment

`monthlyPayment` contains the standard mortgage payment before the optional extra payment is added.

For:

```text
principal: 300000
annualRate: 4.5
years: 30
```

the monthly payment is approximately:

```text
1520.0559294776574
```

### Total Months and Years to Payoff

`totalMonths` contains the actual number of simulated months required to repay the mortgage.

`yearsToPayoff` is returned as a string with one decimal place.

Example:

```text
totalMonths: 218
yearsToPayoff: "18.2"
```

### Interest Saved

The node compares simulated total interest with the total interest calculated for the original mortgage schedule without extra payments.

Small floating-point differences may appear when `extraPayment` is `0`.

### Schedule

The node stores only the first `12` monthly schedule entries, even when the mortgage lasts longer.

Each schedule entry contains:

| Field | Description |
|-------|-------------|
| `month` | Month number. |
| `payment` | Standard monthly payment plus configured extra payment. |
| `principal` | Principal paid during the month. |
| `interest` | Interest charged during the month. |
| `balance` | Remaining mortgage balance. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Standard Mortgage

**Configuration**

```text
principal: 300000
annualRate: 4.5
years: 30
extraPayment: 0
frequency: monthly
```

The calculated result includes:

```text
monthlyPayment: 1520.0559294776574
totalMonths: 360
yearsToPayoff: "30.0"
totalInterest: 247220.1346119449
timeSaved: 0
```

`interestSaved` may contain a very small value close to zero because of floating-point precision.

### Example 2: Mortgage With Extra Payment

**Configuration**

```text
principal: 300000
annualRate: 4.5
years: 30
extraPayment: 500
frequency: monthly
```

The calculated result includes:

```text
monthlyPayment: 1520.0559294776574
extraPayment: 500
totalMonths: 218
yearsToPayoff: "18.2"
totalInterest: 139306.30698645883
interestSaved: 107913.82762549783
timeSaved: 142
```

The first monthly schedule entry is:

```json
{
  "month": 1,
  "payment": 2020.0559294776574,
  "principal": 895.0559294776574,
  "interest": 1125,
  "balance": 299104.9440705223
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Mortgage Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Principal Is Invalid

**Cause:** `principal` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Annual Rate Is Invalid

**Cause:** `annualRate` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Annual Rate Is Zero

**Cause:** `annualRate` is `0`. The current monthly payment formula does not include a separate zero-interest calculation path.

**Solution:** Use an `annualRate` greater than `0` for the current implementation.

### Mortgage Term Is Invalid

**Cause:** `years` is outside the supported range.

**Solution:** Use a value between `1` and `50`.

### Extra Payment Is Invalid

**Cause:** `extraPayment` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Frequency Does Not Change the Result

This is expected in the current implementation.

Although `frequency` supports `monthly` and `yearly`, the mortgage calculation currently always uses monthly periods.

### Interest Saved Is Very Close to Zero

This can occur because of floating-point precision when `extraPayment` is `0`.

A very small value close to zero should be interpreted as no meaningful interest savings.

### Schedule Contains Only 12 Months

This is expected behavior.

The node simulates the complete mortgage payoff but stores only the first `12` months in the returned `schedule`.

### Unexpected Mortgage Result

**Cause:** One or more mortgage parameters may not match the intended calculation.

**Solution:** Verify `principal`, `annualRate`, `years`, and `extraPayment`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Loan Calculator** — Calculate loan payments and interest.
- **Escrow Account Calculator** — Calculate property tax and insurance escrow amounts.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial documentation for the Mortgage Calculator node. |

<!-- /SECTION: changelog -->
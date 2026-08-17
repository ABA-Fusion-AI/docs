---
node_id: "tip-calculator"
title: "Tip Calculator"
description: "Calculate tip, tax, and split bill amounts per person"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - tip
  - tax
  - bill
  - split
  - finance
related_nodes:
  - loan-calculator
  - mortgage-calculator
  - depreciation-calculator
---

<!-- SECTION: header -->
# Tip Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate tip, tax, total bill, and split amounts per person.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Tip Calculator** node calculates tax, tip, total bill amount, and the amount each person should pay.

It uses the configured bill amount, tip percentage, number of people, and optional tax percentage.

### Key Features

- Calculates tax from the bill amount.
- Calculates subtotal after tax.
- Calculates tip from the subtotal.
- Calculates the final total.
- Splits the total equally between multiple people.
- Returns a per-person amount.
- Returns a breakdown array containing one amount for each person.

### Processing Flow

```text
Bill amount
  ↓
Calculate tax
  ↓
Calculate subtotal
  ↓
Calculate tip
  ↓
Calculate total
  ↓
Split total between people
  ↓
Return result
```

### Use Cases

- Calculating restaurant tips.
- Adding tax to a bill.
- Splitting bills between multiple people.
- Calculating individual payment amounts.
- Preparing bill calculations for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `billAmount` | `number` | No | `100` | Base bill amount. Must be greater than or equal to `0`. |
| `tipPercent` | `number` | No | `15` | Tip percentage. Must be between `0` and `100`. |
| `numberOfPeople` | `number` | No | `1` | Number of people sharing the bill. Must be at least `1`. |
| `taxPercent` | `number` | No | `0` | Tax percentage applied to the bill amount. Must be between `0` and `100`. |

### Bill Amount

Provide the original bill amount.

Example:

```text
200
```

### Tip Percent

Provide the percentage used to calculate the tip.

Example:

```text
10
```

The tip is calculated from the subtotal after tax.

### Number of People

Provide the number of people sharing the final bill.

Example:

```text
4
```

The total amount is divided equally between all people.

### Tax Percent

Provide the tax percentage applied to the original bill amount.

Example:

```text
20
```

The tax is calculated as:

```text
billAmount × taxPercent / 100
```

### Calculation Order

The node calculates values in this order:

```text
tax = billAmount × taxPercent / 100
subtotal = billAmount + tax
tip = subtotal × tipPercent / 100
total = subtotal + tip
perPerson = total / numberOfPeople
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `billAmount`
- `tipPercent`
- `numberOfPeople`
- `taxPercent`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- original bill amount;
- tax amount;
- subtotal;
- tip percentage;
- tip amount;
- final total;
- number of people;
- per-person amount;
- breakdown array.

Example:

```json
{
  "success": true,
  "billAmount": 200,
  "tax": 40,
  "subtotal": 240,
  "tipPercent": 10,
  "tip": 24,
  "total": 264,
  "numberOfPeople": 4,
  "perPerson": 66,
  "breakdown": [
    66,
    66,
    66,
    66
  ]
}
```

### Breakdown

The `breakdown` field contains one per-person amount for each person.

For:

```text
numberOfPeople: 4
perPerson: 66
```

the breakdown is:

```json
[
  66,
  66,
  66,
  66
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic Tip Calculation

**Configuration**

```text
billAmount: 100
tipPercent: 15
numberOfPeople: 1
taxPercent: 0
```

**Output**

```json
{
  "success": true,
  "billAmount": 100,
  "tax": 0,
  "subtotal": 100,
  "tipPercent": 15,
  "tip": 15,
  "total": 115,
  "numberOfPeople": 1,
  "perPerson": 115,
  "breakdown": [
    115
  ]
}
```

### Example 2: Tax and Split Bill

**Configuration**

```text
billAmount: 200
tipPercent: 10
numberOfPeople: 4
taxPercent: 20
```

**Output**

```json
{
  "success": true,
  "billAmount": 200,
  "tax": 40,
  "subtotal": 240,
  "tipPercent": 10,
  "tip": 24,
  "total": 264,
  "numberOfPeople": 4,
  "perPerson": 66,
  "breakdown": [
    66,
    66,
    66,
    66
  ]
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Tip Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Bill Amount Is Invalid

**Cause:** `billAmount` is below `0`.

**Solution:** Use a bill amount greater than or equal to `0`.

### Tip Percentage Is Invalid

**Cause:** `tipPercent` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Tax Percentage Is Invalid

**Cause:** `taxPercent` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Number of People Is Invalid

**Cause:** `numberOfPeople` is below `1`.

**Solution:** Use at least `1` person.

### Unexpected Per-Person Amount

**Cause:** One or more bill parameters may not match the intended calculation.

**Solution:** Verify `billAmount`, `tipPercent`, `taxPercent`, and `numberOfPeople`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Loan Calculator** — Calculate loan payments and interest.
- **Mortgage Calculator** — Calculate mortgage payment information.
- **Depreciation Calculator** — Calculate asset depreciation schedules.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Tip Calculator node. |

<!-- /SECTION: changelog -->
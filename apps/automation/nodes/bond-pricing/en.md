---
node_id: "bond-pricing"
title: "Bond Pricing Calculator"
description: "Calculate bond price, duration, and premium/discount analysis"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - bond
  - pricing
  - finance
  - duration
  - premium-discount
related_nodes:
  - loan-calculator
  - mortgage-calculator
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Bond Pricing Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate bond price, Macaulay duration, and premium or discount relative to face value.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Bond Pricing Calculator** node calculates the present value of a bond using its face value, coupon rate, years to maturity, yield rate, and payment frequency.

It also calculates Macaulay duration and indicates whether the bond is trading at a premium or discount relative to its face value.

### Key Features

- Calculates bond price from discounted coupon payments and face value.
- Calculates present value of coupon payments.
- Calculates present value of face value.
- Calculates Macaulay duration.
- Calculates premium or discount amount.
- Calculates premium or discount percentage.
- Supports configurable payment frequency.
- Returns structured financial results.

### Processing Flow

```text
Face value
  ↓
Coupon rate
  ↓
Years to maturity
  ↓
Yield rate
  ↓
Payment frequency
  ↓
Calculate coupon payments
  ↓
Discount cash flows
  ↓
Calculate bond price and duration
  ↓
Calculate premium or discount
  ↓
Return result
```

### Use Cases

- Calculating bond fair value.
- Comparing market yield with coupon rate.
- Identifying premium or discount bonds.
- Estimating bond duration.
- Evaluating fixed-income investments.
- Preparing bond metrics for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `faceValue` | `number` | No | `1000` | Bond face value. The schema accepts values greater than or equal to `0`; use a value greater than `0` for meaningful premium percentage calculations. |
| `couponRate` | `number` | No | `5` | Annual coupon rate as a percentage. Must be between `0` and `100`. |
| `yearsToMaturity` | `number` | No | `10` | Number of years until maturity. Must be between `0` and `100`. |
| `yieldRate` | `number` | No | `5` | Annual yield rate as a percentage. Must be between `0` and `100`. |
| `frequency` | `number` | No | `2` | Number of coupon payments per year. Must be between `1` and `12`. |

### Face Value

Provide the bond face value.

Example:

```text
1000
```

### Coupon Rate

Provide the annual coupon rate as a percentage.

Example:

```text
5
```

### Years to Maturity

Provide the number of years remaining until maturity.

Example:

```text
10
```

### Yield Rate

Provide the annual yield rate as a percentage.

Example:

```text
5
```

### Frequency

Provide the number of coupon payments per year.

Example:

```text
2
```

For `2`, the node uses semiannual coupon periods.

### Calculation Details

Number of periods:

```text
periods = yearsToMaturity × frequency
```

Coupon payment per period:

```text
couponPayment = faceValue × couponRate / 100 / frequency
```

Yield per period:

```text
yieldPerPeriod = yieldRate / 100 / frequency
```

Bond price:

```text
price = present value of coupon payments + present value of face value
```

Premium or discount:

```text
premium = price - faceValue
```

Premium or discount percentage:

```text
premiumPercent = (price - faceValue) / faceValue × 100
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `faceValue`
- `couponRate`
- `yearsToMaturity`
- `yieldRate`
- `frequency`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- calculated bond price;
- face value;
- coupon rate;
- yield rate;
- Macaulay duration;
- premium or discount amount;
- premium or discount percentage.

Example:

```json
{
  "success": true,
  "price": 1000.0000000000014,
  "faceValue": 1000,
  "couponRate": 5,
  "yieldRate": 5,
  "duration": 7.989445671393992,
  "premium": 1.3642420526593924e-12,
  "premiumPercent": 1.3642420526593923e-13
}
```

### Price

When `couponRate` and `yieldRate` are equal, the bond price is approximately equal to face value.

Small floating-point differences may appear in the result.

### Premium and Discount

A positive `premium` means the calculated bond price is above face value.

A negative `premium` means the calculated bond price is below face value.

A value near `0` means the bond is approximately priced at par.

### Duration

The `duration` field contains the Macaulay duration expressed in years.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Bond Priced at Par

**Configuration**

```text
faceValue: 1000
couponRate: 5
yearsToMaturity: 10
yieldRate: 5
frequency: 2
```

**Output**

```json
{
  "success": true,
  "price": 1000.0000000000014,
  "faceValue": 1000,
  "couponRate": 5,
  "yieldRate": 5,
  "duration": 7.989445671393992,
  "premium": 1.3642420526593924e-12,
  "premiumPercent": 1.3642420526593923e-13
}
```

The very small premium value is caused by floating-point precision and is effectively zero.

### Example 2: Premium Bond

**Configuration**

```text
faceValue: 1000
couponRate: 7
yearsToMaturity: 10
yieldRate: 5
frequency: 2
```

The calculated result includes:

```text
price: 1155.8916228564697
premium: 155.8916228564697
premiumPercent: 15.589162285646967
```

The bond is priced above face value because the coupon rate is greater than the yield rate.

### Example 3: Discount Bond

**Configuration**

```text
faceValue: 1000
couponRate: 3
yearsToMaturity: 10
yieldRate: 5
frequency: 2
```

The calculated result includes:

```text
price: 844.1083771435332
premium: -155.89162285646682
premiumPercent: -15.589162285646681
```

The bond is priced below face value because the coupon rate is lower than the yield rate.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Bond Pricing Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Face Value Is Invalid

**Cause:** `faceValue` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Coupon Rate Is Invalid

**Cause:** `couponRate` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Yield Rate Is Invalid

**Cause:** `yieldRate` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Years to Maturity Is Invalid

**Cause:** `yearsToMaturity` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Frequency Is Invalid

**Cause:** `frequency` is outside the supported range.

**Solution:** Use a value between `1` and `12`.

### Face Value Is Zero

**Cause:** `faceValue` is `0`, which prevents a meaningful premium percentage from being calculated.

**Solution:** Use a `faceValue` greater than `0` when premium percentage analysis is required.

### Years to Maturity Is Zero

**Cause:** `yearsToMaturity` is `0`, so the bond has no remaining coupon periods.

**Solution:** Use a value greater than `0` when calculating a standard bond price and duration over future periods.

### Premium Is Very Close to Zero

This can occur because of floating-point precision.

When the coupon rate and yield rate are equal, the calculated bond price should be approximately equal to face value.

### Unexpected Bond Price

**Cause:** One or more bond parameters may not match the intended calculation.

**Solution:** Verify `faceValue`, `couponRate`, `yearsToMaturity`, `yieldRate`, and `frequency`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Loan Calculator** — Calculate loan payments and interest.
- **Mortgage Calculator** — Calculate mortgage payment information.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial documentation for the Bond Pricing Calculator node. |

<!-- /SECTION: changelog -->
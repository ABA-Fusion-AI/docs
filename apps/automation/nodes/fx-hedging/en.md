---
node_id: "fx-hedging"
title: "Foreign Exchange Hedging"
description: "Calculate FX hedging costs and analyze hedge effectiveness"
category: "business-commerce"
subcategory: "finance-accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - fx
  - hedging
  - foreign-exchange
  - forward-rate
  - finance
related_nodes:
  - currency-converter
  - lease-vs-buy
  - debt-consolidation
---

<!-- SECTION: header -->
# Foreign Exchange Hedging

> **Category:** Business & Commerce | **Type:** Action Node

Calculate FX hedging costs and analyze hedge effectiveness.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Foreign Exchange Hedging** node calculates the financial effect of hedging a foreign exchange exposure using the exposure amount, spot rate, forward rate, and hedge ratio.

It calculates the hedged amount, unhedged value, hedged value, hedge cost, hedge cost percentage, and a recommendation based on the calculated hedge cost.

### Key Features

- Calculates the amount of exposure covered by the hedge.
- Calculates the unhedged exposure value.
- Calculates the hedged exposure value.
- Calculates the hedge cost.
- Calculates the hedge cost percentage.
- Supports full and partial hedging.
- Returns a recommendation based on the hedge cost.
- Returns structured FX hedging results.

### Processing Flow

```text
Exposure
  ↓
Spot rate
  ↓
Forward rate
  ↓
Hedge ratio
  ↓
Calculate hedge amount
  ↓
Calculate unhedged value
  ↓
Calculate hedged value
  ↓
Calculate hedge cost
  ↓
Calculate cost percentage
  ↓
Return recommendation
```

### Use Cases

- Estimating FX hedging costs.
- Comparing hedged and unhedged exposure values.
- Evaluating full or partial currency hedging.
- Analyzing the effect of spot and forward rates.
- Preparing FX hedging metrics for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `exposure` | `number` | No | `100000` | Foreign exchange exposure amount. Must be greater than or equal to `0`. |
| `spotRate` | `number` | No | `1.0` | Current spot exchange rate. Must be greater than or equal to `0`. |
| `forwardRate` | `number` | No | `1.02` | Forward exchange rate used for the hedged portion. Must be greater than or equal to `0`. |
| `hedgeRatio` | `number` | No | `100` | Percentage of the exposure to hedge. Must be between `0` and `100`. |

### Exposure

Provide the foreign exchange exposure amount.

Example:

```text
100000
```

### Spot Rate

Provide the current spot exchange rate.

Example:

```text
1
```

### Forward Rate

Provide the forward exchange rate used for the hedged portion.

Example:

```text
1.02
```

### Hedge Ratio

Provide the percentage of the exposure to hedge.

Example:

```text
50
```

A value of `100` hedges the full exposure, while a value of `50` hedges half of the exposure.

### Calculation Details

Hedge amount:

```text
hedgeAmount = exposure × hedgeRatio / 100
```

Unhedged value:

```text
unhedgedValue = exposure × spotRate
```

Hedged value:

```text
hedgedValue = hedgeAmount × forwardRate + (exposure - hedgeAmount) × spotRate
```

Hedge cost:

```text
hedgeCost = hedgedValue - unhedgedValue
```

Hedge cost percentage:

```text
costPercent = hedgeCost / unhedgedValue × 100
```

The recommendation is based on the hedge cost:

```text
hedgeCost < 0
  → Hedge protects against adverse moves

hedgeCost >= 0
  → Hedge costs premium
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `exposure`
- `spotRate`
- `forwardRate`
- `hedgeRatio`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- exposure;
- spot rate;
- forward rate;
- hedge ratio;
- hedge amount;
- unhedged value;
- hedged value;
- hedge cost;
- hedge cost percentage;
- recommendation.

Example:

```json
{
  "success": true,
  "exposure": 100000,
  "spotRate": 1,
  "forwardRate": 0.95,
  "hedgeRatio": 50,
  "hedgeAmount": 50000,
  "unhedgedValue": 100000,
  "hedgedValue": 97500,
  "hedgeCost": -2500,
  "costPercent": -2.5,
  "recommendation": "Hedge protects against adverse moves"
}
```

### Hedge Amount

For:

```text
exposure: 100000
hedgeRatio: 50
```

the hedge amount is:

```text
50000
```

### Hedge Cost

A positive `hedgeCost` means the calculated hedged value is greater than the unhedged value.

A negative `hedgeCost` means the calculated hedged value is lower than the unhedged value.

### Cost Percentage

For:

```text
hedgeCost: -2500
unhedgedValue: 100000
```

the cost percentage is:

```text
-2.5
```

### Recommendation

When `hedgeCost` is negative, the node returns:

```text
Hedge protects against adverse moves
```

Otherwise, it returns:

```text
Hedge costs premium
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Full Hedge

**Configuration**

```text
exposure: 100000
spotRate: 1
forwardRate: 1.02
hedgeRatio: 100
```

**Output**

```json
{
  "success": true,
  "exposure": 100000,
  "spotRate": 1,
  "forwardRate": 1.02,
  "hedgeRatio": 100,
  "hedgeAmount": 100000,
  "unhedgedValue": 100000,
  "hedgedValue": 102000,
  "hedgeCost": 2000,
  "costPercent": 2,
  "recommendation": "Hedge costs premium"
}
```

### Example 2: Partial Hedge

**Configuration**

```text
exposure: 100000
spotRate: 1
forwardRate: 0.95
hedgeRatio: 50
```

**Output**

```json
{
  "success": true,
  "exposure": 100000,
  "spotRate": 1,
  "forwardRate": 0.95,
  "hedgeRatio": 50,
  "hedgeAmount": 50000,
  "unhedgedValue": 100000,
  "hedgedValue": 97500,
  "hedgeCost": -2500,
  "costPercent": -2.5,
  "recommendation": "Hedge protects against adverse moves"
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: FX Hedging Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Exposure Is Invalid

**Cause:** `exposure` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Spot Rate Is Invalid

**Cause:** `spotRate` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Forward Rate Is Invalid

**Cause:** `forwardRate` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Hedge Ratio Is Invalid

**Cause:** `hedgeRatio` is outside the supported range.

**Solution:** Use a value between `0` and `100`.

### Cost Percentage Is Invalid

**Cause:** `unhedgedValue` is `0`. This occurs when `exposure` or `spotRate` is `0`.

The current implementation calculates:

```text
costPercent = hedgeCost / unhedgedValue × 100
```

**Solution:** Use values greater than `0` for `exposure` and `spotRate` when a meaningful cost percentage is required.

### Unexpected Hedge Result

**Cause:** One or more configured FX values may not match the intended calculation.

**Solution:** Verify `exposure`, `spotRate`, `forwardRate`, and `hedgeRatio`.

### Unexpected Recommendation

The recommendation is determined only by the sign of `hedgeCost`.

When `hedgeCost` is below `0`, the node returns `Hedge protects against adverse moves`. Otherwise, it returns `Hedge costs premium`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Currency Converter** — Convert amounts between currencies.
- **Lease vs Buy Calculator** — Compare leasing and purchasing costs.
- **Debt Consolidation Calculator** — Compare current debt payments with a consolidated loan.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Foreign Exchange Hedging node. |

<!-- /SECTION: changelog -->
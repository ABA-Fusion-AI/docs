---
node_id: "stock-automation"
title: "Stock Automation"
description: "Stock automation calculations including Reorder Point, Min-Max inventory system, EOQ (Economic Order Quantity - Wilson), Periodic Review system, and ABC classification."
category: "Supply Chain / Inventory"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- inventory
- supply-chain
- stock-management
- reorder-point
- min-max
- eoq
- economic-order-quantity
- periodic-review
- abc-analysis
- wilson-formula

related_nodes:
- function
- if

---

# Stock Automation

> **Category:** supply-chain-nodes | **Type:** Action Node

Performs standard **inventory management calculations**: Reorder Point, Min-Max system, EOQ (Wilson formula), Periodic Review system, and ABC classification.

The **Stock Automation** node exposes five distinct inventory-planning calculations via an `operation` selector, each with its own conditional set of input parameters.

### Supported Features

- Reorder Point calculation with optional safety stock
- Min-Max inventory system with order quantity recommendation
- EOQ (Economic Order Quantity) using the Wilson formula
- Periodic Review order-up-to calculation
- ABC classification (Pareto analysis) across a list of items
- Conditional parameter display based on the selected `operation`
- Input validation specific to each calculation type

### Use Cases

- Determine when to reorder a SKU based on demand and lead time
- Decide how much to order under a min-max inventory policy
- Calculate the cost-optimal order quantity for a purchased item
- Plan order-up-to quantities under a fixed periodic review cycle
- Classify an inventory catalog into A/B/C categories by annual value for prioritized management
- Feed reorder recommendations into a purchasing or notification workflow

---

## Configuration

### Base Parameter

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"reorderPoint"` | Calculation type: `reorderPoint`, `minMax`, `eoq`, `periodicReview`, or `abc`. |

### Reorder Point Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `dailyDemand` | `number` | ✅ Yes | — | Daily demand in units. |
| `leadTimeDays` | `number` | ✅ Yes | — | Lead time in days. |
| `safetyStock` | `number` | ❌ No | `0` | Safety stock buffer. |

### Min-Max Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `min` | `number` | ✅ Yes | — | Minimum stock level (reorder trigger). |
| `max` | `number` | ✅ Yes | — | Maximum stock level (order-up-to target). |
| `currentStock` | `number` | ✅ Yes | — | Current stock level. |

### EOQ Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `annualDemand` | `number` | ✅ Yes | — | Annual demand in units. |
| `orderCost` | `number` | ✅ Yes | — | Fixed cost per order. |
| `holdingCost` | `number` | ✅ Yes | — | Holding cost per unit per year. Cannot be `0`. |

### Periodic Review Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `dailyDemandReview` | `number` | ✅ Yes | — | Daily demand in units. |
| `reviewPeriodDays` | `number` | ✅ Yes | — | Length of the review cycle, in days. |
| `leadTimeDaysReview` | `number` | ✅ Yes | — | Lead time in days. |
| `safetyStockReview` | `number` | ❌ No | `0` | Safety stock buffer. |
| `currentStockReview` | `number` | ✅ Yes | — | Current stock level. |

### ABC Analysis Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `items` | `array` | ✅ Yes (min. 1 item) | — | Array of `{ sku, annualUsage, unitCost }` objects. |
| `aThreshold` | `number` | ❌ No | `80` | Cumulative value % threshold for Category A (0–100). |
| `bThreshold` | `number` | ❌ No | `95` | Cumulative value % threshold for Category B (0–100). Must be ≥ `aThreshold`. |

---

## Calculation Types

### Reorder Point (`operation: "reorderPoint"`)

```text
demandDuringLead = dailyDemand * leadTimeDays
reorderPoint = demandDuringLead + safetyStock
```

Simple lead-time-demand-plus-buffer formula. No validation beyond the schema's non-negative constraints.

### Min-Max (`operation: "minMax"`)

```text
shouldOrder = currentStock <= min
orderQty = shouldOrder ? max(0, max - currentStock) : 0
```

**Validation:** `max` must be greater than or equal to `min`.

### EOQ / Wilson Formula (`operation: "eoq"`)

```text
eoq = sqrt((2 * annualDemand * orderCost) / holdingCost)
ordersPerYear = annualDemand / eoq
```

**Validation:** `holdingCost` cannot be `0` (division by zero).

### Periodic Review (`operation: "periodicReview"`)

```text
coverageDays = reviewPeriodDays + leadTimeDaysReview
expectedDemand = dailyDemandReview * coverageDays
orderUpTo = expectedDemand + safetyStockReview
orderQty = max(0, orderUpTo - currentStockReview)
```

No additional validation beyond the schema's non-negative constraints.

### ABC Analysis (`operation: "abc"`)

Classic Pareto (80/20-style) inventory classification:

1. Compute `annualValue = annualUsage * unitCost` for each item.
2. Sum all `annualValue`s into `totalValue`.
3. **If `totalValue` is 0** — all items are returned with `sharePct: 0`, `cumulativePct: 0`, and classified as `"C"`, without sorting.
4. Otherwise, sort items by `annualValue` **descending**.
5. Walk the sorted list accumulating `cumulativePct`; classify each item as:
   - `"A"` while `cumulativePct <= aThreshold`
   - `"B"` while `cumulativePct <= bThreshold`
   - `"C"` beyond that
6. Return the classified list plus a `summary` count per category.

**Validation:** `bThreshold` must be greater than or equal to `aThreshold`.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

All configuration is provided through the node configuration.

### Outputs — Reorder Point

| Output | Type | Description |
| ------ | ---- | ----------- |
| `reorderPoint` | `number` | The stock level at which to reorder, rounded to 2 decimals. |
| `demandDuringLead` | `number` | Expected demand consumed during the lead time. |
| `safetyStock` | `number` | Safety stock used in the calculation. |
| `unit` | `string` | Always `"units"`. |

### Outputs — Min-Max

| Output | Type | Description |
| ------ | ---- | ----------- |
| `shouldOrder` | `boolean` | Whether current stock is at or below `min`. |
| `orderQty` | `number` | Recommended order quantity (0 if no order needed). |
| `min` | `number` | Configured minimum. |
| `max` | `number` | Configured maximum. |
| `currentStock` | `number` | Configured current stock. |
| `unit` | `string` | Always `"units"`. |

### Outputs — EOQ

| Output | Type | Description |
| ------ | ---- | ----------- |
| `eoq` | `number` | The economic order quantity, rounded to 2 decimals. |
| `ordersPerYear` | `number` | Implied number of orders per year at the EOQ. |
| `unit` | `string` | Always `"units"`. |

### Outputs — Periodic Review

| Output | Type | Description |
| ------ | ---- | ----------- |
| `orderQty` | `number` | Recommended order quantity. |
| `orderUpTo` | `number` | Target order-up-to level. |
| `expectedDemand` | `number` | Expected demand over the coverage period. |
| `safetyStock` | `number` | Safety stock used. |
| `coverageDays` | `number` | Total days covered (review period + lead time). |
| `unit` | `string` | Always `"units"`. |

### Outputs — ABC Analysis

| Output | Type | Description |
| ------ | ---- | ----------- |
| `totalValue` | `number` | Sum of all items' annual value. |
| `aThreshold` | `number` | Threshold used for Category A. |
| `bThreshold` | `number` | Threshold used for Category B. |
| `classified` | `array` | Each item with `sku`, `annualUsage`, `unitCost`, `annualValue`, `sharePct`, `cumulativePct`, `class` (`"A"`/`"B"`/`"C"`). |
| `summary` | `object` | `{ categoryA, categoryB, categoryC, totalItems }` counts. |

---

## Output Example

### Reorder Point

```json
{
  "reorderPoint": 320,
  "demandDuringLead": 300,
  "safetyStock": 20,
  "unit": "units"
}
```

### Min-Max

```json
{
  "shouldOrder": true,
  "orderQty": 150,
  "min": 100,
  "max": 300,
  "currentStock": 90,
  "unit": "units"
}
```

### EOQ

```json
{
  "eoq": 774.6,
  "ordersPerYear": 12.92,
  "unit": "units"
}
```

### Periodic Review

```json
{
  "orderQty": 210,
  "orderUpTo": 460,
  "expectedDemand": 440,
  "safetyStock": 20,
  "coverageDays": 22,
  "unit": "units"
}
```

### ABC Analysis

```json
{
  "totalValue": 125000,
  "aThreshold": 80,
  "bThreshold": 95,
  "classified": [
    {
      "sku": "SKU-001",
      "annualUsage": 500,
      "unitCost": 150,
      "annualValue": 75000,
      "sharePct": 60,
      "cumulativePct": 60,
      "class": "A"
    },
    {
      "sku": "SKU-002",
      "annualUsage": 1000,
      "unitCost": 25,
      "annualValue": 25000,
      "sharePct": 20,
      "cumulativePct": 80,
      "class": "A"
    },
    {
      "sku": "SKU-003",
      "annualUsage": 2500,
      "unitCost": 10,
      "annualValue": 25000,
      "sharePct": 20,
      "cumulativePct": 100,
      "class": "C"
    }
  ],
  "summary": {
    "categoryA": 2,
    "categoryB": 0,
    "categoryC": 1,
    "totalItems": 3
  }
}
```

---

## Configuration Examples

### Reorder Point

```json
{
  "operation": "reorderPoint",
  "dailyDemand": 20,
  "leadTimeDays": 15,
  "safetyStock": 20
}
```

### Min-Max

```json
{
  "operation": "minMax",
  "min": 100,
  "max": 300,
  "currentStock": 90
}
```

### EOQ

```json
{
  "operation": "eoq",
  "annualDemand": 10000,
  "orderCost": 50,
  "holdingCost": 4.17
}
```

### Periodic Review

```json
{
  "operation": "periodicReview",
  "dailyDemandReview": 20,
  "reviewPeriodDays": 7,
  "leadTimeDaysReview": 15,
  "safetyStockReview": 20,
  "currentStockReview": 250
}
```

### ABC Analysis

```json
{
  "operation": "abc",
  "items": [
    { "sku": "SKU-001", "annualUsage": 500, "unitCost": 150 },
    { "sku": "SKU-002", "annualUsage": 1000, "unitCost": 25 },
    { "sku": "SKU-003", "annualUsage": 2500, "unitCost": 10 }
  ],
  "aThreshold": 80,
  "bThreshold": 95
}
```

---

## Workflow Integration

### Sample Workflow: Reorder Check → If → Purchase Order

```json
{
  "nodes": [
    {
      "id": "min-max-check",
      "type": "stock-automation",
      "config": {
        "operation": "minMax",
        "min": 100,
        "max": 300,
        "currentStock": 90
      }
    },
    {
      "id": "check-should-order",
      "type": "if"
    },
    {
      "id": "create-purchase-order",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: EOQ Planning → Database

```json
{
  "nodes": [
    {
      "id": "eoq-calc",
      "type": "stock-automation",
      "config": {
        "operation": "eoq",
        "annualDemand": 10000,
        "orderCost": 50,
        "holdingCost": 4.17
      }
    },
    {
      "id": "store-eoq",
      "type": "database"
    }
  ]
}
```

### Sample Workflow: ABC Analysis → Function → Report

```json
{
  "nodes": [
    {
      "id": "abc-analysis",
      "type": "stock-automation",
      "config": {
        "operation": "abc",
        "items": []
      }
    },
    {
      "id": "build-priority-report",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- Schedule (daily) → Stock Automation (`minMax`) → If → Notification/Purchase Order — automated reorder alerts
- Database (fetch item catalog) → Function (map to `items`) → Stock Automation (`abc`) — periodic inventory classification
- Stock Automation (`reorderPoint`) → Function → Database — maintain reorder point reference table

---

## Error Handling

### Min-Max: Invalid Range

```text
'max' must be >= 'min'
```

Raised when `max` is less than `min`.

### EOQ: Zero Holding Cost

```text
'holdingCost' cannot be 0
```

Raised when `holdingCost` is `0` (would cause division by zero).

### ABC: Invalid Threshold Order

```text
'bThreshold' must be >= 'aThreshold'
```

Raised when `bThreshold` is less than `aThreshold`.

### Unknown Operation

```text
Unknown operation: <operation>
```

Should not normally occur given the `operation` enum.

---

## Troubleshooting

### "'max' must be >= 'min'"

**Cause**

The configured `max` is smaller than `min` for a `minMax` calculation — an invalid inventory policy.

**Solution**

Set `max` to a value greater than or equal to `min`.

---

### "'holdingCost' cannot be 0"

**Cause**

`holdingCost` was set to `0` for an `eoq` calculation, which would produce a division by zero in the Wilson formula.

**Solution**

Provide a realistic non-zero holding cost per unit per year (e.g. a percentage of unit cost representing storage, insurance, and capital costs).

---

### "'bThreshold' must be >= 'aThreshold'"

**Cause**

`bThreshold` (e.g. `70`) was set lower than `aThreshold` (e.g. `80`) for an `abc` calculation, which would make Category B unreachable or produce inconsistent classification.

**Solution**

Ensure `bThreshold >= aThreshold` — typically `aThreshold` around 70–80% and `bThreshold` around 90–95%.

---

### All ABC Items Classified as "C"

**Cause**

`totalValue` across all `items` is `0` — likely because `annualUsage` or `unitCost` is `0` for every item — which short-circuits the calculation and returns all items unsorted as Category C.

**Solution**

Verify `annualUsage` and `unitCost` are populated with real, non-zero values for the items being analyzed.

---

### EOQ Result Looks Unrealistically Large or Small

**Cause**

`orderCost` or `holdingCost` may not reflect realistic figures — the Wilson formula is highly sensitive to the ratio between these two inputs.

**Solution**

Double check that `orderCost` reflects the true fixed cost per purchase order and `holdingCost` reflects the true annual carrying cost per unit (not, for example, a total holding cost across all units).

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally from the provided configuration values.

No API key or authentication credential is required.

---

## Notes

Each calculation type returns a self-contained result object — there is no unified output schema across operations, since each answers a different inventory-planning question.

The node does not:

- Track live inventory state between calls (all inputs must be supplied fresh each call)
- Support alternative safety-stock formulas (e.g. based on demand variability and service level) — `safetyStock` is a direct input, not computed
- Support multi-echelon or multi-location inventory calculations
- Round `items` inputs for `abc` — only the computed output fields are rounded
- Persist or store ABC classification results

---


## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release |
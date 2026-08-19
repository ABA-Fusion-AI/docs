---
node_id: "mrp-engine"
title: "MRP Engine"
description: "Material Requirements Planning (MRP) engine that calculates net requirements, offsets by lead times, and generates planned orders through recursive BOM explosion."
category: "business-operations"
subcategory: "supply-chain-inventory"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - mrp
  - inventory
  - bom
  - bill-of-materials
  - supply-chain
  - planned-orders
  - lead-time
  - netting
  - explosion
  - manufacturing
  - purchasing
related_nodes:
  - bom-utils
  - inventory
  - production-metrics
  - inventory-logistics
  - log
---

<!-- SECTION: header -->
# MRP Engine

> **Category:** Business Operations | **Subcategory:** Supply Chain & Inventory | **Type:** Action Node

Calculate net material requirements, perform lead-time offsetting, and generate planned manufacturing and purchasing orders through recursive Bill of Materials (BOM) explosion.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **MRP Engine** node executes core Material Requirements Planning (MRP) algorithms for manufacturing, assembly, and inventory supply chain workflows.

Given a master **Bill of Materials (BOM)**, an **Inventory Master** (on-hand stock and lead times), and independent **Demand Requirements** (order quantities and due dates), the engine recursively calculates:
1. **Net Requirements (Netting):** Subtracting available stock on hand from gross demand.
2. **Lead-Time Offsetting:** Determining exact order release dates by subtracting lead time from due dates.
3. **BOM Explosion:** Recursively exploding parent orders into required child component demands down the entire assembly tree.
4. **Order Categorization:** Automatically distinguishing between manufactured sub-assemblies and purchased raw materials.

### Key Features

- **Automated Netting:** Consumes existing stock on hand (`onHand`) before triggering new orders, preventing over-purchasing and excess inventory accumulation.
- **Lead-Time Offsetting:** Calculates exact release (start) dates based on item-specific lead times (`Release Date = Due Date - Lead Time`).
- **Recursive Multi-Level BOM Explosion:** Automatically calculates child component requirements and sets their due date equal to the parent order's release date.
- **Order Classification:** Categorizes planned orders as either `"Manufacture"` (items defined as parents in the BOM) or `"Purchase"` (raw component items without BOM children).
- **Impossible Lead Time Alerting:** Detects schedule infeasibility when an item's lead time requires a negative release date (`Release Date < 0`), populating critical `actionMessages`.
- **Order Aggregation & Summary Metrics:** Groups planned orders chronologically by release date and computes comprehensive manufacturing vs. purchasing statistical summaries.

### Processing Flow

```text
Input Master Configuration (demands, bom, inventory)
                        ↓
            For Each Demand Item
                        ↓
             Netting (Net Qty Calculation)
             Net Qty = Gross Qty - On-Hand Stock
                        ↓
          ┌───────────────────────────┐
          ↓                           ↓
   Net Qty ≤ 0                 Net Qty > 0
(Stock Sufficient)         (Stock Deficit)
   • Consume stock            • Deplete on-hand stock to 0
   • Stop explosion           • Compute Release Date = Due Day - Lead Time
   • No order generated       • Determine Type ("Manufacture" if BOM exists, else "Purchase")
                              • Create Planned Order entry
                              • Check if Release Date < 0 ──→ Trigger CRITICAL Alert
                                      ↓
                              BOM Component Explosion
                              For each child component in BOM:
                                Required Qty = Parent Order Qty × Component Qty
                                Component Due Date = Parent Order Release Date
                                └─→ Recurse process() for child component
                        ↓
Sort Planned Orders Ascending by Release Date & Group by Release Date
                        ↓
Return Structured Output Payload (plannedOrders, ordersByDate, summary, actionMessages, hasAlerts)
```

### Use Cases

- **Manufacturing Production Scheduling:** Calculate when work orders must start on the factory floor so finished goods arrive on schedule.
- **Procurement & Purchasing Planning:** Determine when purchase orders for raw materials must be issued to suppliers based on lead times.
- **Inventory Optimization:** Minimize safety stock and holding costs by ordering components only when needed (*Just-In-Time*).
- **Supply Chain Bottleneck Analysis:** Identify infeasible delivery promises where lead times exceed requested customer delivery dates.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `bom` | `object` | ✅ Yes | `{}` | Bill of Materials dictionary mapping parent item IDs to arrays of child component requirements (`childId`, `qty`). |
| `inventory` | `object` | ✅ Yes | `{}` | Master inventory dictionary mapping item IDs to stock on hand (`onHand`) and lead time in days (`leadTime`). |
| `demands` | `array` | ✅ Yes | `[]` | Array of independent demand requirements (`itemId`, `grossQty`, `dueDay`). Minimum 1 demand item. |

---

### Parameter Details

#### `bom` (Bill of Materials)
A dictionary mapping parent item IDs to an array of child component specifications.
- **Type:** `Record<string, Array<{ childId: string, qty: number }>>`
- **Required:** Yes
- **Child Object Schema:**
  - `childId` (`string`): Unique identifier of the child component item.
  - `qty` (`number`): Units of child component required to produce 1 unit of the parent item (must be `>= 0`).
- **Example:**
```json
{
  "Bicycle": [
    { "childId": "Wheel", "qty": 2 },
    { "childId": "Frame", "qty": 1 }
  ],
  "Wheel": [
    { "childId": "Spoke", "qty": 36 },
    { "childId": "Tire", "qty": 1 }
  ]
}
```

---

#### `inventory` (Inventory Master)
A dictionary defining stock availability and fulfillment lead times for all items.
- **Type:** `Record<string, { onHand: number, leadTime: number }>`
- **Required:** Yes
- **Item Object Schema:**
  - `onHand` (`number`): Current usable inventory stock level (must be `>= 0`).
  - `leadTime` (`number`): Number of days required to manufacture or procure the item (must be `>= 0`).
- **Example:**
```json
{
  "Bicycle": { "onHand": 0, "leadTime": 2 },
  "Wheel": { "onHand": 5, "leadTime": 3 },
  "Frame": { "onHand": 2, "leadTime": 5 },
  "Spoke": { "onHand": 100, "leadTime": 7 },
  "Tire": { "onHand": 10, "leadTime": 4 }
}
```

---

#### `demands` (Demand Requirements)
An array of independent master schedule demand requirements to be fulfilled.
- **Type:** `Array<{ itemId: string, grossQty: number, dueDay: number }>`
- **Required:** Yes (at least 1 demand)
- **Demand Object Schema:**
  - `itemId` (`string`): ID of the top-level finished item to produce.
  - `grossQty` (`number`): Total quantity demanded (must be `>= 0`).
  - `dueDay` (`number`): Target day number when the finished item must be completed (must be `>= 0`).
- **Example:**
```json
[
  {
    "itemId": "Bicycle",
    "grossQty": 10,
    "dueDay": 20
  }
]
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload. Master parameters (`bom`, `inventory`, `demands`) can be dynamically bound using expressions. |

---

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted on successful execution. Contains planned orders array, order breakdown by date, summary statistics, and alert messages. |
| `error` | `Error` | Emitted if parameter validation fails (e.g. empty demands array or negative quantities). |

---

### Output Data Structure Example

```json
{
  "plannedOrders": [
    {
      "item": "Frame",
      "type": "Purchase",
      "qty": 8,
      "releaseDate": 13,
      "dueDate": 18
    },
    {
      "item": "Wheel",
      "type": "Purchase",
      "qty": 15,
      "releaseDate": 15,
      "dueDate": 18
    },
    {
      "item": "Bicycle",
      "type": "Manufacture",
      "qty": 10,
      "releaseDate": 18,
      "dueDate": 20
    }
  ],
  "ordersByDate": {
    "13": [
      { "item": "Frame", "type": "Purchase", "qty": 8, "releaseDate": 13, "dueDate": 18 }
    ],
    "15": [
      { "item": "Wheel", "type": "Purchase", "qty": 15, "releaseDate": 15, "dueDate": 18 }
    ],
    "18": [
      { "item": "Bicycle", "type": "Manufacture", "qty": 10, "releaseDate": 18, "dueDate": 20 }
    ]
  },
  "summary": {
    "totalOrders": 3,
    "manufactureOrders": 1,
    "purchaseOrders": 2,
    "totalManufactureQty": 10,
    "totalPurchaseQty": 23,
    "earliestReleaseDate": 13,
    "latestReleaseDate": 18
  },
  "actionMessages": null,
  "hasAlerts": false
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `plannedOrders` | `array` | Complete list of planned manufacturing and purchase orders sorted by `releaseDate` ascending. |
| `plannedOrders[].item` | `string` | Item ID for the planned order. |
| `plannedOrders[].type` | `string` | Order classification: `"Manufacture"` if BOM components exist, otherwise `"Purchase"`. |
| `plannedOrders[].qty` | `number` | Net required quantity to order/produce. |
| `plannedOrders[].releaseDate` | `number` | Calculated start/issue day (`dueDate - leadTime`). |
| `plannedOrders[].dueDate` | `number` | Target completion day when the item must be available. |
| `ordersByDate` | `object` | Dictionary grouping planned orders by their integer `releaseDate`. |
| `summary.totalOrders` | `number` | Total number of planned orders generated across all levels. |
| `summary.manufactureOrders` | `number` | Total count of manufacturing orders. |
| `summary.purchaseOrders` | `number` | Total count of purchase orders. |
| `summary.totalManufactureQty` | `number` | Cumulative sum of manufactured unit quantities. |
| `summary.totalPurchaseQty` | `number` | Cumulative sum of purchased unit quantities. |
| `summary.earliestReleaseDate` | `number \| null` | Minimum release day across all orders. |
| `summary.latestReleaseDate` | `number \| null` | Maximum release day across all orders. |
| `actionMessages` | `array \| null` | Array of critical alert strings for impossible lead times (`releaseDate < 0`), or `null` if none exist. |
| `hasAlerts` | `boolean` | `true` if any critical action alert messages were generated, `false` otherwise. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Multi-Level Assembly (Bicycle Demand)

Calculate production and component purchases for 10 bicycles due on Day 20.

**Configuration:**

```json
{
  "bom": {
    "Bicycle": [
      { "childId": "Wheel", "qty": 2 },
      { "childId": "Frame", "qty": 1 }
    ]
  },
  "inventory": {
    "Bicycle": { "onHand": 0, "leadTime": 2 },
    "Wheel": { "onHand": 5, "leadTime": 3 },
    "Frame": { "onHand": 2, "leadTime": 5 }
  },
  "demands": [
    { "itemId": "Bicycle", "grossQty": 10, "dueDay": 20 }
  ]
}
```

**Calculation Flow:**
1. **Bicycle (Top Level):**
   - Gross: 10, OnHand: 0 ──→ Net: 10
   - Due: Day 20, LeadTime: 2 days ──→ Release Date: **Day 18**
   - Order: Manufacture 10 Bicycles (Release: Day 18, Due: Day 20)
2. **Explosion to Components (Due on Day 18):**
   - **Wheel:**
     - Gross Needed: 10 Bicycles × 2 Wheels = 20 Wheels
     - Net: 20 - 5 OnHand = 15 Wheels
     - Due: Day 18, LeadTime: 3 days ──→ Release Date: **Day 15**
     - Order: Purchase 15 Wheels (Release: Day 15, Due: Day 18)
   - **Frame:**
     - Gross Needed: 10 Bicycles × 1 Frame = 10 Frames
     - Net: 10 - 2 OnHand = 8 Frames
     - Due: Day 18, LeadTime: 5 days ──→ Release Date: **Day 13**
     - Order: Purchase 8 Frames (Release: Day 13, Due: Day 18)

---

### Example 2: Critical Schedule Alert (Impossible Lead Time)

Detect schedule infeasibility when requested delivery date cannot cover total lead time.

**Configuration:**

```json
{
  "bom": {
    "Bicycle": [
      { "childId": "Wheel", "qty": 2 },
      { "childId": "Frame", "qty": 1 }
    ]
  },
  "inventory": {
    "Bicycle": { "onHand": 0, "leadTime": 5 },
    "Wheel": { "onHand": 5, "leadTime": 5 },
    "Frame": { "onHand": 2, "leadTime": 5 }
  },
  "demands": [
    { "itemId": "Bicycle", "grossQty": 10, "dueDay": 2 }
  ]
}
```

**Result:**
- Bicycle Release Date: `2 - 5 = -3` (Negative day)
- `hasAlerts`: `true`
- `actionMessages`:
```json
[
  "CRITICAL: Impossible lead time for Bicycle. Needed by Day 2, but lead time is 5 days."
]
```

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: MRP Engine Example Workflow
```

### How it flows

1. **Manual Trigger:** Initiates the MRP calculation workflow.
2. **MRP Engine:** Evaluates multi-level BOM trees, checks inventory on-hand balances, offsets lead times, and generates planned manufacture and purchase orders.
3. **Log Node:** Displays the structured planned orders and summary metrics in the execution console.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Keep Lead Times Accurate:** Ensure item lead times reflect real supplier lead times and shop-floor manufacturing queue times to avoid false schedule alerts.
2. **Inventory Stock Isolation:** The MRP node operates on an in-memory copy of inventory during execution to avoid mutating original state. If processing multiple sequential workflows, update your database with consumed stock levels.
3. **Monitor `hasAlerts`:** Always check the `hasAlerts` flag in downstream workflow conditional nodes (e.g. `If/Else`) to automatically flag impossible schedule requests to procurement managers.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Why is my component order release date negative?
- **Cause:** The parent item's requested due date minus cumulative lead times exceeds the available timeframe.
- **Solution:** Increase the top-level `dueDay` or expedite component lead times.

#### Why was no planned order generated for a component?
- **Cause:** Stock on hand (`onHand`) was greater than or equal to the gross component requirement (`netQty <= 0`).
- **Solution:** Check the `inventory` configuration to verify stock levels.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Log](../log/en.md) — Display MRP output results in the execution log console
- [If / Else](../if-else/en.md) — Route workflow execution based on the `hasAlerts` flag
- [Function](../function/en.md) — Pre-process custom inventory or BOM structures before MRP execution

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial release of the MRP Engine node with netting, lead time offsetting, recursive BOM explosion, and schedule alert features |

<!-- /SECTION: changelog -->

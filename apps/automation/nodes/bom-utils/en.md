---
node_id: "bom-utils"
title: "Product & BOM Management"
description: "Manages Bill of Materials (BOM) operations including Cost Rollup (recursive), BOM Explosion (flatten hierarchy), Where-Used Query, and Standard vs Actual Variance calculations."
category: "business-commerce"
subcategory: "logistics-supply-chain"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - bom
  - bill-of-materials
  - cost-rollup
  - bom-explosion
  - where-used
  - variance
  - supply-chain
related_nodes:
  - inventory-logistics
  - mrp-engine
  - geospatial-utils
---

<!-- SECTION: header -->
# Product & BOM Management

> **Category:** Business & Commerce | **Type:** Action Node

Manage Bill of Materials (BOM) operations including recursive cost rollup, BOM explosion, where-used queries, and standard versus actual cost variance calculations.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Product & BOM Management** node provides utilities for working with Bills of Materials and product cost structures.

It supports four operations:

- `costRollup` — recursively calculate the total cost of a parent item from its components;
- `bomExplosion` — flatten a multi-level BOM into its leaf components;
- `whereUsed` — find parent items that reference a specific child item;
- `variance` — compare standard and actual costs.

### Key Features

- Performs recursive BOM cost calculations.
- Supports multi-level BOM structures.
- Explodes BOM hierarchies into flat component lists.
- Merges duplicate leaf components during BOM explosion.
- Finds where a child component is used.
- Removes duplicate parent IDs from where-used results.
- Calculates standard versus actual cost variance.
- Calculates variance percentages.
- Returns budget interpretation as over budget, under budget, or on budget.
- Rounds cost and quantity values to two decimal places where implemented.

### Processing Flow

```text
Select operation
  ↓
costRollup / bomExplosion / whereUsed / variance
  ↓
Validate operation-specific parameters
  ↓
Perform BOM or cost calculation
  ↓
Aggregate or format results
  ↓
Return structured output
```

### Use Cases

- Calculating manufacturing product costs.
- Expanding multi-level Bills of Materials.
- Determining raw-material requirements.
- Finding products that use a specific component.
- Comparing actual and standard production costs.
- Preparing BOM and cost data for ERP or MRP workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `string` | No | `costRollup` | Operation to perform: `costRollup`, `bomExplosion`, `whereUsed`, or `variance`. |
| `itemId` | `string` | Conditional | — | Parent item ID used by `costRollup`. |
| `bomMap` | `record<array<object>>` | Conditional | — | BOM hierarchy used by `costRollup`. |
| `costMap` | `record<number>` | Conditional | — | Cost values used for leaf components in `costRollup`. |
| `itemIdExplosion` | `string` | Conditional | — | Top-level item ID used by `bomExplosion`. |
| `bomMapExplosion` | `record<array<object>>` | Conditional | — | BOM hierarchy used by `bomExplosion`. |
| `qtyMultiplier` | `number` | No | `1` | Quantity multiplier used by `bomExplosion`. Must be greater than or equal to `0`. |
| `childId` | `string` | Conditional | — | Child item ID searched by `whereUsed`. |
| `bomFlatList` | `array<object>` | Conditional | — | Flat parent-child relation list used by `whereUsed`. |
| `standardCost` | `number` | Conditional | — | Standard cost used by `variance`. Must be greater than or equal to `0`. |
| `actualCost` | `number` | Conditional | — | Actual cost used by `variance`. Must be greater than or equal to `0`. |

### Operation

Supported values:

```text
costRollup
bomExplosion
whereUsed
variance
```

---

### Cost Rollup

Use:

```text
operation: costRollup
```

Required parameters:

```text
itemId
bomMap
costMap
```

The node recursively calculates the cost of every component.

If an item has no children in `bomMap`, it is treated as a leaf component and its value is read from `costMap`.

If the leaf item is not present in `costMap`, the current implementation uses:

```text
0
```

#### BOM Map Structure

Example:

```json
{
  "A": [
    {
      "child": "B",
      "qty": 2
    },
    {
      "child": "C",
      "qty": 1
    }
  ],
  "B": [
    {
      "child": "D",
      "qty": 3
    },
    {
      "child": "E",
      "qty": 1
    }
  ]
}
```

#### Cost Map Structure

Example:

```json
{
  "C": 5,
  "D": 2,
  "E": 4
}
```

For this structure:

```text
Cost(B)
= 3 × Cost(D) + 1 × Cost(E)
= 3 × 2 + 4
= 10
```

Then:

```text
Cost(A)
= 2 × Cost(B) + 1 × Cost(C)
= 2 × 10 + 5
= 25
```

The returned total cost is rounded to two decimal places.

---

### BOM Explosion

Use:

```text
operation: bomExplosion
```

Required parameters:

```text
itemIdExplosion
bomMapExplosion
```

Optional parameter:

```text
qtyMultiplier
```

Default:

```text
1
```

The node recursively traverses the BOM hierarchy until it reaches leaf components.

Only leaf components are added to the resulting flat list.

Duplicate leaf components are merged by adding their quantities.

For example:

```text
A
├── B × 2
│   ├── D × 3
│   └── E × 1
└── C × 1
    └── D × 2
```

produces:

```text
D = 8
E = 2
```

because:

```text
D from B = 2 × 3 = 6
D from C = 1 × 2 = 2
D total = 8

E = 2 × 1 = 2
```

---

### Where-Used Query

Use:

```text
operation: whereUsed
```

Required parameters:

```text
childId
bomFlatList
```

Each relation contains:

```json
{
  "parent": "A",
  "child": "B"
}
```

The node filters relations where:

```text
child === childId
```

and returns the matching parent IDs.

Duplicate parent IDs are removed.

---

### Variance

Use:

```text
operation: variance
```

Required parameters:

```text
standardCost
actualCost
```

Variance is calculated as:

```text
variance = actualCost - standardCost
```

Variance percentage is calculated as:

```text
variancePercentage =
(actualCost - standardCost) / standardCost × 100
```

Interpretation:

```text
variance > 0
→ Over budget

variance < 0
→ Under budget

variance = 0
→ On budget
```

Although the schema accepts:

```text
standardCost >= 0
```

the runtime does not allow a standard cost of `0`.

If:

```text
standardCost: 0
```

the node throws:

```text
Standard cost cannot be zero
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses different parameters depending on the selected `operation`.

#### Cost Rollup

```text
itemId
bomMap
costMap
```

#### BOM Explosion

```text
itemIdExplosion
bomMapExplosion
qtyMultiplier
```

#### Where-Used

```text
childId
bomFlatList
```

#### Variance

```text
standardCost
actualCost
```

Incoming workflow data is not used for these calculations.

### Cost Rollup Output

Example:

```json
{
  "itemId": "A",
  "totalCost": 25,
  "unit": "currency"
}
```

The output contains:

- parent item ID;
- calculated total cost;
- unit identifier.

### BOM Explosion Output

Example:

```json
{
  "itemId": "A",
  "qtyMultiplier": 1,
  "flatList": [
    {
      "id": "D",
      "qty": 8
    },
    {
      "id": "E",
      "qty": 2
    }
  ],
  "totalComponents": 2,
  "totalQuantity": 10
}
```

The output contains:

- top-level item ID;
- applied quantity multiplier;
- flattened leaf component list;
- number of unique leaf components;
- total quantity across the flattened list.

### Where-Used Output

Example:

```json
{
  "childId": "B",
  "parents": [
    "A",
    "C"
  ],
  "count": 2
}
```

The output contains:

- searched child ID;
- unique matching parent IDs;
- number of unique parents.

### Variance Output

Example:

```json
{
  "standardCost": 100,
  "actualCost": 120,
  "variance": 20,
  "variancePercentage": 20,
  "interpretation": "Over budget"
}
```

The `variance` and `variancePercentage` values are rounded to two decimal places.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Cost Rollup

**Configuration**

```text
operation: costRollup
itemId: A
```

**BOM Map**

```json
{
  "A": [
    {
      "child": "B",
      "qty": 2
    },
    {
      "child": "C",
      "qty": 1
    }
  ],
  "B": [
    {
      "child": "D",
      "qty": 3
    },
    {
      "child": "E",
      "qty": 1
    }
  ]
}
```

**Cost Map**

```json
{
  "C": 5,
  "D": 2,
  "E": 4
}
```

**Output**

```json
{
  "itemId": "A",
  "totalCost": 25,
  "unit": "currency"
}
```

### Example 2: BOM Explosion

**Configuration**

```text
operation: bomExplosion
itemIdExplosion: A
qtyMultiplier: 1
```

Using the hierarchy:

```text
A
├── B × 2
│   ├── D × 3
│   └── E × 1
└── C × 1
    └── D × 2
```

**Output**

```json
{
  "itemId": "A",
  "qtyMultiplier": 1,
  "flatList": [
    {
      "id": "D",
      "qty": 8
    },
    {
      "id": "E",
      "qty": 2
    }
  ],
  "totalComponents": 2,
  "totalQuantity": 10
}
```

### Example 3: Where-Used

**Configuration**

```text
operation: whereUsed
childId: B
```

Relations:

```json
[
  {
    "parent": "A",
    "child": "B"
  },
  {
    "parent": "C",
    "child": "B"
  },
  {
    "parent": "D",
    "child": "X"
  },
  {
    "parent": "A",
    "child": "B"
  }
]
```

**Output**

```json
{
  "childId": "B",
  "parents": [
    "A",
    "C"
  ],
  "count": 2
}
```

### Example 4: Cost Variance

**Configuration**

```text
operation: variance
standardCost: 100
actualCost: 120
```

**Output**

```json
{
  "standardCost": 100,
  "actualCost": 120,
  "variance": 20,
  "variancePercentage": 20,
  "interpretation": "Over budget"
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Product & BOM Management Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Unknown Operation

The node supports only:

```text
costRollup
bomExplosion
whereUsed
variance
```

An unsupported operation causes:

```text
Unknown operation: <operation>
```

### Missing Leaf Cost

During cost rollup, if an item has no children and does not exist in `costMap`, the current implementation uses a cost of:

```text
0
```

Verify that all expected raw materials are present in `costMap`.

### Unexpected Cost Rollup Result

Verify:

- `itemId`;
- BOM component quantities;
- recursive child relationships;
- leaf costs in `costMap`.

The node recursively calculates costs until it reaches items without child definitions.

### Unexpected BOM Explosion Quantity

Quantities are multiplied at each hierarchy level.

For example:

```text
A → B × 2
B → D × 3
```

produces:

```text
D × 6
```

If the same leaf component appears through multiple branches, its quantities are merged.

### Unexpected Total Components

`totalComponents` represents the number of unique leaf component IDs after duplicates are merged.

It does not represent the total number of relations in the original BOM hierarchy.

### Unexpected Total Quantity

`totalQuantity` is calculated from the final merged flat list.

It represents the sum of all final leaf quantities.

### Duplicate Where-Used Parents

Duplicate parent IDs are intentionally removed.

For relations containing:

```text
A → B
C → B
A → B
```

the node returns:

```text
A
C
```

with:

```text
count: 2
```

### Standard Cost Is Zero

The schema permits:

```text
standardCost: 0
```

but the runtime rejects this value.

The node throws:

```text
Standard cost cannot be zero
```

**Solution:** Use a standard cost greater than `0`.

### Unexpected Variance Interpretation

Interpretation is determined from:

```text
actualCost - standardCost
```

Positive:

```text
Over budget
```

Negative:

```text
Under budget
```

Zero:

```text
On budget
```

### Circular BOM Relationships

The current implementation performs recursive traversal and does not include explicit cycle detection.

Circular relationships such as:

```text
A → B
B → A
```

should be avoided because they can cause recursive processing to continue indefinitely.

Use acyclic BOM hierarchies.

### Rounding Differences

Cost rollup returns `totalCost` rounded to two decimal places.

BOM explosion rounds each final component quantity to two decimal places.

Variance and variance percentage are also rounded to two decimal places.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Inventory & Logistics** — Manage inventory and logistics-related operations.
- **MRP Engine** — Support material requirements planning workflows.
- **Geospatial Utils** — Provide geospatial utilities used in logistics workflows.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for the Product & BOM Management node. |

<!-- /SECTION: changelog -->
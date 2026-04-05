---
  node_id: "merge"
  title: "Merge"
  description: "Combine items from multiple input streams into a single output using configurable merge strategies, including append, positional pairing, Cartesian products, and field-based matching."
  category: "utilities"
  subcategory: "data-processing"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-03-11"
  author: "Fusion Team"
  tags:
    - merge
    - data-fusion
    - utilities
    - combine
    - join
    - no-code
  related_nodes:
    - function
    - filter
    - set-variable
---

  # Merge

  > **Category:** Utilities&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Combine items from multiple input streams into a single output using configurable merge strategies: append, positional pairing, Cartesian products, and field-based matching.

### Use Cases

- Combine customer data from two CRM systems using a common ID
- Create all possible test case combinations from parameter lists
- Pair messages from two event streams based on their arrival order
- Enrich a primary data set with secondary information (left join)
- Merge configuration files with conflict resolution

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputCount` | `number` | ❌ No | `2` | Number of input ports (2–10). The node will create this many named inputs. |
| `strategy` | `enum` | ❌ No | `zip` | How to combine the input observables:<br>‑ `zip`: Wait for all inputs to emit before merging.<br>‑ `combineLatest`: Merge whenever any input emits (using the latest values from all others). |
| `mergeMode` | `enum` | ❌ No | `append` | The merge algorithm to apply (see details below). |

### Merge Mode Parameters

Depending on the selected `mergeMode`, additional fields become available.

#### `append`
- No extra parameters. All items are simply concatenated.

#### `byPosition`
Pairs items by their index in each input array.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clashStrategy` | `enum` | ✅ Yes | `preferInput2` | How to resolve key conflicts when merging two objects (only if `mergeMode` is not `append`). |
| `deepMerge` | `boolean` | ❌ No | `false` | Recursively merge nested objects instead of shallow assignment. |
| `includeUnpaired` | `boolean` | ❌ No | `false` | If `true`, include items from the longer input even when the shorter input runs out. |

#### `allCombinations`
Generates the Cartesian product (all pairs) of items from the inputs.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clashStrategy` | `enum` | ✅ Yes | `preferInput2` | Conflict resolution for object merging. |
| `deepMerge` | `boolean` | ❌ No | `false` | Deep merge objects. |

#### `byMatchingFields`
Joins items based on one or two match fields (like a database join).

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clashStrategy` | `enum` | ✅ Yes | `preferInput2` | Conflict resolution. |
| `deepMerge` | `boolean` | ❌ No | `false` | Deep merge objects. |
| `matchField1` | `string` | ❌ No | `"id"` | Field name to match in the first input. |
| `matchField2` | `string` | ❌ No | `"id"` | Field name to match in the second input (if different from the first). |
| `outputType` | `enum` | ❌ No | `"keepMatches"` | Which records to include:<br>‑ `keepMatches`: only matched pairs<br>‑ `keepNonMatches`: only unmatched items<br>‑ `keepEverything`: all items<br>‑ `enrichInput1`: keep all input1 items, enrich with matched data from input2<br>‑ `enrichInput2`: keep all input2 items, enrich with matched data from input1 |
| `fuzzyCompare` | `boolean` | ❌ No | `false` | If `true`, compare match fields using string conversion (loose equality); otherwise uses strict JSON stringification. |

### Clash Strategy Values

| Value | Description |
|-------|-------------|
| `preferInput1` | When two objects have the same key, keep the value from the **first** input. |
| `preferInput2` | Keep the value from the **second** input. |
| `addInputNumber` | Rename conflicting keys by appending `_input1` and `_input2` instead of overwriting. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node creates **dynamic input ports** based on `inputCount`. They are named `input1`, `input2`, …, `inputN`. Each input accepts any data type; the node normalizes it to an array of `JsonObject`.

| Input | Type | Description |
|-------|------|-------------|
| `input1` | `any` | First data stream |
| `input2` | `any` | Second data stream |
| … | … | … |
| `inputN` | `any` | Nth data stream (N = `inputCount`) |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted with the merged result |
| `error` | `Error` | Emitted when the merge operation fails |

### Output Schema (`success`)

The **Merge** node collects data from up to 10 parallel input branches and merges the incoming items into a single array. It is designed for complex data fusion scenarios where you need to combine records from different sources.

Each incoming event is normalized to an array of JSON objects (primitives are wrapped as `{ value: <primitive> }`). The node then applies the selected **merge mode** to produce a unified array, emitted on the `success` output. Errors are sent to the `error` output.

### Key Features

- **Configurable Input Count:** 2–10 inputs
- **Data Stream Strategies:**
  - `zip`: Wait for all inputs to emit before merging
  - `combineLatest`: Merge whenever any input emits (using latest values from all others)
- **Merge Modes:**
  - `append`: Concatenate all items
  - `byPosition`: Pair items by their index
  - `allCombinations`: Generate the Cartesian product
  - `byMatchingFields`: Join items based on one or two fields
- **Conflict Resolution:** Control how overlapping keys are handled (`preferInput1`, `preferInput2`, `addInputNumber`)
- **Deep Merge Option:** Recursively merge nested objects
- **Field-Based Joins:** Specify match fields, output type (matches, non-matches, everything, enrich), and fuzzy string comparison

### Use Cases

- Combine customer data from two CRM systems using a common ID
- Create all possible test case combinations from parameter lists
- Pair messages from two event streams by arrival order
- Enrich a primary data set with secondary information (left join)
- Merge configuration files with conflict resolution

Combine all items from two inputs, preserving order.

**Configuration:**
```json
{
  "inputCount": 2,
  "strategy": "zip",
  "mergeMode": "append"
}
```

**Input 1:** `[ { "id": 1, "name": "Alice" } ]`  
**Input 2:** `[ { "id": 2, "name": "Bob" } ]`

**Output (success):**
```json
{
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ]
}
```

---

### Example 2: By Position (Zipping by Index)

Pair items by their position in each array.

**Configuration:**
```json
{
  "inputCount": 2,
  "mergeMode": "byPosition",
  "clashStrategy": "preferInput2",
  "includeUnpaired": true
}
```

**Input 1:** `[ { "color": "red" }, { "color": "blue" } ]`  
**Input 2:** `[ { "size": "M" }, { "size": "L" }, { "size": "XL" } ]`

**Output (includeUnpaired = true):**
```json
[
  { "color": "red", "size": "M" },
  { "color": "blue", "size": "L" },
  { "size": "XL" }               // from the longer input, no pair
]
```

---

### Example 3: All Combinations (Cartesian Product)

Generate all possible pairs.

**Configuration:**
```json
{
  "inputCount": 2,
  "mergeMode": "allCombinations",
  "clashStrategy": "addInputNumber"
}
```

**Input 1:** `[ { "letter": "A" }, { "letter": "B" } ]`  
**Input 2:** `[ { "digit": 1 }, { "digit": 2 } ]`

**Output:**
```json
[
  { "letter": "A", "digit_input2": 1, "digit_input1": ??? }  // Actually, addInputNumber renames keys
]
```
  > With `addInputNumber`, conflicting keys get `_input1` and `_input2` suffixes. Here there is no conflict (different keys), so output is plain merged objects:
  ```json
  [
    { "letter": "A", "digit": 1 },
    { "letter": "A", "digit": 2 },
    { "letter": "B", "digit": 1 },
    { "letter": "B", "digit": 2 }
  ]
```

---

### Example 4: By Matching Fields (Join on ID)

Join two arrays using a common field.

**Configuration:**
```json
{
  "inputCount": 2,
  "mergeMode": "byMatchingFields",
  "matchField1": "userId",
  "matchField2": "id",
  "outputType": "keepMatches",
  "clashStrategy": "preferInput1",
  "fuzzyCompare": false
}
```

**Input 1 (orders):**  
`[ { "userId": 101, "product": "Laptop" }, { "userId": 102, "product": "Mouse" } ]`

**Input 2 (users):**  
`[ { "id": 101, "name": "Alice" }, { "id": 103, "name": "Bob" } ]`

**Output (only matches):**
```json
[
  { "userId": 101, "product": "Laptop", "name": "Alice" }
]
```

---

### Example 5: Enrich Input 1 (Left Join)

Keep all items from the first input, add matched fields from the second.

**Configuration:**
```json
{
  "mergeMode": "byMatchingFields",
  "matchField1": "userId",
  "matchField2": "id",
  "outputType": "enrichInput1",
  "clashStrategy": "preferInput1"
}
```

Using the same inputs as above, the output becomes:

```json
[
  { "userId": 101, "product": "Laptop", "name": "Alice" },
  { "userId": 102, "product": "Mouse", "name": null }   // unmatched, second input fields are null
]
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Enrich Webhook Data with Database Records

A webhook delivers order information containing only a customer ID. Merge it with a database query result to produce a complete order summary.

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "query-customer",
      "type": "mysql-action",
      "config": {
        "operation": "Select",
        "selectParams": {
          "where": "id = {{input.customerId}}"
        }
      }
    },
    {
      "id": "merge",
      "type": "merge",
      "config": {
        "inputCount": 2,
        "mergeMode": "byMatchingFields",
        "matchField1": "customerId",
        "matchField2": "id",
        "outputType": "enrichInput1",
        "clashStrategy": "preferInput1"
      }
    }
  ]
}
```

**How it flows:**
1. Webhook emits an object like `{ "customerId": 101, "orderId": "ORD-123", "amount": 99.90 }`
2. MySQL action returns `[{ "id": 101, "name": "Alice", "email": "alice@example.com" }]`
3. Merge node joins on `customerId` (input1) and `id` (input2), producing:
```json
{ "customerId": 101, "orderId": "ORD-123", "amount": 99.90, "name": "Alice", "email": "alice@example.com" }
```

### Common Patterns

- **Data Enrichment:** Merge API response with a database lookup
- **Test Case Generation:** `allCombinations` of input parameters to create exhaustive test data
- **Parallel Stream Synchronisation:** `byPosition` to align messages from two event sources
- **Conflict-Free Merging:** `addInputNumber` to preserve both values when keys collide

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Input count must be between 2 and 10`

**Cause:** `inputCount` is set to a value outside the allowed range.

**Solution:** Choose a value between 2 and 10.

#### `Unexpected output structure – missing fields`

**Cause:** Conflict resolution may have overwritten fields you intended to keep.

**Solution:** Use `clashStrategy: "addInputNumber"` to retain both conflicting values, or adjust the order of inputs if you prefer `preferInput1`/`preferInput2`.

#### `byMatchingFields produces no matches`

**Cause:** The match fields are spelled incorrectly, or the values are of different types (e.g. string vs number). With `fuzzyCompare: false`, the comparison uses strict JSON stringification, so `101` (number) does not match `"101"` (string).

**Solution:** Enable `fuzzyCompare` to convert both sides to strings, or ensure type consistency upstream.

#### `Array items are wrapped as { value: … }`

**Cause:** The input data contained primitives (strings, numbers, etc.) instead of objects. This is expected behavior – the node normalises all items to `JsonObject`.

**Solution:** If you need the original shape, map over the output inside a Function node to unwrap the `value` fields.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `Invalid merge mode` | `mergeMode` is not one of the allowed values | Use `append`, `byPosition`, `allCombinations`, or `byMatchingFields` |
| `matchField1 and matchField2 are required` | `byMatchingFields` selected but fields missing | Provide `matchField1` (and optionally `matchField2`) |
| `clashStrategy required` | A merge mode that requires conflict resolution was chosen without it | Set `clashStrategy` to one of the allowed values |
| `outputType must be one of …` | `byMatchingFields` used with invalid output type | Choose from the enum list |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Further transform merged data with custom JavaScript
- [Filter](./filter.md) – Filter items after merging
- [Set Variable](./set-variable.md) – Store merged results for later use

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->

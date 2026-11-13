---
  node_id: "split-out"
  title: "Split Out"
  description: "Split a list into separate items, emitted one by one. Pair it with Accumulate to combine the results again."
  category: "utilities"
  subcategory: "data-processing"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-11-13"
  author: "Fusion Team"
  tags:
    - split
    - split-out
    - items
    - list
    - iteration
    - utilities
    - no-code
  related_nodes:
    - accumulate
    - loop
    - function
---

<!-- SECTION: overview -->
  # Split Out

  > **Category:** Utilities&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Utility Node

  Turn a list into separate items. Every element of the list is emitted on its own, so the next nodes run once per element. Pair it with [Accumulate](/docs/accumulate) to combine the results back into one.

### How it works

- Split Out reads the list (the input itself, or the field you name) and emits **every element at once**, in order.
- Each emitted item remembers which run it belongs to, its position and how many items the run has. Accumulate uses this to put the run back together, in the original order, even when items finish out of order.
- Items run **in parallel** through the nodes after Split Out. To process one batch at a time instead, use [Loop](/docs/loop).

### Use Cases

- Process every line item of an order, every row of a CSV, or every document returned by a query
- Call an API once per element of a list
- Validate each record separately and collect the results
- Split a nested list (for example `order.lines`) while keeping parent fields such as the order id

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `list` (List To Split) | JSON / expression | ❌ No | — | **Leave empty** to split the list coming into the node. Or choose a list from an earlier step, e.g. `{{ outputs.Order.success.lines }}`, or write a JSON list such as `[1, 2, 3]`. |
| `keepInputFields` (Keep Input Fields) | `enum` | ❌ No | `None` | Copy fields of the node's input into every item: `None`, `All` (every field except the list being split) or `Selected`. |
| `fieldsToKeep` (Fields To Keep) | list of field names | ❌ No | `[]` | The fields to copy when **Keep Input Fields** is `Selected`, e.g. `orderId`, `customer.name`. |
| `itemField` (Put Each Item In Field) | `string` | ❌ No | — | Put each element under this field. When empty, an element that is an object keeps its own fields, and any other value is put in a field named `value`. |
| `fieldNamesContainDots` (Field Names Contain Dots) | `boolean` | ❌ No | `false` | Turn on when a field is really named with a dot, like `"sensor.temp"`. Off, `customer.name` means the `name` inside `customer`. |

### The simplest setup

Connect the step that produces the list and leave every option as it is. Split Out splits whatever list comes in.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The list to split, when **List To Split** is empty. Otherwise the fields that **Keep Input Fields** copies come from here. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | One event per element of the list. |
| `error` | `object` | The list to split is not a list, or refers to a step that has not produced output yet. |

An empty list emits nothing.

### Example

**Configuration:**
```json
{ "list": "{{ outputs.Order.success.lines }}", "keepInputFields": "Selected", "fieldsToKeep": ["orderId"] }
```

**Input** (the output of the step labelled `Order`):
```json
{ "orderId": "ORD-1001", "customer": { "name": "Alice" }, "lines": [ { "sku": "A-1", "qty": 2 }, { "sku": "B-7", "qty": 1 } ] }
```

**Output (two events):**
```json
{ "orderId": "ORD-1001", "sku": "A-1", "qty": 2 }
{ "orderId": "ORD-1001", "sku": "B-7", "qty": 1 }
```

A list of plain values is put in `value`: splitting `["x", "y"]` gives `{ "value": "x" }` and `{ "value": "y" }`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Split an order into line items and total them
```

**How it flows:**
1. **Order** returns an order with three lines.
2. **Split Out** takes `{{ outputs.Order.success.lines }}` and emits each line as its own item, keeping `orderId`.
3. **Line Total** runs once per line and adds `total`.
4. **Accumulate** waits for all three and returns `{ "skus": [...], "lineTotals": [...] }` in the original order.

### Common Patterns

- **Split → action → Accumulate:** run a step per element and get one combined result.
- **Split → Filter or If/Else → Accumulate:** items that are filtered out simply don't reach Accumulate, which combines the rest.
- **Nested:** a Split Out inside another Split Out's path pairs with the nearest Accumulate, like brackets.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

#### `The input is not a list. Choose the list to split in "List To Split"…`

**Cause:** **List To Split** is empty and the input is an object, not a list.

**Solution:** Choose the list in **List To Split**, e.g. `{{ outputs.Order.success.lines }}`.

#### `"List To Split" is not a list.`

**Cause:** The expression points at something that is not a list, or the JSON text is not a list.

**Solution:** Check the path in the expression, or write a JSON list such as `[1, 2, 3]`.

#### `"List To Split" refers to a step that has not produced output yet.`

**Cause:** The expression names a step that has not run before Split Out.

**Solution:** Point it at a step that runs earlier on the same path.

#### Expressions read another item's value

**Cause:** Items run in parallel. An expression such as `{{ outputs.SomeNode.success.field }}` reads that node's **latest** output, which may belong to a different item.

**Solution:** Pass values along with the item itself (use **Keep Input Fields**, or return them from the previous node) instead of reading them back by expression. Use [Loop](/docs/loop) when steps must run strictly one at a time.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Accumulate](/docs/accumulate) – Combine the split items back into one
- [Loop](/docs/loop) – Process a list one batch at a time instead of in parallel
- [Function](/docs/function) – Transform each item

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-13 | Initial release |

<!-- /SECTION: changelog -->

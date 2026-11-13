---
  node_id: "accumulate"
  title: "Accumulate"
  description: "Combine items into one: all items into one list, or chosen fields into lists. Collects every item of a Split Out run, in order, before emitting."
  category: "utilities"
  subcategory: "data-processing"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-11-13"
  author: "Fusion Team"
  tags:
    - accumulate
    - aggregate
    - combine
    - collect
    - items
    - utilities
    - no-code
  related_nodes:
    - split-out
    - loop
    - merge
---

<!-- SECTION: overview -->
  # Accumulate

  > **Category:** Utilities&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Utility Node

  Combine items into a single result: collect chosen fields into lists, or put all items into one list. It is the partner of [Split Out](/docs/split-out).

### How it works

- **Items from a Split Out** are grouped by their run and combined **in the original order** as soon as all of them have arrived, even if they finished out of order.
- **Items that never arrive** (a step failed, or a Filter dropped them) are given up on after **Wait For Missing Items**: the items that did arrive are combined and emitted.
- **Input that did not come from a Split Out** is combined at once: a list counts as several items, anything else as one.
- Two runs going through at the same time never mix.

### Use Cases

- Collect the results of a per-item step back into one list
- Build `{ "emails": [...] }` from a list of contacts
- Total up line items after processing each one separately
- Gather validation results for every record into a single report

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `combine` (Combine) | `enum` | ❌ No | `All Items` | ‑ `All Items`: every item into one list.<br>‑ `Selected Fields`: chosen fields of every item, each into its own list. |
| `fieldNamesContainDots` (Field Names Contain Dots) | `boolean` | ❌ No | `false` | Turn on when a field is really named with a dot, like `"sensor.temp"`. |
| `waitForMissingItems` (Wait For Missing Items) | `number` | ❌ No | `60` | Seconds to wait for items of a Split Out run that never arrive, counted from the last item received. Then the items that arrived are combined. |

### All Items

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `outputField` (Put List In Field) | `string` | `items` | The field that holds the list. |
| `onlyKeepFields` (Only Keep Fields) | list of field names | `[]` | Keep only these fields of each item. Empty keeps every field. |
| `removeFields` (Remove Fields) | list of field names | `[]` | Leave these fields out of each item, e.g. `password`. |

### Selected Fields

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `fields` (Fields) | list of { Field, Output Name } | `[]` | One entry per field, e.g. Field `price`, Output Name `prices`. An empty Output Name uses the field name. |
| `flattenLists` (Flatten Lists) | `boolean` | `false` | When a field is itself a list, add its elements instead of the whole list. |
| `keepEmptyValues` (Keep Empty Values) | `boolean` | `false` | Add `null` for items where the field is missing or empty, so every list has one entry per item. |

### The simplest setup

Leave everything as it is: you get `{ "items": [ ...every item, in order... ] }`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Items to combine, typically from the nodes after a Split Out. A step that returns a list for one item contributes all of its elements, in place. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | One combined result per run. |
| `error` | `object` | A configured field was not found in any item, or no field is configured. |

### Example: All Items (default)

**Configuration:** `{ "outputField": "results", "removeFields": ["password"] }`

**Output:**
```json
{ "results": [ { "id": 1, "name": "Ana", "valid": true }, { "name": "Error", "message": "invalid email for Bob" }, { "id": 3, "name": "Chen", "valid": true } ] }
```

### Example: Selected Fields

**Configuration:**
```json
{
  "combine": "Selected Fields",
  "fields": [
    { "field": "price" },
    { "field": "customer.email", "outputName": "emails" }
  ]
}
```

**Items:** `{ "price": 1, "customer": { "email": "a@x.io" } }`, `{ "price": 2 }`, `{ "price": 3, "customer": { "email": "c@x.io" } }`

**Output:**
```json
{ "price": [1, 2, 3], "emails": ["a@x.io", "c@x.io"] }
```

With **Keep Empty Values**, `emails` would be `["a@x.io", null, "c@x.io"]`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Check every user and collect the results in order
```

**How it flows:**
1. **Users** returns three users.
2. **Split Out** emits each user separately.
3. **Check Email** runs once per user. Both its `success` and `error` outputs go to Accumulate, so a failed check still counts as an item.
4. **Accumulate** returns `{ "results": [...] }` once all three are back, in the original order, without the `password` field.

### Common Patterns

- **Wire error outputs into Accumulate too** when every item must be accounted for. Otherwise a failed item only stops being waited for after the timeout.
- **Filter in the middle:** dropped items are fine; Accumulate combines the rest after the wait. Lower **Wait For Missing Items** if that wait is too long.
- **Slow steps** (for example AI calls taking minutes per item): raise **Wait For Missing Items** above the slowest step, so a run is not combined early.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

#### `The field "x" was not found in any item.`

**Cause:** No item has that field: a misspelled name, or a name with a dot while dot notation is on.

**Solution:** Check the name, turn on **Field Names Contain Dots** for names like `"temp.c"`, or turn on **Keep Empty Values**.

#### The result arrives late, or has fewer items than expected

**Cause:** Some items never reached Accumulate (a step failed without its error output wired here, or a Filter dropped them), so Accumulate waited for **Wait For Missing Items** before combining the rest.

**Solution:** Wire the error outputs of the steps in between into Accumulate as well, or lower the timeout.

#### Results split into two outputs

**Cause:** A step took longer than **Wait For Missing Items**, so the run was combined before that item arrived. A later item is dropped, not emitted again.

**Solution:** Raise **Wait For Missing Items** above the slowest step.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Split Out](/docs/split-out) – Split a list into separate items
- [Loop](/docs/loop) – Process a list one batch at a time and collect the results
- [Merge](/docs/merge) – Combine items from several branches

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-13 | Initial release |

<!-- /SECTION: changelog -->

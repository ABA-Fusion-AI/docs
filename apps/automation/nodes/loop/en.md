---
  node_id: "loop"
  title: "Loop"
  description: "Process a list one batch at a time through a loop body, accumulate the results, and emit them when finished. Backpressured — only one batch is in flight at a time."
  category: "utilities"
  subcategory: "data-processing"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-06-25"
  author: "Fusion Team"
  tags:
    - loop
    - batch
    - split-in-batches
    - iteration
    - accumulate
    - utilities
    - no-code
  related_nodes:
    - function
    - merge
    - filter
---

  # Loop

  > **Category:** Utilities&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Utility Node

  Process a list **one batch at a time**. The node emits a batch on `loop`, you wire that through a body of nodes that process it, and feed the result back into `next`. The node then emits the following batch — and so on — until the list is exhausted, at which point it emits **all accumulated results** on `done`.

  Loop is **backpressured**: it holds the next batch until the current one returns, so only one batch is ever in flight. That has two big consequences:

  - **Config expressions resolve correctly** — an expression in the body that references this node's output (e.g. `{{ outputs.Loop.loop.item }}`, i.e. `outputs.<node label>.<outputId>.<field>`) always sees the batch currently being processed, with **no `stagger` needed** (the fan-out race is gone because the output never advances mid-flight).
  - **Accumulation is built in** — you don't need a separate Accumulate node; results come out together on `done`.

### Use Cases

- Call an API once per item (or per N items) and collect all responses
- Rate-limited processing where each batch must finish before the next starts
- Transform a list through several nodes, then act on the whole result at once

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `batchSize` | `number` | ❌ No | `1` | Number of items processed per iteration. `1` = one item at a time. The final batch may be smaller. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Starts a run. The list is queued and the first batch is emitted on `loop`. A non-array value is treated as a single-item list. |
| `next` | `any` | The processed batch coming **back** from the body. Its data is accumulated, then the next batch is emitted (or `done` fires). Wire the end of your loop body here. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `loop` | `object` | The current batch to process. Wire this into your loop body. |
| `done` | `array` | All accumulated batch results, emitted once when the list is exhausted. |
| `error` | `Error` | Emitted if processing fails. |

### `loop` payload

```ts
{
  item,        // items[0] — convenient for batchSize = 1
  items,       // the current batch (array)
  index,       // source index of the first item in this batch
  total,       // total source items
  batchIndex,  // 0-based batch number
  batchCount,  // total batches
  isFirst,
  isLast
}
```

### `done` payload

An array with **one entry per processed batch** — i.e. whatever your body returned to `next`, in order. The run's session (with `requestId` etc.) is preserved, so a **Respond to Webhook** after `done` responds once with the full result.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Enrich each item, collect the results

**Configuration:** `{ "batchSize": 1 }`

**Wiring (a cycle):**
```
loop.loop → http-request → loop.next
loop.done → store
```

**Input:** `[ "u1", "u2", "u3" ]` — HTTP Request calls `/users/{{ outputs.Loop.loop.item }}` (the loop node is labelled `Loop`; `loop` is the output handle, `item` the field). No stagger needed.

**Output on `done`:**
```json
[ { "id": "u1", … }, { "id": "u2", … }, { "id": "u3", … } ]
```

---

### Example 2: Batches of 2

**Configuration:** `{ "batchSize": 2 }`

**Input:** `[ "a", "b", "c", "d", "e" ]`, body joins each batch with `+`.

**Output on `done`:**
```json
[ "a+b", "c+d", "e" ]
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

```json
{
  "nodes": [
    { "id": "trigger", "type": "manual-trigger" },
    { "id": "loop", "type": "loop", "data": { "label": "Loop" }, "config": { "batchSize": 1 } },
    { "id": "enrich", "type": "http-request", "config": { "url": "https://api.example.com/users/{{ outputs.Loop.loop.item }}" } },
    { "id": "store", "type": "function" }
  ],
  "connections": [
    { "source": "trigger", "sourceHandle": "success", "target": "loop", "targetHandle": "input" },
    { "source": "loop", "sourceHandle": "loop", "target": "enrich", "targetHandle": "input" },
    { "source": "enrich", "sourceHandle": "success", "target": "loop", "targetHandle": "next" },
    { "source": "loop", "sourceHandle": "done", "target": "store", "targetHandle": "input" }
  ]
}
```

The `enrich → loop.next` edge is the **loop-back** that drives the iteration.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

#### The loop never finishes / `done` never fires

**Cause:** The loop body doesn't return exactly one event per batch to `next`. If a node inside the body drops the batch (e.g. a Filter) or splits it into several events, the counter desyncs.

**Solution:** Keep the body **one-event-in, one-event-out** per batch. Operate on `data.items` as a whole if you need per-item work inside the batch, and make sure every batch reaches `next`.

#### `done` returns nested arrays I didn't expect

**Cause:** `done` collects one entry per batch. With `batchSize > 1`, each entry is whatever the body returned for that batch (often an array).

**Solution:** Flatten in a Function node after `done` if you want a single flat list.

#### Each iteration feels slow

**Cause:** Every hop around the cycle carries the engine's per-hop latency, so an iteration costs a bit more than the body's own work. This is the cost of backpressure (one batch at a time).

**Solution:** Increase `batchSize` to process more items per iteration.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Transform each batch inside the loop body, or post-process the `done` result
- [Merge](./merge.md) – Join parallel branches (different role from looping)

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-06-25 | Initial release — backpressured batch loop with built-in accumulation. |

<!-- /SECTION: changelog -->

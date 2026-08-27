---
node_id: "take"
title: "Take"
description: "Takes the first number of items from the input array"
category: "Utility / Stream Control"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:

- stream
- rxjs
- take
- limit
- utility
- reactive

related_nodes:
- function
- if

---

# Take

> **Category:** utility-nodes | **Type:** Streaming Node (`BaseNode`, RxJS-based)

Take the **first N items** emitted on an input stream and forward them to the output.

The **Take** node is built on RxJS's `take()` operator, subscribing to a named `input` stream and re-emitting up to `count` items on a named `output` stream — a different architecture from the request/response `ActionNode` nodes documented elsewhere in this series.

## ⚠️ Different Node Architecture

Unlike the other nodes in this documentation series (which extend `ActionNode` and implement a single `handleTick()` request/response method), **Take extends `BaseNode`** and works with **reactive streams** (RxJS `Observable`s) via named, typed inputs and outputs (`this.inputs.input`, `this.outputs.output`, `this.outputs.error`). It has no `handleTick` method; instead, its behavior is defined in `run()`, which sets up a subscription for the lifetime of the node.

### Supported Features

- Limits an input stream to at most `count` emitted items, using RxJS `take(count)`
- Dedicated `error` output stream, separate from the main `output` stream
- Automatic completion propagation: both `output` and `error` streams are completed when the input completes or after an error
- Normalized error objects via `normalizeWorkflowError`

### Use Cases

- Limit how many items from an upstream loop or paginated source are processed downstream
- Cap the number of records forwarded to a rate-limited or costly downstream node (e.g. an LLM or paid API)
- Take a fixed-size sample from a larger stream for testing or preview purposes
- Combine with a "Skip"-style node (if available) to implement stream-based pagination

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `count` | `number` | ❌ No | `1` | Maximum number of items to take from the input stream. |

---

## Inputs & Outputs

Unlike request/response nodes, Take's inputs and outputs are **named reactive streams**, not a single request payload / return value.

### Inputs

| Input | Description |
| ----- | ----------- |
| `input` | The stream of items to take from. Each emitted value is a candidate to be forwarded. |

### Outputs

| Output | Description |
| ------ | ----------- |
| `output` | Emits the first `count` items from `input`, then completes. |
| `error` | Emits a single normalized error object if the input stream errors, then both `output` and `error` complete. |

### Error Object Shape (on `error` output)

| Field | Type | Description |
| ----- | ---- | ----------- |
| `session` | `object` | Always an empty object (`{}`) in the current implementation. |
| `data` | `object` | The result of `normalizeWorkflowError(err)` — a normalized representation of the upstream error. |

---

## How It Works

1. On `run()`, the node subscribes to `this.inputs.input`, piped through RxJS's `take(count)`.
2. **On each item** (`next`): the item is forwarded as-is to `this.outputs.output`.
3. **On error** (`error`): a normalized error object is emitted once on `this.outputs.error`, and then **both** `output` and `error` are completed — no further items are forwarded after an error, even if fewer than `count` items had been emitted.
4. **On completion** (`complete`): once the input stream completes (either naturally, or because `take(count)` has already emitted `count` items and unsubscribes), both `output` and `error` are completed.

Because `take(count)` unsubscribes from the source once `count` items have been emitted, this node does **not** wait for the upstream input to fully complete if the limit is reached first — it completes its own outputs as soon as the limit is hit.

---

## Output Example

Given an input stream emitting `1, 2, 3, 4, 5` and `count: 3`:

```text
output: 1
output: 2
output: 3
output: (completed)
```

Items `4` and `5` are never received by this node's `output`, since the subscription is dropped after the 3rd item.

### On Input Error (`count: 5`, input errors after 2 items)

```text
output: 1
output: 2
error: { session: {}, data: { <normalized error> } }
output: (completed)
error: (completed)
```

---

## Configuration Examples

### Take the First Item (Default)

```json
{
  "count": 1
}
```

### Take the First 10 Items

```json
{
  "count": 10
}
```

---

## Workflow Integration

### Sample Workflow: Loop/Stream Source → Take → Function

```json
{
  "nodes": [
    {
      "id": "paginated-source",
      "type": "http-request"
    },
    {
      "id": "take-first-5",
      "type": "take",
      "config": {
        "count": 5
      }
    },
    {
      "id": "process-sample",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Take → LLM (limit expensive calls)

```json
{
  "nodes": [
    {
      "id": "search-results-stream",
      "type": "tavily"
    },
    {
      "id": "take-top-3",
      "type": "take",
      "config": {
        "count": 3
      }
    },
    {
      "id": "summarize-each",
      "type": "llm"
    }
  ]
}
```

### Common Patterns

- Stream source → Take (`count: N`) → costly downstream node (LLM, paid API) — cost/rate control
- Stream source → Take (`count: 1`) → Function — "get just the first result"
- Take → error output → Notification/Logging — dedicated error-path handling separate from the main flow

---

## Error Handling

The node does not throw synchronously — errors from the **upstream input stream** are caught by the RxJS subscription's `error` handler and routed to the dedicated `error` output, as a normalized error object (`normalizeWorkflowError(err)`), rather than propagating as a workflow-level exception.

There is no validation on `count` itself (e.g. negative or zero values are not explicitly rejected by this node — RxJS's `take()` operator's own behavior applies: `take(0)` completes immediately with no emissions).

---

## Troubleshooting

### `output` Completes with Fewer Than `count` Items

**Cause**

The upstream `input` stream completed or errored before emitting `count` items — this is not a fault of the Take node itself.

**Solution**

Check the upstream node for early completion or errors; if an error occurred, inspect the `error` output for details.

---

### Downstream Nodes Never Receive Anything

**Cause**

`count` may be set to `0`, in which case RxJS's `take(0)` completes the output stream immediately without emitting any items.

**Solution**

Set `count` to a positive integer.

---

### Error Appears on `error` Output But Workflow Doesn't "Fail" Visibly

**Cause**

By design, input-stream errors are routed to the dedicated `error` output rather than thrown as a workflow-level exception — a workflow that doesn't explicitly wire up the `error` output may silently ignore the failure.

**Solution**

Connect the `error` output to a logging or notification node if error visibility is required.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All behavior is implemented via local RxJS stream operators.

---

## Notes

The node operates on a **stream**, not a single value — its behavior only makes sense in the context of an input that emits multiple discrete items over time (e.g. a paginated fetch, a loop, or another streaming node), unlike the request/response `ActionNode`s documented elsewhere in this series.

The node does not:

- Support taking the *last* N items (only the *first* N, per RxJS `take()` semantics) — no `takeLast` equivalent
- Support conditional taking (e.g. `takeWhile` based on a predicate)
- Validate or clamp `count` to a sensible range
- Retry or resubscribe to the input stream after an error
- Buffer or replay items already emitted before a late subscriber connects

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-27 | Initial release |
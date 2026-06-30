---
  node_id: "delay"
  title: "Delay"
  description: "Delay the emission of each item from the input by a fixed duration (in milliseconds) before forwarding it downstream, without altering the data."
  category: "orchestration"
  subcategory: "timing-rate-control"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-06-30"
  author: "Fusion Team"
  tags:
    - delay
    - timing
    - rate-control
    - orchestration
    - no-code
  related_nodes:
    - debounce-time
    - throttle-time
    - interval
---

<!-- SECTION: overview -->
  # Delay

  > **Category:** Orchestration&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Utility Node

  Hold each incoming item for a fixed `duration` (in **milliseconds**) and then forward it unchanged on the `success` output. The data payload is **not modified** — only its timing is shifted. Every item is delayed by the same amount, preserving order.

### Use Cases

- Space out requests to a rate-limited downstream API
- Wait a fixed period after a trigger before performing a follow-up action
- Stagger items in a pipeline to avoid overwhelming a slow consumer
- Insert a deliberate pause between steps in a sequence

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `duration` | `number` | ✅ Yes | — | Delay in **milliseconds**. Must be a non-negative integer. Each item is held for this long before being emitted. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The item to delay. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The original item, emitted unchanged after `duration` milliseconds. |
| `error` | `Error` | Emitted if the input stream errors. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Wait One Second

**Configuration:**
```json
{
  "duration": 1000
}
```

Each item arriving on `input` is forwarded to `success` exactly 1000 ms later, with its data untouched.

---

### Example 2: No Delay (Pass-Through)

**Configuration:**
```json
{
  "duration": 0
}
```

With `duration: 0` the node forwards items as soon as possible — useful for temporarily disabling a delay without removing the node.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Throttle Follow-Up Calls

```json
{
  "nodes": [
    { "id": "trigger", "type": "manual-trigger" },
    {
      "id": "wait",
      "type": "delay",
      "config": { "duration": 2000 }
    },
    { "id": "notify", "type": "http-request" }
  ]
}
```

**How it flows:**
1. The trigger emits an item.
2. **Delay** holds it for 2000 ms.
3. The HTTP request fires two seconds after the trigger.

### Common Patterns

- **Rate limiting:** add a small delay before calling an external API
- **Sequencing:** introduce a fixed pause between dependent steps
- **Backoff:** delay retries downstream of an error branch

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Delay must be a non-negative integer (milliseconds)`

**Cause:** `duration` is missing, negative, or not an integer.

**Solution:** Provide a whole number ≥ 0, expressed in milliseconds (e.g. `1500` for 1.5 seconds).

#### Items appear later than expected

**Cause:** `duration` is in **milliseconds**, not seconds.

**Solution:** Multiply seconds by 1000 (e.g. 5 seconds → `5000`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Debounce Time](./debounce-time.md) – Emit only after a quiet period
- [Throttle Time](./throttle-time.md) – Limit emission rate over a window
- [Interval](./interval.md) – Emit on a recurring timer

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-06-30 | Initial release (node moved to the builtin package). |

<!-- /SECTION: changelog -->

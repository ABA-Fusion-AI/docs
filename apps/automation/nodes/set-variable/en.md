---
  node_id: "set-variable"
  title: "Set Variable"
  description: "Store one or more values in the workflow's shared variable store using a key/value map. Values accept any type and support expressions, and can be read later by downstream nodes such as Function."
  category: "orchestration"
  subcategory: "functions-variables"
  version: "1.1.0"
  language: "en"
  last_updated: "2026-06-30"
  author: "Fusion Team"
  tags:
    - set-variable
    - variables
    - state
    - orchestration
    - no-code
  related_nodes:
    - function
    - merge
    - filter
---

<!-- SECTION: overview -->
  # Set Variable

  > **Category:** Orchestration&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Write one or more values into the workflow's **shared variable store** using a list of key/value pairs. Variables set here persist for the duration of the run and can be read by downstream nodes (for example inside a **Function** node via the injected `variables` object).

  Values accept **any type** — strings, numbers, booleans, objects, arrays — and support **expressions**, so you can reference upstream node outputs when computing a value. The node passes its **input through unchanged** on the `success` output, so it can be dropped into the middle of a chain without breaking the data flow.

### Use Cases

- Capture a value early in a run and reuse it in several later branches
- Store a counter, flag, or configuration object shared across nodes
- Stash a computed/derived value (via an expression) for a downstream Function node
- Persist an upstream API response under a friendly name for later reference

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `map` | `array` | ❌ No | `[]` | List of key/value pairs to write into the workflow variables. Each entry sets one variable. |

### `map` Entry

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | `string` | ✅ Yes | Name of the workflow variable to set. |
| `value` | `any` | ✅ Yes | Value to store. Accepts any type (string, number, boolean, object, array) and supports expressions. |

> Setting the same `key` more than once (within the map or across multiple Set Variable nodes) overwrites the previous value.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Trigger for the node. The data is **passed through unchanged** to `success`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The original input data, forwarded unchanged after the variables are written. |
| `error` | `Error` | Emitted if writing a variable fails. |

The variables themselves are written as a side effect to the shared workflow variable store; they are **not** placed on the output payload. Read them downstream (e.g. in a **Function** node, `variables.<key>`).

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Set a Single String Variable

**Configuration:**
```json
{
  "map": [
    { "key": "greeting", "value": "Hello, world" }
  ]
}
```

After this node runs, `greeting` is available to downstream nodes. The input data flows through to `success` unchanged.

---

### Example 2: Set Multiple Variables of Different Types

**Configuration:**
```json
{
  "map": [
    { "key": "retries", "value": 3 },
    { "key": "isEnabled", "value": true },
    { "key": "settings", "value": { "timeout": 5000, "mode": "fast" } }
  ]
}
```

`retries` (number), `isEnabled` (boolean), and `settings` (object) are all stored with their native types — no string coercion.

---

### Example 3: Store a Value from an Upstream Node (Expression)

**Configuration:**
```json
{
  "map": [
    { "key": "customerId", "value": "{{ $node['webhook'].output.body.id }}" }
  ]
}
```

The `value` field is an expression that resolves at run time, capturing the customer ID from the webhook output and storing it under `customerId`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Store a value for later steps
```

### Sample Workflow: Capture an ID, Use It Later

```json
{
  "nodes": [
    { "id": "webhook", "type": "webhook" },
    {
      "id": "set-vars",
      "type": "set-variable",
      "config": {
        "map": [
          { "key": "customerId", "value": "{{ $node['webhook'].output.body.id }}" }
        ]
      }
    },
    {
      "id": "build-message",
      "type": "function",
      "config": {
        "code": "return { text: `Processing customer ${variables.customerId}` };"
      }
    }
  ]
}
```

**How it flows:**
1. The **webhook** emits a request body containing `id`.
2. **Set Variable** stores it as `customerId` and forwards the webhook payload unchanged.
3. The **Function** node reads `variables.customerId` to build a message.

### Common Patterns

- **Run-scoped state:** stash a value once and read it from multiple branches
- **Naming derived data:** compute a value with an expression and store it under a friendly key
- **Feature flags / counters:** keep simple booleans or numbers that other nodes consult

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### A downstream node can't see my variable

**Cause:** Variables live in the shared store, not on the output payload. Reading the `success` output gives you the original input, not the variables.

**Solution:** Read the variable from the variable store downstream (e.g. `variables.<key>` inside a **Function** node), and make sure the Set Variable node runs **before** the consumer in the graph.

#### My value was stored as a string when I expected an object/number

**Cause:** The value was provided as a JSON string rather than a real value.

**Solution:** Provide the native value directly (e.g. `3`, `true`, `{ ... }`). The `value` field accepts any type, so no string wrapping is needed.

#### The variable is unexpectedly overwritten

**Cause:** The same `key` was set again later in the run (in this node's map or another Set Variable node).

**Solution:** Use distinct keys, or order the nodes so the intended final value is written last.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Read stored variables via the `variables` object and run custom logic
- [Merge](./merge.md) – Combine data streams before storing a result
- [Filter](./filter.md) – Filter items before deciding what to store

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-06-30 | `value` now accepts any type and supports expressions; input passes through unchanged; node moved to the builtin package. |
| 1.0.0 | 2026-06-02 | Initial release. |

<!-- /SECTION: changelog -->

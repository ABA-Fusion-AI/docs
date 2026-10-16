---
node_id: "if-else"
title: "If-Else"
description: "Routes workflow data down True or False branches based on a JavaScript conditional expression."
category: "orchestration"
subcategory: "flow-control"
version: "1.0.0"
language: "en"
last_updated: "2026-10-16"
author: "Fusion Team"
tags:
  - flow-control
  - conditional
  - branch
  - routing
  - if-else
  - logic
related_nodes:
  - switch
  - filter
  - function
  - manual-trigger
---

<!-- SECTION: overview -->
# If-Else

> **Category:** Orchestration&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Utility Node

The **If-Else** node provides binary conditional branching for your workflows. It evaluates a custom JavaScript expression against the incoming payload and routes the data down either the **True** or **False** branch without altering the original payload structure.

If the conditional expression encounters an evaluation error (such as a syntax error or a null property dereference), execution is routed safely to the **Error** output.

### Key Features

- **Binary Branching:** Direct data flow to either the `true` or `false` output port based on expression truthiness.
- **Payload Preservation:** Transmits the incoming data object and session intact down the chosen branch.
- **Isolated V8 Sandbox:** Evaluates expressions securely in a dedicated VM isolate with memory and execution timeout safeguards.
- **Rich Context Access:** Access the current item via `input`, upstream node results via `outputs`, workflow variables via `variables`, and secrets via `secrets`.
- **Safe Error Handling:** Traps runtime evaluation exceptions and routes error metadata to the `error` port instead of terminating the workflow.

### Use Cases

- **Threshold Alerting:** Route sensor or metric readings to an alert channel only if values exceed an upper limit (e.g. `input.temperature > 80`).
- **Validation Gates:** Check whether required payload fields are present before proceeding with database insertions or API mutations.
- **VIP Customer Routing:** Divert high-priority support tickets or purchase orders to dedicated processing queues.
- **Feature Flagging & Environment Checks:** Check global workflow variables to switch execution between staging endpoints and production systems.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `condition` | `string` | ❌ No | `"input > 10"` | A JavaScript expression that evaluates to a truthy or falsy value. Evaluated inside a secure sandbox. |

### Available Context Variables

Inside the `condition` expression, the following context variables are available:

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `input` | `any` | The payload arriving on the node's `input` port from the immediate upstream node. | `input.test > 3` |
| `outputs` | `object` | Dictionary containing the output payloads emitted by previously executed upstream nodes. | `outputs.Function.success.test > 3` |
| `variables` | `object` | Key-value dictionary of workflow variables configured in the workflow settings. | `variables.env === "production"` |
| `secrets` | `object` | Key-value dictionary of decrypted workflow secrets accessible to this workflow. | `secrets.ENABLE_BETA === "true"` |

> [!TIP]
> **Prefer `input` over `outputs` for direct connections:** When connecting an upstream node directly to **If-Else**, use `input.<property>` (e.g., `input.test > 3`). It is faster, less verbose, and immune to upstream node renaming.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The incoming data payload to evaluate and forward. Accepts objects, arrays, numbers, or strings. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `true` | `any` | Emitted with the original `input` payload when `condition` evaluates to a truthy value. |
| `false` | `any` | Emitted with the original `input` payload when `condition` evaluates to a falsy value. |
| `error` | `Error` | Emitted when an exception occurs during condition evaluation (syntax errors, null dereference, timeouts). |

### Output Data Behavior

When evaluating successfully, the **If-Else** node acts as a pass-through router. It does not modify, wrap, or serialize the payload:

- If `condition` is **truthy**, the exact incoming `{ session, data }` is emitted on `true`.
- If `condition` is **falsy**, the exact incoming `{ session, data }` is emitted on `false`.

### Output Schema (`error`)

If the condition fails to evaluate, error details are emitted on the `error` port:

```json
{
  "name": "TypeError",
  "message": "Cannot read properties of undefined (reading 'test')"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Conditional Data Routing with If-Else
```

### How it flows

1. **Manual Trigger Node:** Starts the workflow on demand when clicked by the user.
2. **Function Node:** Runs custom JavaScript code returning `{ test: 5 }`:
   ```javascript
   const test = 5;
   return { test };
   ```
3. **If-Else Node:** Evaluates the condition `input.test > 3`. Since `5 > 3` evaluates to `true`, the payload is forwarded down the **True** branch.
4. **Log Nodes:**
   - **Log (True):** Receives and displays `{ test: 5 }` because the condition is met.
   - **Log2 (False):** Remains idle as the condition did not evaluate to falsy.
   - **Log1 (Error):** Monitors the error port and catches any unexpected evaluation exceptions.

### Common Patterns

- **Data Validation & Sanitization:** `Webhook` → `If-Else (Boolean(input.email))` → `True: Save User` / `False: Log Bad Request`.
- **Status Code Branching:** `HTTP Request` → `If-Else (input.status === 200)` → `True: Process Response` / `False: Retry Flow`.
- **Inventory Threshold:** `Fetch Inventory` → `If-Else (input.stockLevel < 10)` → `True: Trigger Reorder Notification`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `TypeError: Cannot read properties of undefined (reading '<prop>')`
- **Cause:** The expression attempted to access a property on an undefined object (for example, misspelling `outputs.Function.sucess` instead of `outputs.Function.success`, or accessing a field that does not exist on `input`).
- **Solution:**
  1. For direct connections, use `input.<property>` directly instead of navigating `outputs`.
  2. Use optional chaining in your condition: `input?.test > 3`.
  3. Ensure that upstream nodes actually output the expected property names.

#### Data flows to `False` unexpectedly
- **Cause:** In JavaScript, the values `0`, `""` (empty string), `null`, `undefined`, `NaN`, and `false` are falsy. If your condition checks a property that contains `0` or an empty string without explicit type checks, it will evaluate to `false`.
- **Solution:** Use explicit comparison operators when checking numbers or strings (e.g. `input.count !== undefined && input.count >= 0` rather than `Boolean(input.count)`).

#### Expression Timeout
- **Cause:** Complex evaluations or infinite loops inside the condition exceed the 1,000 ms execution sandbox limit (`EVAL_TIMEOUT_MS`).
- **Solution:** Keep condition expressions lightweight and focused purely on boolean evaluation. Heavy transformations should be performed upstream inside a **Function** node.

### Error Codes

| Error / Message | Cause | Solution |
|-----------------|-------|----------|
| `TypeError: Cannot read properties of undefined` | Dereferencing an uninitialized or misspelled object key | Check property names on `input` and use optional chaining (`?.`) |
| `Script execution timed out.` | Expression took longer than 1,000 ms to compute | Simplify expression logic; avoid complex loops in conditions |
| `SyntaxError: Unexpected token` | Invalid JavaScript syntax in condition expression | Ensure valid expression syntax (e.g. `===` instead of `=`) |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Switch](/docs/switch) – Route data across multiple named branches based on distinct values
- [Filter](/docs/filter) – Filter arrays and event streams matching specific criteria
- [Function](/docs/function) – Execute multi-line JavaScript logic and complex payload transformations
- [Manual Trigger](/docs/manual-trigger) – Trigger workflows on demand for testing conditional logic

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-16 | Initial release of If-Else node documentation |

<!-- /SECTION: changelog -->

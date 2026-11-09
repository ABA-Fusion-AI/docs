---
node_id: "stop-and-error"
title: "Stop and Error"
description: "Stops the workflow and marks it failed with your error."
category: "orchestration"
subcategory: "Flow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-09"
author: "Fusion Team"
tags:
  - error
  - stop
  - flow control
  - validation
related_nodes:
  - error-trigger
  - if-else
  - no-op
---

<!-- SECTION: overview -->
# Stop and Error

> **Category:** Orchestration &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Stop the whole workflow, on purpose, with an error you write. When an item reaches this node, the workflow fails: every trigger stops, its status turns to **error** with your message, and it **stays stopped** until someone runs it again. If the workflow has an **error workflow**, that workflow's Error Trigger is told.

### Use Cases

- **Guard rails:** stop when data is impossible (a negative total, a missing customer id).
- **Give up after a failure:** connect a node's error output here to stop the workflow instead of carrying on.
- **Custom errors for alerting:** send a structured error (a code, a customer, a link) to your error workflow.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `errorType` | `message \| object` | Yes | `message` | Fail with a text, or with an object. |
| `message` | `string` | message | — | The error message. Supports expressions, e.g. `Order {{ $node.Webhook.body.id }} has a negative total`. |
| `errorObject` | `object` | object | — | The error as an object, e.g. `{ "code": "E42", "message": "Out of stock" }`. Its `message` becomes the workflow's error; the whole object reaches the error workflow as `details`. Supports expressions. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The first item to arrive stops the workflow. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `error` | `object` | Shows the error in the logs. It cannot be connected: nothing runs after a stop. |

### Output Schema

```json
{
  "name": "StopAndError",
  "message": "Out of stock",
  "details": { "code": "E42", "message": "Out of stock" }
}
```

`details` is present only for `errorType: object`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### How it flows

1. **Webhook** receives an order.
2. **If/Else** checks `total >= 0`.
3. **false →** **Stop and Error** with `Order {{ $node.Webhook.body.id }} has a negative total`.
4. The workflow stops with that message; the error workflow posts it to Slack.

### Common Patterns

- **Validate then stop:** Trigger → If/Else → (false) Stop and Error
- **Stop on a failed call:** HTTP Request → (error) Stop and Error

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The workflow does not restart on its own
- **Cause:** By design. A workflow stopped by Stop and Error is kept stopped, so it cannot fail and restart forever.
- **Solution:** Fix the cause, then run it again.

#### Items after the first did nothing
- **Cause:** The first item stops the workflow; the others are dropped with it.

#### I only want to drop one item, not stop everything
- **Solution:** Use [No Operation](/docs/no-op) (or simply leave the branch unconnected) instead.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Error Trigger](/docs/error-trigger) – Hear about the stop in another workflow
- [If/Else](/docs/if-else) – Decide when to stop
- [No Operation](/docs/no-op) – End a branch without stopping anything

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-09 | Initial release |

<!-- /SECTION: changelog -->

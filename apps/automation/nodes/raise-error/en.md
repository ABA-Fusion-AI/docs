---
node_id: "raise-error"
title: "Raise Error"
description: "Fails the workflow with your error: it stops and stays stopped."
category: "orchestration"
subcategory: "Workflow Control"
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
# Raise Error

> **Category:** Orchestration &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Fail the whole workflow, on purpose, with an error you write: the node throws. When an item reaches it, the workflow fails: every trigger stops, its status turns to **error** with your message, and it **stays stopped** until someone runs it again. Error Triggers watching the workflow are told.

To stop a workflow **without** an error, use [Stop Workflow](/docs/stop-workflow).

### Use Cases

- **Guard rails:** stop when data is impossible (a negative total, a missing customer id).
- **Give up after a failure:** connect a node's error output here to stop the workflow instead of carrying on.
- **Custom errors for alerting:** send a structured error (a code, a customer, a link) to the Error Triggers watching this workflow.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `errorType` | `message \| object` | Yes | `message` | Fail with a text, or with an object. |
| `message` | `string` | message | — | The error message. Supports expressions, e.g. `Order {{ $node.Webhook.body.id }} has a negative total`. |
| `errorObject` | `object` | object | — | The error as an object, e.g. `{ "code": "E42", "message": "Out of stock" }`. Its `message` becomes the workflow's error; the whole object reaches watching Error Triggers as `details`. Supports expressions. |

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
  "name": "RaiseError",
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
3. **false →** **Raise Error** with `Order {{ $node.Webhook.body.id }} has a negative total`.
4. The workflow stops with that message; an Error Trigger watching it posts it to Slack.

### Common Patterns

- **Validate then fail:** Trigger → If/Else → (false) Raise Error
- **Fail on a failed call:** HTTP Request → (error) Raise Error

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The workflow does not restart on its own
- **Cause:** By design. A workflow failed by Raise Error is kept stopped, so it cannot fail and restart forever.
- **Solution:** Fix the cause, then run it again.

#### Items after the first did nothing
- **Cause:** The first item stops the workflow; the others are dropped with it.

#### I only want to drop one item, not stop everything
- **Solution:** Use [No Operation](/docs/no-op) (or simply leave the branch unconnected) instead.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Error Trigger](/docs/error-trigger) – Hear about the failure in another workflow
- [Stop Workflow](/docs/stop-workflow) – Stop a workflow without an error
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

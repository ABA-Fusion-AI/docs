---
node_id: "error-trigger"
title: "Error Trigger"
description: "Starts this workflow when a workflow that uses it as its error workflow fails."
category: "triggers"
subcategory: "Error Handling"
version: "1.0.0"
language: "en"
last_updated: "2026-11-09"
author: "Fusion Team"
tags:
  - trigger
  - error
  - alerting
  - monitoring
related_nodes:
  - stop-and-error
  - schedule-trigger
  - no-op
---

<!-- SECTION: overview -->
# Error Trigger

> **Category:** Triggers &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Build one workflow that hears about failures in others — and alert someone, open a ticket or retry. A workflow that holds an **Error Trigger** is an **error workflow**. Any workflow can name it in its settings, under **Error workflow**; when that workflow fails, the Error Trigger starts with a report of what went wrong.

### What counts as a failure

- **A node error nothing handles:** a node emits on its `error` output and that output is **not connected**. Connect the error output to handle it yourself, and the error workflow is not told.
- **The workflow failing:** a **Stop and Error** node, a node that cannot be set up, or a crash that stops the workflow.

### Use Cases

- **Alerting:** post every failure to Slack or email with the workflow, node and message.
- **One place for all errors:** many workflows name the same error workflow.
- **Tickets:** open an issue for each workflow stopped by a Stop and Error.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

The node has no settings. To use it:

1. Create a workflow (say *Alerts*) starting with an **Error Trigger**, followed by whatever should happen.
2. **Start** it. It must be running to hear about failures.
3. Open the settings (⚙) of each workflow to watch, and pick *Alerts* under **Error workflow**. Only workflows with an Error Trigger are offered.

A workflow cannot be its own error workflow. Failures are limited to 10 reports per workflow per minute, so a node failing on every tick of a fast trigger cannot flood the error workflow.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it has no data inputs.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | One report per failure. |
| `error` | `object` | Emitted if the engine cannot deliver reports at all. |

### Output Schema

```json
{
  "workflow": { "id": "6730c1f2…", "name": "Orders sync" },
  "execution": {
    "runId": "4f1c…",
    "sessionId": "session-1762689600000-…",
    "mode": "node",
    "time": "2026-11-09T12:00:00.000Z",
    "error": { "name": "AxiosError", "message": "Request failed with status code 502", "status": 502 },
    "lastNodeExecuted": "Fetch orders",
    "node": { "id": "node-3", "label": "Fetch orders", "type": "http-request" }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `workflow.id` / `workflow.name` | `string` | The workflow that failed. |
| `execution.mode` | `node \| workflow` | `node`: a node error nothing handled; the workflow keeps running. `workflow`: the workflow itself failed and stopped. |
| `execution.error` | `object` | The error: always a `message`, plus whatever else it carried (`name`, `status`, `code`, or a Stop and Error's `details`). |
| `execution.node` | `object \| null` | The node that failed. `null` for a workflow failure no single node caused. |
| `execution.lastNodeExecuted` | `string \| null` | That node's label. |
| `execution.runId`, `sessionId`, `time` | `string` | Which run and item, and when. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### How it flows

1. **Error Trigger:** fires with the report.
2. **Function:** builds the message from `workflow.name`, `execution.error.message` and `execution.lastNodeExecuted`.
3. **Slack / Email:** sends it.

### Common Patterns

- **Alert:** Error Trigger → Function (format) → Slack
- **Only workflow stops:** Error Trigger → If/Else (`execution.mode == "workflow"`) → Email
- **Ticket:** Error Trigger → HTTP Request (issue tracker API)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Nothing arrives
- **Cause:** The error workflow is not running. Reports are delivered live; one sent while it is stopped is lost.
- **Solution:** Start the error workflow. The settings dialog warns when the chosen one is not running.

#### A handled error did not arrive
- **Cause:** The failing node's `error` output is connected, so the error is handled there.
- **Solution:** That is by design. Leave the error output unconnected to report it, or end your handler with **Stop and Error**.

#### "That workflow has no Error Trigger node"
- **Solution:** Add an Error Trigger to the workflow you want to pick, save it, then choose it again.

#### Some reports are missing during a burst
- **Cause:** More than 10 failures in a minute from one workflow; the rest are dropped.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Stop and Error](/docs/stop-and-error) – Fail a workflow on purpose, with your own message
- [No Operation](/docs/no-op) – End a branch on purpose

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-09 | Initial release |

<!-- /SECTION: changelog -->

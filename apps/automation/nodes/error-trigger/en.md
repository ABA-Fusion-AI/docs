---
node_id: "error-trigger"
title: "Error Trigger"
description: "Starts this workflow when a workflow it watches fails."
category: "triggers"
subcategory: "Workflow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-10"
author: "Fusion Team"
tags:
  - trigger
  - error
  - alerting
  - monitoring
related_nodes:
  - raise-error
  - stop-workflow
  - workflow-info
---

<!-- SECTION: overview -->
# Error Trigger

> **Category:** Workflow Control &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Build one workflow that hears about failures in others, then alert someone, open a ticket or start a fallback. Pick the workflows to watch; when one of them fails, the Error Trigger starts with a report of what went wrong.

### What counts as a failure

- **A node error nothing handles:** a node emits on its `error` output and that output is **not connected**. The watched workflow keeps running. Connect the error output to handle it yourself, and nothing is reported.
- **The workflow failing:** a **Raise Error** node, a node that cannot be set up, or a crash that stops the workflow.

Stopping a workflow on purpose (the Stop button, or a **Stop Workflow** node) is not a failure.

### Use Cases

- **Alerting:** post every failure to Slack or email with the workflow, node and message.
- **One place for many workflows:** one Error Trigger can watch them all.
- **Fallbacks:** when the main sync fails, **Start Workflow** a backup one.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `workflows` | list of workflows | Yes | — | The workflows to watch. Each is picked from a searchable list of the workflows you can access. |

### Good to know

- **This workflow must be running** to hear about failures. A report sent while it is stopped is lost.
- **Access:** only workflows you can access are offered. When the workflow starts, each watched workflow is checked again as its owner; any they cannot access is left out, and the Error Trigger says so on its error output.
- **Tenant:** an Error Trigger only ever hears workflows in its own tenant.
- **Rate limit:** at most 10 reports a minute per watched workflow, so a node failing on every tick of a fast trigger cannot flood you.

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
| `error` | `object` | Watched workflows that were left out (no access), or an engine that cannot deliver reports. |

### Output Schema

```json
{
  "workflow": { "id": "6730c1f2…", "name": "Orders sync" },
  "execution": {
    "runId": "4f1c…",
    "sessionId": "session-1762689600000-…",
    "mode": "node",
    "time": "2026-11-10T12:00:00.000Z",
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
| `execution.error` | `object` | Always a `message`, plus whatever else the error carried (`name`, `status`, `code`, or a Raise Error's `details`). |
| `execution.node` | `object \| null` | The node that failed; `null` for a failure no single node caused. |
| `execution.lastNodeExecuted` | `string \| null` | That node's label. |
| `execution.runId`, `sessionId`, `time` | `string` | Which run and item, and when. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### How it flows

1. **Error Trigger** watching *Orders sync* and *Billing* fires with the report.
2. **Function** builds a message from `workflow.name`, `execution.error.message` and `execution.lastNodeExecuted`.
3. **Slack / Email** sends it.

### Common Patterns

- **Alert:** Error Trigger → Function (format) → Slack
- **Only stops:** Error Trigger → If/Else (`execution.mode == "workflow"`) → Email
- **Fallback:** Error Trigger → Start Workflow (backup)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Nothing arrives
- **Cause:** This workflow is not running. Reports are delivered live.
- **Solution:** Start it.

#### A handled error did not arrive
- **Cause:** The failing node's `error` output is connected, so the error is handled there. That is by design.

#### "Not watching N workflow(s)"
- **Cause:** This workflow's owner cannot access them, or they were deleted.
- **Solution:** Share them with the owner (through a project), or remove them from the list.

#### Some reports are missing during a burst
- **Cause:** More than 10 failures in a minute from one workflow; the rest are dropped.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Raise Error](/docs/raise-error) – Fail a workflow on purpose, with your own message
- [Stop Workflow](/docs/stop-workflow) – Stop a workflow without an error
- [Workflow Info](/docs/workflow-info) – Read a workflow's status

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-10 | Initial release |

<!-- /SECTION: changelog -->

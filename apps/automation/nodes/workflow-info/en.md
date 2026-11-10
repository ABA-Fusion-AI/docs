---
node_id: "workflow-info"
title: "Workflow Info"
description: "Outputs a workflow's name, status, status message, owner and dates."
category: "orchestration"
subcategory: "Workflow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-10"
author: "Fusion Team"
tags:
  - workflow control
  - status
related_nodes:
  - error-trigger
  - stop-workflow
---

<!-- SECTION: overview -->
# Workflow Info

> **Category:** Workflow Control &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read a workflow: this workflow, or another one you pick. Outputs its name, status (running, stopped, paused, error), the status message, its owner, its project and its dates.

### Use Cases

- **Health checks:** on a schedule, check that an important workflow is still running.
- **Alerts with context:** add the workflow's status to a message.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `target` | `this \| another` | Yes | `this` | This workflow, or another one. |
| `workflow` | workflow | When `target` is `another` | — | The workflow to act on. |

### Who may act on which workflow

The workflow is picked from a searchable list of the workflows you can access. When the node runs, the platform checks again, **as this workflow's owner**: a workflow the owner cannot access (or that was deleted) is treated as not found, and the node goes out its **error** output. Tools are not listed.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Each item reads the workflow. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The workflow's details (below). The item itself is not passed on. |
| `error` | `object` | The workflow was not found, the owner cannot access it, or the action was refused. |

### Output Schema

```json
{
  "id": "6730c1f2…",
  "name": "Orders sync",
  "status": "running",
  "desiredStatus": "running",
  "statusMessage": "Workflow started by user.",
  "statusType": "success",
  "ownerId": "8b0f…",
  "projectId": "c1d2…",
  "createdAt": "2026-11-01T09:00:00.000Z",
  "updatedAt": "2026-11-10T12:00:00.000Z"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Common Patterns

- **Health check:** Schedule (every 5 min) → Workflow Info (sync) → If/Else (`status != "running"`) → Email

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "That workflow does not exist, or this workflow's owner cannot access it"
- **Cause:** The picked workflow was deleted, or isn't shared with this workflow's owner.
- **Solution:** Share it with the owner through a project, or pick another.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Error Trigger](/docs/error-trigger) – Start when watched workflows fail
- [Stop Workflow](/docs/stop-workflow) – Stop a workflow cleanly

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-10 | Initial release |

<!-- /SECTION: changelog -->

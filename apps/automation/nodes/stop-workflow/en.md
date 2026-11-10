---
node_id: "stop-workflow"
title: "Stop Workflow"
description: "Stops this workflow or another one, cleanly (no error). It stays stopped until run again."
category: "orchestration"
subcategory: "Workflow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-10"
author: "Fusion Team"
tags:
  - workflow control
  - stop
related_nodes:
  - pause-workflow
  - raise-error
  - workflow-info
---

<!-- SECTION: overview -->
# Stop Workflow

> **Category:** Workflow Control &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Stop a workflow on purpose, **without an error**: this workflow, or another one you pick. The status becomes **stopped** with a message saying which node did it, and why: *Stopped by "Quota guard" in "Orders sync": daily quota reached.* It stays stopped until someone runs it again; nothing restarts it.

On **this workflow**, the item goes out first, then the workflow stops.

To fail a workflow **with** an error (status error, Error Triggers told), use [Raise Error](/docs/raise-error).

### Use Cases

- **Quotas:** stop when today's limit is reached.
- **One-shot jobs:** a workflow that should stop itself after its work is done.
- **Kill switch:** stop a noisy workflow from another one.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `target` | `this \| another` | Yes | `this` | This workflow, or another one. |
| `workflow` | workflow | When `target` is `another` | — | The workflow to act on. |
| `reason` | `string` | No | — | Shown in the workflow's status, e.g. "Daily quota reached". Supports expressions. |

### Who may act on which workflow

The workflow is picked from a searchable list of the workflows you can access. When the node runs, the platform checks again, **as this workflow's owner**: a workflow the owner cannot access (or that was deleted) is treated as not found, and the node goes out its **error** output. Tools cannot be stopped.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Each item stops the workflow. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The item, unchanged, once the stop was accepted. |
| `error` | `object` | The workflow was not found, the owner cannot access it, or the action was refused. |


<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Common Patterns

- **Stop when done:** Trigger → … → Stop Workflow (this)
- **Stop another:** Schedule → Workflow Info (other) → If/Else (too many errors) → Stop Workflow (other)

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

- [Pause Workflow](/docs/pause-workflow) – Pause a workflow
- [Raise Error](/docs/raise-error) – Fail this workflow with an error
- [Workflow Info](/docs/workflow-info) – Read a workflow's status

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-10 | Initial release |

<!-- /SECTION: changelog -->

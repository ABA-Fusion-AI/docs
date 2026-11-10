---
node_id: "pause-workflow"
title: "Pause Workflow"
description: "Pauses this workflow or another one."
category: "orchestration"
subcategory: "Workflow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-10"
author: "Fusion Team"
tags:
  - workflow control
  - pause
related_nodes:
  - resume-workflow
  - stop-workflow
---

<!-- SECTION: overview -->
# Pause Workflow

> **Category:** Workflow Control &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Pause a workflow: this workflow, or another one you pick. Its triggers stop firing until it is resumed, from its page or with a [Resume Workflow](/docs/resume-workflow) node in another workflow. The status says which node paused it, and why.

On **this workflow**, the item goes out first, then the workflow pauses. A paused workflow cannot resume itself.

### Use Cases

- **Maintenance windows:** pause a sync while a system is down, resume it after.
- **Wait for a person:** pause, and resume when someone approves.

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

The workflow is picked from a searchable list of the workflows you can access. When the node runs, the platform checks again, **as this workflow's owner**: a workflow the owner cannot access (or that was deleted) is treated as not found, and the node goes out its **error** output. Tools cannot be paused.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Each item pauses the workflow. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The item, unchanged, once the pause was accepted. |
| `error` | `object` | The workflow was not found, the owner cannot access it, or the action was refused. |


<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Common Patterns

- **Maintenance:** Schedule (22:00) → Pause Workflow (sync); Schedule (06:00) → Resume Workflow (sync)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "That workflow does not exist, or this workflow's owner cannot access it"
- **Cause:** The picked workflow was deleted, or isn't shared with this workflow's owner.
- **Solution:** Share it with the owner through a project, or pick another.

#### It stays paused
- **Cause:** A workflow cannot resume itself. Resume it from its page, or from another workflow.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Resume Workflow](/docs/resume-workflow) – Resume a paused workflow
- [Stop Workflow](/docs/stop-workflow) – Stop a workflow cleanly

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-10 | Initial release |

<!-- /SECTION: changelog -->

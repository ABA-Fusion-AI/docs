---
node_id: "start-workflow"
title: "Start Workflow"
description: "Starts another workflow, as if its Run button were pressed."
category: "orchestration"
subcategory: "Workflow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-10"
author: "Fusion Team"
tags:
  - workflow control
  - start
related_nodes:
  - stop-workflow
  - error-trigger
---

<!-- SECTION: overview -->
# Start Workflow

> **Category:** Workflow Control &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Start another workflow, exactly as its **Run** button would: the same checks (secrets chosen, nothing started on its own) and the same recorded intent, so it keeps running. The status says which node started it.

### Use Cases

- **Fallbacks:** an Error Trigger starts a backup workflow when the main one fails.
- **Chains:** start the next stage when this one is done.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `workflow` | workflow | Yes | — | The workflow to act on. This workflow is not a choice. |
| `reason` | `string` | No | — | Shown in the workflow's status, e.g. "Daily quota reached". Supports expressions. |

### Who may act on which workflow

The workflow is picked from a searchable list of the workflows you can access. When the node runs, the platform checks again, **as this workflow's owner**: a workflow the owner cannot access (or that was deleted) is treated as not found, and the node goes out its **error** output. Tools cannot be started.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Each item starts the workflow. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The item, unchanged, once the start was accepted. |
| `error` | `object` | The workflow was not found, the owner cannot access it, or the action was refused. |


<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Common Patterns

- **Fallback:** Error Trigger (main) → Start Workflow (backup)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "That workflow does not exist, or this workflow's owner cannot access it"
- **Cause:** The picked workflow was deleted, or isn't shared with this workflow's owner.
- **Solution:** Share it with the owner through a project, or pick another.

#### "Choose which secrets this workflow may use before starting it"
- **Cause:** The target workflow predates secret imports, and its owner has not chosen its secrets yet.
- **Solution:** Its owner opens it and chooses.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Stop Workflow](/docs/stop-workflow) – Stop a workflow cleanly
- [Error Trigger](/docs/error-trigger) – Start when watched workflows fail

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-10 | Initial release |

<!-- /SECTION: changelog -->

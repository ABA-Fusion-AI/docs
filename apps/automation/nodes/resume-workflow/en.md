---
node_id: "resume-workflow"
title: "Resume Workflow"
description: "Resumes a paused workflow."
category: "orchestration"
subcategory: "Workflow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-10"
author: "Fusion Team"
tags:
  - workflow control
  - resume
related_nodes:
  - pause-workflow
  - start-workflow
---

<!-- SECTION: overview -->
# Resume Workflow

> **Category:** Workflow Control &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Resume another workflow that was paused. The status says which node resumed it, and why.

### Use Cases

- **End of a maintenance window.**
- **After an approval.**

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

The workflow is picked from a searchable list of the workflows you can access. When the node runs, the platform checks again, **as this workflow's owner**: a workflow the owner cannot access (or that was deleted) is treated as not found, and the node goes out its **error** output. Tools cannot be resumed.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Each item resumes the workflow. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The item, unchanged, once the resume was accepted. |
| `error` | `object` | The workflow was not found, the owner cannot access it, or the action was refused. |


<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Common Patterns

- **Maintenance:** Schedule (06:00) → Resume Workflow (sync)

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
- [Start Workflow](/docs/start-workflow) – Start another workflow

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-10 | Initial release |

<!-- /SECTION: changelog -->

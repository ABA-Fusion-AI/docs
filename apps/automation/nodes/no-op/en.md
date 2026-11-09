---
node_id: "no-op"
title: "No Operation"
description: "Does nothing: passes each item through unchanged."
category: "orchestration"
subcategory: "Flow Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-09"
author: "Fusion Team"
tags:
  - no-op
  - flow control
  - placeholder
related_nodes:
  - stop-and-error
  - if-else
  - merge
---

<!-- SECTION: overview -->
# No Operation

> **Category:** Orchestration &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

A node that does nothing: every item leaves exactly as it arrived. It makes intent visible on the canvas.

### Use Cases

- **End a branch on purpose:** an If/Else's `false` side that should do nothing, labelled "Ignore drafts".
- **A placeholder:** hold a spot in a workflow you are still building.
- **Pin data:** pin sample output here to test what comes after without running what came before.
- **Tidy wiring:** gather several wires into one point before they fan out again.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

The node has no settings.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Any item. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The same item, unchanged. |
| `error` | `Error` | Never emitted. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Common Patterns

- **Ignore a case:** If/Else → (false) No Operation "Ignore drafts"
- **Placeholder:** Trigger → No Operation "TODO: enrich" → Store

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

Nothing can go wrong in this node: it does nothing.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Stop and Error](/docs/stop-and-error) – Stop the workflow instead
- [If/Else](/docs/if-else) – Split items into branches

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-09 | Initial release |

<!-- /SECTION: changelog -->

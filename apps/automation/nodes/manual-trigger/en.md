---
node_id: "manual-trigger"
title: "Manual Trigger"
description: "Triggers workflow execution manually on demand from the canvas or via API."
category: "triggers"
subcategory: "manual-scheduled"
version: "1.0.0"
language: "en"
last_updated: "2026-10-16"
author: "Fusion Team"
tags:
  - manual
  - trigger
  - on-demand
  - testing
  - execution
  - manual-trigger
related_nodes:
  - cron
  - interval
  - chat-trigger
  - log
---

<!-- SECTION: overview -->
# Manual Trigger

> **Category:** Triggers & Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Triggers workflow execution manually on demand. The **Manual Trigger** node acts as an interactive ingress point that fires whenever an operator explicitly triggers the workflow from the canvas UI or calls the trigger endpoint via the workflow engine API.

Unlike automated trigger nodes (such as **Cron** or **Interval**) that fire based on clock schedules or timers, **Manual Trigger** gives you complete control over when a workflow executes.

### Key Features

- **On-Demand Execution:** Trigger workflows whenever you need without waiting for schedules.
- **Zero Configuration:** Ready to use immediately without configuring complex options or parameters.
- **Execution Tracking:** Emits an incremental `tick` counter and millisecond `timestamp` for every execution.
- **Development & Debugging:** Ideal for prototyping, testing new node configurations, and verifying downstream logic safely.

### Use Cases

- **Interactive Workflow Testing:** Test new workflows and inspect downstream outputs without setting up mock timers or webhooks.
- **On-Demand Operations:** Run maintenance scripts, database syncs, or cache purges whenever required by an operator.
- **Manual Batch Processing:** Initiate batch imports or data exports after manual data validation.
- **Human Gated Automation:** Serve as the starting trigger for operational tasks that require explicit human approval before execution.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

The **Manual Trigger** node requires no configuration parameters. It functions out-of-the-box as an event ingress trigger.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| — | — | — | — | No configurable parameters. The node activates on user interaction. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

None. As a trigger node, **Manual Trigger** represents the root of a workflow graph and does not accept data inputs from upstream nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted each time the trigger is manually activated |
| `error` | `Error` | Emitted if the manual trigger capability is unavailable in the engine environment |

### Output Schema (`success`)

```json
{
  "tick": 1,
  "timestamp": 1792164868073
}
```

| Field | Type | Description |
|-------|------|-------------|
| `tick` | `number` | Incremental execution counter for this node, starting at `1` and incrementing by 1 on each manual trigger event. |
| `timestamp` | `number` | Unix epoch timestamp in milliseconds recorded when the manual trigger fired. |

### Output Schema (`error`)

```json
{
  "message": "Manual trigger capability not available"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `message` | `string` | Error description indicating that the runtime engine does not support interactive manual trigger registration. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Manual Trigger with Console Logger
```

### How it flows

1. **Manual Trigger Node:** Starts the workflow on demand when clicked in the canvas or triggered via API. It generates a payload containing the current `tick` number and millisecond `timestamp`.
2. **Log Node:** Connected to the `success` output of the Manual Trigger. It receives the event payload on its `input` port and logs the trigger details directly to the execution console.

### Common Patterns

- **API Request Testing:** `Manual Trigger` → `HTTP Request` → `Log` (send a single test request to an external API and view the response).
- **Ad-Hoc Data Export:** `Manual Trigger` → `Database Query` → `CSV Generator` → `Email` (manually generate and email a fresh report).
- **Admin Maintenance:** `Manual Trigger` → `Function` → `Cache Invalidation` (manually clear system caches on demand).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The workflow does not fire when clicking the trigger button
- **Cause:** The workflow is currently stopped or paused. Trigger nodes register their event handlers only when the workflow is actively running.
- **Solution:** Click the **Start** button in the canvas toolbar to start the workflow before triggering.

#### Error: `Manual trigger capability not available`
- **Cause:** The execution engine context does not provide the `manualTrigger` resource handler. This can occur in headless environments or test runners where interactive triggers are not registered.
- **Solution:** Verify that your execution host or test environment initializes `externalResources.manualTrigger`.

#### Downstream nodes do not receive data
- **Cause:** The connection handle is misconfigured or connected to an inactive input port.
- **Solution:** Ensure the connection line originates from the `success` output handle of the **Manual Trigger** node and connects directly to the downstream node's input port.

### Error Codes

| Message | Cause | Solution |
|---------|-------|----------|
| `Manual trigger capability not available` | The runtime engine environment lacks `manualTrigger` external resource support | Ensure the workflow is running in an environment that supports interactive trigger events |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Cron](/docs/cron) – Schedule workflows to execute automatically at specific dates and times
- [Interval](/docs/interval) – Trigger workflows repeatedly at fixed time intervals
- [Chat Trigger](/docs/chat-trigger) – Start workflows interactively from user messages in the chat interface
- [Log](/docs/log) – Inspect triggered payload outputs and downstream data in the console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-16 | Initial release of Manual Trigger documentation |

<!-- /SECTION: changelog -->

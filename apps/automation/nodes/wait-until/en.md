---
node_id: "wait-until"
title: "Wait Until"
description: "Hold an incoming event until a specified date and time."
category: "actions"
subcategory: "Timing & Rate Control"
version: "1.0.0"
language: "en"
last_updated: "2026-11-16"
author: "Fusion Team"
tags: [schedule, date, wait, email]
related_nodes: [delay, cron]
---

<!-- SECTION: overview -->
# Wait Until

Receives an event, waits until its date and time, then forwards the original data and session. Only this execution path waits; other inputs and branches continue independently. This is an action, not a recurring trigger.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Field | Meaning |
| --- | --- |
| `dateTime` | Optional ISO date-time. Map a previous node's output using the expression picker. When empty, reads `input.dateTime`, or the input itself if it is a string. |

Include a timezone: `2026-12-01T09:00:00+01:00` or `2026-12-01T08:00:00Z`. Timezone-less dates, natural language, numbers, missing values, and invalid calendar dates produce an error. Past dates continue immediately.

The timezone offset must be appropriate for the intended date. The node does not extract dates from email prose; parse the email first, then map the resulting ISO date-time.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

`input` accepts the preceding node's event. `success` emits the unchanged input once its deadline arrives. `error` reports invalid dates while preserving the input session. Each event has its own deadline; events may complete in a different order from arrival.

Pausing retains pending events and their original deadlines. Resuming releases overdue events. Stopping cancels pending events. **Waiting is in memory and does not survive engine restarts, redeployment, or workflow stop.** Keep the workflow and engine running. This node does not persist scheduled jobs or restart a stopped workflow.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Mail Example

Mail → Extract date and time → Wait Until → Next action.

Send this object to Wait Until with its `dateTime` configuration empty:

```json
{
  "dateTime": "2026-12-01T09:00:00+01:00",
  "email": "user@example.com",
  "task": "Follow up"
}
```

The same object is forwarded at the specified time. Alternatively, map an extracted date field into the node's Date and Time setting while retaining the entire incoming mail payload.
<!-- /SECTION: examples -->

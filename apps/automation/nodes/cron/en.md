---
node_id: "cron"
title: "Cron"
description: "Triggers workflow execution on a cron schedule."
category: "triggers"
subcategory: "Scheduling"
version: "1.1.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - trigger
  - cron
  - schedule
  - automation
  - timing
related_nodes:
  - interval
  - manual-trigger
  - delay
---

<!-- SECTION: overview -->
# Cron

> **Category:** Triggers &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Trigger a workflow automatically based on a cron schedule expression. The **Cron** node fires at precise points in time — every few seconds, hourly, daily, weekly, or on any custom schedule you define.

### Use Cases

- **Scheduled Reports:** Generate and send daily or weekly reports at a fixed time.
- **Data Sync:** Pull data from external APIs every hour and update internal databases.
- **Periodic Health Checks:** Ping external services every few minutes and alert on failure.
- **Monthly Processing:** Run invoice generation or billing jobs on the 1st of each month.
- **Business Hours Automation:** Trigger workflows only on weekdays during office hours.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `schedule` | `string` | Yes | — | A cron expression that defines when the workflow fires. Supports both 5-field (minute-level) and 6-field (second-level) formats. |

### Cron Expression Format

The node supports two formats:

**5-field (standard) — minute precision:**
```
*  *  *  *  *
│  │  │  │  └── Day of week  (0–7, 0 and 7 = Sunday)
│  │  │  └───── Month        (1–12)
│  │  └──────── Day of month (1–31)
│  └─────────── Hour         (0–23)
└────────────── Minute       (0–59)
```

**6-field — second precision:**
```
*  *  *  *  *  *
│  │  │  │  │  └── Day of week  (0–7)
│  │  │  │  └───── Month        (1–12)
│  │  │  └──────── Day of month (1–31)
│  │  └─────────── Hour         (0–23)
│  └──────────────  Minute      (0–59)
└─────────────────  Second      (0–59)
```

### Schedule Reference

| Type | Cron Expression | Description |
|------|-----------------|-------------|
| Every X Seconds | `*/10 * * * * *` | Every 10 seconds. |
| Every X Minutes | `*/5 * * * *` | Every 5 minutes. |
| Hourly | `0 * * * *` | Every hour on the hour. |
| Daily | `0 6 * * *` | At 6:00 AM every day. |
| Weekly | `0 12 * * 1` | At noon every Monday. |
| Monthly | `0 0 1 * *` | At midnight on the 1st of every month. |
| Every X Days | `0 0 */3 * *` | At midnight every 3rd day. |
| Only Weekdays | `0 9 * * 1-5` | At 9:00 AM Monday through Friday. |
| Custom Hourly Range | `0 9-17 * * *` | Every hour from 9:00 AM to 5:00 PM. |
| Quarterly | `0 0 1 1,4,7,10 *` | At midnight on the 1st of Jan, Apr, Jul, and Oct. |

### Special Characters

| Character | Meaning | Example |
|-----------|---------|---------|
| `*` | Every unit | `* * * * *` = every minute |
| `*/n` | Every n units | `*/5 * * * *` = every 5 minutes |
| `n-m` | Range | `1-5` = values 1 through 5 |
| `n,m` | List | `1,4,7` = values 1, 4, and 7 |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it has no data inputs. It fires autonomously based on the configured schedule.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted each time the cron schedule fires. |
| `error` | `Error` | Emitted if the schedule expression is invalid. |

### Output Schema

```json
{
  "timestamp": "2026-08-06T09:00:00.000Z",
  "schedule": "0 9 * * *"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | `string` | ISO 8601 timestamp of when the trigger fired. |
| `schedule` | `string` | The cron expression that triggered this execution. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Cron Trigger Every 3 Seconds
```

### How it flows

1. **Cron Node:** Fires every 3 seconds using the 6-field expression `*/3 * * * * *`.
2. **Function Node:** Runs custom JavaScript that returns a message and the current timestamp.
3. **Log Node:** Displays the result on every execution.

**Function code used in the example:**
```js
return {
  message: "Cron Trigger Works!",
  executedAt: new Date().toISOString()
};
```

### Common Patterns

- **Daily Report:** `Cron (0 9 * * *)` → Query Database → Format → Send Email
- **Hourly Monitor:** `Cron (0 * * * *)` → HTTP Request → Check Status → Alert on Failure
- **Weekly Cleanup:** `Cron (0 0 * * 1)` → Find Old Records → Delete → Log
- **Rate-Paced Batch:** `Cron (*/30 * * * *)` → Fetch Batch → Process → Store

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Trigger does not fire at the expected time
- **Cause:** Timezone mismatch — all times are relative to the server's configured timezone.
- **Solution:** Adjust the cron expression to account for the timezone offset, or verify the server timezone.

#### `Invalid cron expression` error
- **Cause:** The `schedule` value has incorrect syntax, wrong number of fields, or invalid characters.
- **Solution:** Use the schedule reference table above. Validate your expression at [crontab.guru](https://crontab.guru) before saving.

#### Missed executions after a pause
- **Cause:** Cron does not backfill missed runs after the workflow is paused or stopped.
- **Solution:** If you need to handle missed intervals, use the `timestamp` output field to detect and process them in a downstream Function node.

#### Workflow fires too frequently / unexpectedly
- **Cause:** A 6-field expression is being interpreted as second-level when a minute-level trigger was intended.
- **Solution:** Use 5-field expressions for minute-level schedules. Only add the 6th (seconds) field when sub-minute precision is actually required.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `INVALID_CRON` | Invalid cron schedule | Fix the expression syntax |
| `SCHEDULE_REQUIRED` | Cron schedule is required | Provide a non-empty `schedule` value |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Interval](./interval.md) – Trigger at fixed time intervals without cron syntax
- [Manual Trigger](./manual-trigger.md) – Start a workflow on demand
- [Delay](./delay.md) – Add a timed pause between nodes inside a workflow

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-08-06 | Full rewrite — added schedule reference table, 6-field format, workflow example, and expanded troubleshooting |
| 1.0.0 | 2026-01-31 | Initial release |

<!-- /SECTION: changelog -->

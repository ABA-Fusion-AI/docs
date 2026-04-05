---
node_id: "cron"
title: "Cron"
description: "Triggers workflow execution based on a cron schedule"
category: "triggers"
subcategory: "scheduling"
version: "1.0.0"
language: "en"
last_updated: "2026-01-31"
author: "Fusion Team"
tags:
  - trigger
  - schedule
  - cron
  - automation
  - timing
related_nodes:
  - interval
  - socket-manual-trigger
---

<!-- SECTION: header -->
# Cron

> **Category:** Triggers | **Type:** Trigger Node

Schedule workflow execution using standard cron expressions. Perfect for recurring tasks like daily reports, weekly backups, or monthly data processing.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Cron** node triggers your workflow at specific times based on a cron schedule expression. This is ideal for time-based automation that needs to run at precise intervals—daily at midnight, every Monday at 9 AM, or on the first day of each month.

### Key Features

- **Standard Cron Syntax:** Uses the familiar 5-field cron expression format
- **Precise Scheduling:** Execute at exact times, not just intervals
- **Timezone Support:** Respects server timezone configuration
- **Pause/Resume:** Can be paused without losing schedule configuration

### Use Cases

- Daily data synchronization at 2 AM
- Weekly report generation every Monday morning
- Monthly invoice processing on the 1st of each month
- Hourly health checks for external services

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `schedule` | `string` | ✅ Yes | — | Cron expression defining the execution schedule |

### Cron Expression Format

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-7, 0 and 7 are Sunday)
│ │ │ │ │
* * * * *
```

### Common Schedule Examples

| Schedule | Cron Expression | Description |
|----------|-----------------|-------------|
| Every minute | `* * * * *` | Runs every minute |
| Every hour | `0 * * * *` | Runs at the start of every hour |
| Daily at midnight | `0 0 * * *` | Runs at 00:00 every day |
| Daily at 9 AM | `0 9 * * *` | Runs at 09:00 every day |
| Every Monday at 9 AM | `0 9 * * 1` | Runs at 09:00 every Monday |
| First of month at midnight | `0 0 1 * *` | Runs at 00:00 on the 1st of each month |
| Every 15 minutes | `*/15 * * * *` | Runs every 15 minutes |
| Weekdays at 8 AM | `0 8 * * 1-5` | Runs at 08:00 Monday through Friday |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node and does not accept inputs from other nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Emitted each time the cron schedule fires |

### Output Schema

```json
{
  "timestamp": "2026-01-31T09:00:00.000Z",
  "schedule": "0 9 * * *"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | `string` | ISO 8601 timestamp of when the trigger fired |
| `schedule` | `string` | The cron expression that triggered this execution |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Daily Morning Trigger

Run a workflow every day at 9:00 AM.

**Configuration:**
```json
{
  "schedule": "0 9 * * *"
}
```

**Output:**
```json
{
  "timestamp": "2026-01-31T09:00:00.000Z",
  "schedule": "0 9 * * *"
}
```

### Advanced Example: Business Hours Check

Run every 30 minutes during business hours (9 AM - 5 PM) on weekdays.

**Configuration:**
```json
{
  "schedule": "0,30 9-17 * * 1-5"
}
```

### Advanced Example: End of Month Processing

Run at 11:59 PM on the last day of each month (using day 28-31 with conditions).

**Configuration:**
```json
{
  "schedule": "59 23 28-31 * *"
}
```

> **Note:** For true "last day of month" scheduling, combine with a Filter node to check the date.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Daily Data Sync

```json
{
  "nodes": [
    {
      "id": "cron-trigger",
      "type": "cron",
      "position": { "x": 100, "y": 100 },
      "config": {
        "schedule": "0 2 * * *"
      }
    },
    {
      "id": "fetch-data",
      "type": "http-request",
      "position": { "x": 300, "y": 100 },
      "config": {
        "url": "https://api.example.com/data",
        "method": "GET"
      }
    }
  ],
  "connections": [
    {
      "source": "cron-trigger",
      "sourceOutput": "output",
      "target": "fetch-data",
      "targetInput": "input"
    }
  ]
}
```

### Common Patterns

- **Daily Report:** Cron (0 9 * * *) → Query Data → Format → Send Email
- **Hourly Monitoring:** Cron (0 * * * *) → Check Service → Alert on Failure
- **Weekly Cleanup:** Cron (0 0 * * 0) → Find Old Records → Delete → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Trigger doesn't fire at expected time

**Cause:** Timezone mismatch between server and expected schedule.

**Solution:** Verify the server timezone and adjust your cron expression accordingly. All times are relative to the server's configured timezone.

#### Invalid cron expression error

**Cause:** The cron expression contains invalid syntax.

**Solution:** Verify your expression follows the 5-field format. Use an online cron expression validator to test.

#### Missed executions after pause/resume

**Cause:** Cron schedules don't "catch up" on missed executions.

**Solution:** If you need to process missed intervals, use the timestamp output to determine if catch-up processing is needed.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `INVALID_CRON` | Invalid cron schedule | Check cron expression syntax |
| `SCHEDULE_REQUIRED` | Cron schedule is required | Provide a non-empty schedule value |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Interval](./interval.md) - Trigger at fixed time intervals
- [Socket Manual Trigger](./socket-manual-trigger.md) - Manual workflow execution

### See Also

- [Triggers Category Overview](../index.en.md#triggers)
- [Scheduling Best Practices](../../guides/scheduling.md)

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-31 | Initial release |

<!-- /SECTION: changelog -->

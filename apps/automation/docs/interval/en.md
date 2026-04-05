---
node_id: "interval"
title: "Interval"
description: "Triggers workflow execution at fixed time intervals"
category: "triggers"
subcategory: "scheduling"
version: "1.0.0"
language: "en"
last_updated: "2026-01-31"
author: "Fusion Team"
tags:
  - trigger
  - interval
  - timer
  - polling
  - scheduling
related_nodes:
  - cron
  - delay
  - debounce-time
---

<!-- SECTION: header -->
# Interval

> **Category:** Triggers | **Type:** Trigger Node

Trigger your workflow at regular time intervals. Ideal for polling, monitoring, and periodic data collection tasks.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Interval** node emits a tick at specified time intervals, triggering downstream workflow execution. Unlike Cron which fires at specific times, Interval fires repeatedly based on elapsed time since the workflow started.

### Key Features

- **Simple Configuration:** Just specify the interval duration in milliseconds
- **Continuous Execution:** Keeps firing until workflow is stopped
- **Tick Counter:** Tracks the number of executions
- **Next Tick Prediction:** Provides timestamp of next expected execution

### Use Cases

- Polling an API every 30 seconds for updates
- Monitoring system health every 5 minutes
- Collecting metrics data at regular intervals
- Checking for new messages or notifications

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `duration` | `number` | ❌ No | `1000` | Interval duration in milliseconds |

### Duration Reference

| Interval | Milliseconds | Use Case |
|----------|--------------|----------|
| 1 second | `1000` | Real-time updates |
| 5 seconds | `5000` | Quick polling |
| 30 seconds | `30000` | Moderate polling |
| 1 minute | `60000` | Standard polling |
| 5 minutes | `300000` | Health checks |
| 15 minutes | `900000` | Data sync |
| 1 hour | `3600000` | Low-frequency tasks |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node and does not accept inputs from other nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Emitted each time the interval fires |

### Output Schema

```json
{
  "tick": 1,
  "nextTick": 1706698801000,
  "timestamp": 1706698800000
}
```

| Field | Type | Description |
|-------|------|-------------|
| `tick` | `number` | Current tick count (starts at 1, increments each interval) |
| `nextTick` | `number` | Unix timestamp of when the next tick will occur |
| `timestamp` | `number` | Unix timestamp of the current tick |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Every 5 Seconds

Trigger every 5 seconds for real-time monitoring.

**Configuration:**
```json
{
  "duration": 5000
}
```

**Output (first tick):**
```json
{
  "tick": 1,
  "nextTick": 1706698805000,
  "timestamp": 1706698800000
}
```

**Output (second tick):**
```json
{
  "tick": 2,
  "nextTick": 1706698810000,
  "timestamp": 1706698805000
}
```

### Advanced Example: Every Minute Polling

Poll an external service once per minute.

**Configuration:**
```json
{
  "duration": 60000
}
```

### Advanced Example: High-Frequency Updates

Update every 500ms for near real-time data.

**Configuration:**
```json
{
  "duration": 500
}
```

> **Warning:** Very short intervals can impact performance. Use intervals under 1 second only when necessary.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: API Polling

```json
{
  "nodes": [
    {
      "id": "interval-trigger",
      "type": "interval",
      "position": { "x": 100, "y": 100 },
      "config": {
        "duration": 30000
      }
    },
    {
      "id": "fetch-updates",
      "type": "http-request",
      "position": { "x": 300, "y": 100 },
      "config": {
        "url": "https://api.example.com/updates",
        "method": "GET"
      }
    },
    {
      "id": "check-new",
      "type": "filter",
      "position": { "x": 500, "y": 100 },
      "config": {
        "condition": "input.data && input.data.length > 0"
      }
    }
  ],
  "connections": [
    {
      "source": "interval-trigger",
      "sourceOutput": "output",
      "target": "fetch-updates",
      "targetInput": "input"
    },
    {
      "source": "fetch-updates",
      "sourceOutput": "output",
      "target": "check-new",
      "targetInput": "input"
    }
  ]
}
```

### Common Patterns

- **API Polling:** Interval → HTTP Request → Filter (check for changes) → Process
- **Health Monitor:** Interval → Check Service → Switch (status) → Alert/Log
- **Data Collector:** Interval → Gather Metrics → Aggregate → Store
- **Queue Processor:** Interval → Check Queue → Process Items → Mark Complete

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Interval fires too frequently

**Cause:** Duration value is too low.

**Solution:** Increase the duration value. Remember that the value is in milliseconds, not seconds.

#### Missed ticks after pause/resume

**Cause:** Tick counter doesn't reset on resume; interval restarts fresh.

**Solution:** This is expected behavior. The tick count continues from where it left off, but timing restarts.

#### High CPU usage

**Cause:** Very short intervals combined with heavy processing.

**Solution:** Increase interval duration or optimize downstream node processing. Consider using debounce/throttle nodes.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `INVALID_DURATION` | Duration must be at least 1ms | Set duration to 1 or greater |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Cron](./cron.md) - Schedule-based triggering
- [Delay](../utilities/delay.md) - Add delays between operations
- [Debounce Time](../utilities/debounce-time.md) - Rate limit triggers

### See Also

- [Triggers Category Overview](../index.en.md#triggers)
- [Performance Optimization Guide](../../guides/performance.md)

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-31 | Initial release |

<!-- /SECTION: changelog -->

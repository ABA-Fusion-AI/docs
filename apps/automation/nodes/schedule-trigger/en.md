---
node_id: "schedule-trigger"
title: "Schedule Trigger"
description: "Starts the workflow on a schedule: every N minutes, daily, weekly, monthly or by cron."
category: "triggers"
subcategory: "Scheduling"
version: "1.0.0"
language: "en"
last_updated: "2026-11-09"
author: "Fusion Team"
tags:
  - trigger
  - schedule
  - cron
  - timezone
  - automation
related_nodes:
  - cron
  - interval
  - error-trigger
---

<!-- SECTION: overview -->
# Schedule Trigger

> **Category:** Triggers &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Start a workflow at set times without writing cron by hand. Pick a rule — **every N seconds, minutes or hours**, **daily**, **weekly** or **monthly** at a time — or fall back to your own **cron** expression. Every rule runs in the **timezone** you choose.

### Use Cases

- **Morning report:** daily at 08:30 in your office's timezone.
- **Polling an API:** every 15 minutes, on the clock (:00, :15, :30, :45).
- **Weekly digest:** Mondays and Thursdays at 18:00.
- **Month-end billing:** on day 31, which runs on the last day of every month.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | `interval \| daily \| weekly \| monthly \| cron` | Yes | `interval` | Which kind of rule. |
| `every` | `number` | interval | `5` | How many units between runs. |
| `unit` | `seconds \| minutes \| hours` | interval | `minutes` | The unit of `every`. |
| `time` | `string` (HH:MM) | daily, weekly, monthly | `09:00` | Time of day, 24-hour. `HH:MM:SS` also works. |
| `weekdays` | `mon … sun` (list) | weekly | `["mon"]` | The days to run on. |
| `dayOfMonth` | `number` (1–31) | monthly | `1` | The day to run on. **31 means the last day of every month.** 29 and 30 skip months without that day. |
| `cron` | `string` | cron | `0 9 * * 1-5` | Five fields (minute hour day month weekday), or six with seconds first. |
| `timezone` | `string` | No | server's | An IANA timezone such as `Europe/Paris` or `Africa/Casablanca`. Empty uses the server's timezone (usually UTC). |

### How intervals line up

An interval stays **on the clock** when the number divides the hour (or the day, for hours): every 15 minutes runs at :00, :15, :30, :45; every 6 hours at 00:00, 06:00, 12:00, 18:00. Any other number (every 7 minutes) runs at least that far apart, counted from the first run.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it has no data inputs.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted each time the schedule fires. |
| `error` | `Error` | Emitted if a tick fails. |

### Output Schema

```json
{
  "timestamp": "2026-11-10T08:30:00.012Z",
  "nextRun": "2026-11-11T08:30:00.000Z",
  "timezone": "Africa/Casablanca",
  "pattern": "0 30 9 * * *",
  "mode": "daily"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | `string` | When it fired (ISO 8601, UTC). |
| `nextRun` | `string \| null` | When it fires next. |
| `timezone` | `string` | The timezone the rule runs in. |
| `pattern` | `string` | The cron pattern the rule became (six fields, seconds first). |
| `mode` | `string` | The rule's mode. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### How it flows

1. **Schedule Trigger:** `daily` at `08:30`, timezone `Africa/Casablanca`.
2. **HTTP Request:** fetches yesterday's orders.
3. **Function:** builds the summary; **Email** sends it.

### Common Patterns

- **Poll and alert:** `every 5 minutes` → HTTP Request → If/Else → notify
- **Weekly cleanup:** `weekly`, `sun` at `03:00` → find old records → delete → Log
- **Business hours only:** `cron` `0 9-17 * * 1-5` → …

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### It runs an hour off
- **Cause:** No timezone set, so the server's (usually UTC) is used.
- **Solution:** Set `timezone` to an IANA name, e.g. `Europe/Paris`. Daylight saving is then handled for you.

#### The workflow will not start: `Unknown timezone`
- **Cause:** The timezone is not an IANA name (`CET` or `GMT+1` style names are not accepted).
- **Solution:** Use a name from the IANA list, like `Europe/Berlin`.

#### The workflow will not start: `Time "…" is not a time of day`
- **Solution:** Write the time as 24-hour `HH:MM`, e.g. `07:05` or `19:30`.

#### A monthly run skipped February
- **Cause:** Day 29 or 30 does not exist in every month.
- **Solution:** Use day `31` to mean "the last day of the month".

#### No catch-up after a pause
- **Cause:** Runs missed while the workflow was paused or stopped are not replayed.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Cron](/docs/cron) – A raw cron expression, in the server's timezone
- [Interval](/docs/interval) – Fires every N milliseconds from the start
- [Error Trigger](/docs/error-trigger) – Hear about it when a scheduled workflow fails

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-09 | Initial release |

<!-- /SECTION: changelog -->

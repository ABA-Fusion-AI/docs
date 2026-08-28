---
node_id: "throttle-time"
title: "Throttle Time"
description: "Emits a value from the source, then ignores subsequent source values for a specified duration."
category: "Orchestration"
subcategory: "Timing & Rate Control"
version: "1.0.0"
language: "en"
last_updated: "2026-08-28"
author: "Fusion Team"
tags:
  - throttle
  - throttle-time
  - rate-limiting
  - timing
  - flow-control
  - rxjs
  - stream
related_nodes:
  - debounce-time
  - delay
  - interval
  - function
  - log
---

<!-- SECTION: header -->
# Throttle Time

> **Category:** Orchestration | **Subcategory:** Timing & Rate Control | **Type:** Action Node

Control event flow rates by emitting the first received item immediately, then ignoring and dropping all subsequent incoming items for a specified time window.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Throttle Time** node acts as a rate-limiting filter on continuous observable data streams. When an item arrives from the upstream node, **Throttle Time** forwards it immediately to its `success` output and starts a silence timer of `duration` milliseconds.

Any additional items that arrive while this timer is active are discarded. Once the timer expires, the node is primed to emit the next incoming item and start a new silence window.

### How It Works

```text
Input Stream:   ──A───B───C───────D───E───F──────────>
                  │   x   x       │   x   x
                  │ (ignored)     │ (ignored)
Time Window:      ├───────────────┤ ├───────────────┤
                  └── duration ───┘ └── duration ───┘
                  
Output Stream:  ──A───────────────D──────────────────>
```

1. **Immediate Emission:** The first event (`A`) passes through with zero latency.
2. **Silence Window:** Events arriving during the active duration (`B`, `C`) are ignored.
3. **Reset:** Once the duration passes, the next arriving event (`D`) passes through immediately, triggering a new silence window.

### Key Features

- **Leading-Edge Rate Limiting:** Emits the first item without delay and suppresses bursts.
- **Microsecond/Millisecond Precision:** Configurable `duration` in milliseconds (`min: 1`).
- **Downstream Protection:** Protects expensive AI reasoning agents, third-party REST APIs, and database writes from being flooded.
- **Stream Transparency:** Passes error and completion signals to ensure proper workflow lifecycle management.

### Use Cases

- **Protecting AI Agent / LLM Endpoints:** Prevent costly repeated LLM invocations when users or webhooks trigger fast repeated queries.
- **High-Frequency Sensor / IoT Sampling:** Throttle continuous sensor streams or rapid intervals (e.g. from 100ms ticks down to 1 event every 2000ms).
- **External API Rate Limiting:** Smooth out webhook traffic bursts to stay within rate limits of third-party platforms (e.g., Slack, GitHub, Stripe).
- **UI Interaction & Click Debouncing:** Ignore repeated button presses or trigger events within a short timeframe.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `duration` | `number` | ✅ Yes | `2000` | The duration window in milliseconds (`ms`) to ignore subsequent values after an emission (`min: 1`). |

---

### Parameter Details

#### `duration`
The time window in milliseconds for which subsequent incoming values are ignored after an event is emitted.
- **Minimum Value:** `1` ms.
- **Example Values:**
  - `500`: Allows at most 1 event every 0.5 seconds.
  - `2000`: Allows at most 1 event every 2 seconds.
  - `60000`: Allows at most 1 event per minute.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The upstream observable event stream to throttle. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | Emits throttled events (the leading event of each time window). |
| `error` | `object` | Emitted when an error occurs in the upstream stream. Contains normalized workflow error details. |

---

### Throttle Time vs. Debounce Time vs. Delay

| Node | Behavior | Timing |
|------|----------|--------|
| **Throttle Time** | Emits the **first** event immediately, then ignores subsequent events for `duration` ms. | Immediate leading edge |
| **Debounce Time** | Waits until there is a **pause** of `duration` ms with no events, then emits the **latest** event. | Trailing edge after quiet period |
| **Delay** | Shifts **every** event forward in time by `duration` ms without dropping any items. | Fixed time delay |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Throttling a High-Frequency Interval

Throttle a fast 500ms `Interval` trigger down to 1 event every 2000ms.

**Node Parameters:**

```text
Duration: 2000
```

**Workflow Behavior:**
- `t = 0ms`: Interval emits tick #1 ➔ **Emitted immediately**
- `t = 500ms`: Interval emits tick #2 ➔ *Ignored (window active)*
- `t = 1000ms`: Interval emits tick #3 ➔ *Ignored (window active)*
- `t = 1500ms`: Interval emits tick #4 ➔ *Ignored (window active)*
- `t = 2000ms`: Window expires
- `t = 2000ms+`: Interval emits tick #5 ➔ **Emitted immediately**

---

### Example 2: Protecting AI Agent from Query Bursts

Place **Throttle Time** before an **Agent** or **AI Chat** node to safeguard against query spamming.

**Workflow Pattern:**

```text
Webhook / Chat Trigger
  → Throttle Time (duration: 3000)
  → AI Agent (processes request)
  → Log
```

---

### Example 3: External API Notification Rate Limiting

Ensure that system alerts sent via HTTP or SMS are not triggered more than once every 10 seconds during an outage event.

**Workflow Pattern:**

```text
System Error Stream / Webhook
  → Throttle Time (duration: 10000)
  → HTTP Request (Send Slack / SMS notification)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Throttle high-frequency stream events
```

### Common Patterns

- **Trigger Smoothing:** Fast Interval / IoT Sensor → Throttle Time → Transform Function → Database Storage.
- **LLM Rate-Limiter:** Webhook Ingress → Throttle Time → Prompt Builder → AI Chat.
- **Alert Deduplication:** Error Webhook → Throttle Time → Email Send / Notification.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `Validation error: duration must be greater than or equal to 1`

**Cause:** The `duration` parameter was configured with a negative number, `0`, or a non-numeric value.

**Solution:** Provide a positive integer representing milliseconds (e.g., `1000` for 1 second).

#### Issue: Expected items are missing downstream

**Cause:** `Throttle Time` intentionally discards any items that arrive while a time window is running.

**Solution:**
- If you need the *last* item after activity stops, use [Debounce Time](../debounce-time/en.md) instead.
- If you need *all* items without dropping any, use a queue or [Delay](../delay/en.md) node.
- If you need a shorter silence period, decrease the `duration` parameter.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `duration: Number must be greater than or equal to 1` | `duration < 1` or invalid number | Set a valid duration in milliseconds (>= 1) |
| Stream terminated | Upstream error occurred | Inspect upstream node logs and error output |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Debounce Time](../debounce-time/en.md) — Emit the latest value after a quiet period
- [Delay](../delay/en.md) — Delay all emitted items by a specified time
- [Interval](../interval/en.md) — Generate ticks at regular time intervals
- [Function](../function/en.md) — Execute custom JavaScript transformations
- [Log](../log/en.md) — Output results to the console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-28 | Initial release |

<!-- /SECTION: changelog -->

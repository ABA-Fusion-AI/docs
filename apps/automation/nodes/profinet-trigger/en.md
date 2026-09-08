---
node_id: "profinet-trigger"
title: "PROFINET Trigger"
description: "Monitors PROFINET IO data and triggers workflow on value changes. PROFINET is an industrial Ethernet standard for real-time automation."
category: "triggers-ingress"
subcategory: "iot-industrial-ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - profinet
  - trigger
  - industrial
  - iot
  - ethernet
related_nodes:
  - profinet-read
  - profinet-write
  - ethernet-ip-trigger
  - scada-read
---

<!-- SECTION: header -->
# PROFINET Trigger

> **Category:** Triggers & Ingress | **Type:** Trigger Node

Monitor simulated PROFINET IO data and trigger workflows when monitored values change.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **PROFINET Trigger** node monitors simulated PROFINET IO data at a configurable polling interval and emits workflow events when monitored values change.

The current implementation does not establish a real PROFINET connection. It generates simulated values locally for testing and workflow development.

### Key Features

- Monitors one or multiple simulated PROFINET offsets.
- Supports configurable slot, subslot, index, and length.
- Supports `input` and `output` I/O data types.
- Supports configurable polling intervals.
- Can trigger on the first read.
- Can filter changes using a numeric change threshold.
- Generates simulated values between `0` and `255`.
- Emits previous and current values with timestamp information.

### Processing Flow

```text
Start Trigger
  ↓
Start polling interval
  ↓
Generate simulated PROFINET values
  ↓
Compare current values with previous values
  ↓
Apply first-read / change-threshold rules
  ↓
Emit workflow event when conditions match
```

### Use Cases

- Testing PROFINET-triggered workflows without industrial hardware.
- Simulating industrial process value changes.
- Testing polling-based automation.
- Validating change detection logic.
- Prototyping industrial Ethernet event workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `deviceIp` | `string` | Yes | — | IP address associated with the monitored PROFINET device. |
| `slot` | `number` | No | `0` | Device slot number. Must be `0` or greater. |
| `subslot` | `number` | No | `0` | Device subslot number. Must be `0` or greater. |
| `index` | `number` | No | `0` | Starting data index. Must be `0` or greater. |
| `length` | `number` | No | `1` | Number of offsets to monitor. Must be at least `1`. |
| `ioDataType` | `string` | No | `input` | I/O data type. Supported values: `input` and `output`. |
| `port` | `number` | No | `34964` | PROFINET port value. Currently not used for real communication. |
| `pollInterval` | `number` | No | `1000` | Polling interval in milliseconds. Minimum value is `100`. |
| `changeThreshold` | `number` | No | — | Minimum absolute difference required to trigger after the first read. |
| `triggerOnFirstRead` | `boolean` | No | `true` | Emits an event when an offset is read for the first time. |

### Device IP

Example:

```text
192.168.10.1
```

The current implementation does not actually connect to this address.

### Index and Length

With:

```text
index: 10
length: 3
```

the node monitors:

```text
10
11
12
```

Each offset is tracked independently.

### Poll Interval

Example:

```text
pollInterval: 1000
```

This runs one simulated polling cycle every second.

The minimum allowed value is:

```text
100
```

### Change Threshold

When `changeThreshold` is not configured, the node emits an event whenever:

```text
currentValue != previousValue
```

When a threshold is configured:

```text
changeThreshold: 50
```

the node emits only when:

```text
abs(currentValue - previousValue) >= 50
```

### Trigger on First Read

When enabled:

```text
triggerOnFirstRead: true
```

the first value for each monitored offset generates an event with:

```text
previousValue: null
```

When disabled, the first value only initializes the internal state.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

As a Trigger Node, **PROFINET Trigger** starts the workflow and does not require an incoming workflow connection.

Its behavior is controlled by its configured parameters.

### Success Output

A triggered event has a structure similar to:

```json
{
  "deviceIp": "192.168.10.1",
  "slot": 0,
  "subslot": 0,
  "index": 0,
  "ioDataType": "input",
  "value": 140,
  "previousValue": 76,
  "timestamp": "2026-09-08T10:55:37.128Z"
}
```

For the first read, when `triggerOnFirstRead` is enabled:

```json
{
  "deviceIp": "192.168.10.1",
  "slot": 0,
  "subslot": 0,
  "index": 0,
  "ioDataType": "input",
  "value": 154,
  "previousValue": null,
  "timestamp": "2026-09-08T10:53:20.776Z"
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `deviceIp` | `string` | Configured device IP address. |
| `slot` | `number` | Configured slot. |
| `subslot` | `number` | Configured subslot. |
| `index` | `number` | Offset that generated the event. |
| `ioDataType` | `string` | Configured I/O data type. |
| `value` | `number` | Current simulated value. |
| `previousValue` | `number \| null` | Previously stored value, or `null` on the first triggered read. |
| `timestamp` | `string` | Event timestamp in ISO 8601 format. |

### Error Output

If an error occurs during polling, the node emits an error object through its error output:

```json
{
  "error": "Error message",
  "deviceIp": "192.168.10.1",
  "slot": 0,
  "subslot": 0
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Trigger on First Read

```text
deviceIp: 192.168.10.1
slot: 0
subslot: 0
index: 0
length: 1
ioDataType: input
port: 34964
pollInterval: 1000
changeThreshold: [empty]
triggerOnFirstRead: true
```

The first polling cycle generates an event with:

```text
previousValue: null
```

### Example 2: Monitor Multiple Offsets

```text
deviceIp: 192.168.10.1
slot: 1
subslot: 2
index: 10
length: 3
ioDataType: input
pollInterval: 1000
triggerOnFirstRead: true
```

The monitored offsets are:

```text
10
11
12
```

### Example 3: Change Threshold

```text
pollInterval: 500
changeThreshold: 200
triggerOnFirstRead: true
```

After the initial triggered read, subsequent events require:

```text
abs(currentValue - previousValue) >= 200
```

### Example 4: Skip First Read

```text
pollInterval: 500
changeThreshold: [empty]
triggerOnFirstRead: false
```

The first read initializes the stored value without emitting an event.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: PROFINET Trigger Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Device IP Address Is Required

**Cause:** The `deviceIp` parameter is missing or empty.

**Solution:** Provide a non-empty device IP address.

### Poll Interval Is Too Small

**Cause:** `pollInterval` is lower than `100`.

**Solution:** Use a polling interval of at least `100` milliseconds.

### No Event on First Read

**Cause:** `triggerOnFirstRead` is disabled.

**Solution:** Enable `triggerOnFirstRead` if the first monitored value should immediately trigger the workflow.

### Few Events Are Generated

**Cause:** A large `changeThreshold` can prevent most value changes from triggering.

**Solution:** Lower the threshold or leave it empty to trigger whenever the value changes.

### No Real PROFINET Communication Is Performed

The current implementation uses simulated random values and does not open a connection to a physical PROFINET device.

### Port Does Not Affect Polling

The `port` parameter defaults to `34964`, but it is not currently used to establish real network communication.

### Trigger State After Stop

When the node stops, its polling timer is cleared and previously stored values are removed.

A later fresh run begins with a new internal state.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **PROFINET Read** — Simulate PROFINET read operations.
- **PROFINET Write** — Simulate PROFINET write operations.
- **EtherNet/IP Trigger** — Trigger workflows from EtherNet/IP data.
- **SCADA Read** — Read simulated industrial SCADA data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation for the PROFINET Trigger node. |

<!-- /SECTION: changelog -->
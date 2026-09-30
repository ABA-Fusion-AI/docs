---
node_id: "modbus-trigger"
title: "Modbus Trigger"
description: "Monitor Modbus registers or coils over TCP or RTU and trigger workflows when values change."
category: "Triggers & Ingress"
subcategory: "IoT & Industrial Ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags: [modbus, trigger, industrial, iot, tcp, rtu, polling]
related_nodes: []
---

<!-- SECTION: overview -->
# Modbus Trigger

> **Category:** Triggers & Ingress / IoT & Industrial Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Monitor a range of Modbus coils, discrete inputs, holding registers, or input registers over TCP or RTU. The node polls the device and starts downstream workflow steps when an individual address meets the configured change condition.

Use it to record register changes, react to equipment state changes, or monitor numeric values with a minimum change threshold. A poll can emit multiple events: one for each qualifying address, rather than one event containing the entire range.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

### Connection

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `transport` | string enum | `"TCP"` | `"TCP"` for a network connection or `"RTU"` for a serial connection. |
| `host` | string | None | TCP device hostname or IP address. Shown for TCP. |
| `port` | number | `502` | TCP port. Shown for TCP. |
| `serialPort` | string | None | Serial device path accessible to the workflow runtime, such as `COM3` or `/dev/ttyUSB0`. Shown for RTU. |
| `baudRate` | number | `9600` | Serial baud rate. Shown for RTU. |
| `dataBits` | number | `8` | Serial data bits. Shown for RTU. |
| `stopBits` | number | `1` | Serial stop bits. Shown for RTU. |
| `parity` | string enum | `"none"` | `"none"`, `"even"`, or `"odd"`. Shown for RTU. |
| `unitId` | number | `1` | Unit identifier passed to the read helper. |

Connection fields are optional in the schema, but configure `host` for TCP or `serialPort` for RTU so the connection helper can reach the device. RTU settings must match the connected device. The node supplies a fixed connection timeout of `3000` milliseconds and passes a retry setting of `1` to the read helper; neither is exposed as a configurable parameter.

### Monitoring

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `address` | number | Yes | None | Starting address, at least `0`. Passed directly to the read helper without address conversion. |
| `quantity` | number | No | `1` | Number of consecutive values to read, at least `1`. |
| `functionCode` | string enum | No | `"3"` | Read operation from the table below. Use a string, not a number. |
| `pollInterval` | number | No | `1000` | Polling interval in milliseconds, at least `100`. |
| `changeThreshold` | number | No | Omitted | Emit when the absolute difference from the stored baseline is greater than or equal to this value. If omitted, emit on any unequal value. |
| `triggerOnFirstRead` | boolean | No | `true` | Emit an initial event for every address on its first successful read. |

The schema does not declare expression support for these fields. It enforces the listed minimum values but does not add integer or upper-bound checks for numeric fields. Use addresses, quantities, and connection settings supported by your device and the underlying Modbus helpers.

### Function codes

| Value | Operation |
|-------|-----------|
| `"1"` | Read Coils |
| `"2"` | Read Discrete Inputs |
| `"3"` | Read Holding Registers |
| `"4"` | Read Input Registers |

The node maps returned values to consecutive addresses starting at `address`. Boolean values are converted to numbers: `true` becomes `1`, and `false` becomes `0`. Numeric values pass through unchanged; the trigger performs no additional scaling or multi-register decoding.
<!-- /SECTION: configuration -->

<!-- SECTION: behavior -->
## Polling and Change Detection

The first poll is scheduled after `pollInterval`; starting the node does not perform an immediate read. On each poll, the node obtains a connection through the shared connection helper and reads the configured range.

Each address maintains its own baseline:

1. The first successful read stores the value. If `triggerOnFirstRead` is `true`, it also emits an event with `previousValue: null`.
2. Later reads compare the current value with the stored baseline.
3. Without `changeThreshold`, a different value triggers an event.
4. With `changeThreshold`, an event is triggered when `Math.abs(value - baseline) >= changeThreshold`.
5. The baseline updates when an event is triggered. Values below the threshold do not replace it.

For example, with a baseline of `100` and a threshold of `5`, readings of `102` and `104` emit nothing. A reading of `105` emits an event with `value: 105` and `previousValue: 100`, then becomes the new baseline. The same absolute-difference rule applies to decreases.

Use a positive threshold to suppress small changes. The schema permits zero and negative thresholds; either causes every subsequent successful comparison to trigger, even when the value is unchanged. The threshold also applies to coil and discrete-input values after conversion to `0` or `1`.

### Lifecycle

- **Pause:** Clears the polling timer and preserves stored values.
- **Resume:** Restarts polling if no timer exists. Existing baselines remain, so this does not repeat first-read events for known addresses.
- **Stop:** Clears the polling timer and all stored values. A later start establishes fresh baselines and applies `triggerOnFirstRead` again.

Polling uses an asynchronous interval without an in-flight guard. Reads can overlap if a poll takes longer than the configured interval. Clearing the timer does not cancel a read already in progress. The trigger's stop method does not explicitly close pooled connections.
<!-- /SECTION: behavior -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

This trigger starts from its polling schedule and does not require an upstream input.

### Success output

Each event describes one address:

| Field | Type | Description |
|-------|------|-------------|
| `transport` | string | Configured transport, `TCP` or `RTU`. |
| `host` | string, when configured | Configured TCP host. |
| `port` | number, when configured | Configured TCP port. |
| `serialPort` | string, when configured | Configured serial device path. |
| `unitId` | number | Configured unit identifier. |
| `address` | number | Address of the individual value that triggered this event. |
| `functionCode` | string | Configured read function code. |
| `value` | number | Current numeric value, with booleans normalized to `0` or `1`. |
| `previousValue` | number or null | Stored baseline before this event, or `null` on the first read. This may differ from the immediately preceding poll. |
| `timestamp` | string | ISO timestamp generated when the event is emitted. |

The payload includes connection fields from the configuration; unused fields can be undefined and disappear when serialized as JSON. It does not include the full polled range or `quantity`.

Example change event:

```json
{
  "transport": "TCP",
  "host": "192.0.2.10",
  "port": 502,
  "unitId": 1,
  "address": 0,
  "functionCode": "3",
  "value": 105,
  "previousValue": 100,
  "timestamp": "2026-09-30T10:00:00.000Z"
}
```

### Error output

A connection or read failure caught during polling is logged and emitted through the `error` output. The payload contains `error`, `transport`, `host`, `port`, and `serialPort`; unused connection fields may be undefined. It does not include an address, timestamp, or unit identifier.

```json
{
  "error": "Example connection or read failure",
  "transport": "TCP",
  "host": "192.0.2.10",
  "port": 502
}
```

Polling continues after a caught failure, and existing baselines remain unchanged. The exact error message comes from the underlying error. Failures while starting the timer are thrown with the prefix `Modbus trigger error:`.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Monitor Modbus changes and inspect polling errors
```

Connect the trigger's success output to a log to inspect value events, and its error output to a second log to inspect polling failures. The preview omits node parameters; configure the trigger using one of the examples below and replace the device address or serial path with your own.

### TCP: monitor holding registers

```json
{
  "transport": "TCP",
  "host": "192.0.2.10",
  "port": 502,
  "unitId": 1,
  "address": 0,
  "quantity": 4,
  "functionCode": "3",
  "pollInterval": 1000,
  "changeThreshold": 5,
  "triggerOnFirstRead": true
}
```

This configuration monitors addresses `0` through `3`. The first successful read emits four initial events if four values are returned. Subsequent events require an absolute difference of at least `5` from each address's baseline.

### RTU: monitor coil changes without initial events

```json
{
  "transport": "RTU",
  "serialPort": "COM3",
  "baudRate": 9600,
  "dataBits": 8,
  "stopBits": 1,
  "parity": "none",
  "unitId": 1,
  "address": 0,
  "quantity": 8,
  "functionCode": "1",
  "pollInterval": 1000,
  "triggerOnFirstRead": false
}
```

The first successful read establishes baselines silently. With `changeThreshold` omitted, later changes between `0` and `1` trigger individual events.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| Configuration fails validation | Supply `address >= 0`, `quantity >= 1`, and `pollInterval >= 100`. Use the documented enum values and types. |
| TCP polling fails | Verify the runtime can reach the configured host and port, then check the unit identifier, function code, and address range. |
| RTU polling fails | Verify the serial path exists on the workflow runtime, is accessible, and uses the device's baud rate, data bits, stop bits, and parity. |
| No event after starting | The first poll waits for the interval. If `triggerOnFirstRead` is false, the first read only establishes baselines. Inspect the error output for read failures. |
| Small changes produce no event | Check the threshold against the last stored baseline. Sub-threshold readings do not update it. |
| Unchanged values keep producing events | Remove `changeThreshold` to trigger only on unequal values, or set a positive threshold. Zero and negative thresholds allow unchanged values to trigger. |
| One poll produces several events | This is expected: each qualifying address emits separately. |
| Unexpected address or numeric interpretation | The node passes the starting address directly to the read helper and performs no numeric decoding beyond boolean conversion. Check the device register map and process values downstream as needed. |
| Polls overlap or events arrive after pausing | In-flight reads are not cancelled, and polling has no overlap guard. Allow enough time between polls for reads to finish. |

The connection and read helper implementations are not included in the supplied source. Their precise reconnection behavior, retry timing, and device-specific limits are not defined by this node.
<!-- /SECTION: troubleshooting -->

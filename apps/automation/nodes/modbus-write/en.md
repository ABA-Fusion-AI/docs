---
node_id: "modbus-write"
title: "Modbus Write"
description: "Write single or multiple coils and registers to Modbus devices over TCP or RTU."
category: "Databases & Memory"
subcategory: "Data Platforms"
version: "1.0.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags: [modbus, industrial, iot, tcp, rtu, registers, write]
related_nodes: [modbus-read, modbus-trigger]
---

<!-- SECTION: overview -->
# Modbus Write

> **Category:** Databases & Memory / Data Platforms&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Write coil states or register values to a Modbus device using TCP or RTU. Choose a single-value or multiple-value operation, configure the starting address, and supply an array of numbers.

The node performs a write when invoked and returns the submitted values after the client write completes. It does not read the values back or verify the resulting device state. Use a downstream Modbus Read action when the workflow requires readback.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

### Connection parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `transport` | string enum | `"TCP"` | `"TCP"` for a network connection or `"RTU"` for a serial connection. |
| `host` | string | None | TCP hostname or IP address. Shown for TCP. |
| `port` | number | `502` | TCP port. Shown for TCP. |
| `serialPort` | string | None | Serial device path accessible to the workflow runtime, such as `COM3` or `/dev/ttyUSB0`. Shown for RTU. |
| `baudRate` | number | `9600` | Serial baud rate. Shown for RTU. |
| `dataBits` | number | `8` | Serial data bits. Shown for RTU. |
| `stopBits` | number | `1` | Serial stop bits. Shown for RTU. |
| `parity` | string enum | `"none"` | `"none"`, `"even"`, or `"odd"`. Shown for RTU. |
| `unitId` | number | `1` | Unit identifier set on the client before each write attempt. |
| `timeout` | number | `5000` | Timeout setting passed to the connection helper. Units and enforcement are defined by that helper, which is not included in the supplied source. |

Connection fields are optional in the schema. Configure `host` for TCP or `serialPort` for RTU so the connection helper can reach the device. RTU settings must match the connected device.

### Write parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `address` | number | Yes | None | Starting address, at least `0`. Passed to the client without address conversion. |
| `values` | array of numbers | Yes | None | At least one numeric value. Single-value operations require exactly one item. |
| `functionCode` | string enum | No | `"6"` | `"5"`, `"6"`, `"15"`, or `"16"`, as described below. |
| `retries` | number | No | `3` | Additional write attempts after the first. Configuration accepts values from `0` through `5`. Use an integer. |

The schema does not declare expression support. It enforces the listed bounds and the non-empty numeric array, but does not declare integer checks, register-value bounds, maximum write lengths, or a minimum timeout. Supply values supported by the device and client library. Configure both `address` and `values` even when the input will override them, because they are required by the configuration schema.

### Function codes and value handling

| Function code | Operation | Value handling |
|---------------|-----------|----------------|
| `"5"` | Write Single Coil | Exactly one numeric value. Calls `writeCoil(address, values[0] !== 0)`. |
| `"6"` | Write Single Register | Exactly one numeric value. Calls `writeRegister(address, values[0])`. |
| `"15"` | Write Multiple Coils | Converts every value using `value !== 0`, then calls `writeCoils(address, coils)`. |
| `"16"` | Write Multiple Registers | Passes the numeric array unchanged to `writeRegisters(address, values)`. |

For coils, use `0` for off and `1` for on. The implementation actually treats **every nonzero number as on**, including negative numbers. Its single-coil error mentions `0 or 1`, but the code only checks the array length before converting the value.

Multiple-value operations begin at `address` and pass the array to the client. There is no separate `quantity` parameter. Register values are not scaled, packed, or converted by this node.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input overrides

An incoming object can override these settings for one invocation:

| Input field | Resolution order |
|-------------|------------------|
| `transport` | Truthy input value, configured value, then `"TCP"`. |
| `address` | Non-null input value, then configured value. |
| `values` | Truthy input value, then configured array. |
| `functionCode` | Truthy input value, configured value, then `"6"`. |
| `unitId` | Non-null input value, configured value, then `1`. |
| `retries` | Non-null input value, configured value, then `3`. |

For numeric fields, `0` is preserved; only `null` and `undefined` fall back. Empty strings for `transport` or `functionCode` fall back. An empty array is truthy, so input `values: []` overrides the configured values and then fails the non-empty check.

The handler uses TypeScript casts, not runtime conversion or schema validation of incoming overrides. Supply a numeric array for `values`, numbers for numeric fields, and strings for function codes. Do not assume the configuration schema's bounds are applied to payload overrides.

The complete configuration and payload are passed to `getOrCreateConnection`. Connection-field override behavior, including host, port, serial settings, and timeout, depends on that helper. During error cleanup, the node explicitly resolves `host` and `serialPort` from truthy input values and `port` from a non-null input value, falling back to configuration.

Example input that writes two registers without retrying:

```json
{
  "address": 10,
  "values": [100, 200],
  "functionCode": "16",
  "unitId": 1,
  "retries": 0
}
```

### Success output

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | `true` after the write helper completes successfully. |
| `transport` | string | Resolved transport. |
| `unitId` | number | Resolved unit identifier. |
| `address` | number | Resolved starting address. |
| `values` | array of numbers | Original submitted values, including unconverted numeric coil values. |
| `functionCode` | string | Resolved write function code. |
| `timestamp` | string | ISO timestamp generated after the write completes. |

The result does not include the client's raw response, per-address objects, connection details, or the original incoming payload. Returned values confirm what was submitted; they are not a readback.

Illustrative result:

```json
{
  "success": true,
  "transport": "TCP",
  "unitId": 1,
  "address": 10,
  "values": [100, 200],
  "functionCode": "16",
  "timestamp": "2026-09-30T10:00:00.000Z"
}
```

### Error output

The node throws errors instead of returning a `success: false` object. Connect the action's error output to downstream error handling.

If values are absent or empty, it throws `At least one value is required for write operation` before acquiring a connection. This error is not prefixed with `Modbus write error:` and does not run connection cleanup.

Errors caught during connection acquisition or writing trigger cleanup of the associated pooled connection. The node attempts to close it if open, ignores close errors, removes it from the pool, and throws `Modbus write error: <message>`.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Retries and Connection Lifecycle

For a non-negative integer `retries`, the write helper makes up to `retries + 1` attempts. The default of `3` allows four attempts. `retries: 0` makes one attempt.

Between failed attempts, it waits `100 * 2^attempt` milliseconds, with the first attempt numbered zero. Default retry delays are therefore `100`, `200`, and `400` milliseconds, in addition to time spent in the client calls. Each attempt sets the client unit identifier again and reuses the same client.

An `Error` whose message contains the exact, case-sensitive substring `Illegal` is thrown immediately without further retries. Other failures are retried, including single-value length checks and unsupported function-code errors that reach the helper. After exhaustion, the helper throws:

```text
Modbus write failed after <attempts> attempts: <last error>
```

The action wraps this with `Modbus write error:` after connection cleanup. Connection acquisition happens before the retry helper and is not retried by this loop; any connection-helper retry behavior is outside the supplied code.

Retries may repeat a write if the device applied an earlier attempt but its response was lost. Choose `retries` to suit the operation being performed.

Successful writes leave the pooled connection available. The node's `stop()` method returns without closing or clearing pooled connections. It also does not cancel in-progress writes or retry delays.
<!-- /SECTION: behavior -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Write Modbus values and inspect the result
```

The example connects a manual trigger to Modbus Write, sends successful results to a log, and sends errors to a separate log. The preview omits node parameters. Configure the action with the appropriate example below and replace the host or serial path with your device's connection details.

### TCP: write one register

```json
{
  "transport": "TCP",
  "host": "192.0.2.10",
  "port": 502,
  "unitId": 1,
  "address": 10,
  "values": [100],
  "functionCode": "6",
  "timeout": 5000,
  "retries": 3
}
```

To write multiple registers, select `"16"` and supply the full array, such as `[100, 200]`.

### RTU: write multiple coils

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
  "values": [1, 0, 1, 0],
  "functionCode": "15",
  "timeout": 5000,
  "retries": 1
}
```

This submits four coil states starting at address `0`. For a single coil, choose `"5"` and supply exactly one numeric value, such as `[1]`.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | What to check |
|------------------|---------------|
| Configuration fails validation | Supply `address >= 0`, a non-empty numeric `values` array, and `retries` between `0` and `5`. Function codes must be strings. |
| `At least one value is required for write operation` | Supply at least one value. Input `values: []` overrides any valid configured array. |
| `Write Single Coil requires exactly one value (0 or 1)` | For function `"5"`, supply an array with one number, preferably `[0]` or `[1]`. |
| `Write Single Register requires exactly one value` | Supply one value for function `"6"`, or select `"16"` for multiple registers. |
| `Unsupported function code: <code>` | Use `"5"`, `"6"`, `"15"`, or `"16"`, including in incoming overrides. |
| Error message contains `Illegal` | Check the function code, address, values, and device capabilities. Errors containing this exact substring skip retries. |
| TCP connection fails | Verify the runtime can reach the configured host and port. |
| RTU connection fails | Verify the serial path and access permissions, and match baud rate, data bits, stop bits, and parity to the device. |
| Coil result contains numbers instead of booleans | The success output preserves submitted numeric values even though the client receives booleans. |
| Write succeeds but the device state needs confirmation | Add a Modbus Read action to read back the relevant addresses. The write result does not verify device state. |
| Connection remains open after stopping | This node's stop method performs no connection cleanup. |

The connection helper and client internals are not supplied here. Their timeout semantics, protocol validation, and connection management are not fully specified by this node.
<!-- /SECTION: troubleshooting -->

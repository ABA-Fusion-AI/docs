---
node_id: "modbus-read"
title: "Modbus Read"
description: "Read coils, discrete inputs, holding registers, and input registers from Modbus devices over TCP or RTU."
category: "Databases & Memory"
subcategory: "Data Platforms"
version: "1.0.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags: [modbus, industrial, iot, tcp, rtu, registers, read]
related_nodes: [modbus-trigger]
---

<!-- SECTION: overview -->
# Modbus Read

> **Category:** Databases & Memory / Data Platforms&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read a consecutive range of Modbus values using TCP or RTU transport. The node supports coils, discrete inputs, holding registers, and input registers, and returns the values with their addresses in a single result.

Use this action to take a device snapshot when a workflow runs or to retrieve values before a downstream calculation or decision. Each invocation performs a read through the retry helper; the node does not schedule polling or detect changes automatically.
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
| `unitId` | number | `1` | Unit identifier passed to the read helper. |
| `timeout` | number | `5000` | Timeout setting passed to the connection helper. Its units and enforcement are defined by that helper, which is not included in the supplied source. |

Connection fields are optional in the schema. Configure `host` for TCP or `serialPort` for RTU so the connection helper can reach the device. RTU serial settings must match the connected device.

### Read parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `address` | number | Yes | None | Starting address, at least `0`. Passed directly to the read helper without address conversion. |
| `quantity` | number | No | `1` | Number of consecutive values requested, at least `1`. |
| `functionCode` | string enum | No | `"3"` | Read operation from the table below. Supply a string, not a number. |
| `retries` | number | No | `3` | Retry setting passed to the read helper. Configuration accepts values from `0` through `5`. |

The schema does not declare expression support for these parameters. It checks the listed bounds but does not declare integer checks, maximum addresses or quantities, or a minimum timeout. Use values supported by your device and the underlying helpers. Configure `address` even when an incoming payload will override it, because it is required by the configuration schema.

### Function codes

| Value | Operation |
|-------|-----------|
| `"1"` | Read Coils |
| `"2"` | Read Discrete Inputs |
| `"3"` | Read Holding Registers |
| `"4"` | Read Input Registers |

The retry helper implementation is not included in the supplied source. The exact number of attempts, retry delays, supported device limits, and connection reuse behavior cannot be determined from this node alone.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input overrides

An incoming object can override the following configured values for one invocation:

| Input field | Resolution order |
|-------------|------------------|
| `transport` | Truthy input value, configured value, then `"TCP"`. |
| `address` | Non-null input value, then configured value. |
| `quantity` | Non-null input value, configured value, then `1`. |
| `functionCode` | Truthy input value, configured value, then `"3"`. |
| `unitId` | Non-null input value, configured value, then `1`. |
| `retries` | Non-null input value, configured value, then `3`. |

For numeric fields, `0` is preserved by the override logic; only `null` and `undefined` fall back. Empty strings for `transport` and `functionCode` fall back to configuration. Input values are type-cast in the source but are not validated or converted by the handler, so supply correctly typed, valid values. Configuration validation does not imply that incoming overrides receive the same checks.

The complete configuration and payload are passed to `getOrCreateConnection`. Connection-field overrides, including `host`, `port`, `serialPort`, serial settings, and `timeout`, depend on that helper's implementation. The error-cleanup path explicitly resolves `host` and `serialPort` using truthy input values, and `port` using a non-null input value, before falling back to configuration.

Example input that reads two input registers with retries disabled:

```json
{
  "address": 10,
  "quantity": 2,
  "functionCode": "4",
  "unitId": 1,
  "retries": 0
}
```

### Success output

The node returns one object containing all returned values. It does not merge the incoming payload into the result.

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Always `true` for a successful read. |
| `transport` | string | Resolved transport. |
| `unitId` | number | Resolved unit identifier. |
| `address` | number | Resolved starting address. |
| `quantity` | number | Requested quantity, not a separately calculated returned-value count. |
| `functionCode` | string | Resolved read function code. |
| `values` | array of objects | Returned values, each with an `address` and `value`. |
| `values[].address` | number | Starting address plus the item's zero-based index. |
| `values[].value` | number or boolean | Value returned by the read helper, preserved unchanged. |
| `timestamp` | string | ISO timestamp generated after the read. |

Boolean values remain booleans; this action does not convert them to `0` or `1`. It performs no additional scaling, signed conversion, or multi-register decoding. Connection details such as host and serial port are not included in the success result.

Illustrative holding-register result:

```json
{
  "success": true,
  "transport": "TCP",
  "unitId": 1,
  "address": 0,
  "quantity": 2,
  "functionCode": "3",
  "values": [
    {"address": 0, "value": 100},
    {"address": 1, "value": 105}
  ],
  "timestamp": "2026-09-30T10:00:00.000Z"
}
```

### Errors and connection cleanup

When connection acquisition, reading, or result processing fails inside the handler's `try` block, the node looks up the associated pooled connection, attempts to close it if open, and removes it from the pool. Errors from closing the client are ignored. It then throws an error with the prefix `Modbus read error:` followed by the original error message.

The node does not return a `{ "success": false }` object. Connect the action's error output to downstream error handling.

Calling `stop()` attempts to close and remove **every connection in the shared pool**, not only the connection used by the most recent read. Close errors are ignored during this cleanup.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: read-example.workflow.json
title: Read Modbus values and inspect the result
```

The example connects a manual trigger to Modbus Read and sends its success output to a log. A second log receives errors. The preview omits node parameters; configure the read action using an example below and replace the host or serial path with your device's connection details.

### TCP: read two holding registers

```json
{
  "transport": "TCP",
  "host": "192.0.2.10",
  "port": 502,
  "unitId": 1,
  "address": 0,
  "quantity": 2,
  "functionCode": "3",
  "timeout": 5000,
  "retries": 3
}
```

This requests holding registers at addresses `0` and `1`. Downstream steps can inspect `values[0].value` and `values[1].value` when both values are returned.

### RTU: read eight coils

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
  "timeout": 5000,
  "retries": 1
}
```

This requests coils at addresses `0` through `7`. If the helper returns booleans, those booleans appear unchanged in `values`.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| Configuration fails validation | Supply `address >= 0`, `quantity >= 1`, and `retries` between `0` and `5`. Use the documented enum values and types. |
| TCP connection fails | Verify that the workflow runtime can reach the configured host and port. Check the error message from the underlying helper. |
| RTU connection fails | Verify the serial path exists on the runtime, is accessible, and uses settings matching the device. |
| Read fails after connecting | Check the unit identifier, function code, starting address, and quantity against the device's register map. |
| Input override behaves unexpectedly | Use an object with correctly typed fields. Numeric zero is preserved; empty transport or function-code strings fall back to configuration. The handler does not validate payload overrides. |
| Returned numbers need interpretation | Apply device-specific scaling or decoding downstream. This node maps returned values to addresses without further conversion. |
| Values arrive as booleans | Boolean values from the read helper are preserved. Convert them downstream if numeric flags are needed. |
| Repeated reads do not happen automatically | This is an action node. Invoke it from a scheduled workflow or use Modbus Trigger for change monitoring. |
| Other pooled connections are closed when stopping | The stop method closes and removes all connections in the shared pool. |

Connection setup, timeout behavior, and retry behavior are implemented in external helpers and are not fully specified by the supplied node code.
<!-- /SECTION: troubleshooting -->

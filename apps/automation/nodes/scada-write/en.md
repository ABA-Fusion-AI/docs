---
node_id: "scada-write"
title: "SCADA Write"
description: "Generic SCADA node for writing data points to various industrial protocols. Supports Modbus, DNP3, BACnet, OPC, IEC61850, and IEC104."
category: "security-networking"
subcategory: "industrial-iot-protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - scada
  - industrial
  - iot
  - modbus
  - dnp3
  - bacnet
  - opc
related_nodes:
  - scada-read
  - dnp3-master
  - opc
  - bacnet-write
  - ethernet-ip-write
---

<!-- SECTION: header -->
# SCADA Write

> **Category:** Security & Networking | **Type:** Action Node

Write simulated SCADA data points using configurable industrial protocol, connection, point address, value, and data type settings.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **SCADA Write** node provides a generic interface for SCADA-style data write operations.

The current implementation simulates a successful write locally and does not establish a real network connection or write data to an industrial device.

It supports the following protocol values:

- `Modbus`
- `DNP3`
- `BACnet`
- `OPC`
- `IEC61850`
- `IEC104`

### Key Features

- Supports multiple industrial protocol identifiers.
- Accepts configurable connection strings.
- Accepts configurable SCADA point addresses.
- Accepts values of any type.
- Supports optional boolean, integer, float, and string data type identifiers.
- Uses default values for protocol and timeout.
- Can resolve parameters from incoming workflow data.
- Returns a structured SCADA-style write result.
- Generates a timestamp for every execution.

### Processing Flow

```text
Input
  ↓
Resolve payload or configured parameters
  ↓
Resolve protocol, value, and data type
  ↓
Validate that a value is available
  ↓
Simulate SCADA write
  ↓
Return result
```

### Use Cases

- Testing SCADA-oriented write workflows.
- Simulating industrial control commands.
- Building Modbus-style workflow prototypes.
- Building DNP3-style workflow prototypes.
- Testing downstream workflow behavior without a live industrial device.
- Validating workflows that expect a structured SCADA write response.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `protocol` | `string` | No | `Modbus` | Industrial protocol identifier. Supported values: `Modbus`, `DNP3`, `BACnet`, `OPC`, `IEC61850`, and `IEC104`. |
| `connectionString` | `string` | Yes | — | Connection string associated with the target industrial endpoint. |
| `pointAddress` | `string` | Yes | — | Address or identifier of the data point to write. |
| `value` | any | Yes | — | Value to associate with the simulated write operation. |
| `dataType` | `string` | No | — | Optional value type identifier: `boolean`, `integer`, `float`, or `string`. |
| `timeout` | `number` | No | `5000` | Timeout value in milliseconds. It is currently resolved but not used by the simulated implementation. |

### Protocol

Select the industrial protocol identifier associated with the write.

Example:

```text
Modbus
```

### Connection String

Provide the connection string associated with the target endpoint.

Example:

```text
tcp://127.0.0.1:502
```

### Point Address

Provide the target point address.

Example:

```text
holding-register:40001
```

### Value

Provide the value to associate with the write operation.

Examples:

```text
true
```

```text
528
```

```text
42.75
```

```text
SCADA Test
```

If no value is available from either the incoming payload or node configuration, the node throws:

```text
Value is required for write operation
```

### Data Type

Supported configured values:

- `boolean`
- `integer`
- `float`
- `string`

If `dataType` is not provided, the implementation returns JavaScript `typeof value` as the output data type.

For example, a numeric value without an explicit `dataType` would return:

```text
number
```

### Timeout

The default timeout is:

```text
5000
```

The current implementation resolves this parameter but does not use it to perform network communication.

### Input Priority

Parameters are resolved from incoming workflow data before node configuration.

For example, if incoming workflow data contains:

```json
{
  "protocol": "DNP3",
  "value": 75.5
}
```

those values override the configured `protocol` and `value`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts incoming workflow data.

Supported incoming fields include:

```text
protocol
connectionString
pointAddress
value
dataType
timeout
```

Incoming values can override configured node parameters.

### Output

A successful execution returns an object similar to:

```json
{
  "success": true,
  "protocol": "Modbus",
  "connectionString": "tcp://127.0.0.1:502",
  "pointAddress": "holding-register:40001",
  "value": 528,
  "dataType": "integer",
  "timestamp": "2026-09-04T14:48:30.518Z"
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful execution. |
| `protocol` | `string` | Resolved protocol value. |
| `connectionString` | `string` | Resolved connection string. |
| `pointAddress` | `string` | Resolved target point address. |
| `value` | any | Value associated with the simulated write operation. |
| `dataType` | `string` | Explicit configured data type or JavaScript `typeof value` when omitted. |
| `timestamp` | `string` | Execution timestamp in ISO 8601 format. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Boolean Modbus Write

**Configuration**

```text
protocol: Modbus
connectionString: tcp://127.0.0.1:502
pointAddress: coil:00001
value: true
dataType: boolean
timeout: 5000
```

**Output**

```json
{
  "success": true,
  "protocol": "Modbus",
  "connectionString": "tcp://127.0.0.1:502",
  "pointAddress": "coil:00001",
  "value": true,
  "dataType": "boolean",
  "timestamp": "2026-09-04T14:47:45.060Z"
}
```

### Example 2: Integer Modbus Write

**Configuration**

```text
protocol: Modbus
connectionString: tcp://127.0.0.1:502
pointAddress: holding-register:40001
value: 528
dataType: integer
timeout: 5000
```

### Example 3: DNP3 Float Write

**Configuration**

```text
protocol: DNP3
connectionString: tcp://127.0.0.1:20000
pointAddress: analog-output:1
value: 75.5
dataType: float
timeout: 5000
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: SCADA Write Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Connection string is required

**Cause:** The `connectionString` parameter is missing or empty.

**Solution:** Provide a non-empty connection string.

Example:

```text
tcp://127.0.0.1:502
```

### Point address is required

**Cause:** The `pointAddress` parameter is missing or empty.

**Solution:** Provide a valid point address.

Example:

```text
holding-register:40001
```

### Value is required for write operation

**Cause:** No value is available from either the incoming workflow data or node configuration.

**Solution:** Provide a value in the `value` parameter.

### No Real SCADA Write Is Performed

The current implementation simulates a successful write.

The following values are resolved and returned:

```text
protocol
connectionString
pointAddress
value
dataType
```

but no actual industrial protocol handler is called.

### Timeout Does Not Affect Execution

The `timeout` parameter is resolved but not currently used by the simulated write implementation.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **SCADA Read** — Read SCADA-style data points.
- **DNP3 Master** — Work with DNP3 industrial communication.
- **OPC** — Work with OPC-oriented industrial workflows.
- **BACnet Write** — Write BACnet-oriented data points.
- **EtherNet/IP Write** — Write industrial data using EtherNet/IP.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation for the SCADA Write node. |

<!-- /SECTION: changelog -->
---
node_id: "scada-read"
title: "SCADA Read"
description: "Generic SCADA node for reading data points from various industrial protocols. Supports Modbus, DNP3, BACnet, OPC, IEC61850, and IEC104."
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
  - scada-write
  - dnp3-master
  - opc
  - bacnet-read
  - ethernet-ip-read
---

<!-- SECTION: header -->
# SCADA Read

> **Category:** Security & Networking | **Type:** Action Node

Read simulated SCADA data points using configurable industrial protocol, connection, point address, and data type settings.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **SCADA Read** node provides a generic interface for SCADA-style data reads.

The current implementation simulates values locally and does not establish a real network connection to an industrial device.

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
- Supports boolean, integer, float, string, and timestamp values.
- Uses default values for protocol, data type, and timeout.
- Can resolve parameters from incoming workflow data.
- Returns structured SCADA-style output.
- Returns a quality value of `GOOD`.
- Generates a timestamp for every execution.

### Processing Flow

```text
Input
  ↓
Resolve payload or configured parameters
  ↓
Resolve protocol and data type
  ↓
Generate simulated value
  ↓
Build SCADA result
  ↓
Return output
```

### Use Cases

- Testing SCADA-oriented workflows.
- Simulating industrial telemetry.
- Building Modbus-style workflow prototypes.
- Building DNP3-style workflow prototypes.
- Testing downstream processing without a live industrial device.
- Validating workflows that expect SCADA-style structured output.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `protocol` | `string` | No | `Modbus` | Industrial protocol identifier. Supported values: `Modbus`, `DNP3`, `BACnet`, `OPC`, `IEC61850`, `IEC104`. |
| `connectionString` | `string` | Yes | — | Connection string associated with the target industrial endpoint. |
| `pointAddress` | `string` | Yes | — | Address or identifier of the data point to read. |
| `dataType` | `string` | No | `float` | Output value type: `boolean`, `integer`, `float`, `string`, or `timestamp`. |
| `timeout` | `number` | No | `5000` | Timeout value in milliseconds. The current simulated implementation resolves this value but does not use it for network communication. |

### Protocol

Select the industrial protocol identifier associated with the read.

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

Provide the point address to associate with the read.

Example:

```text
holding-register:40001
```

### Data Type

The node supports five data types.

#### boolean

Returns a random boolean value:

```text
true
```

or:

```text
false
```

#### integer

Returns a random integer between `0` and `999`.

#### float

Returns a random floating-point value between `0` and `100`.

#### string

Returns:

```text
Sample SCADA Data
```

#### timestamp

Returns the current timestamp in ISO 8601 format.

### Timeout

The default timeout is:

```text
5000
```

The value is currently resolved by the node but is not used by the simulated implementation.

### Input Priority

Parameters are resolved from incoming workflow data before configured node values.

For example, if incoming data contains:

```json
{
  "protocol": "DNP3"
}
```

that value overrides the configured `protocol`.

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
dataType
timeout
```

Incoming values can override configured node parameters.

### Output

The node returns an object containing the simulated SCADA result.

Example:

```json
{
  "success": true,
  "protocol": "Modbus",
  "connectionString": "tcp://127.0.0.1:502",
  "pointAddress": "holding-register:40001",
  "dataType": "float",
  "value": 78.29866109610084,
  "quality": "GOOD",
  "timestamp": "2026-09-04T14:31:45.003Z"
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful execution. |
| `protocol` | `string` | Resolved protocol value. |
| `connectionString` | `string` | Resolved connection string. |
| `pointAddress` | `string` | Resolved point address. |
| `dataType` | `string` | Selected output data type. |
| `value` | mixed | Simulated value generated according to `dataType`. |
| `quality` | `string` | SCADA-style quality indicator. Currently `GOOD`. |
| `timestamp` | `string` | Execution timestamp in ISO 8601 format. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Simulated Modbus Float Read

**Configuration**

```text
protocol: Modbus
connectionString: tcp://127.0.0.1:502
pointAddress: holding-register:40001
dataType: float
timeout: 5000
```

**Possible Output**

```json
{
  "success": true,
  "protocol": "Modbus",
  "connectionString": "tcp://127.0.0.1:502",
  "pointAddress": "holding-register:40001",
  "dataType": "float",
  "value": 78.29866109610084,
  "quality": "GOOD",
  "timestamp": "2026-09-04T14:31:45.003Z"
}
```

### Example 2: Simulated Boolean Read

**Configuration**

```text
protocol: Modbus
connectionString: tcp://127.0.0.1:502
pointAddress: holding-register:40001
dataType: boolean
timeout: 5000
```

**Possible Value**

```text
false
```

### Example 3: Simulated DNP3 Read

**Configuration**

```text
protocol: DNP3
connectionString: tcp://127.0.0.1:20000
pointAddress: analog-input:1
dataType: float
timeout: 5000
```

The node returns the DNP3 configuration together with a simulated floating-point value.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: SCADA Read Example
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

**Solution:** Provide a point address.

Example:

```text
holding-register:40001
```

### Values Change Between Runs

This is expected behavior for `boolean`, `integer`, and `float` data types.

The current implementation uses random values for these types.

### No Real SCADA Device Is Queried

The current implementation simulates the SCADA value locally.

The following values are resolved and returned:

```text
protocol
connectionString
pointAddress
dataType
```

but they are not currently used to establish a real industrial protocol connection.

### Timeout Does Not Affect the Result

The `timeout` value is accepted and resolved, but it is not currently used during the simulated read.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **SCADA Write** — Write SCADA-style data points.
- **DNP3 Master** — Work with DNP3 industrial communication.
- **OPC** — Work with OPC-oriented industrial workflows.
- **BACnet Read** — Read BACnet-oriented data points.
- **EtherNet/IP Read** — Read industrial data using EtherNet/IP.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation for the SCADA Read node. |

<!-- /SECTION: changelog -->
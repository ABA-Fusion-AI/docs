---
node_id: "profinet-read"
title: "PROFINET Read"
description: "Reads data from PROFINET devices. PROFINET is an industrial Ethernet standard for real-time automation."
category: "security-networking"
subcategory: "industrial-iot-protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - profinet
  - industrial
  - iot
  - automation
  - ethernet
related_nodes:
  - profinet-write
  - profinet-trigger
  - scada-read
  - ethernet-ip-read
---

<!-- SECTION: header -->
# PROFINET Read

> **Category:** Security & Networking | **Type:** Action Node

Read simulated PROFINET data using configurable device, slot, subslot, index, length, and I/O data type settings.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **PROFINET Read** node provides a generic interface for PROFINET-style read operations.

The current implementation simulates PROFINET reads locally and does not establish a real network connection with an industrial device.

### Key Features

- Supports configurable device IP addresses.
- Supports slot and subslot selection.
- Supports configurable starting index.
- Supports reading multiple values with the `length` parameter.
- Supports `input` and `output` I/O data types.
- Uses the standard PROFINET port `34964` by default.
- Generates simulated byte values between `0` and `255`.
- Returns a structured result with timestamp information.

### Processing Flow

```text
Input
  ↓
Resolve payload or configured parameters
  ↓
Resolve device, addressing, length, and I/O type
  ↓
Generate simulated PROFINET values
  ↓
Return structured result
```

### Use Cases

- Testing PROFINET-oriented workflows.
- Simulating industrial automation data reads.
- Testing input and output process data flows.
- Building industrial Ethernet workflow prototypes.
- Validating downstream workflow behavior without physical PROFINET hardware.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `deviceIp` | `string` | Yes | — | IP address of the target PROFINET device. |
| `slot` | `number` | No | `0` | Device slot number. |
| `subslot` | `number` | No | `0` | Device subslot number. |
| `index` | `number` | No | `0` | Starting data index. |
| `length` | `number` | No | `1` | Number of values to generate. Must be at least `1`. |
| `ioDataType` | `string` | No | `input` | I/O data type. Supported values: `input` and `output`. |
| `port` | `number` | No | `34964` | PROFINET port value. Currently resolved but not used for real communication. |

### Device IP

Provide the device IP address.

Example:

```text
192.168.10.1
```

The current implementation does not actually connect to this IP address.

### Slot and Subslot

Example:

```text
slot: 1
subslot: 2
```

### Index

Defines the starting offset for generated values.

Example:

```text
index: 10
```

With `length: 4`, the generated offsets are:

```text
10
11
12
13
```

### Length

Defines how many simulated values are returned.

Example:

```text
length: 4
```

### I/O Data Type

Supported values:

- `input`
- `output`

Default:

```text
input
```

### Port

Default:

```text
34964
```

The port is currently resolved by the node but is not used to establish a real PROFINET connection.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts incoming workflow data.

Supported incoming fields include:

```text
deviceIp
slot
subslot
index
length
ioDataType
port
```

Incoming values override configured node parameters when provided.

### Output

A successful execution returns an object similar to:

```json
{
  "success": true,
  "deviceIp": "192.168.10.1",
  "slot": 0,
  "subslot": 0,
  "index": 0,
  "length": 1,
  "ioDataType": "input",
  "data": [
    {
      "offset": 0,
      "value": 128
    }
  ],
  "timestamp": "2026-09-08T10:33:43.049Z"
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful execution. |
| `deviceIp` | `string` | Resolved device IP address. |
| `slot` | `number` | Resolved slot. |
| `subslot` | `number` | Resolved subslot. |
| `index` | `number` | Starting data index. |
| `length` | `number` | Number of generated values. |
| `ioDataType` | `string` | Resolved I/O data type. |
| `data` | `array` | Simulated values and their offsets. |
| `timestamp` | `string` | Execution timestamp in ISO 8601 format. |

Each generated `value` is an integer between `0` and `255`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic Input Read

**Configuration**

```text
deviceIp: 192.168.10.1
slot: 0
subslot: 0
index: 0
length: 1
ioDataType: input
port: 34964
```

### Example 2: Multiple Input Values

**Configuration**

```text
deviceIp: 192.168.10.1
slot: 1
subslot: 2
index: 10
length: 4
ioDataType: input
port: 34964
```

The returned offsets are:

```text
10
11
12
13
```

### Example 3: Output Data

**Configuration**

```text
deviceIp: 192.168.10.1
slot: 0
subslot: 0
index: 5
length: 2
ioDataType: output
port: 34964
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: PROFINET Read Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Device IP address is required

**Cause:** The `deviceIp` parameter is missing or empty.

**Solution:** Provide a non-empty device IP address.

Example:

```text
192.168.10.1
```

### Length must be at least 1

**Cause:** `length` is lower than the minimum allowed value.

**Solution:** Use a value of `1` or greater.

### No Real PROFINET Communication Is Performed

The current implementation simulates a PROFINET read.

The node does not open a real PROFINET connection and does not communicate with physical industrial hardware.

### Values Change Between Executions

The returned values are generated using a random value between `0` and `255`.

Different executions can therefore return different values even with identical configuration.

### Port Does Not Affect Execution

The `port` parameter defaults to `34964`, but it is not currently used for network communication.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **PROFINET Write** — Simulate PROFINET write operations.
- **PROFINET Trigger** — Trigger workflows from simulated PROFINET data.
- **SCADA Read** — Read simulated SCADA-style data points.
- **EtherNet/IP Read** — Read industrial data using EtherNet/IP.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation for the PROFINET Read node. |

<!-- /SECTION: changelog -->
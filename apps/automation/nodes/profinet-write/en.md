---
node_id: "profinet-write"
title: "PROFINET Write"
description: "Writes data to PROFINET devices. PROFINET is an industrial Ethernet standard for real-time automation."
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
  - profinet-read
  - profinet-trigger
  - scada-write
  - ethernet-ip-write
---

<!-- SECTION: header -->
# PROFINET Write

> **Category:** Security & Networking | **Type:** Action Node

Write simulated data to PROFINET devices using configurable device, slot, subslot, index, and data settings.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **PROFINET Write** node provides a generic interface for PROFINET-style write operations.

The current implementation simulates PROFINET writes locally and does not establish a real network connection with an industrial device.

### Key Features

- Supports configurable device IP addresses.
- Supports slot and subslot selection.
- Supports configurable starting index.
- Supports writing one or multiple numeric values.
- Uses the standard PROFINET port `34964` by default.
- Returns the written data in a structured result.
- Provides timestamp information for each simulated write.

### Processing Flow

```text
Input
  ↓
Resolve payload or configured parameters
  ↓
Validate that write data is available
  ↓
Resolve device and addressing parameters
  ↓
Simulate PROFINET write
  ↓
Return structured result
```

### Use Cases

- Testing PROFINET-oriented workflows.
- Simulating industrial automation write operations.
- Testing process data writes without physical hardware.
- Building industrial Ethernet workflow prototypes.
- Validating downstream workflow behavior.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `deviceIp` | `string` | Yes | — | IP address of the target PROFINET device. |
| `slot` | `number` | No | `0` | Device slot number. Must be `0` or greater. |
| `subslot` | `number` | No | `0` | Device subslot number. Must be `0` or greater. |
| `index` | `number` | No | `0` | Starting data index. Must be `0` or greater. |
| `data` | `number[]` | Yes | — | Array containing at least one numeric value to write. |
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

Defines the starting index for the simulated write.

Example:

```text
index: 10
```

### Data

At least one numeric value is required.

Example:

```text
10
20
30
255
```

The current schema validates the values as numbers but does not restrict individual values to the byte range `0–255`.

For example, a value such as `300` is currently accepted.

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
data
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
  "data": [
    128
  ],
  "timestamp": "2026-09-08T10:41:57.386Z"
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful execution. |
| `deviceIp` | `string` | Resolved device IP address. |
| `slot` | `number` | Resolved slot. |
| `subslot` | `number` | Resolved subslot. |
| `index` | `number` | Resolved starting index. |
| `data` | `number[]` | Numeric values used for the simulated write. |
| `timestamp` | `string` | Execution timestamp in ISO 8601 format. |

The configured `port` is not included in the output.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic Write

**Configuration**

```text
deviceIp: 192.168.10.1
slot: 0
subslot: 0
index: 0
data:
  128
port: 34964
```

### Example 2: Multiple Values

**Configuration**

```text
deviceIp: 192.168.10.1
slot: 1
subslot: 2
index: 10
data:
  10
  20
  30
  255
port: 34964
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: PROFINET Write Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Device IP Address Is Required

**Cause:** The `deviceIp` parameter is missing or empty.

**Solution:** Provide a non-empty device IP address.

Example:

```text
192.168.10.1
```

### At Least One Data Value Is Required

**Cause:** The `data` array is missing or empty.

**Solution:** Add at least one numeric value to the `data` parameter.

### Values Outside the Byte Range Are Accepted

The implementation refers to the values as data bytes, but the current schema only validates that each item is a number.

Values outside `0–255`, such as `300`, can therefore currently be accepted.

### No Real PROFINET Communication Is Performed

The current implementation simulates the write operation.

The node does not open a real PROFINET connection and does not send data to physical industrial hardware.

### Port Does Not Affect Execution

The `port` parameter defaults to `34964`, but it is not currently used for network communication.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **PROFINET Read** — Simulate PROFINET read operations.
- **PROFINET Trigger** — Trigger workflows from simulated PROFINET data.
- **SCADA Write** — Write simulated SCADA-style data points.
- **EtherNet/IP Write** — Write industrial data using EtherNet/IP.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation for the PROFINET Write node. |

<!-- /SECTION: changelog -->
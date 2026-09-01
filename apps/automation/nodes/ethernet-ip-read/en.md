---
node_id: "ethernet-ip-read"
title: "EtherNet/IP Read"
description: "Read tag values from EtherNet/IP devices."
category: "industrial-iot"
subcategory: "industrial-iot-protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - ethernet-ip
  - industrial
  - iot
  - cip
  - plc
  - read
  - automation
related_nodes:
  - ethernet-ip-trigger
  - ethernet-ip-write
  - bacnet-read
  - dnp3-master
---

<!-- SECTION: header -->
# EtherNet/IP Read

> **Category:** Industrial & IoT Protocols | **Type:** Action Node

Read tag values from EtherNet/IP devices using configurable device, path, tag, and port parameters.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EtherNet/IP Read** node reads a configured tag value associated with an EtherNet/IP device.

EtherNet/IP is an industrial communication protocol that adapts the Common Industrial Protocol (CIP) to standard Ethernet networks.

The node accepts a device IP address, CIP path, tag name, and port. These values can be configured directly on the node or, when provided, overridden by incoming workflow data.

The current implementation simulates the EtherNet/IP read operation by generating a numeric value. It does not currently establish a connection to a physical EtherNet/IP controller.

### Key Features

- **Tag Reading:** Reads a configured EtherNet/IP tag value.
- **Device Configuration:** Supports configurable device IP addresses.
- **CIP Path Support:** Supports a configurable CIP routing path.
- **Custom Tag:** Reads the configured tag name.
- **Custom Port:** Supports a configurable EtherNet/IP port.
- **Input Overrides:** Allows incoming workflow data to override configured values.
- **Structured Output:** Returns device information, tag information, value, data type, and timestamp.
- **Validation:** Requires a device IP address and tag name.
- **Simulated Reads:** Generates numeric values for workflow testing without requiring a physical controller.

### Use Cases

- Read industrial equipment values
- Retrieve PLC tag values
- Build industrial monitoring workflows
- Process EtherNet/IP telemetry
- Test industrial workflow integrations
- Simulate PLC data acquisition
- Feed industrial values into downstream processing
- Connect industrial data to logging or storage workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `deviceIp` | `string` | ✅ Yes | — | IP address of the EtherNet/IP device. |
| `path` | `string` | ❌ No | `1,0` | CIP routing path used for the device. |
| `tag` | `string` | ✅ Yes | — | Name of the tag to read. |
| `port` | `number` | ❌ No | `44818` | EtherNet/IP communication port. |

### Default Values

| Parameter | Default |
|-----------|---------|
| `path` | `1,0` |
| `port` | `44818` |

### Device IP

`deviceIp` identifies the EtherNet/IP device associated with the tag.

Example:

```text
192.168.1.10
```

The configured value is required and cannot be empty.

### CIP Path

`path` specifies the CIP routing path.

Default:

```text
1,0
```

### Tag

`tag` specifies the tag to read.

Example:

```text
TestTag
```

The configured value is required and cannot be empty.

### Port

`port` specifies the EtherNet/IP communication port.

Default:

```text
44818
```

### Input Override Behavior

The node can use values received from the incoming workflow data instead of the configured values.

The following fields can be overridden:

```text
deviceIp
path
tag
port
```

For `deviceIp`, `path`, and `tag`, a non-empty incoming value takes precedence over the configured value.

For `port`, an incoming numeric value takes precedence over the configured value.

### Current Read Behavior

The current implementation simulates an EtherNet/IP read operation.

The returned value is generated using a random numeric value in the following range:

```text
0 <= value < 100
```

The returned data type is currently:

```text
REAL
```

A timestamp is generated for every successful read.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow data. Supported fields can override the configured EtherNet/IP parameters. |

The node primarily uses the configured parameters, but incoming workflow data can provide `deviceIp`, `path`, `tag`, or `port`.

Example input:

```json
{
  "deviceIp": "192.168.1.20",
  "path": "1,0",
  "tag": "Temperature",
  "port": 44818
}
```

### Outputs

| Output Field | Type | Description |
|--------------|------|-------------|
| `success` | `boolean` | Indicates whether the simulated read completed successfully. |
| `deviceIp` | `string` | Device IP address used for the read. |
| `path` | `string` | CIP path used for the read. |
| `tag` | `string` | Tag used for the read. |
| `value` | `number` | Simulated tag value. |
| `dataType` | `string` | Data type of the returned value. Currently `REAL`. |
| `timestamp` | `string` | ISO timestamp generated when the value is returned. |

### Successful Read

Example:

```json
{
  "success": true,
  "deviceIp": "192.168.1.10",
  "path": "1,0",
  "tag": "TestTag",
  "value": 15.085182614122628,
  "dataType": "REAL",
  "timestamp": "2026-09-01T10:24:22.063Z"
}
```

The numeric `value` changes between executions because the current implementation generates a simulated value.

### Error Behavior

The node throws an error when the required tag cannot be resolved.

Example:

```text
Tag name is required
```

Errors occurring inside the EtherNet/IP read operation are wrapped as:

```text
EtherNet/IP read error: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Read a Tag

Read a tag using the standard EtherNet/IP configuration.

**Configuration**

```text
Device IP: 192.168.1.10
Path: 1,0
Tag: TestTag
Port: 44818
```

**Output**

```json
{
  "success": true,
  "deviceIp": "192.168.1.10",
  "path": "1,0",
  "tag": "TestTag",
  "value": 15.085182614122628,
  "dataType": "REAL",
  "timestamp": "2026-09-01T10:24:22.063Z"
}
```

---

### Example: Use Default Path

Configure the required parameters and use the default CIP path.

**Configuration**

```text
Device IP: 192.168.1.10
Tag: TestTag
Port: 44818
```

The node uses:

```text
Path: 1,0
```

---

### Example: Use Default Port

Configure the device, path, and tag without specifying a custom port.

**Configuration**

```text
Device IP: 192.168.1.10
Path: 1,0
Tag: TestTag
```

The node uses:

```text
Port: 44818
```

---

### Example: Override Tag from Input

Incoming workflow data can override the configured tag.

**Configured Parameters**

```text
Device IP: 192.168.1.10
Tag: TestTag
```

**Input**

```json
{
  "tag": "Temperature"
}
```

The node uses:

```text
Tag: Temperature
```

for that execution.

---

### Example: Override Device and Tag

Incoming workflow data can override multiple configured parameters.

**Input**

```json
{
  "deviceIp": "192.168.1.20",
  "tag": "Pressure",
  "path": "1,0",
  "port": 44818
}
```

The returned output reflects the values used for that execution.

---

### Example: Missing Tag

If no tag can be resolved, the node throws:

```text
Tag name is required
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Read EtherNet/IP Tag Value
```

### Common Patterns

- **Basic Tag Read:** Manual Trigger → EtherNet/IP Read → Log
- **Industrial Monitoring:** Manual Trigger → EtherNet/IP Read → Processing Node
- **Telemetry Logging:** EtherNet/IP Read → Log
- **Industrial Data Processing:** EtherNet/IP Read → Data Processing → Storage
- **Dynamic Tag Read:** Previous Node → EtherNet/IP Read → Log

The included example workflow uses:

```text
Manual Trigger → EtherNet/IP Read → Log
```

with:

```text
Device IP: 192.168.1.10
Tag: TestTag
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "Device IP address is required"

**Cause**

The configured `deviceIp` is empty or missing.

**Solution**

Provide a non-empty device IP address.

Example:

```text
192.168.1.10
```

---

#### "Tag name is required"

**Cause**

No tag could be resolved from either the incoming workflow data or the configured `tag` parameter.

**Solution**

Configure a tag name or provide a valid `tag` value from the previous workflow node.

Example:

```text
TestTag
```

---

#### No Connection to the Physical Device

**Cause**

The current implementation simulates EtherNet/IP reads and does not establish a real connection to the configured device.

**Solution**

The current implementation can be used to test node behavior and workflow integration.

Connecting to a physical EtherNet/IP controller requires an EtherNet/IP client implementation.

---

#### Value Changes Between Executions

**Cause**

The current implementation generates the returned value using a random number.

The simulated value is generated in the following range:

```text
0 <= value < 100
```

**Solution**

This is expected behavior for the current implementation.

---

#### Unexpected Configuration Values

**Cause**

Incoming workflow data may override the values configured directly on the node.

The following incoming fields are supported:

```text
deviceIp
path
tag
port
```

**Solution**

Check the data received from the previous workflow node and verify whether one of these fields is overriding the node configuration.

---

#### EtherNet/IP Read Error

**Cause**

An error occurred during the read operation.

**Solution**

Check the underlying error message and verify the node configuration.

Errors generated during the read operation use the following format:

```text
EtherNet/IP read error: <error message>
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EtherNet/IP Trigger
- EtherNet/IP Write
- BACnet Read
- DNP3 Master

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial release of the EtherNet/IP Read documentation. |

<!-- /SECTION: changelog -->
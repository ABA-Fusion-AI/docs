---
node_id: "ethernet-ip-write"
title: "EtherNet/IP Write"
description: "Write tag values to EtherNet/IP devices."
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
  - write
  - automation
related_nodes:
  - ethernet-ip-trigger
  - ethernet-ip-read
  - bacnet-write
  - profinet-write
---

<!-- SECTION: header -->
# EtherNet/IP Write

> **Category:** Industrial & IoT Protocols | **Type:** Action Node

Write tag values to EtherNet/IP devices using configurable device, path, tag, value, and port parameters.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EtherNet/IP Write** node writes a configured value to a tag associated with an EtherNet/IP device.

EtherNet/IP is an industrial communication protocol that adapts the Common Industrial Protocol (CIP) to standard Ethernet networks.

The node accepts a device IP address, CIP path, tag name, value, and port. These values can be configured directly on the node or, when provided, overridden by incoming workflow data.

The current implementation simulates the EtherNet/IP write operation. It returns the value that would be written but does not currently establish a connection to a physical EtherNet/IP controller.

### Key Features

- **Tag Writing:** Writes a configured value to an EtherNet/IP tag.
- **Device Configuration:** Supports configurable device IP addresses.
- **CIP Path Support:** Supports a configurable CIP routing path.
- **Custom Tag:** Targets the configured tag name.
- **Flexible Values:** Accepts values without restricting them to a specific data type.
- **Custom Port:** Supports a configurable EtherNet/IP port.
- **Input Overrides:** Allows incoming workflow data to override configured values.
- **Structured Output:** Returns device information, tag information, written value, and timestamp.
- **Validation:** Requires a device IP address and tag name, and rejects write operations when the resolved value is `undefined`.
- **Simulated Writes:** Supports workflow testing without requiring a physical controller.

### Use Cases

- Write industrial equipment values
- Update PLC tag values
- Build industrial control workflows
- Test EtherNet/IP write integrations
- Simulate PLC write operations
- Send dynamically generated values to industrial workflows
- Build industrial automation pipelines
- Connect application data to industrial control workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `deviceIp` | `string` | ✅ Yes | — | IP address of the EtherNet/IP device. |
| `path` | `string` | ❌ No | `1,0` | CIP routing path used for the device. |
| `tag` | `string` | ✅ Yes | — | Name of the tag to write. |
| `value` | `any` | ✅ Yes | — | Value to write to the configured tag. |
| `port` | `number` | ❌ No | `44818` | EtherNet/IP communication port. |

### Default Values

| Parameter | Default |
|-----------|---------|
| `path` | `1,0` |
| `port` | `44818` |

### Device IP

`deviceIp` identifies the EtherNet/IP device associated with the target tag.

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

`tag` specifies the tag to write.

Example:

```text
TestTag
```

The configured value is required and cannot be empty.

### Value

`value` specifies the value to write to the target tag.

Example:

```text
25
```

The node accepts any value type. A value must be provided for the write operation.

### Port

`port` specifies the EtherNet/IP communication port.

Default:

```text
44818
```

### Input Override Behavior

The node can use values received from incoming workflow data instead of the configured values.

The following fields can be overridden:

```text
deviceIp
path
tag
value
port
```

For `deviceIp`, `path`, and `tag`, a non-empty incoming value takes precedence over the configured value.

For `value`, any incoming value other than `undefined` takes precedence over the configured value.

For `port`, an incoming numeric value takes precedence over the configured value.

### Current Write Behavior

The current implementation simulates an EtherNet/IP write operation.

No connection is currently established with the configured EtherNet/IP device, and no value is physically written to a controller.

On a successful simulated operation, the node returns the device information, tag, supplied value, and execution timestamp.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow data. Supported fields can override the configured EtherNet/IP parameters. |

The node primarily uses the configured parameters, but incoming workflow data can provide `deviceIp`, `path`, `tag`, `value`, or `port`.

Example input:

```json
{
  "deviceIp": "192.168.1.20",
  "path": "1,0",
  "tag": "SetPoint",
  "value": 50,
  "port": 44818
}
```

### Outputs

| Output Field | Type | Description |
|--------------|------|-------------|
| `success` | `boolean` | Indicates whether the simulated write completed successfully. |
| `deviceIp` | `string` | Device IP address used for the write. |
| `path` | `string` | CIP path used for the write. |
| `tag` | `string` | Tag used for the write. |
| `value` | `any` | Value supplied to the write operation. |
| `timestamp` | `string` | ISO timestamp generated when the operation completes. |

### Successful Write

Example:

```json
{
  "success": true,
  "deviceIp": "192.168.1.10",
  "path": "1,0",
  "tag": "TestTag",
  "value": 25,
  "timestamp": "2026-09-01T10:32:26.544Z"
}
```

The returned `value` corresponds to the value supplied to the simulated write operation.

### Error Behavior

The node throws an error when the required tag cannot be resolved.

Example:

```text
Tag name is required
```

The node also throws an error when no write value can be resolved.

Example:

```text
Value is required for write operation
```

Errors occurring inside the EtherNet/IP write operation are wrapped as:

```text
EtherNet/IP write error: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Write a Tag Value

Write a numeric value using the standard EtherNet/IP configuration.

**Configuration**

```text
Device IP: 192.168.1.10
Path: 1,0
Tag: TestTag
Value: 25
Port: 44818
```

**Output**

```json
{
  "success": true,
  "deviceIp": "192.168.1.10",
  "path": "1,0",
  "tag": "TestTag",
  "value": 25,
  "timestamp": "2026-09-01T10:32:26.544Z"
}
```

---

### Example: Use Default Path

Configure the required parameters and use the default CIP path.

**Configuration**

```text
Device IP: 192.168.1.10
Tag: TestTag
Value: 25
Port: 44818
```

The node uses:

```text
Path: 1,0
```

---

### Example: Use Default Port

Configure the device, path, tag, and value without specifying a custom port.

**Configuration**

```text
Device IP: 192.168.1.10
Path: 1,0
Tag: TestTag
Value: 25
```

The node uses:

```text
Port: 44818
```

---

### Example: Override Value from Input

Incoming workflow data can override the configured value.

**Configured Parameters**

```text
Device IP: 192.168.1.10
Tag: TestTag
Value: 25
```

**Input**

```json
{
  "value": 50
}
```

The node uses:

```text
Value: 50
```

for that execution.

---

### Example: Override Tag and Value

Incoming workflow data can override multiple configured parameters.

**Input**

```json
{
  "tag": "TemperatureSetPoint",
  "value": 30
}
```

The node uses these values for that execution while retaining the other configured parameters.

---

### Example: Missing Tag

If no tag can be resolved, the node throws:

```text
Tag name is required
```

---

### Example: Missing Value

If the value resolves to `undefined`, the node throws:

```text
Value is required for write operation
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Write EtherNet/IP Tag Value
```

### Common Patterns

- **Basic Tag Write:** Manual Trigger → EtherNet/IP Write → Log
- **Industrial Control:** Manual Trigger → EtherNet/IP Write → Processing Node
- **Dynamic Value Write:** Previous Node → EtherNet/IP Write → Log
- **Industrial Automation:** Data Processing → EtherNet/IP Write → Log
- **Calculated Value Write:** Calculation Node → EtherNet/IP Write → Log

The included example workflow uses:

```text
Manual Trigger → EtherNet/IP Write → Log
```

with:

```text
Device IP: 192.168.1.10
Tag: TestTag
Value: 25
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

#### "Value is required for write operation"

**Cause**

The write value resolved to `undefined`.

**Solution**

Configure a value on the node or provide a `value` from the previous workflow node.

Example:

```text
25
```

Values such as `0`, `false`, an empty string, or `null` are not treated as `undefined` by the write-value check.

---

#### No Value Written to the Physical Device

**Cause**

The current implementation simulates EtherNet/IP writes and does not establish a real connection to the configured device.

**Solution**

The current implementation can be used to test node behavior and workflow integration.

Writing to a physical EtherNet/IP controller requires an EtherNet/IP client implementation.

---

#### Unexpected Configuration Values

**Cause**

Incoming workflow data may override the values configured directly on the node.

The following incoming fields are supported:

```text
deviceIp
path
tag
value
port
```

**Solution**

Check the data received from the previous workflow node and verify whether one of these fields is overriding the node configuration.

---

#### EtherNet/IP Write Error

**Cause**

An error occurred during the write operation.

**Solution**

Check the underlying error message and verify the node configuration.

Errors generated during the write operation use the following format:

```text
EtherNet/IP write error: <error message>
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EtherNet/IP Trigger
- EtherNet/IP Read
- BACnet Write
- PROFINET Write

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial release of the EtherNet/IP Write documentation. |

<!-- /SECTION: changelog -->
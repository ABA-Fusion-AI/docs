---
node_id: "ethernet-ip-trigger"
title: "EtherNet/IP Trigger"
description: "Monitor EtherNet/IP tag values and trigger workflows when values change."
category: "triggers-ingress"
subcategory: "iot-industrial-ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - ethernet-ip
  - industrial
  - iot
  - trigger
  - cip
  - automation
  - monitoring
related_nodes:
  - ethernet-ip-read
  - ethernet-ip-write
  - modbus-trigger
  - opc-trigger
---

<!-- SECTION: header -->
# EtherNet/IP Trigger

> **Category:** Triggers & Ingress | **Type:** Trigger Node

Monitor EtherNet/IP tag values at configurable intervals and trigger workflows when monitored values change.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EtherNet/IP Trigger** node monitors a configured tag associated with an EtherNet/IP device and triggers the workflow when the monitored value changes.

EtherNet/IP is an industrial communication protocol that adapts the Common Industrial Protocol (CIP) to standard Ethernet networks.

The node supports configurable polling intervals, optional change thresholds, first-read triggering, custom device addresses, CIP paths, tags, and ports.

The current implementation simulates tag reads by generating numeric values periodically. It does not currently establish a connection to a physical EtherNet/IP controller.

### Key Features

- **Tag Monitoring:** Monitors a configured EtherNet/IP tag.
- **Change Detection:** Triggers the workflow when the monitored value changes.
- **Change Threshold:** Optionally requires a minimum numeric difference before triggering.
- **Configurable Polling:** Controls how frequently the tag value is checked.
- **First-Read Trigger:** Optionally triggers the workflow on the first value read.
- **Custom Device Configuration:** Supports device IP, CIP path, tag name, and port configuration.
- **Continuous Monitoring:** Continues polling while the trigger is running.
- **Lifecycle Support:** Supports pause, resume, and stop behavior.
- **Error Output:** Provides a dedicated error output for polling failures.

### Use Cases

- Monitor industrial equipment values
- Detect changes in PLC tags
- Trigger workflows from industrial data changes
- Build industrial monitoring automations
- Test EtherNet/IP workflow integrations
- Simulate industrial telemetry events
- Monitor threshold-based value changes
- Connect industrial events to downstream workflow nodes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `deviceIp` | `string` | ✅ Yes | — | IP address of the EtherNet/IP device to monitor. |
| `path` | `string` | ❌ No | `1,0` | CIP routing path used for the device. |
| `tag` | `string` | ✅ Yes | — | Name of the tag to monitor. |
| `port` | `number` | ❌ No | `44818` | EtherNet/IP communication port. |
| `pollInterval` | `number` | ❌ No | `1000` | Interval between polling operations in milliseconds. Minimum value is `100`. |
| `changeThreshold` | `number` | ❌ No | — | Optional minimum numeric difference required before triggering. |
| `triggerOnFirstRead` | `boolean` | ❌ No | `true` | Trigger the workflow when the first value is read. |

### Default Values

| Parameter | Default |
|-----------|---------|
| `path` | `1,0` |
| `port` | `44818` |
| `pollInterval` | `1000` |
| `triggerOnFirstRead` | `true` |

### Device IP

`deviceIp` identifies the EtherNet/IP device associated with the monitored tag.

Example:

```text
192.168.1.10
```

The value is required and cannot be empty.

### CIP Path

`path` specifies the CIP routing path.

Default:

```text
1,0
```

### Tag

`tag` specifies the tag to monitor.

Example:

```text
TestTag
```

The value is required and cannot be empty.

### Port

`port` specifies the EtherNet/IP communication port.

Default:

```text
44818
```

### Poll Interval

`pollInterval` determines how frequently the node checks the monitored value.

The value is expressed in milliseconds.

Default:

```text
1000
```

Minimum:

```text
100
```

For example, a value of `1000` causes the node to perform one polling cycle approximately every second.

### Change Threshold

`changeThreshold` optionally defines the minimum absolute difference required between the current value and the previous value before the workflow is triggered.

When configured, the node calculates:

```text
absolute difference = |current value - previous value|
```

The workflow triggers when the calculated difference is greater than or equal to the configured threshold.

When `changeThreshold` is omitted, any value change can trigger the workflow.

### Trigger on First Read

When `triggerOnFirstRead` is enabled, the first generated value triggers the workflow.

Default:

```text
true
```

When disabled, the first value is stored and used as the baseline for subsequent change detection.

### Current Read Behavior

The current implementation uses simulated numeric values:

```text
0 <= value < 100
```

A new simulated value is generated during every polling cycle.

The emitted data type is currently:

```text
REAL
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The **EtherNet/IP Trigger** is a trigger node and does not require an incoming workflow connection.

Its behavior is controlled through the configured parameters.

### Outputs

#### Success

The `success` output is emitted when the configured trigger condition is satisfied.

| Output Field | Type | Description |
|--------------|------|-------------|
| `deviceIp` | `string` | Configured device IP address. |
| `path` | `string` | Configured CIP path. |
| `tag` | `string` | Monitored tag name. |
| `value` | `number` | Current simulated tag value. |
| `previousValue` | `number \| null` | Previous monitored value. On the first read, the implementation may initialize it with the current value. |
| `dataType` | `string` | Data type of the monitored value. Currently `REAL`. |
| `timestamp` | `string` | ISO timestamp generated when the event is emitted. |

Example:

```json
{
  "deviceIp": "192.168.1.10",
  "path": "1,0",
  "tag": "TestTag",
  "value": 78.95472889540534,
  "previousValue": 60.95744224895089,
  "dataType": "REAL",
  "timestamp": "2026-09-01T09:45:00.000Z"
}
```

#### Error

The `error` output is emitted when an error occurs during a polling operation.

| Output Field | Type | Description |
|--------------|------|-------------|
| `error` | `string` | Error message. |
| `deviceIp` | `string` | Configured device IP address. |
| `tag` | `string` | Configured tag name. |

Example:

```json
{
  "error": "Connection timeout",
  "deviceIp": "192.168.1.10",
  "tag": "TestTag"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Monitor a Tag

Monitor a tag using the default polling behavior.

**Configuration**

```text
Device IP: 192.168.1.10
Path: 1,0
Tag: TestTag
Port: 44818
Poll Interval: 1000
Change Threshold:
Trigger On First Read: true
```

With this configuration, the node generates a simulated value approximately every second.

Because no `changeThreshold` is configured, a workflow event is emitted whenever the current value differs from the previous value.

---

### Example: Trigger on First Read

Enable immediate triggering when the first value is generated.

**Configuration**

```text
Device IP: 192.168.1.10
Tag: TestTag
Poll Interval: 1000
Trigger On First Read: true
```

The first polling cycle initializes the monitored value and triggers the workflow.

---

### Example: Disable First-Read Trigger

Use the first value only as the initial comparison value.

**Configuration**

```text
Device IP: 192.168.1.10
Tag: TestTag
Poll Interval: 1000
Trigger On First Read: false
```

The first generated value is stored without triggering the workflow.

Subsequent values are compared with the stored value.

---

### Example: Change Threshold

Trigger only when the numeric difference reaches a configured threshold.

**Configuration**

```text
Device IP: 192.168.1.10
Tag: TestTag
Poll Interval: 1000
Change Threshold: 10
Trigger On First Read: true
```

After the first read, the workflow triggers when:

```text
|currentValue - previousValue| >= 10
```

---

### Example: Faster Polling

Poll every 500 milliseconds.

**Configuration**

```text
Device IP: 192.168.1.10
Tag: TestTag
Poll Interval: 500
Trigger On First Read: true
```

The node generates a new simulated reading approximately twice per second.

---

### Example: Custom Port and Path

Configure a custom port and CIP path.

**Configuration**

```text
Device IP: 192.168.1.10
Path: 1,0
Tag: TestTag
Port: 44818
Poll Interval: 1000
Trigger On First Read: true
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Monitor EtherNet/IP Tag Changes
```

### Common Patterns

- **Industrial Monitoring:** EtherNet/IP Trigger → Log
- **Tag Change Event:** EtherNet/IP Trigger → Processing Node
- **Industrial Alerting:** EtherNet/IP Trigger → Notification Node
- **Telemetry Processing:** EtherNet/IP Trigger → Data Processing → Storage
- **Threshold Monitoring:** EtherNet/IP Trigger → Conditional Processing

The example workflow uses:

```text
EtherNet/IP Trigger → Log
```

with the following primary configuration:

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

The `deviceIp` parameter is empty.

**Solution**

Provide a non-empty device IP address.

Example:

```text
192.168.1.10
```

---

#### "Tag name is required"

**Cause**

The `tag` parameter is empty.

**Solution**

Provide the name of the tag to monitor.

Example:

```text
TestTag
```

---

#### Invalid Poll Interval

**Cause**

The configured `pollInterval` is lower than the minimum supported value.

The node requires:

```text
pollInterval >= 100
```

**Solution**

Configure a polling interval of at least `100` milliseconds.

---

#### Workflow Triggers Frequently

**Cause**

No `changeThreshold` is configured.

The current implementation generates random simulated values, so consecutive values are normally different.

**Solution**

Configure a `changeThreshold` if the workflow should only trigger for larger changes.

---

#### No Connection to the Physical Device

**Cause**

The current node implementation simulates EtherNet/IP tag reads and does not establish a real EtherNet/IP connection.

**Solution**

The current implementation can be used to test trigger behavior and workflow integration. Connecting to a physical EtherNet/IP controller requires a real EtherNet/IP client implementation.

---

#### First Event Previous Value

**Behavior**

On the first triggered read, the implementation initializes the stored value before emitting the event.

As a result, the first event can contain the same value for `value` and `previousValue`.

Subsequent events use the previously stored value for comparison.

---

#### Polling Error

**Cause**

An error occurred during the polling callback.

**Solution**

Check the error output and verify the configured node parameters.

Example error output:

```json
{
  "error": "Error message",
  "deviceIp": "192.168.1.10",
  "tag": "TestTag"
}
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EtherNet/IP Read
- EtherNet/IP Write
- Modbus Trigger
- OPC Trigger

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial release of the EtherNet/IP Trigger documentation. |

<!-- /SECTION: changelog -->
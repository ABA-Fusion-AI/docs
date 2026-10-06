---
node_id: "dnp3-master"
title: "DNP3 Master"
description: "Communicate with DNP3 outstations (RTUs/IEDs) over TCP or Serial to poll telemetry, issue control commands, freeze counters, and synchronize time."
category: "APIs & Protocols"
subcategory: "Industrial & IoT Protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags:
  - dnp3
  - scada
  - industrial
  - iot
  - rtu
  - ied
  - ieee-1815
  - telemetry
related_nodes:
  - dnp3-outstation
  - modbus-read
  - modbus-write
  - opc
---

<!-- SECTION: overview -->
# DNP3 Master

> **Category:** APIs & Protocols / Industrial & IoT Protocols&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Communicate with DNP3 outstations, Remote Terminal Units (RTUs), and Intelligent Electronic Devices (IEDs) using the international standard **IEEE Std 1815 (DNP3)** protocol over TCP network sockets or Serial interfaces.

The **DNP3 Master** node acts as a centralized supervisory client. It enables workflows to poll real-time telemetry (analog values, binary statuses, accumulators/counters), issue control commands (direct operate, select-before-operate), initiate counter freeze operations, and synchronize outstation clocks with 48-bit UTC time markers.

### Key Features

- **Pure TypeScript IEEE Std 1815 Engine:** Zero native C++ compilation (`node-gyp`) dependencies, ensuring reliable operation across all Docker, Linux, and Cloud environments.
- **Dual Transport Layer Support:** Operates over network TCP sockets (default port `20000`) and serial communication lines (`RS-232`/`RS-485`).
- **Comprehensive Point Type Matrix:** Full support for 7 standardized DNP3 point categories:
  - Binary Inputs (Group 1) & Binary Input Events (Group 2)
  - Binary Outputs / CROB (Group 10 & Group 12)
  - Counters (Group 20) & Frozen Counters (Group 21)
  - Analog Inputs (Group 30) & Analog Input Events (Group 32)
  - Analog Outputs / Setpoints (Group 40 & Group 41)
- **Standard DNP3 Function Operations:** Execute `read`, `write`, `select`, `operate`, `freeze`, and `time-sync`.
- **Industrial Integrity & Framing:** FT3 frame formatting with CRC-16 checksum validation (`0xA6BC` polynomial) and internal indication (IIN) flag evaluation.
- **Fail-Safe Error Routing:** Socket timeouts, network unreachable conditions, and protocol errors are routed safely to the **error** handle without crashing the host workflow.
- **Cloud-Safe Fallback:** Seamless simulation fallback for sandbox environments and test harnesses where physical outstations are unreachable.

### Use Cases

- **Substation & Grid Telemetry:** Periodically poll voltage, frequency, power factor, and breaker statuses from power grid IEDs.
- **Industrial Control & Automation:** Dispatch Select-Before-Operate (SBO) or Direct Operate commands to open/close switches, valves, or relays.
- **Flow Meter & Metering Accumulation:** Freeze and retrieve counter values at scheduled billing intervals.
- **Clock Synchronization:** Broadcast standard 48-bit UTC timestamps across remote RTU stations to maintain accurate event sequence logs (SOE).
<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `transport` | `enum` | ❌ No | `"TCP"` | Transport protocol to use: `"TCP"` for Ethernet/IP network connections, or `"Serial"` for RS-232/RS-485 connections. |
| `host` | `string` | ❌ No | `"localhost"` | Destination IP address or hostname of the DNP3 Outstation. In Cloud environments, specify the reachable SCADA gateway, VPN IP (e.g. `10.x.x.x`), public IP, or FQDN (e.g. `scada.plant.example.com`). Visible when `transport` is `"TCP"`. |
| `port` | `number` | ❌ No | `20000` | Destination TCP port of the outstation (standard DNP3 port is `20000`). Visible when `transport` is `"TCP"`. |
| `serialPort` | `string` | ❌ No | — | Serial port path (e.g. `/dev/ttyUSB0` on Linux or `COM3` on Windows). Visible when `transport` is `"Serial"`. |
| `baudRate` | `number` | ❌ No | `9600` | Baud rate for serial communication (e.g., `9600`, `19200`, `38400`, `115200`). Visible when `transport` is `"Serial"`. |
| `outstationAddress` | `number` | ❌ No | `1` | DNP3 Link Layer destination address of the remote outstation (`0` to `65535`). |

### Operation Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ❌ No | `"read"` | The DNP3 function to execute: `"read"`, `"write"`, `"select"`, `"operate"`, `"freeze"`, or `"time-sync"`. |
| `pointType` | `enum` | ❌ No | `"analog-input"` | Target DNP3 object group: `"binary-input"`, `"binary-output"`, `"analog-input"`, `"analog-output"`, `"counter"`, `"binary-input-event"`, or `"analog-input-event"`. |
| `startIndex` | `number` | ❌ No | `0` | Zero-based starting point index to access (minimum `0`). |
| `count` | `number` | ❌ No | `1` | Number of contiguous points to read or operate on (minimum `1`). |
| `value` | `any` | ❌ No | — | Value to write or command to issue. Required when `operation` is `"write"` or `"operate"`. For binary points, supply boolean or `0`/`1`; for analog points, supply a numeric setpoint. |

### Available Operations

| Operation | Function Code | Description |
|-----------|---------------|-------------|
| `read` | `0x01` (READ) | Polls the outstation for current point values or change events within the requested index range. |
| `write` | `0x02` (WRITE) | Writes values directly to analog or binary output points. |
| `select` | `0x03` (SELECT) | Arms a control point in Select-Before-Operate (SBO) sequences. |
| `operate` | `0x04` (OPERATE) / `0x05` (DIRECT_OPERATE) | Executes a control operation (e.g., trip/close breaker, pulse relay, analog setpoint). |
| `freeze` | `0x07` (IMMEDIATE_FREEZE) | Copies current running counter values into frozen counter buffers. |
| `time-sync` | `0x18` (WRITE TIME) | Writes the current 48-bit UTC millisecond timestamp to synchronize the outstation's internal clock. |
<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | An incoming payload that triggers node execution. Fields in the incoming payload can dynamically override configured values. |

#### Input Overrides

Any property in the incoming data object can override the node's static configuration for a single execution:

| Payload Field | Overrides Config | Type | Example |
|---------------|------------------|------|---------|
| `operation` | `operation` | `string` | `"read"` |
| `pointType` | `pointType` | `string` | `"analog-input"` |
| `startIndex` | `startIndex` | `number` | `10` |
| `count` | `count` | `number` | `4` |
| `value` | `value` | `any` | `42.5` |
| `host` | `host` | `string` | `"192.168.1.120"` |
| `port` | `port` | `number` | `20000` |
| `outstationAddress` | `outstationAddress` | `number` | `2` |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the DNP3 transaction completes successfully. |
| `error` | `Error` | Emitted when socket connection, validation, or protocol execution fails. |

### Success Payload Schema

```json
{
  "success": true,
  "transport": "TCP",
  "host": "192.168.1.100",
  "port": 20000,
  "outstationAddress": 1,
  "operation": "read",
  "pointType": "analog-input",
  "startIndex": 0,
  "count": 2,
  "values": [
    {
      "index": 0,
      "value": 24.5,
      "quality": "GOOD",
      "timestamp": "2026-10-06T10:00:00.000Z"
    },
    {
      "index": 1,
      "value": 26.2,
      "quality": "GOOD",
      "timestamp": "2026-10-06T10:00:00.000Z"
    }
  ],
  "timestamp": "2026-10-06T10:00:00.000Z",
  "mode": "real-wire"
}
```

#### Fields Description

- **`success`** (`boolean`): Indicates whether the request was successful.
- **`transport`** (`string`): Transport used (`"TCP"` or `"Serial"`).
- **`outstationAddress`** (`number`): DNP3 address of the target outstation.
- **`operation`** (`string`): The executed operation (`"read"`, `"write"`, etc.).
- **`pointType`** (`string`): Point type that was accessed.
- **`startIndex`** / **`count`** (`number`): Point index range.
- **`values`** (`array`): Array of point records containing:
  - `index`: Zero-based point index.
  - `value`: Numerical or boolean point value.
  - `quality`: DNP3 quality indicator (`"GOOD"`, `"ONLINE"`, etc.).
  - `timestamp`: Point timestamp formatted in ISO 8601.
- **`mode`** (`string`): `"real-wire"` when communicating with a physical device over TCP, or `"simulated"` when operating in test/sandbox fallback mode.

### Error Payload Schema

When a network failure or misconfiguration occurs, the node routes an error object to the `error` port:

```json
{
  "message": "DNP3 operation error: DNP3 TCP connection to 192.168.1.100:20000 timed out after 3000ms"
}
```
<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Poll DNP3 Analog Telemetry and Log Results
```

In this workflow:
1. **Manual Trigger** initiates the poll on demand.
2. **DNP3 Master** connects over TCP to an outstation at `192.168.1.100:20000` (Address `1`) and reads 5 analog inputs starting at index `0`.
3. **Log** outputs the structured telemetry records and quality flags to the console.

### Common Patterns

- **Scheduled Telemetry Ingestion:** Precede the DNP3 Master with a **Cron** or **Interval Trigger** to poll feeder voltages every 10 seconds, followed by an **InfluxDB** or **PostgreSQL** node to store time-series telemetry.
- **Threshold Alerting:** Read analog values, evaluate temperatures or pressures with a **Condition / Filter Node**, and trigger a **Slack** or **PagerDuty** notification if limits are exceeded.
- **Select-Before-Operate Control:** Wire two DNP3 Master nodes in sequence—the first executing `operation: "select"` and the second executing `operation: "operate"`—to guarantee safe control execution.
<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | Cause | Resolution |
|---------|-------|------------|
| `Connection timed out after 3000ms` | The target outstation IP or port is unreachable, or a firewall is blocking port `20000`. | Check network routing, ensure the outstation service is running, and verify firewall rules allow inbound TCP traffic on port `20000`. |
| `Serial port is required for Serial transport` | `transport` is set to `"Serial"` but `serialPort` was left empty. | Specify the serial device identifier (e.g. `/dev/ttyUSB0` or `COM3`) in the node configuration. |
| `Outstation address mismatch` | The remote device does not respond to Link Layer frames. | Verify the outstation's configured DNP3 link address matches `outstationAddress` in the node parameters. |
| `Data quality reported as OFFLINE or COMM_LOST` | The outstation is unable to read values from physical sensors. | Inspect physical sensor wiring and field module health on the RTU/IED. |
| `Unexpected float value decoding` | Standard IEEE-754 single precision float mismatch. | Verify the outstation points correspond to Group 30 Variation 1 (32-bit floating point with flags) or configure the matching group index. |
<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [DNP3 Outstation](./dnp3-outstation.md) — Listen for incoming DNP3 master control commands and trigger automation workflows.
- [Modbus Read](../modbus-read/en.md) — Read registers and coils over Modbus TCP/RTU.
- [Modbus Write](../modbus-write/en.md) — Write coils and holding registers on industrial PLCs.
- [OPC UA](../opc/en.md) — Connect and subscribe to industrial OPC UA servers.
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Initial release: Pure TypeScript IEEE Std 1815 binary protocol stack with TCP/Serial support, 7 point types, 6 operations, and cloud-safe fallback. |
<!-- /SECTION: changelog -->

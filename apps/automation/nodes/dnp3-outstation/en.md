---
node_id: "dnp3-outstation"
title: "DNP3 Outstation"
description: "Act as an IEEE Std 1815 DNP3 Outstation (RTU/IED) listening on TCP or Serial, serving telemetry and triggering workflows on incoming control commands."
category: "Triggers & Ingress"
subcategory: "IoT & Industrial Ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags:
  - dnp3
  - outstation
  - rtu
  - ied
  - trigger
  - industrial
  - iot
  - scada
  - ieee-1815
related_nodes:
  - dnp3-master
  - modbus-trigger
  - opc-trigger
  - scada-trigger
---

<!-- SECTION: overview -->
# DNP3 Outstation

> **Category:** Triggers & Ingress / IoT & Industrial Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Simulate or deploy an **IEEE Std 1815 (DNP3)** Outstation server, Remote Terminal Unit (RTU), or Intelligent Electronic Device (IED). The node listens for incoming master station connections over TCP or Serial lines, serves real-time telemetry from an in-memory point database, and immediately fires downstream workflow automation whenever a control operation or command is received.

Unlike polling triggers, the **DNP3 Outstation** functions as an event-driven ingress server. When a physical or remote SCADA Master dispatches commands—such as Direct Operate, Select-Before-Operate (SBO), or point writes—the node acknowledges the master according to official DNP3 protocol standards and triggers canvas execution in real time.

### Key Features

- **Embedded Pure TypeScript IEEE 1815 Server:** Fully self-contained DNP3 server running on native Node.js asynchronous sockets, free from external C++ dependencies or compilation tools.
- **Dual Transport Ingress:** Bind as a TCP listener on any network interface (default `0.0.0.0:20000`) or attach to a local serial bus (`RS-232`/`RS-485`).
- **Autonomous Telemetry Service:** Automatically responds to master poll requests (`READ` function code `0x01`) across analog inputs (Group 30), binary inputs (Group 1), and binary counters (Group 20) with valid CRC-16 checks.
- **Instantaneous Control Ingress:** Intercepts `SELECT` (`0x03`), `OPERATE` (`0x04`), and `DIRECT_OPERATE` (`0x05`) function codes to trigger workflows on the canvas.
- **Complete Node Lifecycle:** Fully managed `setup`, `run`, `pause`, `resume`, and `stop` phases ensuring clean socket teardown without port lockups or hanging file descriptors.

### Use Cases

- **Virtual SCADA Gateway & Bridge:** Emulate an RTU endpoint to receive SCADA commands and forward them to modern enterprise APIs, webhooks, or messaging brokers (Kafka, MQTT, RabbitMQ).
- **Control Command Auditing & Notification:** Intercept electrical grid switching orders or breaker trips and dispatch immediate alerts to on-duty engineers via Slack, Teams, or SMS.
- **Physical-to-Cloud Integration:** Bridge traditional industrial control systems with cloud workflows without deploying dedicated physical hardware protocol converters.
<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `transport` | `enum` | ❌ No | `"TCP"` | Ingress transport to host: `"TCP"` for Ethernet/IP network sockets, or `"Serial"` for serial communication. |
| `host` | `string` | ❌ No | `"0.0.0.0"` | Network interface IP to bind. Use `"0.0.0.0"` to accept connections from all network interfaces, or bind to a specific local interface (e.g., `127.0.0.1` or `192.168.1.50`). Visible when `transport` is `"TCP"`. |
| `port` | `number` | ❌ No | `20000` | Local TCP port to bind and listen on (standard DNP3 port is `20000`). Visible when `transport` is `"TCP"`. |
| `serialPort` | `string` | ❌ No | — | Serial port path (e.g., `/dev/ttyUSB0` or `COM1`). Visible when `transport` is `"Serial"`. |
| `baudRate` | `number` | ❌ No | `9600` | Baud rate for serial communication (e.g. `9600`, `19200`, `38400`, `115200`). Visible when `transport` is `"Serial"`. |
| `outstationAddress` | `number` | ❌ No | `1` | Local DNP3 Link Layer destination address (`0` to `65535`). Master requests must match this address to be processed. |
| `eventBufferSize` | `number` | ❌ No | `1000` | Maximum number of internal events retained in buffer memory (minimum `1`). |

---

## Operating Behavior & Point Database

When the workflow begins execution, the Outstation server starts listening on the configured socket and manages an internal 50-point telemetry database:

| Point Type | Object Group | Default Range | Initial State |
|------------|--------------|---------------|---------------|
| **Binary Inputs** | Group 1 Var 2 | Indices `0`–`49` | Alternating boolean states (`true`/`false`) |
| **Binary Outputs** | Group 10 Var 2 | Indices `0`–`49` | Defaulted to `false` |
| **Analog Inputs** | Group 30 Var 1 | Indices `0`–`49` | IEEE-754 32-bit floating points (e.g. `20.0`, `21.5`, etc.) |
| **Analog Outputs** | Group 40 Var 1 | Indices `0`–`49` | Defaulted to `0.0` |
| **Counters** | Group 20 Var 1 | Indices `0`–`49` | 32-bit integers starting from `1000` |

### Master Request Handling

1. **Read Requests (`0x01`):** The server parses incoming range headers, extracts the requested points from the in-memory database, computes CRC-16 checks for every 16-byte FT3 block, and returns a fully compliant DNP3 Application Response (`0x81`).
2. **Control Commands (`0x03`, `0x04`, `0x05`):** The server returns an affirmative DNP3 acknowledgment response to the master with clear Internal Indication flags (`IIN: 0x00 0x00`) and fires a `"control"` event to trigger downstream workflow nodes.
<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

As an ingress Trigger node, the **DNP3 Outstation** node initiates workflow execution and does not require an upstream input node.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted every time an external DNP3 Master executes a control command (`select`, `operate`, `direct_operate`). |
| `error` | `Error` | Emitted if socket binding fails or a fatal network error occurs. |

### Success Payload Schema

When a master sends a control command, the node emits the following payload to the `success` output:

```json
{
  "type": "control",
  "event": {
    "type": "control",
    "functionCode": 5,
    "timestamp": "2026-10-06T10:15:30.123Z"
  },
  "timestamp": "2026-10-06T10:15:30.123Z"
}
```

#### Fields Description

- **`type`** (`string`): Always `"control"` for command-triggered executions.
- **`event`** (`object`): Detailed event descriptor containing:
  - `type`: Trigger category (`"control"`).
  - `functionCode`: Numeric DNP3 function code received from the master:
    - `3`: Select (Select-Before-Operate)
    - `4`: Operate
    - `5`: Direct Operate
  - `timestamp`: ISO 8601 timestamp generated at the exact moment the command frame arrived.
- **`timestamp`** (`string`): Execution arrival timestamp.

### Error Payload Schema

In the event of a port conflict (e.g. port `20000` already bound by another process):

```json
{
  "message": "DNP3 outstation error: listen EADDRINUSE: address already in use 0.0.0.0:20000"
}
```
<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Trigger Workflow on DNP3 Master Control Operation
```

In this workflow:
1. **DNP3 Outstation** acts as a live server listening on `0.0.0.0:20000` (Address `1`).
2. An external DNP3 Master sends a Direct Operate command (`functionCode: 5`) over the network.
3. The Outstation node validates the frame, replies to the master, and fires the trigger.
4. **Log** receives the event and logs the function code and timestamp to the runtime audit log.

### Common Patterns

- **Industrial Command Logger:** Connect the Outstation trigger to a **Database Action** (PostgreSQL / MongoDB) to maintain an immutable audit trail of all SCADA operations.
- **Operational Alarm Dispatcher:** Connect the trigger output to a **Switch / Condition Node** that checks if `event.functionCode === 5`, routing critical commands to a notification service like **Telegram**, **Slack**, or **Twilio SMS**.
- **Automated Workflow Orchestration:** Use the Outstation trigger to launch complex enterprise maintenance workflows whenever an operator trips a virtual breaker from their SCADA HMI.
<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | Cause | Resolution |
|---------|-------|------------|
| `EADDRINUSE: address already in use` | Another service or DNP3 Outstation instance is already using port `20000`. | Change `port` in the configuration to an unused port (e.g. `20001`), or stop the conflicting service. |
| `EACCES: permission denied` | Attempting to bind to a privileged port (< 1024) without root/administrator privileges. | Use the standard DNP3 port `20000` or any non-privileged port (> 1024). |
| Master reports timeout when polling | Firewall blocking incoming TCP traffic on the listening port. | Open the configured TCP port (default `20000`) in the host firewall and cloud security groups. |
| Master receives no response to frames | DNP3 Link Layer destination address mismatch. | Ensure the master's destination address setting matches the `outstationAddress` configured in this node. |
| Workflows not triggering on master polls | Normal behavior. The trigger only fires on control commands (`operate`, `select`, `write`). | Regular `READ` telemetry polls are answered automatically in the background without triggering canvas execution. |
<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [DNP3 Master](../dnp3-master/en.md) — Connect to external DNP3 outstations to poll telemetry and dispatch control commands.
- [Modbus Trigger](../modbus-trigger/en.md) — Monitor Modbus registers or coils and trigger workflows on change.
- [OPC Trigger](../opc-trigger/en.md) — Trigger workflows on OPC UA subscription value changes.
- [SCADA Trigger](../scada-trigger/en.md) — Ingress events from broader industrial SCADA systems.
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Initial release: Event-driven pure TypeScript IEEE Std 1815 Outstation server with automatic telemetry response and control operation triggers. |
<!-- /SECTION: changelog -->

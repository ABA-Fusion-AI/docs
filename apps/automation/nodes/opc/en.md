---
node_id: "opc"
title: "OPC"
description: "Interact with OPC (OLE for Process Control) servers. Supports read, write, browse, and method call operations via an HTTP proxy bridge."
category: "security-and-networking"
subcategory: "industrial-and-iot-protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-09-14"
author: "Fusion Team"
tags:
  - opc
  - opc-ua
  - scada
  - industrial
  - iot
  - automation
  - plc
  - protocols
related_nodes:
  - opc-trigger
  - modbus-read
  - modbus-write
  - profinet-read
  - profinet-write
  - mqtt-publish
  - http-request
---

<!-- SECTION: header -->

# OPC

> **Category:** Security & Networking | **Subcategory:** Industrial & IoT Protocols | **Type:** Action Node

Interact with OPC (OLE for Process Control) and OPC UA servers to read sensor data, write control parameters, explore server address spaces, and execute remote method calls.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **OPC** node provides workflow integration for industrial automation systems and SCADA networks. It connects to OPC UA servers via a dedicated HTTP proxy gateway, allowing workflows to safely interact with PLCs, sensors, industrial equipment, and DCS servers.

### Architecture

Industrial OPC UA servers communicate over binary TCP protocols (`opc.tcp://`). The OPC Action Node interacts with an HTTP-to-OPC bridge/proxy service (such as `http://localhost:3039` or a secured gateway):

```
Workflow (OPC Node)  ──[HTTP/JSON with Bearer Auth]──>  OPC HTTP Proxy  ──[OPC UA TCP]──>  OPC Server / PLC
```

### Key Features

- **Four Core Operations:**
  - `read`: Read tag/node values, data types, timestamps, and quality status.
  - `write`: Update tag values with optional explicit data type casting.
  - `browse`: Inspect child nodes, objects, and variables in the OPC address space hierarchy.
  - `call`: Invoke remote RPC methods with typed input arguments.
- **Flexible Data Input:** Parameters can be configured statically in the builder or overridden dynamically by incoming upstream payload data.
- **Fallback Value Assignment:** For write operations, values can be supplied via configuration, payload properties, or directly from incoming upstream item data.
- **Built-in Safety Checks:** Validates endpoint protocol (HTTP/HTTPS) and alerts if `opc.tcp://` or raw TCP ports (e.g. `53530`) are configured by mistake.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Authentication token sent as `Authorization: Bearer <apiKey>` to the OPC HTTP proxy. |
| `operation` | `enum` | ❌ No | `read` | The operation to execute: `read`, `write`, `browse`, or `call`. |
| `endpointUrl` | `string` (url) | ✅ Yes | — | The HTTP/HTTPS base URL of the OPC Proxy service (e.g., `http://localhost:3039` or `https://opc-gateway.local`). |
| `nodeId` | `string` | Optional\* | — | OPC Node identifier (e.g., `ns=1;s=Temperature` or `ns=2;i=1001`). Required for `read` and `write`. Optional for `browse`. |
| `value` | `any` | Optional\* | — | Value to write to the specified node. Required for `write` (falls back to incoming data if omitted). |
| `dataType` | `string` | ❌ No | — | Optional data type hint for `write` (e.g., `Double`, `Int32`, `Boolean`, `String`). |
| `methodId` | `string` | Optional\* | — | OPC method Node identifier to invoke (e.g., `ns=1;s=ResetMachine`). Required for `call`. |
| `arguments` | `array` | ❌ No | `[]` | List of arguments passed to the method in `call` operation (e.g., `["urgent", 100]`). |

\* *Field requirement depends on the selected `operation`:*
- `read`: requires `nodeId` and `endpointUrl`.
- `write`: requires `nodeId`, `endpointUrl`, and `value` (or incoming item data).
- `browse`: requires `endpointUrl` (`nodeId` is optional; defaults to root).
- `call`: requires `endpointUrl` and `methodId`.

---

### Important: Endpoint URL Format

The `endpointUrl` must always point to the **HTTP proxy gateway** and **not** directly to the OPC UA TCP server:

| Correct (`endpointUrl`) | Incorrect | Reason |
|-------------------------|-----------|--------|
| `http://localhost:3039` | `opc.tcp://localhost:53530` | OPC Node sends REST HTTP requests; raw binary TCP is handled by the proxy. |
| `https://opc-proxy.company.internal` | `http://localhost:53530` | Port 53530 is usually the raw OPC TCP server port. |

---

### Dynamic Payload Overrides

Any parameter can be passed or overridden dynamically at runtime by setting the corresponding key in the incoming item payload:

```jsonc
{
  "operation": "write",
  "nodeId": "ns=1;s=ConveyorSpeed",
  "value": 1500,
  "dataType": "Int32"
}
```

If `value` is not defined in `payload.value` or `config.value`, the node uses the raw incoming `data` as the value to write.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming event or data stream containing trigger data or dynamic parameter overrides. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the OPC operation succeeds with the result payload. |
| `error` | `object` | Emitted when an error occurs during execution or validation. |

---

### Output Shapes

#### 1. Read Operation (`operation: "read"`)

```jsonc
{
  "success": true,
  "nodeId": "ns=1;s=Temperature",
  "value": 24.85,
  "dataType": "Double",
  "timestamp": "2026-09-14T16:00:00.000Z",
  "quality": "Good"
}
```

#### 2. Write Operation (`operation: "write"`)

```jsonc
{
  "success": true,
  "nodeId": "ns=1;s=Temperature",
  "value": 88.5,
  "dataType": "Double",
  "timestamp": "2026-09-14T16:00:05.120Z"
}
```

#### 3. Browse Operation (`operation: "browse"`)

```jsonc
{
  "success": true,
  "nodeId": "root",
  "nodes": [
    {
      "nodeId": "ns=0;i=85",
      "browseName": "Objects",
      "nodeClass": "Object"
    },
    {
      "nodeId": "ns=0;i=86",
      "browseName": "Types",
      "nodeClass": "Object"
    }
  ]
}
```

#### 4. Call Operation (`operation: "call"`)

```jsonc
{
  "success": true,
  "methodId": "ns=1;s=ResetMachine",
  "result": 0,
  "outputArguments": [
    "Machine reset successfully",
    true
  ]
}
```

#### 5. Error Response (Emitted on failure)

```jsonc
{
  "success": false,
  "error": "Node ID is required for read operation",
  "operation": "read"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### 1. Read a PLC Sensor Tag

Read a live temperature value from an industrial OPC UA server:

```json
{
  "apiKey": "{{ $secrets.OPC_PROXY_KEY }}",
  "endpointUrl": "http://localhost:3039",
  "operation": "read",
  "nodeId": "ns=1;s=Temperature"
}
```

### 2. Write a Target Setpoint

Write a new setpoint value (`88.5`) with explicit `Double` type formatting:

```json
{
  "apiKey": "{{ $secrets.OPC_PROXY_KEY }}",
  "endpointUrl": "http://localhost:3039",
  "operation": "write",
  "nodeId": "ns=1;s=TemperatureSetpoint",
  "value": 88.5,
  "dataType": "Double"
}
```

### 3. Browse Server Address Space

Discover available objects and variables starting from a root or specific namespace node:

```json
{
  "apiKey": "{{ $secrets.OPC_PROXY_KEY }}",
  "endpointUrl": "http://localhost:3039",
  "operation": "browse",
  "nodeId": "ns=2;s=ProductionLine1"
}
```

### 4. Execute a Server Method (RPC)

Trigger a remote routine on the PLC with arguments:

```json
{
  "apiKey": "{{ $secrets.OPC_PROXY_KEY }}",
  "endpointUrl": "http://localhost:3039",
  "operation": "call",
  "methodId": "ns=1;s=ResetMachine",
  "arguments": ["urgent", 100]
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Read, Write, Browse and Call OPC UA Server
```

### Practical Scenario: Industrial Monitoring & Automated Control

In this architecture:
1. A **Manual Trigger** or scheduled **Interval** starts the process cycle.
2. The **OPC** node reads sensor parameters (e.g. `ns=1;s=Temperature`).
3. An **If/Else** condition tests if the temperature exceeds a threshold.
4. If exceeded, a second **OPC** node executes a `write` operation to reduce speed or calls `call` to trigger cooling fans.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### `Endpoint URL must be the HTTP proxy URL...`

**Cause:** The `endpointUrl` was entered with an `opc.tcp://` protocol scheme.  
**Solution:** Change `endpointUrl` to the HTTP/HTTPS gateway proxy URL (e.g., `http://localhost:3039`).

#### `Endpoint URL points to the OPC UA TCP server port...`

**Cause:** The URL is using port `53530`, which is the OPC UA TCP binary port.  
**Solution:** Use the HTTP proxy port (e.g., `3039` or your configured gateway port) rather than the direct OPC TCP port.

#### `OPC proxy returned non-JSON data...`

**Cause:** The request reached a non-HTTP port or an unconfigured web server that returned plain text or raw socket bytes.  
**Solution:** Ensure the HTTP proxy server is running and accessible at the specified `endpointUrl`.

#### `API key is required`

**Cause:** No `apiKey` was provided in the node parameters or incoming payload.  
**Solution:** Set the `apiKey` parameter or bind it to an environment secret (`{{ $secrets.OPC_PROXY_KEY }}`).

#### `Node ID is required for read/write operation`

**Cause:** The `nodeId` field is missing for a `read` or `write` action.  
**Solution:** Provide a valid OPC Node ID string (e.g. `ns=1;s=PressureGauge`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [OPC Trigger](./opc-trigger.md) – Listen for tag changes and data events from OPC servers
- [Modbus Read](./modbus-read.md) – Read registers from Modbus TCP/RTU devices
- [Profinet Read](./profinet-read.md) – Integrate with Siemens Profinet industrial networks
- [MQTT Publish](./mqtt-publish.md) – Publish telemetry data to industrial IoT brokers
- [HTTP Request](./http-request.md) – Make general REST API calls

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-14 | Initial release supporting OPC read, write, browse, and call operations via HTTP proxy. |

<!-- /SECTION: changelog -->

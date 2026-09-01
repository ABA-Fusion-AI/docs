---
node_id: "coap-server"
title: "CoAP Server"
description: "Listen for incoming CoAP requests and trigger workflows when messages are received."
category: "Triggers & Ingress"
subcategory: "IoT & Industrial Ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - coap
  - server
  - iot
  - udp
  - networking
  - sensors
  - industrial-iot
related_nodes:
  - coap-client
  - mqtt-trigger
  - log
---

<!-- SECTION: header -->
# CoAP Server

> **Category:** Triggers & Ingress | **Subcategory:** IoT & Industrial Ingress | **Type:** Trigger Node

Listen for incoming CoAP requests and start a workflow when a message is received from an IoT device or another CoAP client.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **CoAP Server** node creates a server that listens for incoming messages using the Constrained Application Protocol over UDP. Each received request can trigger the connected workflow and pass the request data to downstream nodes.

CoAP is designed for constrained devices, sensors, embedded systems, and machine-to-machine communication where lightweight messaging is important.

### Key Features

- **Inbound Requests:** Receive messages from CoAP clients
- **Workflow Triggering:** Start a workflow when a request arrives
- **UDP Communication:** Listen using the CoAP transport used by constrained IoT devices
- **IoT Ready:** Connect sensors, devices, gateways, and automation workflows
- **Request Data:** Make incoming request information available to downstream nodes
- **Error Routing:** Route server and request-processing failures to the error output
- **No API Credentials:** The example contains no API key, token, password, or secret

### Use Cases

- Receive sensor readings from IoT devices
- Trigger workflows from embedded systems
- Build lightweight device-to-workflow integrations
- Receive commands or status messages from local gateways
- Connect CoAP devices to logging, alerting, or data-processing workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

The current node example exposes no configurable parameters:

```json
{}
```

The server is configured by the node runtime and begins listening when the workflow is active. Consult the runtime configuration for the assigned host, port, and resource path when deploying the workflow.

### Protocol

CoAP uses UDP and commonly uses port `5683` for non-secure CoAP traffic. The example workflow connects local CoAP clients to:

```text
coap://127.0.0.1:5683/sensor
```

The exact listening address and resource routing are determined by the deployed node runtime.

### API and Authentication

This node does not use an API key or bearer token. The example workflow has empty `variables` and `secrets` objects, and contains no authentication credentials.

If the deployment requires network-level protection, secure the host, firewall, gateway, or transport configuration outside the workflow node. Do not place passwords or private keys in the workflow example.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| None | — | This is a trigger node and does not require an incoming workflow input. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Incoming CoAP request data delivered to the workflow |

The request object may contain protocol metadata, request method, resource path, payload, and request options depending on the runtime adapter and client request.

### Request Example

```json
{
  "method": "POST",
  "path": "/sensor",
  "payload": {
    "temperature": 25
  }
}
```

### Error Output

| Output | Type | Description |
|--------|------|-------------|
| `error` | `object` | Server startup, network, malformed-request, or request-processing error |

```json
{
  "success": false,
  "error": "CoAP server request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Receive a Sensor Message

Connect a CoAP client to the server's sensor resource. The client can send a payload such as:

```json
{
  "temperature": 25
}
```

Example client URL:

```text
coap://127.0.0.1:5683/sensor
```

### Workflow Pattern

```text
CoAP Device → CoAP Server → Log
```

Use a downstream function, database, notification, or filter node to process the incoming request.

### Local Test Workflow

The included example workflow demonstrates a CoAP Server receiving requests from CoAP Client nodes using GET, POST, PUT, and DELETE methods. It also connects the results to Log nodes for inspection.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Receive CoAP messages and inspect them
```

### Common Patterns

- **Sensor Ingress:** CoAP Device → CoAP Server → Function → Database
- **Device Alerts:** CoAP Device → CoAP Server → Filter → Notification
- **Request Logging:** CoAP Server → Log
- **Gateway Integration:** IoT Gateway → CoAP Server → HTTP Request or Message Queue
- **Bidirectional Testing:** CoAP Server ↔ CoAP Client

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Server does not start

**Cause:** The configured listener address or port is unavailable, or another process is already using it.

**Solution:** Check the runtime listener configuration, confirm that the port is available, and verify that the workflow is active.

#### Client cannot connect

**Cause:** The client URL, host, port, firewall, or network route is incorrect.

**Solution:** Verify the CoAP URL, confirm that the server is running, and allow UDP traffic on the listener port.

#### No workflow execution is triggered

**Cause:** The request is sent to the wrong resource path or the client is connecting to a different host or port.

**Solution:** Compare the client URL with the deployed server listener and resource routing configuration.

#### Malformed request

**Cause:** The incoming packet is not a valid CoAP request or contains an unsupported option or payload.

**Solution:** Check the client method, URL, payload encoding, and CoAP implementation compatibility.

#### Authentication error

**Cause:** This node does not require an API key or token. The issue is likely related to an external gateway, firewall, or deployment security layer.

**Solution:** Check external network and security configuration. Do not add credentials to the example workflow unless the runtime explicitly supports them.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `Address in use` | Listener port is already occupied | Choose an available port or stop the conflicting service |
| `Network error` | Listener or network failure | Check the host, port, firewall, and UDP connectivity |
| `Invalid request` | Malformed CoAP message | Verify the sending client and request format |
| `Route not found` | Unknown resource path | Use the configured CoAP resource path |
| `Workflow error` | Downstream processing failed | Inspect connected nodes and their error outputs |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [CoAP Client](../coap-client/en.md) - Send GET, POST, PUT, and DELETE requests to CoAP servers
- **MQTT Trigger** - Receive MQTT messages in workflows
- **Log** - Inspect incoming request data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial documentation |

<!-- /SECTION: changelog -->

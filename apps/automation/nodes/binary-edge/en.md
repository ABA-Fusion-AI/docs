---
node_id: "binary-edge"
title: "BinaryEdge IP Lookup"
description: "Query BinaryEdge IP intelligence for exposed services, vulnerable endpoints, and leaked credentials associated with an IP address."
category: "Security & Networking"
subcategory: "Security Intelligence"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - security
  - networking
  - ip
  - threat-intelligence
  - binaryedge
  - exposure
  - credentials
related_nodes:
  - abuse-ipdb
  - grey-noise
  - shodan
---

<!-- SECTION: header -->
# BinaryEdge IP Lookup

> **Category:** Security & Networking | **Type:** Action Node

IP intelligence lookup. Excellent for finding leaked credentials or exposed databases (Redis, MongoDB, etc.) on an IP.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **BinaryEdge IP Lookup** node queries BinaryEdge intelligence to investigate an IP address and reveal exposed services, misconfigurations, and security-relevant exposure data. It is especially useful for detecting internet-facing systems that may be leaking credentials, exposing databases, or exposing administrative services.

### Key Features

- **IP Intelligence:** Query BinaryEdge data for an IP address
- **Exposure Discovery:** Identify internet-facing ports and services linked to the IP
- **Credential Leakage Checks:** Surface findings commonly associated with exposed credentials or vulnerable services
- **Database Exposure Detection:** Find publicly reachable databases and services like Redis, MongoDB, or similar
- **Security Investigation:** Enrich incident response and threat hunting workflows
- **Workflow Integration:** Pass results into decision, filtering, or alerting nodes

### Use Cases

- Investigate suspicious IP addresses
- Identify public services exposed on a host
- Detect misconfigured or exposed databases
- Search for leaked credentials and accessible admin interfaces
- Enrich SOC or hunting workflows with IP exposure context
- Prioritize remediation based on publicly visible security issues

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ip` | `string` | ✅ Yes | — | IP address to investigate |
| `apiKey` | `string` | ✅ Yes | — | BinaryEdge API key used for authentication |
| `query` | `string` | ❌ No | — | Optional query or filter used by the API when supported |

### Example

```text
ip: "185.220.101.5"
apiKey: "{{secrets.binaryedgeApiKey}}"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | IP address or object containing an IP and optional query data |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | BinaryEdge intelligence response for the queried IP |
| `error` | `object` | Error details if validation or the API request fails |

### Success Output Example

```json
{
  "ip": "185.220.101.5",
  "status": "success",
  "services": [
    {
      "port": 6379,
      "service": "redis",
      "exposed": true
    },
    {
      "port": 27017,
      "service": "mongodb",
      "exposed": true
    }
  ],
  "tags": [
    "database",
    "redis",
    "public-service"
  ]
}
```

### Error Output Example

```json
{
  "success": false,
  "error": "Invalid BinaryEdge API key or lookup failed.",
  "ip": "185.220.101.5"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Investigate a Suspicious IP

```text
ip: "185.220.101.5"
apiKey: "{{secrets.binaryedgeApiKey}}"
```

**Result:**

```json
{
  "ip": "185.220.101.5",
  "services": [
    {
      "port": 6379,
      "protocol": "tcp",
      "service": "redis"
    }
  ],
  "risk": "high"
}
```

### Example: Threat Hunting Workflow

Use the node early in a security workflow to determine whether an IP is exposing sensitive services before escalating or blocking access.

<!-- /SECTION: examples -->

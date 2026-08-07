---
node_id: "grey-noise"
title: "GreyNoise IP Lookup"
description: "Check whether an IP address is part of internet noise or a targeted attack"
category: "security"
subcategory: "security-intelligence"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - security
  - networking
  - ip
  - threat-intelligence
related_nodes:
  - ip-address
  - abuse-ipdb
  - shodan
---

<!-- SECTION: header -->
# GreyNoise IP Lookup

> **Category:** Security | **Type:** Action Node

Check whether an IP address is associated with internet scanning noise or a targeted attack using GreyNoise intelligence.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GreyNoise IP Lookup** node queries GreyNoise data to determine if an IP address is likely part of benign internet scanning activity or a more suspicious targeted attack. It is commonly used to reduce noise in security monitoring and triage workflows.

### Key Features

- **IP Reputation Check:** Identify whether an IP is known to be noisy or malicious
- **Threat Triage Support:** Separate benign scanners from potentially hostile activity
- **Simple Input:** Use a single IP address as the main input
- **Workflow Integration:** Pass results to alerting, enrichment, or filtering nodes

### Use Cases

- Filter scanner traffic from security alerts
- Investigate suspicious IP addresses
- Enrich incident data with reputation context
- Support SOC and threat-hunting workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ip` | `string` | ✅ Yes | — | IP address to look up |

### Example

```text
ip: "185.220.101.5"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data that can provide the IP address |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | GreyNoise lookup result for the IP |
| `error` | `object` | Error details if the lookup fails |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Check a Suspicious IP

```text
ip: "185.220.101.5"
```

**Result:**

```json
{
  "ip": "185.220.101.5",
  "classification": "noise",
  "seen": true
}
```

### Example: Use in a Security Workflow

Route the result to a decision node to distinguish benign scanner traffic from suspicious activity.

<!-- /SECTION: examples -->

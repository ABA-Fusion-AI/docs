---
node_id: "virus-total"
title: "VirusTotal Lookup"
description: "Retrieve VirusTotal threat-intelligence reports for IPs, domains, URLs, and file hashes."
category: "Security & Networking"
subcategory: "Security Intelligence"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - virustotal
  - security
  - threat-intelligence
  - malware
  - ip
  - domain
  - url
  - file-hash
related_nodes:
  - abuse-ipdb
  - shodan-lookup
  - grey-noise
  - http-request
---

<!-- SECTION: header -->
# VirusTotal Lookup

> **Category:** Security & Networking | **Subcategory:** Security Intelligence | **Type:** Action Node

Retrieve a VirusTotal security report for an IP address, domain, URL, or file hash. Reports aggregate detections and reputation signals from many security vendors and intelligence sources.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **VirusTotal Lookup** node queries the VirusTotal API for an indicator of compromise (IOC) and returns the corresponding report. The workflow example demonstrates IP lookups for `8.8.8.8` and `1.0.0.1`.

### Key Features

- **IP Intelligence:** Retrieve reputation and analysis data for an IP address
- **Domain Intelligence:** Investigate domain reputation and related detections
- **URL Intelligence:** Check a URL's analysis and reputation information
- **File Hash Lookup:** Query a file report by MD5, SHA-1, or SHA-256 hash
- **Aggregated Detection Data:** Inspect vendor results, statistics, tags, and reputation fields when available
- **Dynamic Input:** Use configured values or provide lookup data through the `input` port
- **Authenticated API Access:** Uses a VirusTotal API key sent through the `x-apikey` header

### Use Cases

- Enrich SIEM, EDR, or email-security alerts with reputation context
- Investigate suspicious IPs, domains, URLs, and file hashes
- Add threat-intelligence checks to incident-response workflows
- Prioritize indicators for analyst review
- Correlate VirusTotal findings with Shodan, AbuseIPDB, or GreyNoise results

> VirusTotal detections are intelligence signals, not definitive proof of maliciousness. Review the full context and avoid taking irreversible action based on a single automated result.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `resourceType` | `enum` | Yes | — | Indicator type to look up, such as `ip`, `domain`, `url`, or `file` |
| `resource` | `string` | Yes | — | IP address, domain, URL, or file hash to investigate |
| `apiKey` | `string` | Yes | — | VirusTotal API key used for authentication. Store it as a workflow secret. |

The workflow examples use:

```json
{
  "resourceType": "ip",
  "resource": "8.8.8.8",
  "apiKey": "{{secrets.virusTotalApiKey}}"
}
```

### Resource Types

| Type | Value format | VirusTotal object |
|------|--------------|-------------------|
| `ip` | IPv4 or IPv6 address | IP address report |
| `domain` | Domain name | Domain report |
| `url` | Complete URL | URL report or URL identifier |
| `file` | MD5, SHA-1, or SHA-256 | File report |

### API Behavior

VirusTotal API requests use the `x-apikey` request header:

```text
x-apikey: <apiKey>
```

The API uses resource-specific endpoints under:

```text
https://www.virustotal.com/api/v3/
```

For example, an IP report uses `/ip_addresses/{ip}`. The exact endpoint depends on `resourceType`.

### Credential Security

The provided workflow contains a literal `apiKey` value in both sample nodes. Treat that key as exposed: revoke or rotate it in VirusTotal and replace the workflow value with a secret reference. Never commit or export live API keys in workflow examples.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | An indicator string, or an object containing `resourceType`, `resource`, and optionally `apiKey`. Configured values should be preferred for secrets. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | VirusTotal API report containing `data`, object attributes, type, ID, and links when returned by the API |

Typical response shape:

```json
{
  "data": {
    "type": "ip_address",
    "id": "8.8.8.8",
    "attributes": {
      "as_owner": "Example Network",
      "country": "US",
      "reputation": 0,
      "last_analysis_stats": {
        "harmless": 70,
        "malicious": 0,
        "suspicious": 0,
        "undetected": 5,
        "timeout": 0
      },
      "tags": []
    }
  }
}
```

Attributes vary by resource type and account permissions. Treat vendor counts, reputation, tags, and related objects as optional.

### Error Output

Invalid resources, missing credentials, authentication failures, rate limits, unavailable reports, and network errors are routed to `error`.

```json
{
  "success": false,
  "error": "VirusTotal lookup failed",
  "resourceType": "ip",
  "resource": "203.0.113.10"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### IP Lookup

```json
{
  "resourceType": "ip",
  "resource": "8.8.8.8",
  "apiKey": "{{secrets.virusTotalApiKey}}"
}
```

### Domain Lookup

```json
{
  "resourceType": "domain",
  "resource": "example.com",
  "apiKey": "{{secrets.virusTotalApiKey}}"
}
```

### File Hash Lookup

```json
{
  "resourceType": "file",
  "resource": "275bdde4e8f7b5c9e6c0b8d8d4b0c7c4f9e1c6d8a2b3c4d5e6f7a8b9c0d1e2f3",
  "apiKey": "{{secrets.virusTotalApiKey}}"
}
```

### Dynamic Indicator

Pass an object through `input`:

```json
{
  "resourceType": "ip",
  "resource": "1.0.0.1"
}
```

Keep the API key in node configuration or the workflow secret system rather than passing it through ordinary data fields.

### Branch on Detection Statistics

After a successful lookup, use a Function or conditional node to inspect fields such as `data.attributes.last_analysis_stats.malicious` and `suspicious`. Do not treat a zero count as a guarantee that an indicator is safe.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve VirusTotal reports for IP indicators
```

### Common Patterns

- **Basic lookup:** Manual Trigger → VirusTotal Lookup → Log
- **Alert enrichment:** SIEM/Webhook → VirusTotal Lookup → Incident record
- **Reputation correlation:** VirusTotal Lookup → Shodan or AbuseIPDB → Function
- **Automated triage:** IOC list → VirusTotal Lookup → Conditional branch → Notification

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### API key is required

**Cause:** `apiKey` is missing, invalid, revoked, or not sent in the expected authentication header.

**Solution:** Create or use a valid VirusTotal API key, store it as a secret, and ensure the node can send it as `x-apikey`.

### Invalid resource

**Cause:** The value does not match the selected `resourceType`, or the resource identifier is malformed.

**Solution:** Use an IP address for `ip`, a domain for `domain`, a complete URL for `url`, or a valid MD5/SHA-1/SHA-256 hash for `file`.

### Resource not found

**Cause:** VirusTotal has no report for the indicator or the requested object is not available to the account.

**Solution:** Verify the indicator and resource type. A missing report does not prove the indicator is safe.

### Rate limit exceeded

**Cause:** The API account or plan has exceeded its request quota.

**Solution:** Reduce request volume, add retry backoff, and check the account quota before rerunning large batches.

### Incomplete analysis data

**Cause:** Vendor results and optional attributes differ by resource type, scan age, and account permissions.

**Solution:** Use defensive field mapping and check that nested attributes exist before reading them.

### Exposed API key in workflow

**Cause:** A live-looking key was stored directly in the example workflow parameters.

**Solution:** Revoke or rotate the key immediately and replace it with a secret reference such as `{{secrets.virusTotalApiKey}}`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [AbuseIPDB](./abuse-ipdb.md) — Check IP abuse reports and reputation
- [Shodan IP Lookup](./shodan-lookup.md) — Inspect exposed services and host intelligence
- [GreyNoise](./grey-noise.md) — Identify scanners and benign internet noise
- [HTTP Request](./http-request.md) — Call other security-intelligence APIs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation |

<!-- /SECTION: changelog -->

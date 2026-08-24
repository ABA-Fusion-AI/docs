---
node_id: "shodan-lookup"
title: "Shodan IP Lookup"
description: "Perform IP intelligence lookups using Shodan InternetDB or the Shodan API."
category: "Security & Networking"
subcategory: "Security Intelligence"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - security
  - networking
  - ip
  - shodan
  - threat-intelligence
  - exposure
  - vulnerabilities
related_nodes:
  - binary-edge
  - abuse-ipdb
  - grey-noise
  - ipinfo
---

<!-- SECTION: header -->
# Shodan IP Lookup

> **Category:** Security & Networking | **Subcategory:** Security Intelligence | **Type:** Action Node

Perform IP intelligence lookups using Shodan InternetDB or the authenticated Shodan API. Use this node only to investigate assets you own or are authorized to assess.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Shodan IP Lookup** node retrieves security-relevant information about an IPv4 or IPv6 address. It supports the free Shodan InternetDB service and the full Shodan host-information API when an API key is supplied.

### Lookup Modes

- **InternetDB:** Free lookup with no API key. Returns publicly available summary data such as open ports, hostnames, CPEs, and known vulnerability identifiers when available.
- **Shodan API:** Authenticated host lookup with broader host and service information. The API key is supplied through the node configuration.

### Key Features

- **IP Intelligence:** Enrich an IP address with Shodan data
- **Exposure Discovery:** Identify observed open ports and services
- **Host Context:** Retrieve hostnames, domains, organization, ASN, and location data when available
- **Vulnerability Context:** Surface vulnerability identifiers returned by the selected Shodan source
- **Flexible Input:** Use a configured IP address or provide one dynamically through `input`
- **Workflow Integration:** Route results to logging, filtering, alerting, or incident-response steps

### Use Cases

- Investigate suspicious or observed source IPs
- Monitor the internet-facing attack surface of authorized assets
- Enrich security alerts with open-port and service context
- Prioritize remediation for exposed services or reported vulnerabilities
- Support SOC, threat-hunting, and asset-inventory workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ip` | `string` | Yes | — | IPv4 or IPv6 address to investigate, for example `1.1.1.1`. |
| `apiKey` | `string` | Conditional | — | Shodan API key. Required when `useInternetDb` is `false`; store it as a secret. |
| `useInternetDb` | `boolean` | No | `true` | When `true`, use free InternetDB. When `false`, use the authenticated Shodan host API. |

### InternetDB Mode

InternetDB does not require authentication and is intended for lightweight IP intelligence lookups. The service endpoint is:

```text
GET https://internetdb.shodan.io/{ip}
```

### Shodan API Mode

When `useInternetDb` is `false`, the node uses the Shodan host-information API and requires `apiKey`:

```text
GET https://api.shodan.io/shodan/host/{ip}?key={apiKey}
```

Do not hard-code API keys in workflow exports or source files. Use the workflow secret mechanism.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | An IP address, or an object containing an `ip` field and optional lookup settings. Used when `ip` is not configured. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | IP intelligence response from InternetDB or the Shodan API, depending on `useInternetDb` |

Typical InternetDB response:

```json
{
  "ip": "1.1.1.1",
  "ports": [53, 80, 443],
  "hostnames": ["one.one.one.one"],
  "cpes": [],
  "vulns": [],
  "tags": []
}
```

Typical Shodan API response fields may include `ip_str`, `hostnames`, `domains`, `org`, `asn`, `country_name`, `latitude`, `longitude`, `ports`, and `data` service banners.

### Error Output

Validation, authentication, rate-limit, network, or upstream API errors are routed to `error`:

```json
{
  "success": false,
  "error": "Shodan lookup failed",
  "ip": "203.0.113.10"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Free InternetDB Lookup

```json
{
  "ip": "1.1.1.1",
  "useInternetDb": true
}
```

No API key is required.

### Authenticated Shodan Lookup

```json
{
  "ip": "8.8.8.8",
  "apiKey": "{{secrets.shodanApiKey}}",
  "useInternetDb": false
}
```

Use a secret reference supported by the workflow environment instead of placing a real key in the configuration.

### Dynamic IP from a Previous Node

Pass an IP address directly to the `input` port, or pass an object such as:

```json
{
  "ip": "192.0.2.10"
}
```

The node uses the configured lookup mode unless the incoming object provides supported overrides.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Look up IP intelligence with InternetDB or Shodan API
```

### Common Patterns

- **Basic lookup:** Manual Trigger → Shodan IP Lookup → Log
- **Alert enrichment:** Webhook → Shodan IP Lookup → Conditional branch
- **Asset monitoring:** Asset list → Shodan IP Lookup → Filter → Notification
- **Authorized investigation:** Security alert → Shodan IP Lookup → Incident record

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### IP address is required

**Cause:** No `ip` parameter or usable incoming input was provided.

**Solution:** Set `ip`, pass a string to `input`, or pass an object with an `ip` field.

### API key is required

**Cause:** `useInternetDb` is `false`, but `apiKey` is missing or invalid.

**Solution:** Add a valid Shodan API key through the workflow secret system, or set `useInternetDb` to `true` for the free InternetDB mode.

### No results or incomplete host data

**Cause:** Shodan has not observed services for the IP, or the selected mode returns only summary data.

**Solution:** Check the selected lookup mode and treat optional fields such as `hostnames`, `ports`, `vulns`, and `data` as potentially empty or absent.

### Rate limit or service error

**Cause:** The upstream service rejected the request, is temporarily unavailable, or the account quota was reached.

**Solution:** Inspect the error output, retry with appropriate backoff, and verify the Shodan account quota when using API mode.

### Unauthorized target

**Cause:** The IP is not within your authorized assessment scope.

**Solution:** Stop the lookup and confirm written authorization before continuing security investigation activity.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [BinaryEdge IP Lookup](./binary-edge.md) — Query another IP intelligence provider
- [AbuseIPDB](./abuse-ipdb.md) — Check abuse reports and reputation for an IP
- [GreyNoise](./grey-noise.md) — Add internet-scanner and noise context
- [IP Geolocation](./ip-geolocation.md) — Retrieve IP geolocation and network context

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->

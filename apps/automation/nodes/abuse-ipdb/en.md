---
node_id: "abuse-ipdb"
title: "AbuseIPDB"
description: "Check whether an IPv4 address has been reported for malicious activity using the AbuseIPDB API."
category: "security-nodes"
version: "1.0.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - abuseipdb
  - ip
  - reputation
  - security
  - threat-intelligence
  - lookup
  - cybersecurity
  - peer-only
  - action
related_nodes:
  - http-request
  - function
  - if
---

<!-- SECTION: overview -->
# AbuseIPDB Lookup

> **Category:** security-nodes | **Type:** Action Node

Check whether an IPv4 address has been reported for malicious activity using the AbuseIPDB API.

The **AbuseIPDB Lookup** node queries the AbuseIPDB community database to determine whether an IP address has been associated with spam, hacking attempts, DDoS attacks, malware, brute-force attacks, or other abusive behavior.

The node returns reputation information including abuse confidence, report statistics, ISP information, geographic details, and optional report history.

### Use Cases

- Validate suspicious login IP addresses
- Block malicious traffic in workflows
- Check IP reputation before allowing access
- Analyze firewall or web server logs
- Enrich SIEM or SOC workflows
- Investigate security incidents

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ip` | `string` | ❌ No | Input value | IPv4 address to check. If omitted, the node attempts to read it from the workflow input (`ip` or `ipv4`). |
| `apiKey` | `string` | ✅ Yes | — | AbuseIPDB API key. |
| `maxAgeInDays` | `number` | ❌ No | `90` | Maximum report age to include. |
| `verbose` | `boolean` | ❌ No | `false` | Include individual abuse reports in the response. |

### Input Resolution

If the **ip** parameter is not configured, the node attempts to resolve it from incoming workflow data in the following order:

1. String input
2. `input.ip`
3. `input.ipv4`
4. First element of `input.ipv4` (if it is an array)

If no IP address can be found, the node throws an error.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | IPv4 address or object containing an `ip` or `ipv4` field. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Reputation information returned from AbuseIPDB. |
| `error` | `Error` | Returned if validation or the API request fails. |

### Output Fields

| Field | Type | Description |
|------|------|-------------|
| `ip` | string | Queried IP address |
| `isPublic` | boolean | Whether the IP is publicly routable |
| `ipVersion` | number | IP version |
| `isWhitelisted` | boolean | Whether AbuseIPDB has whitelisted the IP |
| `abuseConfidencePercentage` | number | Abuse confidence score (0–100) |
| `isMalicious` | boolean | True when abuse confidence is greater than 0 |
| `countryCode` | string | Country code |
| `countryName` | string | Country name |
| `usageType` | string | Network usage classification |
| `isp` | string | Internet Service Provider |
| `domain` | string | ISP or organization domain |
| `hostnames` | array | Associated hostnames |
| `isTor` | boolean | Whether the IP belongs to the Tor network |
| `totalReports` | number | Total abuse reports |
| `numDistinctUsers` | number | Number of unique reporters |
| `lastReportedAt` | string | Timestamp of latest report |
| `reports` | array | Individual reports (only when verbose mode is enabled) |
| `raw` | object | Raw AbuseIPDB response |

### Example: Basic Lookup

**Configuration**

```json
{
  "ip": "8.8.8.8",
  "apiKey": "{{secrets.abuseipdbApiKey}}"
}
```

**Output**

```json
{
  "ip": "8.8.8.8",
  "isPublic": true,
  "abuseConfidencePercentage": 0,
  "isMalicious": false,
  "countryCode": "US",
  "isp": "Google LLC",
  "totalReports": 0
}
```

---

### Example: Verbose Lookup

**Configuration**

```json
{
  "ip": "1.2.3.4",
  "apiKey": "{{secrets.abuseipdbApiKey}}",
  "verbose": true,
  "maxAgeInDays": 365
}
```

**Output**

```json
{
  "ip": "1.2.3.4",
  "abuseConfidencePercentage": 100,
  "isMalicious": true,
  "totalReports": 52,
  "reports": [
    {
      "reportedAt": "2026-06-12T14:20:00Z",
      "comment": "SSH brute-force attack",
      "categories": [
        18,
        22
      ]
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Block Malicious Login Attempts

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "abuse-check",
      "type": "abuseipdb-lookup",
      "config": {
        "ip": "{{input.ip}}",
        "apiKey": "{{secrets.abuseipdbApiKey}}"
      }
    },
    {
      "id": "check-score",
      "type": "if",
      "config": {
        "condition": "{{input.isMalicious}}"
      }
    }
  ]
}
```

### Common Patterns

- Login → Check IP Reputation → Allow or Block
- Firewall Logs → AbuseIPDB Lookup → Alert
- Webhook → Reputation Check → Continue Workflow
- VPN Detection → AbuseIPDB → Security Decision
- SIEM → Threat Enrichment → Incident Response

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### IP address is required

**Cause**

No IP address was configured or found in the workflow input.

**Solution**

Provide the `ip` parameter or ensure the incoming data contains an `ip` or `ipv4` field.

---

### Invalid IPv4 address format

**Cause**

The supplied IP is not a valid IPv4 address.

**Solution**

Provide a valid IPv4 address such as:

```
8.8.8.8
```

---

### AbuseIPDB API error

**Cause**

The AbuseIPDB API rejected the request.

**Possible reasons**

- Invalid API key
- Rate limit exceeded
- Invalid parameters
- Subscription limitations

**Solution**

Verify the API key and review the API response.

---

### HTTP Error

**Cause**

The AbuseIPDB service could not be reached.

**Solution**

- Check your internet connection.
- Verify that the AbuseIPDB API is available.
- Retry the request.

---

### No Abuse Reports Found

**Cause**

The returned abuse confidence score is `0`.

**Solution**

This is expected and indicates that AbuseIPDB has no confirmed abuse reports for the IP within the selected time range.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Call external threat intelligence APIs
- [Function](./function.md) – Process reputation scores
- [If](./if.md) – Route workflows based on abuse confidence

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-05 | Initial release |

<!-- /SECTION: changelog -->
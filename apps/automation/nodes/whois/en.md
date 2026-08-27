---
node_id: "whois"
title: "WHOIS Lookup"
description: "Performs WHOIS lookup for domain information including registrar, creation date, expiration date, and nameservers."
category: "Network Tools"
subcategory: "domain-dns"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - whois
  - domain
  - dns
  - registrar
  - nameservers
  - network
  - security
  - domain-expiration
related_nodes:
  - dns-lookup
  - dns-to-ip
  - ssl-checker
  - ip-info
  - function
---

<!-- SECTION: header -->
# WHOIS Lookup

> **Category:** Network Tools | **Type:** Action Node

Perform WHOIS queries on domain names to retrieve registration details, registrar information, creation and expiration dates, domain statuses, and active nameservers.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **WHOIS Lookup** node queries global WHOIS databases across all top-level domains (gTLDs and ccTLDs such as `.com`, `.org`, `.net`, `.io`, `.ma`, `.fr`, etc.). It retrieves critical domain ownership metadata and normalizes the diverse registry formats into a clean, structured JSON response.

The node includes built-in URL parsing and sanitization, allowing you to pass raw URLs (e.g. `https://allotaxi.ma/` or `http://charika.ma/`), plain hostnames, or dynamic payloads from upstream nodes.

### Key Features

- **Global Domain Support:** Query gTLDs (`.com`, `.net`, `.org`, `.ai`, `.io`) and national ccTLDs (`.ma`, `.fr`, `.uk`, `.de`, etc.).
- **Automatic URL Sanitization:** Automatically strips protocols (`http://`, `https://`), paths, and ports before performing the lookup.
- **Normalized Response:** Cleans and unifies inconsistent registry responses into standardized fields (`registrar`, `creationDate`, `expirationDate`, `updatedDate`, `nameservers`, `status`).
- **Complete Raw Output Access:** Exposes the full raw WHOIS response in the `raw` property for advanced parsing and auditing.
- **Flexible Dynamic Inputs:** Accepts domain names from node configuration, upstream plain strings, or objects containing `domain` or `url` fields.
- **Fast Local Resolution:** Uses a resilient 5000ms lookup timeout to prevent blocking workflows.

### Common Use Cases

- **Domain Expiration Monitoring:** Trigger automated alerts to Slack, Discord, or email when mission-critical domains are near expiration.
- **Cybersecurity & Threat Intelligence:** Investigate newly registered domains (NRDs) associated with phishing campaigns, suspicious emails, or security incidents.
- **Competitor & Vendor Analysis:** Identify hosting nameservers, registrars, and domain history for market intelligence.
- **Automated Lead Enrichment:** Extract domain age and registry details from incoming customer lead websites.
- **Asset Inventory & DNS Auditing:** Verify active nameservers and domain statuses across corporate domain portfolios.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## How to Use the WHOIS Lookup Node

The WHOIS Lookup node can be configured manually in the visual canvas or dynamically driven by data from upstream trigger and action nodes.

![WHOIS Lookup Configuration](icon.svg)

### Step-by-Step Setup in the Visual Builder

1. **Add the Node:** Drag the **WHOIS Lookup** node onto the canvas from the **Network Tools** palette.
2. **Connect an Upstream Node:** Connect a trigger (such as `Manual Trigger`, `Webhook`, or `Schedule`) or a data node (like `Function`, `HTTP Request`, or `Form`) to the `input` port.
3. **Configure the Domain (Optional):**
   - Enter a domain name or URL in the **Domain** parameter (e.g. `google.com`, `https://allotaxi.ma/`).
   - If you leave the **Domain** field empty, the node will automatically extract the target domain from the incoming payload.
4. **Connect Downstream Nodes:** Connect the `success` output port to nodes such as `Log`, `Filter Array`, `If`, or notification channels (`Slack`, `Email Send`).

---

### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `domain` | `string` | ❌ No* | — | Target domain name or URL (e.g., `example.com`, `https://charika.ma/`). *Required if not supplied via input. |

---

### Domain Sanitization & Protocol Stripping

You can provide domains in various formats. The node automatically normalizes the input before querying the WHOIS server:

| Input Format | Sanitized Query Domain |
|--------------|------------------------|
| `example.com` | `example.com` |
| `https://allotaxi.ma/` | `allotaxi.ma` |
| `http://charika.ma/companies/search` | `charika.ma` |
| `https://sub.domain.co.uk:8080/path` | `sub.domain.co.uk` |

---

### Dynamic Input Resolution

If the `domain` parameter is left empty in the node configuration, the node resolves the target domain from incoming workflow data using the following priority:

```
                  ┌────────────────────────────────────────┐
                  │          Incoming Input Data           │
                  └──────────────────┬─────────────────────┘
                                     │
                 Is `domain` configured in node parameters?
                                     │
                     ┌───────────────┴───────────────┐
                    YES                              NO
                     │                               │
             Use config `domain`         Check incoming `data`
                                                     │
                             ┌───────────────────────┼───────────────────────┐
                             ▼                       ▼                       ▼
                     String Input             Object with `domain`    Object with `url`
                  e.g. "github.com"          e.g. { domain: "..." }   e.g. { url: "..." }
```

1. **Node Configuration:** `domain` set in parameter inputs.
2. **Plain String:** When the previous node emits a string (e.g. `"openai.com"` or `"https://allotaxi.ma/"`).
3. **Object with `domain` field:** When the previous node emits `{ "domain": "example.org" }`.
4. **Object with `url` field:** When the previous node emits `{ "url": "https://charika.ma/about" }` (automatically extracts the hostname).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional upstream payload containing a domain string or an object with `domain` or `url`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the WHOIS query succeeds. Contains normalized domain information. |
| `error` | `Error` | Emitted when validation fails or WHOIS servers cannot be reached. |

---

### Output Data Schema

```json
{
  "domain": "charika.ma",
  "registrar": "MAROC TELECOM",
  "creationDate": "2015-04-12T10:00:00Z",
  "expirationDate": "2027-04-12T10:00:00Z",
  "updatedDate": "2026-03-01T15:20:00Z",
  "nameservers": [
    "ns1.maroctelecom.ma",
    "ns2.maroctelecom.ma"
  ],
  "status": [
    "active",
    "clientTransferProhibited"
  ],
  "raw": {
    "whois.registre.ma": {
      "Domain Name": "charika.ma",
      "Registrar": "MAROC TELECOM",
      "CreatedDate": "2015-04-12T10:00:00Z",
      "ExpiryDate": "2027-04-12T10:00:00Z",
      "Name Server": [
        "ns1.maroctelecom.ma",
        "ns2.maroctelecom.ma"
      ]
    }
  }
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `domain` | `string` | The cleaned target domain name queried. |
| `registrar` | `string \| null` | The accredited registrar organization managing the domain registration. |
| `creationDate` | `string \| null` | Domain registration / creation timestamp. |
| `expirationDate` | `string \| null` | Domain expiration / renewal deadline timestamp. |
| `updatedDate` | `string \| null` | Timestamp of the most recent domain record modification. |
| `nameservers` | `string[]` | List of authoritative DNS nameservers configured for the domain. |
| `status` | `string[]` | Domain status codes (e.g. `clientTransferProhibited`, `active`, `ok`). |
| `raw` | `object` | Complete raw WHOIS key-value records returned by the authoritative registry servers. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Practical Examples

### Example 1: Direct Domain Lookup with Full URL

Query a domain provided as a full web address with `https://`.

**Node Configuration:**
- **Domain:** `https://allotaxi.ma/`

**Output:**
```json
{
  "domain": "allotaxi.ma",
  "registrar": "GENIOUS COMMUNICATIONS",
  "creationDate": "2018-09-14T11:24:00Z",
  "expirationDate": "2027-09-14T11:24:00Z",
  "updatedDate": "2026-08-10T08:15:00Z",
  "nameservers": [
    "ns1.genious.net",
    "ns2.genious.net"
  ],
  "status": [
    "active"
  ],
  "raw": { ... }
}
```

---

### Example 2: Standard Top-Level Domain Lookup (`.com`)

Query `github.com`.

**Node Configuration:**
- **Domain:** `github.com`

**Output:**
```json
{
  "domain": "github.com",
  "registrar": "MarkMonitor Inc.",
  "creationDate": "2007-10-09T18:20:50Z",
  "expirationDate": "2028-10-09T18:20:50Z",
  "updatedDate": "2026-09-08T09:18:22Z",
  "nameservers": [
    "dns1.p08.nsone.net",
    "dns2.p08.nsone.net",
    "ns-1283.awsdns-32.org",
    "ns-1707.awsdns-21.co.uk"
  ],
  "status": [
    "clientDeleteProhibited",
    "clientTransferProhibited",
    "clientUpdateProhibited"
  ],
  "raw": { ... }
}
```

---

### Example 3: Dynamic Domain Lookup from String Input

An upstream `Function` node generates a domain name dynamically.

**Upstream Function Node Code:**
```javascript
return "charika.ma";
```

**WHOIS Node Configuration:**
- **Domain:** *(leave empty)*

The node automatically takes `"charika.ma"` from the input payload and executes the lookup.

---

### Example 4: Dynamic URL Object Extraction

An upstream webhook or form provides a website URL in an object payload.

**Upstream Node Payload:**
```json
{
  "companyName": "Charika",
  "url": "http://charika.ma/companies"
}
```

**WHOIS Node Configuration:**
- **Domain:** *(leave empty)*

The node automatically extracts `charika.ma` from the `url` property and returns its WHOIS registration data.

---

### Example 5: Domain Expiration Monitoring Workflow

Calculate days remaining until domain renewal and send an alert if expiration is within 30 days.

**Upstream WHOIS Output:**
```json
{
  "domain": "example.ma",
  "expirationDate": "2026-09-15T00:00:00Z"
}
```

**Downstream Function Node:**
```javascript
const exp = new Date(input.expirationDate);
const now = new Date();
const daysRemaining = Math.floor((exp - now) / (1000 * 60 * 60 * 24));

return {
  domain: input.domain,
  daysRemaining: daysRemaining,
  needsRenewal: daysRemaining <= 30
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Interactive Workflow Preview

```fusion-workflow
src: example.workflow.json
title: WHOIS Domain Lookup Examples
```

---

### Sample Workflows

#### 1. Manual Domain Inspection Pipeline: Trigger ➔ WHOIS Lookup ➔ Log

A simple workflow to manually trigger WHOIS lookups for corporate domains and view normalized records:

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger",
      "label": "Run WHOIS Query"
    },
    {
      "id": "whois-node",
      "type": "whois",
      "config": {
        "domain": "https://allotaxi.ma/"
      }
    },
    {
      "id": "log-output",
      "type": "log",
      "label": "Display WHOIS Data"
    }
  ],
  "connections": [
    {
      "source": "trigger",
      "target": "whois-node"
    },
    {
      "source": "whois-node",
      "target": "log-output"
    }
  ]
}
```

---

#### 2. Domain Expiration Alert Pipeline

A scheduled workflow that checks domain expiration weekly and alerts Slack if renewal is required:

```
  ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
  │ Schedule Trigger │ ────▶ │   WHOIS Lookup   │ ────▶ │  Function Filter │ ────▶ │    Slack Send    │
  │  (Every Monday)  │       │  (corporate.com) │       │ (Days left <= 30)│       │ (Renew Domain!)  │
  └──────────────────┘       └──────────────────┘       └──────────────────┘       └──────────────────┘
```

---

### Architecture Patterns

- **Security & Phishing Analysis:** `Email Trigger ➔ Function (Extract sender domain) ➔ WHOIS Lookup ➔ If (Domain age < 14 days) ➔ Quarantine Alert`.
- **Infrastructure Auditing:** `Database (List corporate domains) ➔ WHOIS Lookup ➔ Function (Compare nameservers) ➔ Audit Report`.
- **Enrichment Pipeline:** `CRM Webhook (New Lead) ➔ WHOIS Lookup ➔ MySQL / Notion (Save domain creation date & registrar)`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues & Resolutions

#### `WHOIS lookup failed: Domain is required for WHOIS lookup`
- **Cause:** The `domain` parameter was not configured, and the incoming payload did not contain a valid string, `domain`, or `url` field.
- **Solution:** Provide a valid domain in the node configuration or ensure upstream nodes emit `{ "domain": "example.com" }`, `{ "url": "..." }`, or a plain string.

---

#### `WHOIS lookup failed: Timeout`
- **Cause:** The authoritative registry WHOIS server took longer than 5000ms to respond or blocked the connection.
- **Solution:** Some ccTLD registries have aggressive rate limits. Add a **Delay** node or retry the request after a short interval.

---

#### `Dates or Registrar returned as null`
- **Cause:** The specific TLD registry does not disclose registrar or expiration details publicly (e.g. GDPR redaction or restricted ccTLD privacy policies).
- **Solution:** Inspect the `raw` output object to see all unparsed fields returned by the registry server.

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Domain is required for WHOIS lookup` | Missing domain input | Specify `domain` in config or pass via incoming data. |
| `WHOIS lookup failed: <reason>` | Connection failure / TLD error | Verify domain syntax, ensure network connectivity, or check TLD support. |
| `WHOIS lookup failed: Timeout` | Registry server latency | Retry query or add a Delay node in batch loops. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [DNS Lookup](../dns-lookup/en.md) — Resolve A, AAAA, MX, TXT, and CNAME DNS records
- [DNS to IP](../dns-to-ip/en.md) — Quick IP address resolution for domain names
- [SSL Checker](../ssl-checker/en.md) — Inspect SSL/TLS certificate validity and expiration
- [IP Info](../ip-info/en.md) — Fetch geolocation and ASN details for IP addresses
- [Function](../function/en.md) — Parse dates and compute expiration thresholds

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial release of WHOIS Lookup Action Node with automatic URL sanitization and normalized output schema |

<!-- /SECTION: changelog -->

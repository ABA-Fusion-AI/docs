---
node_id: "dns-lookup"
title: "DNS Lookup"
description: "Resolve hostnames and query DNS records."
category: "security-networking"
subcategory: "network-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - dns
  - networking
  - hostname
  - domain
  - records
related_nodes:
  - dns-to-ip
  - ipinfo
  - whois
---

<!-- SECTION: header -->
# DNS Lookup

> **Category:** Security & Networking | **Type:** Action Node

Resolve hostnames and query DNS records using Node.js DNS resolution. The node supports common DNS record types and can receive a hostname from its configuration or from incoming workflow data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **DNS Lookup** node performs DNS queries for hostnames, domains, URLs, and IP addresses. It uses the native Node.js DNS module to retrieve different record types without requiring an external API or authentication.

The node can resolve IPv4 and IPv6 addresses, mail servers, name servers, aliases, text records, service records, authority information, and reverse DNS records.

### Key Features

- **Multiple DNS Record Types:** Query A, AAAA, CNAME, MX, NS, TXT, SOA, SRV, and PTR records.
- **Dynamic Hostname Input:** Accept a hostname from configuration or incoming workflow data.
- **URL Parsing:** Extract the hostname automatically from a URL.
- **Protocol Removal:** Remove `http://` or `https://` before performing the lookup.
- **No API Key Required:** Uses the built-in Node.js DNS resolver.
- **Structured Output:** Returns the hostname, selected record type, and resolved records.
- **Error Handling:** Wraps DNS resolution errors with a clear error message.

### Supported DNS Record Types

| Record Type | Description |
|-------------|-------------|
| `A` | IPv4 address records |
| `AAAA` | IPv6 address records |
| `CNAME` | Canonical name aliases |
| `MX` | Mail exchange servers |
| `NS` | Authoritative name servers |
| `TXT` | Text records |
| `SOA` | Start of authority record |
| `SRV` | Service location records |
| `PTR` | Reverse DNS pointer records |

### Use Cases

- Resolve domain names to IPv4 or IPv6 addresses
- Retrieve mail server information
- Inspect authoritative name servers
- Query domain verification TXT records
- Retrieve service discovery records
- Perform reverse DNS lookups
- Validate DNS configuration
- Integrate DNS checks into monitoring workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `hostname` | `string` | ⚠️ Conditional | — | Hostname, domain, URL, or IP address to query. May also come from incoming workflow data. |
| `recordType` | `enum` | ❌ No | `A` | DNS record type to resolve. |

> **Note:** A hostname is required during execution, but it may be supplied either through the node configuration or incoming workflow data.

### Record Type Values

| Value | Resolver Used |
|-------|---------------|
| `A` | IPv4 resolution |
| `AAAA` | IPv6 resolution |
| `CNAME` | Canonical name resolution |
| `MX` | Mail exchange resolution |
| `NS` | Name server resolution |
| `TXT` | Text record resolution |
| `SOA` | Start of authority resolution |
| `SRV` | Service record resolution |
| `PTR` | Reverse DNS pointer resolution |

### Default Values

| Parameter | Default |
|-----------|---------|
| `recordType` | `A` |

### Hostname Resolution Priority

The node resolves the target hostname using the following priority:

1. The configured `hostname` parameter.
2. Incoming workflow data when it is a string.
3. The `hostname` property from an incoming object.
4. The `domain` property from an incoming object.
5. The `url` property from an incoming object.

### Input Normalization

If the value contains a protocol or path, the node removes them before performing the DNS query.

For example:

```text
https://example.com/products
```

becomes:

```text
example.com
```

### Configuration Notes

- The `hostname` parameter may contain a domain, hostname, URL, or IP address.
- `A` records return IPv4 addresses.
- `AAAA` records return IPv6 addresses.
- `PTR` lookups are generally used with IP addresses.
- DNS response structures vary depending on the selected record type.
- The node does not require credentials or an API key.

<!-- /SECTION: configuration -->
---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional incoming data containing a hostname, domain, URL, or IP address. |

The node accepts a plain string:

```text
google.com
```

It also accepts an object containing `hostname`:

```json
{
  "hostname": "google.com"
}
```

An object containing `domain`:

```json
{
  "domain": "google.com"
}
```

Or an object containing `url`:

```json
{
  "url": "https://google.com/search"
}
```

When a valid URL is provided, the node extracts its hostname automatically.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `hostname` | `string` | Normalized hostname used for the DNS query. |
| `recordType` | `string` | DNS record type that was queried. |
| `records` | `array or object` | DNS records returned by the resolver. The structure depends on the selected record type. |

### Successful Response

Example response for an `A` record lookup:

```json
{
  "hostname": "google.com",
  "recordType": "A",
  "records": [
    "142.251.209.206"
  ]
}
```

### Record-Specific Output

Different record types return different structures.

#### MX Records

```json
{
  "hostname": "example.com",
  "recordType": "MX",
  "records": [
    {
      "exchange": "mail.example.com",
      "priority": 10
    }
  ]
}
```

#### SOA Record

```json
{
  "hostname": "example.com",
  "recordType": "SOA",
  "records": {
    "nsname": "ns1.example.com",
    "hostmaster": "admin.example.com",
    "serial": 2026080601,
    "refresh": 3600,
    "retry": 600,
    "expire": 86400,
    "minttl": 300
  }
}
```

### Error Response

The node throws an error when the hostname is missing:

```text
Hostname is required for DNS lookup
```

DNS resolution errors are wrapped as:

```text
DNS lookup failed: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Resolve an IPv4 Address

Resolve a domain to its IPv4 address.

**Configuration**

```text
Hostname: google.com
Record Type: A
```

**Output**

```json
{
  "hostname": "google.com",
  "recordType": "A",
  "records": [
    "142.251.209.206"
  ]
}
```

---

### Example: Resolve an IPv6 Address

Query AAAA records.

**Configuration**

```text
Hostname: google.com
Record Type: AAAA
```

The response contains one or more IPv6 addresses when available.

---

### Example: Query Mail Servers

Retrieve MX records for a domain.

**Configuration**

```text
Hostname: gmail.com
Record Type: MX
```

The response contains mail exchange servers and their priorities.

---

### Example: Query Name Servers

Retrieve authoritative name servers.

**Configuration**

```text
Hostname: example.com
Record Type: NS
```

---

### Example: Query TXT Records

Retrieve text records for a domain.

**Configuration**

```text
Hostname: example.com
Record Type: TXT
```

TXT records may contain verification values, SPF policies, or other domain metadata.

---

### Example: Query a CNAME Record

Retrieve canonical name aliases.

**Configuration**

```text
Hostname: www.example.com
Record Type: CNAME
```

---

### Example: Query an SOA Record

Retrieve start-of-authority information.

**Configuration**

```text
Hostname: example.com
Record Type: SOA
```

---

### Example: Query an SRV Record

Retrieve service discovery records.

**Configuration**

```text
Hostname: _sip._tcp.example.com
Record Type: SRV
```

---

### Example: Reverse DNS Lookup

Query a PTR record using an IP address.

**Configuration**

```text
Hostname: 8.8.8.8
Record Type: PTR
```

---

### Example: Dynamic String Input

Receive the hostname from the previous node.

**Input**

```text
google.com
```

**Configuration**

```text
Hostname: (empty)
Record Type: A
```

The node uses the incoming string as the hostname.

---

### Example: Dynamic Domain Input

Receive a domain from an incoming object.

**Input**

```json
{
  "domain": "google.com"
}
```

**Configuration**

```text
Hostname: (empty)
Record Type: A
```

---

### Example: Dynamic URL Input

Receive a complete URL from the previous node.

**Input**

```json
{
  "url": "https://example.com/products?id=10"
}
```

The node extracts:

```text
example.com
```

before performing the lookup.

---

### Example: Missing Hostname

If no hostname is configured or provided by the previous node, the node throws:

```text
Hostname is required for DNS lookup
```

<!-- /SECTION: examples -->
---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Resolve a Domain to an IPv4 Address
```

### Common Patterns

- **IPv4 Resolution:** Manual Trigger → DNS Lookup → Log
- **Dynamic Domain Lookup:** HTTP Request → DNS Lookup → Log
- **URL Hostname Resolution:** Webhook → DNS Lookup → Log
- **Mail Server Inspection:** Manual Trigger → DNS Lookup (MX) → Log
- **DNS Monitoring:** Manual Trigger → DNS Lookup → Log
- **Domain Analysis:** Manual Trigger → DNS Lookup → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "Hostname is required for DNS lookup"

**Cause:** No hostname was configured or received from the previous node.

**Solution:** Provide a value through:

- the `hostname` configuration parameter;
- a plain string input;
- an object containing `hostname`;
- an object containing `domain`;
- an object containing `url`.

---

#### "DNS lookup failed"

**Cause:** The DNS resolver returned an error.

Possible reasons include:

- The hostname does not exist.
- The selected record type is unavailable.
- The DNS server could not be reached.
- The input is invalid for the selected record type.
- A reverse DNS lookup was attempted with an unsupported value.

**Solution:**

- Verify the hostname.
- Confirm that the selected record type exists.
- Try the `A` record type first.
- Use an IP address for `PTR` lookups.
- Check network and DNS connectivity.

---

#### No records returned

**Cause:** The domain exists but does not contain the requested record type.

**Solution:** Try another record type such as:

- `A`
- `AAAA`
- `MX`
- `NS`
- `TXT`

---

#### URL is not resolved correctly

**Cause:** The input URL may be malformed.

**Solution:** Provide a complete URL such as:

```text
https://example.com/path
```

or provide the hostname directly:

```text
example.com
```

---

#### PTR lookup fails

**Cause:** PTR resolution usually requires an IP address with a configured reverse DNS record.

**Solution:** Provide a valid IPv4 or IPv6 address and verify that the IP owner has configured reverse DNS.

---

### Error Messages

| Error | Description |
|-------|-------------|
| `Hostname is required for DNS lookup` | No hostname was configured or received. |
| `Unsupported record type` | The requested record type is not supported by the node. |
| `DNS lookup failed` | The underlying DNS resolver returned an error. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- DNS to IP
- IPInfo
- Whois

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release of the DNS Lookup documentation. |

<!-- /SECTION: changelog -->
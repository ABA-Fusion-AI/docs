---
node_id: "dns-to-ip"
title: "DNS to IP"
description: "Resolve hostnames to IPv4 and IPv6 addresses."
category: "security-networking"
subcategory: "network-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - dns
  - ip
  - networking
  - hostname
  - ipv4
  - ipv6
related_nodes:
  - dns-lookup
  - ipinfo
  - whois
---

<!-- SECTION: header -->
# DNS to IP

> **Category:** Security & Networking | **Type:** Action Node

Resolve hostnames into IPv4 and IPv6 addresses using the native Node.js DNS resolver. The node supports IPv4-only, IPv6-only, or dual-stack resolution without requiring external APIs or authentication.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **DNS to IP** node resolves hostnames into one or more IP addresses using the built-in Node.js DNS module. It supports IPv4, IPv6, or both simultaneously and automatically normalizes hostnames before performing DNS resolution.

The hostname can be provided directly in the node configuration or dynamically from previous workflow nodes.

### Key Features

- **IPv4 Resolution:** Resolve hostnames to IPv4 addresses.
- **IPv6 Resolution:** Resolve hostnames to IPv6 addresses.
- **Dual-Stack Resolution:** Retrieve both IPv4 and IPv6 addresses in a single execution.
- **Dynamic Input:** Accept hostnames from workflow data.
- **Automatic URL Parsing:** Extract the hostname from a complete URL.
- **Protocol Removal:** Automatically remove `http://` and `https://`.
- **No API Required:** Uses the native DNS resolver.
- **Structured Output:** Returns hostname, IPv4 addresses, IPv6 addresses, and a combined IP list.

### IP Resolution Modes

| Mode | Description |
|------|-------------|
| `ipv4` | Resolve only IPv4 addresses. |
| `ipv6` | Resolve only IPv6 addresses. |
| `both` | Resolve both IPv4 and IPv6 addresses. |

### Use Cases

- Resolve domains into IP addresses
- Validate DNS configurations
- Check IPv6 availability
- Prepare network monitoring workflows
- Verify infrastructure endpoints
- Build DNS diagnostic workflows
- Perform hostname validation
- Integrate DNS resolution into automation pipelines

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `hostname` | `string` | ⚠️ Conditional | — | Hostname, domain, URL, or IP address to resolve. May also come from workflow input. |
| `ipVersion` | `enum` | ❌ No | `ipv4` | Determines whether IPv4, IPv6, or both should be resolved. |

> **Note:** A hostname is required during execution, but it may be supplied either through the node configuration or incoming workflow data.

### IP Version Values

| Value | Description |
|-------|-------------|
| `ipv4` | Resolve only IPv4 addresses. |
| `ipv6` | Resolve only IPv6 addresses. |
| `both` | Resolve both IPv4 and IPv6 addresses. |

### Default Values

| Parameter | Default |
|-----------|---------|
| `ipVersion` | `ipv4` |

### Hostname Resolution Priority

The node resolves the target hostname using the following priority:

1. Configured `hostname`
2. Incoming string value
3. Incoming `hostname` property
4. Incoming `domain` property
5. Incoming `url` property

### Input Normalization

If the provided value contains a protocol or path, the node extracts only the hostname.

Example:

```text
https://example.com/products
```

becomes:

```text
example.com
```

### Configuration Notes

- The hostname may be a domain name, hostname, URL, or IP address.
- IPv4 resolution uses the Node.js `resolve4()` function.
- IPv6 resolution uses the Node.js `resolve6()` function.
- When `both` is selected, IPv4 and IPv6 results are merged into the `ip` array.
- Missing IPv4 or IPv6 records return empty arrays rather than causing the node to fail.
- The node does not require authentication or API credentials.

<!-- /SECTION: configuration -->
---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional workflow input containing a hostname, domain, or URL. |

The node accepts a plain hostname:

```text
google.com
```

It also accepts an object containing a `hostname` property:

```json
{
  "hostname": "google.com"
}
```

An object containing a `domain` property:

```json
{
  "domain": "google.com"
}
```

Or an object containing a `url` property:

```json
{
  "url": "https://google.com/search"
}
```

When a URL is provided, the node automatically extracts the hostname before performing DNS resolution.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `hostname` | `string` | Normalized hostname used for the DNS resolution. |
| `ipv4` | `string[]` | IPv4 addresses returned by the resolver. |
| `ipv6` | `string[]` | IPv6 addresses returned by the resolver. |
| `ip` | `string[]` | Combined list of all resolved IP addresses. |

### Successful Response

Example response for IPv4 mode:

```json
{
  "hostname": "google.com",
  "ipv4": [
    "172.217.18.110"
  ],
  "ip": [
    "172.217.18.110"
  ]
}
```

### IPv6 Example

```json
{
  "hostname": "google.com",
  "ipv6": [
    "2a00:1450:4006:80e::200e"
  ],
  "ip": [
    "2a00:1450:4006:80e::200e"
  ]
}
```

### Dual-Stack Example

```json
{
  "hostname": "google.com",
  "ipv4": [
    "172.217.18.110"
  ],
  "ipv6": [
    "2a00:1450:4006:80e::200e"
  ],
  "ip": [
    "172.217.18.110",
    "2a00:1450:4006:80e::200e"
  ]
}
```

When one IP version is unavailable, the node returns an empty array for that version instead of failing.

### Error Response

If no hostname is available, the node throws:

```text
Hostname is required for DNS to IP resolution
```

Unexpected DNS resolution failures are returned as:

```text
DNS to IP resolution failed: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Resolve an IPv4 Address

Resolve a hostname to IPv4.

**Configuration**

```text
Hostname: google.com
IP Version: ipv4
```

**Output**

```json
{
  "hostname": "google.com",
  "ipv4": [
    "172.217.18.110"
  ],
  "ip": [
    "172.217.18.110"
  ]
}
```

---

### Example: Resolve an IPv6 Address

Resolve only IPv6 addresses.

**Configuration**

```text
Hostname: google.com
IP Version: ipv6
```

---

### Example: Resolve IPv4 and IPv6

Resolve both IP versions.

**Configuration**

```text
Hostname: google.com
IP Version: both
```

The node returns IPv4 addresses, IPv6 addresses, and a combined `ip` array.

---

### Example: Dynamic Hostname Input

Receive the hostname from a previous workflow node.

**Input**

```text
google.com
```

**Configuration**

```text
Hostname: (empty)
IP Version: ipv4
```

The incoming value is used automatically.

---

### Example: Dynamic Domain Input

**Input**

```json
{
  "domain": "google.com"
}
```

**Configuration**

```text
Hostname: (empty)
IP Version: ipv4
```

---

### Example: Dynamic URL Input

**Input**

```json
{
  "url": "https://example.com/products"
}
```

The node extracts:

```text
example.com
```

before resolving its IP addresses.

---

### Example: IPv4 Not Available

If no IPv4 record exists, the node returns:

```json
{
  "hostname": "example.com",
  "ipv4": [],
  "ip": []
}
```

---

### Example: IPv6 Not Available

If no IPv6 record exists, the node returns:

```json
{
  "hostname": "example.com",
  "ipv6": [],
  "ip": []
}
```

---

### Example: Missing Hostname

If no hostname is configured or provided by the previous node, the node throws:

```text
Hostname is required for DNS to IP resolution
```

<!-- /SECTION: examples -->
---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Resolve a Hostname to an IP Address
```

### Common Patterns

- **Resolve IPv4:** Manual Trigger → DNS to IP → Log
- **Resolve IPv6:** Manual Trigger → DNS to IP → Log
- **Dual-Stack Resolution:** Manual Trigger → DNS to IP → Log
- **Resolve from URL:** HTTP Request → DNS to IP → Log
- **Network Monitoring:** Scheduler → DNS to IP → Log
- **Infrastructure Validation:** DNS to IP → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "Hostname is required for DNS to IP resolution"

**Cause:** No hostname was configured or received from the previous node.

**Solution:**

Provide the hostname through one of the following:

- The `hostname` configuration parameter
- A string input
- An object containing `hostname`
- An object containing `domain`
- An object containing `url`

---

#### "DNS to IP resolution failed"

**Cause:** The DNS resolver encountered an unexpected error.

Possible reasons include:

- Invalid hostname
- Network connectivity problems
- DNS server unavailable
- Invalid URL format
- Temporary DNS resolution failure

**Solution:**

- Verify the hostname.
- Ensure the domain exists.
- Verify network connectivity.
- Retry the request after a short delay.

---

#### Empty IPv4 Result

**Cause:** The domain does not publish IPv4 records.

**Example**

```json
{
  "hostname": "example.com",
  "ipv4": [],
  "ip": []
}
```

**Solution:**

Use `ipv6` or `both` if IPv6 records are expected.

---

#### Empty IPv6 Result

**Cause:** The domain does not publish IPv6 records.

**Example**

```json
{
  "hostname": "example.com",
  "ipv6": [],
  "ip": []
}
```

**Solution:**

Use `ipv4` or `both` depending on your requirements.

---

#### URL Is Not Parsed Correctly

**Cause:** The provided URL is malformed.

**Solution:**

Use a valid URL such as:

```text
https://example.com/path
```

or provide the hostname directly:

```text
example.com
```

---

### Error Messages

| Error | Description |
|-------|-------------|
| `Hostname is required for DNS to IP resolution` | No hostname was provided. |
| `DNS to IP resolution failed` | An unexpected DNS resolution error occurred. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- DNS Lookup
- IPInfo
- Whois

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release of the DNS to IP documentation. |

<!-- /SECTION: changelog -->
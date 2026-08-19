---
node_id: "port-scanner"
title: "Port Scanner"
description: "Scans IP addresses for open ports. Supports single ports, comma-separated ports, or port ranges."
category: "security-networking"
subcategory: "network-scanning"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - port-scanner
  - network
  - security
  - ip
  - open-ports
  - tcp
  - vulnerability-assessment
  - infrastructure
related_nodes:
  - dns-lookup
  - dns-to-ip
  - http-request
  - log
  - pingdom
---

<!-- SECTION: header -->
# Port Scanner

> **Category:** Security & Networking | **Subcategory:** Network Scanning | **Type:** Action Node

Probe IPv4 addresses for open TCP ports using single ports, comma-separated port lists, or continuous port ranges.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Port Scanner** node audits remote hosts and IP addresses to determine TCP port accessibility and network service availability.

It accepts target IPv4 addresses directly via configuration or dynamically from preceding workflow nodes (such as IP Geolocation or DNS Lookup nodes). You can specify individual ports, lists of ports, or ranges up to 1,000 ports per execution. If no ports are configured, the node automatically scans 20 common service ports (including HTTP, HTTPS, SSH, FTP, DNS, MySQL, and RDP).

### Key Features

- **Flexible Target Resolution:** Accepts target IPv4 addresses from node configuration or extracts IP addresses automatically from upstream payloads (`string`, `data.ip`, or `data.ipv4`).
- **Versatile Port Formatting:** Supports single port numbers (`"80"`), comma-separated lists (`"22, 80, 443"`), port ranges (`"1-1000"`), or combinations (`"22, 80, 8000-8080"`).
- **Default Common Port Profile:** Automatically scans 20 standard network service ports if no ports are specified.
- **Configurable Connection Timeout:** Customizable per-port TCP probe timeout in milliseconds (`default: 400ms`).
- **Detailed Scan Metrics:** Returns total scanned count, open count, closed count, an array of open port numbers, and detailed status objects for every probed port.
- **Safety & Abuse Controls:** Enforces a maximum scan limit of 1,000 ports per execution and validates IPv4 address formatting.

### Processing Flow

```text
Resolve Target IPv4 (Config `ip` > Data String > `data.ip` > `data.ipv4`)
                         ↓
             Valid IPv4 Regex Format?
  ├── No  ──→ Throw Error ("Invalid IPv4 address format")
  └── Yes ──→ Parse Ports Parameter
                         ↓
           Ports Parameter Provided?
  ├── No  ──→ Load 20 Common Default Ports (21, 22, 23, 25, 53, 80, 110, 111, 135, 139, 143, 443, 445, 993, 995, 1723, 3306, 3389, 5900, 8080)
  └── Yes ──→ Parse Ranges ("1-100") & Comma Lists ("80,443") & Deduplicate
                         ↓
            Ports Count > 1000?
  ├── Yes ──→ Throw Error ("Cannot scan more than 1000 ports at once")
  └── No  ──→ Probe TCP Ports Concurrently (portscanner checkPortStatus)
                         ↓
Aggregate Results (scanned, open, closed, openPorts array, detailed results)
                         ↓
Return Structured JSON Payload
```

### Use Cases

- **Infrastructure & Network Auditing:** Verify which network ports are exposed on cloud servers, firewalls, or edge routers.
- **Vulnerability & Compliance Scanning:** Ensure unauthorized or high-risk administrative services (such as Telnet `23`, RDP `3389`, or SMB `445`) are not publicly exposed.
- **Automated Service Health Checks:** Monitor whether application web servers (ports `80`, `443`) or database engines (port `3306`) are actively accepting connections.
- **Security Incident Response:** Dynamically check suspect IP addresses for open command-and-control or proxy ports.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ip` | `string` | ❌ No* | `""` | Target IPv4 address to scan (e.g. `"1.1.1.1"`). If empty, target IP is extracted from incoming workflow payload. |
| `ports` | `string` | ❌ No | `""` | Comma-separated list of ports or ranges to scan (e.g., `"53, 80, 443"` or `"8000-8080"`). Defaults to 20 common ports if empty. |
| `timeout` | `number` | ❌ No | `400` | Socket probe timeout per port in milliseconds (must be `>= 0`). |

*\* Note: Either `ip` must be configured in the parameter field or an IP address string/object must be passed into the input port.*

---

### Parameter Details

#### `ip`
The target IPv4 host address.
- **Type:** `string`
- **Format:** Standard IPv4 notation (`x.x.x.x`)
- **Example:** `"1.1.1.1"`, `"8.8.8.8"`, `"192.168.1.1"`
- **Dynamic Expression:** Can reference upstream outputs using expressions like `{{outputs.DNS_Lookup.success.ip}}`.

---

#### `ports`
Specification of TCP ports to probe.
- **Type:** `string`
- **Supported Formats:**
  - Single Port: `"80"`
  - Comma-Separated List: `"22, 80, 443, 3306"`
  - Port Range: `"1-100"`
  - Combination: `"22, 80, 8000-8080"`
- **Default (when omitted):** Scans 20 common ports: `21` (FTP), `22` (SSH), `23` (Telnet), `25` (SMTP), `53` (DNS), `80` (HTTP), `110` (POP3), `111` (RPC), `135` (RPC), `139` (NetBIOS), `143` (IMAP), `443` (HTTPS), `445` (SMB), `993` (IMAPS), `995` (POP3S), `1723` (PPTP), `3306` (MySQL), `3389` (RDP), `5900` (VNC), `8080` (HTTP Alt).

---

#### `timeout`
Maximum wait time for each port connection attempt.
- **Type:** `number`
- **Default:** `400` (milliseconds)
- **Example:** `500`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload. Can supply the target IP if `ip` parameter is left empty. |

---

### Target IP Resolution Priority

If the `ip` parameter in node configuration is omitted, the node resolves the target IP from incoming workflow data using the following fallback order:

1. **String Input:** Payload is a direct string (e.g. `"1.1.1.1"`).
2. **Object Property `ip`:** Payload object containing property `ip` (e.g. `{ "ip": "1.1.1.1" }`).
3. **Object Property `ipv4`:** Payload object containing property `ipv4` (supports string or array format, e.g. `{ "ipv4": ["1.1.1.1"] }`).

---

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when scan execution completes. Contains IP summary metrics, array of open ports, and detailed per-port status objects. |
| `error` | `Error` | Emitted if IP address is missing, IP format is invalid, or total ports exceed 1,000. |

---

### Output Data Structure Example

Scanning IP `"1.1.1.1"` for ports `"53, 80, 443"`:

```json
{
  "ip": "1.1.1.1",
  "scanned": 3,
  "open": 2,
  "closed": 1,
  "openPorts": [
    53,
    80
  ],
  "results": [
    {
      "port": 53,
      "status": "open",
      "open": true
    },
    {
      "port": 80,
      "status": "open",
      "open": true
    },
    {
      "port": 443,
      "status": "closed",
      "open": false
    }
  ]
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `ip` | `string` | Evaluated target IPv4 address. |
| `scanned` | `number` | Total number of unique valid ports probed during the scan. |
| `open` | `number` | Total count of ports returning an open TCP status. |
| `closed` | `number` | Total count of ports returning a closed or filtered TCP status. |
| `openPorts` | `array` | Array of port numbers (`number[]`) that were confirmed open. |
| `results` | `array` | Array of detailed result objects for every scanned port. |
| `results[].port` | `number` | Port number probed. |
| `results[].status` | `string` | Connection status string (`"open"` or `"closed"`). |
| `results[].open` | `boolean` | `true` if port is open, `false` if closed or timed out. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Scan Standard Public Web & DNS Ports

Scan Cloudflare DNS (`1.1.1.1`) for common DNS and web service ports.

**Configuration:**

```text
IP: 1.1.1.1
Ports: 53, 80, 443
Timeout: 400
```

**Output:**

```json
{
  "ip": "1.1.1.1",
  "scanned": 3,
  "open": 2,
  "closed": 1,
  "openPorts": [53, 80],
  "results": [
    { "port": 53, "status": "open", "open": true },
    { "port": 80, "status": "open", "open": true },
    { "port": 443, "status": "closed", "open": false }
  ]
}
```

---

### Example 2: Continuous Range Scanning

Scan internal gateway IP (`192.168.1.1`) for ports in range `20-80`.

**Configuration:**

```text
IP: 192.168.1.1
Ports: 20-80
Timeout: 300
```

**Result Output:**
- Probes all 61 ports between `20` and `80`.
- Returns open ports (e.g. `[22, 53, 80]`).

---

### Example 3: Dynamic IP Input from Upstream DNS Node

Scan an IP address resolved dynamically from an upstream **DNS to IP** node.

**Workflow Pipeline:**

```text
Manual Trigger
  → DNS to IP (domain: "example.com")
  → Port Scanner (ip: {{outputs.DNS_to_IP.success.ip}}, ports: "80, 443")
  → Log (Display open ports)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Port Scanner Example Workflow
```

### How it flows

1. **Manual Trigger:** Starts the port scanning workflow.
2. **Port Scanner Node:** Probes target IPv4 address (`1.1.1.1`) across specified ports (`53, 80, 443`).
3. **Log Node:** Prints scan metrics (`scanned`, `openPorts`, `results`) to the execution console log.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Adjust Timeout for Latency:** For remote servers across high-latency networks or cellular connections, increase `timeout` (e.g. to `1000` ms) to prevent false closed statuses caused by slow network handshakes.
2. **Respect the 1,000 Port Limit:** To avoid workflow timeouts and excessive bandwidth consumption, scan targeted port groups or smaller ranges (`< 1000` ports) per node execution.
3. **Handle Firewall Packet Dropping:** Note that firewalls silently dropping packets will cause probes to wait for the full `timeout` duration before reporting ports as `closed`.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Error Messages

#### `IP address is required for port scanning`
- **Cause:** No IP address was supplied in node parameters and no valid IP string/object was received from the input port.
- **Solution:** Specify an IPv4 address in `ip` or pass payload data containing an `ip` or `ipv4` property.

#### `Invalid IPv4 address format`
- **Cause:** Target address string fails IPv4 regex validation (e.g. `"example.com"` domain name or `"256.0.0.1"`).
- **Solution:** Pass a valid IPv4 address in `x.x.x.x` format. If starting with a domain name, use a **DNS to IP** node first to resolve the domain to an IP address.

#### `Cannot scan more than 1000 ports at once`
- **Cause:** Configured port range or list contains more than 1,000 unique ports.
- **Solution:** Reduce the port range span (e.g. `"1-500"`) or split scans across multiple nodes.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [DNS Lookup](../dns-lookup/en.md) — Query DNS record types (A, AAAA, MX, TXT)
- [DNS to IP](../dns-to-ip/en.md) — Resolve domain names to IPv4 addresses
- [HTTP Request](../http-request/en.md) — Send HTTP/HTTPS requests to open web ports
- [Log](../log/en.md) — Print scan metrics to the execution log

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial release of the Port Scanner node supporting IPv4 resolution, single ports, port lists, port ranges, default port profiles, and concurrent probing |

<!-- /SECTION: changelog -->

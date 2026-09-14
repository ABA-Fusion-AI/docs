---
node_id: "subdomain-discovery"
title: "Subdomain Discovery"
description: "Discover subdomains using certificate transparency, DNS brute-forcing, SecurityTrails, and optional subfinder execution."
category: "security"
subcategory: "reconnaissance"
version: "1.1.0"
language: "en"
last_updated: "2026-09-10"
author: "Fusion Team"
tags: [security, dns, subdomains, reconnaissance]
---

<!-- SECTION: overview -->
# Subdomain Discovery

> **Category:** Security&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Discovers subdomains by combining certificate transparency data from `crt.sh`, DNS brute-force wordlists, SecurityTrails API lookups, and an optional `subfinder` command. It can also detect wildcard DNS answers and label those results as wildcard subdomains.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `domain` | string | No | — | Target domain to scan. If omitted, the node can read an incoming string, an object with `domain`, or an object with `url` and parse the hostname. |
| `methods` | string[] | No | `['certificate-transparency', 'dns-brute-force']` | Discovery methods to run. Allowed values are `certificate-transparency`, `dns-brute-force`, `securitytrails`, `subfinder`, and `all`. |
| `wordlist` | string[] | No | built-in default list | Custom DNS brute-force wordlist. If omitted or empty, the node uses the built-in default list. |
| `detectWildcard` | boolean | No | `true` | Detects wildcard DNS by resolving a random name under the target domain. |
| `timeout` | number | No | `5000` | Timeout in milliseconds for each network or OS subprocess call. |
| `maxConcurrent` | number | No | `10` | Maximum number of concurrent DNS brute-force checks. |
| `securityTrailsApiKey` | string | No | — | SecurityTrails API key for the `securitytrails` discovery method. Keep in the secrets store. |
| `subfinderPath` | string | No | `subfinder` | Optional executable path or command name for `subfinder`. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** A workflow event. The node accepts either a raw domain string, an event object with a `domain` field, or an event object with a `url` field from which the hostname is extracted.
- **Success:** An object with `domain`, `total`, `subdomains`, `wildcard`, and `bySource`. The `subdomains` entry contains `subdomain`, `ip`, `source`, and optional `isWildcard` markers.
- **Error:** Domain missing, network timeout, DNS resolution failure, SecurityTrails authentication failure, or a missing `subfinder` binary.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Subdomain discovery using brute-force and certificate transparency
```

The repository example workflow shows three sample nodes:

- `openai.com` with `methods: ["dns-brute-force"]`, `wordlist: ["www", "api", "mail", "dev", "staging"]`, and `maxConcurrent: 5`.
- `github.com` with `methods: ["certificate-transparency"]`, `wordlist: []`, and `timeout: 4997`.
- `google.com` with `methods: ["dns-brute-force"]`, `wordlist: ["www", "api", "mail", "accounts", "maps", "drive"]`, and `timeout: 10000`.

See [example.workflow.json](./example.workflow.json) for the full workflow diagram.
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Only scan domains that you own or are authorized to assess. Keep `securityTrailsApiKey` in the platform secret store instead of placing it directly in workflows. The node consults DNS and external services such as `crt.sh`, SecurityTrails, and `subfinder`; respect the service terms and perform responsible subdomain reconnaissance only.

The node executes `subfinder` through the configured command path rather than interpolating shell input into a command string, and the repository code is careful to reject runtimes that cannot resolve a command path.
<!-- /SECTION: security -->

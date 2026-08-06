---
node_id: "subdomain-discovery"
title: "Subdomain Discovery"
description: "Discover subdomains using certificate logs, DNS, SecurityTrails, and subfinder."
category: "security"
subcategory: "reconnaissance"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [security, dns, subdomains, reconnaissance]
---

# Subdomain Discovery

Discovers subdomains using certificate transparency, DNS brute force, SecurityTrails, and optionally `subfinder`.

## Security

Domains must be valid DNS names. The node executes `subfinder` directly with an argument array, without a command shell, so domain values cannot append shell commands. `subfinderPath` must point to an executable named `subfinder` or `subfinder.exe`.

Only scan domains that you own or are authorized to assess.

## Workflow example

See [example.workflow.json](./example.workflow.json).

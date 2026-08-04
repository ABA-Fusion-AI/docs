---
node_id: "google-dork"
title: "Google Dork"
description: "Run Google searches with advanced operators in a controlled headless browser."
category: "web-search"
subcategory: "search-reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [google, dork, security, search, puppeteer]
related_nodes: [google-search, url-scan-search]
---

<!-- SECTION: overview -->
# Google Dork

> **Category:** Web Search&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Search Google with operators such as `site:`, `inurl:`, `intext:`, and `filetype:`. The node launches the repository-pinned Puppeteer browser directly; it does not execute shell commands or download packages at runtime.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `query` | string | Yes | — | Search query and optional dork operators. |
| `maxResults` | number | No | `10` | Maximum results returned. |
| `timeout` | number | No | `30000` | Navigation timeout in milliseconds. |
| `headless` | boolean | No | `true` | Run Chrome without a visible window. |
| `executablePath` | string | No | bundled browser | Optional Chrome or Chromium executable. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An event that starts the configured search.
- **Success:** Query, result count, and normalized title, URL, and snippet objects.
- **Error:** Browser launch, timeout, navigation, or scraping error.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Find public PDF documents on a domain
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Only search systems you are authorized to assess. Google may rate-limit or challenge automated requests.
<!-- /SECTION: security -->

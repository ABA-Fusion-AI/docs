---
node_id: "icon-horse"
title: "IconHorse"
description: "Fetch high-resolution favicons for any website using the IconHorse API"
category: "web-search"
subcategory: "icons"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - favicon
  - icon
  - website
  - web
related_nodes:
  - http-request
  - url-encode
  - function
---

<!-- SECTION: header -->
# IconHorse

> **Category:** Web Search & Information | **Type:** Action Node

Fetch high-resolution favicons for websites using the IconHorse API, making it easy to enrich UI, branding, or asset workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **IconHorse** node queries the IconHorse service to retrieve a website favicon or icon asset. It is useful when a workflow needs to enrich data with visual branding elements or fetch icons for a domain automatically.

### Key Features

- **Favicon Retrieval:** Fetch favicons for any domain
- **High-Resolution Support:** Retrieve assets suitable for UI or branding use cases
- **Simple Input:** Use a domain name as the main input
- **Automation Ready:** Integrate with downstream image or asset handling nodes

### Use Cases

- Collect website icons for directories or dashboards
- Enrich CRM or portal data with brand visuals
- Automate favicon downloads for content systems
- Support web research and website analysis workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | `string` | ✅ Yes | — | Website domain to look up, such as `github.com` |

### Example

```text
domain: "github.com"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data that can supply the domain value |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Favicon or icon response from the API |
| `error` | `object` | Error information if the lookup fails |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Retrieve the GitHub Favicon

```text
domain: "github.com"
```

**Result:**

```json
{
  "domain": "github.com",
  "icon": "https://..."
}
```

### Example: Use in a Web Enrichment Workflow

Pass the returned icon URL to a download or storage node for later use in dashboards or catalogs.

<!-- /SECTION: examples -->

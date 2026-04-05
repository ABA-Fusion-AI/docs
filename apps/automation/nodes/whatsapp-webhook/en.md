---
node_id: "whatsapp-webhook"
title: "Whatsapp Webhook"
description: "Integration node for whatsapp-webhook"
category: "triggers"
subcategory: ""
version: "1.0.0"
language: "en"
last_updated: "2026-01-31"
author: "Fusion Team"
tags:
  - triggers
  - whatsapp-webhook
related_nodes: []
---

<!-- SECTION: header -->
# Whatsapp Webhook

> **Category:** Triggers | **Type:** Action Node

Integration node for whatsapp-webhook.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Whatsapp Webhook** node provides integration with whatsapp-webhook service.

### Key Features

- Feature 1
- Feature 2
- Feature 3

### Use Cases

- Use case 1
- Use case 2

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | API key or token |

### Authentication

Refer to the official whatsapp-webhook documentation for authentication details.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Input data |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Operation result |
| `error` | `Error` | Error message |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example

**Configuration:**
```json
{
  "apiKey": "your-api-key"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

**Authentication Error**
- Verify your API key is correct
- Check if the key has appropriate permissions

<!-- /SECTION: troubleshooting -->

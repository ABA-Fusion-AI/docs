---
node_id: "kobotoolbox-hook"
title: "KoboToolbox Hook"
description: "Manage webhooks, review execution logs, and trigger retries in KoboToolbox."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-12"
author: "Fusion Team"
tags: [kobotoolbox, webhooks, hooks, retry, celery, peer-only]
related_nodes: ["kobotoolbox-form", "kobotoolbox-submission", "kobotoolbox-file", "kobotoolbox"]
---

<!-- SECTION: overview -->
# KoboToolbox Hook

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **KoboToolbox Hook** node manages external REST services (webhooks) attached to KoboToolbox projects. It enables workflows to query active webhooks, inspect delivery logs, diagnose failed attempts, and trigger background retry jobs.

### Key Capabilities
- **List Webhooks (`getMany`)**: List all external webhooks registered for a form.
- **Get Webhook Details (`get`)**: Fetch webhook configuration, target URL, and payload settings.
- **Review Logs (`logs`)**: Fetch delivery history with filtering by execution status (`0`: failed after retries, `1`: pending, `2`: succeeded) and date ranges.
- **Retry All Failed Deliveries (`retryAll`)**: Asynchronously dispatch background retry jobs in KoboToolbox via Celery.
- **Retry Single Delivery (`retryOne`)**: Manually re-trigger an individual failed submission delivery.
<!-- /SECTION: overview -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Configuration Parameters
- **Operation (`operation`)**: `getMany`, `get`, `logs`, `retryAll`, or `retryOne`.
- **API Token (`apiToken`)**: Authentication token.
- **Base URL (`baseUrl`)**: Server URL (Default: `https://kf.kobotoolbox.org`).
- **Form ID (`formId`)**: Form asset UID.
- **Hook ID (`hookId`)**: Webhook UID (required for `get`, `logs`, `retryAll`, `retryOne`).
- **Log ID (`logId`)**: Specific log UID (required for `retryOne`).
- **Status Filter (`status`)**: `0` (failed), `1` (pending), `2` (succeeded).
- **Date Filters (`startDate`, `endDate`)**: ISO-8601 timestamps for filtering logs.
- **Pagination (`start`, `limit`)**: Offset and page size controls.

### Outputs
- **Success Output**: Emits webhook configurations, delivery logs, or retry operation status.
- **Error Output**: Standard error output detailing API error messages and HTTP status codes.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Monitor and Retry Failed Webhook Logs
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

- Webhook payloads can contain submission data; ensure webhook endpoints are protected with HTTPS and signature verification where available.
<!-- /SECTION: security -->

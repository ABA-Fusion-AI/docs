---
node_id: "signl4"
title: "SIGNL4"
description: "Send alerts, resolve or acknowledge incidents via SIGNL4"
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-23"
author: "Fusion Team"
tags: [integration, peer-only]
related_nodes: []
---

<!-- SECTION: overview -->
# SIGNL4

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send alerts, resolve or acknowledge incidents via SIGNL4 using a team webhook secret. Every operation sends a JSON `POST` request to `https://connect.signl4.com/webhook/{teamSecret}` with `Content-Type: application/json` and identifies the source system as `FusionAI`.
<!-- /SECTION: overview -->

<!-- SECTION: parameters -->
## Parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `teamSecret` | String | Yes | None | Team secret (webhook ID). Supports expressions. |
| `operation` | Enum | No | `sendAlert` | `sendAlert`, `resolveAlert`, or `acknowledgeAlert`. |
| `title` | String | No | None | Alert title. Used only by `sendAlert`. Supports expressions. |
| `message` | String | No | None | Alert message. Used only by `sendAlert`. Supports expressions. |
| `severity` | Enum | No | `major` | `critical`, `major`, `minor`, `warning`, or `informational`. Used only by `sendAlert`. |
| `service` | String | No | None | Value sent as `X-S4-Service`. Used only by `sendAlert`. Supports expressions. |
| `externalId` | String | For resolve/acknowledge | None | Value sent as `X-S4-ExternalID`. Optional when sending; required when resolving or acknowledging. Supports expressions. |

The node does not require a title or message. Empty optional string values are omitted from the outgoing alert payload.
<!-- /SECTION: parameters -->

<!-- SECTION: operations -->
## Operations

### Send Alert

Use `sendAlert` to submit an alert. The payload always includes `X-S4-SourceSystem: "FusionAI"`. Non-empty `title` and `message` values are sent under those names; `service` and `externalId` are sent as `X-S4-Service` and `X-S4-ExternalID`.

Severity determines these webhook fields:

| Severity | `X-S4-Severity` | `X-S4-AlertingScenario` |
| --- | --- | --- |
| `informational` | -1 | `single_ack` |
| `warning` | 0 | `multi_ack` |
| `minor` | 1 | `multi_ack` |
| `major` (default) | 2 | `multi_ack` |
| `critical` | 3 | `multi_ack` |

Set an `externalId` when you plan to reference the alert in a later resolve or acknowledge operation.

### Resolve Alert

Use `resolveAlert` with the alert's `externalId`. The payload contains `X-S4-ExternalID`, `X-S4-Status: "resolved"`, and `X-S4-SourceSystem: "FusionAI"`.

### Acknowledge Alert

Use `acknowledgeAlert` with the alert's `externalId`. The payload contains `X-S4-ExternalID`, `X-S4-Status: "acknowledged"`, and `X-S4-SourceSystem: "FusionAI"`.

Resolve and acknowledge operations ignore `title`, `message`, `severity`, and `service`.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Any incoming workflow data. It is preserved in the result and is not automatically merged into the webhook payload.
- **Success:** After reading a successful HTTP response, the node returns:

```typescript
{
  operation,     // Selected operation
  success: true,
  data,          // SIGNL4 response body
  input: incomingData
}
```

The implementation attempts to parse the response as JSON, then attempts to read it as text if JSON parsing fails. That fallback may fail if the response body has already been consumed.

- **Error:** Missing required values, network failures, response-reading failures, and unsuccessful HTTP responses throw errors instead of returning a success object.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: errors -->
## Errors

| Condition | Error |
| --- | --- |
| Missing or empty team secret | `Team secret (webhook ID) is required` |
| Missing or empty external ID for resolve/acknowledge | `externalId is required to resolve/acknowledge an alert` |
| Unsuccessful send response, after reading its body | `SIGNL4 sendAlert failed: <serialized response>` |
| Unsuccessful resolve/acknowledge response, after reading its body | `SIGNL4 <operation> failed: <serialized response>` |
| Unsupported operation reaches the handler | `Unknown operation: <operation>` |

The schema restricts operation and severity values to the listed enums. The handler does not implement retries.
<!-- /SECTION: errors -->

<!-- SECTION: examples -->
## Example Workflow

Configure a send operation with the following values, replacing the team secret placeholder with your team's webhook secret:

```json
{
  "teamSecret": "<team-webhook-secret>",
  "operation": "sendAlert",
  "title": "API health check failed",
  "message": "The API health check returned an error.",
  "severity": "critical",
  "service": "Production API",
  "externalId": "api-health-001"
}
```

To acknowledge the same alert, use the same team secret and external ID:

```json
{
  "teamSecret": "<team-webhook-secret>",
  "operation": "acknowledgeAlert",
  "externalId": "api-health-001"
}
```

To resolve it, change `operation` to `resolveAlert` and keep the same `externalId`.

```fusion-workflow
src: example.workflow.json
title: Use SIGNL4 in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Treat `teamSecret` as a credential: it forms part of the webhook URL. Use a secure expression-backed value where available, and keep real team secrets out of shared workflow exports and examples.
<!-- /SECTION: security -->

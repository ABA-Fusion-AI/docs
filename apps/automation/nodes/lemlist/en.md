---
node_id: "lemlist"
title: "Lemlist"
description: "Cold email outreach with Lemlist"
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
# Lemlist

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Use the Lemlist action node to retrieve campaigns, list campaign leads, add a lead to a campaign, retrieve team information, or fetch activities. Each execution sends one request to Lemlist and returns the parsed JSON response.

### Use Cases

- Retrieve campaigns before selecting one for an outreach workflow.
- Send a contact from an upstream workflow step to a Lemlist campaign.
- Retrieve leads or activities for downstream processing and reporting.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | None | Lemlist API key. Supports expressions. |
| `operation` | `enum` | No | `getCampaigns` | One of `getCampaigns`, `getLeads`, `addLead`, `getTeam`, or `getActivities`. |
| `campaignId` | `string` | For `getLeads` and `addLead` | None | Campaign ID used in the request path. Supports expressions. |
| `email` | `string` | For `addLead` | None | Lead email address used in the request path. Supports expressions. |
| `firstName` | `string` | No | None | Lead's first name, sent only by `addLead`. Supports expressions. |
| `lastName` | `string` | No | None | Lead's last name, sent only by `addLead`. Supports expressions. |
| `companyName` | `string` | No | None | Lead's company name, sent only by `addLead`. Supports expressions. |

`campaignId` and `email` are optional in the schema. The implementation does not check that they are present for operations that use them, so supply them before running those operations. The node does not validate email format or URL-encode these path values.

### Authentication

The node sends HTTP Basic authentication with an empty username and the API key as the password. It builds the `Authorization` header automatically from `base64(":" + apiKey)`. Provide the API key itself, without a `Basic` prefix or Base64 encoding.

### Operations

All paths below are relative to `https://api.lemlist.com/api`. Requests use `Content-Type: application/json`.

| Operation | Method | Path | Behavior |
|-----------|--------|------|----------|
| `getCampaigns` | `GET` | `/campaigns` | Retrieve campaigns. No additional fields are used. |
| `getLeads` | `GET` | `/campaigns/{campaignId}/leads` | Retrieve leads for the specified campaign. |
| `addLead` | `POST` | `/campaigns/{campaignId}/leads/{email}` | Add a lead to the specified campaign. Send the optional name and company fields as JSON. |
| `getTeam` | `GET` | `/team` | Retrieve team information. No additional fields are used. |
| `getActivities` | `GET` | `/activities` | Retrieve activities. No additional fields are used. |

For `addLead`, the request body contains only `firstName`, `lastName`, and `companyName` when supplied. Omitted fields are excluded from the JSON body; if all three are omitted, the body is `{}`. The email address and campaign ID appear in the URL, not in the body.

The node exposes no pagination, filtering, sorting, or custom lead fields. It returns the response from a single request without fetching additional pages.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An incoming workflow event triggers execution. The handler does not read `incomingData` directly; request values come from the node configuration. Use expression-enabled parameters to supply dynamic values.
- **Success:** The parsed Lemlist JSON response is returned unchanged. Its shape depends on the selected operation and the API response; the node adds no wrapper or extra fields.
- **Error:** Validation, network, JSON parsing, or remote-service failures fail the action and are handled through the error output.

### Error Behavior

The node parses the response as JSON before checking its HTTP status. For an unsuccessful HTTP response with valid JSON, it throws an error with this message:

```text
Lemlist error: <JSON-stringified response body>
```

An empty or non-JSON response causes JSON parsing to fail before this message can be constructed. Network errors propagate from `fetch`. The implementation adds no retries or custom timeout, and its `stop()` method does not cancel an in-flight request.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

Connect a manual trigger to Lemlist, then route its success output to a step that inspects or processes the response. Configure your API key and choose `getCampaigns` to retrieve campaigns.

```fusion-workflow
src: example.workflow.json
title: Use Lemlist in a workflow
```

### Add a Lead to a Campaign

Set `operation` to `addLead`, supply the target campaign ID and the contact's email, and optionally map their name and company from an upstream step. For example:

```json
{
  "operation": "addLead",
  "campaignId": "YOUR_CAMPAIGN_ID",
  "email": "alex@example.com",
  "firstName": "Alex",
  "lastName": "Morgan",
  "companyName": "Example Company"
}
```

Supply `apiKey` separately using a secure expression. Replace `YOUR_CAMPAIGN_ID` with the target campaign's ID. The success output contains Lemlist's response to the lead creation request.
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->

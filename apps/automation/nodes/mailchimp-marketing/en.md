---
node_id: "mailchimp-marketing"
title: "Mailchimp Marketing"
description: "Manage audiences and send marketing campaigns."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-07"
author: "Fusion Team"
tags: [mailchimp, marketing, audiences, subscribers, campaigns, integration, peer-only]
related_nodes: [http-request, function]
---

<!-- SECTION: overview -->
# Mailchimp Marketing

> **Category:** Peer-only Integrations | **Type:** Action Node

Add a subscriber to a Mailchimp audience, or create a regular marketing campaign, set its HTML content, and send it. The node calls the Mailchimp Marketing API v3.0 using a Marketing API key.

### Use Cases

- Add contacts to an audience with subscriber status and merge fields.
- Create and send an HTML campaign to the configured audience.
- Handle a detected existing-member response in downstream workflow steps.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes, minimum length 1 in schema | None | Mailchimp Marketing API key, such as `xxxxx-us19`. |
| `serverPrefix` | `string` | No, if extractable from key | None | Data-center prefix, such as `us19`. A truthy configured value overrides extraction from the API key. |
| `operation` | `enum` | No | `add_subscriber` | `add_subscriber` or `create_send_campaign`. |
| `listId` | `string` | Yes, minimum length 1 in schema | None | Audience/list ID for either operation. |
| `email` | `string` | At runtime for `add_subscriber` | None | Subscriber email address. |
| `status` | `enum` | No | `subscribed` | Subscriber status: `subscribed`, `pending`, or `unsubscribed`. |
| `mergeFields` | `string` | No | None | JSON text for subscriber merge fields, for example `{"FNAME":"John"}`. |
| `subject` | `string` | At runtime for `create_send_campaign` | None | Campaign subject. |
| `fromName` | `string` | No local requirement | None | Sender name included in campaign settings when supplied. |
| `replyTo` | `string` | No local requirement | None | Reply-to email included in campaign settings when supplied. |
| `htmlContent` | `string` | At runtime for `create_send_campaign` | None | Campaign HTML content. |

The schema declares no expression metadata or conditional field visibility. Values are not trimmed. Email, subject, and HTML are checked for truthiness; whitespace-only strings pass these local checks. No local email-format validation is performed. Mailchimp determines whether submitted subscriber and campaign settings are acceptable.

### Authentication and Server Selection

The server prefix is resolved with `serverPrefix || apiKey.split("-")[1]`. An empty prefix allows extraction from the key; a whitespace-only prefix does not. A missing resolved prefix raises `Invalid API Key format or missing Server Prefix.`

Requests use the base URL:

```text
https://<serverPrefix>.api.mailchimp.com/3.0
```

Every request includes `Content-Type: application/json` and HTTP Basic authentication with username `anystring` and the API key as password. The implementation uses `btoa` to encode these credentials.

### Add a Subscriber

`add_subscriber` makes one request:

```text
POST /lists/<listId>/members
```

The JSON body contains `email_address`, `status`, and `merge_fields`. If `mergeFields` is empty or omitted, `merge_fields` is `{}`. Otherwise, the node parses the string with `JSON.parse`. Use valid JSON with double-quoted property names and strings; JavaScript object literals with single quotes are not valid JSON.

The parsed value is not checked locally to ensure it is an object. Malformed JSON raises a parsing error before the subscriber request is attempted. `listId` is interpolated into the path without URL encoding.

This implementation attempts to create a member; it does not update an existing subscriber. If a caught request error's message contains the case-sensitive text `Member Exists`, the node returns `{ "status": "exists" }`. Other errors propagate. Existing-member detection depends on the error message containing that exact text.

### Create and Send a Campaign

`create_send_campaign` performs three requests in sequence:

| Step | Request | Submitted values |
|------|---------|------------------|
| Create | `POST /campaigns` | Type `regular`, audience `list_id`, and settings `subject_line`, `from_name`, `reply_to`. |
| Set content | `PUT /campaigns/<campaign.id>/content` | `html` from `htmlContent`. |
| Send | `POST /campaigns/<campaign.id>/actions/send` | No request body. |

Only `subject` and `htmlContent` are required by the handler for this operation. Omitted sender fields are omitted during JSON serialization; the API may reject missing or invalid settings.

Running this operation sends the campaign after creation and content submission. The node exposes no draft-only mode, scheduling, recipient-segment filters, or separate confirmation step. It does not validate the returned campaign ID before constructing subsequent request paths.

If a later request fails, earlier steps are not rolled back. A campaign may remain created if content submission or sending fails. Each execution starts by creating a new campaign, so rerunning the operation can create another campaign.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

Incoming workflow data has type `unknown` and is not read by the handler. Parameters come from node configuration.

| Operation / outcome | Returned result |
|---------------------|-----------------|
| Subscriber added | Parsed API JSON, passed through unchanged. |
| Existing member detected | `{ "status": "exists" }` |
| Campaign flow completed | `{ "success": true, "campaignId": "<campaign.id>" }` |
| Validation or request failure | An error is thrown. |

The request helper attempts to parse every response as JSON and uses `null` if parsing fails. A successful non-JSON or empty subscriber response therefore returns `null`. Content and send response values are ignored.

Non-success HTTP responses raise `Marketing API Error: <detail or statusText>`, using the JSON body's truthy `detail` field when available. The helper does not inspect application-error fields in a successful HTTP response. If campaign creation returns `null`, accessing its ID fails before content submission.

The implementation provides no retries, custom timeout, or rollback. The `stop()` method performs no cleanup or request cancellation.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

These objects represent node parameters. Replace the key and audience placeholders with protected credentials and a real audience ID.

### Add a Subscriber with Merge Fields

Omitting `operation` selects `add_subscriber`.

```json
{
  "apiKey": "YOUR_MARKETING_API_KEY-us19",
  "listId": "YOUR_AUDIENCE_ID",
  "email": "alex@example.com",
  "status": "pending",
  "mergeFields": "{\"FNAME\":\"Alex\",\"LNAME\":\"Smith\"}"
}
```

### Set an Explicit Server Prefix

```json
{
  "apiKey": "YOUR_MARKETING_API_KEY",
  "serverPrefix": "us19",
  "operation": "add_subscriber",
  "listId": "YOUR_AUDIENCE_ID",
  "email": "alex@example.com",
  "status": "subscribed"
}
```

### Create and Send an HTML Campaign

Executing this configuration creates and sends a regular campaign for the specified audience.

```json
{
  "apiKey": "YOUR_MARKETING_API_KEY-us19",
  "operation": "create_send_campaign",
  "listId": "YOUR_AUDIENCE_ID",
  "subject": "Monthly product update",
  "fromName": "Example Team",
  "replyTo": "team@example.com",
  "htmlContent": "<html><body><h1>Product update</h1><p>Read our latest news.</p></body></html>"
}
```

### Illustrative Campaign Output

```json
{
  "success": true,
  "campaignId": "EXAMPLE_CAMPAIGN_ID"
}
```

### Workflow Patterns

- Contact event → Mailchimp Marketing (`add_subscriber`) → downstream subscriber processing.
- Campaign preparation → Mailchimp Marketing (`create_send_campaign`) → record the returned campaign ID.
- Subscriber result → Function to handle the `exists` outcome.

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Cause | Resolution |
|------------------|-------|------------|
| `Invalid API Key format or missing Server Prefix.` | No prefix could be resolved from configuration or the key. | Supply the appropriate server prefix or a key containing its data-center suffix. |
| `Email is required.` | Subscriber email is omitted or empty. | Supply `email` for `add_subscriber`. |
| JSON parsing error | `mergeFields` is not valid JSON. | Use double-quoted JSON keys and strings, or omit the field. |
| `Subject and HTML Content are required.` | Campaign subject or HTML is omitted or empty. | Supply both fields for `create_send_campaign`. |
| `Marketing API Error: <message>` | Mailchimp returned a non-success HTTP status. | Check the returned detail, key, audience, subscriber fields, or campaign settings. |
| `{ "status": "exists" }` | The subscriber request failed with a message containing `Member Exists`. | Handle the existing-member outcome downstream; this node has not updated the member. |
| Campaign creation succeeded but a later step failed | Content submission or sending was rejected. | Inspect the created campaign before rerunning, since the node does not resume or roll back prior steps. |

Network failures propagate directly. A missing or invalid campaign-creation response can also cause an ID-access failure or invalid subsequent request path. An unsupported operation reaching the handler raises `Unknown operation: <operation>`; the schema normally restricts the two supported values.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security

Keep the Marketing API key in protected configuration and avoid including real keys in exported examples or logs. Requests use HTTPS and send the key through the Basic authentication header. Subscriber details and campaign content may contain personal or business information; handle returned data and API error details appropriately in downstream steps. The node does not redact error messages.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) – Make custom Marketing API requests beyond these two operations.
- [Function](../function/en.md) – Prepare merge-field JSON or process subscriber and campaign results.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-07 | Documentation generated from the supplied implementation, covering subscriber creation and the create-content-send campaign flow. |

<!-- /SECTION: changelog -->

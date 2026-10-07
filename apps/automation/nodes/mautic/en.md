---
node_id: "mautic"
title: "Mautic"
description: "Manage contacts, campaigns, emails, and segments in Mautic marketing automation."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags: [mautic, marketing, contacts, campaigns, emails, segments, integration, peer-only]
related_nodes: [http-request, function]
---

<!-- SECTION: overview -->
# Mautic

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage contacts, campaigns, emails, and segments through the configured Mautic instance's API. The node supports eleven operations and authenticates with a username and password using HTTP Basic authentication.

### Use Cases

- Create, retrieve, update, or delete marketing contacts.
- Add an existing contact to a campaign or segment.
- Send an existing Mautic email to a contact.
- Retrieve contacts, campaigns, emails, or segments for downstream processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required at runtime | Default | Description |
|-----------|------|---------------------|---------|-------------|
| `operation` | `enum` | No | `listContacts` | One of the eleven operations listed below. |
| `host` | `string` | All operations | None | Mautic instance URL, such as `https://mautic.example.com`. |
| `username` | `string` | All operations | None | Mautic username for Basic authentication. |
| `password` | `string` | All operations | None | Mautic password for Basic authentication. |
| `contactId` | `string` | Contact lookup, update, deletion, campaign/segment addition, and email sending | None | Existing contact ID. |
| `campaignId` | `string` | `addContactToCampaign` | None | Existing campaign ID. |
| `emailId` | `string` | `sendEmail` | None | Existing Mautic email ID. |
| `segmentId` | `string` | `addContactToSegment` | None | Existing segment ID. |
| `firstName` | `string` | No | None | Contact first name for creation or update. |
| `lastName` | `string` | No | None | Contact last name for creation or update. |
| `email` | `string` | No | None | Contact email address for creation or update. |
| `phone` | `string` | No | None | Contact phone number for creation only. |
| `limit` | `number` | No | `25` | Limit query parameter for the four list operations. |

String fields are optional in the schema, but the handler requires credentials and operation-specific IDs at runtime. The schema declares no expression metadata or conditional field visibility. It imposes no integer or range constraints on `limit`; use a positive integer appropriate for your instance.

### Host and Authentication

The base URL is constructed as `${host}/api` without trimming or normalization. Supply the instance URL without a trailing slash or an existing `/api` suffix. If Mautic is installed under a path, include that installation path in `host`.

Every request includes:

```text
Authorization: Basic <base64(username + ':' + password)>
Content-Type: application/json
```

This implementation uses Basic authentication and does not expose OAuth configuration. Credentials and IDs are checked for truthiness rather than trimmed or validated for format. IDs are interpolated directly into URL paths without URL encoding.

### Operations

Endpoints below are relative to `${host}/api`.

| Operation | Method | Endpoint | Required operation fields |
|-----------|--------|----------|---------------------------|
| `listContacts` | GET | `/contacts?limit=<limit>` | None |
| `getContact` | GET | `/contacts/<contactId>` | `contactId` |
| `createContact` | POST | `/contacts` | No contact fields required locally |
| `updateContact` | PATCH | `/contacts/<contactId>` | `contactId` |
| `deleteContact` | DELETE | `/contacts/<contactId>` | `contactId` |
| `listCampaigns` | GET | `/campaigns?limit=<limit>` | None |
| `addContactToCampaign` | POST | `/campaigns/<campaignId>/contact/<contactId>/add` | `campaignId`, `contactId` |
| `listEmails` | GET | `/emails?limit=<limit>` | None |
| `sendEmail` | POST | `/emails/<emailId>/contact/<contactId>/send` | `emailId`, `contactId` |
| `listSegments` | GET | `/segments?limit=<limit>` | None |
| `addContactToSegment` | POST | `/segments/<segmentId>/contact/<contactId>/add` | `segmentId`, `contactId` |

The table reflects the supplied implementation's request paths. This node does not expose endpoint overrides.

### Contact Request Bodies

Creation maps `firstName` to `firstname` and `lastName` to `lastname`, and also includes configured `email` and `phone` values. Update sends only `firstname`, `lastname`, and `email`; `phone` is ignored for `updateContact`.

Fields with undefined values are omitted by `JSON.stringify`. Empty strings are included, and values are not trimmed. No contact-field requirement or email-format check is enforced locally. If no fields are configured, creation or update sends an empty object; the API determines whether to accept it.

For example, a configured creation request can send:

```json
{
  "firstname": "Alex",
  "lastname": "Smith",
  "email": "alex@example.com",
  "phone": "+12125550123"
}
```

Campaign additions, segment additions, and email sending use POST requests without a body. `sendEmail` selects an existing email and contact by ID; it does not accept a subject or message body.

### Listing Limits

List operations use `limit ?? 25`, so zero is sent as zero rather than replaced with the default. Each execution makes one request. No offset, search filter, automatic pagination, retry, or custom timeout is exposed.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Incoming data has type `unknown` and is not read by the handler. Parameters come from node configuration.
- **Success:** Non-DELETE operations return the parsed API JSON directly, without wrapping or transformation. A successful DELETE returns `{ "success": true }` without reading the response body.
- **Error:** Missing configuration, non-success HTTP responses, network failures, or JSON parsing failures.

The node checks `response.ok`. It does not inspect API-level error fields inside a successful HTTP response. Non-DELETE responses must contain valid JSON; an empty or non-JSON successful response raises a parsing error.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

These JSON objects represent node parameters. Replace credential placeholders with protected values and example IDs with existing records in your Mautic instance.

### List Contacts

Omitting `operation` selects `listContacts`. To list campaigns, emails, or segments, set it to `listCampaigns`, `listEmails`, or `listSegments` with the same credentials and limit.

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "limit": 25
}
```

### Retrieve a Contact

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "getContact",
  "contactId": "123"
}
```

### Create a Contact

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "createContact",
  "firstName": "Alex",
  "lastName": "Smith",
  "email": "alex@example.com",
  "phone": "+12125550123"
}
```

### Update a Contact

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "updateContact",
  "contactId": "123",
  "email": "alex.smith@example.com"
}
```

### Delete a Contact

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "deleteContact",
  "contactId": "123"
}
```

### Add a Contact to a Campaign

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "addContactToCampaign",
  "campaignId": "10",
  "contactId": "123"
}
```

### Send an Existing Email to a Contact

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "sendEmail",
  "emailId": "20",
  "contactId": "123"
}
```

### Add a Contact to a Segment

```json
{
  "host": "https://mautic.example.com",
  "username": "YOUR_USERNAME",
  "password": "YOUR_PASSWORD",
  "operation": "addContactToSegment",
  "segmentId": "30",
  "contactId": "123"
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Mautic in a workflow
```

### Common Patterns

- Lead event → Mautic (`createContact`) → downstream contact processing.
- Existing contact → Mautic (`addContactToCampaign` or `addContactToSegment`).
- Notification event → Mautic (`sendEmail`) with an existing email and contact.
- Scheduled trigger → Mautic list operation → reporting.

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Missing Credentials

The handler checks these values in order and raises the corresponding message:

```text
host is required
username is required
password is required
```

Supply all three settings. Empty strings fail validation, but whitespace-only values are not rejected locally.

### Missing Operation IDs

The handler reports missing IDs with `<field> is required for <operation>`. Contact lookup, update, and deletion require `contactId`. Campaign addition requires `campaignId` and `contactId`; email sending requires `emailId` and `contactId`; segment addition requires `segmentId` and `contactId`.

### HTTP API Failure

```text
Mautic API Error: <status> <statusText> - <response body>
```

The response-body suffix is included only when the error body can be read and is non-empty. Check credentials, account permissions, record IDs, submitted fields, and whether your instance accepts the documented request path. Error response text is included without redaction or truncation.

### Parsing or Network Failure

Network failures and JSON parsing errors propagate directly. Check connectivity and the host URL if the response is HTML, empty, or not valid JSON. The node appends `/api` without normalizing trailing slashes or existing API suffixes.

### Phone Is Not Updated

`phone` is included only in `createContact`. The supplied update body contains first name, last name, and email.

### Unknown Operation

An unsupported value reaching the handler raises `Unknown operation: <operation>`. The schema normally restricts configuration to the eleven documented operations.

The `stop()` method performs no cleanup or cancellation.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.

Use HTTPS for `host` because Basic authentication encodes credentials in Base64. Use an account with permissions appropriate to the selected actions. Handle returned contact data and API error bodies carefully in logs and downstream steps; the implementation does not redact them.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) – Make custom Mautic requests beyond this node's supported operations.
- [Function](../function/en.md) – Prepare contact fields and process returned marketing data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Regenerated documentation from the supplied implementation, covering eleven operations, Basic authentication, request bodies, and error handling. |

<!-- /SECTION: changelog -->

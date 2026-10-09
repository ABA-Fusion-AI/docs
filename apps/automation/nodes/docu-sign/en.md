---
node_id: "docu-sign"
title: "DocuSign"
description: "Manage DocuSign envelopes, recipients, templates, account details, and billing invoices."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-09"
author: "Fusion Team"
tags: [docusign, electronic-signature, envelopes, documents, integration]
related_nodes: []
---

<!-- SECTION: overview -->
# DocuSign

> **Category:** Peer-only Integrations | **Type:** Action Node

Use the DocuSign API to create and manage signing envelopes, inspect documents and recipients, work with templates, and retrieve account or billing information. Select an `operation` for each node instance. The default is `listEnvelopes`.

For example, a workflow can create an envelope from a JSON request body, then use the returned envelope ID in another DocuSign node to check its status.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

### Connection and common fields

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | enum | No | API action to perform. Defaults to `listEnvelopes`. |
| `accessToken` | string | Yes | DocuSign OAuth access token. Configure it on the node: the request header reads the configured value. |
| `accountId` | string | Yes | DocuSign account ID used in every API path. |
| `envelopeId` | string | For envelope-specific operations | ID of the target envelope. |
| `templateId` | string | For `getTemplate`, `updateTemplate`, `deleteTemplate` | ID of the target template. |
| `documentId` | string | For `getEnvelopeDocument`, `downloadEnvelopeDocument` | ID of the target document. |
| `recipientId` | string | For `getEnvelopeRecipient`, `getEnvelopeTabs`, `updateEnvelopeTabs`, `getRecipientView` | ID of the target recipient. |
| `invoiceId` | string | For `getBillingInvoice` | ID of the target invoice. |
| `requestBody` | JSON string | For create and update operations, views, locking, and voiding | JSON sent as the POST or PUT body. The node checks that the string is valid JSON. |

### Supported operations

| Area | Operations |
|------|------------|
| Envelopes | `createEnvelope`, `getEnvelope`, `updateEnvelope`, `deleteEnvelope`, `listEnvelopes`, `voidEnvelope` |
| Documents | `getEnvelopeDocuments`, `getEnvelopeDocument`, `downloadEnvelopeDocument` |
| Recipients and tabs | `getEnvelopeRecipients`, `updateEnvelopeRecipients`, `getEnvelopeRecipient`, `getEnvelopeTabs`, `updateEnvelopeTabs` |
| Envelope details | `getEnvelopeCustomFields`, `updateEnvelopeCustomFields`, `lockEnvelope`, `unlockEnvelope`, `getEnvelopeNotificationSettings`, `updateEnvelopeNotificationSettings`, `getEnvelopeFormData`, `getEnvelopeAuditEvents` |
| Views | `getRecipientView`, `getSenderView` |
| Account and templates | `getAccount`, `listTemplates`, `getTemplate`, `createTemplate`, `updateTemplate`, `deleteTemplate` |
| Billing | `listBillingInvoices`, `getBillingInvoice` |

For any operation whose path contains an envelope, template, document, recipient, or invoice ID, provide that ID even if the editor does not display its field for the selected operation. Some paths are built with an empty ID when it is omitted.

### Filters and operation-specific fields

| Parameter | Used by | Description |
|-----------|---------|-------------|
| `fromDate`, `toDate` | `listEnvelopes` | Date range in `YYYY-MM-DD` format. |
| `status` | `listEnvelopes` | Envelope status filter. |
| `count` | `listEnvelopes`, `listTemplates` | Maximum number of results requested. |
| `startPosition` | `listEnvelopes`, `listTemplates`, `listBillingInvoices` | Pagination start position. The current implementation adds it to requests for envelopes and templates only. |
| `fromDateInvoice`, `toDateInvoice` | `listBillingInvoices` | Invoice date range in `YYYY-MM-DD` format. |
| `encoding` | `downloadEnvelopeDocument` | Document encoding query value, such as `base64` or `bytes`. |
| `returnUrl` | `getRecipientView`, `getSenderView` | Required view return URL. Include the corresponding value in `requestBody` as well; this field is validated but is not inserted into the body automatically. |
| `voidReason` | `voidEnvelope` | Required void reason. Include it in `requestBody` as well; this field is validated but is not inserted into the body automatically. |

`requestBody` is required at runtime for `createEnvelope`, `updateEnvelope`, `updateEnvelopeRecipients`, `createTemplate`, `updateTemplate`, `updateEnvelopeTabs`, `updateEnvelopeCustomFields`, `lockEnvelope`, `updateEnvelopeNotificationSettings`, `getRecipientView`, `getSenderView`, and `voidEnvelope`. Although the editor exposes it for only some of these operations, the runtime still requires it.

The API host is fixed in this node to `https://www.docusign.net/restapi`.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

The node accepts workflow input as an object or string. Configured values take precedence. Object input can supply `accountId`/`account_id`, `envelopeId`/`envelope_id`, `templateId`/`template_id`, `documentId`/`document_id`, `recipientId`/`recipient_id`, `invoiceId`/`invoice_id`, `returnUrl`/`return_url`, `voidReason`/`void_reason`, invoice dates, `encoding`, and `requestBody`/`body`. If `requestBody` is absent, an object in `input.data` is serialized to JSON for it. A string input is treated as an envelope ID for operations beginning with `getEnvelope`, `updateEnvelope`, or `deleteEnvelope`, or as a template ID for `getTemplate`.

The node also reads `accessToken`, `access_token`, or `token` from object input during validation, but the HTTP authorization header uses the **configured** `accessToken`. Set the token on the node before running it.

| Output | Description |
|--------|-------------|
| `success` | Parsed JSON returned by the DocuSign API. The shape depends on the selected operation. |
| `error` | Validation, authentication, rate-limit, HTTP, or JSON parsing failure. The error message includes the operation name. |

The implementation parses every successful response as JSON. Operations that return a binary document or an empty response may therefore fail during response parsing; this node does not currently expose a raw file download.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

This three-node workflow runs `listEnvelopes` and displays the API response. Configure `accessToken` and `accountId` on the DocuSign node before running it.

```fusion-workflow
src: example.workflow.json
title: List DocuSign envelopes and inspect the response
```

To create an envelope, select `createEnvelope` and provide a JSON string in `requestBody` containing the envelope definition required by your DocuSign account. For example, a draft request can start with `{"emailSubject":"Please sign","status":"created"}` and then include the documents and recipients for your use case.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | What to check |
|------------------|---------------|
| `accessToken is required` or HTTP 401 | Set a valid OAuth access token in the node configuration. An input token alone does not populate the request header. |
| `accountId is required` | Set `accountId` in configuration or supply `accountId`/`account_id` in object input. |
| `requestBody is required` | Provide a JSON string for the selected write or view operation. |
| `Invalid JSON in requestBody` | Correct the JSON syntax; `requestBody` must be a JSON string. |
| HTTP 403 or 429 | Check the token's permissions or reduce request frequency, respectively. |
| JSON parsing error on a document download | The response may be binary; the current node expects JSON for every operation. |
<!-- /SECTION: troubleshooting -->

<!-- SECTION: security -->
## Security

Keep OAuth tokens and document contents out of exported workflows and logs. Use Fusion's credential system for the token where available, and avoid placing real personal or signing data in documentation examples.
<!-- /SECTION: security -->

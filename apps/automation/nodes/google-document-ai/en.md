---
node_id: "google-document-ai"
title: "Google Document AI"
description: "OCR, layout, and table extraction for documents using Google Document AI, with normalized text, table, and confidence extraction."
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-07-03"
author: "Fusion Team"
tags:
  - ocr
  - document-ai
  - google
  - google-cloud
  - tables
  - confidence
  - no-code
related_nodes:
  - mistral-doc
  - aws-textract
  - azure-document-intelligence
---

# Google Document AI

> **Category:** Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Extract text and tables with per-element confidence scores from documents using a Google Cloud Document AI processor.

### Use Cases

- Extract full text and reconstructed tables from scanned PDFs and images.
- Feed structured tables from a Layout Parser or Form Parser processor into downstream nodes.
- Score extraction quality per-page/paragraph before acting on it.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

This node uses the same Google OAuth pattern as other Google Cloud nodes in this platform (e.g. Google BigQuery, Google Cloud Storage) — there is no separate credential store, so a refresh token is supplied directly and the node refreshes the access token automatically on every call.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ✅ Yes | — | Google OAuth2 Client ID. |
| `clientSecret` | `string` | ✅ Yes | — | Google OAuth2 Client Secret. |
| `redirectUri` | `string` | ❌ No | `http://localhost` | OAuth2 Redirect URI. |
| `refreshToken` | `string` | ❌ No | — | OAuth2 Refresh Token. Recommended for unattended/offline use — the node refreshes the access token itself, so you never need to re-authenticate manually. |
| `accessToken` | `string` | ❌ No | — | OAuth2 Access Token, if you'd rather manage refreshing yourself. |

The OAuth client must be granted the `https://www.googleapis.com/auth/cloud-platform` scope, and the authenticated identity needs the `roles/documentai.apiUser` IAM role (or broader) on the target project.

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `projectId` | `string` | ✅ Yes | — | Google Cloud Project ID. |
| `location` | `string` | ❌ No | `us` | Processor region, e.g. `us` or `eu`. |
| `processorId` | `string` | ✅ Yes | — | Document AI processor ID. Use a **Layout Parser** or **Form Parser** processor to get tables — the plain Document OCR processor does not extract tables. |
| `inputType` | `enum` | ❌ No | `url` | Whether `document` is a public URL or Base64 content. |
| `document` | `string` | ✅ Yes | — | The public URL or Base64 content of the document/image. |
| `mimeType` | `string` | ❌ No | `application/pdf` | MIME type of the document (`application/pdf`, `image/png`, `image/jpeg`, ...). |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Triggers the node; the incoming data is not used directly. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted with the extraction result. |
| `error` | `Error` | Emitted when the request fails. |

### Output Schema (`success`)

```ts
{
  text: string;                    // document.text — the full extracted text
  tables: { rows: string[][] }[];  // one entry per detected table (header rows first, then body rows)
  confidence: number | null;       // average paragraph layout confidence across all pages, 0-1
  pages: number | null;            // number of pages processed
  raw: unknown;                    // untouched Document AI `Document` object
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Extract text and tables from a PDF

**Configuration:**
```json
{
  "clientId": "{{credentials.google.clientId}}",
  "clientSecret": "{{credentials.google.clientSecret}}",
  "refreshToken": "{{credentials.google.refreshToken}}",
  "projectId": "my-gcp-project",
  "location": "us",
  "processorId": "abcd1234ef567890",
  "inputType": "url",
  "document": "https://example.com/report.pdf",
  "mimeType": "application/pdf"
}
```

**Output (success):**
```json
{
  "text": "Quarterly Report\n\nRevenue increased 12%...",
  "tables": [
    { "rows": [["Quarter", "Revenue"], ["Q1", "$1.2M"], ["Q2", "$1.4M"]] }
  ],
  "confidence": 0.98,
  "pages": 3,
  "raw": { "text": "...", "pages": ["..."] }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Document AI processing failed: ... invalid_grant`

**Cause:** The `refreshToken` is expired or was revoked.

**Solution:** Generate a new refresh token for the OAuth client and update the node's configuration.

#### `Document AI processing failed: ... PERMISSION_DENIED`

**Cause:** The authenticated identity lacks the `documentai.apiUser` role on `projectId`, or the OAuth scope doesn't include `cloud-platform`.

**Solution:** Grant the IAM role and re-consent with the required scope.

#### `tables` is an empty array

**Cause:** `processorId` points to a plain Document OCR processor, which does not extract tables.

**Solution:** Use a Layout Parser or Form Parser processor instead.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Mistral Document AI](./mistral-doc.md) – Alternative OCR/table extraction via Mistral
- [AWS Textract](./aws-textract.md) – Alternative OCR/table extraction via AWS
- [Azure Document Intelligence](./azure-document-intelligence.md) – Alternative OCR/table extraction via Azure

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-07-03 | Initial release. |

<!-- /SECTION: changelog -->

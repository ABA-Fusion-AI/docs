---
node_id: "azure-document-intelligence"
title: "Azure Document Intelligence"
description: "OCR, layout, and table extraction for documents using Azure AI Document Intelligence, with normalized text, table, and confidence extraction."
category: "integrations"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-07-03"
author: "Fusion Team"
tags:
  - ocr
  - document-ai
  - azure
  - document-intelligence
  - tables
  - confidence
  - no-code
related_nodes:
  - mistral-doc
  - aws-textract
  - google-document-ai
---

# Azure Document Intelligence

> **Category:** Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Extract text and tables with per-word confidence scores from documents and images using Azure AI Document Intelligence.

### Use Cases

- Extract full text and reconstructed tables from scanned PDFs and images.
- Route low-confidence extractions to human review.
- Feed structured invoice/report tables into downstream processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `endpoint` | `string` | ✅ Yes | Your Document Intelligence resource endpoint, e.g. `https://<resource>.cognitiveservices.azure.com`. |
| `apiKey` | `string` | ✅ Yes | Your Document Intelligence API key (sent as `Ocp-Apim-Subscription-Key`). |

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputType` | `enum` | ❌ No | `url` | Whether `document` is a public URL or Base64 content. |
| `document` | `string` | ✅ Yes | — | The public URL or Base64 content of the document/image. |
| `modelId` | `string` | ❌ No | `prebuilt-layout` | Document Intelligence model to use. `prebuilt-layout` extracts text and tables. |

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
| `error` | `Error` | Emitted when the request or polling fails, or times out. |

### Output Schema (`success`)

```ts
{
  text: string;                    // analyzeResult.content — the full extracted text
  tables: { rows: string[][] }[];  // one entry per detected table, as a 2D cell matrix
  confidence: number | null;       // average word confidence across all pages, 0-1
  pages: number | null;            // number of pages analyzed
  raw: unknown;                    // untouched { status, analyzeResult } response
}
```

### How it works

This node submits the document to Azure's asynchronous `analyze` endpoint, then polls the returned `Operation-Location` until the analysis finishes (up to 60 attempts, 1s apart). This can add a few seconds of latency for larger documents.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Extract text and tables from a PDF

**Configuration:**
```json
{
  "endpoint": "https://my-resource.cognitiveservices.azure.com",
  "apiKey": "{{credentials.azure.docIntelligenceKey}}",
  "inputType": "url",
  "document": "https://example.com/report.pdf",
  "modelId": "prebuilt-layout"
}
```

**Output (success):**
```json
{
  "text": "Quarterly Report\n\nRevenue increased 12%...",
  "tables": [
    { "rows": [["Quarter", "Revenue"], ["Q1", "$1.2M"], ["Q2", "$1.4M"]] }
  ],
  "confidence": 0.99,
  "pages": 3,
  "raw": { "status": "succeeded", "analyzeResult": "..." }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Azure Document Intelligence request failed (401): ...`

**Cause:** Invalid `apiKey`, or `endpoint` doesn't match the key's resource.

**Solution:** Confirm both `endpoint` and `apiKey` come from the same Azure Document Intelligence resource.

#### `Azure Document Intelligence analysis timed out while polling for a result.`

**Cause:** The document is very large, or the service is under heavy load.

**Solution:** Retry with a smaller document, or split large PDFs into smaller batches.

#### `confidence` is `null`

**Cause:** The response's pages didn't include per-word confidence (uncommon for `prebuilt-layout`, but possible for very sparse documents).

**Solution:** Treat `confidence: null` as "unavailable" — `text`/`tables` are still populated.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Mistral Document AI](./mistral-doc.md) – Alternative OCR/table extraction via Mistral
- [AWS Textract](./aws-textract.md) – Alternative OCR/table extraction via AWS
- [Google Document AI](./google-document-ai.md) – Alternative OCR/table extraction via Google Cloud

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-07-03 | Initial release. |

<!-- /SECTION: changelog -->

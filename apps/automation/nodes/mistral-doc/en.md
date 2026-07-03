---
node_id: "mistral-doc"
title: "Mistral Document AI"
description: "OCR, layout annotations, and document Q&A for PDFs and images via Mistral, with normalized text, table, and confidence extraction."
category: "integrations"
subcategory: "ai"
version: "1.1.0"
language: "en"
last_updated: "2026-07-03"
author: "Fusion Team"
tags:
  - ocr
  - document-ai
  - mistral
  - tables
  - confidence
  - no-code
related_nodes:
  - aws-textract
  - azure-document-intelligence
  - google-document-ai
---

# Mistral Document AI

> **Category:** Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Run OCR (text + table extraction with confidence scores) or ask a question about a PDF/image using Mistral's Document AI models.

### Use Cases

- Extract full text and tables from scanned invoices, contracts, or reports.
- Turn a PDF's tables into structured rows for downstream processing.
- Ask a natural-language question about the contents of a document or image.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | ✅ Yes | Your Mistral API key. |

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ❌ No | `ocr` | `ocr` (extract text/markdown/tables) or `document_qna` (chat with the document). |
| `inputType` | `enum` | ❌ No | `url` | Whether `document` is a public URL or Base64 content. |
| `document` | `string` | ✅ Yes | — | The public URL or Base64 content of the file. |

### OCR Parameters (`operation: "ocr"`)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ocrModel` | `string` | ❌ No | `mistral-ocr-latest` | Model used for OCR. |
| `includeImageBase64` | `boolean` | ❌ No | `false` | Include base64 crops of extracted images in the raw response. |
| `tableFormat` | `enum` | ❌ No | `markdown` | Format Mistral renders tables in (`markdown` or `html`) before this node parses them into `tables`. |
| `confidenceGranularity` | `enum` | ❌ No | `page` | Granularity of confidence scores requested from the model (`page` or `word`). |

### Document Q&A Parameters (`operation: "document_qna"`)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `qnaModel` | `string` | ❌ No | `pixtral-12b-2409` | Multimodal model used to answer the question. |
| `question` | `string` | ✅ Yes | — | The question to ask about the document/image. |

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
| `success` | `object` | Emitted with the extraction/answer result. |
| `error` | `Error` | Emitted when the request fails. |

### Output Schema (`success`) — `operation: "ocr"`

```ts
{
  text: string;                    // full document text (all pages' markdown joined)
  tables: { rows: string[][] }[];  // one entry per detected table, as a 2D cell matrix
  confidence: number | null;       // average page/word confidence, 0-1, or null if unavailable
  pages: number | null;            // number of pages processed
  raw: unknown;                    // untouched Mistral OCR response
}
```

### Output Schema (`success`) — `operation: "document_qna"`

```ts
{
  answer: string;        // the model's answer to `question`
  full_response: unknown; // untouched chat completion response
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Extract text and tables from a PDF

**Configuration:**
```json
{
  "apiKey": "{{credentials.mistral}}",
  "operation": "ocr",
  "inputType": "url",
  "document": "https://example.com/invoice.pdf",
  "tableFormat": "markdown"
}
```

**Output (success):**
```json
{
  "text": "Invoice #1042\n\n| Item | Qty | Price |\n...",
  "tables": [
    { "rows": [["Item", "Qty", "Price"], ["Widget", "3", "$9.00"]] }
  ],
  "confidence": 0.98,
  "pages": 1,
  "raw": { "pages": ["..."] }
}
```

### Example 2: Ask a question about a document

**Configuration:**
```json
{
  "apiKey": "{{credentials.mistral}}",
  "operation": "document_qna",
  "inputType": "url",
  "document": "https://example.com/contract.pdf",
  "question": "What is the termination notice period?"
}
```

**Output (success):**
```json
{
  "answer": "The contract requires 30 days written notice.",
  "full_response": { "choices": ["..."] }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `OCR Failed (401): ...`

**Cause:** Invalid or missing `apiKey`.

**Solution:** Verify the Mistral API key has access to the OCR endpoint.

#### `tables` is an empty array

**Cause:** The document has no detectable tables, or `tableFormat` doesn't match how the model rendered them (e.g. an `html`-formatted response parsed with the markdown table parser).

**Solution:** Confirm the source document actually contains tabular data; try switching `tableFormat`.

#### `confidence` is `null`

**Cause:** The requested `confidenceGranularity` wasn't returned for this document/model combination.

**Solution:** Treat `confidence: null` as "unavailable" rather than a failure — the `text`/`tables` extraction is unaffected.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [AWS Textract](./aws-textract.md) – Alternative OCR/table extraction via AWS
- [Azure Document Intelligence](./azure-document-intelligence.md) – Alternative OCR/table extraction via Azure
- [Google Document AI](./google-document-ai.md) – Alternative OCR/table extraction via Google Cloud

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-07-03 | Re-enabled the node; added `tableFormat`/`confidenceGranularity` and normalized `text`/`tables`/`confidence` output for `ocr`. |
| 1.0.0 | — | Initial implementation (raw passthrough). |

<!-- /SECTION: changelog -->

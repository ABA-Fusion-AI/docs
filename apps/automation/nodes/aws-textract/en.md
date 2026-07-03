---
node_id: "aws-textract"
title: "AWS Textract"
description: "Extract text, tables, forms, and expense data from documents using AWS Textract, with normalized text, table, and confidence extraction."
category: "integrations"
subcategory: "aws"
version: "1.1.0"
language: "en"
last_updated: "2026-07-03"
author: "Fusion Team"
tags:
  - ocr
  - document-ai
  - aws
  - textract
  - tables
  - confidence
  - no-code
related_nodes:
  - mistral-doc
  - azure-document-intelligence
  - google-document-ai
---

# AWS Textract

> **Category:** Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Extract text, tables, forms, and expense data from scanned documents using AWS Textract.

### Use Cases

- Extract line-by-line text from scanned forms or receipts.
- Reconstruct tables from a document into structured rows/columns.
- Pull vendor/total/line-item data out of receipts and invoices.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `region` | `string` | ❌ No | `us-east-1` | AWS region, e.g. `us-east-1`. |
| `accessKeyId` | `string` | ✅ Yes | — | AWS Access Key ID. |
| `secretAccessKey` | `string` | ✅ Yes | — | AWS Secret Access Key. |
| `sessionToken` | `string` | ❌ No | — | AWS Session Token, for temporary credentials. |

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ❌ No | `detectText` | `detectText`, `analyzeDocument`, or `analyzeExpense`. |
| `s3Bucket` / `s3Key` | `string` | ❌ No | — | S3 location of the document (alternative to `documentBytes`). |
| `documentBytes` | `string` | ❌ No | — | Base64-encoded document bytes (alternative to S3). |
| `featureTypes` | `string` | ❌ No | `TABLES,FORMS` | Comma-separated Textract features for `analyzeDocument` (`TABLES`, `FORMS`, `QUERIES`, `SIGNATURES`). Tables are only returned when `TABLES` is included. |

Provide **either** `s3Bucket`+`s3Key` **or** `documentBytes` — one of the two is required.

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
  text: string;                    // full document text (LINE blocks joined; SummaryFields for analyzeExpense)
  tables: { rows: string[][] }[];  // one entry per detected table (or per LineItemGroup for analyzeExpense)
  confidence: number | null;       // average block/field confidence, 0-1
  pages: number | null;            // not tracked by Textract responses, always null
  raw: unknown;                    // untouched Textract response
}
```

`detectText` never returns tables (`tables: []`). `analyzeDocument` returns tables only when `featureTypes` includes `TABLES`. `analyzeExpense` returns one "table" per detected line-item group, where each row is a line item's fields.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Extract text and tables from an S3 document

**Configuration:**
```json
{
  "region": "us-east-1",
  "accessKeyId": "{{credentials.aws.accessKeyId}}",
  "secretAccessKey": "{{credentials.aws.secretAccessKey}}",
  "operation": "analyzeDocument",
  "s3Bucket": "my-documents",
  "s3Key": "invoices/invoice-1042.pdf",
  "featureTypes": "TABLES,FORMS"
}
```

**Output (success):**
```json
{
  "text": "Invoice #1042\nBill To: Acme Corp\n...",
  "tables": [
    { "rows": [["Item", "Qty", "Price"], ["Widget", "3", "$9.00"]] }
  ],
  "confidence": 0.97,
  "pages": null,
  "raw": { "Blocks": ["..."] }
}
```

### Example 2: Extract expense fields from a receipt

**Configuration:**
```json
{
  "region": "us-east-1",
  "accessKeyId": "{{credentials.aws.accessKeyId}}",
  "secretAccessKey": "{{credentials.aws.secretAccessKey}}",
  "operation": "analyzeExpense",
  "documentBytes": "{{$node.upload.data.base64}}"
}
```

**Output (success):**
```json
{
  "text": "VENDOR_NAME: Coffee Shop\nTOTAL: $12.50",
  "tables": [
    { "rows": [["ITEM: Latte", "PRICE: $5.00"], ["ITEM: Muffin", "PRICE: $7.50"]] }
  ],
  "confidence": 0.95,
  "pages": null,
  "raw": { "ExpenseDocuments": ["..."] }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Either S3 bucket/key or document bytes must be provided`

**Cause:** Neither `s3Bucket`/`s3Key` nor `documentBytes` was set.

**Solution:** Provide one of the two document sources.

#### `tables` is an empty array with `analyzeDocument`

**Cause:** `featureTypes` didn't include `TABLES`.

**Solution:** Set `featureTypes` to include `TABLES` (the default already does).

#### `AWS Textract API Error: 403 ...`

**Cause:** The IAM credentials don't have `textract:*` permissions, or the region is wrong.

**Solution:** Verify the IAM policy and that `region` matches where you intend to call Textract.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Mistral Document AI](./mistral-doc.md) – Alternative OCR/table extraction via Mistral
- [Azure Document Intelligence](./azure-document-intelligence.md) – Alternative OCR/table extraction via Azure
- [Google Document AI](./google-document-ai.md) – Alternative OCR/table extraction via Google Cloud

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-07-03 | Normalized `text`/`tables`/`confidence` output for `detectText`, `analyzeDocument`, and `analyzeExpense`. |
| 1.0.0 | — | Initial implementation (raw passthrough). |

<!-- /SECTION: changelog -->

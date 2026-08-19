---
node_id: "data-platform-document-upload"
title: "Data Platform Document Upload"
description: "Upload a document, finalize it, and start automatic Data Platform ingestion."
category: "databases"
subcategory: "Data Platform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags: [data-platform, rag, ingestion, upload, embeddings]
related_nodes: [data-platform-dataset, data-platform-document-wait, data-platform-document]
---

<!-- SECTION: overview -->
# Data Platform Document Upload

Uploads a file into a dataset and starts the complete ingestion pipeline in `auto` mode. The node prepares a presigned target, transfers the bytes, calculates SHA-256, and finalizes the upload. Connect it to **Data Platform Document Wait** before retrieval or chunk operations.
<!-- /SECTION: overview -->

## Upload lifecycle

1. `POST /documents/{datasetId}/prepare-upload`
2. `PUT` the bytes to the returned presigned storage URL
3. `POST /documents/{datasetId}/finalize` with the upload identity, checksum, automatic mode, and selected AI configuration

Finalization does not upload the bytes; the presigned `PUT` is required.

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `datasetId` | string | — | Required destination dataset UUID. |
| `file` | any | incoming input | File content or file object. See supported shapes below. |
| `fileUrl` | string | — | HTTP(S) URL to download when `file` does not contain a URL. |
| `filename` | string | inferred | Explicit filename. It must have a supported extension. |
| `contentType` | string | inferred | MIME type. It must agree with the filename extension. |
| `embeddingProvider` | enum | `qwen` | `qwen` or `openai`. Provider selects the embedding model. |
| `summaryProvider` | enum | `ollama` | `ollama` or `openai`. |
| `summaryModel` | enum | `qwen3-vl:30b-a3b-instruct-q8_0` | Concrete summarization model. |
| `timeoutMs` | number | `60000` | Timeout for each network stage, 1–300 seconds. |

The ingestion mode is intentionally not configurable: Automation always sends `mode: auto`.

## Accepted file shapes

The node accepts:

- Plain text: `"document content"`
- A base64 object: `{ "base64": "...", "fileName": "paper.pdf", "contentType": "application/pdf" }`
- Automation-style base64: `{ "contentBase64": "...", "filename": "notes.md", "mimeType": "text/markdown" }`
- A data URL: `data:application/pdf;base64,...`
- `Buffer`, `Uint8Array`, `ArrayBuffer`, or serialized Node Buffer
- A URL string supplied through `fileUrl`, `url`, or `downloadUrl`
- A nested `file`, `pdf`, or `document` object

For detected PDF bytes, the node adds `.pdf` when upstream metadata supplies a missing or unsupported suffix.

Supported Data Platform extensions are `csv`, `docx`, `html`, `json`, `jsonl`, `md`, `pdf`, `pptx`, `txt`, `xlsx`, `png`, `jpg`, `tiff`, `bmp`, and `webp`.

## Model compatibility

| Embedding choice | Effective model |
|---|---|
| `openai` | `text-embedding-3-large` |
| `qwen` | `Qwen3-VL-Embedding-8B` |

| Summary provider | Valid model |
|---|---|
| `openai` | `gpt-4.1-mini` |
| `ollama` | Any listed local Qwen summary model |

An incompatible summary provider/model combination fails before upload.

## Output

The success value includes the Data Platform finalization response plus:

```json
{
  "datasetId": "dataset-uuid",
  "data": { "id": "document-uuid" },
  "upload": {
    "documentId": "prepared-upload-uuid",
    "filename": "document.md",
    "fileSize": 1234,
    "contentType": "text/markdown",
    "sha256": "...",
    "mode": "auto",
    "embeddingProvider": "qwen",
    "summaryProvider": "ollama",
    "summaryModel": "qwen3-vl:30b-a3b-instruct-q8_0"
  }
}
```

Use the finalized document ID downstream:

```text
{{ outputs.uploadKnowledgeDocument.success.data.id }}
```

## Example workflow

Import `apps/automation/workflows/fusion-work-flow/data-platform/data-platform-ingestion-retrieval-example.json`. It demonstrates dataset ensure, upload, automatic waiting, hybrid retrieval, chunk listing, and error handling.

```fusion-workflow
src: example.workflow.json
title: Upload a document and wait for ingestion
```

## Troubleshooting

- **Unsupported file type:** Provide a supported filename extension and matching `contentType`.
- **Presigned upload failed:** Verify object storage is reachable from the Automation engine.
- **Unresolved expression:** Use labels without spaces and dot syntax, such as `outputs.ensureDataset.success.id`.
- **Model unavailable:** Start the selected local service or configure the required OpenAI credentials in Data Platform.

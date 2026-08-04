---
node_id: "pdf-extractor"
title: "PDF Extractor"
description: "Extract text, page content, counts, and metadata from PDF documents."
category: "storage-files"
subcategory: "files-documents"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [pdf, extraction, text, metadata]
related_nodes: [html-to-pdf, ilovepdf]
---

<!-- SECTION: overview -->
# PDF Extractor

> **Category:** Storage & Files&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Load a PDF from a URL or Base64 input and extract its text, page sections, metadata, or page count.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `operation` | enum | No | `extractText` | Extraction or inspection operation. |
| `pdfUrl` | string | Conditional | — | URL of the PDF document. |
| `pdfBase64` | string | Conditional | — | Base64 PDF content used instead of a URL. |
| `pageRange` | string | No | — | Pages such as `1-3` or `2,4,6`. |
| `outputFormat` | enum | No | `text` | Text, JSON, or Markdown output. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Success:** Extracted text/sections, metadata, or page count according to the operation.
- **Error:** Download, Base64, or PDF parsing error.

`extractImages` reports that image extraction requires a specialized library; it does not return embedded images.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Extract PDF text and log it
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Only process trusted URLs and enforce upstream size limits for remote or Base64 documents.
<!-- /SECTION: security -->

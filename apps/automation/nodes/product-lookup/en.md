---
node_id: "product-lookup"
title: "Product Lookup"
description: "Look up books, movies, music, games, and general products using ISBN, EAN, UPC, and other identifiers."
category: "Business & Commerce"
subcategory: "Logistics & Supply Chain"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - product
  - catalog
  - isbn
  - ean
  - upc
  - barcode
  - inventory
  - logistics
related_nodes:
  - amazon-product-search
  - barcode-generator
  - function
  - log
---

<!-- SECTION: header -->
# Product Lookup

> **Category:** Business & Commerce | **Subcategory:** Logistics & Supply Chain | **Type:** Action Node

Look up product and media information using ISBN, EAN, UPC, title, or barcode values. The node can query sources such as Open Library, Google Books, OMDb, UPCItemDB, and MusicBrainz, depending on the selected operation.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Product Lookup** node provides a single interface for common product-identification workflows. It supports book identifiers, movie titles, music barcodes, retail barcodes, batch lookups, identifier validation, and ISBN conversion.

### Supported Operations

- **Lookup Product:** Automatically identify and query a product from a general code
- **Lookup ISBN:** Find book metadata from an ISBN-10 or ISBN-13
- **Lookup Google Books:** Query Google Books using an ISBN
- **Lookup Movie:** Search OMDb by movie title
- **Lookup UPC/EAN:** Query retail product information by UPC or EAN
- **Lookup Music:** Find music metadata using a barcode
- **Batch Lookup:** Look up multiple product codes in one operation
- **Validate Checksum:** Validate the checksum of a supported identifier
- **Convert ISBN-10 to ISBN-13:** Convert a valid ISBN-10
- **Detect Barcode Type:** Identify whether a code is ISBN, UPC, EAN, or another supported type

### Use Cases

- Enrich inventory or catalog records with product metadata
- Validate product identifiers before importing or shipping goods
- Identify books, movies, music, and retail products from scanned codes
- Match supplier or warehouse data to external catalog records
- Build product intake, fulfillment, and logistics workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Common Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | — | Lookup or validation operation to perform |
| `code` | `string` | Conditional | — | General product code, ISBN, UPC, or EAN used by several operations |
| `upcCode` | `string` | Conditional | — | UPC or EAN used by `Lookup UPC/EAN` |
| `movieTitle` | `string` | Conditional | — | Movie title used by `Lookup Movie` |
| `musicBarcode` | `string` | Conditional | — | Music barcode used by `Lookup Music` |
| `codes` | `string` | Conditional | — | JSON array of codes used by `Batch Lookup` |
| `checksumCode` | `string` | Conditional | — | Identifier used by `Validate Checksum` |
| `isbn10` | `string` | Conditional | — | ISBN-10 used by `Convert ISBN-10 to ISBN-13` |
| `omdbApiKey` | `string` | Conditional | — | OMDb API key required by `Lookup Movie` when the movie source requires authentication |

Required fields depend on the selected operation. The example workflow demonstrates each supported parameter pattern.

### Operation Reference

| Operation | Required value | Typical source |
|-----------|----------------|----------------|
| `Lookup Product` | `code` | Automatic identifier detection and supported product sources |
| `Lookup ISBN` | `code` | Open Library or book metadata source |
| `Lookup Google Books` | `code` | Google Books |
| `Lookup Movie` | `movieTitle`, `omdbApiKey` when required | OMDb |
| `Lookup UPC/EAN` | `upcCode` | UPCItemDB or retail product source |
| `Lookup Music` | `musicBarcode` | MusicBrainz or music metadata source |
| `Batch Lookup` | `codes` | Multiple supported identifiers |
| `Validate Checksum` | `checksumCode` | Local identifier validation |
| `Convert ISBN-10 to ISBN-13` | `isbn10` | Local ISBN conversion |
| `Detect Barcode Type` | `code` | Local code detection |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional dynamic product code, title, or operation data. Use an object when passing named fields such as `code`, `movieTitle`, or `upcCode`. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Product metadata, validation result, converted identifier, or batch results depending on the selected operation |

Example book lookup result:

```json
{
  "operation": "Lookup ISBN",
  "code": "9780140328721",
  "title": "Fantastic Mr Fox",
  "authors": ["Roald Dahl"],
  "type": "book",
  "source": "Open Library"
}
```

Example identifier result:

```json
{
  "operation": "Convert ISBN-10 to ISBN-13",
  "isbn10": "0306406152",
  "isbn13": "9780306406157",
  "valid": true
}
```

The exact metadata fields vary by operation and upstream source.

### Error Output

Invalid identifiers, missing operation-specific values, unavailable APIs, authentication failures, and malformed batch input are routed to `error`.

```json
{
  "success": false,
  "error": "A valid product code is required",
  "operation": "Lookup Product"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Automatic Product Lookup

```json
{
  "operation": "Lookup Product",
  "code": "9780140328721"
}
```

### Book Lookup

```json
{
  "operation": "Lookup ISBN",
  "code": "9780140328721"
}
```

### Movie Lookup

```json
{
  "operation": "Lookup Movie",
  "omdbApiKey": "{{secrets.omdbApiKey}}",
  "movieTitle": "Inception"
}
```

### UPC/EAN Lookup

```json
{
  "operation": "Lookup UPC/EAN",
  "upcCode": "190198764690"
}
```

### Batch Lookup

The `codes` value is a JSON array represented as a string:

```json
{
  "operation": "Batch Lookup",
  "codes": "[\"9780140328721\",\"3017620422003\"]"
}
```

### Dynamic Input

Pass a code or named product fields from a previous node through `input`:

```json
{
  "code": "9780306406157"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Look up and validate products using multiple identifiers
```

### Common Patterns

- **Catalog enrichment:** Product code → Product Lookup → Update record
- **Inventory intake:** Barcode scanner → Product Lookup → Inventory system
- **Batch import:** File parser → Product Lookup → Filter → Catalog database
- **Identifier validation:** Form/Webhook → Product Lookup → Conditional branch
- **Metadata inspection:** Product Lookup → Log or Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Operation-specific value is missing

**Cause:** The selected operation requires a field that was not provided, such as `movieTitle`, `upcCode`, or `isbn10`.

**Solution:** Review the operation reference and provide the required parameter.

### Product not found

**Cause:** The identifier is invalid, unsupported, or not present in the selected external catalog.

**Solution:** Verify the code, remove unintended spaces, select a more specific operation, or try another supported source.

### Movie lookup authentication failed

**Cause:** The OMDb API key is missing, invalid, or has reached its quota.

**Solution:** Store a valid key in the workflow secret system and reference it with `omdbApiKey`.

### Batch lookup failed

**Cause:** `codes` is not valid JSON or does not contain an array of product codes.

**Solution:** Pass a JSON array string, for example `\"[\\\"9780140328721\\\",\\\"3017620422003\\\"]\"`.

### Incomplete metadata

**Cause:** External sources do not provide the same fields for every product.

**Solution:** Treat optional fields as nullable and use the returned `source` and `operation` values when mapping results downstream.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Amazon Product Search](./amazon-product-search.md) — Search Amazon product listings
- [Barcode Generator](./barcode-generator.md) — Generate product barcodes
- [Function](./function.md) — Transform lookup results or prepare dynamic inputs
- [Log](./log.md) — Inspect product lookup output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial documentation |

<!-- /SECTION: changelog -->

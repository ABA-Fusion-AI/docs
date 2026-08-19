---
node_id: "sec-edgar"
title: "SEC EDGAR"
description: "Get official US financial filings (10-K, 10-Q) for public companies from SEC EDGAR database."
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - sec
  - edgar
  - finance
  - accounting
  - filings
  - stocks
  - 10-k
  - 10-q
related_nodes:
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# SEC EDGAR

> **Category:** Business & Commerce | **Subcategory:** Finance & Accounting | **Type:** Action Node

Retrieve official financial filing information for publicly traded US companies from the SEC EDGAR database using a company’s Central Index Key (CIK).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **SEC EDGAR** node queries the US Securities and Exchange Commission’s EDGAR database for a public company identified by its CIK. It is designed for workflows that need official filing data such as annual reports (`10-K`), quarterly reports (`10-Q`), and other company submissions.

### Key Features

- **CIK Lookup:** Retrieve filings using a company’s SEC Central Index Key
- **Official Source:** Read filing information from the SEC EDGAR database
- **Financial Filing Data:** Access filing metadata and available filing records
- **Workflow Ready:** Pass results to Log, Function, storage, reporting, or analysis nodes
- **Error Routing:** Send request and lookup failures to the error output
- **Progress Visibility:** Supports running status while the SEC request is being processed

### Typical Use Cases

- Monitor new annual and quarterly filings
- Collect public-company filing metadata for analysis
- Build financial research and due-diligence workflows
- Feed SEC filing data into reports or data warehouses
- Compare filings across multiple public companies
- Trigger downstream processing when a filing lookup succeeds

### Supported Filing Types

The SEC EDGAR database contains many submission types. Common financial filing types include:

| Filing type | Description |
|-------------|-------------|
| `10-K` | Annual report |
| `10-Q` | Quarterly report |
| `8-K` | Current report for significant events |
| `20-F` | Annual report for certain foreign private issuers |
| `DEF 14A` | Proxy statement |

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `cik` | `string` | Yes | — | SEC Central Index Key for the company to query. Use the SEC’s 10-digit zero-padded CIK format. |

### CIK Format

CIKs are numeric identifiers assigned by the SEC. Provide them as strings and preserve leading zeroes.

```text
0000320193
```

The example workflow uses:

| Company | CIK |
|---------|-----|
| Apple Inc. | `0000320193` |
| Microsoft Corporation | `0000789019` |

Do not replace a CIK with a stock ticker unless the node or a preceding lookup step has converted the ticker to the company’s SEC CIK.

### Dynamic Configuration

The `cik` value can be supplied from workflow data using the node’s input connection when dynamic expressions are supported by the workflow runtime.

Example incoming data:

```json
{
  "cik": "0000320193"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `any` | Optional incoming data that can provide a dynamic CIK or support a preceding lookup step |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | SEC EDGAR filing response for the requested company |
| `error` | `object` | Error details when the CIK is invalid, the company cannot be found, or the SEC request fails |

### Success Output

The successful response contains SEC EDGAR data for the requested CIK. The exact fields depend on the filing data returned by the service and may include company identity, filing accession numbers, filing dates, form types, report periods, and document links.

A representative filing record may look like:

```json
{
  "form": "10-K",
  "filingDate": "2024-11-01",
  "reportDate": "2024-09-28",
  "accessionNumber": "0000320193-24-000123",
  "primaryDocument": "aapl-20240928.htm"
}
```

Treat the result as SEC filing data rather than as a guaranteed fixed schema. Use a Function or Restructure node when downstream steps require a specific shape.

### Error Output Example

```json
{
  "success": false,
  "error": "SEC EDGAR request failed",
  "details": "Company not found for the supplied CIK"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Retrieve Apple Filings

Use Apple’s SEC CIK to retrieve official filing data.

**Configuration:**

```json
{
  "cik": "0000320193"
}
```

Connect the success output to a Log node to inspect the returned filings.

---

### Example 2: Retrieve Microsoft Filings

Use Microsoft’s SEC CIK in a separate workflow branch.

**Configuration:**

```json
{
  "cik": "0000789019"
}
```

The example workflow included with this node demonstrates both the Apple and Microsoft configurations.

---

### Example 3: Use a Dynamic CIK

A previous step can provide the company identifier.

**Input:**

```json
{
  "cik": "0000320193"
}
```

Pass the incoming CIK to SEC EDGAR, then send the result to a Function node to select only the filing types or dates needed by the workflow.

---

### Example 4: Prepare Filing Data for Storage

Use SEC EDGAR followed by a transformation step before writing to a database.

**Suggested fields:**

```json
{
  "companyCik": "0000320193",
  "form": "10-K",
  "filingDate": "2024-11-01",
  "accessionNumber": "0000320193-24-000123"
}
```

The exact source field names should be confirmed from the returned response before configuring the downstream mapping.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve SEC EDGAR filings and inspect the response
```

### Common Patterns

- **Single Company Lookup:** Manual Trigger → SEC EDGAR → Log
- **Data Preparation:** SEC EDGAR → Function → Storage
- **Response Shaping:** SEC EDGAR → Restructure → Database or Report
- **Company Comparison:** Multiple SEC EDGAR branches → Merge → Function
- **Filing Monitoring:** Scheduled Trigger → SEC EDGAR → Filter → Notification

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Company not found

**Cause:** The supplied CIK is invalid, incomplete, or does not identify an SEC-registered company.

**Solution:** Verify the company’s CIK and use the 10-digit zero-padded format.

#### Leading zeroes were removed

**Cause:** The CIK was entered as a number instead of a string.

**Solution:** Enclose the CIK in quotes, for example `"0000320193"`.

#### SEC request is rejected or unavailable

**Cause:** The SEC service may reject malformed requests, enforce access policies, or be temporarily unavailable.

**Solution:** Verify the CIK, retry after a short interval, and ensure the workflow environment can reach SEC EDGAR.

#### No expected filing records appear

**Cause:** The response may contain a different filing structure than expected, or the company may not have the filing type being searched for.

**Solution:** Log the complete success response first, then inspect the available form names, dates, and accession numbers before filtering.

#### Downstream mapping fails

**Cause:** Filing responses can contain nested lists and service-specific field names.

**Solution:** Inspect the response with Log and use Function or Restructure to create the exact schema required downstream.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Invalid CIK` | CIK is malformed or not zero-padded | Use a valid 10-digit CIK string |
| `Company not found` | No SEC company record matches the CIK | Verify the company identifier |
| `SEC EDGAR request failed` | Network, service, or request failure | Check connectivity and retry safely |
| `Rate limit or access policy` | SEC request frequency or access requirements were exceeded | Reduce request frequency and follow SEC access requirements |
| `Unexpected response format` | Returned data differs from the expected structure | Log and inspect the complete response |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](../function/en.md) — Filter or transform filing data with custom logic
- [Restructure](../restructure/en.md) — Map filing responses into a downstream schema
- [Merge](../merge/en.md) — Combine results from multiple company lookups
- Log — Inspect SEC EDGAR responses during workflow development

<!-- /SECTION: related -->

---

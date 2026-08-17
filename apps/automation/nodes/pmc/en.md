---
node_id: "pmc"
title: "PMC (PubMed Central)"
description: "Search and retrieve full-text articles from PubMed Central."
category: "Research / Literature"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:

- pmc
- pubmed-central
- ncbi
- eutils
- research
- literature
- articles
- abstracts
- biomedical
- academic-search

related_nodes:
- function
- if
- http-request

---

# PMC (PubMed Central)

> **Category:** research-nodes | **Type:** Action Node

Search and retrieve **full-text article metadata and abstracts** from **PubMed Central (PMC)** via the NCBI E-utilities API.

The **PMC** node supports two operations — searching PMC by keyword query, and fetching a single article by its PMC ID — with built-in rate limiting and automatic retry on NCBI's 429 responses.

### Supported Features

- Keyword search across PMC (`esearch` + `esummary`)
- Paginated search results
- Single-article lookup by PMC ID (`esummary`)
- Automatic abstract extraction via `efetch`, batched in groups of 10
- Built-in NCBI rate limiting (3 req/sec without API key, 10 req/sec with API key)
- Automatic retry with exponential backoff on HTTP 429 (rate-limit) responses
- Optional NCBI API key support for higher rate limits
- Accepts query/PMC ID from either node configuration or upstream workflow data
- Automatic `PMC` prefix stripping from provided IDs

### Use Cases

- Search biomedical literature by keyword for a research workflow
- Retrieve full article metadata (title, authors, journal, DOI, abstract) for a known PMC ID
- Build a literature review or citation-gathering pipeline
- Feed article abstracts into an LLM node for summarization
- Monitor new publications matching a search query
- Enrich a gene/protein/interaction workflow (e.g. alongside a genomics node) with supporting literature

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ❌ No | `""` | NCBI API key. Increases rate limit from 3 to 10 requests/second. |
| `operation` | `enum` | ❌ No | `"search"` | Operation to perform: `search` or `getById`. |
| `query` | `string` | ✅ Yes (for `search`, unless provided via input data) | — | Search query text. |
| `page` | `number` | ❌ No | `1` | Page number for search results (1-indexed). |
| `pageSize` | `number` | ❌ No | `20` | Number of results per page. |
| `pmcId` | `string` | ✅ Yes (for `getById`, unless provided via input data) | — | PMC article ID. The `PMC` prefix is stripped automatically if present. |

---

## Input Data Fallback

Both operations can source their required identifier from **incoming workflow data** if the corresponding config field is left empty:

| Operation | Config Field | Input Data Fallback |
| --------- | ------------- | -------------------- |
| `search` | `query` | If `query` is empty, uses the input data directly (coerced to string). |
| `getById` | `pmcId` | If `pmcId` is empty, uses input data: a plain string, or an object's `pmcId` field, or an object's `id` field. In all cases, a leading `PMC` prefix is stripped. |

---

## Rate Limiting & Retry

The node enforces NCBI's E-utilities rate limits internally:

| Condition | Minimum Delay Between Requests | Effective Rate |
| --------- | ------------------------------- | --------------- |
| No `apiKey` | 334 ms | 3 requests/second |
| With `apiKey` | 100 ms | 10 requests/second |

If NCBI responds with **HTTP 429**, the node retries automatically with **exponential backoff**:

```text
delay = 1000ms * 2^attempt
```

Up to **3 retries** (4 total attempts) before giving up. Non-429 errors are thrown immediately without retry.

---

## Search Flow

The `search` operation performs a 3-step process:

1. **`esearch.fcgi`** — resolves the query into a list of PMC IDs (`retmax` = `pageSize`, `retstart` = `(page - 1) * pageSize`), and returns the total match count.
2. **`esummary.fcgi`** — fetches metadata (title, authors, journal, publication date, DOI) for the returned IDs.
3. **`efetch.fcgi`** (via abstract batching) — fetches and parses abstracts for the same IDs, in batches of 10.

If the search returns zero IDs, the node short-circuits and returns an empty `results` array with the total count and pagination info.

---

## Abstract Extraction

Abstracts are fetched via `efetch.fcgi` in **batches of 10 PMC IDs**, using a regex-based extraction that looks for text following `ABSTRACT` up to the next section heading (`INTRODUCTION`, `BACKGROUND`, `METHODS`, `RESULTS`, `CONCLUSION`) or a copyright/DOI/PMID/PMCID marker.

The extracted abstract text is:
- Trimmed and de-indented line by line
- Joined into a single string
- Truncated to **2000 characters**

If extraction fails for a batch (network error, no match), the batch is skipped silently (logged to console) and affected articles fall back to the `abstract` field from the summary data, if present, or an empty string.

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `query` (search) or `pmcId`/`id` (getById) when the corresponding config field is empty.

### Outputs — `search`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `results` | `array` | List of article objects (see below). |
| `total` | `number` | Total number of matching articles in PMC. |
| `page` | `number` | Current page number. |
| `pageSize` | `number` | Results per page. |
| `totalPages` | `number` | Total number of pages (`ceil(total / pageSize)`). |

### Outputs — `getById`

A single article object, or `null` if the ID was not found or returned an error from NCBI.

### Article Object Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | Numeric PMC ID. |
| `title` | `string` | Article title, or `"No title"` if missing. |
| `authors` | `string[]` | List of author names. |
| `abstract` | `string` | Extracted abstract text (up to 2000 chars). |
| `journal` | `string` | Journal/source name. |
| `year` | `string` | Publication year, extracted from `pubdate`. |
| `doi` | `string` | DOI / e-location ID, when available. |
| `url` | `string` | PMC article page URL. |
| `pdfUrl` | `string` | PMC PDF URL. |
| `publicationDate` | `string` | Raw NCBI publication date string. |
| `metadata.pmcId` | `string` | Numeric PMC ID. |
| `metadata.pmcIdFormatted` | `string` | PMC ID formatted with `PMC` prefix. |

---

## Output Example

### `search`

```json
{
  "results": [
    {
      "id": "10123456",
      "title": "Machine learning approaches in plant genomics",
      "authors": ["Doe J", "Smith A"],
      "abstract": "This review examines recent applications of machine learning...",
      "journal": "Plant Biology Reports",
      "year": "2026",
      "doi": "10.1234/pbr.2026.001",
      "url": "https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10123456",
      "pdfUrl": "https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10123456/pdf/",
      "publicationDate": "2026 Feb",
      "metadata": {
        "pmcId": "10123456",
        "pmcIdFormatted": "PMC10123456"
      }
    }
  ],
  "total": 1842,
  "page": 1,
  "pageSize": 20,
  "totalPages": 93
}
```

### `getById`

```json
{
  "id": "10123456",
  "title": "Machine learning approaches in plant genomics",
  "authors": ["Doe J", "Smith A"],
  "abstract": "This review examines recent applications of machine learning...",
  "journal": "Plant Biology Reports",
  "year": "2026",
  "doi": "10.1234/pbr.2026.001",
  "url": "https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10123456",
  "pdfUrl": "https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10123456/pdf/",
  "publicationDate": "2026 Feb",
  "metadata": {
    "pmcId": "10123456",
    "pmcIdFormatted": "PMC10123456"
  }
}
```

---

## Configuration Examples

### Default Search

```json
{
  "operation": "search",
  "query": "CRISPR gene editing rice"
}
```

### Paginated Search

```json
{
  "operation": "search",
  "query": "CRISPR gene editing rice",
  "page": 2,
  "pageSize": 10
}
```

### Search with API Key

```json
{
  "operation": "search",
  "query": "CRISPR gene editing rice",
  "apiKey": "your-ncbi-api-key"
}
```

### Get Article by PMC ID

```json
{
  "operation": "getById",
  "pmcId": "PMC10123456"
}
```

### Get Article by Numeric ID (prefix optional)

```json
{
  "operation": "getById",
  "pmcId": "10123456"
}
```

---

## Workflow Integration

### Sample Workflow: Search → Function

```json
{
  "nodes": [
    {
      "id": "pmc-search",
      "type": "pmc",
      "config": {
        "operation": "search",
        "query": "plant genomics machine learning"
      }
    },
    {
      "id": "process-results",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Search → Get By ID (drill-down)

```json
{
  "nodes": [
    {
      "id": "pmc-search",
      "type": "pmc",
      "config": {
        "operation": "search",
        "query": "CRISPR rice"
      }
    },
    {
      "id": "pmc-get-detail",
      "type": "pmc",
      "config": {
        "operation": "getById"
      }
    }
  ]
}
```

The second node relies on the [input data fallback](#input-data-fallback) — it reads `pmcId`/`id` from the upstream result rather than the config field.

### Sample Workflow: PMC → LLM Summarization

```json
{
  "nodes": [
    {
      "id": "pmc-search",
      "type": "pmc",
      "config": {
        "operation": "search",
        "query": "vector database RAG systems",
        "pageSize": 5
      }
    },
    {
      "id": "summarize-abstracts",
      "type": "llm"
    }
  ]
}
```

### Common Patterns

- Schedule → PMC (`search`) → Function → Database — literature monitoring
- PMC (`search`) → PMC (`getById`) — search-then-drill-down
- PMC (`search`) → If (filter by year/journal) → Notification
- PMC → LLM — abstract summarization or literature synthesis

---

## Error Handling

All errors are wrapped with the operation name:

```text
PMC <operation> failed: <underlying error message>
```

### Missing Query

```text
Search query is required
```

Raised when `operation` is `search` and both `query` and the input data are empty.

### Missing PMC ID

```text
PMC ID is required for getById operation
```

Raised when `operation` is `getById` and neither `pmcId` nor a usable ID from the input data is available.

### Unknown Operation

```text
Unknown operation: <operation>
```

### Rate Limit Exceeded

```text
NCBI rate limit exceeded (429). Maximum retries (3) reached. Please wait before retrying or use an API key for higher rate limits.
```

Raised only after all retry attempts are exhausted.

### Generic NCBI Request Failure

```text
NCBI request failed with status <status>
```

Raised for any non-OK, non-429 HTTP response from NCBI.

---

## Troubleshooting

### "Search query is required"

**Cause**

`query` is empty and no usable input data was provided for a `search` operation.

**Solution**

Set `query` in the node configuration, or ensure the upstream node passes a string as input data.

---

### "PMC ID is required for getById operation"

**Cause**

`pmcId` is empty and the input data did not contain a string, `pmcId` field, or `id` field.

**Solution**

Set `pmcId` explicitly, or confirm the upstream node's output includes an `id` or `pmcId` field.

---

### "NCBI rate limit exceeded (429). Maximum retries (3) reached."

**Cause**

Sustained request volume exceeded NCBI's rate limit even after 3 exponential-backoff retries — common when running many PMC nodes concurrently without an API key.

**Solution**

Provide an `apiKey` to raise the limit from 3 to 10 requests/second, or reduce concurrent PMC calls in the workflow.

---

### "NCBI request failed with status <status>"

**Cause**

NCBI returned a non-OK, non-429 status — e.g. a malformed query or a temporary NCBI outage.

**Solution**

Verify the `query` or `pmcId` value, and retry later if the issue appears to be on NCBI's side.

---

### Empty or Missing Abstracts

**Cause**

The regex-based abstract extraction in `getAbstractsBatch` did not find a matching `ABSTRACT ... <next section>` pattern in the `efetch` plain-text response — this happens for articles with non-standard formatting, or when the batch request itself failed (logged to console, not thrown).

**Solution**

Treat `abstract` as best-effort; for guaranteed content, follow the returned `url` or `pdfUrl` to the full article.

---

### Search Returns Empty `results` with a Non-Zero `total`

**Cause**

This should not normally occur, but a mismatch between `esearch` and `esummary` results (e.g. an article removed between the two calls) can cause an entry to be skipped, since results are only added `if (article && !article.error)`.

**Solution**

Treat this as expected occasional data drift from NCBI; retry the search if the result count looks inconsistent.

---

## Security

The node performs outbound HTTP requests to NCBI's public E-utilities API (`eutils.ncbi.nlm.nih.gov`).

If `apiKey` is provided, it is sent as a query parameter (`api_key=...`) on every request, as required by the NCBI E-utilities API.

No other authentication or credential handling is performed.

---

## Notes

The node returns parsed and partially normalized PMC data rather than raw NCBI XML/JSON.

The node does not:

- Retrieve full article body text (only metadata and abstract)
- Download PDFs (only returns a `pdfUrl` link)
- Guarantee abstract extraction accuracy (regex-based, best-effort)
- Cache results between calls
- Deduplicate results across pages
- Validate `pmcId` format beyond stripping a `PMC` prefix

It is intended to provide fast, rate-limit-safe search and lookup access to PMC metadata and abstracts for downstream research and literature workflows.

---

## Related

- [Function](./function.md) – Transform, filter, or format article results
- [If](./if.md) – Route workflows based on search results or article metadata
- [HTTP Request](./http-request.md) – Make additional custom NCBI E-utilities calls
- [LLM](./llm.md) – Summarize or synthesize retrieved abstracts

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-12 | Initial release |
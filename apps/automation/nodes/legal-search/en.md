---
node_id: "legal-search"
title: "Legal Search"
description: "Search for Moroccan legal texts using Adala.justice.gov.ma portal."
category: "Legal / Government Data"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- legal
- morocco
- maroc
- adala
- justice
- law
- government
- legal-texts
- legislation

related_nodes:
- function
- if
- http-request

---

# Legal Search

> **Category:** legal-nodes | **Type:** Action Node

Search **Moroccan legal texts** — laws, decrees, and other official legislation — via the **Adala** portal (`adala.justice.gov.ma`), Morocco's Ministry of Justice legal documentation platform.

The **Legal Search** node queries Adala's internal Next.js data endpoint (not a documented public API) with a keyword and optional theme/law-type filters, and parses the response into a structured list of legal documents, available themes, and law types.

### Supported Features

- Keyword search across Moroccan legal texts
- Optional filtering by theme and law type
- Result count limiting (`maxResults`)
- Structured document metadata extraction (name, type, language, PDF link, law number, keywords, date)
- Extraction of available themes and law types with document counts, for building filter UIs
- Accepts the search keyword from either node configuration or upstream workflow data
- Graceful degradation: HTTP/network failures return `{ success: false, error }` instead of throwing, while unexpected internal errors still throw

---

## ⚠️ Important: Unofficial Internal Endpoint

This node calls Adala's **internal Next.js data-fetching endpoint** (`/_next/data/<build-id>/fr/search.json`), not a published public API. The URL embeds a **build ID** (`THP5ZL1eNCinRAZ1hWfN0` at the time of writing) that changes whenever the Adala website is redeployed. If Adala redeploys, this hardcoded build ID becomes stale and the endpoint will return a 404 or unexpected response, requiring the node's implementation to be updated with the new build ID.

### Use Cases

- Search Moroccan legislation by keyword for legal research
- Build a legal-document lookup tool for a compliance or legal-tech workflow
- Retrieve official PDF links for specific laws or decrees
- Populate a theme/law-type filter UI using the extracted `themes` and `law_types` lists
- Monitor for newly published legal texts matching a recurring keyword search

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `keyword` | `string` | ✅ Yes (unless provided via input data) | — | Search keyword/term. |
| `theme` | `string` | ❌ No | `""` | Filter by theme (Adala theme key). |
| `lawType` | `string` | ❌ No | `""` | Filter by law type (Adala law-type key). |
| `maxResults` | `number` | ❌ No | `10` | Maximum number of documents to include in the parsed result (client-side slicing, not a server-side parameter). |

---

## Input Data Fallback

If `keyword` is left empty in the configuration, the node falls back to the **incoming workflow data**: a string is used directly, and any other data type is coerced with `String(data)`.

---

## How It Works

1. Builds a request to Adala's internal search JSON endpoint with `term`, `themes`, `type`, and several other fixed/empty query parameters (`resources`, `number`, date range fields) always included, even when unused.
2. Sends the request with browser-like headers (`User-Agent`, `Accept`, `Accept-Language`, `Accept-Encoding`, `Referer`) to mimic a real browser request to the Adala site.
3. On a non-OK HTTP response, returns `{ success: false, error, httpCode }` **without throwing**.
4. On success, parses the nested `pageProps.searchResult` structure from the response, extracting up to `maxResults` documents, plus the full `themesResult` and `lawsResult` lists (not limited by `maxResults`).
5. Wraps everything into a `search_summary` object and returns it alongside the original keyword and total count.

---

## Result Parsing Details

The parser expects the response shape produced by Adala's Next.js `search.json` page-data endpoint:

- `data.pageProps.searchResult.data` — array of matched documents (sliced to `maxResults`).
- `data.pageProps.searchResult.total` — total match count across the full result set (not just the sliced page).
- `data.pageProps.themesResult` — array of available themes with counts.
- `data.pageProps.lawsResult` — array of available law types with counts.

If `data.pageProps.searchResult` is missing or the expected arrays aren't present, the parser returns an **empty but well-formed result** (`documents: []`, `themes: []`, `law_types: []`, `total_found: 0`) rather than throwing.

Each document is mapped with safe fallbacks (`|| ""`) for every field, so missing metadata results in empty strings rather than `undefined`.

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `keyword` when the config field is empty.

### Outputs — Success

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | `true` on a successful search. |
| `keyword` | `string` | The search keyword that was used. |
| `data` | `object` | Parsed results — `documents`, `themes`, `law_types`, `total_found`, `search_summary`. |
| `source` | `string` | Always `"Adala.justice.gov.ma"`. |
| `total_found` | `number` | Total number of matching documents reported by Adala (may exceed `documents.length` if `maxResults` limited the returned page). |

### Outputs — Failure (HTTP/Network Error)

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | `false`. |
| `error` | `string` | Error message (HTTP status text or network error message). |
| `keyword` | `string` | The search keyword that was attempted. |

This shape is returned directly (not thrown) for HTTP and network-level failures — see [Error Handling](#error-handling).

### Document Object Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `name` | `string` | Document title. |
| `type` | `string` | Document type. |
| `language` | `string` | Document language. |
| `pdf_url` | `string` | Constructed PDF download URL (`https://adala.justice.gov.ma/api/<path>`), or `""` if unavailable. |
| `object` | `string` | Subject/object of the legal text. |
| `law_type` | `string` | Law type name. |
| `theme` | `string` | Theme name. |
| `resource` | `string` | Resource/publication name. |
| `law_number` | `string` | Official law number. |
| `keywords` | `string` | Associated keywords. |
| `gregorian_date` | `string` | Publication date (Gregorian calendar). |

### Theme / Law Type Object Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | Theme/law-type key. |
| `name` | `string` | Display name. |
| `count` | `number` | Number of matching documents for this theme/law-type. |

### `search_summary` Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `total_documents` | `number` | Number of documents included in this response (after `maxResults` slicing). |
| `total_found` | `number` | Total matches reported by Adala. |
| `available_themes` | `number` | Count of themes returned. |
| `available_law_types` | `number` | Count of law types returned. |
| `recommendations` | `array` | Always an empty array — reserved field, not currently populated. |

---

## Output Example

```json
{
  "success": true,
  "keyword": "code du travail",
  "data": {
    "documents": [
      {
        "name": "Loi n° 65-99 relative au Code du travail",
        "type": "Loi",
        "language": "fr",
        "pdf_url": "https://adala.justice.gov.ma/api/files/loi-65-99.pdf",
        "object": "Réglementation du travail",
        "law_type": "Loi",
        "theme": "Droit du travail",
        "resource": "Bulletin Officiel",
        "law_number": "65-99",
        "keywords": "travail, salarié, employeur",
        "gregorian_date": "2004-05-11"
      }
    ],
    "themes": [
      { "id": "droit-travail", "name": "Droit du travail", "count": 42 }
    ],
    "law_types": [
      { "id": "loi", "name": "Loi", "count": 128 }
    ],
    "total_found": 42,
    "search_summary": {
      "total_documents": 1,
      "total_found": 42,
      "available_themes": 1,
      "available_law_types": 1,
      "recommendations": []
    }
  },
  "source": "Adala.justice.gov.ma",
  "total_found": 42
}
```

### Failure Example

```json
{
  "success": false,
  "error": "HTTP error: 404",
  "keyword": "code du travail"
}
```

---

## Configuration Examples

### Basic Keyword Search

```json
{
  "keyword": "code du travail"
}
```

### Filtered by Theme and Law Type

```json
{
  "keyword": "propriété",
  "theme": "droit-civil",
  "lawType": "loi"
}
```

### Larger Result Set

```json
{
  "keyword": "environnement",
  "maxResults": 25
}
```

---

## Workflow Integration

### Sample Workflow: Search → Function

```json
{
  "nodes": [
    {
      "id": "legal-search",
      "type": "legal-search",
      "config": {
        "keyword": "code du travail"
      }
    },
    {
      "id": "format-results",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Search → If → Notification

```json
{
  "nodes": [
    {
      "id": "legal-search",
      "type": "legal-search",
      "config": {
        "keyword": "loi de finances 2027"
      }
    },
    {
      "id": "check-found",
      "type": "if"
    },
    {
      "id": "notify-legal-team",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Legal Search → LLM Summarization

```json
{
  "nodes": [
    {
      "id": "legal-search",
      "type": "legal-search",
      "config": {
        "keyword": "protection des données personnelles",
        "maxResults": 5
      }
    },
    {
      "id": "summarize-laws",
      "type": "llm"
    }
  ]
}
```

### Common Patterns

- Legal Search → Function (extract `pdf_url`) → HTTP Request — download matched legal texts
- Schedule → Legal Search → If (`total_found` increased) → Notification — monitor for new legislation
- Legal Search → LLM → Function — summarize and categorize search results

---

## Error Handling

The node has **two distinct failure modes**:

### Returned (Not Thrown): HTTP or Network Failure

```json
{ "success": false, "error": "HTTP error: <status>", "keyword": "<keyword>" }
```
or
```json
{ "success": false, "error": "<network error message>", "keyword": "<keyword>" }
```

These come from `searchLegalTexts` catching its own errors and returning a failure object, which `handleTick` passes through directly without throwing.

### Thrown: Missing Keyword

```text
Search keyword is required
```

Raised when both `keyword` and the input data are empty.

### Thrown: Unexpected Internal Error

```text
Legal search failed: <underlying error message>
```

Raised only if an error occurs **outside** the `searchLegalTexts` call itself (e.g. in `parseLegalResults`) — genuinely unexpected failures, since `searchLegalTexts` already catches its own errors.

---

## Troubleshooting

### `success: false, error: "HTTP error: 404"`

**Cause**

Most likely cause: Adala redeployed their site, invalidating the **hardcoded build ID** (`THP5ZL1eNCinRAZ1hWfN0`) embedded in the endpoint URL — this is the primary fragility point of this node (see [Important: Unofficial Internal Endpoint](#️-important-unofficial-internal-endpoint)).

**Solution**

Visit `adala.justice.gov.ma`, perform a search in the browser, and inspect the network tab for the current `/_next/data/<build-id>/fr/search.json` URL to find the new build ID. The node's implementation needs to be updated with this new ID.

---

### `success: false, error: "HTTP error: 403"`

**Cause**

Adala may be blocking the request based on missing/mismatched headers, IP-based restrictions, or bot detection.

**Solution**

Verify the request headers (`User-Agent`, `Referer`) still match what a real browser sends; Adala may have added stricter checks since this node was built.

---

### "Legal search failed: Search keyword is required"

**Cause**

`keyword` is empty and no usable input data was provided.

**Solution**

Set `keyword` explicitly, or ensure the upstream node passes a non-empty value as input data.

---

### `data.documents` is Empty but `total_found` is Non-Zero

**Cause**

This should not normally happen given the slicing logic, but could occur if `maxResults` is set to `0` or a negative number, since `slice(0, maxResults)` with `0` returns an empty array while `total` still reflects the full match count.

**Solution**

Ensure `maxResults` is set to a positive integer.

---

### Response Structure Looks Completely Different / All Fields Empty

**Cause**

Adala changed their page's internal data structure (`pageProps.searchResult`, `themesResult`, `lawsResult`) as part of a site update — since this node relies on an undocumented internal Next.js data shape, any frontend refactor on Adala's side can silently break field mapping.

**Solution**

Compare the raw JSON response against the field paths listed in [Result Parsing Details](#result-parsing-details); the node's parsing logic would need to be updated to match the new structure.

---

## Security

The node performs outbound HTTP requests to Morocco's Ministry of Justice legal portal (`adala.justice.gov.ma`), sending browser-like headers (`User-Agent`, `Accept-Language`, `Referer`) to mimic a normal browser request.

No API key or authentication credential is required — this is publicly accessible legal document data.

---

## Notes

This node relies on Adala's **internal, undocumented Next.js data endpoint** rather than a stable public API — it is inherently fragile to site redeployments and structural changes on Adala's end, more so than most other nodes in this documentation series.

`maxResults` only limits the **documents** array client-side after fetching — it is not passed to Adala as a server-side pagination parameter, and `themes`/`law_types` are always returned in full regardless of `maxResults`.

The node does not:

- Support pagination beyond the first page of results (no `page`/`offset` parameter)
- Cache search results between calls
- Validate that `theme`/`lawType` values correspond to real Adala theme/law-type keys before searching
- Retry on failure (HTTP/network errors are returned once, not retried)
- Download the PDF documents themselves — only returns the `pdf_url` link

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
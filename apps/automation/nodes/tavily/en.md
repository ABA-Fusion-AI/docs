---
node_id: "tavily"
title: "Tavily"
description: "Perform web search or extract content from URLs using Tavily API."
category: "Web Search / Content Extraction"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:

- tavily
- web-search
- content-extraction
- rag
- llm-search
- news
- finance
- api

related_nodes:
- function
- if
- http-request
- bing-search

---

# Tavily

> **Category:** search-nodes | **Type:** Action Node

Perform an **LLM-optimized web search** or **extract content from specific URLs** using the **Tavily** API — a search API purpose-built for feeding AI/RAG pipelines.

The **Tavily** node exposes two operations: `search`, which performs a topic-aware web search with configurable depth and result count, and `extract`, which pulls clean content from a list of URLs.

### Supported Features

- Web search with topic targeting (`general`, `news`, `finance`)
- Configurable search depth (`basic` or `advanced`)
- Optional raw content and image inclusion in search results
- Configurable maximum result count
- Content extraction from one or more URLs, with basic or advanced extraction modes
- Accepts search query or extraction URLs from either node configuration or upstream workflow data, with flexible input shape handling for `extract`
- Structured error messages surfaced from Tavily's own API error payload

### Use Cases

- Perform search-augmented generation (RAG) by retrieving current web results for an LLM
- Search recent news or financial information with topic-specific relevance tuning
- Extract clean, LLM-ready text content from a list of known URLs (e.g. found via a prior search)
- Build a research or fact-checking workflow combining search and extraction
- Retrieve images alongside search results for a visual content pipeline

---

## Configuration

### Base Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ✅ Yes | `""` | Tavily API key. Marked for secrets usage (`usesecrets: true`) in the schema. |
| `operation` | `enum` | ❌ No | `"search"` | Operation to perform: `search` or `extract`. |

### Search Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `query` | `string` | ✅ Yes (unless provided via input data) | — | Search query text. |
| `topic` | `enum` | ❌ No | `"general"` | Search topic focus: `general`, `news`, or `finance`. |
| `includeRawContent` | `boolean` | ❌ No | `false` | Whether to include the raw page content in results. |
| `includeImages` | `boolean` | ❌ No | `false` | Whether to include images in results. |
| `searchDepth` | `enum` | ❌ No | `"basic"` | Search depth: `basic` or `advanced` (more thorough, typically slower/costlier). |
| `maxResults` | `number` | ❌ No | `5` | Maximum number of search results to return. |

### Extract Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `urls` | `string[]` | ✅ Yes (unless provided via input data) | — | List of URLs to extract content from. |
| `extractionType` | `enum` | ❌ No | `"basic"` | Extraction mode: `basic` or `advanced`. |

---

## Input Data Fallback

### Search

If `query` is left empty, the node falls back to the **incoming workflow data**: a string is used directly, and any other data type is coerced with `String(data)`.

### Extract

If `urls` is empty or unset, the node attempts to derive URLs from the **incoming workflow data**, checked in this order:

1. If `data` is a `string` — used as a single-element URL list.
2. If `data` is an `array` — each element is coerced to a string and used as the URL list.
3. If `data` is an `object` with a `url` field — that field's value (coerced to string) is used as a single-element list.
4. If `data` is an `object` with a `urls` field — if that field is an array, each element is coerced to string; otherwise the whole value is coerced to a single-element list.

If none of these produce at least one URL, the node throws.

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `query` (search) or to derive `urls` (extract), per the resolution rules above.

### Outputs

The node returns **Tavily's raw JSON response** for the selected operation — there is no reshaping.

#### `search` Output (typical fields)

| Field | Type | Description |
| ----- | ---- | ----------- |
| `query` | `string` | The query that was searched. |
| `results` | `array` | List of result objects, each with `title`, `url`, `content`, `score`, and (if `includeRawContent`) `raw_content`. |
| `images` | `array` | Image results, present when `includeImages` is `true`. |
| `response_time` | `number` | Tavily's reported response time. |

#### `extract` Output (typical fields)

| Field | Type | Description |
| ----- | ---- | ----------- |
| `results` | `array` | List of `{ url, raw_content, images? }` objects, one per successfully extracted URL. |
| `failed_results` | `array` | List of URLs that could not be extracted, with an error reason. |

The exact fields present depend on Tavily's API version and the options used — this documentation reflects typical response shapes, not a guaranteed schema.

---

## Output Example

### `search`

```json
{
  "query": "renewable energy Morocco",
  "results": [
    {
      "title": "Morocco's renewable energy strategy",
      "url": "https://example.com/morocco-renewable-energy",
      "content": "Morocco has become a regional leader in renewable energy investment...",
      "score": 0.91
    }
  ],
  "response_time": 1.12
}
```

### `extract`

```json
{
  "results": [
    {
      "url": "https://example.com/article",
      "raw_content": "Full extracted article text..."
    }
  ],
  "failed_results": []
}
```

---

## Configuration Examples

### Basic Search

```json
{
  "operation": "search",
  "apiKey": "your-tavily-api-key",
  "query": "renewable energy Morocco"
}
```

### News Search, Advanced Depth

```json
{
  "operation": "search",
  "apiKey": "your-tavily-api-key",
  "query": "central bank interest rate decision",
  "topic": "finance",
  "searchDepth": "advanced",
  "maxResults": 10
}
```

### Search with Images and Raw Content

```json
{
  "operation": "search",
  "apiKey": "your-tavily-api-key",
  "query": "solar panel installation guide",
  "includeImages": true,
  "includeRawContent": true
}
```

### Extract from Known URLs

```json
{
  "operation": "extract",
  "apiKey": "your-tavily-api-key",
  "urls": ["https://example.com/article-1", "https://example.com/article-2"],
  "extractionType": "advanced"
}
```

---

## Workflow Integration

### Sample Workflow: Search → LLM

```json
{
  "nodes": [
    {
      "id": "tavily-search",
      "type": "tavily",
      "config": {
        "operation": "search",
        "apiKey": "your-tavily-api-key",
        "query": "latest AI regulation news",
        "topic": "news"
      }
    },
    {
      "id": "synthesize-answer",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: Search → Function (extract URLs) → Extract

```json
{
  "nodes": [
    {
      "id": "tavily-search",
      "type": "tavily",
      "config": {
        "operation": "search",
        "apiKey": "your-tavily-api-key",
        "query": "renewable energy Morocco"
      }
    },
    {
      "id": "pick-top-urls",
      "type": "function"
    },
    {
      "id": "tavily-extract",
      "type": "tavily",
      "config": {
        "operation": "extract",
        "apiKey": "your-tavily-api-key"
      }
    }
  ]
}
```

Here, `tavily-extract` relies on the [input data fallback](#input-data-fallback) to receive its `urls` from the upstream `pick-top-urls` node's output.

### Sample Workflow: LLM (generate query) → Tavily Search → LLM (synthesize)

```json
{
  "nodes": [
    {
      "id": "generate-search-query",
      "type": "llm"
    },
    {
      "id": "tavily-search",
      "type": "tavily",
      "config": {
        "apiKey": "your-tavily-api-key",
        "operation": "search"
      }
    },
    {
      "id": "synthesize-final-answer",
      "type": "llm"
    }
  ]
}
```

### Common Patterns

- LLM (generate query) → Tavily (`search`) → LLM (synthesize) — search-augmented generation (RAG)
- Tavily (`search`) → Function (rank/select) → Tavily (`extract`) → LLM — search then deep-read top results
- Tavily (`extract`) → Function → Database — archive extracted content from known URLs

---

## Error Handling

### Missing API Key

```text
Tavily API key is required. Please provide it in the node parameters or use a template expression like {{secrets.tavilyApiKey}}
```

Raised when `apiKey` is empty or whitespace-only.

### Missing Search Query

```text
Search query is required
```

Raised for `search` when both `query` and the input data are empty.

### Missing Extraction URLs

```text
At least one URL is required for content extraction
```

Raised for `extract` when `urls` is empty and no URLs could be derived from the input data.

### Unknown Operation

```text
Unknown operation: <operation>
```

Should not normally occur given the `operation` enum.

### Tavily API Error

```text
Tavily API error: <error message>
```

Raised when Tavily returns a non-OK HTTP status; the message is taken from the response body's `error` or `message` field when present, otherwise the HTTP status.

### Wrapped Failure

```text
Tavily <operation> failed: <underlying error message>
```

All errors, including the ones above, are re-thrown wrapped in this message, with `<operation>` reflecting whichever operation (`search`/`extract`) was running.

---

## Troubleshooting

### "Tavily search failed: Tavily API key is required. ..."

**Cause**

`apiKey` was left empty or whitespace-only.

**Solution**

Set a valid Tavily API key, ideally via a secrets/template expression like `{{secrets.tavilyApiKey}}` given the schema's `usesecrets: true` hint, rather than hardcoding it.

---

### "Tavily search failed: Search query is required"

**Cause**

`query` is empty and no usable input data was provided for a `search` operation.

**Solution**

Set `query` explicitly, or ensure the upstream node passes a non-empty string as input data.

---

### "Tavily extract failed: At least one URL is required for content extraction"

**Cause**

`urls` is empty and the input data didn't match any of the recognized fallback shapes (string, array, `{ url }`, or `{ urls }`).

**Solution**

Set `urls` explicitly, or ensure the upstream node's output matches one of the supported input shapes.

---

### "Tavily <operation> failed: Tavily API error: Invalid API key"

**Cause**

The provided key is invalid, revoked, or expired.

**Solution**

Verify the key on the Tavily dashboard.

---

### "Tavily <operation> failed: Tavily API error: ..." (Rate Limit / Credit Exhaustion)

**Cause**

The Tavily account has exceeded its request or credit quota for the current billing period.

**Solution**

Check usage on the Tavily dashboard; `advanced` search/extraction modes typically consume more credits per call than `basic`.

---

### `extract` Partially Fails for Some URLs

**Cause**

Individual URLs can fail extraction (e.g. paywalled, JavaScript-rendered, or blocked pages) — Tavily reports these in `failed_results` rather than failing the entire call.

**Solution**

Check `failed_results` in the output to identify which URLs need a different retrieval approach (e.g. an [HTTP Request](./http-request.md) node with different headers).

---

## Security

The node performs outbound HTTP requests to the Tavily API (`api.tavily.com`).

The `apiKey` is sent in the JSON request body (`api_key` field) rather than as a URL query parameter.

The schema marks `apiKey` with `usesecrets: true`, signaling it should be sourced from the platform's secrets/credentials mechanism (e.g. `{{secrets.tavilyApiKey}}`) rather than entered as plain configuration.

---

## Notes

The node returns Tavily's raw JSON response for both operations, with no reshaping.

The node does not:

- Support Tavily's other endpoints (e.g. `/qna_search` question-answering shortcut)
- Cache search or extraction results between calls
- Retry on rate-limit or credit-exhaustion errors
- Validate URL format before sending an `extract` request
- Deduplicate URLs in an `extract` call

It is intended to provide direct access to Tavily's search and extraction capabilities for downstream RAG, research, and content-aggregation workflows.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-26 | Initial release |
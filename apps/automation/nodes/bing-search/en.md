---
node_id: "bing-search"
title: "Bing Search"
description: "Search Bing using SerpApi (official Bing Search API was retired). Returns web search results, images, videos, and news."
category: "Web Search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- bing
- web-search
- serpapi
- search
- news
- images
- videos
- api

related_nodes:
- function
- if
- http-request

---

# Bing Search

> **Category:** search-nodes | **Type:** Action Node

Search **Bing** via **SerpApi**, since Microsoft's official Bing Search API (Azure Cognitive Services) has been retired.

The **Bing Search** node proxies a Bing search through SerpApi's `/search.json?engine=bing` endpoint, accepting a query with market, safe-search, and pagination options, and returns SerpApi's full parsed Bing results payload (web results, and where available, images/videos/news blocks).

### Supported Features

- Keyword search against Bing (via SerpApi)
- Configurable result count and pagination offset
- Market/locale targeting (`mkt`, e.g. `en-US`, `fr-FR`)
- Configurable safe search level (`Off`, `Moderate`, `Strict`)
- Accepts the search query from either node configuration or upstream workflow data
- Dedicated fallback error if `useSerpApi` is disabled, explaining that the official Bing API no longer exists
- Fallback to a general `apiKey` field if a dedicated `serpApiKey` isn't set

---

## ⚠️ Important: Requires SerpApi, Not a Direct Bing API Key

Microsoft retired the official Bing Search API, so this node **only works through SerpApi** — a third-party service that scrapes and normalizes search engine results, including Bing. The `apiKey`/`serpApiKey` configured on this node must be a **SerpApi API key**, not a Microsoft/Azure key. Setting `useSerpApi` to `false` will always throw, since there is no alternative implementation.

### Use Cases

- Perform web searches from a workflow without a browser
- Retrieve current search results for a research or monitoring pipeline
- Build a search-augmented LLM workflow (retrieve then summarize)
- Localize search results to a specific market/language via `mkt`
- Feed search results into a `Function` node for filtering or ranking

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ❌ No (see [SerpApi Key Resolution](#serpapi-key-resolution)) | `""` | Fallback API key, used only if `serpApiKey` is empty. |
| `query` | `string` | ✅ Yes (unless provided via input data) | — | Search query text. |
| `count` | `number` | ❌ No | `10` | Number of results to request from SerpApi/Bing. |
| `offset` | `number` | ❌ No | `0` | Result offset for pagination (sent as SerpApi's `first` parameter). |
| `mkt` | `string` | ❌ No | `"en-US"` | Market/locale code for the search, e.g. `en-US`, `fr-FR`. |
| `safeSearch` | `enum` | ❌ No | `"Moderate"` | Safe search level: `Off`, `Moderate`, or `Strict`. |
| `useSerpApi` | `boolean` | ❌ No | `true` | Must remain `true` — the node throws immediately if set to `false`. |
| `serpApiKey` | `string` | ✅ Yes (recommended; see below) | `""` | Dedicated SerpApi API key. Takes priority over `apiKey` when both are set. |

---

## SerpApi Key Resolution

The node resolves which key to send to SerpApi as follows:

```text
apiKeyToUse = serpApiKey || apiKey
```

`serpApiKey` is checked first; if it's empty, the node falls back to `apiKey`. If **both** are empty, the node throws before making any request. This allows either field to be used interchangeably, but `serpApiKey` is the more explicit/preferred option since it avoids ambiguity with the generic `apiKey` name.

---

## Input Data Fallback

If `query` is left empty in the configuration, the node falls back to the **incoming workflow data**: a string is used directly, and any other data type is coerced with `String(data)`. The resulting query is also checked for whitespace-only content, not just emptiness.

---

## Request Parameters Sent to SerpApi

| SerpApi Parameter | Source | Notes |
| ------------------ | ------ | ----- |
| `engine` | Fixed | Always `"bing"`. |
| `q` | `query` (or input data) | The search text. |
| `api_key` | Resolved per [SerpApi Key Resolution](#serpapi-key-resolution) | — |
| `count` | `count` | Defaults to `10` if falsy. |
| `first` | `offset` | Defaults to `0` if falsy. SerpApi/Bing's pagination parameter name. |
| `mkt` | `mkt` | Defaults to `"en-US"` if falsy. |
| `safe` | `safeSearch` | Mapped: `Off` → `"off"`, `Strict` → `"strict"`, anything else (including `Moderate`) → `"moderate"`. |

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `query` when the config field is empty.

### Outputs

The node returns **SerpApi's raw JSON response** for the Bing engine — there is no reshaping or field selection. The response typically includes:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `search_metadata` | `object` | SerpApi request metadata (status, processing time, result URL). |
| `search_parameters` | `object` | Echo of the parameters SerpApi used for the search. |
| `organic_results` | `array` | Web search results, each with `title`, `link`, `snippet`, `position`. |
| `images_results` | `array` | Image results, when available for the query. |
| `videos_results` | `array` | Video results, when available for the query. |
| `news_results` | `array` | News results, when available for the query. |
| `related_searches` | `array` | Related search suggestions, when available. |

The exact set of fields present depends entirely on what SerpApi returns for the given query — not all result types are present for every search.

---

## Output Example (Abbreviated)

```json
{
  "search_metadata": {
    "status": "Success",
    "total_time_taken": 1.24
  },
  "search_parameters": {
    "engine": "bing",
    "q": "renewable energy Morocco",
    "mkt": "en-US"
  },
  "organic_results": [
    {
      "position": 1,
      "title": "Morocco's renewable energy strategy",
      "link": "https://example.com/morocco-renewable-energy",
      "snippet": "Morocco has become a regional leader in renewable energy..."
    }
  ],
  "related_searches": [
    { "query": "Morocco solar energy projects" }
  ]
}
```

---

## Configuration Examples

### Basic Search

```json
{
  "query": "renewable energy Morocco",
  "serpApiKey": "your-serpapi-key"
}
```

### Localized Search (French Market)

```json
{
  "query": "énergie renouvelable Maroc",
  "serpApiKey": "your-serpapi-key",
  "mkt": "fr-FR"
}
```

### Paginated, Larger Result Set

```json
{
  "query": "artificial intelligence policy",
  "serpApiKey": "your-serpapi-key",
  "count": 20,
  "offset": 20
}
```

### Strict Safe Search

```json
{
  "query": "family friendly content",
  "serpApiKey": "your-serpapi-key",
  "safeSearch": "Strict"
}
```

---

## Workflow Integration

### Sample Workflow: Search → Function

```json
{
  "nodes": [
    {
      "id": "bing-search",
      "type": "bing-search",
      "config": {
        "query": "renewable energy Morocco",
        "serpApiKey": "your-serpapi-key"
      }
    },
    {
      "id": "extract-links",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Search → LLM Summarization

```json
{
  "nodes": [
    {
      "id": "bing-search",
      "type": "bing-search",
      "config": {
        "query": "latest AI regulation news",
        "serpApiKey": "your-serpapi-key",
        "count": 10
      }
    },
    {
      "id": "summarize-results",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: LLM (generate query) → Bing Search → Function

```json
{
  "nodes": [
    {
      "id": "generate-search-query",
      "type": "llm"
    },
    {
      "id": "bing-search",
      "type": "bing-search",
      "config": {
        "serpApiKey": "your-serpapi-key"
      }
    },
    {
      "id": "rank-results",
      "type": "function"
    }
  ]
}
```

Here, `bing-search` relies on the [input data fallback](#input-data-fallback) to receive its `query` from the upstream LLM node's output.

### Common Patterns

- LLM (generate query) → Bing Search → Function (rank/filter) → LLM (synthesize answer) — search-augmented generation (RAG-style web search)
- Bing Search → Function (extract `organic_results`) → Database — build a search result archive
- Bing Search → If (check `organic_results.length`) → Notification — alert on no results

---

## Error Handling

### Missing Query

```text
Search query is required
```

Raised when both `query` and the input data are empty or whitespace-only.

### Missing API Key

```text
SerpApi API key is required for Bing search. Please provide it in the node parameters or use a template expression like {{secrets.serpApiKey}}
```

Raised when both `serpApiKey` and `apiKey` are empty.

### `useSerpApi` Disabled

```text
Official Bing Search API was retired. Please use SerpApi by setting useSerpApi to true.
```

Raised unconditionally when `useSerpApi` is `false` — there is no alternative code path.

### SerpApi Error

```text
SerpApi Bing search error: <error message>
```

Raised when SerpApi returns a non-OK HTTP status; the message is taken from the response body's `error` field when present, otherwise the HTTP status.

### Wrapped Failure

```text
Bing search failed: <underlying error message>
```

All errors from the SerpApi request path (including the one directly above) are re-thrown wrapped in this message.

---

## Troubleshooting

### "Bing search failed: SerpApi API key is required for Bing search. ..."

**Cause**

Neither `serpApiKey` nor `apiKey` was set.

**Solution**

Set `serpApiKey` (preferred) to a valid SerpApi API key, obtained from a SerpApi account.

---

### "Official Bing Search API was retired. Please use SerpApi by setting useSerpApi to true."

**Cause**

`useSerpApi` was explicitly set to `false`.

**Solution**

Set `useSerpApi` to `true` (the default) — this node has no other way to search Bing, since Microsoft retired the direct API.

---

### "Bing search failed: SerpApi Bing search error: Invalid API key"

**Cause**

The configured key is not a valid, active SerpApi key — a common mistake is using a Microsoft/Azure Bing key here instead, which SerpApi will not recognize.

**Solution**

Verify the key on the SerpApi dashboard; confirm it is a SerpApi key, not a retired Bing API key.

---

### "Bing search failed: SerpApi Bing search error: ..." (Rate Limit / Quota)

**Cause**

The SerpApi account has exceeded its monthly search quota for its current plan.

**Solution**

Check usage on the SerpApi dashboard and upgrade the plan or wait for the quota to reset if needed.

---

### Some Result Fields (e.g. `images_results`, `news_results`) Are Missing

**Cause**

SerpApi only includes result-type blocks (images, videos, news, related searches) when Bing actually returns that type of content for the given query — a plain web query will often omit them entirely.

**Solution**

Check for the presence of each field before accessing it downstream (e.g. in a `Function` node), rather than assuming all fields are always present.

---

## Security

The node performs outbound HTTP requests to SerpApi (`serpapi.com`), which in turn queries Bing on the node's behalf.

The resolved API key is sent as a query parameter (`api_key=...`) on every request, as required by SerpApi.

The error message for a missing key explicitly suggests using a secrets/template expression (e.g. `{{secrets.serpApiKey}}`) rather than hardcoding the key directly in the node configuration — follow this guidance where the platform supports it.

---

## Notes

The node returns SerpApi's full, unmodified JSON response for the Bing engine — including SerpApi-specific metadata fields (`search_metadata`, `search_parameters`) alongside the actual search results.

The node does not:

- Connect directly to Microsoft's Bing API (which no longer exists in this form)
- Reshape or filter SerpApi's response — all result blocks are passed through as-is
- Support other SerpApi engines (Google, Yahoo, DuckDuckGo, etc.) — only `engine=bing` is used
- Cache search results between calls
- Retry on rate-limit or quota errors

It is intended to provide Bing-sourced search results for downstream research, monitoring, and search-augmented generation workflows, via SerpApi as the only viable path since the official API's retirement.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
---
node_id: "naver-search"
title: "Naver Search"
description: "Search Naver (Korean search engine) using SerpAPI. Returns structured web search results."
category: "Web Search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:

- naver
- korean-search
- serpapi
- web-search
- search
- south-korea
- api

related_nodes:
- bing-search
- tavily
- function
- if
- http-request

---

# Naver Search

> **Category:** search-nodes | **Type:** Action Node

Search **Naver**, South Korea's dominant search engine, via **SerpApi**.

The **Naver Search** node queries SerpApi's `/search.json?engine=naver` endpoint with a search query, content-type filter (`web`, `news`, `blog`, `image`, `video`), and pagination, then returns a trimmed set of organic results alongside related searches and search metadata.

### Supported Features

- Keyword search against Naver
- Content-type targeting via `where` (`web`, `news`, `blog`, `image`, `video`)
- Page-based pagination, converted internally to Naver/SerpApi's `start`/`display` offset model
- Accepts the search query from either node configuration or upstream workflow data
- API key fallback to a `SERPAPI_API_KEY` environment variable
- Client-side result trimming to `maxResults`, independent of the `display` count requested from the API

### Use Cases

- Search Korean-language web content, news, or blogs where Naver has better coverage than Google/Bing
- Build a Korea-market content or brand monitoring workflow
- Retrieve Naver blog or news results for a research or localization pipeline
- Combine with an LLM node for search-augmented generation targeting Korean sources
- Search Naver images or videos for a media-discovery workflow

---

## ⚠️ Important: Requires a SerpApi Key, Not a Naver API Key

Like [Bing Search](./bing-search.md), this node **only works through SerpApi** — a third-party service that queries and normalizes results from search engines including Naver. The configured `apiKey` must be a **SerpApi API key**, not a Naver Developers API credential.

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ✅ Yes (see [API Key Resolution](#api-key-resolution)) | `""` | SerpApi API key. |
| `query` | `string` | ✅ Yes (unless provided via input data) | — | Search query text. |
| `page` | `number` | ❌ No | `1` | Page number (1-indexed). Combined with `maxResults` to compute Naver's `start` offset. |
| `maxResults` | `number` | ❌ No | `10` | Number of results requested per page (sent as `display`), and also the cap applied to the returned `organic_results` array. |
| `where` | `enum` | ❌ No | `"web"` | Content type to search: `web`, `news`, `blog`, `image`, or `video`. |

---

## API Key Resolution

```text
apiKey = config.apiKey || process.env.SERPAPI_API_KEY
```

If `apiKey` is not set in the node configuration, the node falls back to the `SERPAPI_API_KEY` environment variable. If neither is available, the node throws before making any request.

---

## Input Data Fallback

If `query` is left empty in the configuration, the node falls back to the **incoming workflow data**: a string is used directly, and any other data type is coerced with `String(data)`. The resulting query is also checked for whitespace-only content, not just emptiness.

---

## Pagination Mapping

Naver's (and SerpApi's) pagination for this engine uses a 1-indexed `start` offset and a `display` count per page, rather than a simple page number. The node computes this as:

```text
start = (page - 1) * maxResults + 1
display = maxResults
```

For example, `page: 2, maxResults: 10` → `start: 11, display: 10` (results 11–20).

---

## Request Parameters Sent to SerpApi

| SerpApi Parameter | Source | Notes |
| ------------------ | ------ | ----- |
| `engine` | Fixed | Always `"naver"`. |
| `query` | `query` (or input data) | The search text. |
| `api_key` | Resolved per [API Key Resolution](#api-key-resolution) | — |
| `start` | Computed per [Pagination Mapping](#pagination-mapping) | — |
| `display` | `maxResults` | Defaults to `10` if falsy. |
| `where` | `where` | Defaults to `"web"` if falsy. |

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `query` when the config field is empty.

### Outputs

Unlike [Bing Search](./bing-search.md) (which returns SerpApi's full raw response), this node returns a **trimmed subset** of fields:

| Output | Type | Description |
| ------ | ---- | ----------- |
| `query` | `string` | The search query that was used. |
| `organic_results` | `array` | Organic search results, sliced client-side to at most `maxResults` entries. |
| `related_searches` | `array` | Related search suggestions, or `[]` if none were returned. |
| `search_parameters` | `object \| undefined` | SerpApi's echo of the parameters used for the search. |
| `search_information` | `object \| undefined` | SerpApi's metadata about the search (e.g. total results, time taken). |

Note: fields like `images_results`, `news_results`, or engine-specific blocks that SerpApi may include in its raw response are **not** passed through — only the five fields above are returned, regardless of what SerpApi's response actually contains.

---

## Output Example

```json
{
  "query": "서울 맛집",
  "organic_results": [
    {
      "position": 1,
      "title": "서울 맛집 베스트 10",
      "link": "https://example.com/seoul-restaurants",
      "snippet": "서울에서 꼭 가봐야 할 맛집 리스트..."
    }
  ],
  "related_searches": [
    { "query": "강남 맛집" }
  ],
  "search_parameters": {
    "engine": "naver",
    "query": "서울 맛집",
    "where": "web"
  },
  "search_information": {
    "total_results": 128000
  }
}
```

---

## Configuration Examples

### Basic Web Search

```json
{
  "apiKey": "your-serpapi-key",
  "query": "서울 맛집"
}
```

### News Search

```json
{
  "apiKey": "your-serpapi-key",
  "query": "한국 경제 전망",
  "where": "news"
}
```

### Blog Search, Paginated

```json
{
  "apiKey": "your-serpapi-key",
  "query": "제주도 여행",
  "where": "blog",
  "page": 2,
  "maxResults": 20
}
```

### Using Environment Variable for the Key

```json
{
  "query": "K-pop concert 2027"
}
```

(with `SERPAPI_API_KEY` set in the runtime environment)

---

## Workflow Integration

### Sample Workflow: Search → Function

```json
{
  "nodes": [
    {
      "id": "naver-search",
      "type": "naver-search",
      "config": {
        "apiKey": "your-serpapi-key",
        "query": "서울 맛집"
      }
    },
    {
      "id": "format-results",
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
      "id": "naver-search",
      "type": "naver-search",
      "config": {
        "apiKey": "your-serpapi-key",
        "query": "한국 반도체 산업 동향",
        "where": "news"
      }
    },
    {
      "id": "summarize-korean-news",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: LLM (generate query) → Naver Search → Function

```json
{
  "nodes": [
    {
      "id": "generate-search-query",
      "type": "llm"
    },
    {
      "id": "naver-search",
      "type": "naver-search",
      "config": {
        "apiKey": "your-serpapi-key"
      }
    },
    {
      "id": "rank-results",
      "type": "function"
    }
  ]
}
```

Here, `naver-search` relies on the [input data fallback](#input-data-fallback) to receive its `query` from the upstream LLM node's output.

### Common Patterns

- LLM (generate Korean query) → Naver Search → LLM (synthesize) — search-augmented generation targeting Korean sources
- Naver Search (`where: "news"`) → Function → Database — Korea-market news monitoring
- Naver Search (`where: "blog"`) → If (check `organic_results.length`) → Notification — brand mention alerting

---

## Error Handling

### Missing Query

```text
Search query is required
```

Raised when both `query` and the input data are empty or whitespace-only.

### Missing API Key

```text
SerpAPI API key is required. Either provide it in the node configuration or set SERPAPI_API_KEY environment variable. Get an API key from https://serpapi.com/users/sign_up
```

Raised when both `apiKey` and the `SERPAPI_API_KEY` environment variable are unset/empty.

### SerpApi Error

```text
SerpAPI error: <error message>
```

Raised when SerpApi returns a non-OK HTTP status; the message is taken from the response body's `error` field when present, otherwise the HTTP status.

### Wrapped Failure

```text
Naver search via SerpAPI failed: <underlying error message>
```

All errors, including the ones above, are re-thrown wrapped in this message.

---

## Troubleshooting

### "Naver search via SerpAPI failed: SerpAPI API key is required. ..."

**Cause**

Neither `apiKey` nor the `SERPAPI_API_KEY` environment variable was set.

**Solution**

Set `apiKey` in the node configuration, or configure `SERPAPI_API_KEY` in the runtime environment.

---

### "Naver search via SerpAPI failed: Search query is required"

**Cause**

`query` is empty and no usable input data was provided.

**Solution**

Set `query` explicitly, or ensure the upstream node passes a non-empty string as input data.

---

### "Naver search via SerpAPI failed: SerpAPI error: Invalid API key"

**Cause**

The configured key is not a valid, active SerpApi key — a common mistake is attempting to use a Naver Developers API key here instead, which SerpApi will not recognize.

**Solution**

Verify the key on the SerpApi dashboard; confirm it is a SerpApi key, not a Naver-issued credential.

---

### `organic_results` is Empty Despite a Valid Query

**Cause**

Either the query genuinely returned no Naver results, or `where` is set to a content type (e.g. `image`, `video`) where SerpApi's Naver engine returns results under a **different field** than `organic_results` (e.g. an image- or video-specific results array) — this node only reads and returns `organic_results`, `related_searches`, `search_parameters`, and `search_information`, so results in other fields are silently dropped.

**Solution**

For `image`/`video` searches specifically, be aware this node may not surface the relevant result data; consider an [HTTP Request](./http-request.md) node against the same SerpApi endpoint if the full raw response is needed.

---

### Wrong Page of Results Returned

**Cause**

Changing `maxResults` between calls **also changes** which absolute result range `page` refers to, since `start` is computed as `(page - 1) * maxResults + 1` — e.g. `page: 2` means something different at `maxResults: 10` (`start: 11`) versus `maxResults: 20` (`start: 21`).

**Solution**

Keep `maxResults` consistent across paginated calls for a given search session to avoid gaps or overlaps between pages.

---

## Security

The node performs outbound HTTP requests to SerpApi (`serpapi.com`), which in turn queries Naver on the node's behalf.

The resolved API key is sent as a query parameter (`api_key=...`) on every request, as required by SerpApi.

The node reads `process.env.SERPAPI_API_KEY` as a fallback — ensure this environment variable, if used, is scoped appropriately in the deployment environment rather than being globally accessible.

---

## Notes

Unlike [Bing Search](./bing-search.md), which returns SerpApi's entire raw response, this node **returns only a fixed subset of fields** (`query`, `organic_results`, `related_searches`, `search_parameters`, `search_information`) — any other data SerpApi includes for a given `where` type is dropped.

The node does not:

- Connect directly to Naver's own API (Naver Developers API) — only via SerpApi
- Return image/video-specific result arrays even when `where` is set to `image`/`video`
- Cache search results between calls
- Retry on rate-limit or quota errors
- Support other SerpApi engines — only `engine=naver`

It is intended to provide Naver-sourced search results, primarily for web/news/blog content, for downstream Korea-focused research and monitoring workflows.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-31 | Initial release |
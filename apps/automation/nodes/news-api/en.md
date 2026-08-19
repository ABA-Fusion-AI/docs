---
node_id: "newsapi"
title: "NewsAPI"
description: "Search news articles from NewsAPI. Monitor brand mentions or industry news."
category: "Media / News"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- newsapi
- news
- media-monitoring
- brand-monitoring
- articles
- search
- api

related_nodes:
- function
- if
- http-request

---

# NewsAPI

> **Category:** media-nodes | **Type:** Action Node

Search **news articles** across thousands of sources using **NewsAPI**'s `/v2/everything` endpoint.

The **NewsAPI** node queries for articles matching a keyword, with configurable sorting, language, and pagination, and returns a normalized article list — useful for brand monitoring, industry news tracking, or content curation.

### Supported Features

- Keyword search across NewsAPI's full article index
- Configurable sort order (`publishedAt`, `relevancy`, `popularity`)
- Configurable language filter
- Pagination with automatic bounds clamping
- Accepts the search query from either node configuration or upstream workflow data
- Normalized article result objects
- Explicit handling of NewsAPI's own `status: "error"` response field, separate from HTTP-level errors

### Use Cases

- Monitor brand or product mentions across news sources
- Track industry or competitor news for a research or intelligence workflow
- Curate a news digest or newsletter automatically
- Feed recent articles into an LLM node for summarization
- Alert a team when news matching specific keywords is published

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `q` | `string` | ✅ Yes (unless provided via input data) | — | Search query keywords. |
| `apiKey` | `string` | ✅ Yes | `""` | NewsAPI API key. Required — the node throws if empty. |
| `sortBy` | `string` | ❌ No | `"publishedAt"` | Sort order: `relevancy`, `popularity`, or `publishedAt`. |
| `language` | `string` | ❌ No | `"en"` | 2-letter ISO language code to filter articles by. |
| `pageSize` | `number` | ❌ No | `20` | Number of articles per page. Clamped to the 1–100 range. |
| `page` | `number` | ❌ No | `1` | Page number to fetch. Clamped to a minimum of 1. |

---

## Input Data Fallback

If `q` is left empty in the configuration, the node falls back to the **incoming workflow data**: a string is used directly, and any other data type is coerced with `String(data)`.

---

## Parameter Clamping

To stay within NewsAPI's accepted bounds, the node clamps two parameters before sending the request:

| Parameter | Clamping |
| --------- | -------- |
| `pageSize` | `Math.min(Math.max(pageSize, 1), 100)` — forced into the 1–100 range. |
| `page` | `Math.max(page, 1)` — forced to be at least 1. |

Values outside these bounds are silently corrected rather than raising an error.

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `q` when the config field is empty.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `query` | `string` | The search query that was used. |
| `status` | `string` | NewsAPI's own status field (`"ok"` on success). |
| `totalResults` | `number` | Total number of matching articles across all pages, as reported by NewsAPI. |
| `articles` | `array` | List of normalized article objects (see below), for the current page. |
| `total_articles` | `number` | Number of articles returned on this page (`articles.length`). |

### Article Object Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `source` | `object` | Source info (`id`, `name`). |
| `author` | `string` | Article author, when available. |
| `title` | `string` | Article title. |
| `description` | `string` | Short article description/summary. |
| `url` | `string` | Article URL. |
| `urlToImage` | `string` | URL to the article's featured image. |
| `publishedAt` | `string` | ISO 8601 publication timestamp. |
| `content` | `string` | Article content snippet (NewsAPI truncates this on free-tier plans). |

---

## Output Example

```json
{
  "success": true,
  "query": "renewable energy Morocco",
  "status": "ok",
  "totalResults": 342,
  "articles": [
    {
      "source": { "id": null, "name": "Reuters" },
      "author": "Jane Doe",
      "title": "Morocco expands solar capacity with new plant",
      "description": "Morocco has announced a major expansion of its solar energy infrastructure...",
      "url": "https://example.com/morocco-solar-expansion",
      "urlToImage": "https://example.com/images/solar-plant.jpg",
      "publishedAt": "2026-08-15T09:32:00Z",
      "content": "Morocco has announced a major expansion... [+1832 chars]"
    }
  ],
  "total_articles": 1
}
```

---

## Configuration Examples

### Basic Search

```json
{
  "q": "renewable energy Morocco",
  "apiKey": "your-newsapi-key"
}
```

### Sorted by Relevancy, French Language

```json
{
  "q": "énergie renouvelable Maroc",
  "apiKey": "your-newsapi-key",
  "sortBy": "relevancy",
  "language": "fr"
}
```

### Paginated Brand Monitoring

```json
{
  "q": "\"Acme Corp\"",
  "apiKey": "your-newsapi-key",
  "pageSize": 50,
  "page": 2,
  "sortBy": "publishedAt"
}
```

---

## Workflow Integration

### Sample Workflow: Search → Function

```json
{
  "nodes": [
    {
      "id": "newsapi-search",
      "type": "newsapi",
      "config": {
        "q": "renewable energy Morocco",
        "apiKey": "your-newsapi-key"
      }
    },
    {
      "id": "format-digest",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Brand Monitoring → If → Notification

```json
{
  "nodes": [
    {
      "id": "newsapi-brand-search",
      "type": "newsapi",
      "config": {
        "q": "\"Acme Corp\"",
        "apiKey": "your-newsapi-key",
        "sortBy": "publishedAt"
      }
    },
    {
      "id": "check-new-mentions",
      "type": "if"
    },
    {
      "id": "notify-pr-team",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: NewsAPI → LLM Summarization

```json
{
  "nodes": [
    {
      "id": "newsapi-search",
      "type": "newsapi",
      "config": {
        "q": "artificial intelligence regulation",
        "apiKey": "your-newsapi-key",
        "pageSize": 10
      }
    },
    {
      "id": "summarize-articles",
      "type": "llm"
    }
  ]
}
```

### Common Patterns

- Schedule (daily) → NewsAPI → If (new results) → Notification — brand/industry monitoring
- NewsAPI → LLM → Function (format digest) → Email — automated news digest
- NewsAPI → Database — build a searchable news archive

---

## Error Handling

### Missing API Key

```text
NewsAPI key is required
```

Raised when `apiKey` is empty.

### Missing Query

```text
Search query is required
```

Raised when both `q` and the input data are empty.

### HTTP Error

```text
NewsAPI error: <status>
```

Raised when NewsAPI returns a non-OK HTTP status.

### NewsAPI Status Error

```text
NewsAPI error: <message>
```

Raised when the HTTP response is OK, but NewsAPI's own JSON body reports `status: "error"` — common for issues like a rate-limited or invalid API key, which NewsAPI reports as `200 OK` with an error payload rather than a non-OK HTTP status.

### Wrapped Failure

```text
NewsAPI search failed: <underlying error message>
```

All errors, including the ones above, are re-thrown wrapped in this message from `handleTick`.

---

## Troubleshooting

### "NewsAPI search failed: NewsAPI key is required"

**Cause**

`apiKey` was left empty in the node configuration.

**Solution**

Set a valid NewsAPI key in `apiKey`.

---

### "NewsAPI search failed: Search query is required"

**Cause**

`q` is empty and no usable input data was provided.

**Solution**

Set `q` explicitly, or ensure the upstream node passes a non-empty value as input data.

---

### "NewsAPI search failed: NewsAPI error: <message>" (e.g. "rateLimited", "apiKeyInvalid")

**Cause**

NewsAPI returned `status: "error"` in its JSON body — common causes include an invalid API key, exceeding the free-tier rate limit, or a request parameter NewsAPI rejects (e.g. malformed query syntax).

**Solution**

Check the specific `message` text for the exact cause; verify the API key and, on the free developer plan, be aware of the daily request quota and the 1-month article history limit.

---

### "NewsAPI search failed: NewsAPI error: 426" or Similar HTTP Status

**Cause**

A non-OK HTTP status was returned directly — for example, `426` can occur on NewsAPI's free tier when requesting results older than what the plan allows, or making requests from a non-localhost origin on certain restricted plans.

**Solution**

Review NewsAPI's plan restrictions for the API key being used.

---

### Fewer Articles Than `totalResults` Suggests

**Cause**

`articles` only contains the current `page`'s results (bounded by `pageSize`), while `totalResults` reflects the full match count across all pages.

**Solution**

Iterate `page` to retrieve additional results, respecting NewsAPI's plan-specific pagination limits.

---

### `content` Field Looks Truncated

**Cause**

NewsAPI truncates the `content` field (commonly to ~200 characters, appending `"[+N chars]"`) on its free developer plan.

**Solution**

Follow the `url` field to read the full article, or upgrade the NewsAPI plan for full content access.

---

## Security

The node performs outbound HTTP requests to the NewsAPI service (`newsapi.org`).

The `apiKey` is sent as a query parameter (`apiKey=...`) on every request, as required by the NewsAPI API.

---

## Notes

The node returns a normalized article list alongside NewsAPI's native pagination metadata (`totalResults`).

The node does not:

- Support NewsAPI's `/v2/top-headlines` endpoint (only `/v2/everything`)
- Support filtering by domain, source, or date range (only `q`, `sortBy`, `language`, pagination)
- Cache search results between calls
- Automatically paginate through all results in a single call
- Retry on rate-limit errors

It is intended to provide straightforward, filtered news search access for downstream media-monitoring and content-curation workflows.

---


## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release 
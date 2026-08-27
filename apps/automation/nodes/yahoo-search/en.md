---
node_id: "yahoo-search"
title: "Yahoo Search"
description: "Search Yahoo through SerpAPI and return structured web search results."
category: "Web Search & Information"
subcategory: "Regional Search Engines"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - yahoo
  - search
  - serpapi
  - web-search
  - regional-search
  - research
  - information-retrieval
related_nodes:
  - bing-search
  - baidu-search
  - google-search
  - log
---

<!-- SECTION: header -->
# Yahoo Search

> **Category:** Web Search & Information | **Subcategory:** Regional Search Engines | **Type:** Action Node

Search Yahoo through SerpAPI and return structured web search results, including organic listings, snippets, links, and search metadata.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Yahoo Search** node sends a query to SerpAPI's Yahoo engine and returns the provider's structured JSON response. It is useful for Yahoo-specific search coverage, regional research, monitoring, and downstream content processing.

### Key Features

- **Yahoo Search:** Search Yahoo using SerpAPI's `yahoo` engine
- **Structured Results:** Return organic results with titles, links, snippets, and positions
- **Search Metadata:** Preserve query and processing metadata returned by SerpAPI
- **Regional Coverage:** Use Yahoo's regional search context through the SerpAPI engine
- **Dynamic Input:** Use a configured query or provide search data through the `input` port
- **API Authentication:** Requires a SerpAPI API key

### Use Cases

- Research topics using Yahoo search results
- Monitor brand, competitor, or news mentions
- Enrich content and information-retrieval workflows
- Compare regional search coverage with other search engines
- Feed search results into filtering, summarization, or reporting nodes

> Search results are dynamic and may vary by time, location, query interpretation, and SerpAPI availability. Validate important information against authoritative sources.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | Yes | — | Search terms to submit to Yahoo, for example `artificial intelligence` |
| `apiKey` | `string` | Yes | — | SerpAPI API key used to authenticate the request. Store it as a workflow secret. |

If `query` is not configured, the node can read a direct string from the incoming `input` value. An incoming object can provide a `query` property and, where supported, an `apiKey` property.

### API Behavior

The node uses SerpAPI's Yahoo search endpoint with the Yahoo engine:

```text
GET https://serpapi.com/search.json?engine=yahoo&p={query}&api_key={apiKey}
```

Yahoo search uses the `p` parameter for the search phrase in SerpAPI requests. The node fixes the engine to `yahoo`.

### Credential Security

The provided example workflow contains a literal `apiKey` value. Treat that key as exposed: revoke or rotate it in SerpAPI and replace the workflow value with a secret reference. Never commit live API keys in workflow examples.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | A query string, or an object containing `query` and optional lookup configuration. Configured secret values should be preferred for API keys. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Raw structured SerpAPI response for the Yahoo engine |

Typical response fields include:

```json
{
  "search_metadata": {
    "status": "Success",
    "yahoo_url": "https://search.yahoo.com/search?p=artificial+intelligence"
  },
  "search_parameters": {
    "engine": "yahoo",
    "p": "artificial intelligence"
  },
  "search_information": {
    "query_displayed": "artificial intelligence"
  },
  "organic_results": [
    {
      "position": 1,
      "title": "Example result",
      "link": "https://example.com/article",
      "displayed_link": "example.com/article",
      "snippet": "Example search-result summary."
    }
  ]
}
```

The exact response fields depend on Yahoo's result page and SerpAPI parsing. Some searches may also include ads, answer boxes, images, news, videos, related searches, or other result blocks.

### Error Output

Missing credentials, missing queries, invalid parameters, quota exhaustion, network failures, and SerpAPI errors are routed to `error`.

```json
{
  "success": false,
  "error": "SerpAPI Yahoo search failed",
  "query": "artificial intelligence"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Yahoo Search

```json
{
  "query": "artificial intelligence",
  "apiKey": "{{secrets.serpApiKey}}"
}
```

### Regional Research Query

```json
{
  "query": "Morocco renewable energy",
  "apiKey": "{{secrets.serpApiKey}}"
}
```

### Dynamic Query from a Previous Node

Pass a query directly through `input`:

```text
latest cybersecurity news
```

Or pass a named object:

```json
{
  "query": "latest cybersecurity news"
}
```

### Extract Organic Results

Use a Function node after the lookup to keep only the primary fields:

```js
return (input.organic_results || []).map(result => ({
  title: result.title,
  link: result.link,
  snippet: result.snippet,
  position: result.position
}));
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Yahoo and inspect structured results
```

### Common Patterns

- **Basic search:** Manual Trigger → Yahoo Search → Log
- **Research pipeline:** Query source → Yahoo Search → Function → Report
- **Monitoring:** Schedule → Yahoo Search → Filter → Notification
- **Cross-engine comparison:** Yahoo Search + Bing Search → Function → Comparison

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### API key is required

**Cause:** `apiKey` is missing, invalid, revoked, or is not a SerpAPI key.

**Solution:** Provide a valid SerpAPI key through the workflow secret system. Do not use a Yahoo account password or an unrelated search-provider key.

### Query is required

**Cause:** Neither `query` nor a usable incoming `input` value was provided.

**Solution:** Configure a search phrase or pass a string/object containing `query` to the input port.

### Quota or rate limit exceeded

**Cause:** The SerpAPI account has reached its plan limit or request rate.

**Solution:** Check the SerpAPI dashboard, reduce request frequency, add backoff, or wait for the quota to reset.

### Few or unexpected results

**Cause:** Search rankings and result blocks vary by query, region, time, and Yahoo response.

**Solution:** Refine the query and handle optional fields such as `organic_results`, `answer_box`, and `news_results` defensively.

### API response error

**Cause:** SerpAPI rejected the request or Yahoo returned a response that could not be parsed.

**Solution:** Inspect the `error` output, verify the query and key, and retry only after addressing quota or service issues.

### Exposed API key in workflow

**Cause:** A literal key was stored in the example workflow parameters.

**Solution:** Revoke or rotate the key immediately and replace it with a secret reference such as `{{secrets.serpApiKey}}`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Bing Search](./bing-search.md) — Search Bing through SerpAPI
- [Baidu Search](./baidu-search.md) — Search Baidu through SerpAPI
- [HTTP Request](./http-request.md) — Call a custom search endpoint
- [Log](./log.md) — Inspect search results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation |

<!-- /SECTION: changelog -->

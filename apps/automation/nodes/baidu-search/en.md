---
node_id: "baidu-search"
title: "Baidu Search"
description: "Search Baidu using SerpAPI. Returns structured web search results."
category: "peer-only"
subcategory: "search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:
  - baidu
  - search
  - serpapi
  - web-search
  - china
  - scraping
related_nodes:
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# Baidu Search

> **Category:** Peer-Only Integrations | **Type:** Action Node

Execute web searches on **Baidu** (China's primary search engine) via **SerpAPI** and return structured JSON results including organic search listings, titles, snippets, links, and related search queries.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Baidu Search** node enables automated search queries on Baidu using SerpAPI's search gateway (`engine=baidu`).

It simplifies querying the Chinese web market by handling API request formatting, page pagination, and structured JSON parsing. The returned object contains organic search results (title, URL, snippet, position), search metadata, and related search terms.

### Key Features

- **Structured Search Data:** Extracts title, link, snippet, and position for organic search results.
- **Pagination Support:** Easily step through result pages using the `page` parameter.
- **Result Count Control:** Limit organic results using the `maxResults` parameter.
- **Environment Variable Fallback:** Reads `SERPAPI_API_KEY` environment variable automatically if `apiKey` is not specified in the node parameters.
- **Input Expression Fallback:** Accepts query inputs dynamically from preceding workflow nodes.

### Use Cases

- **Market Intelligence in China:** Monitor brand presence, product listings, or competitor mentions on Baidu.
- **SEO & Ranking Tracking:** Track website ranking positions on Baidu SERP over time.
- **Content Aggregation:** Fetch Chinese news, articles, or market data for localization workflows.
- **AI Research Pipelines:** Feed Chinese search context into LLMs or translation nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Conditional | — | SerpAPI API key. If omitted, the node looks for the `SERPAPI_API_KEY` environment variable. |
| `query` | `string` | Conditional | — | Search query string. If omitted, the node uses incoming data from the preceding node. |
| `page` | `number` | ❌ No | `1` | Results page number (1-indexed). |
| `maxResults` | `number` | ❌ No | `10` | Maximum number of organic search results to return. |

---

### Parameter Details

#### `apiKey`
Your secret API key from SerpAPI. You can create a free account and get an API key at [SerpAPI Sign Up](https://serpapi.com/users/sign_up).
- If not provided in the node UI, set the environment variable: `SERPAPI_API_KEY="your_api_key"`.

#### `query`
The text term or phrase to search on Baidu (e.g. `"riad Marrakech Morocco"` or `"人工智能"`).

#### `page`
Specifies the page number to fetch (1-indexed). Maps internally to SerpAPI's `pn` offset parameter.

#### `maxResults`
Limits the maximum number of items returned in `organic_results`. Default is `10`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. If `query` parameter is omitted, `input` string is used as the query. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when search succeeds. Contains query string, organic results list, and search metadata. |
| `error` | `Error` | Emitted when API key is missing, network request fails, or SerpAPI returns an error. |

---

### Output Data Structure

```json
{
  "query": "riad Marrakech Morocco",
  "organic_results": [
    {
      "position": 1,
      "title": "Marrakech Riads - Official Booking Portal",
      "link": "https://example.com/marrakech-riads",
      "snippet": "Discover authentic luxury riads in the heart of Marrakech medina...",
      "displayed_link": "example.com/marrakech-riads"
    },
    {
      "position": 2,
      "title": "Top Rated Stays in Morocco",
      "link": "https://example.org/morocco-hotels",
      "snippet": "Book handpicked traditional riads with pools and rooftop terraces...",
      "displayed_link": "example.org/morocco-hotels"
    }
  ],
  "related_searches": [
    {
      "query": "Morocco tourism Baidu",
      "link": "https://serpapi.com/search.json?engine=baidu&q=Morocco+tourism+Baidu"
    }
  ],
  "search_parameters": {
    "engine": "baidu",
    "q": "riad Marrakech Morocco",
    "pn": "0"
  },
  "search_information": {
    "organic_results_state": "Results for exact spelling"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `query` | `string` | The active search term used |
| `organic_results` | `array` | List of organic web search result objects |
| `related_searches` | `array` | List of related Baidu search queries |
| `search_parameters` | `object` | Parameters sent to SerpAPI engine |
| `search_information` | `object` | Additional metadata regarding SERP state |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Search Baidu for Tourism Keywords

Search Baidu for Moroccan tourism locations with customized page and result limits.

**Parameter Configuration:**

```text
ApiKey:     YOUR_SERPAPI_API_KEY
Query:      riad Marrakech Morocco
Page:       1
MaxResults: 5
```

---

### Example 2: Dynamic Search Query from Upstream Node

Pass search terms dynamically from an upstream Function or HTTP trigger node.

**Workflow Pattern:**

```text
Manual Trigger
  → Function (returns "Moroccan Argan Oil export")
  → Baidu Search (query: expression from Function output)
  → Log
```

---

### Example 3: SERP Monitoring & AI Summarization

Search Baidu and feed organic snippets to an AI node to create market summary reports.

**Workflow Pattern:**

```text
Cron Trigger (Weekly)
  → Baidu Search (query: "Morocco renewable energy")
  → Function (extract snippets into text block)
  → OpenRouter / OpenAI Node (summarize findings)
  → Email Send
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Baidu using SerpAPI
```

### Common Patterns

- **Weekly Rank Tracking:** Cron Trigger → Baidu Search → Store to Database.
- **Multilingual Search:** Webhook Input → Baidu Search → Filter Array → Log.
- **Competitive Analysis:** Baidu Search → For Each → HTTP Request (Fetch Webpage) → AI Analysis.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `SerpAPI API key is required...`

**Cause:** Neither the `apiKey` parameter was set in the node configuration nor was `SERPAPI_API_KEY` set in the environment variables.

**Solution:** Provide a valid key in the `apiKey` parameter or set `SERPAPI_API_KEY` in environment variables. Obtain a free key at [serpapi.com](https://serpapi.com/).

#### Error: `Search query is required`

**Cause:** Both the `query` parameter and the incoming workflow data were empty.

**Solution:** Specify a search string in `query` or pass a string output from the preceding node.

#### Error: `SerpAPI error: Invalid API key`

**Cause:** The provided API key is invalid, suspended, or expired.

**Solution:** Check your API key balance and credentials on the [SerpAPI Dashboard](https://serpapi.com/dashboard).

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `SerpAPI API key is required...` | Missing API Key | Set `apiKey` parameter or `SERPAPI_API_KEY` env var |
| `Search query is required` | Missing query input | Provide a search term in `query` or input data |
| `SerpAPI error: ...` | API quota exceeded or engine error | Check SerpAPI dashboard quota and plan limits |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) — Make custom API calls or scrape web pages
- [Function](../function/en.md) — Format or process search result items
- [Filter Array](../filter-array/en.md) — Filter search results by keyword or position
- [Log](../log/en.md) — Display search results in workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-11 | Initial release |

<!-- /SECTION: changelog -->

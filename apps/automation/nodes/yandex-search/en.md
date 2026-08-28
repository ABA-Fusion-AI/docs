---
node_id: "yandex-search"
title: "Yandex Search"
description: "Search Yandex using SerpAPI. Returns structured web search results."
category: "Web Search & Information"
subcategory: "Regional Search Engines"
version: "1.0.0"
language: "en"
last_updated: "2026-08-28"
author: "Fusion Team"
tags:
  - yandex
  - search
  - serpapi
  - web-search
  - regional-search
  - scraping
related_nodes:
  - baidu-search
  - bing-search
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# Yandex Search

> **Category:** Web Search & Information | **Subcategory:** Regional Search Engines | **Type:** Action Node

Execute web searches on **Yandex** (the leading search engine in Eastern Europe and Central Asia) using **SerpAPI** and receive clean, structured JSON results including organic web listings, snippets, links, ranking positions, and related queries.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Yandex Search** node connects workflows to Yandex's search engine via SerpAPI's search gateway (`engine=yandex`). It eliminates the complexity of scraping, CAPTCHAs, and proxy management by providing normalized search results ready for downstream data transformation, AI augmentation, or monitoring pipelines.

The node automatically handles query fallback resolution from incoming workflow payloads, 0-indexed SerpAPI pagination offsets, and client-side result slicing.

### Key Features

- **Structured Search Results:** Extracts organic listings with title, URL, snippet, and position.
- **Intelligent Query Fallback:** If `query` is omitted in the node config, the node dynamically inspects incoming data (strings, objects with `query`, `searchQuery`, `text`, `q`, `prompt`, or `message`).
- **Pagination & Result Control:** Easily navigate result pages (`page`) and limit output size (`maxResults`).
- **Flexible Authentication:** Supports direct API key entry or automatic fallback to the `SERPAPI_API_KEY` environment variable.
- **Related Queries:** Retrieves related search suggestions to explore adjacent topics.

### Use Cases

- **Eastern European & Eurasian Market Research:** Track brand mentions, localized content, and regional competitors on Yandex.
- **SEO & Ranking Monitoring:** Monitor keyword visibility and domain rankings across Yandex SERPs.
- **AI & RAG Enrichment:** Retrieve real-time search context to ground LLM responses with Yandex search data.
- **Content Discovery & Aggregation:** Automatically discover trending articles, research, and regional web resources.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Conditional | `""` | SerpAPI API key. If omitted or empty, the node falls back to the `SERPAPI_API_KEY` environment variable. |
| `query` | `string` | Conditional | — | The search query string. If omitted, incoming data from the previous node is used. |
| `page` | `number` | ❌ No | `1` | Results page number (1-indexed). Mapped internally to SerpAPI's 0-indexed `p` parameter (`p = page - 1`). |
| `maxResults` | `number` | ❌ No | `10` | Maximum number of organic search results to return (slices `organic_results`). |

---

### Parameter Details

#### `apiKey`
Your SerpAPI API key. Obtain a key by registering at [SerpAPI Sign Up](https://serpapi.com/users/sign_up).
- You can provide the key directly in the node configuration parameters or expression (e.g., `{{secrets.SERPAPI_API_KEY}}`).
- If left blank, the node automatically reads `process.env.SERPAPI_API_KEY`.

#### `query`
The keyword or search phrase to query on Yandex (e.g., `"web development"`, `"ecommerce logistics"`, or `"технологии"`).
- If left empty, the node dynamically checks incoming input data.

#### `page`
The 1-based page number to fetch. Default is `1`.
- Page `1` maps to SerpAPI parameter `p=0`.
- Page `2` maps to SerpAPI parameter `p=1`.

#### `maxResults`
Controls how many organic search results are returned in the `organic_results` array. Default is `10`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. Used as the search query if the `query` parameter is not configured. Supports strings or objects containing `query`, `searchQuery`, `text`, `q`, `prompt`, or `message`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted upon a successful search. Contains the query, limited organic search results, related searches, and SerpAPI search metadata. |
| `error` | `Error` | Emitted when authentication fails, the query is missing, or SerpAPI returns an HTTP error. |

---

### Output Data Structure

```json
{
  "query": "web development",
  "organic_results": [
    {
      "position": 1,
      "title": "MDN Web Docs",
      "link": "https://developer.mozilla.org/en-US/",
      "snippet": "The MDN Web Docs site provides information about Open Web technologies including HTML, CSS, and APIs for both Web sites and progressive web apps.",
      "displayed_link": "developer.mozilla.org"
    },
    {
      "position": 2,
      "title": "W3Schools Online Web Tutorials",
      "link": "https://www.w3schools.com/",
      "snippet": "W3Schools offers free online tutorials, references and exercises in all the major languages of the web.",
      "displayed_link": "w3schools.com"
    }
  ],
  "related_searches": [
    {
      "query": "web development course",
      "link": "https://serpapi.com/search.json?engine=yandex&text=web+development+course"
    },
    {
      "query": "web development roadmap",
      "link": "https://serpapi.com/search.json?engine=yandex&text=web+development+roadmap"
    }
  ],
  "search_parameters": {
    "engine": "yandex",
    "text": "web development",
    "p": "0"
  },
  "search_information": {
    "organic_results_state": "Results for exact spelling"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `query` | `string` | The resolved search query executed against Yandex. |
| `organic_results` | `array` | List of organic result objects containing `position`, `title`, `link`, `snippet`, and `displayed_link`. |
| `related_searches` | `array` | Suggestions and related search queries provided by Yandex. |
| `search_parameters` | `object` | Echo of request parameters processed by SerpAPI (`engine`, `text`, `p`). |
| `search_information` | `object` | Metadata about the search execution and spelling state. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Direct Keyword Search

Perform a search on Yandex with a fixed query and limit the results to the top 5 entries.

**Parameter Configuration:**

```text
ApiKey:     YOUR_SERPAPI_API_KEY
Query:      web development
Page:       1
MaxResults: 5
```

---

### Example 2: Dynamic Query from Upstream Node

Receive a search query dynamically from an incoming webhook, AI prompt, or form trigger.

**Workflow Pattern:**

```text
Webhook / Chat Trigger
  → Yandex Search (query: {{input.body.prompt}})
  → Function (extract links and titles)
  → Log
```

---

### Example 3: Search-Augmented AI Pipeline

Search Yandex for regional information and feed search snippets into an LLM node to generate synthesized summaries.

**Workflow Pattern:**

```text
Manual Trigger
  → Yandex Search (query: "renewable energy developments in Eurasia")
  → Function (format organic_results into context text)
  → AI Chat / LLM Node (generate summary from search context)
  → Log
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Yandex using SerpAPI
```

### Common Patterns

- **Search & Log:** Trigger → Yandex Search → Log output to console.
- **RAG & Context Enrichment:** User Prompt → Yandex Search → Prompt Builder → AI Chat.
- **SERP Archival:** Schedule / Cron Trigger → Yandex Search → Database / Google Sheets (store top ranking URLs).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `SerpAPI API key is required...`

**Cause:** No API key was provided in the node parameters, and the `SERPAPI_API_KEY` environment variable is not defined.

**Solution:** Provide your SerpAPI key in the `apiKey` configuration parameter, or configure `SERPAPI_API_KEY` in your environment. You can sign up for a free key at [serpapi.com](https://serpapi.com/users/sign_up).

#### Error: `Search query is required`

**Cause:** The `query` field was left empty and the incoming workflow payload (`data`) did not contain a valid query string or property.

**Solution:** Set the `query` parameter explicitly in the node UI, or ensure the preceding node outputs a string or an object with one of the supported fields: `query`, `searchQuery`, `text`, `q`, `prompt`, or `message`.

#### Error: `Yandex search via SerpAPI failed: SerpAPI error: Invalid API key`

**Cause:** The provided SerpAPI API key is incorrect, revoked, or expired.

**Solution:** Verify your API key in the [SerpAPI Dashboard](https://serpapi.com/dashboard) and ensure your account has active search credits.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `SerpAPI API key is required...` | Missing API key | Configure `apiKey` or set `SERPAPI_API_KEY` env var |
| `Search query is required` | Missing search text | Supply a query in config or via upstream data |
| `SerpAPI error: Invalid API key` | Incorrect credentials | Check API key on serpapi.com |
| `SerpAPI error: Your searches have been exhausted` | Quota exceeded | Check SerpAPI plan quota or upgrade |
| `Yandex search via SerpAPI failed: HTTP error! status: 5xx` | SerpAPI or Yandex service interruption | Retry the request after a short interval |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Baidu Search](../baidu-search/en.md) — Search the Chinese web via SerpAPI
- [Bing Search](../bing-search/en.md) — Search Bing via SerpAPI
- [HTTP Request](../http-request/en.md) — Make custom API calls
- [Function](../function/en.md) — Transform and filter search results
- [Log](../log/en.md) — View execution outputs and logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-28 | Initial release |

<!-- /SECTION: changelog -->

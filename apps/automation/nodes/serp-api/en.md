---
node_id: "serp-api"
title: "SerpApi"
description: "Search Google, Bing, Baidu, Yahoo, Yandex, DuckDuckGo, eBay and Amazon through SerpApi and get structured JSON results (organic results, ads, images, videos, shopping and more)."
category: "Web Search"
subcategory: "Search Engines"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - serpapi
  - search
  - google
  - bing
  - web-search
  - scraping
  - seo
related_nodes:
  - bing-copilot-search
  - bing-search
  - tavily
  - function
---

<!-- SECTION: overview -->
# SerpApi

> **Category:** Web Search &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Run searches on multiple search engines through **SerpApi** (`https://serpapi.com/search.json`) and get the results page as structured JSON. Supported engines:

`google` · `bing` · `baidu` · `yahoo` · `yandex` · `duckduckgo` · `ebay` · `amazon`

The node returns the raw SerpApi response, which can contain organic results, ads, knowledge graph, images, videos, news, shopping results and more depending on the engine.

### Use Cases

- **SEO monitoring:** Track the ranking of a website for a list of keywords on Google or Bing.
- **Market & price research:** Search products on Amazon or eBay and compare prices.
- **News and image search:** Use Google's `tbm` modes to collect news articles, images or videos on a topic.
- **Localized results:** Compare search results across countries, languages and devices.
- **RAG / AI agents:** Provide fresh web results as context to an LLM node.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | SerpApi API key. Supports expressions (e.g. `{{ secrets.serpApiKey }}`). |
| `engine` | `enum` | No | `google` | Search engine: `google`, `bing`, `baidu`, `yahoo`, `yandex`, `duckduckgo`, `ebay`, `amazon`. |
| `query` | `string` | No* | — | Search query. Supports expressions. |
| `location` | `string` | No | — | Location the search originates from (e.g. `Casablanca, Morocco`). |
| `googleDomain` | `string` | No | `google.com` | Google domain to use (e.g. `google.fr`, `google.co.ma`). Google only. |
| `gl` | `string` | No | — | Country code (e.g. `ma`, `fr`). Google: country; Bing: market (`mkt`, e.g. `fr-FR`). |
| `hl` | `string` | No | — | Interface language (e.g. `fr`, `en`). Google only. |
| `num` | `number` | No | `10` | Number of results. Google only. |
| `start` | `number` | No | `0` | Result offset for pagination. Google (`start`) and Bing (`first`). |
| `safe` | `enum` | No | `active` | SafeSearch: `active` or `off`. Google only. |
| `tbm` | `enum` | No | — | Google search type: `isch` (images), `vid` (videos), `nws` (news), `shop` (shopping). |
| `device` | `enum` | No | `desktop` | Device to simulate: `desktop`, `tablet` or `mobile`. |
| `noCache` | `boolean` | No | `false` | Forces a fresh search instead of a cached result (consumes a search). |
| `async` | `boolean` | No | `false` | Submits the search and returns immediately with a `search_id`. |

\* If `query` is empty, the node uses the incoming data **only when it is a plain string**. With an object input, fill in `query` with an expression such as `{{ input.query }}`.

### Parameters by Engine

| Engine | Query sent as | Engine-specific parameters used |
|--------|---------------|---------------------------------|
| `google` | `q` | `num`, `start`, `safe`, `googleDomain`, `gl`, `hl`, `location`, `tbm` |
| `bing` | `q` | `start` (as `first`), `gl` (as `mkt`), `location` |
| `yahoo` | `p` | `location` |
| `yandex` | `text` | `location` |
| `baidu`, `duckduckgo` | `q` | `location` |
| `ebay` | `_nkw` | `location` |
| `amazon` | `k` | `location` |

`device`, `noCache` and `async` apply to all engines.

### Authentication & API Key

1. Create an account on **SerpApi** (`https://serpapi.com`).
2. Open **Dashboard → API Key**.
3. Copy the key and paste it into the `apiKey` field (or store it in Fusion Secrets).

> Each run consumes one search from your SerpApi plan (cached results are free unless `noCache` is enabled).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` / `object` | Triggers the node. A string input is used as the query when `query` is empty. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Raw SerpApi JSON response. In async mode, also contains a `message` field. |
| `error` | `Error` | Emitted on missing API key, empty query, HTTP or SerpApi errors. |

### Main Response Fields

| Field | Description |
|-------|-------------|
| `search_metadata` | Search ID, status (`Success`, `Processing`, `Error`), timing and SerpApi URLs. |
| `search_parameters` | Parameters actually used for the search. |
| `search_information` | Total results, query time, spelling corrections. |
| `organic_results` | Main results: `position`, `title`, `link`, `snippet`, `source`. |
| `ads`, `knowledge_graph`, `related_questions`, `news_results`, `images_results`, `shopping_results`… | Present depending on the engine and the query. |

### Output Examples

#### Google Search

```json
{
  "search_metadata": {
    "id": "673a1f2b3c4d5e6f7a8b9c0d",
    "status": "Success",
    "total_time_taken": 1.42
  },
  "search_parameters": {
    "engine": "google",
    "q": "fusion automation workflow",
    "google_domain": "google.com",
    "device": "desktop"
  },
  "organic_results": [
    {
      "position": 1,
      "title": "Workflow Automation Explained",
      "link": "https://example.com/workflow-automation",
      "snippet": "Workflow automation uses software to run repetitive tasks...",
      "source": "example.com"
    }
  ]
}
```

#### Async Mode

```json
{
  "search_metadata": {
    "id": "673a1f2b3c4d5e6f7a8b9c0d",
    "status": "Processing",
    "json_endpoint": "https://serpapi.com/searches/673a1f2b3c4d5e6f7a8b9c0d.json"
  },
  "message": "Search is processing asynchronously. Use the search_id to retrieve results later."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Google with SerpApi
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **SerpApi Node:** Sends the query to the selected search engine through SerpApi.
3. **Log Node:** Displays the structured search results.

### Example Configuration

```json
{
  "apiKey": "{{ secrets.SERPAPI_API_KEY }}",
  "engine": "google",
  "query": "best restaurants in Rabat",
  "googleDomain": "google.co.ma",
  "gl": "ma",
  "hl": "fr",
  "num": 10,
  "device": "desktop"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your SerpApi `apiKey` in Fusion's **Secrets** vault. Do not commit hardcoded keys into version control.

- The key is sent as a URL query parameter: avoid logging full request URLs.
- Use `noCache` only when necessary; it consumes a search on every run.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `SerpApi API key is required...`
- **Cause:** `apiKey` is empty.
- **Solution:** Provide the key, or an expression like `{{ secrets.serpApiKey }}`.

#### `Search query is required`
- **Cause:** `query` is empty and the input is an empty string.
- **Solution:** Fill in `query`.

#### Results for `[object Object]`
- **Cause:** `query` is empty and the input is an object; the object is converted to text instead of being read.
- **Solution:** Set `query` explicitly, e.g. `{{ input.query }}`.

#### `SerpApi search failed: SerpApi error: Invalid API key...`
- **Cause:** The API key is wrong or revoked.
- **Solution:** Copy the key again from the SerpApi dashboard.

#### `SerpApi search failed: SerpApi error: Your account has run out of searches.`
- **Cause:** Monthly search quota reached.
- **Solution:** Wait for the quota reset or upgrade your SerpApi plan.

#### Empty `organic_results`
- **Cause:** No results for this query, or a non-Google parameter (`num`, `hl`, `tbm`…) was expected to apply on another engine.
- **Solution:** Check the *Parameters by Engine* table and the `search_parameters` field of the response.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: engines, parameter reference, output schema, and workflow integration |

<!-- /SECTION: changelog -->

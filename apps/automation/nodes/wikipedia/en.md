---
node_id: "wikipedia"
title: "Wikipedia"
description: "Search and retrieve articles, summaries, and metadata from Wikipedia."
category: "Content & Feeds"
subcategory: "search-reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - wikipedia
  - encyclopedia
  - search
  - knowledge
  - articles
  - content
  - mediawiki
  - getbyid
related_nodes:
  - function
  - manual-trigger
  - hacker-news
  - google-search
  - duck-duck-go-search
  - arxiv
---

<!-- SECTION: header -->
# Wikipedia

> **Category:** Content & Feeds | **Type:** Action Node

Search Wikipedia and fetch clean article extracts, rich summaries, lead thumbnails, and geographic coordinates without requiring an API key.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Wikipedia** node integrates workflows with the open Wikipedia encyclopedia. It enables you to perform full-text keyword searches across Wikipedia articles or retrieve comprehensive information for a specific topic by title.

Under the hood, the node connects directly to Wikipedia's public endpoints:
1. **MediaWiki Search API** (`en.wikipedia.org/w/api.php`): Executes keyword searches with pagination and relevance ranking.
2. **Wikipedia REST API** (`en.wikipedia.org/api/rest_v1`): Automatically enriches search results and article lookups with clean plain-text summaries, lead image thumbnails, and geographic coordinates.

### Key Capabilities

- **No Authentication / API Key Required:** Connects seamlessly to Wikipedia's open public APIs out of the box.
- **Two Flexible Operations:**
  - **`search`**: Keyword-based search across millions of articles with pagination support.
  - **`getById`**: Direct lookup of a specific article by title or URL key.
- **Automated Summary Enrichment:** In `search` mode, each hit automatically receives its clean extract summary.
- **Multi-Language Search Support:** Supports international character sets (e.g. English, Arabic, French, Spanish, etc.).
- **Dynamic Parameter Resolution:** Seamlessly accepts queries or page titles passed dynamically from upstream nodes (as plain strings or objects).
- **Graceful Error & 404 Handling:** Returns structured empty results (`results: []`, `total: 0`) or `null` for non-existent pages instead of crashing the pipeline.

### Common Use Cases

- **AI Research & Knowledge Retrieval (RAG):** Feed real-time encyclopedic context into LLM prompts.
- **Dynamic Topic Enrichment:** Look up incoming entities (companies, landmarks, technologies) and attach background summaries.
- **Geographic Data Extraction:** Retrieve latitude and longitude coordinates for cities, monuments, and historical sites.
- **Automated Fact-Checking & Definitions:** Verify concepts, historical dates, and terminology dynamically in data pipelines.
- **Content Aggregation & Newsletters:** Generate topic digests and educational content automatically.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## How to Use the Wikipedia Node

The Wikipedia node can be configured directly in the visual builder or driven dynamically through incoming workflow data.

![Wikipedia Node Configuration](icon.svg)

### Step-by-Step Setup in the Visual Builder

1. **Add the Node:** Drag the **Wikipedia** node onto the canvas from the **Content & Feeds** palette.
2. **Connect a Trigger or Preceding Node:** Connect a trigger (such as `Manual Trigger`, `Webhook`, or `Schedule`) or an action node (like `Function`) to the `input` port.
3. **Select the Operation:**
   - Choose **`search`** if you want to find articles by keywords.
   - Choose **`getById`** if you have a specific article title and want its complete details.
4. **Configure Parameters:** Fill in the query or title, or leave them empty to read dynamically from upstream data.
5. **Connect Outputs:** Connect the `success` output to downstream nodes (such as `Log`, `AI Chat`, `Function`, or a database action).

---

### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `operation` | `enum` | ❌ No | `search` | Choose between `search` (keyword search) and `getById` (fetch article by title). |
| `query` | `string` | ❌ No* | — | Search keywords (used in `search` operation). *Required if not supplied via input. |
| `page` | `number` | ❌ No | `1` | Page number for pagination (used in `search` operation). |
| `pageSize` | `number` | ❌ No | `20` | Number of results per page (used in `search` operation). |
| `pageTitle` | `string` | ❌ No* | — | Exact article title or page key (used in `getById` operation). *Required if not supplied via input. |

---

### Operations Explained

#### 1. `search` Operation (Default)

Executes a full-text search across Wikipedia and returns a paginated list of matching articles.

- **How it works:**
  1. Calls Wikipedia's MediaWiki search API with your search terms.
  2. Calculates the pagination offset (`offset = (page - 1) * pageSize`).
  3. Iterates over each result hit and fetches the full article extract from the Wikipedia REST summary API.
  4. Returns the structured result array along with `total`, `page`, `pageSize`, and `totalPages`.
- **When no articles match:** Returns `{ "results": [], "total": 0, "page": 1, "pageSize": 20, "totalPages": 0 }`.

#### 2. `getById` Operation

Fetches a single article's full extract, thumbnail image, desktop URL, and coordinates using its exact title.

- **How it works:**
  1. Requests `https://en.wikipedia.org/api/rest_v1/page/summary/{pageTitle}`.
  2. Extracts `title`, `extract` (summary text), `content_urls.desktop.page`, `thumbnail.url`, and `coordinates`.
- **When the page does not exist (404):** Returns `null` instead of throwing an unhandled exception.

---

### Dynamic Value Resolution (Passing Data from Upstream Nodes)

You do not need to hardcode queries or titles in the node configuration. The node automatically inspects the incoming data payload:

```
                  ┌────────────────────────────────────────┐
                  │          Incoming Input Data           │
                  └──────────────────┬─────────────────────┘
                                     │
                    Is Operation "search" or "getById"?
                                     │
            ┌────────────────────────┴────────────────────────┐
            ▼                                                 ▼
      ["search"]                                         ["getById"]
 1. Config `query`                                  1. Config `pageTitle`
 2. Input (if string)                               2. Input (if string)
                                                    3. Input object `pageTitle`
                                                    4. Input object `title`
                                                    5. Input object `id`
```

#### In `search` operation:
1. Uses `query` from node configuration if specified.
2. If `query` is empty, uses incoming `input` data if it is a string (e.g. `"artificial intelligence"` or `"طنجة"`).

#### In `getById` operation:
1. Uses `pageTitle` from node configuration if specified.
2. If empty, uses incoming `input` if it is a string (e.g. `"linux"`).
3. If input is an object, checks for `input.pageTitle` (e.g. `{ "pageTitle": "Morocco" }`).
4. If not found, checks for `input.title` (e.g. `{ "title": "Docker (software)" }`).
5. If not found, checks for `input.id` (e.g. `{ "id": "Python_(programming_language)" }`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional upstream data payload. Supplies `query` for search or `pageTitle` / `title` / `id` for getById. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the operation succeeds. Contains search results or article data. |
| `error` | `Error` | Emitted when validation fails (e.g. missing query/title) or a network/API failure occurs. |

---

### Output Data Schema

#### Schema for `search` Operation

```json
{
  "results": [
    {
      "id": "Artificial_intelligence",
      "title": "Artificial intelligence",
      "authors": [],
      "abstract": "Artificial intelligence is the intelligence of machines or software, as opposed to the intelligence of living beings, primarily of humans.",
      "journal": "Wikipedia",
      "year": "2026",
      "url": "https://en.wikipedia.org/wiki/Artificial_intelligence",
      "keywords": [],
      "metadata": {
        "wordCount": 18450,
        "size": 224100,
        "timestamp": "2026-08-20T14:32:00Z"
      }
    }
  ],
  "total": 45120,
  "page": 1,
  "pageSize": 5,
  "totalPages": 9024
}
```

| Field | Type | Description |
|-------|------|-------------|
| `results` | `array` | List of matching article result objects. |
| `results[].id` | `string` | URL-safe article key (spaces converted to underscores). |
| `results[].title` | `string` | Human-readable title of the Wikipedia article. |
| `results[].abstract` | `string` | Clean plain-text lead extract or snippet of the article. |
| `results[].url` | `string` | Direct link to the article on Wikipedia (`https://en.wikipedia.org/wiki/...`). |
| `results[].metadata.wordCount` | `number` | Total word count of the article page. |
| `results[].metadata.size` | `number` | Size of the page content in bytes. |
| `results[].metadata.timestamp` | `string` | ISO 8601 timestamp of the most recent revision. |
| `total` | `number` | Total number of search hits found across Wikipedia. |
| `page` | `number` | Current page index. |
| `pageSize` | `number` | Number of items per page. |
| `totalPages` | `number` | Total number of available pages. |

---

#### Schema for `getById` Operation

```json
{
  "id": "Python_(programming_language)",
  "title": "Python (programming language)",
  "authors": [],
  "abstract": "Python is a high-level, general-purpose programming language. Its design philosophy emphasizes code readability with the use of significant indentation.",
  "journal": "Wikipedia",
  "year": "2026",
  "url": "https://en.wikipedia.org/wiki/Python_(programming_language)",
  "keywords": [],
  "metadata": {
    "thumbnail": "https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Python-logo-notext.svg/320px-Python-logo-notext.svg.png",
    "coordinates": undefined
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Article title or page key. |
| `title` | `string` | Display title of the article. |
| `abstract` | `string` | Complete lead summary paragraph(s) of the article. |
| `url` | `string` | Full desktop URL to the Wikipedia article. |
| `metadata.thumbnail` | `string \| undefined` | Direct URL to the article's lead thumbnail image (if available). |
| `metadata.coordinates` | `object \| undefined` | Geographic coordinates `{ "lat": number, "lon": number }` for locations/landmarks. |

> **Note:** If an article does not exist, `getById` returns `null`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Practical Examples & Usage Patterns

### Example 1: Basic Search for Articles

Search Wikipedia for articles matching `"artificial intelligence"` and return the top 5 results.

**Node Configuration:**
- **Operation:** `search`
- **Query:** `artificial intelligence`
- **PageSize:** `5`

**Output:**
```json
{
  "results": [
    {
      "id": "Artificial_intelligence",
      "title": "Artificial intelligence",
      "abstract": "Artificial intelligence is the intelligence of machines or software...",
      "url": "https://en.wikipedia.org/wiki/Artificial_intelligence",
      "metadata": {
        "wordCount": 18450,
        "size": 224100,
        "timestamp": "2026-08-20T14:32:00Z"
      }
    }
  ],
  "total": 45120,
  "page": 1,
  "pageSize": 5,
  "totalPages": 9024
}
```

---

### Example 2: Search with Pagination

Retrieve the 2nd page of results for `"SpaceX"`, 3 items per page.

**Node Configuration:**
- **Operation:** `search`
- **Query:** `SpaceX`
- **Page:** `2`
- **PageSize:** `3`

---

### Example 3: Multi-Language Search (Arabic / Non-Latin Characters)

Search Wikipedia for Arabic topics such as `"طنجة"` (Tangier).

**Node Configuration:**
- **Operation:** `search`
- **Query:** `طنجة`
- **PageSize:** `3`

**Output:**
```json
{
  "results": [
    {
      "id": "Tangier",
      "title": "Tangier",
      "abstract": "Tangier is a city in northwestern Morocco...",
      "url": "https://en.wikipedia.org/wiki/Tangier",
      "metadata": {
        "wordCount": 7820,
        "size": 95400,
        "timestamp": "2026-08-15T09:12:00Z"
      }
    }
  ],
  "total": 142,
  "page": 1,
  "pageSize": 3,
  "totalPages": 48
}
```

---

### Example 4: Direct Article Lookup by Exact Title (`getById`)

Retrieve detailed information, thumbnail, and coordinates for `"Morocco"`.

**Node Configuration:**
- **Operation:** `getById`
- **PageTitle:** `Morocco`

**Output:**
```json
{
  "id": "Morocco",
  "title": "Morocco",
  "authors": [],
  "abstract": "Morocco, officially the Kingdom of Morocco, is a country in the Maghreb region of North Africa.",
  "journal": "Wikipedia",
  "year": "2026",
  "url": "https://en.wikipedia.org/wiki/Morocco",
  "keywords": [],
  "metadata": {
    "thumbnail": "https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Flag_of_Morocco.svg/320px-Flag_of_Morocco.svg.png",
    "coordinates": {
      "lat": 31.7917,
      "lon": -7.0926
    }
  }
}
```

---

### Example 5: Dynamic Lookup via Function Node (String Input)

An upstream `Function` node outputs a plain string `"linux"`.

**Upstream Function Node Code:**
```javascript
return "linux";
```

**Wikipedia Node Configuration:**
- **Operation:** `getById`
- **PageTitle:** *(leave empty)*

The Wikipedia node receives `"linux"` and automatically fetches the Linux article summary.

---

### Example 6: Dynamic Lookup via Function Node (Object Input)

An upstream `Function` node outputs an object containing a `title` field.

**Upstream Function Node Code:**
```javascript
return {
  "title": "Docker (software)"
};
```

**Wikipedia Node Configuration:**
- **Operation:** `getById`
- **PageTitle:** *(leave empty)*

The Wikipedia node extracts `title` (`"Docker (software)"`) from the incoming object and returns the full Docker summary.

---

### Example 7: Handling Non-Existent Articles (404)

Looking up an article title that does not exist (e.g. `"random_non_existent_page_123456789"`).

**Node Configuration:**
- **Operation:** `getById`
- **PageTitle:** `random_non_existent_page_123456789`

**Output:**
```json
null
```

> **Tip:** You can connect an **If** or **Guard** node downstream to check whether the result is not `null` before continuing the workflow.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Interactive Workflow Preview

```fusion-workflow
src: example.workflow.json
title: Wikipedia Search and Retrieval Workflow
```

---

### Sample Workflows

#### 1. Research Pipeline: Trigger ➔ Wikipedia Search ➔ Log

A workflow that triggers manually, searches Wikipedia for `"Artificial intelligence"`, and outputs the top 5 articles with full summaries to the log:

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger",
      "label": "Start Research"
    },
    {
      "id": "search-wikipedia",
      "type": "wikipedia",
      "config": {
        "operation": "search",
        "query": "Artificial intelligence",
        "pageSize": 5
      }
    },
    {
      "id": "display-results",
      "type": "log",
      "label": "Log Results"
    }
  ],
  "connections": [
    {
      "source": "trigger",
      "target": "search-wikipedia"
    },
    {
      "source": "search-wikipedia",
      "target": "display-results"
    }
  ]
}
```

---

#### 2. Dynamic Enrichment Pipeline: Function ➔ Wikipedia GetById

A workflow where a JavaScript `Function` node computes or extracts a topic, and passes it dynamically to Wikipedia for full summary and thumbnail lookup:

```json
{
  "nodes": [
    {
      "id": "start",
      "type": "manual-trigger"
    },
    {
      "id": "prepare-topic",
      "type": "function",
      "config": {
        "code": "return { title: 'Docker (software)' };"
      }
    },
    {
      "id": "get-article",
      "type": "wikipedia",
      "config": {
        "operation": "getById"
      }
    }
  ],
  "connections": [
    {
      "source": "start",
      "target": "prepare-topic"
    },
    {
      "source": "prepare-topic",
      "target": "get-article"
    }
  ]
}
```

---

### Architecture Patterns

- **RAG & AI Prompt Context:** `User Query ➔ Wikipedia (search) ➔ Function (Extract top 3 abstracts) ➔ AI Chat (System context + Query)`.
- **Entity Enrichment:** `Webhook (New Company/Place) ➔ Wikipedia (getById) ➔ Notion / PostgreSQL (Save summary + thumbnail)`.
- **Location Mapping:** `Form Input (City Name) ➔ Wikipedia (getById) ➔ Map Viewer (Render coordinates: lat, lon)`.
- **Fact Verification:** `AI Agent Tool ➔ Wikipedia (search & getById) ➔ Evaluator (Check ground truth)`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors & Solutions

#### `Wikipedia search failed: Search query is required`
- **Cause:** The node is set to `search` mode, but `query` is empty in the configuration and no string input was provided by the upstream node.
- **Solution:** Enter a search query in the node parameters or ensure the preceding node emits a non-empty string payload.

---

#### `Wikipedia getById failed: Page title is required for getById operation`
- **Cause:** The node is set to `getById` mode, but `pageTitle` is empty in configuration and the incoming data did not contain a valid string or object with `pageTitle`, `title`, or `id`.
- **Solution:** Provide `pageTitle` in the node configuration, or ensure the upstream node passes a string or an object containing `{ "title": "..." }`, `{ "pageTitle": "..." }`, or `{ "id": "..." }`.

---

#### `Output is null`
- **Cause:** The requested article was not found on Wikipedia (HTTP 404).
- **Solution:** Check the title spelling and casing. For broad topics, use the `search` operation first to discover the exact page title, then pass the title to `getById`.

---

#### `Wikipedia API error: 429 Too Many Requests`
- **Cause:** Rate limits reached on Wikipedia public API due to sending too many concurrent requests.
- **Solution:** Add a `Delay` or `Throttle` node before Wikipedia in high-volume batch loops.

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Search query is required` | Missing query in search mode | Provide a `query` parameter or pass string input. |
| `Page title is required for getById operation` | Missing title in getById mode | Set `pageTitle` or pass `title`/`pageTitle`/`id` object. |
| `Unknown operation: <operation>` | Unsupported operation name | Select either `search` or `getById`. |
| `Wikipedia API error: <status> <statusText>` | Wikipedia endpoint error | Check network connectivity or retry the request. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Function](../function/en.md) — Transform, format, and filter Wikipedia article payloads
- [Manual Trigger](../manual-trigger/en.md) — Manually initiate Wikipedia test workflows
- [Google Search](../google-search/en.md) — Search the broader web via Google
- [DuckDuckGo](../duck-duck-go-search/en.md) — Query instant answers and web links via DuckDuckGo
- [ArXiv](../arxiv/en.md) — Search academic publications and scientific preprints

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial release with `search` and `getById` operations, dynamic input resolution, and automated summary enrichment |

<!-- /SECTION: changelog -->

---
node_id: "wiki-data"
title: "WikiData"
description: "Search for Entity IDs in WikiData knowledge base."
category: "Web Search & Information"
subcategory: "Reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-28"
author: "Fusion Team"
tags:
  - wikidata
  - wikipedia
  - knowledge-base
  - entity-resolution
  - linked-data
  - semantic-web
  - reference
related_nodes:
  - rest-countries
  - geo-names
  - openalex
  - semantic-scholar
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# WikiData

> **Category:** Web Search & Information | **Subcategory:** Reference | **Type:** Action Node

Search the global **WikiData** knowledge base to resolve canonical Entity IDs (Q-IDs), retrieve multilingual labels, descriptions, aliases, and semantic concept URIs.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **WikiData** node connects workflows to the official Wikimedia Wikibase API (`https://www.wikidata.org/w/api.php?action=wbsearchentities`). It performs real-time entity resolution across millions of structured concepts, including people, organizations, geographic locations, scientific terms, and historical events.

Each matched entity returns its permanent WikiData Q-ID (such as `Q7331` for *Ibn Battuta* or `Q1028` for *Morocco*), along with official descriptions, aliases, and linked data concept URIs.

### Key Features

- **Entity ID Resolution (Q-IDs):** Converts plain text search strings into unambiguous, globally unique WikiData Q-identifiers.
- **Multilingual Support:** Supports localized searches and returns labels and descriptions in any language (e.g. English `en`, Arabic `ar`, French `fr`, Spanish `es`, etc.).
- **Rich Semantic Metadata:** Supplies canonical concept URIs (`http://www.wikidata.org/entity/Q...`), direct web URLs, page IDs, and alias lists.
- **Configurable Result Limits & Pagination:** Returns between 1 and 50 results per request and supports `continue` offsets for paginated fetching.
- **No API Key Required:** Connects directly to the public, open Wikimedia API without requiring authentication tokens or registration.

### Use Cases

- **AI Knowledge Grounding & RAG:** Resolve user queries into verified WikiData entity IDs before generating knowledge graph queries or vector embeddings.
- **Entity Disambiguation:** Differentiate between distinct concepts with identical names (e.g., *Apple* the corporation vs. *Apple* the fruit).
- **Multilingual Term Translation:** Query an entity name in English and retrieve its canonical name and description in Arabic, French, or Japanese.
- **Automated Content Classification:** Tag blog articles, documents, and database entries with standardized semantic identifiers.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `search` | `string` | Conditional | — | The entity keyword or name to search (e.g., `Ibn Battuta`, `Casablanca`, `Python`). If omitted, incoming data from `input` is used. |
| `language` | `string` | ❌ No | `"en"` | ISO language code for search matching and localized labels (e.g., `en`, `ar`, `fr`, `es`). |
| `format` | `string` | ❌ No | `"json"` | Response serialization format from the MediaWiki API. |
| `limit` | `number` | ❌ No | `10` | Maximum number of results to return (clamped between `1` and `50`). |
| `continue` | `number` | ❌ No | `0` | Offset index for paginated result retrieval. |

---

### Parameter Details

#### `search`
The search keyword or entity name to look up in the WikiData knowledge base.
- If left empty, the node automatically uses the string value passed into its `input` port.

#### `language`
Specifies the language used for text search matching and for the localized `label` and `description` returned in results.
- Examples: `"en"` (English), `"ar"` (Arabic), `"fr"` (French), `"es"` (Spanish), `"de"` (German).

#### `limit`
The maximum number of matching entities to return.
- The node automatically clamps the value to a safe range: `Math.min(Math.max(limit, 1), 50)`.

#### `continue`
The starting offset for pagination. Set to `0` for the first page, or pass the offset to retrieve subsequent pages of results.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. If the `search` parameter is empty, `input` (or `String(input)`) is used as the search term. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when WikiData successfully processes the search query. Contains an array of matched entities and query metadata. |
| `error` | `Error` | Emitted when the search query is missing or if the WikiData API returns a network or HTTP error. |

---

### Output Data Structure

```json
{
  "success": true,
  "query": "Ibn Battuta",
  "searchinfo": {
    "search": "Ibn Battuta"
  },
  "results": [
    {
      "id": "Q7331",
      "title": "Q7331",
      "pageid": 8515,
      "display": {
        "label": {
          "value": "ابن بطوطة",
          "language": "ar"
        },
        "description": {
          "value": "رحالة ومؤرخ وفقيه مغربي",
          "language": "ar"
        }
      },
      "repository": "wikidata",
      "url": "//www.wikidata.org/wiki/Q7331",
      "concepturi": "http://www.wikidata.org/entity/Q7331",
      "label": "ابن بطوطة",
      "description": "رحالة ومؤرخ وفقيه مغربي",
      "aliases": [
        "شمس الدين ابن بطوطة",
        "محمد بن عبد الله بن محمد اللواتي الطنجي"
      ],
      "match": {
        "type": "label",
        "language": "ar",
        "text": "ابن بطوطة"
      }
    }
  ],
  "total_results": 1,
  "api_success": 1
}
```

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` if the node executed successfully |
| `query` | `string` | The original search query string |
| `searchinfo` | `object` | Metadata returned by the WikiData search engine |
| `total_results` | `number` | Number of entities returned in the `results` array |
| `api_success` | `number` | API status flag from MediaWiki (`1` on success) |
| `results[].id` | `string` | The unique WikiData Entity ID (e.g. `Q7331`) |
| `results[].label` | `string` | Canonical entity name in the requested language |
| `results[].description` | `string` | Short definition or summary of the entity |
| `results[].aliases` | `array` | Alternative names, spellings, or titles |
| `results[].concepturi` | `string` | Full semantic linked-data URI (`http://www.wikidata.org/entity/Q...`) |
| `results[].url` | `string` | Relative URL to the WikiData entity page |
| `results[].pageid` | `number` | Internal MediaWiki numeric page ID |
| `results[].match` | `object` | Details on how the query matched the entity (type, language, text) |
| `results[].display` | `object` | Language-specific label and description objects |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Multilingual Entity Search (Arabic)

Search for *Ibn Battuta* and retrieve Arabic labels and descriptions.

**Parameter Configuration:**

```text
Search:   Ibn Battuta
Language: ar
Limit:    5
```

---

### Example 2: Dynamic Entity Lookup from Webhook or LLM

Extract a keyword from an upstream prompt and resolve its WikiData Q-ID.

**Workflow Pattern:**

```text
Webhook Trigger (receives: { "keyword": "Casablanca" })
  → Function (prepare query)
  → WikiData (search: {{Function.keyword}}, language: "fr", limit: 3)
  → Function (extract results[0].id and concepturi)
  → Log
```

---

### Example 3: Disambiguation & Semantic Grounding

Search for ambiguous terms (e.g., `"Python"`) to list distinct entities (programming language, snake, mythology).

**Parameter Configuration:**

```text
Search:   Python
Language: en
Limit:    10
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search entity IDs in WikiData knowledge base
```

### Common Patterns

- **Semantic Enrichment:** User Prompt → WikiData → Extract `id` & `description` → Inject into AI Agent System Prompt.
- **Multilingual Knowledge Base:** Product / Entity Name → WikiData (Language: `ar` / `fr`) → Store localized names.
- **SPARQL Preparation:** Search Term → WikiData → Extract `Q-ID` → SPARQL Query Node.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `Search query is required`

**Cause:** The `search` parameter was empty and no string data was sent through the `input` port.

**Solution:** Enter a search query in the node parameters or ensure the upstream node sends a non-empty string.

#### Error: `WikiData search failed: WikiData API error: 503 / 504`

**Cause:** WikiData API rate limiting or temporary upstream Wikimedia service outage.

**Solution:** Implement retry logic or a brief delay between high-frequency automated requests.

#### Issue: No Results Returned (`total_results: 0`)

**Cause:** The search query does not match any existing label or alias in the specified `language`.

**Solution:** Try querying in English (`language: "en"`) or verify spelling variations.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Search query is required` | Missing search term | Provide `search` parameter or input data |
| `WikiData API error: <status>` | Wikimedia HTTP error | Check network connectivity and retry |
| `WikiData search failed: <error>` | General fetch error | Verify query formatting |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [REST Countries](../rest-countries/en.md) — Query country metadata, capitals, and currencies
- [GeoNames](../geo-names/en.md) — Search geographical places and coordinates
- [OpenAlex](../openalex/en.md) — Search scholarly research papers and authors
- [Semantic Scholar](../semantic-scholar/en.md) — Academic paper and citation discovery
- [HTTP Request](../http-request/en.md) — Make custom API calls
- [Function](../function/en.md) — Parse and filter WikiData entities
- [Log](../log/en.md) — Output entity search results in console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-28 | Initial release |

<!-- /SECTION: changelog -->

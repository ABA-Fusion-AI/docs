---
node_id: "ffxiv"
title: "FFXIV API"
description: "Search for Final Fantasy XIV content including items, quests, titles, achievements, and actions using the XIVAPI."
category: "utilities-misc"
subcategory: "games"
version: "1.0.0"
language: "en"
last_updated: "2026-09-16"
author: "Fusion Team"
tags:
  - ffxiv
  - final-fantasy
  - xivapi
  - gaming
  - games
  - search
  - items
  - quests
related_nodes:
  - dn-d5e
  - poke-api
  - http-request
  - log
---

<!-- SECTION: header -->

# FFXIV API

> **Category:** Utilities & Misc | **Subcategory:** Games | **Type:** Action Node

Search and retrieve Final Fantasy XIV game content from the XIVAPI service, including items, quests, titles, achievements, recipes, NPCs, and actions.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **FFXIV API** node integrates with [XIVAPI](https://xivapi.com), a comprehensive REST API service for Final Fantasy XIV game data. It allows workflows to perform fast, structured searches across the FFXIV game database.

You can configure static searches in the node parameters or dynamically pass search queries through incoming workflow data. Searches can span all game content or be targeted to specific indexes such as items, quests, or achievements.

### Key Features

- **Global & Targeted Search:** Search across all game data or restrict queries to specific content categories (e.g., `Item`, `Quest`, `Title`).
- **Dynamic Query Fallback:** Accepts queries configured directly in the node parameters or passed dynamically from previous nodes in the workflow.
- **Configurable Result Limits:** Control the number of results returned per query (up to 100 by default).
- **Optional API Key Support:** Connect without authentication for standard rate limits, or provide an XIVAPI key for higher throughput.
- **Rich Result Payloads:** Returns structured data including item IDs, names, icons, URLs, pagination info, and result counts.
- **Robust Error Handling:** Surfaces detailed HTTP error messages and XIVAPI operational errors for straightforward workflow debugging.

### Use Cases

- **Discord & Community Bots:** Allow players to query item stats, quest requirements, or achievement details directly from chat commands.
- **Crafting & Market Automations:** Look up item IDs and recipe data to feed into crafting cost calculators or market board monitors.
- **Game Content Discovery:** Build search interfaces or automated lookup tools for guild portals and companion apps.
- **AI Agent Tooling:** Provide game knowledge tools to LLMs and AI agents answering Final Fantasy XIV player queries.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | ❌ No | — | Search term for FFXIV content (e.g., `"Excalibur"`, `"Potion"`). If omitted, the node falls back to the incoming workflow data. |
| `apiKey` | `string` | ❌ No | — | Optional XIVAPI API key. Recommended when running high-frequency workflows to benefit from elevated rate limits. |
| `indexes` | `string` | ❌ No | — | Comma-separated list of XIVAPI indexes/sheets to search within (e.g., `"Item,Quest,Title"`). If omitted, the search runs across all available indexes. |
| `limit` | `number` | ❌ No | `100` | Maximum number of results to return (default: `100`). |

---

### Parameter Details

#### `query`

The text string to look up in the XIVAPI database.

- If provided in the configuration, this value takes precedence.
- If left blank or undefined, the node inspects the incoming data payload from the previous node. If the incoming data is a string, it is used directly; otherwise, it is converted to a string representation via `String(data)`.
- If neither the configuration nor the incoming data provides a non-empty query, the node fails with:
  ```text
  Search query is required
  ```

#### `apiKey`

An optional personal API key obtained from [XIVAPI](https://xivapi.com).

- Without a key, requests are subject to XIVAPI's standard anonymous rate limit (typically up to 12 requests per second).
- Providing an API key attaches `key=<apiKey>` to the query string and provides higher throughput for production workflows.

#### `indexes`

A comma-delimited string specifying which game sheets to search.

Common indexes include:

| Index Name | Content Type | Example Search Query |
|------------|--------------|----------------------|
| `Item` | Equipment, consumables, materials, currencies | `"Excalibur"`, `"Hi-Potion"` |
| `Quest` | Main scenario, side quests, job quests | `"The Ultimate Weapon"` |
| `Title` | Player character titles | `"The Legend"`, `"Postman"` |
| `Achievement` | In-game achievements and milestones | `"Mapping the Realm"` |
| `Action` | Class, job, and general combat skills | `"Cure"`, `"Fire IV"` |
| `Recipe` | Crafting recipes and synthesis formulas | `"Titanium Ingot"` |
| `NPC` / `ENpcResident` | Non-player characters across Eorzea | `"Tataru"`, `"Alphinaud"` |
| `Fate` | Full Active Time Events | `"The Eyes Have It"` |
| `InstanceContent` | Dungeons, raids, and trials | `"The Praetorium"` |

If `indexes` is left blank, the API performs a global search across all searchable game indexes.

#### `limit`

Controls the number of matching records returned in the `results` array.

- Default value: `100`.
- Reduce this number (e.g., `5` or `10`) when only top matches are required, reducing payload size and downstream processing time.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `input` | `string` \| `any` | Conditional | Incoming workflow data. If `query` is not set in the node configuration, this input is used as the search query. Simple strings are used directly, while non-string objects or primitives are converted via `String(data)`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the search query is successfully resolved and processed by XIVAPI. |
| `error` | `Error` | Emitted when the query is empty, network requests fail, or the API returns an error response. |

---

### Output Schema (`success`)

When a query succeeds, the node emits an object with the following structure:

```json
{
  "success": true,
  "query": "Excalibur",
  "results": [
    {
      "ID": 1601,
      "Icon": "/i/001000/001552.png",
      "Name": "Excalibur",
      "Url": "/Item/1601",
      "UrlType": "Item",
      "_": "item"
    }
  ],
  "pagination": {
    "Page": 1,
    "PageNext": 2,
    "PagePrev": null,
    "PageTotal": 4,
    "Results": 100,
    "ResultsPerPage": 100,
    "ResultsTotal": 352
  },
  "total_results": 100
}
```

#### Output Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Always `true` on successful queries. |
| `query` | `string` | The exact search string that was queried. |
| `results` | `array` | List of content objects matching the search criteria. |
| `results[].ID` | `number` | Unique numeric identifier of the entity within its XIVAPI sheet. |
| `results[].Name` | `string` | Display name of the item, quest, title, or asset. |
| `results[].Icon` | `string` | Relative icon path on XIVAPI CDN (e.g., `https://xivapi.com/i/001000/001552.png`). |
| `results[].Url` | `string` | Relative API path to fetch full record details. |
| `results[].UrlType` | `string` | The index/sheet type of this entity (e.g., `"Item"`, `"Quest"`). |
| `pagination` | `object` | Detailed page information returned by XIVAPI. |
| `pagination.Page` | `number` | Current page number. |
| `pagination.PageTotal` | `number` | Total number of pages available for this query. |
| `pagination.ResultsTotal` | `number` | Total count of all matching records across all pages. |
| `total_results` | `number` | The number of results included in this output batch (`results.length`). |

---

### Output Schema (`error`)

If a query fails, an error is emitted with an informative message:

```text
FFXIV search failed: <detailed error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Example 1: Search for an Item

Search for weapons matching `"Excalibur"` specifically within the `Item` index:

```json
{
  "query": "Excalibur",
  "indexes": "Item",
  "limit": 5
}
```

**Output:**

```json
{
  "success": true,
  "query": "Excalibur",
  "results": [
    {
      "ID": 1601,
      "Icon": "/i/001000/001552.png",
      "Name": "Excalibur",
      "Url": "/Item/1601",
      "UrlType": "Item",
      "_": "item"
    },
    {
      "ID": 9491,
      "Icon": "/i/031000/031554.png",
      "Name": "Excalibur Zeta",
      "Url": "/Item/9491",
      "UrlType": "Item",
      "_": "item"
    }
  ],
  "pagination": {
    "Page": 1,
    "PageNext": null,
    "PagePrev": null,
    "PageTotal": 1,
    "Results": 2,
    "ResultsPerPage": 5,
    "ResultsTotal": 2
  },
  "total_results": 2
}
```

---

### Example 2: Multi-Index Content Search

Search for lore, achievements, and titles associated with `"Bahamut"`:

```json
{
  "query": "Bahamut",
  "indexes": "Quest,Achievement,Title,Action",
  "limit": 10
}
```

---

### Example 3: Dynamic Search from Workflow Trigger

Leave the `query` field empty in the node configuration:

```json
{
  "indexes": "Item",
  "limit": 20
}
```

When a webhook or chat trigger sends `"Super Potion"`, the node dynamically reads the incoming string and executes the search for `"Super Potion"`.

---

### Example 4: High-Throughput Search with API Key

Use an explicit XIVAPI API key for higher rate limits in production workflows:

```json
{
  "query": "Titanium",
  "indexes": "Item,Recipe",
  "limit": 50,
  "apiKey": "your_xivapi_key_here"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Final Fantasy XIV content
```

### Workflow Structure

```text
Manual Trigger ──(success)──> FFXIV API ──(success)──> Log
```

1. **Manual Trigger:** Triggers the workflow execution on demand.
2. **FFXIV API:** Searches for `"Excalibur"` filtered to the `"Item"` index with a result limit of `10`.
3. **Log:** Displays the structured results, pagination metadata, and total results in the workflow console for inspection.

### Common Patterns

- **Chatbot Command Integration:**
  ```text
  Discord Trigger / Slack Trigger ──> FFXIV API ──> Format Message ──> Discord Send
  ```
  Extracts the user's search term, queries XIVAPI, formats item icons and URLs into an embed, and posts the reply.

- **AI Agent Tooling:**
  Connect the **FFXIV API** node into an AI Agent or LLM Tool configuration to give conversational assistants real-time access to Final Fantasy XIV game knowledge.

- **Bulk Asset Enrichment:**
  ```text
  Database Query / CSV Read ──> Loop ──> FFXIV API ──> Update Database
  ```
  Enriches item inventories with official game icons and asset URLs from XIVAPI.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Search query is required

**Cause:** The `query` parameter in the node configuration is empty or whitespace-only, and no valid query string was provided in the incoming workflow data.

**Solution:** Provide a query in the node parameters, or ensure upstream nodes (e.g., triggers, forms, or webhook payloads) pass a non-empty string to this node.

---

#### `FFXIV API error: 429 Too Many Requests`

**Cause:** You have exceeded the public rate limit of the XIVAPI public tier.

**Solution:**
- Obtain a free API key from [XIVAPI](https://xivapi.com) and configure it in the `apiKey` parameter.
- Use a **Delay** or **Throttle** node upstream if processing queries in a batch or loop.

---

#### Unexpected string conversion: `[object Object]`

**Cause:** When `query` is not set in the configuration, the node converts the incoming workflow data using `String(data)`. If the upstream node sends a JSON object (e.g., `{ "search": "Potion" }`), it turns into `"[object Object]"`.

**Solution:** Use a **Function** node or **Select Fields** node upstream to extract the string property before passing it to the FFXIV API node, or configure the query expression directly.

---

#### No results found

**Cause:** The query string has typos, or the specified `indexes` filter does not contain the content you are looking for (e.g., searching for an achievement while `indexes` is set to `"Item"`).

**Solution:** Verify the spelling of the search term or clear the `indexes` parameter to search globally across all content types.

---

### Behavior Reference

| Symptom | Cause | Resolution |
|---------|-------|------------|
| `Search query is required` | Both config `query` and incoming `data` are missing or empty | Provide a search term in config or feed a valid string from an upstream node |
| `FFXIV API error: 429 Too Many Requests` | Exceeded anonymous rate limit | Add an `apiKey` or introduce rate limiting upstream |
| Query searches for `[object Object]` | Upstream node passed an object instead of a string | Extract the target string property before passing data to the node |
| Empty `results` array | No items match query and index combination | Check spelling or expand `indexes` |
| `FFXIV API error: <status>` | XIVAPI server error or connection failure | Check [XIVAPI status](https://xivapi.com) or inspect network connectivity |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [D&D 5e API](./dn-d5e.md) – Query Dungeons & Dragons 5th Edition rules and game reference data
- [PokéAPI](./poke-api.md) – Search and retrieve Pokémon and generation information
- [HTTP Request](../../../automation/nodes/http-request/en.md) – Execute custom HTTP requests against external REST APIs
- [Log](../../../automation/nodes/log/en.md) – Log and inspect output payloads within workflows

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-16 | Initial release of the FFXIV API node documentation |

<!-- /SECTION: changelog -->

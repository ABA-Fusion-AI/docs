---
node_id: "usda-search-foods"
title: "USDA Search Foods"
description: "Search for foods in FoodData Central database with filters"
category: "healthcare-life-sciences"
subcategory: "food-data"
version: "1.0.0"
language: "en"
last_updated: "2026-09-10"
author: "Fusion Team"
tags:
  - "usda"
  - "food"
  - "nutrition"
  - "fooddata-central"
  - "search"
  - "healthcare"
related_nodes:
  - "usda-get-food-by-id"
  - "usda-get-foods-list"
---

<!-- SECTION: header -->

# USDA Search Foods

> **Category:** Healthcare & Life Sciences / Food Data | **Type:** Action Node

Search for foods in USDA FoodData Central using a text query and optional filters.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **USDA Search Foods** node searches the USDA FoodData Central database using a text query and optional filtering, sorting, and pagination parameters.

### Key Features

- Search foods by keyword
- Filter results by USDA food data type
- Support multiple food data types in the same search
- Control the number of results returned per page
- Navigate through paginated search results
- Sort results by supported USDA fields
- Control ascending or descending sort order
- Use `DEMO_KEY` as the default USDA API key
- Handle common USDA API and network errors

### Processing Flow

1. Provide a search query.
2. Provide a USDA API key or use `DEMO_KEY`.
3. Optionally select one or more food data types.
4. Configure pagination and sorting parameters when needed.
5. The node sends the search request to USDA FoodData Central.
6. USDA returns matching foods and pagination information.
7. The node returns the search results through the success output.

### Use Cases

- Search USDA FoodData Central by food name
- Find branded or foundation food records
- Build food lookup and nutrition workflows
- Filter foods by USDA data type
- Browse large food datasets using pagination
- Retrieve FoodData Central IDs before requesting detailed food information

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `query` | String | Yes | — | Search text used to find foods in USDA FoodData Central. |
| `apiKey` | String | Yes | `DEMO_KEY` | USDA FoodData Central API key. |
| `dataType` | Array | No | — | Optional list of USDA food data types to include in the search. |
| `pageSize` | Number | No | `50` | Number of results returned per page. Must be between `1` and `200`. |
| `pageNumber` | Number | No | `1` | Page number to retrieve. Must be greater than or equal to `1`. |
| `sortBy` | String | No | — | Field used to sort the search results. |
| `sortOrder` | String | No | `asc` | Sort direction. Supported values are `asc` and `desc`. |

### Supported Data Types

The `dataType` parameter supports:

- `Foundation`
- `SR Legacy`
- `Survey (FNDDS)`
- `Branded`

Multiple values can be selected in the same request.

### Supported Sort Fields

The `sortBy` parameter supports:

- `dataType.keyword`
- `lowercaseDescription.keyword`
- `fdcId`
- `publishedDate`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|---|---|---|
| `input` | Any | Optional incoming workflow value. Search behavior is controlled by the configured node parameters. |

### Outputs

| Output | Type | Description |
|---|---|---|
| `success` | Object | Returns the USDA food search results and pagination information. |
| `error` | Error | Returns validation, API, network, or USDA service errors. |

### Success Output

The success output can contain:

- `query`
- `total_hits`
- `current_page`
- `total_pages`
- `page_size`
- `returned_results`
- `foods`

Each returned food can contain information such as:

- `fdc_id`
- `description`
- `data_type`
- `publication_date`
- `brand_owner`
- `gtin_upc`
- `ingredients`
- `food_category`
- `serving_size`
- `serving_size_unit`

Available fields can vary depending on the USDA food data type.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Search Foods

Search for `apple` across Branded and Foundation food records.

**Parameters:**

```json
{
  "query": "apple",
  "apiKey": "DEMO_KEY",
  "dataType": [
    "Branded",
    "Foundation"
  ],
  "pageSize": 200,
  "pageNumber": 1,
  "sortBy": "dataType.keyword",
  "sortOrder": "asc"
}
```

This example searches USDA FoodData Central for foods matching `apple` across the selected food data types.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: USDA Search Foods Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Empty Search Query

**Cause:** The `query` parameter is empty or missing.

**Solution:** Provide a non-empty search term such as `apple`.

### Invalid Page Size

**Cause:** `pageSize` is lower than `1` or greater than `200`.

**Solution:** Use a value between `1` and `200`.

### Invalid Page Number

**Cause:** `pageNumber` is lower than `1`.

**Solution:** Use a page number greater than or equal to `1`.

### No Results Found

**Cause:** The search query or selected filters do not match any USDA food records.

**Solution:** Try a broader query or remove restrictive `dataType` filters.

### Invalid API Key

**Cause:** The USDA API key is invalid or its quota has been exceeded.

**Solution:** Provide a valid USDA FoodData Central API key or use `DEMO_KEY` for testing.

### Rate Limit Error

**Cause:** Too many requests were sent to the USDA API.

**Solution:** Reduce the request frequency and retry later.

### USDA Server Error

**Cause:** USDA FoodData Central returned a temporary server or network error.

**Solution:** Retry the workflow after a short delay.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **USDA Get Food by ID** — Retrieve detailed information for a specific FoodData Central ID.
- **USDA Get Foods List** — Retrieve multiple foods by their FoodData Central IDs.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-10` | Initial release |

<!-- /SECTION: changelog -->
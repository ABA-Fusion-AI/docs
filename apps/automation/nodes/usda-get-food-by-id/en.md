---
node_id: "usda-get-food-by-id"
title: "USDA Get Food by ID"
description: "Get detailed food information by FoodData Central ID"
category: "healthcare-life-sciences"
subcategory: "food-data"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - "usda"
  - "food"
  - "nutrition"
  - "fooddata-central"
  - "healthcare"
related_nodes:
  - "usda-get-foods-list"
  - "usda-search-foods"
---

<!-- SECTION: header -->

# USDA Get Food by ID

> **Category:** Healthcare & Life Sciences / Food Data | **Type:** Action Node

Retrieve detailed food information from USDA FoodData Central using a FoodData Central ID.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **USDA Get Food by ID** node retrieves detailed information for a specific food item from the USDA FoodData Central API using its FoodData Central ID (`fdcId`).

### Key Features

- Retrieve a food item by FoodData Central ID
- Access food description, data type, publication date, and category
- Retrieve brand, ingredient, and serving information when available
- Retrieve detailed nutrient information
- Use `DEMO_KEY` as the default USDA API key
- Handle common USDA API and network errors
- Retry temporary failures automatically

### Processing Flow

1. Provide a valid FoodData Central ID.
2. Provide a USDA API key or use `DEMO_KEY`.
3. The node requests the food record from USDA FoodData Central.
4. The API response is normalized into structured food and nutrient data.
5. The result is returned through the success output.

### Use Cases

- Retrieve nutritional information for a known food
- Enrich food records with USDA data
- Build nutrition analysis workflows
- Retrieve food metadata for healthcare applications
- Integrate USDA FoodData Central with automated workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `fdcId` | Number | Yes | — | FoodData Central ID of the food item. Must be greater than or equal to `1`. |
| `apiKey` | String | Yes | `DEMO_KEY` | USDA FoodData Central API key. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|---|---|---|
| `input` | Any | Optional incoming value. A numeric value or numeric string can be used as the FoodData Central ID when `fdcId` is not configured. |

### Outputs

| Output | Type | Description |
|---|---|---|
| `success` | Object | Returns the normalized USDA food details and nutrient information. |
| `error` | Error | Returns validation, API, network, or USDA service errors. |

### Success Output

The success output can contain:

- `fdc_id`
- `description`
- `data_type`
- `publication_date`
- `brand_owner`
- `gtin_upc`
- `ingredients`
- `serving_size`
- `serving_size_unit`
- `household_serving`
- `food_category`
- `nutrients`

Each nutrient can include:

- `id`
- `name`
- `number`
- `value`
- `unit`
- `derivation`

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Get Food Details

Retrieve detailed information for the USDA FoodData Central food with ID `1750340`.

**Parameters:**

```json
{
  "fdcId": 1750340,
  "apiKey": "DEMO_KEY"
}
```

This example retrieves information for **Apples, fuji, with skin, raw**, including its food category and nutrient data.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: USDA Get Food by ID Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Invalid FoodData Central ID

**Cause:** The provided `fdcId` is missing or lower than `1`.

**Solution:** Provide a valid FoodData Central ID greater than or equal to `1`.

### Food Not Found

**Cause:** The FoodData Central ID does not correspond to an available food record.

**Solution:** Verify the `fdcId` and retry with a valid FoodData Central food ID.

### Invalid API Key

**Cause:** The USDA API key is invalid or its quota has been exceeded.

**Solution:** Provide a valid USDA FoodData Central API key or use `DEMO_KEY` for testing.

### Rate Limit Error

**Cause:** Too many requests were sent to the USDA API.

**Solution:** Reduce the request frequency and retry later. The node retries temporary rate-limit errors automatically.

### USDA Server Error

**Cause:** USDA FoodData Central returned a temporary server error.

**Solution:** Retry the workflow. The node automatically retries supported temporary server and network failures.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **USDA Get Foods List** — Retrieve multiple foods by their FoodData Central IDs.
- **USDA Search Foods** — Search for foods in USDA FoodData Central.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-02` | Initial release |

<!-- /SECTION: changelog -->
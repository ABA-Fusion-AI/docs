---
node_id: "perenual-info"
title: "Perenual Info"
description: "Get plant care guides, watering needs, and sunlight info."
category: "Botany / Plant Care"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:

- perenual
- plants
- botany
- plant-care
- gardening
- watering
- sunlight
- species-search
- api

related_nodes:
- function
- if
- http-request

---

# Perenual Info

> **Category:** botany-nodes | **Type:** Action Node

Search plant species and retrieve **plant care guides** — watering needs, sunlight requirements, and other care details — from the **Perenual** API.

The **Perenual Info** node exposes two operations: searching the species list by name with optional edible/indoor filters, and fetching full care details for a specific species by ID.

### Supported Features

- Search species by name query
- Filter search results by `edible` and/or `indoor`
- Fetch full care detail for a species by ID
- Simple pass-through of the raw Perenual API JSON response

### Use Cases

- Look up a plant by common or scientific name before adding it to a garden-planning workflow
- Retrieve watering and sunlight guidance for a plant identified by another node (e.g. an image-identification node)
- Build a plant-care reminder or notification system
- Filter a species search to edible or indoor plants for a specific use case
- Combine with a plant identification node to go from photo → species → care guide

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ✅ Yes | — | Perenual API key. Must be at least 1 character. |
| `operation` | `enum` | ❌ No | `"search_species"` | Operation to perform: `search_species` or `get_details`. |
| `query` | `string` | ✅ Yes (for `search_species`) | — | Plant name to search for (e.g. `"Monstera"`). |
| `edible` | `boolean` | ❌ No (for `search_species`) | — | Filter results to edible plants only, when set. |
| `indoor` | `boolean` | ❌ No (for `search_species`) | — | Filter results to indoor plants only, when set. |
| `speciesId` | `string` | ✅ Yes (for `get_details`) | — | Species ID, typically taken from a prior `search_species` result. |

`query`, `edible`, and `indoor` are only shown for `search_species`; `speciesId` is only shown for `get_details`.

---

## Operations

| Operation | Endpoint | Description |
| --------- | -------- | ----------- |
| `search_species` | `GET /api/species-list` | Search the Perenual species list by name, with optional `edible`/`indoor` filters. |
| `get_details` | `GET /api/species/details/{speciesId}` | Get full care details for a specific species ID. |

### Search Filters

`edible` and `indoor` are sent to the API only **when explicitly set** (`!== undefined`), as `"1"` for `true` or `"0"` for `false`. If left unset, no filter is applied for that field.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

The node returns the **raw JSON response** from the Perenual API for the selected operation — there is no normalization or reshaping.

| Operation | Output |
| --------- | ------ |
| `search_species` | Perenual species-list response: a paginated list of matching species with basic info (`id`, `common_name`, `scientific_name`, thumbnail, etc.). |
| `get_details` | Perenual species-details response: full care guide fields (watering, sunlight, cycle, care level, toxicity, etc.) for the requested `speciesId`. |

---

## Output Example

### `search_species`

```json
{
  "data": [
    {
      "id": 3,
      "common_name": "Monstera",
      "scientific_name": ["Monstera deliciosa"],
      "other_name": ["Swiss Cheese Plant"],
      "default_image": {
        "thumbnail": "https://perenual.com/storage/species_image/3_thumb.jpg"
      }
    }
  ],
  "to": 30,
  "per_page": 30,
  "current_page": 1,
  "from": 1,
  "last_page": 5,
  "total": 142
}
```

### `get_details`

```json
{
  "id": 3,
  "common_name": "Monstera",
  "scientific_name": ["Monstera deliciosa"],
  "watering": "Average",
  "sunlight": ["part_shade"],
  "cycle": "Perennial",
  "care_level": "Medium",
  "poisonous_to_humans": 1,
  "poisonous_to_pets": 1,
  "indoor": true,
  "edible_fruit": false
}
```

The exact fields returned depend on the Perenual API's live response for the requested plant.

---

## Configuration Examples

### Basic Species Search

```json
{
  "apiKey": "your-perenual-api-key",
  "operation": "search_species",
  "query": "Monstera"
}
```

### Search Indoor Edible Plants

```json
{
  "apiKey": "your-perenual-api-key",
  "operation": "search_species",
  "query": "basil",
  "edible": true,
  "indoor": true
}
```

### Get Species Details

```json
{
  "apiKey": "your-perenual-api-key",
  "operation": "get_details",
  "speciesId": "3"
}
```

---

## Workflow Integration

### Sample Workflow: Search → Get Details

```json
{
  "nodes": [
    {
      "id": "perenual-search",
      "type": "perenual-info",
      "config": {
        "apiKey": "your-perenual-api-key",
        "operation": "search_species",
        "query": "Monstera"
      }
    },
    {
      "id": "perenual-details",
      "type": "perenual-info",
      "config": {
        "apiKey": "your-perenual-api-key",
        "operation": "get_details"
      }
    }
  ]
}
```

The second node needs `speciesId` supplied from the first node's result — typically wired through a `Function` node that extracts `data[0].id`.

### Sample Workflow: Identify → Perenual Care Guide

```json
{
  "nodes": [
    {
      "id": "plantnet-id",
      "type": "plantnet-id"
    },
    {
      "id": "extract-species-name",
      "type": "function"
    },
    {
      "id": "perenual-search",
      "type": "perenual-info",
      "config": {
        "apiKey": "your-perenual-api-key",
        "operation": "search_species"
      }
    }
  ]
}
```

### Sample Workflow: Perenual → Notification

```json
{
  "nodes": [
    {
      "id": "perenual-details",
      "type": "perenual-info",
      "config": {
        "apiKey": "your-perenual-api-key",
        "operation": "get_details",
        "speciesId": "3"
      }
    },
    {
      "id": "send-care-reminder",
      "type": "notification"
    }
  ]
}
```

### Common Patterns

- Pl@ntNet ID → Function (extract name) → Perenual (`search_species`) → Perenual (`get_details`) — photo-to-care-guide pipeline
- Perenual (`search_species`, `edible: true`) → Function → Database — build an edible-plant catalog
- Schedule → Perenual (`get_details`) → If (watering due) → Notification — care reminders

---

## Error Handling

### Missing Search Query

```text
Search query is required.
```

Raised for `search_species` when `query` is empty.

### Missing Species ID

```text
Species ID is required.
```

Raised for `get_details` when `speciesId` is empty.

### Perenual API Error

```text
Perenual Error: <statusText>
```

Raised when the Perenual API returns a non-OK HTTP status, for either operation.

---

## Troubleshooting

### "Search query is required."

**Cause**

`query` was left empty while `operation` is `search_species`.

**Solution**

Provide a plant name in `query`.

---

### "Species ID is required."

**Cause**

`speciesId` was left empty while `operation` is `get_details`.

**Solution**

Provide a valid species ID, typically obtained from a prior `search_species` call's `data[].id` field.

---

### "Perenual Error: Unauthorized" or similar

**Cause**

The `apiKey` is invalid, expired, or missing required access on the Perenual account.

**Solution**

Verify the API key in the Perenual developer dashboard.

---

### "Perenual Error: Too Many Requests"

**Cause**

The Perenual free tier enforces a daily request quota, which may have been exceeded.

**Solution**

Wait for the quota to reset, or upgrade the Perenual API plan.

---

### `get_details` Returns Unexpected or Empty Data

**Cause**

The `speciesId` does not correspond to a valid species in Perenual's database — for example, an ID copied incorrectly or from a different data source.

**Solution**

Re-run `search_species` to confirm the correct `id` for the intended plant before calling `get_details`.

---

## Security

The node performs outbound HTTP requests to the public Perenual API (`perenual.com/api`).

The `apiKey` is sent as a query parameter (`key=...`) on every request, as required by the Perenual API.

---

## Notes

The node returns the raw Perenual API response with no reshaping or field selection.

The node does not:

- Validate `speciesId` before calling `get_details`
- Support pagination parameters for `search_species` beyond what Perenual returns by default
- Cache search or detail results
- Combine search and details into a single call
- Handle Perenual's rate limits with retries

It is intended to provide direct search and detail lookup access to Perenual's plant care database for downstream gardening and plant-care workflows.

---

## Related

- [Function](./function.md) – Extract `speciesId` or format care-guide fields
- [If](./if.md) – Route workflows based on care requirements
- [HTTP Request](./http-request.md) – Make additional custom Perenual API calls

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-17 | Initial release |
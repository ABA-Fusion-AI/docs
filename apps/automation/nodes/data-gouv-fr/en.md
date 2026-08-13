---
node_id: "data-gouv-fr"
title: "Data.gouv.fr"
description: "Search and validate French addresses using the official French address API."
category: "Web Search & Information"
subcategory: "Maps & Geospatial"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - france
  - address
  - geocoding
  - postal-code
  - maps
  - location
  - api
related_nodes:
  - google-maps
  - open-street-map
  - ip-geolocation
---

<!-- SECTION: header -->
# Data.gouv.fr

> **Category:** Web Search & Information | **Type:** Action Node

Search and validate French addresses using the official French address API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Data.gouv.fr** node queries the official French address API to find, validate, and enrich French postal addresses. It is especially useful for workflows involving French customer records, form validation, logistics, government datasets, and geospatial enrichment.

### Key Features

- **Address Search:** Search for French addresses by query text
- **Postal Validation:** Validate addresses against the official French dataset
- **Geocoding Support:** Resolve coordinates and location details when available
- **Structured Output:** Return normalized address records and metadata
- **Form Enrichment:** Improve address quality before sending data to systems or databases
- **Workflow Integration:** Use results in routing, validation, or mapping processes

### Use Cases

- Validate shipping or billing addresses in France
- Clean up address data captured from user forms
- Enrich lead or customer records with normalized location information
- Build location-aware workflows for French service areas
- Support geospatial analysis using validated French addresses

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | ✅ Yes | — | Search text or address fragment to look up |
| `limit` | `number` | ❌ No | `10` | Maximum number of results to return |
| `autocomplete` | `boolean` | ❌ No | `true` | Whether to use autocomplete-style address search |

### Example

```text
query: "10 rue de la Paix Paris"
limit: 5
autocomplete: true
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | An address query or object containing search parameters |

### Outputs

| Output | Type | Description |
|--------|------|-------------| 
| `success` | `object` | Matching address records from the Data.gouv.fr API |
| `error` | `object` | Validation or API request error details |

### Success Output Example

```json
{
  "features": [
    {
      "properties": {
        "label": "10 Rue de la Paix, 75002 Paris",
        "postcode": "75002",
        "city": "Paris",
        "context": "75, Paris, France"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [2.3324, 48.8698]
      }
    }
  ],
  "count": 1
}
```

### Error Output Example

```json
{
  "success": false,
  "error": "The address lookup request was invalid or the API could not be reached.",
  "query": "10 rue de la Paix Paris"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Validate a French Address

```text
query: "10 rue de la Paix Paris"
limit: 5
```

**Result:**

```json
{
  "features": [
    {
      "properties": {
        "label": "10 Rue de la Paix, 75002 Paris",
        "postcode": "75002",
        "city": "Paris"
      }
    }
  ]
}
```

### Example: Use in a Form Validation Workflow

Use the node to validate address fields before storing customer data or generating shipping labels.

<!-- /SECTION: examples -->

---
  node_id: "mapbox"
  title: "Mapbox Geocoding"
  description: "Geocode addresses using the Mapbox Geocoding API."
  category: "Maps & Geospatial"
  subcategory: "geocoding"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-08-12"
  author: "Fusion Team"
  tags:
    - mapbox
    - geocoding
    - location
    - maps
    - address
    - coordinates
    - gis
  related_nodes:
    - address-validator
    - geo-names
    - graph-hopper-route
    - open-street-map
    - tom-tom
---

<!-- SECTION: overview -->
# Mapbox Geocoding

> **Category:** Maps & Geospatial&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Mapbox Geocoding** node converts human-readable street addresses or place names into geographic coordinates (latitude and longitude) using the Mapbox Geocoding API. It enables location-based workflows such as store locators, address normalization, delivery routing, and GIS data enrichment.

### Key Features

- **Forward Geocoding:** Search addresses, cities, landmarks, or POIs and retrieve geographic coordinates.
- **Dynamic Input Expression Support:** Pass static search strings or evaluate address parameters dynamically from incoming workflow items.
- **Rich Location Metadata:** Returns standardized place names, bounding boxes, region/country hierarchies, and relevance scores.
- **Flexible Token Configuration:** Accepts optional per-node Mapbox API Access Tokens for custom or enterprise tier access.

### Use Cases

- **E-commerce & Logistics:** Convert customer delivery addresses into precise geographical coordinates for route planning.
- **Store & Asset Locator:** Match user search queries to geographic locations to query nearby locations or facilities.
- **Data Cleanliness & Standardization:** Normalize free-form user address entries into canonical GeoJSON feature structures.
- **Real Estate & Travel:** Map properties, hotels, and attractions directly on custom maps.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `q` | `string` | ✅ Yes | `""` | The address, city, place name, or search query string to geocode (e.g. `"13 rue tantan hassan rabat"`). Supports expressions. |
| `accessToken` | `string` | ❌ No | `""` | Optional Mapbox API Access Token (starts with `pk.`). If provided, overrides default or environment Mapbox credentials. |

> [!NOTE]
> Mapbox API returns coordinates in `[longitude, latitude]` (GeoJSON standard) format.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Incoming workflow trigger or item payload feeding address parameter `q`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when Mapbox successfully processes the geocoding request and returns location results. |
| `error` | `Error` | Emitted when network request fails, token is invalid, or API returns an error response. |

### Output Schema (`success`)

The output payload contains the full GeoJSON `FeatureCollection` returned by the Mapbox Geocoding API.

```json
{
  "type": "FeatureCollection",
  "query": [
    "13",
    "rue",
    "tantan",
    "hassan",
    "rabat"
  ],
  "features": [
    {
      "id": "address.123456789",
      "type": "Feature",
      "place_type": [
        "address"
      ],
      "relevance": 0.95,
      "properties": {
        "accuracy": "point"
      },
      "text": "Rue Tantan",
      "place_name": "13 Rue Tantan, Hassan, Rabat 10000, Morocco",
      "center": [
        -6.8327,
        34.0209
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -6.8327,
          34.0209
        ]
      },
      "context": [
        {
          "id": "neighborhood.9876",
          "text": "Hassan"
        },
        {
          "id": "place.5432",
          "text": "Rabat"
        },
        {
          "id": "country.1111",
          "wikidata": "Q1028",
          "short_code": "ma",
          "text": "Morocco"
        }
      ]
    }
  ],
  "attribution": "NOTICE: © 2026 Mapbox and its suppliers. All rights reserved."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Geocode an Address with Mapbox
```

### Sample Workflow: Convert Webhook Address to Geolocation Log

Below is an overview of how the **Mapbox Geocoding** node functions in a standard workflow pipeline:

1. **Manual Trigger / Webhook Input:** Triggers execution with input data containing address fields.
2. **Mapbox Geocoding Node:** Evaluates parameter `q` (e.g. `"13 rue tantan hassan rabat"`), sends the request to Mapbox API, and receives GeoJSON coordinate results.
3. **Log Node:** Receives the `success` output containing `features` with `center` coordinates `[-6.8327, 34.0209]` and full location metadata.

### Common Patterns

- **Address to Coordinates Transformation:** Extract `features[0].geometry.coordinates` `[lng, lat]` for downstream spatial databases (PostGIS, MongoDB GeoJSON).
- **Fallback Handling:** Check `features.length > 0` before attempting to extract point centers to handle unresolvable query strings gracefully.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `HTTP 401 Unauthorized / Invalid Access Token`

**Cause:** The provided `accessToken` is invalid, expired, or revoked.

**Solution:** Check your Mapbox dashboard to generate a valid public access token (`pk.eyJ1...`) and update the parameter.

#### `Empty features array returned`

**Cause:** Mapbox could not find any matching geographic location for search string `q`.

**Solution:** Verify input address spelling, remove highly specific internal unit numbers, or format the query with city/country hints.

#### `Reversed Latitude and Longitude`

**Cause:** GeoJSON standard uses `[longitude, latitude]` order, whereas some mapping APIs expect `[latitude, longitude]`.

**Solution:** Note that Mapbox returns `coordinates: [lng, lat]`. Swap array elements `[1], [0]` if your downstream service requires `[lat, lng]`.

### Error Codes

| Error / Code | Cause | Solution |
|--------------|-------|----------|
| `401 Unauthorized` | Invalid or missing Mapbox Access Token | Provide a valid `accessToken` starting with `pk.` |
| `429 Too Many Requests` | Exceeded API rate limits | Implement rate limiting or upgrade Mapbox subscription tier |
| `422 Unprocessable Entity` | Malformed search query parameter | Ensure query string `q` is non-empty |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Address Validator](./address-validator.md) – Validate address structure before geocoding
- [OpenStreetMap](./open-street-map.md) – Alternative open-source mapping services
- [GraphHopper Route](./graph-hopper-route.md) – Calculate routing directions using geographic coordinates
- [GeoNames](./geo-names.md) – Query global postal code and geographical databases

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-12 | Initial documentation for Mapbox Geocoding node |

<!-- /SECTION: changelog -->

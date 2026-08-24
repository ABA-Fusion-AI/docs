---
node_id: "postcode-io"
title: "Postcode.io"
description: "Look up UK postcode information, coordinates, and administrative areas using the Postcode.io API."
category: "Web Search & Information"
subcategory: "Maps & Geospatial"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - postcode
  - uk
  - address
  - geocoding
  - location
  - geospatial
  - api
related_nodes:
  - geo-names
  - data-gouv-fr
  - google-maps
  - http-request
---

<!-- SECTION: header -->
# Postcode.io

> **Category:** Web Search & Information | **Subcategory:** Maps & Geospatial | **Type:** Action Node

Look up detailed UK postcode information using the free Postcode.io API. Results can include geographic coordinates, country, region, local authority, constituency, and other administrative data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Postcode.io** node sends a postcode lookup request to Postcode.io and returns the API response to the workflow. The API does not require authentication.

### Key Features

- **Postcode Lookup:** Resolve a full UK postcode to structured location data
- **Geographic Data:** Return latitude, longitude, eastings, and northings when available
- **Administrative Data:** Return country, region, local authority, ward, constituency, and related areas
- **Flexible Input:** Use a configured postcode or provide one dynamically through the `input` port
- **No Credentials Required:** Postcode.io is a public API

### Use Cases

- Validate UK postcodes before creating or updating records
- Enrich customer, supplier, or delivery data with location details
- Route orders or service requests by region or local authority
- Add coordinates to UK addresses for mapping workflows
- Build postcode-aware forms and automations

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `postcode` | `string` | Yes | — | UK postcode to look up, for example `SW1A 1AA`. Spaces and letter casing are normalized by the API. |

If `postcode` is not configured, the node can read the postcode from the incoming `input` value. A direct string is treated as the postcode; an object can provide a `postcode` property.

### API Endpoint

The node uses the Postcode.io postcode lookup endpoint:

```text
GET https://api.postcodes.io/postcodes/{postcode}
```

No API key is required.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | A postcode string or an object containing a `postcode` field. Used when the `postcode` parameter is empty. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Postcode.io response containing `status` and a `result` object with postcode and location data |

Example success response:

```json
{
  "status": 200,
  "result": {
    "postcode": "SW1A 1AA",
    "country": "England",
    "region": "London",
    "latitude": 51.501009,
    "longitude": -0.141588,
    "parliamentary_constituency": "Cities of London and Westminster"
  }
}
```

The exact fields in `result` depend on the postcode record and the data returned by Postcode.io.

### Error Output

Invalid, missing, or unavailable postcode lookups are routed to `error`.

```json
{
  "success": false,
  "error": "Postcode not found",
  "postcode": "INVALID"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Postcode Lookup

Configuration:

```json
{
  "postcode": "SW1A 1AA"
}
```

The node returns the matching postcode record and its geographic and administrative information.

### Dynamic Postcode from a Previous Node

Pass a postcode directly from another node:

```json
{
  "$expr": "output",
  "node": "Function",
  "outputId": "success"
}
```

Alternatively, pass an object such as:

```json
{
  "postcode": "M1 1AE"
}
```

### Extract Coordinates

Connect Postcode.io to a Function node to use the returned coordinates:

```js
return {
  latitude: input.result.latitude,
  longitude: input.result.longitude,
  postcode: input.result.postcode
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Look up a UK postcode and inspect the location data
```

### Common Patterns

- **Basic lookup:** Manual Trigger → Postcode.io → Log
- **Form validation:** Form/Webhook → Postcode.io → Conditional branch
- **Location enrichment:** Customer record → Postcode.io → Update record
- **Coordinate workflow:** Postcode.io → Function → Map or geospatial node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Postcode is required

**Cause:** No `postcode` parameter or usable incoming input was provided.

**Solution:** Set `postcode`, pass a string to `input`, or pass an object with a `postcode` field.

### Postcode not found

**Cause:** The postcode is invalid, incomplete, or not present in the Postcode.io dataset.

**Solution:** Check the spelling and format. Use a complete UK postcode such as `SW1A 1AA`.

### No coordinates in the result

**Cause:** Some records may not contain every geographic field.

**Solution:** Check for `latitude` and `longitude` before using them in downstream mapping or distance calculations.

### API request failed

**Cause:** The public API may be unavailable, rate-limited, or unreachable from the workflow runtime.

**Solution:** Retry the workflow, verify network access, and inspect the error output for the returned status and message.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [GeoNames](./geo-names.md) — Search for geographical places and locations
- [Data.gouv.fr](./data-gouv-fr.md) — Search and validate French addresses
- [Google Maps](./google-maps.md) — Work with Google Maps and location services
- [HTTP Request](./http-request.md) — Call a custom geocoding or postcode API

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->

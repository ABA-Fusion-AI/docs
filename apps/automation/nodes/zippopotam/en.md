---
node_id: "zippopotam"
title: "Zippopotam.us"
description: "Get location and geographic information (city, state, region, latitude, longitude) based on zip or postal codes across 60+ countries."
category: "Location & Mapping"
subcategory: "Geocoding & Postal Codes"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - zippopotam
  - zipcode
  - postal-code
  - geocoding
  - location
  - address
  - places
  - coordinates
  - latitude
  - longitude
  - maps
  - reverse-geocoding
  - address-enrichment
related_nodes:
  - http-request
  - ip-geolocation
  - address-validator
  - function
  - webhook
  - google-maps
  - mapbox
  - open-street-map
  - slack
---

<!-- SECTION: header -->
# Zippopotam.us

> **Category:** Location & Mapping | **Subcategory:** Geocoding & Postal Codes | **Type:** Action Node

Retrieve detailed geographic and location information (city, state, region, ISO country abbreviation, latitude, and longitude) from postal codes and country codes worldwide using the free [Zippopotam.us API](https://www.zippopotam.us/).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Zippopotam.us** action node converts international postal and ZIP codes into structured geographic metadata across more than 60 supported countries. With every successful query, the node returns the official city/place name, state or province, administrative division codes, and precise GPS coordinates (latitude and longitude).

Because the underlying Zippopotam.us API is completely free, open, and requires **no API key or registration**, this node can be instantly integrated into automated pipelines for address autofill, checkout validation, CRM data enrichment, shipping zone calculation, and localized weather dispatching.

```
                  ┌────────────────────────┐
                  │    Trigger / Webhook   │
                  │  (e.g., Form / Order)  │
                  └───────────┬────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │     Zippopotam.us      │
                  │ country: "FR", zip: "75001" │
                  └───────────┬────────────┘
                              │
            ┌─────────────────┴─────────────────┐
            │                                   │
      [success]                           [error]
            │                                   │
            ▼                                   ▼
┌───────────────────────────┐       ┌───────────────────────────┐
│     Downstream Nodes      │       │     Error Handling        │
│ • Autofill City / State   │       │ • Fallback Geocoder       │
│ • Distance / Freight Calc │       │ • Alert / Manual Review   │
│ • Weather & Route Mapping │       │                           │
└───────────────────────────┘       └───────────────────────────┘
```

### Key Capabilities

- **Global Coverage (60+ Countries):** Supports postal code lookup across North America, Europe, Asia, Latin America, and Oceania (including USA, Canada, UK, France, Germany, Spain, Italy, Australia, and more).
- **Zero Authentication / Keyless:** No API keys, credentials, or billing setups required. Ready to run out of the box.
- **Accurate GPS Geocoordinates:** Returns latitude and longitude in decimal degrees for mapping, distance calculations, and proximity routing.
- **Case-Insensitive Country Codes:** Automatically normalizes country inputs to lowercase (e.g. `US`, `us`, `Us` are all handled identically).
- **Multi-Place Resolution:** Seamlessly provides all locations and neighborhoods associated with a single postal code inside an array of `places`.
- **Dynamic Expression Support:** Effortlessly bind upstream form values, webhook payloads, or customer records using expressions (e.g., `outputs.Function.success.country`, `outputs.Function.success.zip`).

### Supported Countries Reference

Below is a representative sample of popular countries supported by Zippopotam.us:

| Country | Code | Postal Format Example | Example City / Region |
|---------|:----:|-----------------------|-----------------------|
| **United States** | `US` | `90210`, `10001` | Beverly Hills, CA / New York, NY |
| **France** | `FR` | `75001`, `69001` | Paris / Lyon |
| **Germany** | `DE` | `10115`, `80331` | Berlin / Munich |
| **Spain** | `ES` | `08001`, `28001` | Barcelona / Madrid |
| **Italy** | `IT` | `00184`, `20121` | Rome / Milan |
| **Canada** | `CA` | `M5V`, `H3Z` (First 3 chars / FSA) | Toronto, ON / Montreal, QC |
| **United Kingdom** | `GB` | `SW1A`, `EC1A` (Outward code) | London, Westminster |
| **Netherlands** | `NL` | `1012` (4 digits) | Amsterdam |
| **Belgium** | `BE` | `1000` | Brussels |
| **Switzerland** | `CH` | `8001` | Zurich |
| **Austria** | `AT` | `1010` | Vienna |
| **Australia** | `AU` | `2000` | Sydney, NSW |
| **Mexico** | `MX` | `01000` | Mexico City |
| **Brazil** | `BR` | `01000-000` or `01000` | São Paulo |
| **Japan** | `JP` | `100-0001` or `1000001` | Tokyo |

### Common Use Cases

- **E-Commerce Checkout & Form Autofill:** Automatically populate City and State as soon as the user finishes typing their postal code, reducing checkout friction and typos.
- **Logistics & Delivery Zone Routing:** Calculate direct geometric distance (via latitude and longitude) between a customer's postal code and the nearest fulfillment warehouse.
- **Localized Weather & Notifications:** Extract geocoordinates from a user's postal code to query localized forecast APIs and trigger push alerts.
- **CRM & Lead Data Cleansing:** Standardize inbound customer address records by filling in missing municipal or administrative region data.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure the Zippopotam.us node in the workflow canvas by setting the target country code and postal/ZIP code.

![Zippopotam Node Configuration](icon.svg)

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|:----:|:--------:|:-------:|-------------|
| `country` | `string` | ✅ Yes | — | Two-letter ISO 3166-1 alpha-2 country code (e.g. `US`, `FR`, `DE`, `ES`, `CA`, `GB`, `IT`). Case-insensitive. |
| `zip` | `string` | ✅ Yes | — | Postal or ZIP code to search for (e.g. `90210`, `75001`, `10115`, `08001`, `M5V`). |

---

### Detailed Parameter Descriptions

#### `country` (Required)
The standard two-letter ISO 3166-1 alpha-2 country code corresponding to the postal code.
- **Format:** 2 characters (e.g. `US`, `FR`, `DE`, `CA`, `GB`, `ES`, `IT`, `AU`).
- **Case Sensitivity:** Completely case-insensitive. The node automatically normalizes values to lowercase before dispatching the request (`US` and `us` behave identically).
- **Dynamic Expression:** Click the **Expression** button in the node modal to pass dynamic values from upstream nodes, such as `outputs.Function.success.country` or `outputs.Webhook.success.body.countryCode`.

#### `zip` (Required)
The postal or ZIP code to look up.
- **Format:** Standard postal code format for the specified country (e.g., `90210`, `75001`, `10115`, `08001`).
- **Country Specific Rules:**
  - **United States:** Standard 5-digit ZIP code (e.g. `90210`, `10001`, `02138`).
  - **Canada:** Use the 3-character Forward Sortation Area (FSA), e.g. `M5V`, `H3Z`, `V6B`.
  - **United Kingdom:** Use the outward code (first half of the postal code), e.g. `SW1A`, `EC1A`, `OX1`.
  - **Leading Zeros:** Ensure leading zeros are preserved as strings (e.g. `"00184"` for Rome, `"08001"` for Barcelona, `"01001"` for Agawam, MA).
- **Dynamic Expression:** Supports dynamic expressions such as `outputs.Function.success.zip` or `outputs.FormTrigger.success.postal_code`.

---

### Static vs Expression Mode

The node supports both static text entries and dynamic expressions:

```
┌─────────────────────────────────────────────────────────────┐
│ Parameters                                                  │
│                                                             │
│ Country                                         [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Function.success.country                        │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ Zip                                             [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Function.success.zip                            │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

- **Static Mode:** Type fixed strings directly (e.g., `FR` in Country and `75001` in Zip).
- **Expression Mode:** Click **Expression** to bind incoming workflow attributes from upstream nodes using `outputs.<NodeLabel>.<outputPort>.<field>`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|:----:|-------------|
| `input` | `any` | Incoming event payload or trigger data that activates the node. Upstream fields can be referenced via expressions in the node parameters. |

### Outputs

| Output | Type | Description |
|--------|:----:|-------------|
| `success` | `object` | Emitted when the postal code is successfully resolved. Contains the country metadata and an array of matching place records with coordinates. |
| `error` | `Error` | Emitted if the postal code is not found (404), if parameters are missing, or if an upstream network error occurs. |

---

### Output Data Structure Example

When querying `country: "FR"` and `zip: "75001"`:

```json
{
  "success": true,
  "country": "France",
  "country_abbreviation": "FR",
  "post_code": "75001",
  "places": [
    {
      "place_name": "Paris 01 Louvre",
      "longitude": "2.3417",
      "latitude": "48.8604",
      "state": "Île-de-France",
      "state_abbreviation": "11"
    }
  ],
  "total_places": 1
}
```

---

### Output Field Reference

| Field | Type | Description | Example |
|-------|:----:|-------------|---------|
| `success` | `boolean` | Indicates that the query was successfully executed. | `true` |
| `country` | `string` | Full name of the queried country. | `"France"`, `"United States"` |
| `country_abbreviation` | `string` | Official 2-letter uppercase ISO country code. | `"FR"`, `"US"`, `"DE"` |
| `post_code` | `string` | The postal/ZIP code that was queried. | `"75001"`, `"90210"` |
| `places` | `array` | List of places/neighborhoods associated with this postal code. | `[...]` |
| `places[i].place_name` | `string` | City, district, or municipality name. | `"Paris 01 Louvre"`, `"Beverly Hills"` |
| `places[i].state` | `string` | State, province, or primary administrative region. | `"Île-de-France"`, `"California"` |
| `places[i].state_abbreviation` | `string` | State, province, or departmental abbreviation. | `"11"`, `"CA"`, `"BY"` |
| `places[i].latitude` | `string` | Latitude coordinate in decimal degrees. | `"48.8604"`, `"34.0901"` |
| `places[i].longitude` | `string` | Longitude coordinate in decimal degrees. | `"2.3417"`, `"-118.4065"` |
| `total_places` | `number` | Count of records in the `places` array. | `1` |

---

### Consuming Output Data in Downstream Nodes

Downstream nodes can reference output fields from **Zippopotam.us** (assuming the node is labeled `Zippopotam` or `Zippopotam_us`) using standard expression syntax:

| Desired Data | Expression Syntax | Result Example |
|--------------|-------------------|----------------|
| **City / Place Name** | `outputs.Zippopotam.success.places[0].place_name` | `"Paris 01 Louvre"` |
| **State / Province** | `outputs.Zippopotam.success.places[0].state` | `"Île-de-France"` |
| **State Abbreviation** | `outputs.Zippopotam.success.places[0].state_abbreviation` | `"11"` |
| **Latitude** | `outputs.Zippopotam.success.places[0].latitude` | `"48.8604"` |
| **Longitude** | `outputs.Zippopotam.success.places[0].longitude` | `"2.3417"` |
| **Country Name** | `outputs.Zippopotam.success.country` | `"France"` |
| **Numeric Latitude** | `parseFloat(outputs.Zippopotam.success.places[0].latitude)` | `48.8604` |
| **Numeric Longitude** | `parseFloat(outputs.Zippopotam.success.places[0].longitude)` | `2.3417` |

> [!TIP]
> **Handling Multiple Places:** Some postal codes cover multiple towns or districts. To get the primary location, access index `0` via `places[0]`. To process all returned locations, connect the node to a **For Each** or **Function** node.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: United States ZIP Code (Beverly Hills)

Look up geographic data for Beverly Hills, California (`90210`):

**Configuration:**
- **Country:** `US`
- **Zip:** `90210`

**Output (`success`):**
```json
{
  "success": true,
  "country": "United States",
  "country_abbreviation": "US",
  "post_code": "90210",
  "places": [
    {
      "place_name": "Beverly Hills",
      "longitude": "-118.4065",
      "latitude": "34.0901",
      "state": "California",
      "state_abbreviation": "CA"
    }
  ],
  "total_places": 1
}
```

---

### Example 2: France Postal Code (Paris Louvre)

Retrieve municipal district and GPS coordinates for Paris, France:

**Configuration:**
- **Country:** `FR`
- **Zip:** `75001`

**Output (`success`):**
```json
{
  "success": true,
  "country": "France",
  "country_abbreviation": "FR",
  "post_code": "75001",
  "places": [
    {
      "place_name": "Paris 01 Louvre",
      "longitude": "2.3417",
      "latitude": "48.8604",
      "state": "Île-de-France",
      "state_abbreviation": "11"
    }
  ],
  "total_places": 1
}
```

---

### Example 3: Germany Postal Code (Berlin Mitte)

Look up city, state, and coordinates for central Berlin:

**Configuration:**
- **Country:** `de`
- **Zip:** `10115`

**Output (`success`):**
```json
{
  "success": true,
  "country": "Germany",
  "country_abbreviation": "DE",
  "post_code": "10115",
  "places": [
    {
      "place_name": "Berlin",
      "longitude": "13.3889",
      "latitude": "52.5323",
      "state": "Berlin",
      "state_abbreviation": "BE"
    }
  ],
  "total_places": 1
}
```

---

### Example 4: Canada Postal Code (Toronto Downtown)

Look up location information for Canadian Forward Sortation Area `M5V`:

**Configuration:**
- **Country:** `CA`
- **Zip:** `M5V`

**Output (`success`):**
```json
{
  "success": true,
  "country": "Canada",
  "country_abbreviation": "CA",
  "post_code": "M5V",
  "places": [
    {
      "place_name": "Downtown Toronto (CN Tower / King and Spadina)",
      "longitude": "-79.3957",
      "latitude": "43.6426",
      "state": "Ontario",
      "state_abbreviation": "ON"
    }
  ],
  "total_places": 1
}
```

---

### Example 5: Spain Postal Code (Barcelona)

Retrieve municipal district data for central Barcelona:

**Configuration:**
- **Country:** `ES`
- **Zip:** `08001`

**Output (`success`):**
```json
{
  "success": true,
  "country": "Spain",
  "country_abbreviation": "ES",
  "post_code": "08001",
  "places": [
    {
      "place_name": "Barcelona",
      "longitude": "2.1701",
      "latitude": "41.3818",
      "state": "Cataluna",
      "state_abbreviation": "CT"
    }
  ],
  "total_places": 1
}
```

---

### Example 6: Dynamic Lookup from Inbound Webhook

Receive dynamic address payload from an upstream webhook or checkout form:

**Upstream Webhook Payload (`Webhook`):**
```json
{
  "orderId": "ORD-55421",
  "customer": {
    "name": "Jane Doe",
    "countryCode": "US",
    "zipCode": "94103"
  }
}
```

**Node Configuration:**
- **Country:** `outputs.Webhook.success.customer.countryCode`
- **Zip:** `outputs.Webhook.success.customer.zipCode`

**Downstream Usage in Function Node (Distance Calculation):**
```javascript
// Calculate distance to fulfillment center in San Jose (37.3382, -121.8863)
const destLat = parseFloat(input.places[0].latitude);
const destLng = parseFloat(input.places[0].longitude);
const centerLat = 37.3382;
const centerLng = -121.8863;

const R = 6371; // Earth radius in km
const dLat = (destLat - centerLat) * Math.PI / 180;
const dLng = (destLng - centerLng) * Math.PI / 180;
const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
          Math.cos(centerLat * Math.PI / 180) * Math.cos(destLat * Math.PI / 180) *
          Math.sin(dLng/2) * Math.sin(dLng/2);
const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
const distanceKm = R * c;

return {
  orderId: input.orderId,
  city: input.places[0].place_name,
  state: input.places[0].state_abbreviation,
  deliveryDistanceKm: Math.round(distanceKm * 100) / 100,
  shippingFee: distanceKm > 50 ? 15.00 : 5.00
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Lookup geographic location and coordinates using Zippopotam.us
```

### Common Architecture Patterns

#### Pattern 1: E-Commerce Address Autofill & Validation
```text
[Form Submission Trigger]
          ↓
[Zippopotam.us] (country: outputs.FormTrigger.success.country, zip: outputs.FormTrigger.success.postalCode)
          ↓
[Function: Verify State / City Match]
    ├── (Match) ──> [Database / Order Created]
    └── (Mismatch) ──> [Prompt User to Confirm City]
```

#### Pattern 2: Proximity Routing & Freight Zone Assignment
```text
[Shopify Order Webhook]
          ↓
[Zippopotam.us] (Lookup Customer Postal Coordinates)
          ↓
[Function: Haversine Distance to Warehouses]
          ↓
[Branch by Zone]
    ├── Zone 1 (< 50 km)  ──> [Assign Local Same-Day Courier]
    └── Zone 2 (>= 50 km) ──> [Generate National Freight Label]
```

#### Pattern 3: Localized Weather Forecast Notification
```text
[User Chat Trigger / Telegram Bot] (User sends ZIP)
          ↓
[Zippopotam.us] (Get Latitude & Longitude)
          ↓
[HTTP Request / Open-Meteo API] (lat: outputs.Zippopotam.success.places[0].latitude, lon: outputs.Zippopotam.success.places[0].longitude)
          ↓
[Slack / Discord / SMS] (Send Daily Local Weather Summary)
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues & Solutions

#### 1. `Zippopotam.us request failed: Postal code not found`
- **Cause:** The requested postal code does not exist in the Zippopotam database for that country, or the country code is invalid/unsupported.
- **Solutions:**
  - Verify that the country code is an ISO 3166-1 alpha-2 code (e.g. `US`, `FR`, `DE`, `CA`, `GB`, `ES`, `IT`).
  - For **Canada**, verify you are passing the 3-character Forward Sortation Area (e.g. `M5V` instead of full `M5V 2H1`).
  - For the **United Kingdom**, verify you are passing the outward code (e.g. `SW1A` or `EC1A`).
  - Check for accidental leading or trailing whitespace in expression inputs (use `.trim()` if needed).

#### 2. `Country code and zip code are required`
- **Cause:** One or both parameters (`country`, `zip`) were evaluated as empty strings, `null`, or `undefined`.
- **Solutions:**
  - Check upstream node outputs to ensure the fields exist before triggering the Zippopotam node.
  - Set default fallback values in expressions: `outputs.Function.success.country || 'US'` and `outputs.Function.success.zip || '90210'`.

#### 3. Leading Zeros Stripped from Postal Codes
- **Cause:** Upstream JSON parsers or numerical conversions converted strings like `"00184"` or `"08001"` to numbers (`184` or `8001`).
- **Solutions:**
  - Keep postal codes formatted as strings throughout your workflow.
  - In a Function node, pad numbers if necessary: `String(input.zip).padStart(5, '0')`.

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Zippopotam.us request failed: Postal code not found` | HTTP 404 — Postal code does not exist in the country registry | Double-check country code and postal code format. Ensure Canadian/UK postal codes use outward format. |
| `Country code and zip code are required` | Required parameter is missing or empty | Provide both `country` and `zip` in node parameters or expressions. |
| `Zippopotam.us request failed: Zippopotam.us API error: 500` | Temporary upstream server issue at zippopotam.us | Configure workflow retry policy or add fallback geocoding node. |
| `Zippopotam.us request failed: TypeError: fetch failed` | Network connection timeout or DNS resolution failure | Verify network connectivity from the workflow runner. |

---

### Best Practices

- **Validate Input Existence:** When accepting user input from forms, verify that the country and postal code fields are populated before invoking the node.
- **Route Errors with the Error Handle:** Wire the red `error` output port to a fallback geocoding service (e.g., [HTTP Request](../http-request/en.md) or [IP Geolocation](../ip-geolocation/en.md)) to maintain high availability.
- **Parse Coordinates for Math:** Because latitude and longitude are returned as strings (e.g. `"48.8604"`), convert them with `parseFloat()` before doing mathematical distance comparisons.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [IP Geolocation](../ip-geolocation/en.md) — Identify location, city, and coordinates from IP addresses
- [Address Validator](../address-validator/en.md) — Validate and standardize street address records
- [HTTP Request](../http-request/en.md) — Query custom external geocoding and mapping APIs
- [Function](../function/en.md) — Transform coordinates, calculate distances, and structure address payloads
- [Webhook](../webhook/en.md) — Ingest inbound postal code requests from web forms and e-commerce platforms

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|:-------:|:----:|---------|
| `1.0.0` | 2026-09-01 | Comprehensive documentation update with parameter guides, supported countries reference, expression syntax (`outputs.<node>.<output>.<field>`), distance calculation patterns, and troubleshooting. |
| `1.0.0` | 2026-08-26 | Initial release of Zippopotam.us Geocoding Action Node. |

<!-- /SECTION: changelog -->

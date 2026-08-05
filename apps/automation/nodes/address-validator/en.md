---
node_id: "address-validator"
title: "Address Validator"
description: "Parse, validate, format, and verify postal addresses with optional Google Maps Geocoding API integration."
category: "peer-only"
subcategory: "business-utils-nodes"
version: "1.0.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - address
  - validation
  - parser
  - formatter
  - geocoding
  - google
  - maps
  - postal
  - Peer-only 
  - action
related_nodes:
  - function
  - webhook-trigger
  - http-request
---

<!-- SECTION: overview -->
# Address Validator

> **Category:** Peer-only  | **Type:** Action Node

Parse, validate, format, and optionally verify postal addresses using the Google Maps Geocoding API.

The Address Validator node helps normalize address data inside Fusion workflows. It supports simple local parsing and formatting, basic address validation, and full address verification using Google's Geocoding API.

### Use Cases

- Parse customer-entered addresses into structured data
- Verify that an address contains required fields
- Format addresses for printing or storage
- Validate shipping addresses before processing orders
- Standardize addresses using Google Maps
- Retrieve latitude and longitude from an address

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `parseAddress` | Operation to execute |
| `address` | `string` | ❌ No | `""` | Raw address string |
| `street` | `string` | ❌ No | `""` | Street address |
| `city` | `string` | ❌ No | `""` | City |
| `state` | `string` | ❌ No | `""` | State or province |
| `zipCode` | `string` | ❌ No | `""` | ZIP or postal code |
| `country` | `string` | ❌ No | `""` | Country |
| `apiKey` | `string` | ✅ Yes | `""` | Google Maps API key |

### Supported Operations

| Operation | Description |
|-----------|-------------|
| `parseAddress` | Parse a raw address into structured fields |
| `validateAddressFormat` | Validate that required address fields exist |
| `formatAddress` | Format address components into a printable address |
| `validateWithGoogle` | Validate and normalize an address using Google Maps |

---

### Operation: Parse Address

Parses a multi-line address into a structured object.

#### Parameters

| Field | Required | Description |
|-------|----------|-------------|
| `address` | ✅ Yes | Address to parse |

#### Output

```json
{
  "success": true,
  "original": "1600 Amphitheatre Parkway\nMountain View, CA 94043",
  "street": "1600 Amphitheatre Parkway",
  "city": null,
  "state": null,
  "zipCode": "94043",
  "country": null
}
```

> **Note**
>
> The parser extracts:
>
> - Street (first line)
> - ZIP code (US ZIP format)
>
> City, state, and country are not automatically detected.

---

### Operation: Validate Address Format

Checks whether the address contains the required fields.

Required fields:

- Street
- City
- ZIP Code

#### Parameters

| Field | Required |
|-------|----------|
| `address` | ✅ Yes |

#### Output

```json
{
  "success": true,
  "valid": false,
  "missing": [
    "city"
  ],
  "parsed": {}
}
```

---

### Operation: Format Address

Formats address components into a multi-line postal address.

#### Parameters

| Field | Required |
|-------|----------|
| `street` | No |
| `city` | No |
| `state` | No |
| `zipCode` | No |
| `country` | No |

#### Output

```text
1600 Amphitheatre Parkway
Mountain View
CA 94043
USA
```

---

### Operation: Validate With Google

Validates and standardizes an address using the Google Maps Geocoding API.

#### Parameters

| Field | Required |
|-------|----------|
| `address` | ✅ Yes |
| `apiKey` | ✅ Yes |

#### Output

```json
{
  "success": true,
  "formatted": "1600 Amphitheatre Pkwy, Mountain View, CA 94043, USA",
  "components": [],
  "location": {
    "lat": 37.422,
    "lng": -122.084
  },
  "placeId": "ChIJ2eUgeAK6j4ARbn5u_wAGqWA"
}
```

If validation fails:

```json
{
  "success": false,
  "error": "Address not found"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Workflow data used to populate address fields via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Operation result |
| `error` | `Error` | Emitted if the operation fails |

### Output Summary

| Operation | Output |
|-----------|--------|
| `parseAddress` | Parsed address object |
| `validateAddressFormat` | Validation result |
| `formatAddress` | Formatted address string |
| `validateWithGoogle` | Google validation result |

### Example: Parse Address

**Configuration**

```json
{
  "operation": "parseAddress",
  "address": "1600 Amphitheatre Parkway\nMountain View, CA 94043\nUSA"
}
```

**Output**

```json
{
  "success": true,
  "street": "1600 Amphitheatre Parkway",
  "zipCode": "94043"
}
```

---

### Example: Format Address

**Configuration**

```json
{
  "operation": "formatAddress",
  "street": "221B Baker Street",
  "city": "London",
  "zipCode": "NW1",
  "country": "United Kingdom"
}
```

**Output**

```text
221B Baker Street
London
NW1
United Kingdom
```

---

### Example: Validate With Google

**Configuration**

```json
{
  "operation": "validateWithGoogle",
  "address": "1600 Amphitheatre Parkway, Mountain View, CA",
  "apiKey": "{{secrets.googleApiKey}}"
}
```

**Output**

```json
{
  "success": true,
  "formatted": "1600 Amphitheatre Pkwy, Mountain View, CA 94043, USA",
  "location": {
    "lat": 37.422,
    "lng": -122.084
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Validate Shipping Address

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "validate-address",
      "type": "address-validator",
      "config": {
        "operation": "validateWithGoogle",
        "address": "{{input.shippingAddress}}",
        "apiKey": "{{secrets.googleApiKey}}"
      }
    },
    {
      "id": "create-order",
      "type": "order-action"
    }
  ]
}
```

### Common Patterns

- Customer Registration → Validate Address → Save Customer
- Checkout → Validate Shipping Address → Create Shipment
- CSV Import → Parse Address → Store
- Invoice Generation → Format Address → PDF Generator
- Webhook → Validate With Google → Continue Workflow

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Address is required

**Cause**

The selected operation requires an address, but none was supplied.

**Solution**

Provide a non-empty `address` value.

---

### Google Maps API key required

**Cause**

The Google validation operation was selected without providing an API key.

**Solution**

Provide a valid Google Maps Geocoding API key.

---

### Address not found

**Cause**

Google could not locate the supplied address.

**Solution**

- Verify the spelling.
- Include city and country.
- Use a complete postal address.

---

### Invalid API Key

**Cause**

The provided Google API key is invalid or the Geocoding API is not enabled.

**Solution**

- Verify the API key.
- Enable the Google Geocoding API.
- Ensure billing is enabled in Google Cloud.

---

### Parser Limitations

The built-in parser is intentionally lightweight.

It currently extracts:

- Street (first line)
- ZIP code

It does not automatically detect:

- City
- State
- Country

Use **validateWithGoogle** for complete address normalization.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Execute custom JavaScript
- [Webhook Trigger](./webhook-trigger.md) – Receive incoming requests
- [HTTP Request](./http-request.md) – Call external APIs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-05 | Initial release |

<!-- /SECTION: changelog -->
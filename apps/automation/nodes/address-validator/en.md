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
  - utility
  - action
related_nodes:
  - function
  - webhook-trigger
  - http-request
---

<!-- SECTION: header -->
# Address Validator Action

> **Category:** Peer-only | **Type:** Action Node

Parse, validate, format, and optionally verify postal addresses using the Google Maps Geocoding API. This node helps normalize address data, verify required fields, and retrieve standardized address information.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Address Validator Action** provides a simple toolkit for working with postal addresses inside Fusion workflows.

It supports four operations:

- Parse a raw address into structured fields
- Validate that an address contains required information
- Format address components into a printable address
- Validate and normalize an address using the Google Maps Geocoding API

The first three operations run locally and require no external services. Google validation requires a valid Google Maps API key.

### Key Features

- **4 Operations:** Parse, Validate, Format, and Google Validation
- **Local Processing:** No API required for parsing, formatting, or basic validation
- **Google Maps Integration:** Standardize and validate addresses
- **Geolocation Support:** Returns latitude and longitude
- **Workflow Friendly:** Simple JSON outputs
- **Supports Multi-line Addresses**

### Use Cases

- Validate customer shipping addresses
- Format addresses before generating invoices
- Normalize addresses before saving them to a database
- Retrieve coordinates from an address
- Check incoming webhook address data
- Verify user-entered addresses before processing

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to execute |
| `address` | `string` | Conditional | Raw address string |
| `street` | `string` | Conditional | Street address |
| `city` | `string` | Conditional | City |
| `state` | `string` | Conditional | State or province |
| `zipCode` | `string` | Conditional | ZIP or postal code |
| `country` | `string` | Conditional | Country |
| `apiKey` | `string` | Conditional | Google Maps API key |

Each conditional field is only required for the selected operation.

---

### Operation: Parse Address

Parses a raw address into a simple structured object.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `address` | `string` | ✅ Yes | Address to parse |

**Output**

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
> The parser currently extracts:
>
> - Street (first line)
> - ZIP code using a regular expression

---

### Operation: Validate Address Format

Checks whether an address contains the required fields.

Required fields:

- Street
- City
- ZIP Code

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `address` | `string` | ✅ Yes | Address to validate |

**Output**

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

Formats address components into a printable multi-line address.

| Field | Type | Required |
|-------|------|----------|
| `street` | `string` | No |
| `city` | `string` | No |
| `state` | `string` | No |
| `zipCode` | `string` | No |
| `country` | `string` | No |

**Output**

```text
1600 Amphitheatre Parkway
Mountain View
CA 94043
USA
```

---

### Operation: Validate With Google

Validates and normalizes an address using the Google Maps Geocoding API.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `address` | `string` | ✅ Yes | Address to validate |
| `apiKey` | `string` | ✅ Yes | Google Maps API key |

**Output**

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

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow data used to populate address fields |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Operation result |
| `error` | `Error` | Returned if the operation fails |

### Output Summary

| Operation | Output |
|-----------|--------|
| Parse Address | Parsed address object |
| Validate Address Format | Validation result |
| Format Address | Formatted address |
| Validate With Google | Google validation result |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Parse Address

```json
{
  "operation": "parseAddress",
  "address": "1600 Amphitheatre Parkway\nMountain View, CA 94043\nUSA"
}
```

**Returns**

```json
{
  "success": true,
  "street": "1600 Amphitheatre Parkway",
  "zipCode": "94043"
}
```

---

### Example: Validate Address

```json
{
  "operation": "validateAddressFormat",
  "address": "1600 Amphitheatre Parkway\nMountain View, CA 94043"
}
```

**Returns**

```json
{
  "success": true,
  "valid": false,
  "missing": [
    "city"
  ]
}
```

---

### Example: Format Address

```json
{
  "operation": "formatAddress",
  "street": "221B Baker Street",
  "city": "London",
  "zipCode": "NW1",
  "country": "United Kingdom"
}
```

**Returns**

```text
221B Baker Street
London
NW1
United Kingdom
```

---

### Example: Validate With Google

```json
{
  "operation": "validateWithGoogle",
  "address": "1600 Amphitheatre Parkway, Mountain View, CA",
  "apiKey": "{{secrets.googleApiKey}}"
}
```

**Returns**

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

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow

Validate a shipping address before creating an order.

```text
Webhook Trigger
       │
       ▼
Address Validator
(validateWithGoogle)
       │
       ▼
Function
(Check success)
       │
       ▼
Database Insert
```

### Common Patterns

- Customer Registration → Validate Address → Save Customer
- Order Checkout → Validate Address → Create Shipment
- CSV Import → Parse Address → Normalize → Store
- Invoice Generator → Format Address → Generate PDF
- Webhook → Google Validation → Continue Workflow

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Address is required

**Cause**

The selected operation requires an address but none was supplied.

**Solution**

Provide a non-empty `address` value.

---

### Google Maps API key required

**Cause**

`validateWithGoogle` was selected without an API key.

**Solution**

Provide a valid Google Maps Geocoding API key.

---

### Address not found

**Cause**

Google could not locate the supplied address.

**Solution**

- Verify spelling.
- Include city and country.
- Provide a complete postal address.

---

### Invalid API Key

**Cause**

The supplied Google API key is invalid or the Geocoding API is not enabled.

**Solution**

- Verify the API key.
- Enable the Geocoding API.
- Ensure billing is enabled for your Google Cloud project.

---

### Network Error

**Cause**

The request to Google Maps failed.

**Solution**

Retry the request or verify internet connectivity.

---

### Parser Limitations

The built-in parser currently extracts:

- Street (first line)
- ZIP code

City, state, and country are **not** automatically detected.

For full normalization, use **Validate With Google**.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Execute custom JavaScript
- [Webhook Trigger](./webhook-trigger.md) – Receive incoming address data
- [HTTP Request](./http-request.md) – Call external APIs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-05 | Initial release |

<!-- /SECTION: changelog -->
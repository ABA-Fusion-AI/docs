---
node_id: "openfda-get-drug-by-generic-name"
title: "OpenFDA Get Drug by Generic Name"
description: "Get drug information by generic (active ingredient) name"
category: "healthcare-life-sciences"
subcategory: "openfda"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - "openfda"
  - "drug"
  - "generic-name"
  - "healthcare"
  - "pharmaceutical"
related_nodes:
  - "openfda-get-drug-by-name"
  - "openfda-get-drugs-by-manufacturer"
  - "openfda-get-safety-info"
---

<!-- SECTION: header -->

# OpenFDA Get Drug by Generic Name

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Get drug information by generic (active ingredient) name.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **OpenFDA Get Drug by Generic Name** node retrieves drug label information from the OpenFDA API using a generic or active ingredient name.

### Key Features

- Search drugs by generic name
- Retrieve multiple matching drug records
- Control the maximum number of returned results
- Return normalized drug information
- Handle searches with no matching results
- Use the OpenFDA drug label dataset

### Processing Flow

1. Provide a generic drug name.
2. Set the maximum number of results.
3. Build an OpenFDA search query using the generic name.
4. Send the request to the OpenFDA drug label endpoint.
5. Normalize and return the matching drug records.

### Use Cases

- Search medications by active ingredient
- Find brand names associated with a generic drug
- Retrieve drug manufacturer information
- Identify available administration routes
- Build pharmaceutical lookup workflows
- Enrich healthcare or drug-related datasets

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `genericName` | String | Yes | Generic drug or active ingredient name to search for. |
| `limit` | Number | No | Maximum number of drug records to return. Default: `5`. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The generic drug name can be configured directly in the node.

If `genericName` is not configured and the incoming workflow data is a string, the node uses that string as the generic name.

Example:

```text
ibuprofen
```

### Outputs

For a successful search, the node returns:

```json
{
  "total": 1184,
  "count": 5,
  "drugs": [
    {
      "brand_name": "Example Brand",
      "generic_name": "IBUPROFEN",
      "manufacturer_name": "Example Manufacturer",
      "product_type": "HUMAN OTC DRUG",
      "route": [
        "ORAL"
      ]
    }
  ]
}
```

Each drug item can include:

- `brand_name`
- `generic_name`
- `manufacturer_name`
- `product_type`
- `route`

If no matching drug is found, the node returns an error object with an empty `drugs` array.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Search for Ibuprofen

Configuration:

```json
{
  "genericName": "ibuprofen",
  "limit": 5
}
```

This retrieves up to five OpenFDA drug label records matching the generic name `ibuprofen`.

### Limit Results

```json
{
  "genericName": "ibuprofen",
  "limit": 2
}
```

This limits the result to two matching drug records.

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: OpenFDA Get Drug by Generic Name Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Generic Name Is Required

**Cause:** No generic drug name was configured and no string value was received from the previous node.

**Solution:** Provide a valid value for `genericName`.

### No Drugs Found

**Cause:** OpenFDA returned no matching drug labels for the supplied generic name.

**Solution:** Verify the spelling of the generic drug name and try another active ingredient name.

### OpenFDA Request Error

**Cause:** The OpenFDA service returned an HTTP or network error.

**Solution:** Verify network connectivity and retry the workflow. The node automatically retries temporary server, rate-limit, and network errors.

### Rate Limiting

**Cause:** OpenFDA returned HTTP `429` because too many requests were made.

**Solution:** Wait before retrying. The node automatically performs retry attempts with exponential backoff.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **OpenFDA Get Drug by Name** — Retrieve drug information using a brand name.
- **OpenFDA Get Drugs by Manufacturer** — Retrieve drugs produced by a specific manufacturer.
- **OpenFDA Get Safety Info** — Retrieve warnings, contraindications, interactions, and other safety information.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-04` | Initial documentation. |

<!-- /SECTION: changelog -->
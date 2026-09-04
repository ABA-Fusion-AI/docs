---
node_id: "openfda-get-drug-by-name"
title: "OpenFDA Get Drug by Name"
description: "Get drug information by brand name from OpenFDA"
category: "healthcare-life-sciences"
subcategory: "openfda"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - "openfda"
  - "drug"
  - "brand-name"
  - "healthcare"
  - "pharmaceutical"
related_nodes:
  - "openfda-get-drug-by-generic-name"
  - "openfda-get-drugs-by-manufacturer"
  - "openfda-get-safety-info"
---

<!-- SECTION: header -->

# OpenFDA Get Drug by Name

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Get drug information by brand name from OpenFDA.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **OpenFDA Get Drug by Name** node retrieves drug label information from the OpenFDA API using a brand name.

### Key Features

- Search drugs by brand name
- Retrieve normalized OpenFDA drug information
- Access product and manufacturer details
- Retrieve indications, warnings, and safety-related label data
- Handle cases where no matching drug is found
- Use the OpenFDA drug label dataset

### Processing Flow

1. Provide a drug brand name.
2. Build an OpenFDA search query using the brand name.
3. Send the request to the OpenFDA drug label endpoint.
4. Retrieve the matching drug label record.
5. Normalize and return the available drug information.

### Use Cases

- Search medication information by brand name
- Retrieve generic names associated with a brand
- Identify manufacturers and product types
- Access indications and usage information
- Retrieve warnings and usage precautions
- Build healthcare and pharmaceutical lookup workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `drugName` | String | Yes | Brand name of the drug to search for. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The drug brand name can be configured directly in the node.

If `drugName` is not configured and the incoming workflow data is a string, the node can use that string as the drug name.

Example:

```text
Advil
```

### Outputs

For a successful search, the node can return:

```json
{
  "brand_name": [
    "Advil Dual Action with Acetaminophen"
  ],
  "generic_name": [
    "IBUPROFEN, ACETAMINOPHEN"
  ],
  "manufacturer_name": [
    "L'il Drug Store Products, Inc."
  ],
  "product_ndc": [
    "66715-6401",
    "66715-9601"
  ],
  "product_type": [
    "HUMAN OTC DRUG"
  ],
  "route": [
    "ORAL"
  ]
}
```

Depending on the OpenFDA record, the output can also include:

- `substance_name`
- `indications_and_usage`
- `warnings`
- `do_not_use`
- `ask_doctor`
- `stop_use`
- `pregnancy_or_breast_feeding`

If no matching drug is found, the node returns an error object with suggestions.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Search for Advil

Configuration:

```json
{
  "drugName": "Advil"
}
```

This searches OpenFDA for a drug label matching the brand name `Advil`.

### No Matching Drug

Configuration:

```json
{
  "drugName": "zzznonexistentdrug12345"
}
```

If no matching drug is found, the node returns an error message and suggestions instead of failing the workflow.

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: OpenFDA Get Drug by Name Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Drug Name Is Required

**Cause:** No drug brand name was configured and no usable string input was received.

**Solution:** Provide a valid value for `drugName`.

### No Drug Information Found

**Cause:** OpenFDA returned no matching label for the supplied brand name.

**Solution:** Verify the exact brand name spelling, try the generic name instead, or confirm that the drug is FDA-listed.

### OpenFDA Request Error

**Cause:** The OpenFDA service returned an HTTP or network error.

**Solution:** Verify network connectivity and retry the workflow. Temporary server, rate-limit, and network errors are retried automatically.

### Rate Limiting

**Cause:** OpenFDA returned HTTP `429` because too many requests were made.

**Solution:** Wait before retrying. The node automatically performs retry attempts with exponential backoff.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **OpenFDA Get Drug by Generic Name** — Retrieve drug information using a generic or active ingredient name.
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
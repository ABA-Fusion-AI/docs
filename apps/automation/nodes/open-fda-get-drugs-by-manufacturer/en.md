---
node_id: "openfda-get-drugs-by-manufacturer"
title: "OpenFDA Get Drugs by Manufacturer"
description: "Get all drugs manufactured by a specific company"
category: "healthcare-life-sciences"
subcategory: "openfda"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - "openfda"
  - "drug"
  - "manufacturer"
  - "healthcare"
  - "pharmaceutical"
related_nodes:
  - "openfda-get-drug-by-name"
  - "openfda-get-drug-by-generic-name"
  - "openfda-get-safety-info"
---

<!-- SECTION: header -->

# OpenFDA Get Drugs by Manufacturer

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Get all drugs manufactured by a specific company.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **OpenFDA Get Drugs by Manufacturer** node retrieves drug information from the OpenFDA API using a manufacturer name.

### Key Features

- Search drugs by manufacturer name
- Control the number of returned results
- Retrieve brand and generic drug names
- Access product type, administration route, and NDC information
- Return the total number of matching OpenFDA records
- Handle manufacturers with no matching drugs

### Processing Flow

1. Provide a manufacturer name.
2. Configure the maximum number of results to return.
3. Build an OpenFDA query using the manufacturer name.
4. Search the OpenFDA drug label dataset.
5. Normalize matching drug records.
6. Return the manufacturer, result count, total matches, and drug information.

### Use Cases

- Find drugs associated with a pharmaceutical manufacturer
- Explore a manufacturer's drug portfolio
- Retrieve brand and generic drug names
- Retrieve NDC information
- Build pharmaceutical research workflows
- Integrate manufacturer-based drug searches into healthcare applications

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `manufacturerName` | String | Yes | — | Name of the manufacturer to search for. |
| `limit` | Number | No | `20` | Maximum number of matching drugs to return. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

Configure the manufacturer and optional result limit directly in the node.

Example:

```json
{
  "manufacturerName": "Pfizer",
  "limit": 5
}
```

### Outputs

A successful search returns the total number of matches, the number of returned records, the manufacturer name, and a list of drugs.

Example:

```json
{
  "total": 96,
  "count": 5,
  "manufacturer": "Pfizer",
  "drugs": [
    {
      "brand_name": "Doxorubicin Hydrochloride",
      "generic_name": "DOXORUBICIN HYDROCHLORIDE",
      "product_type": "HUMAN PRESCRIPTION DRUG",
      "route": [
        "INTRAVENOUS"
      ],
      "ndc": "0069-0255"
    }
  ]
}
```

Each drug can include:

- `brand_name`
- `generic_name`
- `product_type`
- `route`
- `ndc`

If no drugs are found, the node returns an error message with `total: 0` and an empty `drugs` array.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Search for Pfizer Drugs

Configuration:

```json
{
  "manufacturerName": "Pfizer",
  "limit": 5
}
```

The node searches OpenFDA for drugs associated with `Pfizer` and returns up to five matching records.

### Limit Results

Configuration:

```json
{
  "manufacturerName": "Pfizer",
  "limit": 2
}
```

The node returns two drug records while preserving the total number of matches reported by OpenFDA.

### No Matching Manufacturer

Configuration:

```json
{
  "manufacturerName": "zzznonexistentmanufacturer12345",
  "limit": 5
}
```

When no matching drugs are found, the node returns a structured no-result response.

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: OpenFDA Get Drugs by Manufacturer Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Manufacturer Name Is Required

**Cause:** No manufacturer name was provided.

**Solution:** Configure `manufacturerName` with a valid pharmaceutical manufacturer name.

### No Drugs Found

**Cause:** OpenFDA contains no matching records for the supplied manufacturer name.

**Solution:** Verify the manufacturer name and try a different or more exact company name.

### OpenFDA Request Error

**Cause:** The OpenFDA API returned an HTTP or network error.

**Solution:** Verify network connectivity and retry the workflow. Temporary server, rate-limit, and network errors are retried automatically.

### Rate Limiting

**Cause:** OpenFDA returned HTTP `429` because too many requests were made.

**Solution:** Wait before retrying. The node automatically performs retry attempts with exponential backoff.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **OpenFDA Get Drug by Name** — Retrieve drug information using a brand name.
- **OpenFDA Get Drug by Generic Name** — Retrieve drugs using a generic or active ingredient name.
- **OpenFDA Get Safety Info** — Retrieve warnings, contraindications, interactions, and other safety information.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-04` | Initial documentation. |

<!-- /SECTION: changelog -->
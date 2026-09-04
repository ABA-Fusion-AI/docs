---
node_id: "openfda-get-safety-info"
title: "OpenFDA Get Safety Info"
description: "Get comprehensive safety information including warnings and contraindications"
category: "healthcare-life-sciences"
subcategory: "openfda"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - "openfda"
  - "drug"
  - "safety"
  - "healthcare"
  - "warnings"
related_nodes:
  - "openfda-get-drug-by-name"
  - "openfda-get-drug-by-generic-name"
  - "openfda-get-drugs-by-manufacturer"
---

<!-- SECTION: header -->

# OpenFDA Get Safety Info

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Get comprehensive safety information including warnings and contraindications.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **OpenFDA Get Safety Info** node retrieves comprehensive drug safety information from the OpenFDA drug label dataset using a drug brand name.

### Key Features

- Search safety information by drug name
- Retrieve warnings and contraindications
- Access drug interaction information
- Retrieve precautions and adverse reactions
- Access overdose and usage safety information
- Handle drugs with no matching safety information

### Processing Flow

1. Provide a drug name.
2. Build an OpenFDA query using the drug brand name.
3. Search the OpenFDA drug label dataset.
4. Retrieve the matching drug label.
5. Extract available safety-related information.
6. Return the drug name, generic name, and safety fields.

### Use Cases

- Retrieve drug warnings
- Review contraindications
- Access drug interaction information
- Build medication safety workflows
- Retrieve precautions and adverse reactions
- Integrate OpenFDA safety data into healthcare applications

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `drugName` | String | Yes | — | Brand name of the drug whose safety information should be retrieved. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

Configure the drug name directly in the node.

Example:

```json
{
  "drugName": "Advil"
}
```

### Outputs

A successful search returns the drug identity and the available safety information from OpenFDA.

The response can include:

- `drug_name`
- `generic_name`
- `warnings`
- `contraindications`
- `drug_interactions`
- `precautions`
- `adverse_reactions`
- `overdosage`
- `do_not_use`
- `ask_doctor`
- `stop_use`
- `pregnancy_or_breast_feeding`

Example structure:

```json
{
  "drug_name": "Advil Dual Action with Acetaminophen, Travel BASIX",
  "generic_name": "IBUPROFEN, ACETAMINOPHEN TABLET, FILM COATED",
  "warnings": [
    "Safety warning information..."
  ],
  "contraindications": [],
  "drug_interactions": [],
  "precautions": []
}
```

The exact safety fields depend on the information available in the OpenFDA drug label.

If no safety information is found, the node returns a structured error response.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Retrieve Advil Safety Information

Configuration:

```json
{
  "drugName": "Advil"
}
```

The node searches OpenFDA for an Advil drug label and returns available warnings, contraindications, interactions, precautions, and other safety information.

### No Matching Drug

Configuration:

```json
{
  "drugName": "zzznonexistentdrug12345"
}
```

The node returns:

```json
{
  "error": "No safety information found for \"zzznonexistentdrug12345\""
}
```

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: OpenFDA Get Safety Info Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Drug Name Is Required

**Cause:** No drug name was provided.

**Solution:** Configure `drugName` with a valid drug brand name.

### No Safety Information Found

**Cause:** OpenFDA contains no matching drug label for the supplied name.

**Solution:** Verify the drug brand name and try another FDA-listed name.

### Some Safety Fields Are Empty

**Cause:** Not every OpenFDA drug label contains every safety field.

**Solution:** Check the other returned safety fields. Empty arrays indicate that the corresponding information was not available in the matched label.

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
- **OpenFDA Get Drug by Generic Name** — Retrieve drug information using a generic name.
- **OpenFDA Get Drugs by Manufacturer** — Retrieve drugs associated with a manufacturer.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-04` | Initial documentation. |

<!-- /SECTION: changelog -->
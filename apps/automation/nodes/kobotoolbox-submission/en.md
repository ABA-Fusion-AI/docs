---
node_id: "kobotoolbox-submission"
title: "KoboToolbox Submission"
description: "Automate submission retrieval, deletion, validation status updates, and advanced filtering."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-12"
author: "Fusion Team"
tags: [kobotoolbox, submissions, data, validation, geojson, peer-only]
related_nodes: ["kobotoolbox-form", "kobotoolbox-file", "kobotoolbox-hook", "kobotoolbox"]
---

<!-- SECTION: overview -->
# KoboToolbox Submission

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **KoboToolbox Submission** node enables workflows to automate responses data management, data quality assurance, validation workflows, and data reformatting for KoboToolbox survey submissions.

### Key Capabilities
- **Submissions Querying (`getMany`)**:
  - Filter using standard MongoDB JSON query syntax (e.g. `{"_submission_time": {"$gt": "2026-01-01"}}`).
  - Field selection projection to reduce payload size.
  - Sorting criteria via MongoDB JSON syntax (e.g. `{"_id": -1}`).
  - Built-in pagination handling fetching up to 30,000 records.
- **Data Reformatting Engine**:
  - Convert flat slash-separated keys (`group/subgroup/field`) into nested hierarchical JSON objects.
  - Convert geopoints, geotraces, and geoshapes to standardized GeoJSON geometries (`Point`, `LineString`, `Polygon`).
  - Split multi-select space-separated strings into native arrays via wildcard mask patterns (e.g., `Crops_*`).
  - Convert numeric string fields into numbers via wildcard mask patterns (e.g., `*_sqm`).
- **Single Submission Operations**: Retrieve (`get`) or delete (`delete`) individual submissions by ID or UUID.
- **Validation Status**: Inspect (`getValidationStatus`) and update (`updateValidationStatus`) submission validation status (`approved`, `not_approved`, `on_hold`).
<!-- /SECTION: overview -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Configuration Parameters
- **Operation (`operation`)**: `getMany`, `get`, `delete`, `getValidationStatus`, or `updateValidationStatus`.
- **Form ID (`formId`)**: The asset UID of the deployed form.
- **Submission ID (`submissionId`)**: Submission primary key or rootUuid (for single operations).
- **Validation Status (`validationStatus`)**: Status UID (`validation_status_approved`, `validation_status_not_approved`, `validation_status_on_hold`).
- **Query Filters (`query`)**: MongoDB JSON filter string.
- **Fields (`fields`)**: Comma-separated list or JSON array of fields to return.
- **Sort (`sort`)**: MongoDB JSON sort object string.
- **Reformat (`reformat`)**: Boolean to enable unflattening into hierarchical group objects.
- **Multiselect Mask (`multiselectMask`)**: Wildcard pattern (e.g., `Crops_*`) for splitting multi-select values.
- **Number Mask (`numberMask`)**: Wildcard pattern (e.g., `*_sqm, age`) for casting numbers.
- **Parse Geodata (`parseGeodata`)**: Boolean to parse GPS coordinates into GeoJSON geometry.

### Outputs
- **Success Output**: Emits retrieved submissions, formatted data structures, or deletion/status confirmations.
- **Error Output**: Emits detailed KoboToolbox error messages and HTTP status codes.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch and Validate Submissions
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

- Use tokens with minimum necessary permissions (e.g., validate permissions for status updates).
- Protect sensitive personally identifiable information (PII) collected in humanitarian and field surveys.
<!-- /SECTION: security -->

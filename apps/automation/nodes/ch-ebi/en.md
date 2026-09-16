---
node_id: "ch-ebi"
title: "ChEBI"
description: "Search chemical entities, retrieve compound structures and ontology relationships, and calculate molecular properties with ChEBI 2.0 WebServices."
category: "Healthcare & Life Sciences"
subcategory: "Biology & Life Sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-09-15"
author: "Fusion Team"
tags:
  - chebi
  - chemistry
  - compounds
  - ontology
  - biology
  - life-sciences
related_nodes:
  - log
---

<!-- SECTION: header -->
# ChEBI

> **Category:** Healthcare & Life Sciences | **Subcategory:** Biology & Life Sciences | **Type:** Action Node

Access Chemical Entities of Biological Interest through ChEBI 2.0 WebServices to find compounds, retrieve structures, explore chemical classifications, and calculate molecular properties.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **ChEBI** node connects workflows to ChEBI public endpoints at `https://www.ebi.ac.uk`. Select an operation and provide its search criteria, identifiers, structure, or formula. The node does not expose an API-key parameter or send an authentication token.

### Key Features

- **Search:** General text search, advanced criteria, and connectivity, similarity, or substructure searches.
- **Compound retrieval:** Look up individual or multiple compounds, retrieve MOL files, and request structure depictions.
- **Ontology navigation:** Retrieve parents, children, or descendants along a selected relationship.
- **Molecular calculations:** Request average mass, monoisotopic mass, molecular formula, or net charge.
- **Structure depiction:** Generate a depiction from a supplied structure using Indigo.
- **Dynamic input:** Fill selected missing parameters from incoming workflow data.

### Use Cases

- Enrich chemical datasets with ChEBI records and identifiers.
- Find related compounds by structure or ontology classification.
- Retrieve molecular structures for downstream analysis or display.
- Calculate properties from chemical formulas or molecular structures.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Operations

`operation` defaults to `esSearch`. Parameter names and operation values are case-sensitive.

| Operation | Purpose | Parameters to supply |
|-----------|---------|----------------------|
| `esSearch` | General search by name, identifier, formula, or other supported term | `term` |
| `advancedSearch` | Search with structured criteria | `fullSpecification` |
| `advancedSearchSourcesList` | Retrieve sources available for advanced search | None |
| `structureSearch` | Search by molecular structure | `structureSearchBody`, or `smiles` and `searchType`; optionally `similarity` |
| `getCompound` | Retrieve one compound | `chebiId` |
| `getCompoundStructure` | Retrieve a compound structure depiction | `compoundId`; optionally `width`, `height` |
| `getCompounds` | Retrieve multiple compounds | `compoundsBody` or `chebiIds` |
| `getMolfile` | Retrieve a compound MOL file | `compoundId` |
| `getOntologyAllChildrenInPath` | Retrieve descendants along a relationship using GET | `relation`, `entity` |
| `getOntologyAllChildrenInPathPost` | Retrieve descendants using POST | `ontologyBody`, or `relation` and `entity` |
| `getOntologyChildren` | Retrieve ontology children | `chebiId` |
| `getOntologyParents` | Retrieve ontology parents | `chebiId` |
| `calculateAvgMass` | Calculate average mass from a structure | `structure` |
| `calculateAvgMassFromFormula` | Calculate average mass from a formula | `formula` |
| `calculateMonoisotopicMass` | Calculate monoisotopic mass from a structure | `structure` |
| `calculateMonoisotopicMassFromFormula` | Calculate monoisotopic mass from a formula | `formula` |
| `calculateMolFormula` | Calculate a molecular formula | `structure` |
| `calculateNetCharge` | Calculate net charge | `structure` |
| `depictIndigo` | Depict a supplied structure | `structure`; optionally `width`, `height`, `transbg` |
| `getStructure` | Retrieve a structure through the structure endpoint | `compoundId`; optionally `width`, `height` |

Most operation-specific fields are optional in the schema. POST body construction checks for missing required content, but missing GET identifiers or search criteria can reach the service and cause an API error. Supply the parameters listed above for the selected operation.

### Search and Result Options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `term` | `string` | — | General search term, such as `glucose` or `paracetamol` |
| `fullSpecification` | `string` | — | JSON-encoded advanced-search specification |
| `page` | `number` | `1` | Result page for text, advanced, structure, and descendant searches |
| `size` | `number` | `15` | Page size for those searches; the node does not automatically retrieve subsequent pages |
| `threeStarOnly` | `boolean` | `true` | Restrict advanced, structure, and descendant searches to three-star entries; `false` includes two- and three-star entries |
| `hasStructure` | `boolean` | — | Filter advanced and descendant searches by presence or absence of a structure; omit to avoid this filter |
| `download` | `boolean` | `false` | Request download-format results for advanced, structure, and descendant searches |

### Compound and Depiction Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `chebiId` | `string` | — | Identifier for compound and direct ontology lookups, for example `CHEBI:46195` |
| `compoundId` | `number` | — | Numeric compound identifier for structure and MOL-file retrieval |
| `chebiIds` | `string` | — | Comma-separated identifiers for `getCompounds`; whitespace around each identifier is trimmed when constructing the body |
| `compoundsBody` | `string` | — | JSON request body for `getCompounds`; takes precedence over the body generated from `chebiIds` |
| `onlyOntologyParents` | `boolean` | `false` | Request only ontology parents with `getCompound` |
| `onlyOntologyChildren` | `boolean` | `false` | Request only ontology children with `getCompound` |
| `width` | `number` | `300` | Depiction width in pixels for `getCompoundStructure`, `getStructure`, and `depictIndigo` |
| `height` | `number` | `300` | Depiction height in pixels for the same operations |
| `transbg` | `boolean` | `false` | Request a transparent background for `depictIndigo` |

Use a prefixed string such as `CHEBI:46195` for `chebiId`, and a number such as `46195` for `compoundId`. The node URL-encodes identifiers but does not normalize or validate their format.

### Ontology Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `relation` | `enum` | Relationship to follow for descendant searches |
| `entity` | `string` | Starting ChEBI identifier, for example `CHEBI:30879` |
| `ontologyBody` | `string` | JSON body for `getOntologyAllChildrenInPathPost`; takes precedence over the generated body |

Supported `relation` values: `has_functional_parent`, `has_parent_hydride`, `has_part`, `has_role`, `is_a`, `is_conjugate_acid_of`, `is_conjugate_base_of`, `is_enantiomer_of`, `is_part_of`, `is_substituent_group_from`, and `is_tautomer_of`.

### Structure Search and Calculation Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `smiles` | `string` | SMILES structure for `structureSearch`, for example `c1ccccc1` |
| `searchType` | `enum` | `connectivity`, `similarity`, or `substructure`; no default |
| `similarity` | `number` | Similarity threshold from `0.4` to `1.0`, inclusive; no default |
| `structureSearchBody` | `string` | JSON body for structure search; takes precedence over the generated body |
| `structure` | `string` | SMILES or MOLFILE/SDF text for structure calculations and Indigo depiction |
| `formula` | `string` | Chemical formula for formula-based mass calculations, for example `C2H6O` |

### JSON Body Fields

`fullSpecification`, `compoundsBody`, `ontologyBody`, and `structureSearchBody` must contain JSON **as a string**, not a nested configuration object. The node sends supplied body strings without parsing or validating their JSON syntax. Structure and formula calculations instead send plain text.

When using an explicit body, avoid also setting conflicting convenience fields: some are still sent as URL query parameters even though the explicit body takes precedence during body construction.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Incoming workflow data; selected values can fill missing configuration |
| `success` | JSON value, string, or image object | Service response, decoded according to its content type |
| `error` | Runtime error | Failed request or invalid request-body configuration |

### Dynamic Input Resolution

Automatic input resolution runs only when **both `chebiId` and `compoundId` are unset** and incoming data is truthy. If either identifier is configured, all automatic input resolution is skipped, including search terms and structures. Configured values take precedence over incoming values.

For object input, the node checks these fields in order:

| Configuration field | Incoming fields, in priority order |
|---------------------|------------------------------------|
| `chebiId` | `chebiId`, `chebi_id`, `id` |
| `compoundId` | `compoundId`, `compound_id`, `id` |
| `term` | `term`, `query` |
| `smiles` | `smiles`, `structure` |
| `formula` | `formula` |
| `structure` | `structure`, `smiles` |

A number fills `compoundId`. A string is first passed through integer parsing: if parsing succeeds, it fills `compoundId`; otherwise it fills `chebiId`. For compound lookups, prefer `{"chebiId":"CHEBI:46195"}` over an unprefixed numeric string. For text searches, use `{"term":"glucose"}` rather than a plain string. Prefer explicit identifier fields over the ambiguous `id` alias.

Incoming data does not override `operation` or automatically populate pagination, relationship controls, search type, or JSON body fields.

### Response Formats

- **JSON:** Returned directly, without a fixed wrapper or standardized result fields.
- **Plain text:** Returned as a string when the response content type is `text/plain`.
- **SVG:** Returned as SVG text when the content type is `image/svg+xml`.
- **PNG:** Returned as an object containing `image` (a `data:image/png;base64,...` URL) and `contentType` (`image/png`).

Other content types fall back to JSON parsing. A download response in another format may therefore fail to decode. Connect a Log node to inspect the actual response before mapping downstream fields.

Errors are thrown with a message beginning `ChEBI <operation> failed:`. The workflow runtime determines the error output envelope; the node does not construct a fixed error JSON object.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

These examples show node parameter configurations, not captured service responses.

### Search by Name

```json
{
  "operation": "esSearch",
  "term": "glucose",
  "page": 1,
  "size": 15
}
```

### Retrieve One Compound

```json
{
  "operation": "getCompound",
  "chebiId": "CHEBI:46195"
}
```

### Retrieve Multiple Compounds

```json
{
  "operation": "getCompounds",
  "chebiIds": "CHEBI:18357,CHEBI:22982"
}
```

Alternatively, provide the request body explicitly:

```json
{
  "operation": "getCompounds",
  "compoundsBody": "{\"chebi_ids\":[\"CHEBI:18357\",\"CHEBI:22982\"]}"
}
```

### Advanced Formula Search

```json
{
  "operation": "advancedSearch",
  "fullSpecification": "{\"formula_specification\":{\"and_specification\":[{\"term\":\"C6H12O7\"}]}}",
  "threeStarOnly": true,
  "page": 1,
  "size": 15
}
```

### Search for Similar Structures

```json
{
  "operation": "structureSearch",
  "smiles": "c1ccccc1",
  "searchType": "similarity",
  "similarity": 0.9
}
```

The equivalent explicit body uses `structure` and `type`, rather than the configuration names `smiles` and `searchType`:

```json
{
  "operation": "structureSearch",
  "structureSearchBody": "{\"structure\":\"c1ccccc1\",\"type\":\"similarity\",\"similarity\":0.9}"
}
```

### Explore Ontology Descendants

```json
{
  "operation": "getOntologyAllChildrenInPathPost",
  "relation": "is_a",
  "entity": "CHEBI:30879",
  "page": 1,
  "size": 15
}
```

### Calculate Mass from a Formula

```json
{
  "operation": "calculateAvgMassFromFormula",
  "formula": "C2H6O"
}
```

### Calculate a Molecular Formula from a Structure

```json
{
  "operation": "calculateMolFormula",
  "structure": "CCO"
}
```

### Retrieve a MOL File

```json
{
  "operation": "getMolfile",
  "compoundId": 46195
}
```

### Depict a Structure

```json
{
  "operation": "depictIndigo",
  "structure": "CCO",
  "width": 400,
  "height": 300,
  "transbg": true
}
```

### Search from Incoming Data

Configure the node with `{"operation":"esSearch"}`, leave both identifier parameters unset, and send this object to its input:

```json
{
  "query": "paracetamol"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The supplied workflow connects **Manual Trigger → ChEBI → Log**. It uses `esSearch` with the term `glucose` and sends the success response to Log.

```fusion-workflow
src: example.workflow.json
title: Search ChEBI for glucose
```

### Common Patterns

- **Compound enrichment:** Incoming identifiers → ChEBI compound lookup → Data storage.
- **Chemical search:** Search criteria → ChEBI → Result processing.
- **Ontology exploration:** ChEBI descendant search → Classification report.
- **Structure analysis:** Incoming molecular structure → ChEBI calculation or depiction → Result display.

Connect the error output to an error-handling step when building a production workflow. Pagination and retries must be arranged in the surrounding workflow; the node makes one request per execution.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | Likely cause | Action |
|-------|--------------|--------|
| `Invalid request body: ... is required ...` | Missing content for the selected POST operation | Supply the body field or its supported fallback parameters from the operations table |
| API rejects a JSON body | Invalid JSON or unsupported specification | Use a valid JSON-encoded string and the body keys shown in the examples |
| `ChEBI Resource not found (404)` | Missing, incorrect, or unavailable identifier or resource | Check the identifier and use `chebiId` or `compoundId` as required by the operation |
| Search input is ignored | Plain string input, or an identifier is already configured | Supply an object containing `term` or `query` and leave both identifiers unset for automatic resolution |
| Structure search fails validation | Missing search type or an out-of-range similarity value | Set `searchType` and keep `similarity` between `0.4` and `1.0` |
| Calculation fails | Missing or malformed structure or formula | Use `structure` for structure operations and `formula` for formula-based operations |
| Response cannot be parsed | Response type falls outside JSON, plain text, SVG, and PNG handling | Try `download: false` for supported searches and inspect the service error |
| HTTP error or network failure | Service rejection, unavailability, or connectivity issue | Check the reported HTTP status and message; retry through the workflow when appropriate |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Log** — Inspect search responses, compound records, calculation results, and returned image data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-15 | Initial documentation for ChEBI operations, parameters, input resolution, response handling, and the supplied search workflow |

<!-- /SECTION: changelog -->

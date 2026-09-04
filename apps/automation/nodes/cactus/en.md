---
node_id: "cactus"
title: "Cactus NCI"
description: "Convert chemical structures between formats and retrieve chemical identifiers, properties, and 2D/3D structure files using the NCI/CADD Chemical Identifier Resolver."
category: "Healthcare & Life Sciences"
subcategory: "Chemistry & Life Sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - cactus
  - nci
  - chemistry
  - cheminformatics
  - smiles
  - inchi
  - cas
  - molecular-weight
  - chemical-structure
  - life-sciences
related_nodes:
  - pub-chem
  - chembl
  - ncbi-pccompound
  - ncbi-pcsubstance
---

<!-- SECTION: header -->
# Cactus NCI

> **Category:** Healthcare & Life Sciences&nbsp;&nbsp;|&nbsp;&nbsp;**Subcategory:** Chemistry & Life Sciences&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Convert chemical structures between standard formats and retrieve molecular properties, identifiers, names, and 2D/3D structure files using the National Cancer Institute (NCI/CADD) Chemical Identifier Resolver (CACTUS).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Cactus NCI** node provides direct access to the National Cancer Institute’s Computer-Aided Drug Design Group (NCI/CADD) Chemical Identifier Resolver (CACTUS). It acts as a universal chemical converter and property resolver, translating any supported chemical representation into another format, identifier, physicochemical property, or molecular structure file.

Whether working with common drug names (e.g., `aspirin`, `caffeine`, `paracetamol`), CAS registry numbers (e.g., `50-78-2`), SMILES strings, or InChI identifiers, this node allows automated molecular data normalization, property calculation, and drug-likeness screening directly inside Fusion workflows.

### Key Features

- **Multi-Format Structure Conversion:** Seamlessly convert between common names, IUPAC names, CAS numbers, SMILES, InChI, and InChIKey.
- **24 Specialized Methods:** Retrieve molecular weights, chemical formulas, ring counts, rotatable bonds, hydrogen bond donors/acceptors, and structure files.
- **Drug-Likeness Evaluation:** Instantly query Lipinski's Rule of Five violation counts (`rule_of_5_violation_count`) for early-stage pharmacological assessment.
- **Structural Hashcodes:** Generate unique NCI structural representations including FICTS, FICuS, uuuuu, and HASHISY codes.
- **File & Diagram Retrieval:** Obtain MDL Structure-Data Files (`file?format=sdf`), 2D diagram images, and 3D interactive models (`twirl`).
- **Dynamic Input Resolution:** Automatically infer chemical structures from incoming workflow payloads when not statically configured.
- **Zero Authentication Required:** Connect directly to the public NCI CACTUS database without needing API keys or credentials.

### Use Cases

- **Chemical Entity Normalization:** Standardize heterogeneous compound lists from vendor catalogs, CSVs, or databases into canonical SMILES or InChIKey.
- **Drug Discovery & Pharmacological Filtering:** Automate Lipinski Rule of 5 checks, molecular weight constraints, and hydrogen bonding profiles for candidate screening.
- **Compound Property Enrichment:** Enrich chemical inventories with calculated molecular formulas, CAS numbers, and IUPAC nomenclature.
- **Structure Visualization Preparation:** Fetch SDF coordinate files or 2D image previews for automated report generation or user dashboards.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `structure` | `string` | ❌ No* | `aspirin` | The chemical identifier to resolve (name, SMILES, InChI, InChIKey, CAS, etc.). Optional if supplied via incoming data. |
| `method` | `enum` | ✅ Yes | `smiles` | The conversion method or property to retrieve (see [Available Methods](#available-methods)). |

*\* Note: If `structure` is left empty in the node parameters, the node will dynamically look for a structure identifier in the incoming workflow data.*

---

### Dynamic Structure Resolution

When `structure` is omitted from the node configuration, the node dynamically resolves the target molecule from the incoming `input` payload using the following precedence:

1. **Direct String:** If `input` is a raw string (e.g., `"caffeine"` or `"CCO"`), it is used directly.
2. **Object Property Fallback:** If `input` is a JSON object, the node inspects the following keys in order:
   - `input.structure`
   - `input.name`
   - `input.smiles`
   - `input.inchi`
   - `input.inchikey`
   - `input.cas`
   - `input.id`
   - `input.identifier`

If none of these fields contain a valid string, the node throws a descriptive validation error.

---

### Available Methods

The `method` parameter selects the operation to perform against the NCI CACTUS service:

#### 1. Identifiers & Structural Representations

| Method | Output Description | Example Result (for `aspirin`) |
|--------|---------------------|--------------------------------|
| `smiles` | Canonical SMILES string | `CC(=O)Oc1ccccc1C(=O)O` |
| `stdinchi` | Standard InChI string | `InChI=1S/C9H8O4/c1-6(10)13-8-5-3-2-4-7(8)9(11)12/h2-5H,1H3,(H,11,12)` |
| `stdinchikey` | 27-character standard hashed InChIKey | `BSYNRYMUTXBXSQ-UHFFFAOYSA-N` |
| `cas` | Primary CAS Registry Number(s) | `50-78-2` |
| `chemspider_id` | ChemSpider compound identifier | `2157` |
| `ficts` | FICTS hashcode (Full, Isotope, Charge, Tautomer, Stereochemistry) | Hashcode string |
| `ficus` | FICuS hashcode (Full, Isotope, Charge, Unconstrained tautomer, Stereo) | Hashcode string |
| `uuuuu` | NCI uuuuu (Unique Unconstrained Unit Understood Unit) identifier | Hashcode string |
| `hashisy` | HASHISY representation string | Hashcode string |

#### 2. Names & Taxonomy

| Method | Output Description | Example Result (for `aspirin`) |
|--------|---------------------|--------------------------------|
| `iupac_name` | Systematic IUPAC chemical name | `2-acetyloxybenzoic acid` |
| `names` | Full list of all recorded chemical synonyms, trade names, and registry names (newline-separated) | `Aspirin\nAcetylsalicylic acid\n2-Acetoxybenzoic acid...` |

#### 3. Physicochemical Properties & Molecular Counts

| Method | Output Description | Example Result (for `aspirin`) |
|--------|---------------------|--------------------------------|
| `mw` | Molecular Weight in g/mol | `180.1574` |
| `formula` | Hill-system chemical formula | `C9H8O4` |
| `h_bond_donor_count` | Number of Hydrogen Bond Donors | `1` |
| `h_bond_acceptor_count` | Number of Hydrogen Bond Acceptors | `4` |
| `h_bond_center_count` | Total Hydrogen Bond Centers (donors + acceptors) | `5` |
| `rule_of_5_violation_count` | Lipinski's Rule of Five violation count (0–4) | `0` |
| `rotor_count` | Number of rotatable bonds (molecular flexibility) | `3` |
| `effective_rotor_count` | Count of effective rotatable bonds | `3` |
| `ring_count` | Total count of rings in the molecule | `1` |
| `ringsys_count` | Count of independent ring systems | `1` |

#### 4. Structure Files & Visuals

| Method | Output Description | Example Result |
|--------|---------------------|----------------|
| `file?format=sdf` | Complete MDL Structure-Data File (SDF / Molfile) containing 2D/3D atomic coordinates and bond tables | Multiline SDF formatted text |
| `image` | 2D structure diagram representation pointer or URL | Image reference data |
| `twirl` | 3D interactive molecular visualization (VRML format) | 3D coordinate/scene data |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. Can be a string identifier or an object containing fields such as `structure`, `name`, `smiles`, `inchi`, `cas`, or `id`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when CACTUS successfully resolves the query. Contains the resolved text encoded in `data`. |
| `error` | `Error` | Emitted when the structure is missing, not found (404), or if the NCI API encounters an error. |

---

### Output Schema

The node returns an object containing a single `data` field with the JSON-encoded response string:

```json
{
  "data": "\"C9H8O4\\n\""
}
```

> **Note on String Encoding:** Because the CACTUS API returns raw text (`text/plain`), the node encodes the response with `jsonEncode()`. To access the clean value in downstream nodes, you can reference it directly as a string or parse it using `JSON.parse(input.data)` if needed.

---

### Response Examples by Method

#### Example 1: Molecular Formula (`formula`)
**Query:** `structure: "aspirin"`, `method: "formula"`
```json
{
  "data": "\"C9H8O4\\n\""
}
```

#### Example 2: Molecular Weight (`mw`)
**Query:** `structure: "aspirin"`, `method: "mw"`
```json
{
  "data": "\"180.1574\\n\""
}
```

#### Example 3: Standard InChIKey (`stdinchikey`)
**Query:** `structure: "aspirin"`, `method: "stdinchikey"`
```json
{
  "data": "\"BSYNRYMUTXBXSQ-UHFFFAOYSA-N\\n\""
}
```

#### Example 4: IUPAC Name (`iupac_name`)
**Query:** `structure: "CCO"`, `method: "iupac_name"`
```json
{
  "data": "\"ethanol\\n\""
}
```

#### Example 5: Lipinski Rule of 5 Violations (`rule_of_5_violation_count`)
**Query:** `structure: "aspirin"`, `method: "rule_of_5_violation_count"`
```json
{
  "data": "\"0\\n\""
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Cactus NCI Chemical Property and Structure Resolver
```

---

### Common Implementation Patterns

#### Pattern 1: Chemical Structure Normalization
1. Receive incoming raw compound data from a database or webhook (e.g., with common names like `"paracetamol"`).
2. Wire the data to a **Cactus NCI** node configured with:
   - `method`: `smiles`
3. Connect the output to a second **Cactus NCI** node with:
   - `method`: `stdinchikey`
4. The result produces canonical, hashable identifiers ready for deduplication and database indexing.

#### Pattern 2: Drug-Likeness & Lipinski Filter
1. Pass a library of SMILES strings through a **Cactus NCI** node configured with `method: "rule_of_5_violation_count"`.
2. Connect to an **If-Else** node checking if `parseInt(JSON.parse(input.data)) === 0`.
3. Valid drug candidates proceed to downstream scoring, while failing candidates are routed to an exclusion log.

#### Pattern 3: Structure File Retrieval (SDF Generation)
1. Supply a CAS number (e.g. `50-78-2`) to **Cactus NCI** with:
   - `method`: `file?format=sdf`
2. Send the resulting SDF text to a **Convert to File** or storage node to archive 3D atomic coordinates for computational chemistry workflows.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `structure is required. Provide a chemical structure identifier (e.g., 'aspirin', SMILES, InChI, CAS number).`
- **Cause:** No `structure` was defined in the node configuration, and the incoming payload either did not exist or did not contain any recognized structure property (`structure`, `name`, `smiles`, `inchi`, `inchikey`, `cas`, `id`, `identifier`).
- **Solution:** Specify a static value in the `structure` parameter, or ensure the upstream node emits an object with one of the supported field names.

#### `Cactus NCI Resource not found (404): Structure or method not found`
- **Cause:** The NCI CACTUS database does not recognize the specified compound name, the SMILES string is chemically invalid, or the method requested cannot be computed for that compound.
- **Solution:** Verify the spelling of the chemical name, test the SMILES string for syntax errors, or try querying by CAS number or InChI instead.

#### Output string contains trailing `\n` or quotation marks
- **Cause:** The CACTUS service responds in plain text format ending with a newline, which the node serializes using `JSON.stringify()`.
- **Solution:** In downstream JavaScript or expressions, use `JSON.parse(input.data).trim()` or `input.data.replace(/["\n]/g, '')` to obtain a sanitized string or numeric value.

### Error Reference

| Error Pattern | Cause | Resolution |
|---------------|-------|------------|
| `Cactus NCI Resource not found (404)` | Unknown compound or unsupported method | Verify chemical syntax or supply standard InChI / SMILES |
| `structure is required...` | Missing input data and empty configuration | Provide a structure string or ensure upstream emits `name` or `smiles` |
| `Cactus NCI API Error: 500 / 503` | NCI service temporary downtime or network issue | Implement a retry mechanism or check NCI server status |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [PubChem](../pub-chem/en.md) – Search compounds, bioassays, and patents in the NCBI PubChem database
- [ChEMBL](../chembl/en.md) – Query bioactive molecules, drug targets, and bioactivities
- [NCBI PCCompound](../ncbi-pccompound/en.md) – Access compound records from NCBI Entrez
- [NCBI PCSubstance](../ncbi-pcsubstance/en.md) – Query substance depositions in NCBI PubChem

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial release with support for 24 conversion methods, dynamic input resolution, and error routing |

<!-- /SECTION: changelog -->

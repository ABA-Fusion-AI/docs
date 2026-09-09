---
node_id: "nvidia-bionemo-smiles-to-sdf"
title: "BioNeMo: SMILES to SDF"
description: "Convert SMILES strings to SDF molecular structure files using the CACTUS chemistry service"
category: "Healthcare & Life Sciences"
subcategory: "BioNeMo"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - healthcare
  - life-sciences
  - bionemo
  - chemistry
  - smiles
  - sdf
  - molecular-structure
related_nodes:
  - cactus
  - log
---

<!-- SECTION: header -->

# BioNeMo: SMILES to SDF

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Convert one or more SMILES strings into SDF molecular structure format using the CACTUS chemistry service.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: SMILES to SDF** node converts chemical structures represented as SMILES into SDF format. It supports a single SMILES string or a comma-separated list of SMILES strings and can assign a filename to the generated SDF output.

The node uses the CACTUS chemistry service to perform the conversion. The example workflow sends the conversion result to a Log node for inspection.

### Key Features

- **SMILES conversion:** Convert SMILES input into SDF format
- **Single or multiple molecules:** Process one SMILES string or comma-separated SMILES strings
- **Custom filename:** Specify the filename for the generated SDF output
- **Workflow outputs:** Route successful conversions or errors to downstream nodes
- **No credentials in the example:** The example contains no API key, token, secret, password, or authorization header

### Use Cases

- Convert molecular SMILES data into structure files
- Prepare compounds for cheminformatics or molecular modeling workflows
- Transform comma-separated compound lists into SDF output
- Pass generated molecular files to downstream storage, analysis, or logging nodes
- Normalize chemical data for life-sciences automation

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `smiles` | `string` | ✅ Yes | — | A single SMILES string or a comma-separated list of SMILES strings |
| `filename` | `string` | No | — | Filename for the generated SDF output |

### SMILES Input

A single molecule can be provided as a SMILES string:

```text
CCO
```

Multiple molecules can be provided as a comma-separated list:

```text
CCO,CC(=O)Oc1ccccc1C(=O)O
```

Use valid SMILES notation for each molecule. Invalid or unsupported structures are reported through the node's `error` output.

### Output Filename

The `filename` parameter controls the name assigned to the generated SDF output.

Example:

```json
{
  "smiles": "CCO",
  "filename": "ethanol.sdf"
}
```

The example workflow uses `ethanole.sdf` as its filename.

### Authentication and Secrets

No authentication parameter is present in the example workflow. The workflow has an empty `secrets` object, and the node configuration contains no API key, access token, secret, password, bearer token, or authorization header.

If the CACTUS service or the Fusion runtime later requires authentication, credentials should be stored in Fusion's protected credential or secret system rather than in workflow parameters or exported examples.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node |

The node configuration accepts the `smiles` and `filename` parameters. A preceding node can be used to provide or prepare input data before conversion.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `string` | Generated SDF content or the successful conversion result |
| `error` | `object` or `string` | Input validation, conversion, service, or network failure |

The exact success response shape depends on the CACTUS service response and the Fusion node runtime. Use a Log node or downstream processing node to inspect the returned result.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Convert One SMILES String

Convert ethanol from SMILES to SDF.

```json
{
  "smiles": "CCO",
  "filename": "ethanol.sdf"
}
```

### Convert Multiple SMILES Strings

Convert a comma-separated list of molecules.

```json
{
  "smiles": "CCO,CC(=O)Oc1ccccc1C(=O)O",
  "filename": "compounds.sdf"
}
```

Each SMILES value is sent for conversion and included in the resulting SDF output when the CACTUS service successfully resolves it.

### Use a Dynamic SMILES Input

A previous node can prepare a SMILES value before the BioNeMo node executes.

Example input:

```json
{
  "smiles": "C1=CC=CC=C1"
}
```

Configure the node's filename separately if a specific output filename is required.

### Example Error Case

An invalid SMILES value may cause the conversion to fail:

```json
{
  "smiles": "not-a-valid-smiles",
  "filename": "invalid.sdf"
}
```

Inspect the `error` output for details from validation or the CACTUS chemistry service.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow uses a Manual Trigger, the BioNeMo conversion node, and a Log node:

```fusion-workflow
src: example.workflow.json
title: Convert SMILES to SDF with BioNeMo
```

### Common Patterns

- **Single molecule:** Manual Trigger → BioNeMo: SMILES to SDF → Log
- **Compound list:** Data source → BioNeMo: SMILES to SDF → File or storage node
- **Dynamic conversion:** Function → BioNeMo: SMILES to SDF → Downstream analysis
- **Error handling:** BioNeMo: SMILES to SDF `error` output → Notification or recovery node
- **Structure preparation:** Input data → SMILES validation or transformation → BioNeMo: SMILES to SDF

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### SMILES conversion failed

**Cause:** The input contains invalid, unsupported, or incorrectly formatted SMILES notation.

**Solution:** Validate each SMILES string and test a single molecule before processing a comma-separated list.

#### Multiple molecules are not processed as expected

**Cause:** The input list is not formatted as comma-separated SMILES strings, or one molecule contains an invalid value.

**Solution:** Separate molecules with commas and verify every SMILES value independently.

#### Output filename is incorrect

**Cause:** The `filename` parameter is missing or contains an unintended extension or spelling.

**Solution:** Provide a clear filename, such as `compounds.sdf`, and verify it in the downstream node.

#### CACTUS service is unavailable

**Cause:** The external chemistry service cannot be reached or has rejected the request.

**Solution:** Check network connectivity, retry the workflow, and inspect the node's `error` output.

#### Node is not registered

**Cause:** The package containing the `nvidia-bionemo-smiles-to-sdf` node is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid SMILES | Malformed or unsupported chemical notation | Validate and correct the SMILES input |
| Empty input | No SMILES value was provided | Configure `smiles` or provide it from an upstream node |
| Conversion error | CACTUS could not resolve or convert the structure | Check the molecule and retry |
| Service or network error | CACTUS could not be reached or completed the request | Check connectivity and service availability |
| Output error | Result could not be returned or handled by the workflow | Inspect downstream configuration and the `success` output |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [Cactus NCI](../cactus/en.md) - Convert and resolve chemical structures through the CACTUS service
- [Log](./log.md) - Inspect conversion results during workflow development

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Added documentation for SMILES-to-SDF conversion and the example workflow |

<!-- /SECTION: changelog -->


---
node_id: "nvidia-bionemo-genmol-generate"
title: "BioNeMo: GenMol Generate"
description: "Generate fragment-based molecules using NVIDIA BioNeMo GenMol"
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
  - genmol
  - molecular-generation
  - fragments
  - drug-discovery
related_nodes:
  - nvidia-bionemo-filter-molecules
  - nvidia-bionemo-molmim-generate
  - nvidia-bionemo-smiles-to-sdf
  - log
---

<!-- SECTION: header -->

# BioNeMo: GenMol Generate

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Generate novel molecules from a molecular scaffold and fragment placeholders using NVIDIA BioNeMo GenMol.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: GenMol Generate** node performs fragment-based molecular generation. It accepts a SMILES string containing a GenMol fragment placeholder in the form `[*{min-max}]` and generates candidate molecules that replace the placeholder with a fragment of the requested size range.

The example workflow uses QED scoring, generates five molecules, and sends the successful result to a Log node.

### Key Features

- **Fragment-based generation:** Generate molecules by replacing fragment placeholders in a scaffold
- **Scaffold control:** Provide a starting molecular structure as SMILES
- **Size constraints:** Use `[*{min-max}]` to define the fragment size range
- **Scoring:** Score or guide generation with a target such as QED
- **Sampling controls:** Configure temperature and noise
- **Uniqueness control:** Request unique generated molecules
- **NVIDIA authentication:** Authenticate requests with an NVIDIA API key

### Use Cases

- Expand molecular scaffolds with new chemical fragments
- Generate candidate compounds for drug-discovery workflows
- Explore structure variations around a known core
- Optimize or rank generated candidates with downstream nodes
- Prepare generated molecules for SDF conversion, filtering, or docking

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `scoring` | `enum` or `string` | ✅ Yes | `QED` in the example | Scoring property used to evaluate generated molecules |
| `apiKey` | `string` | ✅ Yes | — | NVIDIA API key used to authenticate the BioNeMo service |
| `smiles` | `string` | ✅ Yes | — | Scaffold SMILES containing a GenMol fragment placeholder |
| `num_molecules` | `number` | ✅ Yes | `5` in the example | Number of candidate molecules to generate |
| `temperature` | `number` | No | `1.5` in the example | Sampling temperature controlling generation diversity |
| `noise` | `number` | No | `1` in the example | Noise level used during molecular generation |
| `unique` | `boolean` | No | `true` in the example | Whether to request unique generated molecules |

### Example Configuration

```json
{
  "scoring": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smiles": "c1ccccc1.[*{5-15}]",
  "num_molecules": 5,
  "temperature": 1.5,
  "noise": 1,
  "unique": true
}
```

### Fragment Placeholders

GenMol fragment placeholders use this format:

```text
[*{min-max}]
```

For example:

```text
c1ccccc1.[*{5-15}]
```

This example provides a benzene scaffold and requests a generated fragment within the specified size range. Use a valid placeholder format supported by the configured GenMol service.

### Scoring

The example uses:

```text
QED
```

The `scoring` parameter identifies the property used to score generated molecules. Use a scoring option supported by the BioNeMo GenMol service.

### Sampling Controls

- `temperature` influences the diversity of generated candidates. Higher values may increase variation.
- `noise` controls the noise applied during sampling.
- `unique` requests that generated candidates be unique when set to `true`.
- `num_molecules` controls the number of candidates requested.

Higher sampling values may increase exploration and execution time.

### API Key and Secrets

The node requires an NVIDIA API key. The example workflow currently contains a hard-coded `apiKey` value in its node parameters.

Use a protected Fusion credential or secret reference instead of placing a real key directly in workflow JSON. Documentation and exported examples should contain only a placeholder:

```text
<NVIDIA_API_KEY>
```

**Security notice:** Rotate or revoke the exposed key in `apps/automation/nodes/nvidia-bionemo-genmol-generate/example.workflow.json` before sharing or committing the workflow.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node |

The node configuration defines the scaffold SMILES, scoring property, API key, molecule count, and sampling controls. An upstream node can prepare a dynamic scaffold or fragment specification before execution.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Successful GenMol result containing generated molecular candidates |
| `error` | `object` or `string` | Authentication, validation, service, network, or generation failure |

The exact success response depends on the BioNeMo GenMol service response and the Fusion runtime. Use a Log node to inspect the returned structure during workflow development.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Generate Molecules from a Benzene Scaffold

Generate five candidates by replacing a fragment placeholder attached to a benzene scaffold.

```json
{
  "scoring": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smiles": "c1ccccc1.[*{5-15}]",
  "num_molecules": 5,
  "temperature": 1.5,
  "noise": 1,
  "unique": true
}
```

### Request Unique Candidates

Set `unique` to `true` to request non-duplicate generated molecules.

```json
{
  "scoring": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smiles": "c1ccccc1.[*{5-15}]",
  "num_molecules": 5,
  "unique": true
}
```

### Dynamic Scaffold Input

An upstream node can provide a scaffold and fragment placeholder:

```json
{
  "smiles": "c1ccccc1.[*{5-15}]"
}
```

Keep the API key and generation settings in protected node configuration or secret management.

### Example Error Case

An invalid fragment placeholder may cause generation to fail:

```json
{
  "scoring": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smiles": "c1ccccc1.[invalid-fragment]",
  "num_molecules": 5,
  "temperature": 1.5,
  "noise": 1,
  "unique": true
}
```

Inspect the `error` output for validation or service details. Never log the real API key.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow connects a Manual Trigger to the GenMol node and sends the successful result to a Log node.

```fusion-workflow
src: example.workflow.json
title: Generate fragment-based molecules with GenMol
```

### Common Patterns

- **Scaffold expansion:** Manual Trigger → BioNeMo: GenMol Generate → Log
- **Dynamic generation:** Data source → Function → BioNeMo: GenMol Generate
- **Candidate filtering:** GenMol Generate → BioNeMo: Filter Molecules
- **Structure export:** GenMol Generate → BioNeMo: SMILES to SDF
- **Error handling:** GenMol Generate `error` output → Notification or recovery node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Authentication failed

**Cause:** The NVIDIA API key is missing, invalid, expired, revoked, or unavailable to the configured service.

**Solution:** Configure a valid NVIDIA API key through protected secret management. Rotate the key if it has been exposed in a workflow file.

#### Invalid fragment placeholder

**Cause:** The `smiles` value does not contain a valid `[*{min-max}]` placeholder or the range is malformed.

**Solution:** Use the supported placeholder syntax and verify that both minimum and maximum values are valid.

#### Invalid scaffold SMILES

**Cause:** The scaffold contains malformed SMILES notation or cannot be interpreted by GenMol.

**Solution:** Validate the scaffold independently and test with a known valid molecular structure.

#### Unsupported scoring option

**Cause:** The selected `scoring` value is not supported by the configured GenMol service.

**Solution:** Use a supported scoring property such as `QED`, depending on service capabilities.

#### Generation failed

**Cause:** GenMol could not generate candidates from the supplied scaffold, fragment range, or sampling configuration.

**Solution:** Start with a simple scaffold, a moderate fragment range, and the example sampling values. Adjust `temperature`, `noise`, and `num_molecules` gradually.

#### Request timed out

**Cause:** A large number of molecules or high sampling settings increased execution time.

**Solution:** Reduce `num_molecules`, `temperature`, or `noise` while troubleshooting.

#### Node is not registered

**Cause:** The package containing the `nvidia-bionemo-genmol-generate` node is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication error | Missing or invalid NVIDIA API key | Configure or rotate the protected API key |
| Invalid fragment syntax | Malformed `[*{min-max}]` placeholder | Correct the fragment placeholder |
| Invalid SMILES | Malformed scaffold structure | Validate the scaffold SMILES |
| Unsupported scoring | Target property is unavailable | Select a supported scoring option |
| Validation error | Missing or invalid generation parameter | Check numeric and boolean controls |
| Generation error | GenMol could not produce candidates | Adjust scaffold and sampling settings |
| Service or network error | BioNeMo service could not be reached | Check connectivity and service availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [BioNeMo: Filter Molecules](../nvidia-bionemo-filter-molecules/en.md) - Filter or select generated candidates
- [BioNeMo: MolMIM Generate](../nvidia-bionemo-molmim-generate/en.md) - Generate property-optimized molecules
- [BioNeMo: SMILES to SDF](../nvidia-bionemo-smiles-to-sdf/en.md) - Convert generated SMILES into SDF format
- [Log](./log.md) - Inspect GenMol generation results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Added GenMol molecular-generation documentation and security guidance |

<!-- /SECTION: changelog -->


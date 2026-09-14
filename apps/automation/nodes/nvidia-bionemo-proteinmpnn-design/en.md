---
node_id: "nvidia-bionemo-proteinmpnn-design"
title: "BioNeMo: ProteinMPNN Design"
description: "Design amino acid sequences for protein backbones using NVIDIA BioNeMo ProteinMPNN."
category: "Healthcare & Life Sciences"
subcategory: "BioNeMo"
version: "1.0.0"
language: "en"
last_updated: "2026-09-14"
author: "Fusion Team"
tags:
  - healthcare
  - life-sciences
  - bionemo
  - proteinmpnn
  - protein-design
  - rf-diffusion
  - bioinformatics
  - structure-design
related_nodes:
  - nvidia-bionemo-openfold2-predict
  - nvidia-bionemo-openfold3-predict
  - log
---

<!-- SECTION: header -->

# BioNeMo: ProteinMPNN Design

> **Category:** Healthcare & Life Sciences | **Subcategory:** BioNeMo | **Type:** Action Node

Design amino acid sequences for protein backbones with NVIDIA BioNeMo ProteinMPNN. This node is typically used after an RFdiffusion or backbone-generation step to recover a sequence that matches the target 3D structure.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: ProteinMPNN Design** node sends a protein backbone represented as a PDB structure to the NVIDIA BioNeMo ProteinMPNN endpoint and returns designed amino-acid sequences on the `success` output. It is designed for protein design workflows where you already have a backbone and want to generate plausible sequences that fit it.

The node normalizes the incoming PDB by removing all non-`ATOM` records before sending the request. If no `ATOM` lines remain, the node raises a validation error instead of sending an empty backbone.

### Key Features

- **Protein sequence design:** Generate sequences for fixed protein backbones
- **Post-RFdiffusion workflow:** Use this node after backbone generation or design tools
- **PDB validation:** Reject empty or invalid PDB payloads early
- **Optional chain control:** Restrict design to selected chains
- **Fixed-position handling:** Preserve certain residue positions if needed
- **Sampling control:** Tune sequence diversity through temperature and batch size

### Use Cases

- Generate candidate sequences from a designed protein backbone
- Feed PDB structures from RFdiffusion or structural modeling tools into sequence design
- Evaluate multiple sequence options for a target structure
- Create sequence libraries for downstream experimental screening
- Build protein design pipelines that combine structure generation and sequence optimization

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | NVIDIA API key used to authenticate the BioNeMo service |
| `pdb` | `string` | Yes | — | Protein backbone structure in PDB format. The node keeps only `ATOM` records before sending the request |
| `input_pdb_chains` | `array<string>` | No | — | Optional chain IDs to include from the provided structure |
| `fixed_positions_jsonl` | `string` | No | — | Optional JSONL-formatted fixed-position constraints for residue selection |
| `num_seq_per_target` | `number` | No | `8` | Number of sequences to generate per target. Valid range: `1` to `48` |
| `sampling_temp` | `number` | No | `0.1` | Sampling temperature used by ProteinMPNN. Valid range: `0` to `10` |

### Example Configuration

```json
{
  "apiKey": "{{secrets.nvidiaApiKey}}",
  "pdb": "ATOM...",
  "num_seq_per_target": 8,
  "sampling_temp": 0.1
}
```

### PDB Input

Provide a valid PDB backbone in plain text. The node strips out all lines that do not start with `ATOM`, so partial or malformed PDB files must be cleaned before execution.

Example:

```text
ATOM      1  CA  ALA A   1      16.742  22.456  18.910  1.00 10.00           C
ATOM      2  CA  GLY A   2      17.566  21.894  18.114  1.00 10.00           C
...
```

### Chain and Fixed-Position Controls

Use `input_pdb_chains` when the design should be limited to specific chains inside a multi-chain structure. Use `fixed_positions_jsonl` to keep specific residues fixed while the model designs the remaining positions.

### Temperature and Batch Size

- `num_seq_per_target` controls how many candidate sequences are produced for each target backbone.
- `sampling_temp` controls sequence diversity; lower values produce more conservative designs, while higher values introduce more variation.

### API Key and Secrets

The node requires an NVIDIA API key for the BioNeMo service. Do not commit a real key to a workflow export. Prefer Fusion secrets or environment-based configuration.

> No real API key or access token was found in the repository example workflow. The example currently uses an empty value, so it must be replaced with a secure secret reference such as `{{secrets.nvidiaApiKey}}` before execution.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional workflow data passed into the node. Use it to provide dynamic PDB or parameter values when mapped through the workflow runtime. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | The ProteinMPNN result payload returned by the BioNeMo service |
| `error` | `object` or `string` | Validation, authentication, service, or network failures |

The exact structure of the `success` payload depends on the BioNeMo deployment and the current Fusion runtime. Connect a Log node to inspect the returned object during workflow development.

### Example Success Payload

```json
{
  "sequences": [
    "MKTIIALSYIFCLVFADYKDDDDK",
    "MKTIIALSYVFCLVFADYKDDDDK"
  ],
  "metadata": {
    "num_seq_per_target": 2,
    "sampling_temp": 0.1
  }
}
```

This example is illustrative; use the actual runtime response to map downstream fields correctly.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic Sequence Design

```json
{
  "apiKey": "{{secrets.nvidiaApiKey}}",
  "pdb": "ATOM...",
  "num_seq_per_target": 8,
  "sampling_temp": 0.1
}
```

### Design Only Selected Chains

```json
{
  "apiKey": "{{secrets.nvidiaApiKey}}",
  "pdb": "ATOM...",
  "input_pdb_chains": ["A", "B"],
  "num_seq_per_target": 12,
  "sampling_temp": 0.2
}
```

### Keep Specific Residues Fixed

```json
{
  "apiKey": "{{secrets.nvidiaApiKey}}",
  "pdb": "ATOM...",
  "fixed_positions_jsonl": "{\"positions\":[1,2,3]}\n",
  "num_seq_per_target": 10
}
```

### Dynamic Input

A previous node can prepare the PDB text or parameter values before this node executes:

```json
{
  "pdb": "{{input.pdb}}",
  "num_seq_per_target": "{{input.numSeqPerTarget}}",
  "sampling_temp": "{{input.samplingTemp}}"
}
```

Keep the API key in secured configuration and never log it with the request payload.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow connects a Manual Trigger to ProteinMPNN Design and sends the result to a Log node.

```fusion-workflow
src: example.workflow.json
title: Design protein sequences with BioNeMo ProteinMPNN
```

### Typical Pattern

- **Manual Trigger** → **BioNeMo: ProteinMPNN Design** → **Log**
- **RFdiffusion / Backbone generation** → **BioNeMo: ProteinMPNN Design** → **Structure or sequence analysis**
- **Sequence optimization loop** → **ProteinMPNN Design** → **Filter or scoring node**

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Cause: Missing or invalid `apiKey`

The NVIDIA API key is missing, empty, invalid, expired, or not available to the BioNeMo service.

**Solution:** Provide a valid NVIDIA API key in the node configuration or reference it through a secure Fusion secret such as `{{secrets.nvidiaApiKey}}`.

### Cause: Empty or invalid PDB input

The `pdb` value is blank or contains no `ATOM` records after filtering.

**Solution:** Confirm that the input contains a real PDB structure with `ATOM` records. Remove headers, HETATM entries, and other unsupported lines before running the node.

### Cause: Parameter out of range

`num_seq_per_target` or `sampling_temp` exceeds the allowed limits for the service.

**Solution:** Keep `num_seq_per_target` between `1` and `48`, and keep `sampling_temp` between `0` and `10`.

### Cause: Service or network error

The BioNeMo endpoint is unreachable, rate-limited, or temporarily unavailable.

**Solution:** Check connectivity, confirm the service is available, and verify that the API key is valid and not revoked.

### Cause: Package or runtime issue

The node package is not loaded or registered in the Fusion runtime.

**Solution:** Reload the workflow environment and confirm that the `nvidia-bionemo-proteinmpnn-design` node is installed and available.

<!-- /SECTION: troubleshooting -->

---

## Related Nodes

- [BioNeMo: OpenFold2 Predict](../nvidia-bionemo-openfold2-predict/en.md) - Predict protein structures from sequence data
- [BioNeMo: OpenFold3 Predict](../nvidia-bionemo-openfold3-predict/en.md) - Predict biomolecular structures with the latest OpenFold model
- [Log](../log/en.md) - Inspect raw node outputs for debugging and data mapping

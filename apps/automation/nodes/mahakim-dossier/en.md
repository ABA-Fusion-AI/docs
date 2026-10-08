---
node_id: "mahakim-dossier"
title: "Mahakim Dossier"
description: "Retrieve jurisdictions and dossier information from the Mahakim API."
category: "government"
version: "1.0.0"
language: "en"
last_updated: "2026-10-08"
author: "Fusion Team"
tags:
  - mahakim
  - morocco
  - dossier
  - court
  - action
---

# Mahakim Dossier

> **Category:** government | **Type:** Action Node

The **Mahakim Dossier** node retrieves case information from the Mahakim API. Select an operation and supply its required identifiers in the node configuration.

## Operations

| Operation | What it retrieves | Required parameters |
| --- | --- | --- |
| `listJuridictions` | Jurisdictions for a Code D value | `codeD` |
| `getCarteDossier` | A dossier card | `numeroCompletDossier`, `idJuridiction` |
| `listDecisions` | Decisions for a dossier | `idDossiers` |
| `listParties` | Parties for a dossier | `idDossiers` |
| `listExperts` | Judicial expert information for a dossier | `idDossiers` |
| `listDepotDossier` | Deposits for a dossier | `idDossiers` |
| `listDossiersAttaches` | Attached dossiers | `dossier` |

`listJuridictions` is the default operation. The four operations using `idDossiers`, plus `listDossiersAttaches`, also accept `typeAffaire`.

## Configuration

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `operation` | `string` | No | `listJuridictions` | One of the operations listed above. |
| `codeD` | `string` | For `listJuridictions` | — | Code D, for example `7206`. |
| `numeroCompletDossier` | `string` | For `getCarteDossier` | — | Complete dossier number, for example `202472061032`. |
| `idJuridiction` | `string` | For `getCarteDossier` | — | Jurisdiction ID, for example `285`. |
| `idDossiers` | `string` | For `listDecisions`, `listParties`, `listExperts`, and `listDepotDossier` | — | Dossier ID used by the selected operation. |
| `dossier` | `string` | For `listDossiersAttaches` | — | Dossier value used to find attached dossiers. |
| `typeAffaire` | `string` | No | `DA` | Case type for operations that accept it. |
| `csrt` | `string` | No | Built-in token | CSRT token appended to each request. |
| `baseUrl` | `string` | No | `https://www.mahakim.ma/middleware/api/SuiviDossiers` | Base URL for the Mahakim dossier API. |

The schema marks operation-specific identifiers as optional so that only the relevant fields appear for the selected operation. The node checks the required identifiers when it runs and throws an error if one is missing.

### Examples

List jurisdictions:

```json
{
  "operation": "listJuridictions",
  "codeD": "7206"
}
```

Get a dossier card:

```json
{
  "operation": "getCarteDossier",
  "numeroCompletDossier": "202472061032",
  "idJuridiction": "285"
}
```

List decisions:

```json
{
  "operation": "listDecisions",
  "idDossiers": "<dossier ID>",
  "typeAffaire": "DA"
}
```

The node encrypts configured query values before sending them.

## Inputs and outputs

The node uses its configuration to build the request; it does not read incoming workflow data. Connect a trigger or another node to its input to run it.

On success, it returns the Mahakim API response. When the response is an object with a string-valued `data` field, the node decrypts that field. If the decrypted text is valid JSON, `data` becomes the parsed JSON value; otherwise, it remains a string. Other response fields are preserved. The exact fields depend on the selected operation and the API response.

On failure, the node throws an error through its error path. Missing identifiers, HTTP errors, invalid JSON responses, and decryption failures can cause the operation to fail.

## Request behavior

The node sends a `GET` request to the endpoint for the selected operation. It encrypts nonempty operation parameters and adds `csrt` without encryption.

| Operation | API endpoint |
| --- | --- |
| `listJuridictions` | `ListeJuridictions2Instance` |
| `getCarteDossier` | `CarteDossier` |
| `listDecisions` | `ListeDicisions` |
| `listParties` | `ListeParties` |
| `listExperts` | `ListeExpertisesJudiciaire` |
| `listDepotDossier` | `ListeDepotsDossier` |
| `listDossiersAttaches` | `ListeDossiersAttache` |

These endpoint names match the node implementation, including `ListeDicisions`.

## Troubleshooting

- **A required parameter is missing:** Set the identifier listed for the selected operation. For example, `getCarteDossier` needs both `numeroCompletDossier` and `idJuridiction`.
- **The API returns an HTTP error:** Check the identifier, case type, CSRT token, and API availability. The error includes the HTTP status and requested URL; handle logs containing dossier identifiers with care.
- **Response decryption fails:** Check that the response came from the expected API and that the configured base URL is correct. The node reports `Failed to decrypt response data` when it cannot decrypt a string-valued `data` field.

## Changelog

| Version | Date | Changes |
| --- | --- | --- |
| 1.0.0 | 2026-10-08 | Initial documentation. |

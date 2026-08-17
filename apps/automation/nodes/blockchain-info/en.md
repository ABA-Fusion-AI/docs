---
node_id: "blockchain-info"
title: "Blockchain.com Info"
description: "Retrieve blockchain network data, block details, and transaction information from the Blockchain.com API."
category: "business-commerce"
subcategory: "finance-accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - blockchain
  - finance
  - crypto
  - blockchain-data
  - network
related_nodes: []
---

<!-- SECTION: header -->
# Blockchain.com Info

> **Category:** Business & Commerce | **Type:** Action Node

Get raw blockchain information from the Blockchain.com API, including block data and transaction or network details for workflows focused on cryptocurrency and financial monitoring.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The Blockchain.com Info node calls the Blockchain.com API to fetch blockchain-related data. It is useful for retrieving block details, transaction metadata, and general network statistics in workflows that need on-chain information.

### Key Features

- **Block Data Retrieval:** Fetch specific block information using block identifiers or hashes.
- **Transaction Lookup:** Retrieve transaction-related metadata from the blockchain API.
- **Network Insights:** Access blockchain stats and status information.
- **Finance & Crypto Integration:** Use blockchain data in monitoring, reporting, and financial automation flows.
- **Workflow Error Handling:** Return failures through the standard error output.

### Typical Use Cases

- Monitor blockchain transactions
- Verify recent block activity
- Retrieve blockchain metadata for reporting
- Integrate cryptocurrency data into dashboards and operational workflows
- Support financial analysis that relies on chain activity

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `input` | `object` | ❌ No | — | Workflow input used to provide request data. |
| `block` | `string` | ❌ No | — | Block identifier or hash used for block-related requests. |
| `txHash` | `string` | ❌ No | — | Transaction hash used when requesting transaction details. |
| `apiKey` | `string` | ❌ No | — | Optional API credential if the selected Blockchain.com endpoint requires authentication. |

### Query Resolution

The node resolves values using the preceding workflow data first and then the configured node parameters.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional workflow data used to provide blockchain request details. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Blockchain.com API response data. |
| `error` | `Error` | Returned when the request fails or the API responds with an error. |

### Successful Response

```json
{
  "hash": "0000000000000000000000000000000000000000000000000000000000000000",
  "height": 857123,
  "time": 1712345678,
  "tx_count": 1321
}
```

### Error Response

```json
{
  "success": false,
  "error": "Request failed"
}
```

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Blockchain.com Info in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->

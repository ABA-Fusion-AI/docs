---
node_id: "checksum-calculator"
title: "Checksum Calculator"
description: "Calculate various checksums and hashes (MD5, SHA1, SHA256, SHA512, CRC32)"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - checksum
  - hash
  - md5
  - sha
  - crc32
related_nodes:
  - hash
  - hmac
  - binary-to-text
---

<!-- SECTION: header -->
# Checksum Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate checksums and hashes using MD5, SHA1, SHA256, SHA512, and CRC32.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Checksum Calculator** node calculates checksums and hashes from input text.

It supports seven operations:

- Calculate an MD5 hash.
- Calculate a SHA1 hash.
- Calculate a SHA256 hash.
- Calculate a SHA512 hash.
- Calculate a CRC32 checksum.
- Calculate all supported hashes at once.
- Verify a calculated hash against an expected hash.

### Key Features

- Calculates MD5 hashes.
- Calculates SHA1 hashes.
- Calculates SHA256 hashes.
- Calculates SHA512 hashes.
- Calculates CRC32 checksums.
- Calculates all supported values in a single operation.
- Verifies hashes using a selected algorithm.
- Returns structured calculation results.

### Processing Flow

```text
Input
  ↓
Select operation
  ↓
Calculate checksum or hash
  ↓
Verify expected hash when required
  ↓
Return result
```

### Use Cases

- Calculating hashes from text.
- Generating checksums for data.
- Checking data integrity.
- Comparing calculated and expected hashes.
- Generating multiple hash values from the same input.
- Preparing checksum values for downstream processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `string` | No | `md5` | Operation to perform: `md5`, `sha1`, `sha256`, `sha512`, `crc32`, `calculateAll`, or `verify`. |
| `input` | `string` | No | `""` | Input text used for checksum or hash calculation. |
| `algorithm` | `string` | No | `md5` | Algorithm used when the `verify` operation is selected. |
| `expectedHash` | `string` | No | `""` | Expected hash used when the `verify` operation is selected. |

### Operation

Select the checksum or hash operation to perform.

Supported values:

```text
md5
sha1
sha256
sha512
crc32
calculateAll
verify
```

### Input

Provide the text that should be processed.

Example:

```text
hello
```

### Algorithm

The `algorithm` parameter is used by the `verify` operation to select the hash algorithm.

Example:

```text
md5
```

### Expected Hash

The `expectedHash` parameter contains the hash that should be compared with the calculated value when using `verify`.

Example:

```text
5d41402abc4b2a76b9719d911017c592
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses the configured `input` string for checksum and hash calculations.

The selected `operation` determines which calculation is performed.

For the `verify` operation, `algorithm` selects the algorithm and `expectedHash` provides the value to compare.

### Single Algorithm Output

Operations such as `md5`, `sha1`, `sha256`, `sha512`, and `crc32` return an object containing the input, calculated hash, and algorithm.

Example:

```json
{
  "success": true,
  "input": "hello",
  "hash": "5d41402abc4b2a76b9719d911017c592",
  "algorithm": "md5"
}
```

### Calculate All Output

The `calculateAll` operation returns MD5, SHA1, SHA256, SHA512, and CRC32 values for the same input.

Example structure:

```json
{
  "success": true,
  "input": "hello",
  "md5": "5d41402abc4b2a76b9719d911017c592",
  "sha1": "aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d",
  "sha256": "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824",
  "sha512": "...",
  "crc32": "3610A686"
}
```

### Verify Output

The `verify` operation returns the calculated value, expected value, selected algorithm, and a `match` boolean.

Example:

```json
{
  "success": true,
  "match": true,
  "calculated": "5d41402abc4b2a76b9719d911017c592",
  "expected": "5d41402abc4b2a76b9719d911017c592",
  "algorithm": "md5"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Calculate MD5

**Input**

```text
hello
```

**Configuration**

```text
operation: md5
```

**Output**

```text
5d41402abc4b2a76b9719d911017c592
```

### Example 2: Calculate SHA256

**Input**

```text
hello
```

**Configuration**

```text
operation: sha256
```

**Output**

```text
2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

### Example 3: Calculate CRC32

**Input**

```text
hello
```

**Configuration**

```text
operation: crc32
```

**Output**

```text
3610A686
```

### Example 4: Calculate All Checksums

**Input**

```text
hello
```

**Configuration**

```text
operation: calculateAll
```

The node calculates MD5, SHA1, SHA256, SHA512, and CRC32 for the input.

### Example 5: Verify a Hash

**Input**

```text
hello
```

**Configuration**

```text
operation: verify
algorithm: md5
expectedHash: 5d41402abc4b2a76b9719d911017c592
```

The output includes a `match` value indicating whether the calculated hash matches the expected hash.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Checksum Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Expected hash is required

**Cause:** The `verify` operation was selected without providing an expected hash.

**Solution:** Provide a value in `expectedHash` before running the `verify` operation.

### Invalid Operation

**Cause:** The selected operation is not supported.

**Solution:** Select one of the supported operations: `md5`, `sha1`, `sha256`, `sha512`, `crc32`, `calculateAll`, or `verify`.

### Hash Does Not Match

**Cause:** The calculated hash does not match the expected hash.

**Solution:** Verify that the input, algorithm, and expected hash correspond to the same data and hashing method.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Hash** — Generate hash values using supported algorithms.
- **HMAC** — Generate keyed hash-based message authentication codes.
- **Binary to Text** — Convert binary data into text representation.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Checksum Calculator node. |

<!-- /SECTION: changelog -->
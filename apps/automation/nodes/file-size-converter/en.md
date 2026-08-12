---
node_id: "file-size-converter"
title: "File Size Converter"
description: "Convert between all digital storage units (B, KB, MB, GB, TB, PB) with binary and decimal options."
category: "Storage & Files"
subcategory: "Files & Documents"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:
  - storage
  - file size
  - conversion
  - bytes
  - units
related_nodes:
  - base-converter
  - binary-to-text
---

<!-- SECTION:header -->
# File Size Converter

> **Category:** Storage & Files | **Subcategory:** Files & Documents | **Type:** Action Node

Convert file size values between bytes and higher storage units using either decimal (1000-based) or binary (1024-based) unit definitions.

<!-- /SECTION:header -->

---

## Overview

The **File Size Converter** node supports four operations: `convert`, `format`, `parse`, and `calculateTransferTime`.

- `convert` converts a numeric size from one unit to another.
- `format` converts a byte count into a formatted size string and returns the equivalent values across all supported units.
- `parse` reads a size string like `100 MB` and returns the interpreted bytes and formatted output.
- `calculateTransferTime` estimates transfer duration for a byte count over a given bandwidth in megabits per second.

Supported units:

- Binary / 1024-based: `B`, `KB`, `MB`, `GB`, `TB`, `PB`
- Decimal / 1000-based: `kB`, `mB`, `gB`, `tB`, `pB`

### Features

- Convert between bytes, kilobytes, megabytes, gigabytes, terabytes, and petabytes.
- Format raw byte counts into human-readable size strings.
- Parse size expressions into bytes and normalized outputs.
- Calculate transfer time from bytes and Mbps.

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `convert` | Operation to perform: `convert`, `format`, `parse`, `calculateTransferTime`. |
| `value` | `number` | No | `1024` | Numeric amount used by `convert` along with `fromUnit` and `toUnit`. |
| `fromUnit` | `string` | No | `KB` | Source unit for `convert`. Supported units: `B`, `KB`, `MB`, `GB`, `TB`, `PB`, `kB`, `mB`, `gB`, `tB`, `pB`. |
| `toUnit` | `string` | No | `MB` | Target unit for `convert`. Same supported units as `fromUnit`. |
| `sizeString` | `string` | No | `""` | Size expression for `parse`, such as `100 MB` or `1.5 GB`. |
| `bytes` | `number` | No | `0` | Byte count used by `format` and `calculateTransferTime`. |
| `speedMbps` | `number` | No | `10` | Network speed in megabits per second for `calculateTransferTime`. |
| `decimals` | `number` | No | `2` | Number of decimal places for formatted output and unit conversion display. |

### Operation examples

Convert 2048 kilobytes to megabytes:

```json
{
  "operation": "convert",
  "value": 2048,
  "fromUnit": "KB",
  "toUnit": "MB",
  "decimals": 2
}
```

Format 1234567 bytes:

```json
{
  "operation": "format",
  "bytes": 1234567,
  "decimals": 2
}
```

Parse a size string:

```json
{
  "operation": "parse",
  "sizeString": "1.5 GB",
  "decimals": 2
}
```

Calculate transfer time for 5 GB over 50 Mbps:

```json
{
  "operation": "calculateTransferTime",
  "bytes": 5368709120,
  "speedMbps": 50
}
```

---

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Node reads `input.value` when parameter values are not provided directly. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Operation result object with fields dependent on the selected operation. |
| `error` | `Error` | Returned when parsing or conversion fails. |

#### Example outputs by operation

`convert` success payload:

```json
{
  "success": true,
  "value": 2,
  "fromUnit": "KB",
  "toUnit": "MB",
  "bytes": 2097152,
  "formatted": "2 MB",
  "allUnits": {
    "B": 2097152,
    "KB": 2048,
    "MB": 2,
    "GB": 0.001953125,
    "TB": 0.0000019073486328125,
    "PB": 0.000000001862645149230957
  }
}
```

`format` success payload:

```json
{
  "success": true,
  "bytes": 1234567,
  "formatted": "1.18 MB",
  "allUnits": {
    "B": 1234567,
    "KB": 1205.63,
    "MB": 1.18,
    "GB": 0.00,
    "TB": 0.00,
    "PB": 0.00
  }
}
```

`parse` success payload:

```json
{
  "success": true,
  "input": "1.5 GB",
  "value": 1.5,
  "unit": "GB",
  "bytes": 1610612736,
  "formatted": "1.50 GB"
}
```

`calculateTransferTime` success payload:

```json
{
  "success": true,
  "totalSeconds": 858.9934592,
  "formatted": "0h 14m 18s",
  "hours": 0,
  "minutes": 14,
  "seconds": 18
}
```

---

## Notes

- The node uses uppercase units (`KB`, `MB`, `GB`, `TB`, `PB`) for 1024-based conversions and lowercase units (`kB`, `mB`, `gB`, `tB`, `pB`) for 1000-based conversions.
- `parse` accepts a numeric value followed by a unit suffix, such as `100 MB` or `1.5 GB`.
- `calculateTransferTime` converts bytes to bits before dividing by the network speed in megabits per second.
- `decimals` controls rounding for formatted outputs.

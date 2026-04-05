---
node_id: "parse-base64"
title: "Parse Base64"
description: "Decode Base64 input and parse it as text, JSON, or CSV"
category: "utilities"
subcategory: "data-transformation"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - utility
  - base64
  - parser
  - transform
related_nodes:
  - parse-json
---

# Parse Base64

> **Category:** Utilities | **Type:** Action Node

Decodes Base64 and returns one of: plain text, parsed JSON, or parsed CSV.

## Configuration

| Parameter   | Type      | Required | Default | Description                              |
| ----------- | --------- | -------- | ------- | ---------------------------------------- |
| `data`      | `string`  | ❌ No    | —       | Base64 input string                      |
| `format`    | `enum`    | ❌ No    | `txt`   | `txt`, `json`, or `csv`                  |
| `delimiter` | `string`  | ❌ No    | `,`     | CSV delimiter (used with `format=csv`)   |
| `hasHeader` | `boolean` | ❌ No    | `true`  | CSV header mode (used with `format=csv`) |
| `encoding`  | `enum`    | ❌ No    | `utf8`  | `utf8`, `ascii`, `latin1`                |

### Accepted Input Sources

If `config.data` is not provided, the node checks:

- incoming string input
- `input.base64`
- `input.data`

### Output Behavior

- `format=txt`: returns decoded string
- `format=json`: returns parsed JSON object/array
- `format=csv`:
  - `hasHeader=true`: returns `Array<Record<string,string>>`
  - `hasHeader=false`: returns `string[][]`

### Validation Notes

- Accepts Base64 URL-safe variants (`-`, `_`) and optional `data:*;base64,` prefix.
- Throws on invalid/empty Base64.
- Throws `JSON parsing failed: ...` for invalid JSON content.

---
node_id: "replace"
title: "Replace"
description: "Replace text pattern with replacement string using string or regex matching."
category: "Data Transformation (ETL)"
subcategory: "Text & Document Transform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - text
  - string
  - replace
  - regex
  - transform
  - pattern
related_nodes:
  - extract
  - regex-match
  - regex-replace
  - function
---

<!-- SECTION: header -->
# Replace

> **Category:** Data Transformation (ETL) | **Subcategory:** Text & Document Transform | **Type:** Action Node

Replace text patterns with replacement strings in source text using simple string matching or regular expressions.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Replace** node finds and replaces text patterns within a source string. It supports both simple string replacement and advanced regex-based pattern matching, making it useful for text normalization, data cleaning, and content transformation workflows.

### Key Features

- **Simple String Replacement:** Replace exact text matches with a replacement string
- **Regex Support:** Use regular expressions for advanced pattern matching and replacement
- **Case Sensitivity:** Control whether matching is case-sensitive or case-insensitive
- **Global Replacement:** Replace all occurrences or just the first match
- **Capture Groups:** Support regex capture groups in replacement patterns
- **Workflow Friendly:** Combine with extraction, validation, or formatting nodes

### Typical Use Cases

- Normalize data by replacing unwanted characters or patterns
- Redact sensitive information from text
- Standardize formatting (e.g., phone numbers, dates, addresses)
- Clean up whitespace and special characters
- Transform text content for downstream processing
- Find and replace across large text blocks

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | ✅ Yes | — | Source text to search and replace within |
| `pattern` | `string` | ✅ Yes | — | Text or regex pattern to find |
| `replacement` | `string` | ✅ Yes | — | Replacement string (supports capture group references like `$1`, `$2` for regex) |
| `mode` | `enum` | ❌ No | `string` | Matching mode: `string` for literal text or `regex` for regex patterns |
| `caseSensitive` | `boolean` | ❌ No | `true` | Whether matching is case-sensitive |
| `global` | `boolean` | ❌ No | `true` | Replace all occurrences (true) or just first (false) |

### Mode Reference

| Mode | Description | Pattern Example |
|------|-------------|-----------------|
| `string` | Exact text matching | `Meryem` |
| `regex` | Regular expression matching | `[A-Z][a-z]+` or `\d{3}-\d{4}` |

### Example Configurations

**Simple String Replacement:**

```text
data: "I'm Meryem, I have 26y"
pattern: "Meryem"
replacement: "Huda"
mode: "string"
global: true
```

**Regex Pattern with Capture Groups:**

```text
data: "Call me at 555-1234"
pattern: "(\d{3})-(\d{4})"
replacement: "$1 $2"
mode: "regex"
global: true
```

**Case-Insensitive Replacement:**

```text
data: "The Quick Brown Fox"
pattern: "quick"
replacement: "slow"
mode: "string"
caseSensitive: false
global: true
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data; can supply `data` parameter value via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Result object containing the modified text and metadata |
| `error` | `object` | Error details if replacement fails or pattern is invalid |

---

### Success Output Structure

```json
{
  "original": "I'm Meryem, I have 26y",
  "result": "I'm Huda, I have 26y",
  "pattern": "Meryem",
  "replacement": "Huda",
  "mode": "string",
  "matchCount": 1,
  "caseSensitive": true,
  "global": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `original` | `string` | The original unmodified source text |
| `result` | `string` | The text after replacement has been applied |
| `pattern` | `string` | The pattern that was searched for |
| `replacement` | `string` | The replacement string that was used |
| `mode` | `string` | The matching mode used (`string` or `regex`) |
| `matchCount` | `number` | Number of matches found and replaced |
| `caseSensitive` | `boolean` | Whether matching was case-sensitive |
| `global` | `boolean` | Whether all occurrences were replaced |

### Error Response Example

```json
{
  "success": false,
  "error": "Invalid regex pattern",
  "details": "Unclosed group in pattern"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Simple String Replacement

Replace a person's name in a sentence.

**Configuration:**

```text
data: "I'm Meryem, I have 26y"
pattern: "Meryem"
replacement: "Huda"
mode: "string"
global: true
```

**Result:**

```json
{
  "original": "I'm Meryem, I have 26y",
  "result": "I'm Huda, I have 26y",
  "matchCount": 1
}
```

---

### Example 2: Replace All Occurrences

Replace multiple instances of a word.

**Configuration:**

```text
data: "Apple is a fruit. Apple is also a company."
pattern: "Apple"
replacement: "Orange"
mode: "string"
global: true
caseSensitive: true
```

**Result:**

```text
Orange is a fruit. Orange is also a company.
```

---

### Example 3: Regex Pattern Replacement

Standardize phone number formatting.

**Configuration:**

```text
data: "Call me at 555-1234 or 555-5678"
pattern: "(\d{3})-(\d{4})"
replacement: "($1) $2"
mode: "regex"
global: true
```

**Result:**

```text
Call me at (555) 1234 or (555) 5678
```

---

### Example 4: Whitespace Normalization

Remove extra spaces from text.

**Configuration:**

```text
data: "The   quick   brown   fox"
pattern: "\s+"
replacement: " "
mode: "regex"
global: true
```

**Result:**

```text
The quick brown fox
```

---

### Example 5: Redact Sensitive Information

Mask email addresses or personal data.

**Configuration:**

```text
data: "Contact john.doe@example.com or jane.smith@example.com"
pattern: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"
replacement: "[REDACTED]"
mode: "regex"
global: true
```

**Result:**

```text
Contact [REDACTED] or [REDACTED]
```

---

### Example 6: Case-Insensitive Replacement

Replace text regardless of case.

**Configuration:**

```text
data: "The QUICK brown Quick fox"
pattern: "quick"
replacement: "SLOW"
mode: "string"
caseSensitive: false
global: true
```

**Result:**

```text
The SLOW brown SLOW fox
```

<!-- /SECTION: examples -->

---

<!-- SECTION: regex-reference -->
## Regex Patterns Reference

### Common Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| `\d+` | One or more digits | `123` |
| `\w+` | One or more word characters | `word123` |
| `\s+` | One or more whitespace characters | Spaces, tabs, newlines |
| `[A-Z]` | Any uppercase letter | `A`, `B`, `Z` |
| `[a-z]` | Any lowercase letter | `a`, `b`, `z` |
| `[0-9]` | Any digit | `0` through `9` |
| `.` | Any single character | Matches `a`, `1`, `!` |
| `^` | Start of string | `^The` matches at start |
| `$` | End of string | `end$` matches at end |
| `+` | One or more repetitions | `a+` matches `a`, `aa`, `aaa` |
| `*` | Zero or more repetitions | `a*` matches empty, `a`, `aa` |
| `?` | Zero or one repetition | `a?` matches empty or `a` |
| `\|` | OR operator | `cat\|dog` matches `cat` or `dog` |
| `()` | Capture group | `(\d{3})` captures 3 digits |

### Capture Group References

In regex mode, use `$1`, `$2`, etc. in the replacement string to reference capture groups:

```text
Pattern: "(\d{3})-(\d{4})"
Replacement: "($1) $2"
Input: "555-1234"
Output: "(555) 1234"
```

<!-- /SECTION: regex-reference -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Replace text pattern in string
```

### Common Patterns

- **Data Cleaning:** Replace → Extract → Validate
- **Normalization:** Replace (multiple patterns) → Store in Database
- **Redaction:** Replace (sensitive patterns) → Log or Archive
- **Formatting:** Replace (standardization patterns) → Display or Export

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No matches found

**Cause:** The pattern doesn't match any text in the source string.

**Solution:** 
- Verify the pattern spelling and format
- Check the `caseSensitive` setting
- Test the regex pattern with a regex tester if using `regex` mode

#### Regex pattern error

**Cause:** Invalid regular expression syntax.

**Solution:**
- Check for unclosed groups `()` or brackets `[]`
- Escape special characters that should be literal: `\.`, `\+`, `\*`
- Test the regex pattern separately to validate syntax

#### Unexpected replacement results

**Cause:** Capture group references or special characters in replacement string.

**Solution:**
- Verify capture group numbers match the pattern: `(\d{3})` uses `$1`, `(\d{4})` uses `$2`
- Escape literal `$` as `\$` if needed
- Test with simpler patterns first

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Invalid regex pattern` | Malformed regular expression | Check regex syntax |
| `Pattern not found` | No matches in source text | Verify pattern and source |
| `Capture group reference out of range` | `$N` references non-existent group | Adjust replacement references |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Extract](../extract/en.md) — Extract substrings or regex groups
- [Regex Match](../regex-match/en.md) — Test if text matches a pattern
- [Regex Replace](../regex-replace/en.md) — Advanced regex replacement
- [Function](../function/en.md) — Custom text transformation logic

<!-- /SECTION: related -->

---

<!-- SECTION: performance -->
## Performance Notes

- **Simple mode** (`string`) is faster for large texts with many replacements
- **Regex mode** is more powerful but slightly slower for complex patterns
- For very large texts (>1MB), consider breaking the text into chunks and processing separately
- Global replacement (`global: true`) processes all matches; single replacement is slightly faster

<!-- /SECTION: performance -->

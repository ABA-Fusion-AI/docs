---
node_id: "text-analyzer"
title: "Text Analyzer"
description: "Comprehensive text analysis with character/word counts, readability scores, and word frequency"
category: "Utility / Text Processing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- text-analysis
- readability
- flesch-reading-ease
- word-frequency
- word-count
- text-processing
- utility

related_nodes:
- function
- regex-match

---

# Text Analyzer

> **Category:** utility-nodes | **Type:** Action Node

Perform **comprehensive text analysis** — character/word/sentence counts, structural stats, readability scoring, and word-frequency analysis — entirely locally, with no external API calls.

The **Text Analyzer** node exposes two operations: `analyze`, which returns a full statistical and readability breakdown of a text, and `getWordFrequency`, which returns the most frequent words.

### Supported Features

- Character-level breakdown (total, letters, digits, spaces, punctuation)
- Word-level stats (count, unique count, average length, longest word)
- Sentence-level stats (count, average words per sentence, average sentence length)
- Structural stats (line count, paragraph count)
- Estimated reading time and speaking time
- Flesch Reading Ease score with a mapped reading-level label
- Word frequency analysis with a configurable result limit, filtering out short words
- Graceful, non-throwing error responses — all operations return `{ success: false, error }` instead of throwing

### Use Cases

- Evaluate the readability of generated or user-submitted content before publishing
- Estimate reading/speaking time for an article, script, or presentation
- Get quick statistics (word count, sentence count) for content editing workflows
- Identify the most frequent meaningful words in a document (basic keyword extraction)
- Validate that generated content meets length or readability targets

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"analyze"` | Operation to perform: `analyze` or `getWordFrequency`. |
| `text` | `string` | ✅ Yes | `""` | The text to analyze. |
| `limit` | `number` | ❌ No (for `getWordFrequency`) | `10` | Maximum number of top words to return, between 1 and 100. |

---

## Operations

| Operation | Description |
| --------- | ----------- |
| `analyze` | Returns a full breakdown: characters, words, sentences, structure, and readability. |
| `getWordFrequency` | Returns the top `limit` most frequent words (longer than 3 characters), sorted by count descending. |

---

## Analysis Logic

### Text Splitting

- **Words**: `text.trim().split(/\s+/)`, filtered to non-empty strings.
- **Lines**: `text.split("\n")` — no filtering, so a trailing newline produces an extra empty line entry.
- **Sentences**: `text.split(/[.!?]+/)`, filtered to non-empty (after trim) strings.
- **Paragraphs**: `text.split(/\n\n+/)`, filtered to non-empty (after trim) strings.

### Character Counts

- `total`: raw `text.length`.
- `withoutSpaces`: length after stripping all whitespace (`\s`).
- `letters`: count of `[a-zA-Z]` characters only (does not count accented or non-Latin letters).
- `digits`: count of `[0-9]` characters.
- `spaces`: count of literal space characters (splitting on `" "`, not all whitespace types).
- `punctuation`: count of characters that are neither letters, digits, nor whitespace.

### Word Stats

- `unique`: count of distinct lowercased words (case-insensitive deduplication).
- `averageLength`: average word length, formatted to 2 decimal places as a string.
- `longest`: the single longest word found (first one, in case of ties).

### Sentence Stats

- `averageWords`: words divided by sentence count, to 2 decimals.
- `averageLength`: average sentence character length, to 2 decimals.

### Reading/Speaking Time

```text
readingTime = ceil(wordCount / 200)   // minutes, assuming ~200 wpm reading speed
speakingTime = ceil(wordCount / 130)  // minutes, assuming ~130 wpm speaking speed
```

### Syllable Counting

A heuristic syllable counter: words of 3 characters or fewer count as 1 syllable; longer words are stripped of common silent-e/-ed/-es endings and a leading `y`, then vowel-cluster groups (`[aeiouy]{1,2}`) are counted. This is an approximation, not a dictionary-based count, and can be inaccurate for irregular words.

### Flesch Reading Ease

```text
fleschScore = 206.835 - 1.015 * (words / sentences) - 84.6 * (syllables / words)
```

Returns `0` if there are zero sentences or zero words (avoiding division by zero), rather than `NaN`.

### Reading Level Mapping

| Flesch Score Range | Level Label |
| -------------------- | ----------- |
| ≥ 90 | Very Easy (5th grade) |
| 80–89 | Easy (6th grade) |
| 70–79 | Fairly Easy (7th grade) |
| 60–69 | Standard (8th-9th grade) |
| 50–59 | Fairly Difficult (10th-12th grade) |
| 30–49 | Difficult (College) |
| < 30 | Very Difficult (College graduate) |

### Word Frequency Logic

1. Lowercase the text and strip all non-word, non-whitespace characters (`[^\w\s]`).
2. Split on whitespace.
3. **Filter out words of 3 characters or fewer** (excludes common short words like "the", "and", "is").
4. Count occurrences, sort descending by count, and return the top `limit` entries.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

Every operation returns a `success` boolean. On failure, `success: false` is returned with an `error` string — **the node never throws**; errors are always returned as normal output data.

#### `analyze`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Whether the analysis succeeded. |
| `characters.total` | `number` | Total character count. |
| `characters.withoutSpaces` | `number` | Character count excluding whitespace. |
| `characters.letters` | `number` | Count of `[a-zA-Z]` characters. |
| `characters.digits` | `number` | Count of digit characters. |
| `characters.spaces` | `number` | Count of literal space characters. |
| `characters.punctuation` | `number` | Count of non-alphanumeric, non-whitespace characters. |
| `words.total` | `number` | Total word count. |
| `words.unique` | `number` | Count of distinct (case-insensitive) words. |
| `words.averageLength` | `string` | Average word length, 2 decimal places. |
| `words.longest` | `string` | The longest word found. |
| `sentences.total` | `number` | Total sentence count. |
| `sentences.averageWords` | `string` | Average words per sentence, 2 decimal places. |
| `sentences.averageLength` | `string` | Average sentence character length, 2 decimal places. |
| `structure.lines` | `number` | Number of lines. |
| `structure.paragraphs` | `number` | Number of paragraphs. |
| `readability.readingTime` | `string` | Estimated reading time, e.g. `"3 min"`. |
| `readability.speakingTime` | `string` | Estimated speaking time, e.g. `"5 min"`. |
| `readability.fleschScore` | `string` | Flesch Reading Ease score, 1 decimal place. |
| `readability.level` | `string` | Mapped reading-level label. |

#### `getWordFrequency`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Whether the operation succeeded. |
| `frequency` | `array` | List of `{ word, count }` objects, sorted by count descending, limited to `limit` entries. |

---

## Output Example

### `analyze`

```json
{
  "success": true,
  "characters": {
    "total": 142,
    "withoutSpaces": 118,
    "letters": 112,
    "digits": 0,
    "spaces": 24,
    "punctuation": 6
  },
  "words": {
    "total": 24,
    "unique": 22,
    "averageLength": "4.67",
    "longest": "comprehensive"
  },
  "sentences": {
    "total": 2,
    "averageWords": "12.00",
    "averageLength": "71.00"
  },
  "structure": {
    "lines": 1,
    "paragraphs": 1
  },
  "readability": {
    "readingTime": "1 min",
    "speakingTime": "1 min",
    "fleschScore": "58.3",
    "level": "Fairly Difficult (10th-12th grade)"
  }
}
```

### `getWordFrequency`

```json
{
  "success": true,
  "frequency": [
    { "word": "workflow", "count": 5 },
    { "word": "analysis", "count": 3 },
    { "word": "readability", "count": 2 }
  ]
}
```

---

## Configuration Examples

### Full Analysis

```json
{
  "operation": "analyze",
  "text": "This is a sample paragraph. It has two sentences for testing purposes."
}
```

### Word Frequency (Top 5)

```json
{
  "operation": "getWordFrequency",
  "text": "The workflow processes text data. Text analysis helps understand text patterns in workflow data.",
  "limit": 5
}
```

---

## Workflow Integration

### Sample Workflow: LLM → Text Analyzer → If (readability gate)

```json
{
  "nodes": [
    {
      "id": "generate-content",
      "type": "llm"
    },
    {
      "id": "analyze-content",
      "type": "text-analyzer",
      "config": {
        "operation": "analyze"
      }
    },
    {
      "id": "check-readability",
      "type": "if"
    }
  ]
}
```

### Sample Workflow: Text Analyzer → Function (display stats)

```json
{
  "nodes": [
    {
      "id": "analyze-article",
      "type": "text-analyzer",
      "config": {
        "operation": "analyze",
        "text": "Article body text goes here..."
      }
    },
    {
      "id": "format-stats",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Word Frequency → Function (keyword tags)

```json
{
  "nodes": [
    {
      "id": "extract-keywords",
      "type": "text-analyzer",
      "config": {
        "operation": "getWordFrequency",
        "limit": 10
      }
    },
    {
      "id": "build-tags",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- LLM (generate content) → Text Analyzer (`analyze`) → If (Flesch score threshold) → LLM (rewrite for simpler reading level) — readability gate
- Text Analyzer (`getWordFrequency`) → Function → Database — basic keyword tagging pipeline
- Function (assemble document text) → Text Analyzer (`analyze`) → Notification — content stats reporting

---

## Error Handling

The node **does not throw errors** — all failures are caught and returned as normal output with `success: false`.

### Missing Text

```json
{ "success": false, "error": "Text is required" }
```

Raised for both `analyze` and `getWordFrequency` when `text` is empty.

### Invalid Operation

```json
{ "success": false, "error": "Invalid operation" }
```

Should not normally occur given the `operation` enum, but is returned defensively if an unrecognized value slips through.

### Internal Errors

Any unexpected error thrown internally during analysis is caught and returned as:

```json
{ "success": false, "error": "<message>" }
```

---

## Troubleshooting

### `success: false, error: "Text is required"`

**Cause**

`text` was left empty (falsy) in the node configuration.

**Solution**

Provide non-empty text content.

---

### `fleschScore` is `"0.0"` Despite Non-Empty Text

**Cause**

The text produced zero detected sentences (e.g. no `.`, `!`, or `?` characters) or zero words, which the formula defensively short-circuits to `0` rather than computing (and risking division by zero).

**Solution**

Ensure the text contains standard sentence-ending punctuation if a meaningful Flesch score is needed; text without any terminal punctuation will always score `0`.

---

### `structure.lines` Seems One Higher Than Expected

**Cause**

`text.split("\n")` produces an extra trailing empty-string entry if the text ends with a newline character — this is not filtered out (unlike `sentences` and `paragraphs`, which are).

**Solution**

Trim trailing newlines from the input text if an exact visual line count is required.

---

### `getWordFrequency` Omits Common Short Words

**Cause**

Words of 3 characters or fewer (e.g. "the", "and", "for", "is") are filtered out by design, to focus on more meaningful/content-bearing words.

**Solution**

This is expected behavior — there is no configuration option to include short words; use [Regex Match](./regex-match.md) directly on the text if a raw, unfiltered word count is needed.

---

### Syllable Count / Flesch Score Looks Off for Certain Words

**Cause**

Syllable counting uses a regex-based heuristic (vowel-cluster counting with common suffix stripping), not a dictionary — it can misestimate syllables for irregular English words, proper nouns, or non-English text.

**Solution**

Treat the Flesch score as an approximation suitable for general English prose, not an exact linguistic measurement.

---

### `letters` Count Excludes Accented or Non-Latin Characters

**Cause**

The `letters` count only matches `[a-zA-Z]` — accented characters (é, ñ, ü) and non-Latin scripts are not counted as letters, and instead fall into the `punctuation` bucket (since they're neither letters, digits, nor whitespace by this regex's definition).

**Solution**

Be aware this node is tuned for plain English text; results on non-English or accented text may be misleading, particularly for the `letters`/`punctuation` split.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally using JavaScript string and regex operations.

---

## Notes

The node is designed to **never throw** — every operation, including internal errors, is caught and returned as a normal `{ success: false, error }` output object, so downstream nodes should check `success` rather than relying on workflow-level error handling.

The node does not:

- Support languages other than English for readability scoring (Flesch Reading Ease is an English-language formula) or syllable counting
- Use a dictionary-based syllable counter (heuristic/regex-based only)
- Filter or configure which words are excluded from `getWordFrequency` beyond the fixed 3-character-or-fewer rule
- Count accented or non-Latin characters as letters
- Support other readability formulas (e.g. Flesch-Kincaid Grade Level, Gunning Fog)

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
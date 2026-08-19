---
node_id: "language-tool"
title: "LanguageTool"
description: "Check grammar and spelling using LanguageTool API. Better used as POST for long text."
category: "text-processing-nlp"
subcategory: "grammar-spelling"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - grammar
  - spelling
  - proofreading
  - languagetool
  - text-analysis
  - spellcheck
  - nlp
  - multi-language
related_nodes:
  - google-translate-action
  - free-dictionary
  - html-to-text
  - normalize-whitespace
  - log
---

<!-- SECTION: header -->
# LanguageTool

> **Category:** Text Processing & NLP | **Subcategory:** Grammar & Spelling | **Type:** Action Node

Proofread text for grammatical errors, spelling mistakes, and style improvements across over 30 languages using the LanguageTool API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **LanguageTool** node integrates the LanguageTool REST API (`https://api.languagetool.org/v2/check`) to analyze written text for grammar, punctuation, style, and spelling issues.

It evaluates input text against language-specific rule sets, returning detailed error annotations, contextual snippets, error classifications, and suggested corrections. The node uses an HTTP `POST` request payload to support both short strings and long, multi-paragraph documents without URL character limit restrictions.

### Key Features

- **Multi-Language Proofreading:** Supports over 30 languages including English (`en`), French (`fr`), German (`de`), Spanish (`es`), and automatic language detection (`auto`).
- **Comprehensive Error Breakdown:** Provides exact character offsets, length, offending sentence, context snippet, rule description, and replacement suggestions for every detected issue.
- **Custom Rule Filtering:** Enable specific checking rules (`enabledRules`) or disable unwanted rules (`disabledRules`) via comma-separated rule lists.
- **Flexible Data Sourcing:** Uses configured `text` parameter or automatically falls back to incoming payload data from upstream nodes.
- **Robust POST API Handling:** Transmits text via `application/x-www-form-urlencoded` HTTP POST for reliable processing of long articles, emails, or blog posts.

### Processing Flow

```text
Input Text (Config Parameter or Input Port Payload)
                       ↓
  Is Text Present? ── No ──→ Throw Error ("Text is required")
                       ↓ Yes
Build URLSearchParams (text, language, enabledRules, disabledRules)
                       ↓
HTTP POST Request to https://api.languagetool.org/v2/check
                       ↓
         Response Status OK?
  ├── No  ──→ Throw Error ("LanguageTool API error: STATUS")
  └── Yes ──→ Parse API JSON Response
                       ↓
Map Matches Array (message, shortMessage, replacements, offset, length, context, sentence, rule)
                       ↓
Return Structured Payload (success: true, language, software, warnings, matches, total_matches)
```

### Use Cases

- **Content Publishing & CMS Automation:** Automatically proofread blog posts, newsletter drafts, or documentation prior to publishing.
- **Customer Support QA:** Check outgoing agent email replies or chat messages for grammatical correctness and tone.
- **User-Generated Content Moderation:** Flag poor spelling or low-quality text submissions in community forums or review systems.
- **Automated Translation Verification:** Validate translated text strings to ensure grammatical accuracy before committing to localization files.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `text` | `string` | ❌ No* | `""` | The text content to analyze. If left empty, text is automatically extracted from the incoming workflow payload. |
| `language` | `string` | ❌ No | `"en"` | Language code of the text (e.g. `"en"`, `"fr"`, `"de"`, `"es"`, or `"auto"` for automatic detection). |
| `enabledRules` | `string` | ❌ No | `""` | Comma-separated list of rule IDs to explicitly enable (e.g., `"UPPERCASE_SENTENCE_START"`). |
| `disabledRules` | `string` | ❌ No | `""` | Comma-separated list of rule IDs to disable (e.g., `"WHITESPACE_RULE,PUNCTUATION_PARAGRAPH_END"`). |

*\* Note: Either `text` must be provided in the node configuration or non-empty text data must be passed into the input port.*

---

### Parameter Details

#### `text`
The target text to be proofread.
- **Type:** `string`
- **Required:** Conditional (must exist in parameter or input payload)
- **Example:** `"Bonjour tout le monde, comment allez-vous?"`
- **Dynamic Expression:** Can reference upstream node outputs using expressions like `{{outputs.Function.success}}`.

---

#### `language`
The ISO language code for grammar and spellchecking.
- **Type:** `string`
- **Default:** `"en"`
- **Supported Values:**
  - `"en"` — English (General/US/UK)
  - `"fr"` — French
  - `"de"` — German
  - `"es"` — Spanish
  - `"auto"` — Automatic language detection
  - *Full list of ISO 639-1 language codes supported by LanguageTool.*

---

#### `enabledRules` & `disabledRules`
Fine-tune proofreading behavior by overriding default rule sets.
- **Type:** `string` (comma-separated rule IDs)
- **Default:** `""`
- **Example `disabledRules`:** `"WHITESPACE_RULE,MORFOLOGIK_RULE_EN_US"`
- **Example `enabledRules`:** `"COMP_THAN,UPPERCASE_SENTENCE_START"`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload. Can supply text directly if the `text` parameter is left empty. |

---

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the API call succeeds. Returns detected language, matches array, replacement suggestions, and total match count. |
| `error` | `Error` | Emitted if text is missing or the LanguageTool HTTP request fails. |

---

### Output Data Structure Example

```json
{
  "success": true,
  "language": {
    "name": "English (US)",
    "code": "en-US",
    "detectedLanguage": {
      "name": "English",
      "code": "en",
      "confidence": 0.99
    }
  },
  "software": {
    "name": "LanguageTool",
    "version": "6.4",
    "buildDate": "2024-01-01"
  },
  "warnings": {
    "incompleteResults": false
  },
  "matches": [
    {
      "message": "Did you mean 'have'?",
      "shortMessage": "Grammar error",
      "replacements": [
        { "value": "have" }
      ],
      "offset": 2,
      "length": 3,
      "context": {
        "text": "I has a dream.",
        "offset": 2,
        "length": 3
      },
      "sentence": "I has a dream.",
      "type": {
        "typeName": "Other"
      },
      "rule": {
        "id": "PERS_PRON_VERB_AGREEMENT",
        "description": "Personal pronoun + verb agreement (e.g. 'I has' -> 'I have')",
        "issueType": "grammar"
      }
    }
  ],
  "total_matches": 1
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful API processing (`true`). |
| `language` | `object` | Information about the language used and confidence of detection. |
| `matches` | `array` | Array of grammar, spelling, and style rule matches found in the text. |
| `matches[].message` | `string` | Descriptive explanation of the detected issue. |
| `matches[].shortMessage` | `string` | Short summary header of the issue (e.g. `"Spelling mistake"`). |
| `matches[].replacements` | `array` | Suggested correction objects containing `{ "value": "replacement_word" }`. |
| `matches[].offset` | `number` | Zero-based starting character index of the error in the input text. |
| `matches[].length` | `number` | Character length of the offending word or phrase. |
| `matches[].context` | `object` | Text snippet around the error highlighting the exact offset and length. |
| `matches[].sentence` | `string` | Complete sentence containing the error. |
| `matches[].rule` | `object` | Rule metadata including rule `id`, `description`, and `issueType`. |
| `total_matches` | `number` | Total number of grammar/spelling issues detected. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: French Grammar Check

Proofread a French sentence for proper spelling and punctuation.

**Configuration:**

```text
Text: Bonjour tout le monde, comment allez-vous?
Language: fr
```

**Output Summary:**

```json
{
  "success": true,
  "language": { "name": "French", "code": "fr" },
  "matches": [],
  "total_matches": 0
}
```

---

### Example 2: Automatic Language Detection & Correction

Analyze text using automatic language detection (`language: "auto"`).

**Input Data (from Function Node):**

```text
"I has a dream."
```

**Configuration:**

```text
Language: auto
```

**Result Output:**
- Detected Language: `English`
- Error Found: `"I has a dream."`
- Suggested Replacement: `"have"`
- Total Matches: `1`

---

### Example 3: Disabling Specific Rules

Disable whitespace checking rules while proofreading technical text.

**Configuration:**

```text
Text: This is a sample text  with extra spaces.
Language: en
Disabled Rules: WHITESPACE_RULE
```

**Result Output:**
- Ignores extra space warnings and returns only critical spelling or grammar errors.

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: LanguageTool Proofreading Example Workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow execution.
2. **Function Node (Optional):** Generates or fetches draft text for proofreading.
3. **LanguageTool Node:** Sends text to the LanguageTool API with target or auto-detected language settings.
4. **Log Node:** Prints detected grammar errors, suggested replacements, and total match count to the execution log.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Use `POST` for Document Proofreading:** The node automatically uses HTTP `POST` requests, ensuring that long documents or multi-page articles can be checked without hitting URL length limits.
2. **Auto Detection Confidence:** When using `language: "auto"`, verify short phrases manually as language detection confidence increases with text length.
3. **Filter Noise with `disabledRules`:** If your text contains custom jargon, code snippets, or brand names, add corresponding rule IDs to `disabledRules` to prevent false positive spelling alerts.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors

#### `Text is required`
- **Cause:** No text was configured in the `text` parameter, and no valid text payload was received from the upstream node.
- **Solution:** Provide a text string in the node configuration or connect an upstream node that emits text.

#### `LanguageTool API error: 400`
- **Cause:** Unsupported language code or malformed parameter values (such as invalid rule IDs).
- **Solution:** Verify that `language` contains a valid ISO language code (e.g. `"en"`, `"fr"`, `"de"`, `"auto"`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Google Translate](../google-translate-action/en.md) — Translate text between languages
- [Free Dictionary](../free-dictionary/en.md) — Lookup word definitions, phonetics, and synonyms
- [Normalize Whitespace](../normalize-whitespace/en.md) — Clean up extra spaces and formatting characters
- [Log](../log/en.md) — Print proofreading results to the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial release of the LanguageTool node supporting grammar checking, spellchecking, rule customization, and multi-language support |

<!-- /SECTION: changelog -->

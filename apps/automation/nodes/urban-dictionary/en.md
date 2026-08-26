---
node_id: "urban-dictionary"
title: "Urban Dictionary"
description: "Look up slang definitions, examples, and meanings from Urban Dictionary."
category: "Web Search & Information"
subcategory: "Language"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - urban-dictionary
  - slang
  - definitions
  - language
  - vocabulary
  - search
  - api
related_nodes:
  - free-dictionary
  - oxford-dictionaries
  - google-translate-action
  - log
---

<!-- SECTION: header -->
# Urban Dictionary

> **Category:** Web Search & Information | **Subcategory:** Language | **Type:** Action Node

Look up user-submitted slang definitions, usage examples, tags, and vote information for a term using Urban Dictionary.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Urban Dictionary** node searches Urban Dictionary for a word or phrase and returns matching community-submitted definitions. It is useful for informal language research, content enrichment, and workflows that need to recognize modern slang or internet terminology.

### Key Features

- **Term Lookup:** Search for a word, phrase, abbreviation, or slang term
- **Multiple Definitions:** Return the available definitions for a term
- **Usage Examples:** Include example sentences when provided
- **Community Signals:** Return author, vote, permalink, and publication metadata when available
- **Dynamic Input:** Use a configured `term` or pass a term through the `input` port
- **No Credentials Required:** The workflow example contains no API key, token, or secret

### Use Cases

- Explain slang in user-generated content
- Enrich moderation or content-review workflows with context
- Build vocabulary and language-learning tools
- Detect and classify informal or internet terminology
- Add definitions to chat, support, or research workflows

> Urban Dictionary definitions are user submitted and may be offensive, inaccurate, outdated, or unsuitable for all audiences. Review content before displaying or acting on it.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `term` | `string` | Yes | — | Word or phrase to look up, for example `ghosting` |

If `term` is not configured, the node can read a direct string from the incoming `input` value. An incoming object can provide a `term` property.

### API Behavior

The node performs a term lookup against the Urban Dictionary API using the requested term:

```text
GET https://api.urbandictionary.com/v0/define?term={term}
```

The provided workflow configures only:

```json
{
  "term": "ghosting"
}
```

No API key, access token, authorization header, or credential reference is present in the workflow example.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | A term string or an object containing a `term` field. Used when the `term` parameter is empty. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Urban Dictionary response containing matching definitions and related metadata |

Typical response shape:

```json
{
  "list": [
    {
      "definition": "The act of avoiding or ignoring someone...",
      "example": "They stopped replying and started ghosting me.",
      "author": "community-user",
      "word": "ghosting",
      "thumbs_up": 120,
      "thumbs_down": 8,
      "permalink": "https://www.urbandictionary.com/define.php?term=ghosting"
    }
  ],
  "result_type": "exact"
}
```

The number and order of definitions, as well as optional metadata, can change over time.

### Error Output

Missing terms, request failures, malformed responses, or no-result conditions are routed to `error`.

```json
{
  "success": false,
  "error": "No definitions found",
  "term": "unknown-term"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Lookup

```json
{
  "term": "ghosting"
}
```

The node returns matching definitions and usage examples for `ghosting`.

### Dynamic Term from a Previous Node

Pass a term directly through `input`:

```text
ghosting
```

Or pass a named object:

```json
{
  "term": "doomscrolling"
}
```

### Select the Top-Rated Definition

Use a Function node after the lookup to select the definition with the highest `thumbs_up` value:

```js
const definitions = input.list || [];
return definitions.reduce((best, item) =>
  !best || item.thumbs_up > best.thumbs_up ? item : best,
  null
);
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Look up a slang term and inspect its definitions
```

### Common Patterns

- **Basic lookup:** Manual Trigger → Urban Dictionary → Log
- **Content enrichment:** Text input → Urban Dictionary → Function
- **Moderation context:** Webhook → Urban Dictionary → Conditional review
- **Definition selection:** Urban Dictionary → Function → Response

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Term is required

**Cause:** Neither `term` nor a usable incoming `input` value was provided.

**Solution:** Configure a word or phrase in `term`, pass a string to `input`, or pass an object with a `term` property.

### No definitions found

**Cause:** The term is not defined, is misspelled, or the service returned an empty result list.

**Solution:** Check the spelling, try a shorter phrase, or inspect the returned `list` before treating the lookup as successful.

### Unexpected or incomplete fields

**Cause:** Urban Dictionary content is community submitted and fields may vary between entries.

**Solution:** Treat `definition`, `example`, vote counts, author, and permalink as optional values in downstream mappings.

### Request failed

**Cause:** The upstream service is unavailable, rate-limited, or inaccessible from the workflow runtime.

**Solution:** Inspect the `error` output, retry with appropriate backoff, and verify network access.

### Inappropriate content

**Cause:** Urban Dictionary may contain profanity, sexual content, hate speech, or other user-generated material.

**Solution:** Apply filtering, moderation, and access controls before presenting definitions to end users or storing them in business systems.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Free Dictionary](./free-dictionary.md) — Retrieve standard dictionary definitions
- [Oxford Dictionaries](./oxford-dictionaries.md) — Query dictionary definitions and word metadata
- [Google Translate](./google-translate-action.md) — Translate terms between languages
- [Log](./log.md) — Inspect lookup results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial documentation |

<!-- /SECTION: changelog -->

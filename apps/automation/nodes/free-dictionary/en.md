---
node_id: "free-dictionary"
title: "Free Dictionary"
description: "Get word definitions, meanings, synonyms, antonyms, phonetics, and pronunciation audio from the Free Dictionary API."
category: "Dictionary"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:

- dictionary
- free-dictionary
- definitions
- meanings
- synonyms
- antonyms
- phonetics
- pronunciation
- language
- api
- action

related_nodes:
- function
- if
- http-request

---

# Free Dictionary

> **Category:** dictionary-nodes | **Type:** Action Node

Get word definitions, meanings, synonyms, antonyms, phonetic information, and pronunciation audio from the **Free Dictionary API**.

The **Free Dictionary** node queries the Free Dictionary API using a word and language code.

The node returns structured dictionary data including definitions, parts of speech, synonyms, antonyms, phonetic spellings, pronunciation audio URLs, licenses, and source URLs.

The node can receive the word from the node configuration or from workflow input.

### Supported Features

- Look up word definitions
- Specify the dictionary language
- Extract meanings
- Extract parts of speech
- Extract definitions
- Extract synonyms
- Extract antonyms
- Extract examples
- Extract phonetic spellings
- Extract pronunciation audio URLs
- Extract phonetic source URLs
- Extract license information
- Extract dictionary source URLs
- Accept the word from workflow input
- Return normalized structured JSON data
- Handle missing words
- Handle API errors

### Use Cases

- Build dictionary workflows
- Get definitions of words
- Build language-learning applications
- Retrieve synonyms and antonyms
- Display pronunciation information
- Provide pronunciation audio
- Build vocabulary applications
- Create educational workflows
- Check word meanings automatically
- Transform definitions using a `Function` node
- Filter dictionary results using an `If` node

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| ---------- | ---- | -------- | ------- | ----------- |
| `word` | `string` | ✅ Yes | — | Word to look up in the dictionary. |
| `language` | `string` | ❌ No | `"en"` | Language code used by the Free Dictionary API. |

### Configuration Schema

```typescript
const schema: SchemaTypeAny = v.object({
  word: v.string(),
  language: v.string().optional().default("en"),
});
```

The `word` parameter is required by the node schema.

The `language` parameter is optional and defaults to:

```text
en
```

---

## Free Dictionary API

The node uses the following API endpoint:

```text
https://api.dictionaryapi.dev/api/v2/entries/{language}/{word}
```

The word is URL-encoded before being added to the request URL.

For example:

```text
word = hello
language = en
```

produces:

```text
https://api.dictionaryapi.dev/api/v2/entries/en/hello
```

The node sends the following HTTP header:

```text
Content-Type: application/json
```

---

## Inputs & Outputs

### Inputs

The node can receive the word from configuration or workflow input.

The input resolution follows this logic:

1. Use the configured `word` when provided.
2. Otherwise, if workflow input is a string, use the input as the word.
3. Otherwise, convert the workflow input to a string.

For example, a previous node can output:

```text
hello
```

The Free Dictionary node will use:

```text
hello
```

as the lookup word.

The language is always taken from the node configuration and defaults to:

```text
en
```

### Outputs

The node returns a structured dictionary result.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Indicates whether the dictionary lookup was successful. |
| `word` | `string` | Word that was searched. |
| `language` | `string` | Language used for the dictionary lookup. |
| `results` | `array` | Dictionary entries returned by the API. |
| `total_results` | `number` | Number of dictionary entries returned. |
| `note` | `string` | Information about where definitions and pronunciation data can be found. |

### Result Fields

Each dictionary result may contain:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `word` | `string` | Word returned by the dictionary API. |
| `phonetic` | `string \| null` | Primary phonetic representation. |
| `phonetics` | `array` | Available phonetic and pronunciation information. |
| `meanings` | `array` | Word meanings grouped by part of speech. |
| `license` | `object \| null` | License information for the dictionary entry. |
| `source_urls` | `string[]` | Source URLs associated with the entry. |

### Phonetic Fields

Each phonetic entry may contain:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `text` | `string \| null` | Phonetic representation. |
| `audio` | `string \| null` | URL to pronunciation audio. |
| `source_url` | `string \| null` | Source URL for the phonetic information. |
| `license` | `object \| null` | License information for the phonetic data. |

### Phonetic License Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `name` | `string` | Name of the license. |
| `url` | `string` | URL of the license. |

### Meaning Fields

Each meaning contains:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `part_of_speech` | `string` | Grammatical category of the meaning. |
| `definitions` | `array` | Definitions associated with the meaning. |
| `synonyms` | `string[]` | Synonyms associated with the meaning. |
| `antonyms` | `string[]` | Antonyms associated with the meaning. |

### Definition Fields

Each definition contains:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `definition` | `string` | Definition text. |
| `synonyms` | `string[]` | Synonyms associated with the definition. |
| `antonyms` | `string[]` | Antonyms associated with the definition. |
| `example` | `string \| null` | Example sentence when available. |

---

## Output Example

```json
{
  "success": true,
  "word": "hello",
  "language": "en",
  "results": [
    {
      "word": "hello",
      "phonetic": "həˈləʊ",
      "phonetics": [
        {
          "text": "həˈləʊ",
          "audio": "https://api.dictionaryapi.dev/media/pronunciations/en/hello-au.mp3",
          "source_url": "https://commons.wikimedia.org/wiki/File:en-hello.ogg",
          "license": {
            "name": "BY-SA 3.0",
            "url": "https://creativecommons.org/licenses/by-sa/3.0/"
          }
        }
      ],
      "meanings": [
        {
          "part_of_speech": "noun",
          "definitions": [
            {
              "definition": "A greeting used when answering the telephone.",
              "synonyms": [],
              "antonyms": [],
              "example": null
            }
          ],
          "synonyms": [],
          "antonyms": []
        }
      ],
      "license": {
        "name": "CC BY-SA 3.0",
        "url": "https://creativecommons.org/licenses/by-sa/3.0/"
      },
      "source_urls": [
        "https://en.wiktionary.org/wiki/hello"
      ]
    }
  ],
  "total_results": 1,
  "note": "Use the 'meanings' field for definitions and 'phonetics.audio' for pronunciation audio"
}
```

The exact definitions, meanings, phonetics, audio URLs, licenses, and source URLs depend on the data returned by the Free Dictionary API.

---

## Configuration Examples

### Default Language

The language defaults to English.

```json
{
  "word": "hello"
}
```

Equivalent configuration:

```json
{
  "word": "hello",
  "language": "en"
}
```

### Search a Word in English

```json
{
  "word": "computer",
  "language": "en"
}
```

### Search a Word in Another Supported Language

```json
{
  "word": "bonjour",
  "language": "fr"
}
```

The language code is inserted directly into the API URL.

### Workflow Input

The word can be provided by a previous node.

Configuration:

```json
{
  "word": "",
  "language": "en"
}
```

If the previous node outputs:

```text
beautiful
```

the node searches for:

```text
beautiful
```

---

## Workflow Integration

### Sample Workflow: Dictionary Lookup

```json
{
  "nodes": [
    {
      "id": "free-dictionary",
      "type": "free-dictionary",
      "config": {
        "word": "hello",
        "language": "en"
      }
    }
  ]
}
```

### Sample Workflow: Input → Free Dictionary

```json
{
  "nodes": [
    {
      "id": "input",
      "type": "input"
    },
    {
      "id": "free-dictionary",
      "type": "free-dictionary",
      "config": {
        "word": "",
        "language": "en"
      }
    }
  ]
}
```

If the input node produces:

```text
beautiful
```

the Free Dictionary node performs a lookup for:

```text
beautiful
```

### Sample Workflow: Free Dictionary → Function

```json
{
  "nodes": [
    {
      "id": "free-dictionary",
      "type": "free-dictionary",
      "config": {
        "word": "important",
        "language": "en"
      }
    },
    {
      "id": "process-definition",
      "type": "function"
    }
  ]
}
```

The `Function` node can transform the dictionary response into a custom format.

### Sample Workflow: Free Dictionary → If

```json
{
  "nodes": [
    {
      "id": "free-dictionary",
      "type": "free-dictionary",
      "config": {
        "word": "example",
        "language": "en"
      }
    },
    {
      "id": "check-result",
      "type": "if"
    }
  ]
}
```

The `If` node can be used to route the workflow based on dictionary results.

### Common Patterns

- Input → Free Dictionary → Function
- Free Dictionary → Function → Database
- Free Dictionary → Function → Notification
- Free Dictionary → If → Conditional Workflow
- Free Dictionary → Function → Vocabulary Processing
- Free Dictionary → Function → Language Learning
- Free Dictionary → HTTP Request → Additional Processing

---

## Lookup Process

The node performs the following operations:

1. Reads the configured `word` and `language`.
2. Checks that a word is available.
3. Uses workflow input when appropriate.
4. URL-encodes the word.
5. Builds the Free Dictionary API URL.
6. Sends an HTTP GET request.
7. Checks the HTTP response status.
8. Handles a `404` response separately.
9. Parses the JSON response.
10. Normalizes dictionary entries.
11. Extracts phonetic information.
12. Extracts meanings and definitions.
13. Extracts synonyms and antonyms.
14. Extracts examples.
15. Extracts license information.
16. Extracts source URLs.
17. Returns the structured dictionary result.

---

## Word Lookup

The `word` parameter identifies the word to search.

Example:

```json
{
  "word": "hello"
}
```

The node creates the following API path:

```text
/api/v2/entries/en/hello
```

The word is encoded using `encodeURIComponent()` before being inserted into the URL.

This allows words containing spaces or special characters to be safely included in the request.

---

## Language

The `language` parameter controls the language used by the Free Dictionary API.

The default value is:

```text
en
```

Example:

```json
{
  "word": "bonjour",
  "language": "fr"
}
```

The resulting API endpoint is:

```text
https://api.dictionaryapi.dev/api/v2/entries/fr/bonjour
```

The language value is not transformed by the node before being added to the URL.

---

## Meanings and Definitions

Definitions are returned inside the `meanings` array.

For example:

```json
{
  "meanings": [
    {
      "part_of_speech": "noun",
      "definitions": [
        {
          "definition": "Example definition.",
          "synonyms": [],
          "antonyms": [],
          "example": "Example sentence."
        }
      ],
      "synonyms": [],
      "antonyms": []
    }
  ]
}
```

A word can have multiple meanings and multiple definitions.

The `part_of_speech` field identifies the grammatical category associated with each meaning.

---

## Synonyms and Antonyms

Synonyms and antonyms can be available at both the meaning and definition levels.

Meaning-level example:

```json
{
  "synonyms": [
    "example",
    "sample"
  ],
  "antonyms": []
}
```

Definition-level example:

```json
{
  "definition": "Example definition.",
  "synonyms": [
    "sample"
  ],
  "antonyms": [
    "opposite"
  ],
  "example": null
}
```

If no synonyms or antonyms are returned by the API, the node returns an empty array.

---

## Phonetics

The node extracts phonetic information from the API response.

Example:

```json
{
  "phonetic": "həˈləʊ",
  "phonetics": [
    {
      "text": "həˈləʊ",
      "audio": "https://example.com/audio.mp3",
      "source_url": "https://example.com/source",
      "license": {
        "name": "License",
        "url": "https://example.com/license"
      }
    }
  ]
}
```

The `audio` field can contain a pronunciation audio URL when available.

The node does not download or process the audio file. It only returns the URL provided by the API.

---

## License Information

License information can be returned for dictionary entries and phonetic entries.

Example:

```json
{
  "license": {
    "name": "CC BY-SA 3.0",
    "url": "https://creativecommons.org/licenses/by-sa/3.0/"
  }
}
```

If the API does not provide license information, the corresponding field is returned as `null`.

---

## Source URLs

The node preserves source URLs returned by the API.

Example:

```json
{
  "source_urls": [
    "https://en.wiktionary.org/wiki/hello"
  ]
}
```

These URLs identify the source information associated with the dictionary entry.

---

## Error Handling

If the lookup fails, the node throws an error.

Errors are wrapped by `handleTick()` using the following format:

```text
Free Dictionary lookup failed: <error message>
```

### Word Required

**Cause**

No word is available for the lookup.

**Error**

```text
Word is required
```

**Solution**

Provide a `word` in the node configuration or provide a string as workflow input.

---

### Word Not Found

**Cause**

The Free Dictionary API returned HTTP status `404`.

**Error**

```text
Word "<word>" not found in dictionary
```

For example:

```text
Word "exampleword123" not found in dictionary
```

**Solution**

Verify the spelling of the word or try another language.

---

### API Error

**Cause**

The Free Dictionary API returned a non-success HTTP status other than `404`.

**Error**

```text
Free Dictionary API error: <status>
```

For example:

```text
Free Dictionary API error: 500
```

**Solution**

Check the API availability and verify the configured language and word.

---

## Troubleshooting

### No Definition Returned

**Cause**

The API may not have dictionary data for the requested word or language.

**Solution**

Verify the word spelling and language code.

---

### Word Not Found

**Cause**

The API returned `404`.

**Solution**

Check that the word exists in the selected language and try another spelling or language.

---

### Invalid Language

**Cause**

The configured language code may not be supported by the API.

**Solution**

Use a valid language code supported by the Free Dictionary API.

---

### Pronunciation Audio Missing

**Cause**

Not every dictionary entry contains pronunciation audio.

**Solution**

Check the `phonetics` array and use the `audio` field when a URL is available.

---

### No Synonyms or Antonyms

**Cause**

The API may not provide synonyms or antonyms for a particular meaning.

**Solution**

Check both the meaning-level and definition-level `synonyms` and `antonyms` arrays.

Empty arrays are returned when no values are available.

---

## Output Data Structure

The complete normalized output has the following structure:

```text
success
word
language
results[]
    word
    phonetic
    phonetics[]
        text
        audio
        source_url
        license
            name
            url
    meanings[]
        part_of_speech
        definitions[]
            definition
            synonyms[]
            antonyms[]
            example
        synonyms[]
        antonyms[]
    license
        name
        url
    source_urls[]
total_results
note
```

---

## Notes

The node normalizes the API response instead of returning the original API response directly.

Missing string values are generally converted to:

```text
null
```

or:

```text
""
```

depending on the field.

Missing arrays are returned as empty arrays.

The `total_results` field contains the number of dictionary entries returned by the API.

The response also contains the following note:

```text
Use the 'meanings' field for definitions and 'phonetics.audio' for pronunciation audio
```

---

## Related

- [Function](./function.md) – Transform and process dictionary results
- [If](./if.md) – Filter and route dictionary results
- [HTTP Request](./http-request.md) – Make requests to external APIs

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-11 | Initial release |

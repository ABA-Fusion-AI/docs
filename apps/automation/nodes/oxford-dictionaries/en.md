---
node_id: "oxford-dictionaries"
title: "Oxford Dictionaries"
description: "Access Oxford Dictionaries API for definitions, translations, and corpus example sentences."
category: "Language"
subcategory: "dictionary-lexical"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - oxford
  - dictionary
  - definitions
  - translations
  - examples
  - pronunciation
  - linguistics
  - language
  - etymology
related_nodes:
  - datamuse
  - openthesaurus
  - language-tool
  - libre-translate
  - deepl
  - lingvanex
  - function
---

<!-- SECTION: header -->
# Oxford Dictionaries

> **Category:** Language | **Type:** Action Node

Access the authoritative **Oxford Dictionaries API (v2)** to retrieve word definitions, phonetic pronunciations, audio files, bilingual translations, and corpus-backed example sentences.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Oxford Dictionaries** node integrates workflows with Oxford University Press's lexical database. It provides programmatic access to rich dictionary entries, parts of speech, lexical categories, bilingual translations across language pairs, and authentic example sentences.

The node supports three primary operations:
- **`getDefinition`**: Fetches comprehensive dictionary entries including grammatical categories, primary and subsense definitions, phonetic transcriptions (IPA), and high-quality audio pronunciation links.
- **`getTranslation`**: Translates words and phrases between supported language pairs (e.g. English ↔ Spanish, English ↔ French, etc.), returning target lexical equivalents and contextual senses.
- **`getExamples`**: Retrieves natural example sentences sourced from real-world Oxford corpora illustrating practical usage of words in context.

### Key Capabilities

- **Three Specialized Operations:** Seamlessly switch between definitions, bilingual translations, and sentence examples.
- **Phonetics & Audio Pronunciations:** Access International Phonetic Alphabet (IPA) spellings and direct `.mp3` audio files.
- **Sandbox & Production Environments:** Toggle between Oxford's free testing sandbox and full production endpoints.
- **Multi-Dialect Support:** Supports regional variants such as British English (`en-gb`) and American English (`en-us`).
- **Automatic Case & URL Normalization:** Automatically trims and lowercase-converts word inputs and languages before dispatching requests.
- **Dynamic Expression Binding:** Accepts words, language pairs, and API credentials from incoming workflow properties via expressions.

### Common Use Cases

- **Language Learning & Flashcards:** Automatically generate vocabulary cards with definitions, phonetic transcriptions, and audio links in Anki or Notion.
- **Content Enrichment & Tooltips:** Attach authoritative definitions and synonyms to technical terms in blogs and documentation.
- **Translation Pipelines:** Provide contextual translation lookups and cross-lingual sense mappings for multilingual applications.
- **NLP & Corpus Analysis:** Extract real-world sentence examples for grammar validation and machine learning training prompts.
- **Voice & Speech Workflows:** Extract direct audio pronunciation URLs to embed into speech bots and educational tools.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## How to Use the Oxford Dictionaries Node

Configure the node with your Oxford Dictionaries Developer credentials (`App ID` and `App Key`), choose your desired operation, and specify the target word and language.

![Oxford Dictionaries Configuration](icon.svg)

### Step-by-Step Setup in the Visual Builder

1. **Obtain API Credentials:** Sign up at the [Oxford Dictionaries Developer Portal](https://developer.oxforddictionaries.com/) to obtain your `App ID` and `App Key`.
2. **Add the Node:** Drag the **Oxford Dictionaries** node from the **Language** category onto the canvas.
3. **Set Credentials:** Enter your `App ID` and `App Key` in the respective parameter fields.
4. **Choose the Environment:** Check **Sandbox** if using Oxford's free prototype endpoint, or uncheck it for production accounts.
5. **Select the Operation:**
   - **`getDefinition`**: Retrieve definitions, senses, and audio pronunciations.
   - **`getTranslation`**: Translate terms between language pairs.
   - **`getExamples`**: Get real-world sentence examples.
6. **Configure Operation Parameters:**
   - For `getDefinition` & `getExamples`: Set **Language** (e.g. `en-gb`, `en-us`, `en`) and **WordId** (e.g. `ace`, `apple`).
   - For `getTranslation`: Set **SourceLang** (e.g. `en`), **TargetLang** (e.g. `es`), and **WordId** (e.g. `ace`).
7. **Connect Outputs:** Wire the `success` output port to a `Function`, `Log`, or downstream AI node.

---

### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `appId` | `string` | ✅ Yes | — | Your Oxford Dictionaries Developer Application ID. |
| `appKey` | `string` | ✅ Yes | — | Your Oxford Dictionaries Developer Application Key. |
| `sandbox` | `boolean` | ❌ No | `true` | When `true`, targets the Sandbox API (`od-api-sandbox.oxforddictionaries.com`). Set to `false` for production. |
| `operation` | `enum` | ❌ No | `getDefinition` | The lexical lookup operation: `getDefinition`, `getTranslation`, or `getExamples`. |
| `wordId` | `string` | ✅ Yes* | — | The word or phrase to query (e.g. `ace`, `apple`, `action`). *Required at runtime. |
| `language` | `string` | ❌ No* | `en-gb` | Language code for definitions/examples (e.g. `en-gb`, `en-us`, `en`, `es`). *Required for `getDefinition` and `getExamples`. |
| `sourceLang` | `string` | ❌ No* | — | Source language code for translation (e.g. `en`, `es`). *Required for `getTranslation`. |
| `targetLang` | `string` | ❌ No* | — | Target language code for translation (e.g. `es`, `en`). *Required for `getTranslation`. |

---

### Operations & Endpoints

| Operation | Target API Endpoint | Required Fields | Returned Data |
|-----------|---------------------|-----------------|---------------|
| **`getDefinition`** | `/api/v2/entries/{language}/{wordId}` | `appId`, `appKey`, `language`, `wordId` | Senses, definitions, grammatical categories, phonetics, audio files, etymologies. |
| **`getTranslation`** | `/api/v2/translations/{sourceLang}/{targetLang}/{wordId}` | `appId`, `appKey`, `sourceLang`, `targetLang`, `wordId` | Bilingual sense equivalents, translated words, grammatical notes. |
| **`getExamples`** | `/api/v2/sentences/{language}/{wordId}` | `appId`, `appKey`, `language`, `wordId` | Corpus sentence examples showing practical contextual usage. |

---

### Sandbox vs. Production Environments

- **Sandbox (`sandbox: true`):** Targets `https://od-api-sandbox.oxforddictionaries.com/api/v2`. This free environment is restricted to a curated subset of test headwords (such as `ace`, `apple`, `action`, `test`, etc.). Ideal for developing and testing workflows without rate-limit constraints or billing.
- **Production (`sandbox: false`):** Targets `https://od-api.oxforddictionaries.com/api/v2`. Provides unrestricted access to Oxford's entire vocabulary across all licensed monolingual and bilingual datasets.

---

### Dynamic Value Resolution

Parameters can be mapped dynamically using expression syntax (e.g. `{{Function.success.searchWord}}`):

```json
{
  "operation": "getDefinition",
  "wordId": {
    "$expr": "output",
    "node": "Function",
    "outputId": "success",
    "path": "searchWord"
  },
  "language": {
    "$expr": "output",
    "node": "Function",
    "outputId": "success",
    "path": "searchLang"
  }
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Trigger payload or preceding node output containing dynamic lookup terms. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when Oxford Dictionaries API returns a valid response. Contains full lexical data. |
| `error` | `Error` | Emitted when authentication fails, required fields are missing, or word is not found. |

---

### Output Schema: `getDefinition`

```json
{
  "id": "ace",
  "metadata": {
    "operation": "retrieve",
    "provider": "Oxford University Press",
    "schema": "RetrieveEntry"
  },
  "results": [
    {
      "id": "ace",
      "language": "en-gb",
      "lexicalEntries": [
        {
          "entries": [
            {
              "etymologies": [
                "Middle English: via Old French from Latin as 'unity, a unit'..."
              ],
              "pronunciations": [
                {
                  "audioFile": "https://audio.oxforddictionaries.com/en/mp3/ace_gb_1.mp3",
                  "dialects": [
                    "British English"
                  ],
                  "phoneticNotation": "IPA",
                  "phoneticSpelling": "e\u026as"
                }
              ],
              "senses": [
                {
                  "definitions": [
                    "a playing card with a single spot on it, ranked as the highest card in its suit in most card games."
                  ],
                  "id": "m_en_gbus0005680.006",
                  "shortDefinitions": [
                    "playing card with single spot"
                  ]
                }
              ]
            }
          ],
          "lexicalCategory": {
            "id": "noun",
            "text": "Noun"
          },
          "text": "ace"
        }
      ],
      "type": "headword",
      "word": "ace"
    }
  ]
}
```

---

### Output Schema: `getTranslation`

```json
{
  "id": "ace",
  "metadata": {
    "operation": "translations",
    "provider": "Oxford University Press"
  },
  "results": [
    {
      "id": "ace",
      "language": "en",
      "lexicalEntries": [
        {
          "entries": [
            {
              "senses": [
                {
                  "translations": [
                    {
                      "language": "es",
                      "text": "as",
                      "notes": [
                        {
                          "text": "masculine",
                          "type": "grammatical note"
                        }
                      ]
                    }
                  ]
                }
              ]
            }
          ],
          "lexicalCategory": {
            "id": "noun",
            "text": "Noun"
          },
          "text": "ace"
        }
      ],
      "word": "ace"
    }
  ]
}
```

---

### Output Schema: `getExamples`

```json
{
  "id": "ace",
  "metadata": {
    "operation": "sentences",
    "provider": "Oxford University Press"
  },
  "results": [
    {
      "id": "ace",
      "language": "en",
      "lexicalEntries": [
        {
          "lexicalCategory": {
            "id": "noun",
            "text": "Noun"
          },
          "sentences": [
            {
              "regions": [
                "British"
              ],
              "senseIds": [
                "m_en_gbus0005680.006"
              ],
              "text": "Life had dealt him an ace and he was going to play it."
            },
            {
              "text": "He served an ace to win the opening set."
            }
          ],
          "text": "ace"
        }
      ],
      "word": "ace"
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Practical Examples

### Example 1: British English Definition Lookup

Lookup the definition and phonetic pronunciation for `"ace"` in British English (`en-gb`).

**Configuration:**
```json
{
  "operation": "getDefinition",
  "appId": "YOUR_APP_ID",
  "appKey": "YOUR_APP_KEY",
  "sandbox": true,
  "language": "en-gb",
  "wordId": "ace"
}
```

---

### Example 2: English to Spanish Translation

Translate `"apple"` from English (`en`) to Spanish (`es`).

**Configuration:**
```json
{
  "operation": "getTranslation",
  "appId": "YOUR_APP_ID",
  "appKey": "YOUR_APP_KEY",
  "sandbox": true,
  "sourceLang": "en",
  "targetLang": "es",
  "wordId": "apple"
}
```

**Output Translation Sense:**
```json
{
  "language": "es",
  "text": "manzana",
  "notes": [
    {
      "text": "feminine",
      "type": "grammatical note"
    }
  ]
}
```

---

### Example 3: Natural Sentence Examples Retrieval

Fetch contextual sentences for `"action"` in English.

**Configuration:**
```json
{
  "operation": "getExamples",
  "appId": "YOUR_APP_ID",
  "appKey": "YOUR_APP_KEY",
  "sandbox": true,
  "language": "en",
  "wordId": "action"
}
```

---

### Example 4: Extracting Audio Pronunciation & Phonetic Spelling

Chain Oxford Dictionaries with a `Function` node to extract the direct MP3 audio URL and IPA spelling.

**Downstream Function Code:**
```javascript
const pronunciation = input.results[0].lexicalEntries[0].entries[0].pronunciations[0];

return {
  word: input.results[0].word,
  phonetic: pronunciation.phoneticSpelling,
  audioUrl: pronunciation.audioFile
};
```

**Result Object:**
```json
{
  "word": "ace",
  "phonetic": "e\u026as",
  "audioUrl": "https://audio.oxforddictionaries.com/en/mp3/ace_gb_1.mp3"
}
```

---

### Example 5: Extracting Plain-Text Definitions

Use a `Function` node to flatten Oxford's nested response into a simple definition string.

**Downstream Function Code:**
```javascript
const definition = input.results[0].lexicalEntries[0].entries[0].senses[0].definitions[0];

return {
  definition: definition
};
```

---

### Example 6: Dynamic Lookup via Upstream Input

Pass vocabulary terms dynamically from a spreadsheet or form.

**Upstream Function Payload:**
```json
{
  "searchWord": "ace",
  "searchLang": "en-gb"
}
```

**Oxford Dictionaries Parameter Mappings:**
- **WordId (Expression):** `{{Function.success.searchWord}}`
- **Language (Expression):** `{{Function.success.searchLang}}`

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Interactive Workflow Preview

```fusion-workflow
src: example.workflow.json
title: Oxford Dictionaries Lookup and Translation Workflows
```

---

### Sample Workflows

#### 1. Vocabulary Extraction Pipeline: Trigger ➔ Oxford Dictionaries ➔ Function ➔ Log

A workflow that retrieves definitions, extracts audio links, and logs clean results:

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger",
      "label": "Lookup Word"
    },
    {
      "id": "oxford",
      "type": "oxford-dictionaries",
      "config": {
        "operation": "getDefinition",
        "appId": "YOUR_APP_ID",
        "appKey": "YOUR_APP_KEY",
        "sandbox": true,
        "language": "en-gb",
        "wordId": "ace"
      }
    },
    {
      "id": "format",
      "type": "function",
      "config": {
        "code": "const entry = input.results[0].lexicalEntries[0];\nreturn {\n  word: input.results[0].word,\n  category: entry.lexicalCategory.text,\n  definition: entry.entries[0].senses[0].definitions[0],\n  audio: entry.entries[0].pronunciations?.[0]?.audioFile\n};"
      }
    },
    {
      "id": "output",
      "type": "log"
    }
  ],
  "connections": [
    {
      "source": "trigger",
      "target": "oxford"
    },
    {
      "source": "oxford",
      "target": "format"
    },
    {
      "source": "format",
      "target": "output"
    }
  ]
}
```

---

### Architecture Patterns

- **Automated Flashcard Generator:** `Google Sheets (New Word) ➔ Oxford Dictionaries (getDefinition) ➔ Function (Extract IPA + Definition + Audio) ➔ Anki / Notion API`.
- **Multilingual Content Assistant:** `Webhook (Article text) ➔ Oxford Dictionaries (getTranslation) ➔ AI Chat (Draft localized copy)`.
- **Grammar & Sentence Helper:** `User Query ➔ Oxford Dictionaries (getExamples) ➔ Function (Slice top 3 sentences) ➔ Discord / Slack Bot`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors & Resolutions

#### `Language and word ID are required`
- **Cause:** When executing `getDefinition` or `getExamples`, either the `language` or `wordId` parameter was left empty.
- **Solution:** Specify both `language` (e.g. `en-gb`, `en`) and `wordId` (e.g. `ace`).

---

#### `Source language, target language, and word ID are required`
- **Cause:** When executing `getTranslation`, one of `sourceLang`, `targetLang`, or `wordId` was missing.
- **Solution:** Ensure all three fields are provided (e.g. `sourceLang: "en"`, `targetLang: "es"`, `wordId: "apple"`).

---

#### `Oxford Dictionaries API error (404): Not Found`
- **Cause:**
  1. The headword was not found in the Oxford Dictionaries lexicon.
  2. You are using the **Sandbox** (`sandbox: true`) with a word not included in the sandbox prototype dataset.
- **Solution:**
  - Verify spelling.
  - In sandbox mode, test with supported sandbox headwords (e.g. `ace`, `apple`, `action`). For full dictionary access, switch to production (`sandbox: false`).

---

#### `Oxford Dictionaries API error (403): Forbidden`
- **Cause:** Invalid or expired `App ID` / `App Key`, or your plan does not have access to the requested language dataset.
- **Solution:** Check your developer credentials on [developer.oxforddictionaries.com](https://developer.oxforddictionaries.com/).

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Oxford Dictionaries client not initialized` | Setup failure | Ensure `appId` and `appKey` are non-empty strings. |
| `Language and word ID are required` | Missing parameters | Provide `language` and `wordId`. |
| `Source language, target language, and word ID are required` | Missing translation parameters | Provide `sourceLang`, `targetLang`, and `wordId`. |
| `API error (404): Not Found` | Word not in lexicon / sandbox restriction | Check spelling or test with sandbox-supported headwords. |
| `API error (403): Forbidden` | Authentication failure | Verify `App ID` and `App Key`. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security & Best Practices

- **Never Commit Plain Credentials:** Avoid hardcoding live Oxford `App ID` and `App Key` values into exported workflows. Use Fusion credential management or workflow secrets.
- **Rate Limits:** Free developer tiers have request-per-minute limitations. Use a `Delay` or `Throttle` node when processing batch vocabulary lists.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related Nodes

- [Datamuse](../datamuse/en.md) — Find synonyms, rhymes, and word associations
- [OpenThesaurus](../openthesaurus/en.md) — Query German synsets and thesaurus data
- [LanguageTool](../language-tool/en.md) — Grammar and spelling checking
- [DeepL](../deepl/en.md) — High-accuracy machine translation
- [Function](../function/en.md) — Extract and transform Oxford dictionary structures

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial release of Oxford Dictionaries Action Node with `getDefinition`, `getTranslation`, and `getExamples` operations |

<!-- /SECTION: changelog -->

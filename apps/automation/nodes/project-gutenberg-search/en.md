---
node_id: "project-gutenberg-search"
title: "Project Gutenberg Search"
description: "Search for free books from Project Gutenberg via Gutendex API."
category: "web-search-information"
subcategory: "Book Search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - project-gutenberg
  - gutendex
  - books
  - ebooks
  - literature
  - search
  - public-domain
  - author
  - library
related_nodes:
  - open-library-search
  - wikipedia
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# Project Gutenberg Search

> **Category:** Web Search & Information | **Subcategory:** Book Search | **Type:** Action Node

Search and discover over 70,000 free, public-domain ebooks from the [Project Gutenberg](https://www.gutenberg.org/) digital library via the [Gutendex](https://gutendex.com/) API. Retrieve rich metadata, author biographies, subject classifications, book covers, and direct download links in EPUB, HTML, Kindle, and plain text formats.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Project Gutenberg Search** node connects workflows to [Gutendex](https://gutendex.com/), a high-performance RESTful Web API for Project Gutenberg's catalog. Project Gutenberg is the world's oldest and largest digital repository of free public-domain cultural works, classic literature, and historical texts.

With this node, you can look up ebooks by exact Gutenberg ID, execute full-text keyword searches matching book titles and author names, or filter collections by language codes (e.g., English, French, Spanish, German).

Each search result returns comprehensive book objects, including author birth/death years, subject classifications, cover image URLs, download counts, and direct links to downloadable formats (`.epub`, `.mobi`, `.html`, `.txt`).

### Key Features

- **Multi-Parameter Search:** Search by keywords (title/author), lookup specific Gutenberg IDs, or filter by language codes.
- **Direct Ebook Download Links:** Get immediate download links for multiple file formats, including EPUB, Mobipocket/Kindle, HTML, UTF-8 plain text, and cover images.
- **Rich Bibliographic Metadata:** Access author details (birth/death years), translators, Library of Congress subject headings, bookshelves, and total download counts.
- **Batch ID Lookup:** Pass single or comma-separated Project Gutenberg IDs (e.g., `1342,11,84`) to retrieve multiple specific titles in one request.
- **Multi-Language Filtering:** Filter works by 2-letter ISO 639-1 language codes (such as `en`, `fr`, `es`, `de`, `zh`, `it`) or comma-separated lists (`en,fr`).
- **Zero Configuration / Public API:** Direct integration with no API keys, tokens, or credentials required.

### Data Flow

```text
Incoming Trigger / Payload (e.g. search query or IDs)
                    ↓
        Configure Search Parameters
        (ids, search, languages)
                    ↓
         Query Gutendex API
   (https://gutendex.com/books/?...)
                    ↓
          Parse JSON Results
   (count, next/previous URLs, results array)
                    ↓
     Success Output (Log / AI / Pipeline)
```

### Use Cases

- **AI Summarization & Literature Pipelines:** Fetch classic book texts and summaries by author or topic and pass them to AI nodes for chapter extraction, analysis, or quiz generation.
- **Digital Library & Reader Apps:** Build automated catalogs and discovery feeds for reading apps with cover images and direct EPUB download links.
- **Educational & Academic Research:** Query historical texts, literature by period, or author bibliographies for educational workflows and school curriculums.
- **Ebook Archiving & Format Distribution:** Automate downloading and storing public domain classics into cloud storage buckets (AWS S3, Google Drive).
- **Book Recommendation Bots:** Power Slack, Discord, or Telegram bots that recommend public domain books based on user queries or reading preferences.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ids` | `string` | ❌ No | — | One or more comma-separated Project Gutenberg book IDs to fetch (e.g., `1342`, `1342,11,84`). |
| `search` | `string` | ❌ No | — | Search query terms matching book titles, author names, or keywords (e.g., `"victor hugo"`, `"pride and prejudice"`, `"dostoevsky"`). |
| `languages` | `string` | ❌ No | — | Filter results by comma-separated 2-letter ISO 639-1 language codes (e.g., `en`, `fr`, `es`, `de`, `en,fr`). |

---

### Parameter Details

#### `ids`
Allows direct lookup of specific Project Gutenberg book numbers.
- **Type:** `string`
- **Required:** No
- **Format:** Single numeric ID (`1342`) or comma-separated numeric IDs (`1342,11,84`).
- **Example Values:**
  - `1342` *(Pride and Prejudice by Jane Austen)*
  - `11` *(Alice's Adventures in Wonderland by Lewis Carroll)*
  - `84` *(Frankenstein by Mary Wollstonecraft Shelley)*
  - `1342,11,84` *(Batch retrieval of all three classics)*

#### `search`
Searches for text appearing in book titles or author names.
- **Type:** `string`
- **Required:** No
- **Behavior:** Words are matched case-insensitively across titles and authors. Multiple words are treated as search terms.
- **Example Values:**
  - `"victor hugo"`
  - `"sherlock holmes"`
  - `"the time machine"`
  - `"dante alighieri"`

#### `languages`
Restricts search results to books written in specific languages.
- **Type:** `string`
- **Required:** No
- **Format:** Single or comma-separated 2-letter ISO 639-1 language codes.
- **Common Language Codes:**
  - `en` — English
  - `fr` — French
  - `es` — Spanish
  - `de` — German
  - `it` — Italian
  - `pt` — Portuguese
  - `zh` — Chinese
  - `la` — Latin
  - `ar` — Arabic
- **Example Values:** `"en"`, `"fr"`, `"es,fr"`

---

### API Endpoint

The node queries the Gutendex public API endpoint:

```text
https://gutendex.com/books/?ids={ids}&search={search}&languages={languages}
```

> [!NOTE]
> All parameters are optional. Calling the node without any parameters returns the most popular books on Project Gutenberg ordered by download count.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution data. Parameters can be bound dynamically using expressions (e.g., `{{outputs.Function.success.query}}`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the search succeeds. Contains total match count, pagination URLs, and the list of book objects. |
| `error` | `Error` | Emitted when API communication fails or invalid parameters are provided. |

---

### Output Data Structure

The `success` output payload provides the following JSON structure:

```json
{
  "count": 73245,
  "next": "https://gutendex.com/books/?page=2&search=victor+hugo",
  "previous": null,
  "results": [
    {
      "id": 135,
      "title": "Les Misérables",
      "authors": [
        {
          "name": "Hugo, Victor",
          "birth_year": 1802,
          "death_year": 1885
        }
      ],
      "translators": [
        {
          "name": "Hapgood, Isabel Florence",
          "birth_year": 1850,
          "death_year": 1928
        }
      ],
      "subjects": [
        "Epic literature",
        "Ex-convicts -- Fiction",
        "Historical fiction",
        "Orphans -- Fiction",
        "Paris (France) -- Fiction"
      ],
      "bookshelves": [
        "Historical Fiction",
        "Movie Books"
      ],
      "languages": [
        "en"
      ],
      "copyright": false,
      "media_type": "Text",
      "formats": {
        "text/html": "https://www.gutenberg.org/ebooks/135.html.images",
        "application/epub+zip": "https://www.gutenberg.org/ebooks/135.epub3.images",
        "application/x-mobipocket-ebook": "https://www.gutenberg.org/ebooks/135.kindle.images",
        "application/rdf+xml": "https://www.gutenberg.org/ebooks/135.rdf",
        "image/jpeg": "https://www.gutenberg.org/cache/epub/135/pg135.cover.medium.jpg",
        "text/plain; charset=us-ascii": "https://www.gutenberg.org/ebooks/135.txt.utf-8",
        "application/octet-stream": "https://www.gutenberg.org/cache/epub/135/pg135-h.zip"
      },
      "download_count": 8943
    }
  ]
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `count` | `number` | Total number of books in Project Gutenberg matching the query filters. |
| `next` | `string \| null` | URL link to fetch the subsequent page of results (if more results exist). |
| `previous` | `string \| null` | URL link to the previous page of results (if on page 2 or higher). |
| `results` | `array` | Array of book metadata objects matching the search criteria. |
| `results[].id` | `number` | Unique Project Gutenberg catalog identifier for the book. |
| `results[].title` | `string` | Full title of the book. |
| `results[].authors` | `array` | List of author objects with `name` (*"Last, First"*), `birth_year`, and `death_year`. |
| `results[].translators` | `array` | List of translator objects with `name`, `birth_year`, and `death_year`. |
| `results[].subjects` | `array` | Library of Congress Subject Headings (LCSH) describing themes and genres. |
| `results[].bookshelves` | `array` | Curated Project Gutenberg collection categories and bookshelves. |
| `results[].languages` | `array` | ISO 639-1 language code array of the text edition. |
| `results[].copyright` | `boolean` | `false` indicates public domain status in the United States. |
| `results[].media_type` | `string` | Type of media resource (e.g., `"Text"`). |
| `results[].formats` | `object` | Map of MIME types to direct downloadable file URLs (EPUB, HTML, Plain Text, Cover Images). |
| `results[].download_count` | `number` | Total historical download count on Project Gutenberg. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Popular Books Discovery (Default)

Retrieve the most popular books on Project Gutenberg without specifying filters.

**Parameter Configuration:**

```text
Ids: (empty)
Search: (empty)
Languages: (empty)
```

**Output:**
Returns the top downloaded books across all languages, including titles like *Pride and Prejudice*, *Frankenstein*, and *Romeo and Juliet*.

---

### Example 2: Search by Author & Language

Search for works written by Victor Hugo available in French.

**Parameter Configuration:**

```text
Ids: (empty)
Search: victor hugo
Languages: fr
```

**Output:**
Returns French editions of Victor Hugo's masterpieces, such as *Les Misérables*, *Notre-Dame de Paris*, and *Les Châtiments*.

---

### Example 3: Exact Book ID Lookup

Fetch precise metadata and download links for specific Gutenberg titles (*Pride and Prejudice* ID 1342, *Alice in Wonderland* ID 11, and *Frankenstein* ID 84).

**Parameter Configuration:**

```text
Ids: 1342,11,84
Search: (empty)
Languages: (empty)
```

**Output:**
Returns the exact 3 book objects with cover images, EPUB links, and author details.

---

### Example 4: Title Keyword Search

Search for books containing "sherlock holmes" in English.

**Parameter Configuration:**

```text
Ids: (empty)
Search: sherlock holmes
Languages: en
```

**Output:**
Returns Arthur Conan Doyle's Sherlock Holmes collection (*The Adventures of Sherlock Holmes*, *The Hound of the Baskervilles*, *A Study in Scarlet*).

---

### Example 5: Automated Ebook Ingestion Pipeline

Search for classic science fiction books and forward direct EPUB links to cloud storage.

**Workflow Pipeline:**

```text
Manual Trigger
  → Project Gutenberg Search (Search: "h.g. wells", Languages: "en")
  → Function (extract results[0].formats["application/epub+zip"])
  → HTTP Request (Download EPUB binary)
  → AWS S3 (Upload to "my-ebooks-bucket")
  → Log (print S3 URL)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: exampl.workflow.json
title: Search for free books from Project Gutenberg via Gutendex API
```

### Common Workflow Patterns

- **Search & Log:** `Manual Trigger` → `Project Gutenberg Search` → `Log` (Quickly test searches and inspect JSON structures in the console).
- **AI Book Summarizer:** `Webhook Trigger` → `Project Gutenberg Search` → `Function` (Fetch Plain Text URL) → `AI Chat / LLM` (Summarize key themes or chapters).
- **Catalog Database Ingestion:** `Cron Trigger` → `Project Gutenberg Search` (`languages: "en"`) → `For Each` → `Postgres / MongoDB Action` (Upsert book records).
- **Reading Bot Notification:** `Manual Trigger` → `Project Gutenberg Search` → `Function` → `Slack / Discord Webhook` (Post "Classic Book of the Day" with cover and download link).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No Books Found / Empty Results Array
- **Symptom:** The `results` array is empty (`[]`) and `count` is `0`.
- **Cause:** Search query is too specific, misspelled, or combines an incompatible language filter.
- **Solution:** Broaden search keywords (e.g., search `"tolstoy"` instead of `"leo tolstoy war and peace special edition"`), verify author spelling, or clear the `languages` filter.

#### Invalid Book ID
- **Symptom:** Looking up an ID returns an empty result set.
- **Cause:** The ID does not exist in Project Gutenberg or is formatted with non-numeric characters.
- **Solution:** Verify the book ID on [gutenberg.org](https://www.gutenberg.org/) and ensure IDs are purely numeric and comma-separated (e.g., `1342,11`).

#### Desired Download Format Unavailable
- **Symptom:** A specific MIME type (such as `application/epub+zip` or `text/plain; charset=utf-8`) is missing in `formats`.
- **Cause:** Older or special Gutenberg items might only be available in HTML or plain ASCII text.
- **Solution:** In downstream **Function** nodes, check for format fallback keys: `formats['application/epub+zip'] || formats['text/html'] || formats['text/plain']`.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Network error` | Could not reach `gutendex.com` API | Verify internet connectivity and check if `gutendex.com` is operational |
| `HTTP 429 Too Many Requests` | Too many frequent queries to Gutendex | Introduce rate-limiting using the **Delay** or **Debounce** node |
| `HTTP 500 / 502 Bad Gateway` | Upstream Gutendex service maintenance | Retry the request after a short interval |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Open Library Search](../open-library-search/en.md) — Search millions of books and editions via Open Library API
- [Wikipedia](../wikipedia/en.md) — Search and retrieve articles and summaries from Wikipedia
- [HTTP Request](../http-request/en.md) — Download ebook binaries or query other public REST APIs
- [Function](../function/en.md) — Transform, filter, and extract specific ebook format links
- [Log](../log/en.md) — Inspect search results and metadata objects in the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial release supporting Project Gutenberg search by ID, keywords, and language filters via Gutendex API |

<!-- /SECTION: changelog -->

---
node_id: "listen-notes"
title: "Listen Notes"
description: "Search podcasts and episodes using the Listen API from Listen Notes."
category: "Web Search & Information"
subcategory: "Media"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - listen-notes
  - podcasts
  - episodes
  - media
  - search
  - audio
related_nodes:
  - news-api
  - giphy-gif-search
  - tv-maze
  - log
---

<!-- SECTION: header -->
# Listen Notes

> **Category:** Web Search & Information | **Subcategory:** Media | **Type:** Action Node

Search the Listen Notes podcast directory for podcasts and episodes using the Listen API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Listen Notes** node searches podcast and episode metadata through the Listen API. It can return podcast results, episode results, or both, based on a search query.

### Key Features

- **Podcast Search:** Find podcasts by topic, person, place, or keyword
- **Episode Search:** Search individual podcast episodes
- **Combined Search:** Search podcasts and episodes in one request
- **Result Limits:** Control the number of results returned per page
- **API Authentication:** Use a Listen API key securely through workflow secrets
- **Workflow Ready:** Pass structured search results to downstream nodes
- **Error Routing:** Route authentication, request, and API errors to the error output

### Use Cases

- Build podcast discovery and recommendation workflows
- Find episodes about a specific topic or keyword
- Monitor podcast content for research or media analysis
- Enrich content-management and marketing workflows with podcast metadata
- Send matching podcasts or episodes to a database, report, or notification

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `api_key` | `string` | Yes | — | Listen API key. Store it as a workflow secret. |
| `q` | `string` | Yes | — | Search query, such as `artificial intelligence` |
| `type` | `enum` | No | `podcast` | Result type. Use `podcast`, `episode`, or `both` when supported by the API. |
| `page_size` | `number` | No | API default | Number of results to return per page |

### Search Query

The `q` parameter contains the text to search for. Examples include:

```text
artificial intelligence
technology news
climate change
```

### Result Type

Use `type` to focus the search:

- `podcast` — return podcast-level results
- `episode` — return individual episode results
- `both` — return both types when supported by the configured API version

### API and Authentication

Production requests use the Listen API and require a Listen API key. Configure the value through Fusion’s secret system, for example:

```json
{
  "api_key": "{{secrets.listenNotesApiKey}}"
}
```

Never commit a real Listen API key in a workflow file. The included workflow example contains only the placeholder `Tap your ListenNotes API key here`.

The production API base URL is:

```text
https://listen-api.listennotes.com/api/v2
```

See the [Listen API documentation](https://www.listennotes.com/en/api/docs/) for the current endpoint and plan details.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional dynamic search query or configuration object containing `api_key`, `q`, `type`, and `page_size` |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Listen API search response containing result records and pagination metadata |

### Success Output Example

```json
{
  "total": 1250,
  "results": [
    {
      "id": "example-podcast-id",
      "title_original": "AI Explained",
      "publisher_original": "Example Publisher",
      "description_original": "A podcast about artificial intelligence and technology."
    }
  ],
  "next_offset": 5
}
```

The exact fields vary according to the selected result type and the Listen API response.

### Error Output

Authentication failures, invalid parameters, rate-limit responses, network failures, and upstream API errors are routed to the error output.

```json
{
  "success": false,
  "error": "Listen API request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search Podcasts

Find podcasts about artificial intelligence.

```json
{
  "api_key": "{{secrets.listenNotesApiKey}}",
  "q": "artificial intelligence"
}
```

### Search Episodes

Find individual episodes about artificial intelligence and return up to five results.

```json
{
  "api_key": "{{secrets.listenNotesApiKey}}",
  "q": "artificial intelligence",
  "type": "episode",
  "page_size": 5
}
```

### Dynamic Search from Input

A previous node can provide the query and search options dynamically:

```json
{
  "q": "renewable energy",
  "type": "episode",
  "page_size": 10
}
```

Keep the API key configured as a workflow secret even when the other values come from incoming data.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search podcasts and episodes with Listen Notes
```

### Common Patterns

- **Podcast Discovery:** Manual Trigger → Listen Notes → Log
- **Scheduled Monitoring:** Cron → Listen Notes → Database
- **Episode Research:** Trigger → Listen Notes → Function → Report
- **Content Alerts:** Listen Notes → Filter → Notification

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### API key is missing

**Cause:** The `api_key` parameter is empty or the referenced workflow secret is not configured.

**Solution:** Add a valid Listen API key to the Fusion secret system and reference it with `{{secrets.listenNotesApiKey}}`.

#### Unauthorized request

**Cause:** The API key is invalid, expired, revoked, or not sent in the expected field.

**Solution:** Verify the key in Listen Notes and confirm that the node receives the secret through `api_key`.

#### Search query is missing

**Cause:** The required `q` parameter was not provided.

**Solution:** Add a non-empty search query or pass one through the incoming input.

#### Invalid result type

**Cause:** The `type` value is not supported by the configured Listen API version.

**Solution:** Use the supported podcast or episode result type and consult the current API documentation.

#### Rate limit exceeded

**Cause:** The workflow has exceeded the request limit for the configured Listen API plan.

**Solution:** Reduce request frequency, add a delay, or review the plan limits.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 401` | Missing or invalid API key | Configure a valid secret-backed API key |
| `HTTP 403` | API plan or permission restriction | Check the Listen API plan and endpoint access |
| `HTTP 429` | Rate limit exceeded | Reduce request frequency and retry later |
| `HTTP 4xx` | Invalid request parameters | Verify `q`, `type`, and `page_size` |
| `HTTP 5xx` | Listen API service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and API availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [News API](../news-api/en.md) - Search and retrieve news content
- [GIPHY GIF Search](../giphy-gif-search/en.md) - Search media content and GIFs
- [TV Maze](../tv-maze/en.md) - Search television shows and episodes
- **Log** - Inspect node output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial documentation |

<!-- /SECTION: changelog -->

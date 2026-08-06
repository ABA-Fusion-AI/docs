---
node_id: "health-topics-search"
title: "Health Topics Search"
description: "Get evidence-based health information from Health.gov MyHealthfinder API."
category: "health"
subcategory: "Public Health"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - health
  - medical
  - search
  - api
  - health.gov
related_nodes:
  - http-request
  - function
  - ai-chat
---

<!-- SECTION: overview -->
# Health Topics Search

> **Category:** Health &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Search for evidence-based health topics using the [Health.gov MyHealthfinder API](https://health.gov/myhealthfinder). Returns a list of matching health articles with titles and URLs — available in English and Spanish.

### Use Cases

- **Health Chatbots:** Feed user health questions into this node and return relevant government-backed articles.
- **Patient Education:** Automatically surface related health topics based on a patient's condition or diagnosis.
- **Content Pipelines:** Search for health topics and pass results to an AI node for summarization or triage.
- **Wellness Apps:** Dynamically populate health tip sections based on user-selected topics.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `topic` | `string` | Yes | — | The health topic or keyword to search for (e.g., `"diabetes"`, `"nutrition"`, `"exercise"`). |
| `language` | `enum` | No | `en` | Response language. Accepted values: `en` (English) or `es` (Spanish). |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | The search topic, either as a plain string or passed from an upstream node via expression. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the API call succeeds. Contains the search results. |
| `error` | `Error` | Emitted when the API call fails or returns no results. |

### Output Schema (`success`)

```json
{
  "search_term": "diabetes",
  "language": "en",
  "total_results": 11,
  "health_topics": [
    {
      "title": "Take Steps to Prevent Type 2 Diabetes",
      "url": "https://odphp.health.gov/myhealthfinder/health-conditions/diabetes/take-steps-prevent-type-2-diabetes",
      "last_updated": "1775595973",
      "section": "",
      "description": "",
      "content": []
    },
    {
      "title": "Gestational Diabetes Screening: Questions for the Doctor",
      "url": "https://odphp.health.gov/myhealthfinder/doctor-visits/talking-doctor/gestational-diabetes-screening-questions-doctor",
      "last_updated": "1779386509",
      "section": "",
      "description": "",
      "content": []
    }
  ]
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `search_term` | `string` | The keyword that was searched. |
| `language` | `string` | The language of the results (`en` or `es`). |
| `total_results` | `number` | Total number of matching health topics found. |
| `health_topics` | `array` | List of matching health topic articles. |
| `health_topics[].title` | `string` | Title of the health article. |
| `health_topics[].url` | `string` | Direct link to the full article on Health.gov. |
| `health_topics[].last_updated` | `string` | Unix timestamp of the last update. |
| `health_topics[].section` | `string` | Section category of the article (if available). |
| `health_topics[].description` | `string` | Short description of the article (if available). |
| `health_topics[].content` | `array` | Full article content sections (if available). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Health Topics by Keyword
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Function Node:** Returns the search keyword `"diabetes"` as a string.
3. **Health Topics Search Node:** Uses the keyword to query the Health.gov API with `language: en` and returns matching health articles.
4. **Log Node:** Displays the full results array.

**Function code used in the example:**
```js
return "diabetes";
```

### Common Patterns

- **User Input → Search:** Take a topic from a form or chatbot input, pass it via expression to `topic`, and return the matching articles.
- **AI Summarization:** Pass `health_topics` results to an AI Chat node to summarize the top articles in plain language.
- **Language Toggle:** Dynamically set `language` to `en` or `es` based on the user's locale preference.
- **Loop Over Results:** Feed the `health_topics` array into a For Each node to process or enrich each article individually.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `No results returned`
- **Cause:** The search term is too specific, misspelled, or not indexed in the MyHealthfinder database.
- **Solution:** Try broader keywords (e.g., `"heart"` instead of `"myocardial infarction"`). The API covers common health topics in plain-language terms.

#### Empty `description` and `content` fields
- **Cause:** The MyHealthfinder API returns topic metadata only in the search response. Full content is available when fetching individual topic pages.
- **Solution:** Use the `url` field from the results and pass it to an HTTP Request node to fetch the full article content.

#### Results in wrong language
- **Cause:** The `language` parameter is not set, defaulting to `en`.
- **Solution:** Explicitly set `language` to `es` for Spanish results.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Fetch the full article content using the `url` field from the results
- [AI Chat](./ai-chat.md) – Summarize or explain returned health topics using an LLM
- [Function](./function.md) – Build or transform the search topic before passing it to this node

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial documentation |

<!-- /SECTION: changelog -->

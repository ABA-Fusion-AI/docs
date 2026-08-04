---
node_id: "medium"
title: "Medium"
description: "Extract articles, publication details, and engagement metadata from Medium using the Medium API."
category: "content"
subcategory: "publishing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags:
  - medium
  - content
  - publishing
  - blog
  - article-extractor
  - no-code
related_nodes:
  - http-request
  - ai-agent
  - markdown
  - json-parser
  - slack
---

<!-- SECTION: overview -->
# Medium

> **Category:** Content&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Extract articles and metadata from Medium using the Medium API. The Medium node allows workflows to retrieve detailed information about a Medium article using its unique article ID.

### Use Cases

- Extract articles for AI summarization and newsletter creation
- Synchronize Medium blog posts with an internal CMS or knowledge base
- Track article performance metrics (claps, reading time, responses)
- Analyze post SEO structure, tags, and publishing dates
- Automate social media sharing upon publishing new articles

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `getArticle` | Operation to perform (`getArticle`). |
| `apiKey` | `string` | ✅ Yes | — | Your Medium API authentication key. |
| `articleId` | `string` | ✅ Yes | — | The unique Medium article identifier. |

### Operation Parameters

#### `getArticle`
Retrieves the complete information of a Medium article.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `includeContent` | `boolean` | ❌ No | `true` | Whether to include full HTML content in the output. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `main` | `object` | Triggers the node execution |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when article details are successfully retrieved |
| `error` | `Error` | Emitted when the request fails |

### Output Schema (`success`)

The **Medium** node queries the Medium REST API to fetch metadata, structural details, and content for a given post using its unique article ID.

### Key Features

- **Article Details Retrieval:** Extract title, subtitle, canonical URL, and full body content
- **Author & Publication Info:** Retrieve author username, author ID, and publication details
- **Engagement Statistics:** Access clap count, estimated reading time, and response counts
- **Tagging Metadata:** Retrieve all tags associated with the post

### Example 1: Get Article Details

Retrieve full details for a Medium article by its article ID.

**Configuration:**
```json
{
  "operation": "getArticle",
  "apiKey": "sec_medium_token_12345",
  "articleId": "034685e17707"
}
```

**Input:**
```json
{
  "articleId": "034685e17707"
}
```

**Output (success):**
```json
{
  "id": "034685e17707",
  "title": "Understanding AI Agents",
  "subtitle": "Building intelligent automation systems",
  "author": {
    "id": "123456",
    "name": "John Doe",
    "username": "@johndoe"
  },
  "publication": {
    "id": "987654",
    "name": "AI Weekly"
  },
  "url": "https://medium.com/ai-weekly/understanding-ai-agents-034685e17707",
  "publishedAt": "2026-07-21T14:20:00Z",
  "readingTime": 8,
  "claps": 1350,
  "responses": 42,
  "tags": [
    "AI",
    "Automation",
    "LLM"
  ],
  "content": "<html><body><h1>Understanding AI Agents</h1>...</body></html>"
}
```

---

### Example 2: Error Response

Returned when the API request fails (e.g. article not found).

**Output (error):**
```json
{
  "success": false,
  "error": {
    "message": "Article not found",
    "code": 404
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Medium Article AI Summarization to Slack

Automatically fetch a published Medium article, generate a summary using an AI Agent, and post the update to Slack.

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "medium-fetch",
      "type": "medium",
      "config": {
        "operation": "getArticle",
        "apiKey": "{{secrets.MEDIUM_API_KEY}}",
        "articleId": "{{input.body.articleId}}"
      }
    },
    {
      "id": "ai-summarize",
      "type": "ai-agent",
      "config": {
        "prompt": "Summarize this article: {{nodes.medium-fetch.output.content}}"
      }
    },
    {
      "id": "slack-notify",
      "type": "slack",
      "config": {
        "channel": "#content-updates",
        "message": "New Medium Article: {{nodes.medium-fetch.output.title}}\n\nSummary:\n{{nodes.ai-summarize.output.text}}"
      }
    }
  ]
}
```

**How it flows:**
1. Webhook receives a payload containing the `articleId`.
2. Medium node fetches full article content, author, and metadata.
3. AI Agent processes the article content and generates a summary.
4. Slack node posts the title and AI summary to the `#content-updates` channel.

### Common Patterns

- **AI Summarization:** Retrieve a Medium article and generate an AI-powered summary.
- **Knowledge Base Synchronization:** Import Medium articles into an internal CMS or documentation system.
- **SEO Analytics:** Analyze article structure, tags, engagement, and metadata across published posts.
- **Content Monitoring:** Track newly published articles for notification workflows.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Invalid API Key`

**Cause:** Authentication failed because the API key is missing or invalid.

**Solution:** Store the API key securely using secrets and verify it in your Medium account settings.

#### `Article Not Found`

**Cause:** The provided article ID does not exist or refers to a private post.

**Solution:** Check the article ID in the Medium URL (e.g. `034685e17707`) and verify post visibility.

#### `Rate Limit Exceeded`

**Cause:** API request limit reached.

**Solution:** Handle rate limiting with exponential backoff retries and cache article data when possible.

#### `Network Error`

**Cause:** Unable to reach the Medium API endpoints.

**Solution:** Check network connectivity and verify API status.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `Invalid API Key` | Authentication failed | Verify credentials and permissions |
| `Article Not Found` | Provided ID does not exist | Check `articleId` format and existence |
| `Rate Limit Exceeded` | Request threshold exceeded | Implement retry logic and caching |
| `Network Error` | Connectivity failure | Check network connection or API status |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Execute generic HTTP requests to external APIs
- [AI Agent](./ai-agent.md) – Generate summaries or content insights using AI
- [Markdown](./markdown.md) – Parse and transform markdown or HTML content
- [JSON Parser](./json-parser.md) – Extract specific JSON fields from outputs
- [Slack](./slack.md) – Send notifications and summaries to Slack channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-04 | Standardized documentation structure according to Node Documentation Guidelines |

<!-- /SECTION: changelog -->

```
---
node_id: "medium"
title: "Medium"
description: "Medium API - Extract data from Medium's website. Get articles, users, publications, and more."
category: "peer-only"
subcategory: "Medium"
version: "1.0.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - medium
  - social-media
  - blog
  - content
  - articles
related_nodes:
  - http-request
  - ai-chat
  - slack
---

<!-- SECTION: overview -->
# Medium

> **Category:** Peer Only&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Extract articles, user profiles, publications, and engagement data from Medium using the Medium API. The Medium node allows workflows to fetch detailed metadata, post content, top writers, and related articles for automated content ingestion, summarization, and distribution.

### Use Cases

- **Content Summarization:** Automatically extract Medium articles and generate summaries via AI LLM nodes.
- **CMS & Knowledge Base Sync:** Sync published Medium posts to internal document stores or blog platforms.
- **Performance & Analytics Tracking:** Track article claps, fan counts, response metrics, and reading times over time.
- **Author & Community Intelligence:** Discover top writers and related articles across specific topics.
- **Social Media Automation:** Trigger notifications or social posts whenever new articles or responses are published.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `getArticle` | The action to perform. See full list of operations below. |
| `apiKey` | `string` | ✅ Yes | — | Your Medium API authentication key. |

### Available Operations

| Operation | Description | Required Parameters |
|-----------|-------------|---------------------|
| `getArticle` | Retrieve complete details and metadata for a Medium article. | `articleId` |
| `getArticleContent` | Extract the raw HTML content of an article. | `articleId` |
| `getArticleMarkdown` | Extract the article content converted to Markdown format. | `articleId` |
| `getArticleFans` | Retrieve list of users who clapped/fanned an article. | `articleId` |
| `getArticleRelated` | Retrieve related articles for a given post. | `articleId` |
| `getUser` | Fetch profile information and statistics for a Medium user. | `username` / `userId` |
| `getUserArticles` | Retrieve a list of articles written by a specific user. | `username` / `userId` |
| `getUserTopArticles` | Fetch top-performing articles published by a user. | `username` / `userId` |
| `getPublication` | Get details and metadata of a Medium publication. | `publicationId` / `slug` |
| `getPublicationArticles` | Retrieve articles published within a specific publication. | `publicationId` / `slug` |
| `searchArticles` | Search for articles matching a specific keyword or query. | `query` |
| `getLatestPosts` | Fetch latest published posts by topic or global stream. | `topic` (optional) |
| `getTopWriters` | Retrieve top writers for a specified topic or tag. | `topic` |

### Operation Parameters

#### `getArticle`, `getArticleContent`, `getArticleMarkdown`, `getArticleFans`, `getArticleRelated`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `articleId` | `string` | ✅ Yes | — | The unique Medium article ID (e.g., `034685e17707`). |

#### `getUser`, `getUserArticles`, `getUserTopArticles`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `username` | `string` | ✅ Yes | — | Medium username (with or without `@`, e.g., `@johndoe`). |

#### `getPublication`, `getPublicationArticles`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `publicationId` | `string` | ✅ Yes | — | Medium publication ID or URL slug. |

#### `searchArticles`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | ✅ Yes | — | Search query string or keyword. |

#### `getTopWriters` / `getLatestPosts`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `topic` | `string` | ❌ No | — | Medium topic or tag category (e.g., `technology`, `artificial-intelligence`). |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node execution with incoming payload data. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the Medium API request succeeds. |
| `error` | `Error` | Emitted when the API request encounters an error. |

### Output Schema (`success`)

Depending on the selected `operation`, the node returns formatted JSON object output.

#### `getArticle` Output Example:

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

#### `getUser` Output Example:

```json
{
  "id": "123456",
  "username": "johndoe",
  "name": "John Doe",
  "bio": "AI Researcher & Software Engineer",
  "followersCount": 4520,
  "followingCount": 310,
  "imageUrl": "https://miro.medium.com/fit/c/128/128/1*sample.jpeg"
}
```

#### `error` Output Example:

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

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Extract Medium Article Details
```

### How it flows

1. **Manual Trigger:** Triggers the workflow execution manually on demand.
2. **Medium Node:** Queries the Medium API to fetch complete article details and metadata for a specified `articleId`.
3. **Log Node:** Receives the article payload from Medium and logs the structured result.

### Common Patterns

- **AI Content Processing:** Extract published Medium posts, convert to Markdown, and send to LLMs for translation, summarization, or audio synthesis.
- **Multi-Channel Publishing:** Monitor Medium publication articles and syndicate snippets to Slack, Discord, or newsletter distribution lists.
- **Author & Competitor Monitoring:** Periodically query `getUserTopArticles` or `getTopWriters` to track industry leaders and top content trends.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Invalid API Key`
- **Cause:** The provided API key is invalid, expired, or missing.
- **Solution:** Verify your Medium credentials and ensure the API key is passed securely via secrets.

#### `Article Not Found (404)`
- **Cause:** The `articleId` provided does not exist or the post is draft/private.
- **Solution:** Confirm the article ID from the Medium URL string (e.g. `034685e17707`) and check visibility.

#### `User Not Found`
- **Cause:** The specified `username` or `userId` does not exist on Medium.
- **Solution:** Verify the username handle spelling and ensure proper format without extraneous path components.

#### `Rate Limit Exceeded (429)`
- **Cause:** API request rate limit threshold exceeded.
- **Solution:** Implement delay or retry logic with exponential backoff between automated queries.

### Error Codes Summary

| Error Message | HTTP Code | Solution |
|---------------|-----------|----------|
| `Invalid API Key` | 401 | Check API key authentication settings |
| `Article / Resource Not Found` | 404 | Verify input IDs, slugs, or usernames |
| `Rate Limit Exceeded` | 429 | Reduce request frequency or add backoff delays |
| `Internal Server Error` | 500 / 503 | Check network connectivity and Medium service status |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Send custom REST API calls to external services
- [AI Chat](./ai-chat.md) – Generate summaries and insights using LLMs
- [Slack](./slack.md) – Distribute notifications and articles to Slack channels
- [Markdown](./markdown.md) – Parse, transform, or render markdown documents

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-05 | Standardized documentation structure according to Node Documentation Guidelines |

<!-- /SECTION: changelog -->

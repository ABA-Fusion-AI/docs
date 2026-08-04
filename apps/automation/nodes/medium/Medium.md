# Medium Node

Extract articles and metadata from Medium using the Medium API.

---

## Overview

The **Medium** node allows workflows to retrieve detailed information about a Medium article using its unique article ID.

This node is useful for:

- Content aggregation
- Blog synchronization
- AI summarization pipelines
- SEO analysis
- Content archiving
- Newsletter automation

---

## Operation

### Get Article

Retrieve the complete information of a Medium article.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| API Key | String | Yes | Your Medium API key. |
| Article ID | String | Yes | The unique Medium article identifier. |

---

## Inputs

| Input | Description |
|-------|-------------|
| Input | Starts the node execution. |

---

## Outputs

### Success

Returns the article data.

Example response:

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
  "url": "https://medium.com/...",
  "publishedAt": "2026-07-21T14:20:00Z",
  "readingTime": 8,
  "claps": 1350,
  "responses": 42,
  "tags": [
    "AI",
    "Automation",
    "LLM"
  ],
  "content": "<html>...</html>"
}
```

---

### Error

Returned when the request fails.

Example:

```json
{
  "success": false,
  "error": {
    "message": "Article not found"
  }
}
```

---

## Configuration Example

```text
Operation: Get Article

API Key:
xxxxxxxxxxxxxxxxxxxxxxxx

Article ID:
034685e17707
```

---

## Example Workflow

```
Webhook
    │
    ▼
Medium (Get Article)
    │
    ▼
AI Summarizer
    │
    ▼
Slack
```

---

## Common Use Cases

### AI Summarization

Retrieve a Medium article and generate an AI-powered summary.

---

### Knowledge Base

Import Medium articles into an internal documentation system.

---

### SEO Analytics

Analyze article structure, tags, and metadata.

---

### Content Monitoring

Track newly published articles for a specific workflow.

---

## Error Handling

Possible errors include:

| Error | Description |
|--------|-------------|
| Invalid API Key | Authentication failed. |
| Article Not Found | The provided article ID does not exist. |
| Rate Limit Exceeded | API request limit reached. |
| Network Error | Unable to reach the Medium API. |

---

## Best Practices

- Store the API key securely using secrets.
- Validate the article ID before execution.
- Handle rate limiting with retries.
- Cache article data when possible.
- Connect the **Error** output to a fallback or logging node.

---

## Notes

- The article ID is the unique identifier found in the Medium article URL.
- Only publicly accessible articles can be retrieved.
- Response fields may vary depending on the Medium API version.

---

## Related Nodes

- HTTP Request
- AI Agent
- Markdown
- JSON Parser
- Database
- Slack
- Email
```
---
node_id: "bing-copilot-search"
title: "Bing Copilot Search"
description: "Search Bing Copilot (AI chat) through SerpApi and get the AI-generated answer together with its source references."
category: "Web Search"
subcategory: "AI Search"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - bing
  - copilot
  - serpapi
  - search
  - ai-search
  - web-search
related_nodes:
  - bing-search
  - serp-api
  - tavily
  - ai-chat-openai
---

<!-- SECTION: overview -->
# Bing Copilot Search

> **Category:** Web Search &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Ask a question to **Bing Copilot** (Microsoft's AI chat) and get back its generated answer plus the web pages it cites. The node calls the **SerpApi** `bing_copilot` engine (`https://serpapi.com/search.json`), so it needs a **SerpApi API key**, not a Microsoft key.

The node returns:
- **`copilotResponse`:** The full AI answer (header + text blocks, code blocks kept with their line breaks).
- **`results`:** The references cited by Copilot (title, URL, snippet, source).

### Use Cases

- **AI-assisted research:** Get a sourced summary on a topic in one step, ready for a report or a newsletter.
- **Fact checking:** Retrieve an answer with its references so they can be verified downstream.
- **Content generation:** Feed the Copilot answer and its sources to an LLM node to write an article or a post.
- **Q&A automations:** Answer questions coming from a form, a chat or an email with an up-to-date web answer.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | SerpApi API key (from serpapi.com). Supports expressions. |
| `query` | `string` | No* | — | Question or search query (max 1000 characters). Supports expressions. |
| `num` | `number` | No | `10` | Maximum number of references to return (1–50). |
| `timeout` | `number` | No | `10000` | Request timeout in milliseconds (1000–30000). |
| `extractMetadata` | `boolean` | No | `true` | Adds a `metadata.source` field to each result. |
| `conversationStyle` | `enum` | No | `balanced` | `creative`, `balanced` or `precise`. **Not supported by SerpApi:** kept for compatibility, has no effect. |
| `includeRelatedQueries` | `boolean` | No | `false` | **Not supported by SerpApi:** kept for compatibility, always returns an empty list. |

\* If `query` is empty, the node uses the incoming data: a plain string, or the `query` field of the input object. If no query is found, the node fails.

### Authentication & API Key

1. Create an account on **SerpApi** (`https://serpapi.com`).
2. Open **Dashboard → API Key**.
3. Copy the key and paste it into the `apiKey` field (or store it in Fusion Secrets).

> Each run consumes one SerpApi search from your plan. Copilot answers take longer than classic searches: raise `timeout` (e.g. `20000`–`30000`) if you get timeouts.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` / `object` | Triggers the node. Used as the query when `query` is empty (string, or object with a `query` field). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Copilot answer and references (see below). |
| `error` | `Error` | Emitted on empty query, timeout, HTTP or SerpApi errors. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `query` | `string` | Query that was sent. |
| `copilotResponse` | `string` | AI-generated answer of Bing Copilot. |
| `results` | `array` | References: `title`, `url`, `snippet`, `displayUrl`, `metadata.source`. |
| `relatedQueries` | `array` | Only when `includeRelatedQueries` is `true`; always empty. |
| `total` | `number` | Number of references returned. |
| `timestamp` | `string` | ISO date of the search. |
| `conversationStyle` | `string` | Value of the `conversationStyle` parameter. |

### Output Example

```json
{
  "query": "What is the Model Context Protocol?",
  "copilotResponse": "The Model Context Protocol (MCP) is an open standard that lets AI applications connect to external tools and data sources...\n\nIt defines a client-server architecture where...",
  "results": [
    {
      "title": "Introduction - Model Context Protocol",
      "url": "https://modelcontextprotocol.io/introduction",
      "snippet": "MCP is an open protocol that standardizes how applications provide context to LLMs.",
      "displayUrl": "modelcontextprotocol.io",
      "metadata": { "source": "modelcontextprotocol.io" }
    }
  ],
  "total": 1,
  "timestamp": "2026-11-17T11:10:00.000Z",
  "conversationStyle": "balanced"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Ask Bing Copilot a question
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Bing Copilot Search Node:** Sends the query to Bing Copilot through SerpApi.
3. **Log Node:** Displays the AI answer and its references.

### Example Configuration

```json
{
  "apiKey": "{{ secrets.SERPAPI_API_KEY }}",
  "query": "{{ input.question }}",
  "num": 5,
  "timeout": 20000,
  "extractMetadata": true
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your SerpApi `apiKey` in Fusion's **Secrets** vault. Do not commit hardcoded keys into version control.

- The key is sent as a URL query parameter to SerpApi: avoid logging the full request URL.
- Monitor your SerpApi quota; each workflow run consumes one search.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Search query is required and cannot be empty`
- **Cause:** `query` is empty and the input contains no string or `query` field.
- **Solution:** Fill in `query`, or send `{ "query": "..." }` from the previous node.

#### `Search query exceeds maximum length of 1000 characters`
- **Cause:** The query is too long.
- **Solution:** Shorten the question.

#### `Search request timed out`
- **Cause:** Copilot took longer than `timeout` to answer.
- **Solution:** Increase `timeout` (up to `30000`).

#### `Bing search failed: HTTP 401: Invalid API key...`
- **Cause:** The SerpApi key is wrong (a Microsoft/Bing key does not work).
- **Solution:** Copy the key from your SerpApi dashboard.

#### `Bing search failed: HTTP 429: ...`
- **Cause:** No searches left on the SerpApi plan, or rate limit reached.
- **Solution:** Check your SerpApi usage or upgrade your plan.

#### `Search failed: SerpApi search returned status: Error`
- **Cause:** SerpApi could not complete the Copilot search.
- **Solution:** Retry, or rephrase the query.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: parameter reference, SerpApi authentication, output schema, and workflow integration |

<!-- /SECTION: changelog -->

---
node_id: "stack-exchange"
title: "StackExchange"
description: "Search questions from StackExchange API."
category: "devops-cloud-management"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - stack-exchange
  - stackoverflow
  - questions
  - developer-tools
  - search
  - programming
  - api
related_nodes:
  - http-request
  - github
  - ai-chat
  - json-placeholder
---

<!-- SECTION: header -->
# StackExchange

> **Category:** DevOps & Cloud Management | **Subcategory:** Developer Tools | **Type:** Action Node

Search questions, discussions, and developer solutions across Stack Overflow and the entire Stack Exchange network.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **StackExchange** node queries the official Stack Exchange 2.3 REST API to fetch questions, answers count, scores, and author details from any site across the Stack Exchange network (such as Stack Overflow, Server Fault, Super User, Ask Ubuntu, and others).

It allows workflows to monitor technical discussions, find top-voted solutions for specific technologies, aggregate recent questions by tag, or feed developer issues into downstream AI analysis or ticketing nodes.

### Key Features

- **Multi-Site Search:** Query Stack Overflow or any other community on the Stack Exchange network (e.g. `askubuntu`, `serverfault`, `superuser`, `math`, `unix`).
- **Tag-Based Filtering:** Narrow down searches by language, framework, or tool tags (e.g. `javascript`, `python`, `docker`, `kubernetes`).
- **Flexible Sorting:** Sort results by `activity`, `votes`, `creation`, or `relevance` in ascending or descending order.
- **Custom Page Size:** Control how many questions are returned per execution (default: `10`).
- **Rich Question Metadata:** Returns question ID, title, full question URL, tags, view count, answer count, vote score, accepted answer status, timestamps, and author/owner profile information.
- **Quota Tracking:** Exposes the remaining API request quota to help manage rate limits safely.

### Processing Flow

```text
Workflow Trigger / Upstream Data
  ↓
Validate node parameters (site, tag, sort, order, pagesize)
  ↓
Build query parameters & call Stack Exchange 2.3 Questions API
  ↓
Parse API response & extract question metadata
  ↓
Emit structured output envelope { success, questions, quota, total_results }
  ↓
Downstream nodes (AI Chat, Slack, Email, Database, Function)
```

### Use Cases

- **Developer Tech Monitoring:** Automatically fetch the latest questions tagged with your open-source library or company tool and notify your team on Slack or Discord.
- **AI-Powered Knowledge Base:** Search top-voted solutions on Stack Overflow and pass them to an AI Chat or Summarizer node to answer technical queries.
- **Daily Digest / Newsletter:** Aggregate weekly top questions for specific topics (e.g. `rust`, `devops`, `reactjs`) and email a summary.
- **Automated Issue Triaging:** Match newly logged error messages against known Stack Overflow questions to find solutions automatically.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `site` | `string` | No | `stackoverflow` | The Stack Exchange site identifier to query (e.g., `stackoverflow`, `serverfault`, `superuser`, `askubuntu`, `math`, `unix`). |
| `tagged` | `string` | No | `""` | Filter questions by tag (e.g., `javascript`, `python`, `docker`). Leave empty to search across all tags. |
| `sort` | `string` | No | `activity` | Sort criteria for the results: `activity` (last active), `votes` (highest score), `creation` (newest), or `relevance`. |
| `order` | `string` | No | `desc` | Sort direction: `desc` (descending, highest first) or `asc` (ascending, lowest first). |
| `pagesize` | `number` | No | `10` | Maximum number of questions to retrieve per request (typically between 1 and 100). |

### Popular Stack Exchange Site Identifiers

| Site Value | Community Name | Primary Focus |
|------------|----------------|---------------|
| `stackoverflow` | Stack Overflow | Software development and programming |
| `serverfault` | Server Fault | System administration and infrastructure |
| `superuser` | Super User | Computer enthusiasts and power users |
| `askubuntu` | Ask Ubuntu | Ubuntu operating system and tools |
| `unix` | Unix & Linux | Unix/Linux operating systems and shell |
| `dba` | Database Administrators | Database administration and SQL tuning |
| `security` | Information Security | Cybersecurity and digital security |
| `devops` | DevOps | Continuous delivery, CI/CD, and automation |

### Default Configuration

```json
{
  "site": "stackoverflow",
  "tagged": "",
  "sort": "activity",
  "order": "desc",
  "pagesize": 10
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or previous node payload passed into the execution step. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful API response, containing the list of questions, pagination metadata, and quota information. |
| `error` | `object` | Returned if the API request fails (e.g., invalid site name, network error, or rate limiting). |

### Output Schema

```typescript
{
  success: boolean;        // Always true when request succeeds
  site: string;            // The site queried (e.g., "stackoverflow")
  query: {
    tagged?: string;       // The tag filter used, if any
    sort: string;          // Sort criteria used
    order: string;         // Sort order (desc or asc)
    pagesize: number;      // Number of requested items
  };
  has_more: boolean;       // Whether more pages are available
  quota: {
    max: number | null;       // Maximum API quota allowed
    remaining: number | null; // Remaining API calls in current quota window
  };
  questions: Array<{
    question_id: number;           // Unique Stack Exchange question ID
    title: string;                 // Question title
    link: string;                  // Direct URL to the question
    tags: string[];                // List of tags attached to the question
    is_answered: boolean;          // True if the question has at least one accepted/valid answer
    view_count: number;            // Number of page views
    answer_count: number;          // Total number of submitted answers
    score: number;                 // Net upvotes minus downvotes
    accepted_answer_id: number | null; // ID of the accepted answer, if any
    creation_date: number;         // Unix timestamp of question creation
    last_activity_date: number;    // Unix timestamp of last activity
    owner: {
      user_id: number | null;      // Author user ID
      display_name: string;        // Author display name
      reputation: number;          // Author reputation points
      user_type: string;           // Account type (e.g., "registered")
      profile_image: string | null; // URL to author avatar
      link: string | null;         // URL to author profile
    };
  }>;
  total_results: number;   // Number of questions returned in this execution
  note: string;            // Helpful reminder of common sites
}
```

### Successful Response Example

```json
{
  "success": true,
  "site": "stackoverflow",
  "query": {
    "tagged": "python",
    "sort": "votes",
    "order": "desc",
    "pagesize": 1
  },
  "has_more": true,
  "quota": {
    "max": 300,
    "remaining": 296
  },
  "questions": [
    {
      "question_id": 231767,
      "title": "What does the \"yield\" keyword do in Python?",
      "link": "https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do-in-python",
      "tags": ["python", "iterator", "generator", "yield", "coroutine"],
      "is_answered": true,
      "view_count": 3284000,
      "answer_count": 48,
      "score": 13420,
      "accepted_answer_id": 231855,
      "creation_date": 1224754567,
      "last_activity_date": 1720000000,
      "owner": {
        "user_id": 18300,
        "display_name": "Alex.M.",
        "reputation": 165400,
        "user_type": "registered",
        "profile_image": "https://www.gravatar.com/avatar/example",
        "link": "https://stackoverflow.com/users/18300/alex-m"
      }
    }
  ],
  "total_results": 1,
  "note": "Common sites: stackoverflow, serverfault, superuser, etc."
}
```

### Error Response Example

```json
{
  "success": false,
  "error": "StackExchange lookup failed: StackExchange API error: 400"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Top Voted JavaScript Questions

Retrieve the top 3 highest-voted JavaScript questions on Stack Overflow:

```json
{
  "site": "stackoverflow",
  "tagged": "javascript",
  "sort": "votes",
  "order": "desc",
  "pagesize": 3
}
```

### Example 2: Recent Docker Questions on Ask Ubuntu

Fetch the 5 newest questions about Docker from Ask Ubuntu:

```json
{
  "site": "askubuntu",
  "tagged": "docker",
  "sort": "creation",
  "order": "desc",
  "pagesize": 5
}
```

### Example 3: Active Server Fault Discussions

Monitor recently active server administration discussions:

```json
{
  "site": "serverfault",
  "tagged": "nginx",
  "sort": "activity",
  "order": "desc",
  "pagesize": 10
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Stack Overflow and Ask Ubuntu questions
```

### Common Workflow Patterns

- **DevOps Question Alert:** Scheduled Trigger (Hourly) → StackExchange (`tagged: "kubernetes"`) → For-Each → Slack / Discord Webhook (Post new questions).
- **AI Solution Finder:** Webhook Trigger (Incoming Developer Question) → StackExchange → AI Chat (Synthesize answers from top questions) → Reply to User.
- **Community Engagement Dashboard:** Manual Trigger → StackExchange → Function (Filter by `is_answered === false`) → Log / Notification.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### API returns 400 Bad Request

**Cause:** The `site` parameter contains an invalid or misspelled site identifier (e.g. `stack-overflow` instead of `stackoverflow`).

**Solution:** Verify the site identifier against the official Stack Exchange site list. For main sites, use names like `stackoverflow`, `askubuntu`, `serverfault`, or `superuser`.

### Empty questions array returned (`total_results: 0`)

**Cause:** The specified tag combination does not exist, is misspelled, or has no matching questions for the chosen site.

**Solution:** Check the tag spelling on the target website (e.g., `reactjs` instead of `react.js`).

### Rate limiting / Quota exceeded

**Cause:** The Stack Exchange public API enforces daily quota limits per IP address (typically 300 requests per 24 hours for unauthenticated client requests).

**Solution:** Inspect the `quota.remaining` property in the output envelope. Increase intervals in scheduled workflows (e.g., run every hour instead of every minute) to avoid exhausting your quota.

### Negative or invalid pagesize

**Cause:** A negative number or 0 was provided for `pagesize`.

**Solution:** Set `pagesize` to a positive integer between `1` and `100`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) - Make custom API requests to any external service
- [GitHub](../github/en.md) - Search repositories, issues, and code on GitHub
- [AI Chat](../ai-chat/en.md) - Analyze and summarize developer questions with AI models

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for StackExchange node |

<!-- /SECTION: changelog -->

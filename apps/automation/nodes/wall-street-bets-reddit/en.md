---
node_id: "wall-street-bets-reddit"
title: "WallStreetBets Reddit"
description: "Retrieve the top 50 stocks discussed on Reddit's WallStreetBets subreddit, including comment counts and sentiment data, through the Tradestie API."
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - reddit
  - wallstreetbets
  - stocks
  - sentiment
  - finance
  - market-data
  - tradestie
related_nodes:
  - reddit-api
  - stocktwits
  - log
---

<!-- SECTION: header -->
# WallStreetBets Reddit

> **Category:** Business & Commerce | **Subcategory:** Finance & Accounting | **Type:** Action Node

Retrieve the top 50 stocks discussed on Reddit's WallStreetBets community, with discussion volume and sentiment information from the Tradestie API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **WallStreetBets Reddit** node retrieves Tradestie's latest analysis of stock tickers discussed on the `r/wallstreetbets` subreddit. The API returns up to 50 ranked stock records with comment counts, sentiment labels, and sentiment scores.

### Key Features

- **Top 50 Stocks:** Retrieve the most discussed stock tickers
- **Sentiment Data:** Return a sentiment label and numeric sentiment score for each ticker
- **Discussion Volume:** Include the number of comments associated with each ticker
- **Historical Date:** Optionally request data for a specific date
- **No Authentication Required:** The public Tradestie endpoint does not require an API key, access token, or secret
- **Error Routing:** Route request and API failures to the error output

### Use Cases

- Monitor stocks trending in WallStreetBets discussions
- Build social-investing and market-sentiment dashboards
- Enrich stock-analysis workflows with Reddit discussion signals
- Trigger notifications when discussion volume or sentiment changes
- Pass ticker sentiment data to downstream analysis or reporting nodes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

This node has no configurable parameters. It always requests the latest available WallStreetBets data.

### API Request

The node sends a `GET` request to:

```text
https://api.tradestie.com/v1/apps/reddit
```

To request a specific date, the API accepts an optional `date` query parameter in `MM-DD-YYYY` format:

```text
https://api.tradestie.com/v1/apps/reddit?date=11-02-2025
```

### API and Authentication

No API key, access token, bearer token, or other secret is required for this endpoint. Do not add credentials to the workflow configuration.

### Rate Limit and Freshness

- Rate limit: 20 requests per minute per IP address
- Data refresh: approximately every 15 minutes
- Historical availability depends on Tradestie's Reddit data collection

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data. The node does not require input parameters. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `array` | Up to 50 stock records returned by the Tradestie API |

### Success Output Example

```json
[
  {
    "no_of_comments": 179,
    "sentiment": "Bullish",
    "sentiment_score": 0.13,
    "ticker": "GME"
  },
  {
    "no_of_comments": 37,
    "sentiment": "Bullish",
    "sentiment_score": 0.159,
    "ticker": "AMC"
  }
]
```

### Error Output

Request failures, rate-limit responses, unavailable API responses, and malformed upstream data are routed to the error output.

```json
{
  "success": false,
  "error": "Tradestie Reddit API request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Retrieve Latest WallStreetBets Data

Use the node with no parameters to retrieve the latest available results.

```json
{}
```

### Retrieve Data for a Specific Date

The underlying API supports a date query parameter in `MM-DD-YYYY` format. The node implementation may expose this option through a future parameter update.

```text
GET https://api.tradestie.com/v1/apps/reddit?date=11-02-2025
```

### Analyze Sentiment Downstream

Connect the success output to a function, filter, log, or notification node to inspect `ticker`, `sentiment`, `sentiment_score`, and `no_of_comments`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve WallStreetBets stock sentiment
```

### Common Patterns

- **Scheduled Monitoring:** Cron → WallStreetBets Reddit → Log
- **Manual Inspection:** Manual Trigger → WallStreetBets Reddit → Log
- **Sentiment Filtering:** WallStreetBets Reddit → Filter → Notification
- **Market Dashboard:** WallStreetBets Reddit → Function → Database or Dashboard

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Request failed

**Cause:** The Tradestie endpoint is unavailable, the network request failed, or the API returned an error.

**Solution:** Verify network access and retry the workflow. Check the endpoint status and avoid exceeding the rate limit.

#### Rate limit exceeded

**Cause:** More than 20 requests were made from the same IP address within one minute.

**Solution:** Reduce request frequency, add a delay between requests, or use scheduled execution at a lower frequency.

#### Empty or incomplete response

**Cause:** Historical data may not be available for the requested date, or the upstream dataset may not contain enough records.

**Solution:** Try the latest data or another date and handle empty arrays in downstream nodes.

#### Authentication error

**Cause:** This public endpoint does not require authentication. An authentication error may indicate that the wrong endpoint is being used.

**Solution:** Use `https://api.tradestie.com/v1/apps/reddit` and remove API keys, bearer tokens, and custom credentials.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 429` | Rate limit exceeded | Reduce request frequency and retry later |
| `HTTP 4xx` | Invalid request or unavailable data | Verify the endpoint and optional date format |
| `HTTP 5xx` | Tradestie service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and endpoint availability |
| `Invalid response` | Unexpected response structure | Inspect the upstream response and handle missing fields |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Reddit API** - Search Reddit posts, comments, and subreddit content
- **StockTwits** - Work with stock-market social sentiment data
- **Log** - Inspect node output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-31 | Initial documentation |

<!-- /SECTION: changelog -->

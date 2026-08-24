---
node_id: "kanye-rest"
title: "Kanye Rest"
description: "Get a random Kanye West quote from the Kanye Rest API."
category: "Entertainment"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- kanye
- quotes
- entertainment
- fun
- api
- random

related_nodes:
- function
- if

---

# Kanye Rest

> **Category:** entertainment-nodes | **Type:** Action Node

Get a **random Kanye West quote** from the Kanye Rest API.

The **Kanye Rest** node calls the single-endpoint Kanye Rest API and returns one randomly generated quote per call. It takes no configuration.

### Supported Features

- Fetch a single random Kanye West quote
- No configuration required

### Use Cases

- Add a fun, random quote to a notification, digest, or dashboard
- Build a "quote of the day" bot
- Use as a lightweight demo/test node when building or testing a workflow
- Inject light entertainment content into a chatbot response

---

## Configuration

### Parameters

This node has **no configuration parameters**. It takes no input and requires no setup.

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| — | — | — | — | This node has no configurable parameters. |

---

## API Details

The node calls the following endpoint:

```text
GET https://api.kanye.rest/
```

No API key or authentication is required. Each call returns one random quote.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `quote` | `string` | The random Kanye West quote text. |

---

## Output Example

```json
{
  "success": true,
  "quote": "I feel like I'm the number one rock star on the planet."
}
```

---

## Configuration Examples

### Default (Only Option)

The node has no parameters, so it is always used with an empty configuration.

```json
{}
```

---

## Workflow Integration

### Sample Workflow: Get Quote → Function

```json
{
  "nodes": [
    {
      "id": "kanye-quote",
      "type": "kanye-rest",
      "config": {}
    },
    {
      "id": "format-quote",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Get Quote → Notification

```json
{
  "nodes": [
    {
      "id": "kanye-quote",
      "type": "kanye-rest",
      "config": {}
    },
    {
      "id": "send-quote",
      "type": "notification"
    }
  ]
}
```

### Common Patterns

- Schedule (daily) → Kanye Rest → Notification — quote-of-the-day bot
- Kanye Rest → Function → Chat/embed message formatting

---

## Error Handling

### API Error

```text
Kanye Rest API error: <status>
```

Raised when the API returns a non-OK HTTP status.

### Wrapped Failure

```text
Kanye Rest request failed: <underlying error message>
```

All errors, including the API error above, are re-thrown wrapped in this message from `handleTick`.

---

## Troubleshooting

### "Kanye Rest request failed: Kanye Rest API error: <status>"

**Cause**

The Kanye Rest API is temporarily unavailable or returned an unexpected status.

**Solution**

Retry later; this is a small community-run API with no formal uptime guarantee.

---

### "Kanye Rest request failed: ..." (Network Error)

**Cause**

The underlying fetch call failed entirely (network error, DNS failure, timeout).

**Solution**

Check network connectivity to `api.kanye.rest` and retry.

---

## Security

The node performs outbound HTTP requests to the public Kanye Rest API (`api.kanye.rest`).

No API key or authentication credential is required or accepted by the node.

---

## Notes

The node returns a single random quote per call, with no way to request more than one, filter by topic, or avoid repeats across calls.

The node does not:

- Support fetching multiple quotes in one call
- Support filtering or searching quotes
- Cache or deduplicate quotes across calls
- Guarantee the quote is authentic Kanye West material — quotes come from a community-maintained dataset

It is intended to provide a simple, low-effort source of fun content for downstream entertainment or demo workflows.

---


## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
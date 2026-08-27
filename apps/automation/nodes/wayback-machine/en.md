---
node_id: "wayback-machine"
title: "Wayback Machine"
description: "Check whether a webpage was archived and retrieve the closest snapshot for a specified date."
category: "Security & Networking"
subcategory: "Security Intelligence"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - wayback-machine
  - archive
  - internet-archive
  - web-history
  - url
  - reconnaissance
  - security
related_nodes:
  - http-request
  - url-encode
  - log
---

<!-- SECTION: header -->
# Wayback Machine

> **Category:** Security & Networking | **Subcategory:** Security Intelligence | **Type:** Action Node

Check whether a webpage was archived by the Internet Archive and retrieve the closest available snapshot for a requested date.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Wayback Machine** node queries the Internet Archive's Wayback availability service for a URL and optional timestamp. It helps determine whether an archived capture exists and provides a replay URL when one is available.

### Key Features

- **Historical Availability:** Check whether a URL existed in the archive near a target date
- **Closest Snapshot:** Retrieve the nearest available archived capture
- **Archive URL:** Return a link to replay the archived page
- **Date-Based Investigation:** Compare historical page availability with security or business events
- **Dynamic Input:** Use configured values or provide URL data through the `input` port
- **No Credentials Required:** The workflow example contains no API key, token, authorization header, or secret

### Use Cases

- Investigate historical changes to authorized websites
- Verify whether a page was publicly available at a particular time
- Support brand, content, and domain-history investigations
- Enrich incident timelines with archived web evidence
- Check historical pages before migration, takedown, or compliance review
- Identify older versions of public documentation or policy pages

> Use archived content responsibly. Confirm that collection, access, and reuse of a page are permitted, and remember that an archived snapshot may be incomplete or altered by replay behavior.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | `url` | Yes | — | Webpage URL to check, for example `https://www.google.com` |
| `timestamp` | `string` | No | Current/nearest available capture | Target date or timestamp, commonly formatted as `YYYYMMDD` or `YYYYMMDDhhmmss` |

The workflow example uses:

```json
{
  "url": "https://www.google.com",
  "timestamp": "20200101"
}
```

If `timestamp` is supplied, the service searches for the closest available capture around that date. If it is omitted, the service can return the closest available snapshot for the URL.

### API Behavior

The node uses the Internet Archive Wayback availability API:

```text
GET https://archive.org/wayback/available?url={url}&timestamp={timestamp}
```

No API key or authentication token is required by the workflow configuration.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | A URL string, or an object containing `url` and optional `timestamp`. Used when configured parameters are empty. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Wayback availability response with the requested URL and, when found, a closest archived snapshot |

Example response with an available snapshot:

```json
{
  "url": "https://www.google.com",
  "archived_snapshots": {
    "closest": {
      "status": "200",
      "available": true,
      "url": "https://web.archive.org/web/20200101000000/https://www.google.com/",
      "timestamp": "20200101000000"
    }
  },
  "timestamp": "20200101"
}
```

When no snapshot is found, the success response may contain an empty `archived_snapshots` object or indicate that no capture is available.

### Error Output

Missing or invalid URLs, malformed timestamps, network failures, and upstream service errors are routed to `error`.

```json
{
  "success": false,
  "error": "Wayback Machine lookup failed",
  "url": "not-a-url"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Check a Page on a Specific Date

```json
{
  "url": "https://www.google.com",
  "timestamp": "20200101"
}
```

### Check the Most Recent Available Capture

```json
{
  "url": "https://example.com"
}
```

Leave `timestamp` empty to let the availability service select an available capture without a specified historical date.

### Dynamic URL from a Previous Node

Pass a URL directly through `input`:

```text
https://example.com/security-policy
```

Or pass a URL and target date together:

```json
{
  "url": "https://example.com/security-policy",
  "timestamp": "20240115"
}
```

### Use the Snapshot URL Downstream

After a successful lookup, pass the returned `archived_snapshots.closest.url` to a Log, HTTP Request, or notification node for inspection or reporting.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Check a webpage's historical availability
```

### Common Patterns

- **Basic check:** Manual Trigger → Wayback Machine → Log
- **Incident timeline:** URL list → Wayback Machine → Function → Report
- **Historical comparison:** Wayback Machine → HTTP Request or parser
- **Evidence capture:** Wayback Machine → Log → Storage or notification

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### URL is required

**Cause:** Neither `url` nor a usable incoming `input` value was provided.

**Solution:** Configure a complete URL including its scheme, such as `https://example.com`.

### No archived snapshot found

**Cause:** The URL was not captured, the target date is outside the available archive history, or the exact URL differs from the archived version.

**Solution:** Try the site root, remove unnecessary query parameters, or omit `timestamp` to search for another available capture.

### Snapshot is incomplete

**Cause:** Web archives may not contain every image, stylesheet, script, or linked resource from the original page.

**Solution:** Treat the result as an archived representation, inspect the replay URL, and preserve the timestamp and original URL in your report.

### Invalid timestamp

**Cause:** The timestamp is not in a format accepted by the availability service.

**Solution:** Use `YYYYMMDD` for a date or `YYYYMMDDhhmmss` for a more precise timestamp.

### Request failed or timed out

**Cause:** The Internet Archive service is unavailable, rate-limited, or unreachable from the workflow runtime.

**Solution:** Retry with backoff, avoid high-volume bursts, and inspect the `error` output for upstream details.

### Access or authorization concern

**Cause:** The target content may be restricted, private, or outside the permitted investigation scope.

**Solution:** Limit lookups to public pages and assets you are authorized to investigate. The node does not bypass access controls.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) — Retrieve or inspect a returned snapshot URL
- [URL Encode](./url-encode.md) — Prepare URL values with special characters
- [Log](./log.md) — Inspect availability results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation |

<!-- /SECTION: changelog -->

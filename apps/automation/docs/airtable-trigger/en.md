---
node_id: "airtable-trigger"
title: "Airtable Trigger"
description: "Trigger workflow execution when records are created, updated, or deleted in an Airtable table — uses polling with change detection"
category: "triggers"
subcategory: "airtable"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - airtable
  - trigger
  - polling
  - change-detection
  - no-code
  - spreadsheet
related_nodes:
  - airtable-action
  - baserow-action
  - interval
  - cron
---

<!-- SECTION: header -->
# Airtable Trigger

> **Category:** Triggers | **Type:** Trigger Node

Poll an Airtable table at a configurable interval and trigger a workflow when records are created, updated, or deleted. Uses hash-based change detection to identify what changed between polls.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Airtable Trigger** node connects to the Airtable API and periodically fetches all records from a target table. It compares the current state of each record to its previously observed state using a hash of the record's fields. When a change is detected — a new record appears, a record's fields change, or a record disappears — the node emits a trigger output with the event type and record data.

The first poll establishes a baseline and does not fire any triggers. Subsequent polls compare against this baseline.

### Key Features

- **Four Event Types:** `new_record`, `updated_record`, `deleted_record`, `any_change`
- **Configurable Polling Interval:** Set the polling frequency from 1 second to 3600 seconds (1 hour)
- **Hash-Based Change Detection:** Detects field-level changes without requiring Airtable webhooks
- **Baseline on First Poll:** The first poll establishes the initial state — no false triggers on startup
- **Pause / Resume:** Stop and restart polling without losing connection state
- **Structured Output:** Each trigger includes `recordId`, `fields`, `eventType`, and `timestamp`

### Use Cases

- React to new form submissions added to an Airtable table
- Trigger notifications when a record's status field changes
- Sync updated Airtable records to a database or external API
- Detect and process deleted records for audit or cleanup workflows
- Build real-time-like integrations without Airtable webhook access

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | ✅ Yes | Airtable personal access token — generate from **airtable.com/create/tokens** |
| `baseId` | `string` | ✅ Yes | Airtable Base ID — found in the base URL: `airtable.com/{baseId}/{tableId}` |
| `tableName` | `string` | ✅ Yes | Name of the table to watch (must match exactly as shown in the UI) |

### Trigger Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `triggerOn` | `enum` | ✅ Yes | `new_record` | Which change type triggers the workflow (see below) |
| `pollingInterval` | `number` | ❌ No | `60` | How often to poll the table, in seconds (1–3600) |

### Trigger Event Types

| Value | Triggers When |
|-------|--------------|
| `new_record` | A record that did not exist in the previous poll is detected |
| `updated_record` | An existing record's fields have changed since the last poll |
| `deleted_record` | A record that existed in the previous poll no longer exists |
| `any_change` | Any of the above — new, updated, or deleted records |

> **Note on `deleted_record`:** Airtable does not provide a deletion event natively. Deleted records are detected by their absence from the polled results. Only records previously seen by the trigger can be detected as deleted.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it activates automatically on each polling interval. It does not accept inputs from other nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted for each detected change (one trigger per changed record) |
| `error` | `object` | Emitted on polling errors (API failure or network issue) |

### Output Schema (`success`)

**For `new_record` and `updated_record`:**
```json
{
  "recordId": "recABC123XYZ",
  "fields": {
    "Name": "Alice",
    "Status": "Active",
    "Score": 85,
    "Created At": "2026-03-10"
  },
  "createdTime": "2026-03-10T08:00:00.000Z",
  "eventType": "new_record",
  "timestamp": "2026-03-10T08:01:00.000Z"
}
```

**For `deleted_record`:**
```json
{
  "recordId": "recABC123XYZ",
  "eventType": "deleted_record",
  "timestamp": "2026-03-10T08:05:00.000Z"
}
```

> **Note:** For deleted records, only `recordId`, `eventType`, and `timestamp` are available — the fields are no longer accessible once the record is deleted.

| Field | Type | Description |
|-------|------|-------------|
| `recordId` | `string` | Airtable record ID (`recXXXXXXXXXXXXXX`) |
| `fields` | `object` | All field values of the record at the time of detection |
| `createdTime` | `string` | ISO timestamp when the record was originally created in Airtable |
| `eventType` | `string` | One of `new_record`, `updated_record`, `deleted_record` |
| `timestamp` | `string` | ISO timestamp when the change was detected by the node |

### Output Schema (`error`)

```json
{
  "type": "airtable_error",
  "statusCode": 429,
  "message": "Rate limit exceeded",
  "timestamp": "2026-03-10T08:01:00.000Z"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Trigger on New Records

Fire a workflow whenever a new row is added to the table.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Leads",
  "triggerOn": "new_record",
  "pollingInterval": 60
}
```

**Output (on new record detected):**
```json
{
  "recordId": "recNEW456ABC",
  "fields": {
    "Name": "David",
    "Email": "david@example.com",
    "Status": "New",
    "Score": 72
  },
  "createdTime": "2026-03-10T10:30:00.000Z",
  "eventType": "new_record",
  "timestamp": "2026-03-10T10:31:00.000Z"
}
```

---

### Example: Trigger on Any Change

React to any modification, creation, or deletion in the table.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Orders",
  "triggerOn": "any_change",
  "pollingInterval": 30
}
```

> **Tip:** Use a Function node after this trigger to branch on `input.eventType`:
> ```js
> switch (input.eventType) {
>   case 'new_record': // Handle new order
>   case 'updated_record': // Handle order update
>   case 'deleted_record': // Handle deletion
> }
> ```

---

### Example: Near Real-Time Polling

Poll every 5 seconds for time-sensitive updates.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Support Tickets",
  "triggerOn": "updated_record",
  "pollingInterval": 5
}
```

> **Important:** Very short polling intervals will generate more API requests. Airtable rate-limits to 5 requests per second per base. Set `pollingInterval` to at least `5` to avoid `429` errors on large tables.

---

### Example: Detect Deleted Records

Trigger a cleanup workflow when a record is removed.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Subscriptions",
  "triggerOn": "deleted_record",
  "pollingInterval": 120
}
```

**Output:**
```json
{
  "recordId": "recABC123XYZ",
  "eventType": "deleted_record",
  "timestamp": "2026-03-10T12:00:00.000Z"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: New Lead → Send Welcome Email

Trigger a welcome email workflow whenever a new lead is added to an Airtable table.

```json
{
  "nodes": [
    {
      "id": "airtable-watcher",
      "type": "airtable-trigger",
      "position": { "x": 100, "y": 100 },
      "config": {
        "apiKey": "patXXXXXXXXXXXXXX",
        "baseId": "appXXXXXXXXXXXXXX",
        "tableName": "Leads",
        "triggerOn": "new_record",
        "pollingInterval": 60
      }
    },
    {
      "id": "send-email",
      "type": "email-send",
      "position": { "x": 300, "y": 100 },
      "config": {
        "to": "{{input.fields.Email}}",
        "subject": "Welcome, {{input.fields.Name}}!",
        "body": "Thanks for signing up. We'll be in touch soon."
      }
    },
    {
      "id": "update-status",
      "type": "airtable-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "apiKey": "patXXXXXXXXXXXXXX",
        "baseId": "appXXXXXXXXXXXXXX",
        "tableName": "Leads",
        "operation": "Update data in table",
        "recordId": "{{input.recordId}}",
        "recordData": [
          { "fieldName": "Status", "fieldValue": "Contacted" }
        ]
      }
    }
  ]
}
```

### Common Patterns

- **New Record → Action:** Airtable Trigger (`new_record`) → Process → External API
- **Status Change → Notification:** Airtable Trigger (`updated_record`) → Check `fields.Status` → Send alert
- **Record Deleted → Cleanup:** Airtable Trigger (`deleted_record`) → Delete from DB by `recordId`
- **Any Change → Sync:** Airtable Trigger (`any_change`) → Branch on `eventType` → Sync to DB

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Trigger fires for all records on startup

**Cause:** This should not happen — the first poll establishes a baseline and does not emit triggers.

**Solution:** If this occurs, check that the node was not restarted mid-run with a dirty in-memory state. Restarting the node resets the baseline.

#### Updated records not detected

**Cause:** The change occurred between two polls and was then reverted before the next poll, or the field that changed is not included in the polled record hash.

**Solution:** Decrease `pollingInterval` to reduce the detection window. Note that all fields are included in the hash — including hidden fields in the Airtable view.

#### `429 Rate limit exceeded` error

**Cause:** The polling interval is too short for the number of records in the table, causing too many API requests per second.

**Solution:** Increase `pollingInterval`. For large tables (>1000 records), use a minimum interval of 30–60 seconds. Airtable allows 5 requests per second per base.

#### `deleted_record` event not emitted

**Cause:** The trigger node was restarted after the deletion — the baseline was reset and the deleted record was never seen.

**Solution:** This is a known limitation of poll-based change detection. Deleted records can only be detected if the node saw them in a previous poll within the same session.

#### `airtable_error: 404 — Table not found`

**Cause:** `tableName` does not match any table in the specified base, or `baseId` is incorrect.

**Solution:** Verify `baseId` and `tableName` exactly as they appear in Airtable. Both are case-sensitive.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `401` | Unauthorized | Check API token and scopes |
| `404` | Not found | Verify `baseId` and `tableName` |
| `429` | Rate limited | Increase `pollingInterval` |
| `ECONNREFUSED` | Connection refused | Check network access to `api.airtable.com` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Airtable Action](./airtable-action.md) - Perform CRUD operations on Airtable records
- [Baserow Action](./baserow.md) - Similar operations on self-hosted Baserow instances
- [Interval](./interval.md) - Fixed-interval trigger for custom polling workflows
- [Cron](./cron.md) - Schedule-based trigger

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->

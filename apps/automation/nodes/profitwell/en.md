---
node_id: "profitwell"
title: "ProfitWell"
description: "Read daily metrics, plans, user subscription history, and monthly customer churn."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-29"
author: "Fusion Team"
tags: [integration, profitwell, subscriptions, churn]
related_nodes: []
---

<!-- SECTION: overview -->
# ProfitWell

Read daily financial metrics, manually-added plans, one manually-added user's subscription history, or a monthly customer churn percentage. This Action Node makes one GET request per execution and displays its running status.

This page describes the supplied implementation, without independently verifying API availability.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | string | Yes | None | Nonblank API key, trimmed before use. Supports expressions. |
| `operation` | enum | No | `getFinancialMetrics` | `getFinancialMetrics`, `getPlans`, `getSubscriptions`, or `getChurnRate`. |
| `month` | string | For metrics and churn | None | Exact YYYY-MM format, month 01 through 12. Supports expressions. |
| `planId` | string | No | Unset | Plan filter for metrics and churn. Trimmed; omitted when blank. Supports expressions. |
| `userId` | string | For subscriptions | None | ProfitWell user ID or alias. Supports expressions. |

The schema attaches conditional dependencies to month and plan for metric operations, and user ID for subscriptions. The handler validates fields used by the selected operation.

The intended hardcoded base URL is `https://api.profitwell.com/v2`. If the pasted Markdown link syntax is present literally in `BASE`, replace it with this plain URL. The URL is not configurable.

Every request uses GET, no body, `Accept: application/json`, and `Authorization` containing the trimmed key. No Bearer prefix is added. Keep credentials out of exported examples.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

| Operation | Endpoint | Behavior |
|-----------|----------|----------|
| `getFinancialMetrics` | `/metrics/daily/` | Sends month and optional plan_id; returns parsed JSON unchanged. |
| `getPlans` | `/plans/` | Returns manually-added plans as parsed API JSON. |
| `getSubscriptions` | `/users/<encoded-userId>/` | Returns one manually-added user's subscription history. |
| `getChurnRate` | `/metrics/monthly/` | Sends metrics=customers_churn_rate and optional plan_id, then selects one month locally. |

### Daily metrics

The supplied code documents support for the current or previous month only. The API enforces this window using its own clock; local validation checks only the month format. Some daily metrics may be unavailable per plan.

Month values are not trimmed: `2026-09` passes; surrounding whitespace, `2026-9`, and `2026-13` fail. Query parameters are encoded using URLSearchParams.

### Plans and subscriptions

Plans ignore month, plan, and user settings. Subscription history is for one manually-added user, not every subscription in the account. The user ID or alias is trimmed and URL-encoded; blank values, `.`, and `..` are rejected.

### Monthly customer churn

The monthly request does not send month, start_date, or end_date. The node selects the first entry whose date exactly matches the configured month.

The value must be a finite number or null. Numbers are already percentages and are not multiplied by 100. Null means undefined, not zero. A missing month raises an error.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Direction | Description |
|------|-----------|-------------|
| `input` | Input | Starts execution; incomingData is accepted but unused. |
| `success` | Output | Parsed API JSON or the normalized churn record. |
| `error` | Output | Error path for failed execution. |

Set configuration directly or through expression-enabled fields. Incoming objects are not automatically read for configuration values.

Daily metrics, plans, and subscriptions return the entire parsed response without extracting data, adding an envelope, or validating its structure. Inspect actual responses before mapping fields.

Illustrative churn output, not a live result:

```json
{
  "month": "2026-08",
  "metric": "customers_churn_rate",
  "value": 2.5,
  "unit": "percent"
}
```

If the matching history entry has a null value, value is null and the other fields are unchanged. The output does not include planId.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Examples

Keys and identifiers below are placeholders.

### Daily metrics

```json
{
  "apiKey": "<your-api-key>",
  "operation": "getFinancialMetrics",
  "month": "2026-09"
}
```

Choose the current or previous month at execution time. Add planId to filter by plan.

### Manually-added plans

```json
{
  "apiKey": "<your-api-key>",
  "operation": "getPlans"
}
```

### One user's subscription history

```json
{
  "apiKey": "<your-api-key>",
  "operation": "getSubscriptions",
  "userId": "example-user-alias"
}
```

### Monthly churn by plan

```json
{
  "apiKey": "<your-api-key>",
  "operation": "getChurnRate",
  "month": "2026-08",
  "planId": "example-plan"
}
```

Use identifiers from your account. Omit planId to send no plan filter. The month must exist in the returned history.
<!-- /SECTION: examples -->

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve ProfitWell metrics and inspect the response
```

Connect **Manual Trigger -> ProfitWell -> Log**. Set a private API key, select getFinancialMetrics, and configure a supported month. Run the trigger and inspect the result in Log. For churn reporting, select getChurnRate and consume month, value, and unit downstream.

The existing preview contains only wiring; configure parameters and credentials before running. Connect error to an error-handling branch as needed.
<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error | Explanation and action |
|-------|------------------------|
| `apiKey is required` | Supply a nonblank string key. |
| `month is required in YYYY-MM format (01 through 12)` | Use four year digits and two month digits without surrounding whitespace. |
| `planId must be a string` | Supply a string identifier. This runtime guard supplements schema validation. |
| `userId (ProfitWell user_id or user_alias) is required` | Provide a nonblank identifier other than `.` or `..`. |
| `ProfitWell error (<status>)` | A non-2xx response was received. Check status, credentials, identifiers, and permitted month window. |
| `ProfitWell returned invalid JSON (<status>)` | A successful HTTP response could not be parsed as JSON. |
| `ProfitWell response is missing customers_churn_rate` | No array was found at data.customers_churn_rate. |
| `No customer churn rate available for <month>` | No history entry matched the requested month exactly. |
| `ProfitWell returned an invalid customers_churn_rate value` | The selected value was neither a finite number nor null. |
| `Unknown operation: <operation>` | Choose one of the four supported operations. |

For malformed URL errors, check that BASE contains the plain URL. Network errors propagate without a custom wrapper. If input values appear ignored, set configuration or expressions explicitly.

The helper checks HTTP status before parsing JSON and deliberately omits remote error bodies, which could contain credentials or customer data. Successful responses have no application-level success check apart from churn-specific validation.

No automatic retries, explicit timeout, or pagination loop are implemented. stop() is a no-op and does not cancel an in-flight request.
<!-- /SECTION: troubleshooting -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-29 | Regenerated documentation for all four operations, churn handling, validation, and examples. |
<!-- /SECTION: changelog -->

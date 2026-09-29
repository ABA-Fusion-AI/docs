---
node_id: "splunk"
title: "Splunk"
description: "Submit searches, retrieve results, manage indexes, ingest data, and dispatch saved searches."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-29"
author: "Fusion Team"
tags: [integration, splunk, search, indexes, logs]
related_nodes: []
---

<!-- SECTION: overview -->
# Splunk

> **Category:** Peer-only Integrations | **Type:** Action Node

Submit search jobs, retrieve their results, list or create indexes, ingest raw data, and list or dispatch saved searches. Each execution makes one HTTP request and displays its running status.

This page describes the supplied SplunkActionNode implementation. Available endpoints and permissions depend on your Splunk deployment; they have not been verified through live requests.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | string | By schema | None | Splunk REST base address including scheme and any required port, for example https://splunk.example.com:8089. |
| `token` | string | One authentication method | Unset | Sent as a Bearer token when truthy. |
| `username` | string | With password if no token | Unset | Basic authentication username. |
| `password` | string | With username if no token | Unset | Basic authentication password. |
| `operation` | enum | No | `runSearch` | One of the seven operations below. |
| `searchQuery` | string | For runSearch | Unset | Complete search expression, sent unchanged. |
| `earliestTime` | string | No | Unset | Search start bound for runSearch and runSavedSearch. |
| `latestTime` | string | No | Unset | Search end bound for runSearch and runSavedSearch. |
| `sid` | string | For getSearchResults | Unset | Search job identifier. |
| `limit` | number | No | 100 | Request count for getSearchResults and listSavedSearches only. |
| `indexName` | string | For createIndex | Unset | Name of the new index, or target index for addData. |
| `datatype` | string | No | Unset | Optional createIndex datatype, forwarded unchanged. |
| `rawData` | string | No | Unset | Raw text sent by addData; omitted values become an empty body. |
| `sourcetype` | string | No | Unset | addData source type; omitted values use generic. |
| `searchName` | string | For runSavedSearch | Unset | Saved search name. |

All string fields support expressions. Operation and limit do not declare expression metadata. No operation-dependent field visibility is declared.

The schema does not enforce a nonempty host, positive/integer limit, time syntax, or an allowed datatype list. Required operation parameters are checked for truthiness at runtime; whitespace-only strings are not trimmed or rejected.

### Host and authentication

The request helper appends /services followed by the operation path to host. Configure the REST base address without a trailing /services segment.

**Source formatting caveat:** The pasted regular expression in host.replace is malformed. The intended trailing-slash removal should be valid TypeScript, for example:

```typescript
const h = this.config.host.replace(/\/$/, "");
```

This removes one final slash; it is not general URL normalization. Correct the source if it contains the pasted formatting literally.

Authentication precedence:

1. A truthy token produces Authorization: Bearer followed by the token.
2. Otherwise, truthy username and password produce Basic authentication using base64-encoded username:password.
3. Otherwise, execution throws an authentication error.

Values are not trimmed. A configured token takes precedence over username/password, and a failed token request does not fall back to Basic authentication. Keep credentials out of exported examples.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

Paths below are appended after /services.

| Operation | Method | Path | Request details |
|-----------|--------|------|-----------------|
| `runSearch` | POST | /search/jobs | Form fields search, output_mode=json, and optional earliest_time/latest_time. |
| `getSearchResults` | GET | /search/jobs/<sid>/results | Query fields output_mode=json and count=limit. |
| `listIndexes` | GET | /data/indexes | Query field output_mode=json. |
| `createIndex` | POST | /data/indexes | Form field name and optional datatype. |
| `addData` | POST | /receivers/simple | Query fields sourcetype and index; raw text body. |
| `listSavedSearches` | GET | /saved/searches | Query fields output_mode=json and count=limit. |
| `runSavedSearch` | POST | /saved/searches/<encoded-name>/dispatch | Optional form fields dispatch.earliest_time and dispatch.latest_time. |

Requests use Content-Type: application/x-www-form-urlencoded except addData, which uses text/plain. The header is set for GET requests too.

### Search jobs

runSearch submits the configured query unchanged; it does not add a search prefix, wait for job completion, or fetch results. Use getSearchResults in a later execution with the returned job identifier configured in sid. The node does not implement job-status polling or cancellation.

The sid is inserted directly into the path without URL encoding. limit defaults to 100 and is forwarded as count; it does not affect runSearch or listIndexes. There is no offset setting or automatic pagination.

### Indexes and data ingestion

createIndex requires indexName and forwards datatype only when truthy.

addData sends rawData as text, defaulting to an empty string. Its index defaults to main and sourcetype to generic only when the corresponding configuration is null or undefined. Explicit empty strings remain empty. Both query values are URL-encoded.

Ingestion uses /receivers/simple, not an HTTP Event Collector endpoint. No HEC-specific payload, authentication scheme, or batching is implemented.

### Saved searches

listSavedSearches requests up to the configured count according to service behavior. runSavedSearch URL-encodes searchName and optionally forwards time bounds as dispatch fields. Dispatch does not wait for completion or retrieve results.

createIndex, addData, and runSavedSearch do not request output_mode=json. Their successful responses may therefore use the text fallback described below.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Direction | Description |
|------|-----------|-------------|
| `input` | Input | Starts execution. The incoming payload is unused. |
| `success` | Output | Parsed response JSON, or an object containing the response text. |
| `error` | Output | Error path for failed execution. |

Configure values directly or through expression-enabled fields. Incoming data is not automatically mapped to a query, sid, or ingestion body.

The helper reads the complete response as text. For a non-2xx status, it throws Splunk error (<status>): followed by the remote text. For a successful status, it attempts JSON.parse and returns the parsed value unchanged.

If successful text is not JSON, it returns:

```json
{
  "result": "<response text>"
}
```

An empty successful response returns {"result": ""}; there is no special HTTP 204 handling. No response schema or application-level success flag is checked. Search results are not extracted from the response envelope.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Examples

Connection values and credentials below are placeholders.

### Submit a search

```json
{
  "host": "https://splunk.example.com:8089",
  "token": "<splunk-token>",
  "operation": "runSearch",
  "searchQuery": "search index=main | head 10",
  "earliestTime": "-1h",
  "latestTime": "now"
}
```

Inspect the submission response for the job identifier. Completion is not awaited by this operation.

### Retrieve search results

```json
{
  "host": "https://splunk.example.com:8089",
  "token": "<splunk-token>",
  "operation": "getSearchResults",
  "sid": "<job-id-from-submission>",
  "limit": 100
}
```

Run this after the job is ready, with sid configured explicitly or through an expression.

### List indexes using Basic authentication

```json
{
  "host": "https://splunk.example.com:8089",
  "username": "<username>",
  "password": "<password>",
  "operation": "listIndexes"
}
```

Omit token to use this authentication method.

### Create an index

```json
{
  "host": "https://splunk.example.com:8089",
  "token": "<splunk-token>",
  "operation": "createIndex",
  "indexName": "workflow_events"
}
```

Add datatype only when appropriate for your deployment. The node does not validate allowed values.

### Add raw data

```json
{
  "host": "https://splunk.example.com:8089",
  "token": "<splunk-token>",
  "operation": "addData",
  "indexName": "workflow_events",
  "sourcetype": "generic",
  "rawData": "workflow=example status=completed"
}
```

The text is sent as-is. Repeated executions submit it again; no deduplication is implemented.

### List saved searches

```json
{
  "host": "https://splunk.example.com:8089",
  "token": "<splunk-token>",
  "operation": "listSavedSearches",
  "limit": 50
}
```

### Dispatch a saved search

```json
{
  "host": "https://splunk.example.com:8089",
  "token": "<splunk-token>",
  "operation": "runSavedSearch",
  "searchName": "Example saved search",
  "earliestTime": "-24h",
  "latestTime": "now"
}
```

Use a saved search accessible to the configured account. The dispatch response is returned as parsed JSON or response text.
<!-- /SECTION: examples -->

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Submit a Splunk search and inspect the response
```

Connect **Manual Trigger -> Splunk -> Log**. Configure host, private authentication, operation runSearch, and searchQuery. Run the trigger and inspect the submission response in Log.

For results, arrange a later getSearchResults execution with the actual sid once the job is ready. Connecting the response directly to another Splunk input does not populate sid automatically.

The existing preview contains wiring only; configure parameters before running. Connect error to an error-handling branch if needed.
<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Explanation and action |
|------------------|------------------------|
| `Either token or username+password is required` | Supply a truthy token or both username and password. |
| `searchQuery is required` | Configure a nonempty query for runSearch. |
| `sid is required` | Supply the search job identifier for result retrieval. |
| `indexName is required` | Supply an index name for createIndex. |
| `searchName is required` | Supply the saved search name for dispatch. |
| `Splunk error (<status>): <text>` | Review the server status/body, credentials, permissions, endpoint, and request values. |
| `Unknown operation: <operation>` | Choose one of the seven supported operations. |
| A response appears under result instead of JSON fields | The successful body was not valid JSON; the node preserved it as text. |
| Search submission contains no final results | Submission and result retrieval are separate; the node does not wait or poll. |
| Incoming values are ignored | Configure expressions explicitly; the incoming payload is unused. |
| URL or source syntax failure | Verify host and correct the malformed trailing-slash expression if copied literally. |

Network failures propagate without a custom wrapper. The node defines no explicit timeout, automatic retry, rate-limit handling, or custom TLS settings. stop() is a no-op and does not abort requests or cancel search jobs.

Remote response text is included in HTTP errors; handle logs with awareness of any data the service returns.
<!-- /SECTION: troubleshooting -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-29 | Regenerated documentation covering authentication, seven operations, response handling, and examples. |
<!-- /SECTION: changelog -->

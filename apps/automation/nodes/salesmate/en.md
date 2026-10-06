---
node_id: "salesmate"
title: "Salesmate"
description: "Manage contacts, companies, deals and activities in Salesmate CRM"
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags: [salesmate, crm, contacts, companies, deals, activities, integration, peer-only]
related_nodes: [http-request, function]
---

<!-- SECTION: overview -->
# Salesmate

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Use Salesmate's v4 API to retrieve contacts, companies, deals, and activities, or create contacts, companies, and deals. The node uses a workspace-specific HTTPS URL and a session token.

### Use Cases

- Create CRM contacts and companies from workflow events.
- Create sales deals linked to an owner, contact, pipeline, and stage.
- Retrieve CRM records in pages for reporting or downstream processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | `string` | All operations | None | Workspace name, Salesmate hostname, or HTTPS workspace URL. Supports expressions. |
| `sessionToken` | `string` | All operations | None | Non-blank session token sent in the `accessToken` header. Supports expressions. |
| `accessKey` | `string` | No | None | Legacy field; not used to authenticate v4 requests. Included in error-text redaction. Supports expressions. |
| `operation` | `enum` | No | `getContacts` | One of the seven operations listed below. |
| `firstName` | `string` | No | None | Optional first name for `createContact`. Supports expressions. |
| `lastName` | `string` | `createContact` | None | Non-blank contact last name. Supports expressions. |
| `email` | `string` | No | None | Optional email for `createContact`. Supports expressions. |
| `companyName` | `string` | `createCompany` | None | Non-blank company name. Supports expressions. |
| `dealTitle` | `string` | `createDeal` | None | Non-blank deal title. Supports expressions. |
| `dealValue` | `number` | No | None | Optional finite deal value; zero is included when supplied. |
| `ownerId` | `string` | All create operations | None | Positive safe-integer ID encoded as a digit-only string. Supports expressions. |
| `primaryContactId` | `string` | `createDeal` | None | Positive safe-integer contact ID encoded as a digit-only string. Supports expressions. |
| `pipeline` | `string` | `createDeal`, unless legacy fallback supplied | None | Existing pipeline name, rather than its numeric ID. Supports expressions. |
| `pipelineId` | `string` | No | None | Legacy fallback for `pipeline`; must contain the pipeline name. Supports expressions. |
| `stage` | `string` | `createDeal` | None | Existing stage name in the selected pipeline. Supports expressions. |
| `rows` | `number` | No | `250` | Records per search page; integer from 1 to 250. |
| `from` | `number` | No | `0` | Search offset; non-negative safe integer. |
| `timeoutMs` | `number` | No | `30000` | Request timeout; integer from 1 to 300000 milliseconds. |

All fields except `domain` and `sessionToken` are optional in the schema; the handler enforces operation-specific requirements at runtime. No conditional field visibility is declared. The numeric parameters and `operation` have no expression metadata.

### Workspace and Authentication

For a workspace named `mze`, all of these domain values resolve to `https://mze.salesmate.io/apis`:

- `mze`
- `mze.salesmate.io`
- `https://mze.salesmate.io`

The node trims and lowercases the domain and accepts one trailing slash. The workspace name must be 1–63 letters, digits, or hyphens, with a letter or digit at each end. HTTP URLs, paths, queries, credentials, ports, and unrelated hostnames are rejected.

All operations use POST with these headers:

```text
accessToken: <trimmed sessionToken>
x-linkname: <workspace>.salesmate.io
Content-Type: application/json
Accept: application/json
```

The node rejects redirects and applies the configured timeout to both fetching and reading the response body.

### Operations

Endpoints below are relative to `https://<workspace>.salesmate.io/apis`.

| Operation | POST endpoint | Required operation fields |
|-----------|---------------|---------------------------|
| `getContacts` | `/contact/v4/search?rows=<rows>&from=<from>` | None |
| `createContact` | `/contact/v4` | `lastName`, `ownerId` |
| `getCompanies` | `/company/v4/search?rows=<rows>&from=<from>` | None |
| `createCompany` | `/company/v4` | `companyName`, `ownerId` |
| `getDeals` | `/deal/v4/search?rows=<rows>&from=<from>` | None |
| `createDeal` | `/deal/v4` | `dealTitle`, `ownerId`, `primaryContactId`, pipeline name, `stage` |
| `getActivities` | `/activity/v4/search?rows=<rows>&from=<from>&viewType=list` | None |

### Search Behavior and Pagination

Search requests select each record's ID, name or title, and creation date:

| Record type | Requested fields | Module ID |
|-------------|------------------|-----------|
| Contact | `contact.id`, `contact.name`, `contact.createdAt` | 1 |
| Company | `company.id`, `company.name`, `company.createdAt` | 5 |
| Deal | `deal.id`, `deal.title`, `deal.createdAt` | 4 |
| Activity | `activity.id`, `activity.title`, `activity.createdAt` | 2 |

Every search uses a fixed creation-date filter: `IS_AFTER` with the value `Jan 01, 1970 05:30 AM`. It requests record counts with `getRecordsCount: true` and sends empty sort settings. Activity filters use the module name `Task`; deal searches send an empty pipeline field. Configured `pipeline` and `stage` values are used for deal creation, not search filtering.

Each execution retrieves one page. For example, with `rows: 100`, use `from: 0` for the first page and `from: 100` for the next. There is no automatic pagination, retry, or configurable search filter.

### Creation Behavior

- **Contact:** Sends trimmed `lastName` and numeric `owner`. Includes trimmed `firstName` and `email` only when non-blank.
- **Company:** Sends trimmed `companyName` as `name` and numeric `owner`.
- **Deal:** Sends trimmed title, pipeline name, and stage, plus numeric `owner` and `primaryContact`. The status is always `Open`. Includes `dealValue` when supplied and finite; no non-negative constraint is imposed locally.

IDs are trimmed, checked for digits only and a value greater than zero, then converted to safe integers. The handler does not check whether the referenced records, pipeline, or stage exist; the API validates the submitted values. Email format is not validated locally.

The pipeline value is selected with `pipeline ?? pipelineId`. An empty or whitespace-only `pipeline` prevents legacy fallback and fails validation. Prefer `pipeline` for new configurations.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Incoming data has type `unknown`. It triggers execution but is not read directly by the handler; operation parameters come from node configuration.
- **Success:** Parsed API JSON is returned directly without wrapping or transformation. An empty or whitespace-only successful response returns `null`.
- **Error:** Configuration validation failures, HTTP failures, application errors, unexpected response formats, or network failures.

For JSON responses, the node also detects application errors when `Status` or `status` is `failure`, `failed`, or `error` (case-insensitive), when `success` or `Success` is `false`, or when `Error` or `error` is truthy. This check applies even if the HTTP status indicates success.

Error messages include HTTP and sanitized URL context when available. Response excerpts are redacted, whitespace-normalized, and limited to 300 characters. Successful response data is returned as supplied by the API and is not redacted.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

These JSON objects represent node parameters. Replace the token placeholder with a protected value or expression and replace example IDs and names with values from your workspace.

### Retrieve Contacts

Omitting `operation` selects `getContacts`.

```json
{
  "domain": "mze",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "rows": 100,
  "from": 0
}
```

### Create a Contact

```json
{
  "domain": "mze.salesmate.io",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "operation": "createContact",
  "lastName": "Smith",
  "firstName": "Alex",
  "email": "alex@example.com",
  "ownerId": "123"
}
```

### Retrieve Companies

```json
{
  "domain": "mze",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "operation": "getCompanies",
  "rows": 100,
  "from": 100
}
```

### Create a Company

```json
{
  "domain": "mze",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "operation": "createCompany",
  "companyName": "Example Company",
  "ownerId": "123"
}
```

### Retrieve Deals

```json
{
  "domain": "mze",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "operation": "getDeals"
}
```

### Create a Deal

```json
{
  "domain": "https://mze.salesmate.io",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "operation": "createDeal",
  "dealTitle": "Example sales opportunity",
  "ownerId": "123",
  "primaryContactId": "456",
  "pipeline": "Sales",
  "stage": "Qualified",
  "dealValue": 1500
}
```

### Retrieve Activities

```json
{
  "domain": "mze",
  "sessionToken": "YOUR_SESSION_TOKEN",
  "operation": "getActivities",
  "rows": 50,
  "from": 0,
  "timeoutMs": 60000
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Salesmate in a workflow
```

### Common Patterns

- Incoming lead → Salesmate (`createContact`) → downstream processing.
- Company event → Salesmate (`createCompany`).
- Sales opportunity → Salesmate (`createDeal`) with an existing owner and contact.
- Scheduled trigger → Salesmate search operation → reporting.

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Cause | Resolution |
|------------------|-------|------------|
| `Salesmate: <field> is required for <operation>.` | A required string is missing or blank. | Supply workspace credentials and the selected operation's required fields. |
| `Salesmate: <field> must be a positive safe integer.` | An owner or contact ID is not a positive digit-only safe integer. | Supply an existing numeric ID as a string, such as `"123"`. |
| Domain validation error | The workspace value contains an unsupported hostname, scheme, path, query, credentials, or port. | Use a workspace name, its Salesmate hostname, or its HTTPS URL. |
| `Salesmate: rows must be 1–250 and from must be a non-negative integer.` | Invalid pagination values. | Use an integer from 1 to 250 for `rows` and a non-negative safe integer for `from`. |
| `Salesmate: timeoutMs must be an integer between 1 and 300000.` | Timeout is outside the allowed range or is not an integer. | Set a timeout within the documented range. |
| `Salesmate: dealValue must be a finite number.` | Deal value is not numeric or is infinite/NaN. | Supply a finite number or omit the value. |
| `Salesmate: HTTP <status>; URL <url>; <excerpt>` | The API returned a non-success HTTP status. | Read the redacted excerpt and check credentials, permissions, and submitted values. |
| `Salesmate: application error; ...` | The JSON response reports an error despite a successful HTTP status. | Check the returned error details, referenced IDs, and pipeline/stage names. |
| `Salesmate: unexpected HTML response; ...` | A successful HTTP response contains HTML. | Verify the workspace and service response. Redirects are rejected. |
| `Salesmate: unexpected text/invalid JSON response; ...` | A non-empty successful response is not valid JSON. | Check the returned excerpt and API response format. |

Network errors distinguish timeouts, DNS failures, TLS failures, connection failures, and other request/redirect/response-read failures. These errors include available HTTP status, a sanitized URL, and `response incomplete`. A timeout is reported as `timeout after <timeoutMs> ms`.

For deal creation, supply the pipeline's name, not its numeric ID, even when using the legacy `pipelineId` field. The schema restricts supported operations; an unsupported value reaching the handler raises `Unknown operation: <operation>`.

The `stop()` method performs no cleanup and does not cancel an active request. Request cancellation is handled by the configured timeout.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.

Requests use HTTPS, reject redirects, and authenticate with the trimmed session token. The legacy `accessKey` does not provide authentication.

Error text redacts configured session-token and access-key values, including trimmed, URL-encoded, and JSON-escaped variants, plus values following recognized authentication field names. Reported URLs remove credentials, query parameters, and fragments. Redaction is limited to diagnostics; handle successful CRM responses and other personal data appropriately in downstream workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) – Make custom CRM requests beyond this node's supported operations.
- [Function](../function/en.md) – Prepare record fields or process CRM results.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Regenerated documentation from the supplied v4 implementation, including validation, pagination, timeouts, and diagnostic redaction. |

<!-- /SECTION: changelog -->

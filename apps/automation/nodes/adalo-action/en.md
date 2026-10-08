---
node_id: "adalo-action"
title: "Adalo"
description: "Manage records in Adalo no-code app collections via the Adalo REST API with full support for CRUD operations, pagination, and dynamic expression mapping."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-08"
author: "Fusion Team"
tags:
  - adalo
  - no-code
  - database
  - crud
  - rest-api
  - records
  - collections
  - integration
related_nodes:
  - function
  - manual-trigger
  - webhook
  - log
  - if
---

<!-- SECTION: overview -->
# Adalo

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage records in **Adalo** no-code app collections via the Adalo REST API.

The **Adalo** node enables automated data workflows between Adalo apps and external services. It provides complete CRUD (Create, Read, Update, Delete) support for any collection in your Adalo apps, allowing you to list records with pagination, fetch specific records by ID, create new records with custom JSON payloads, update existing entries, and delete obsolete data.

### Key Features

- **Complete CRUD Suite:** Supports `listRecords`, `getRecord`, `createRecord`, `updateRecord`, and `deleteRecord`.
- **Flexible Field Mapping:** Pass record fields as JSON strings to create or modify custom schema attributes dynamically.
- **Configurable Pagination:** Query record collections with custom `limit` parameters to control payload size.
- **Dynamic Expressions:** Every parameter supports dynamic data binding (`expression: true`) to seamlessly pass outputs from triggers and upstream nodes.
- **Secure URL Encoding:** Automatically applies `encodeURIComponent` to app IDs, collection IDs, and record IDs to prevent request failures.
- **JSON Syntax Validation:** Validates JSON syntax in `fields` payloads before submission to prevent malformed API requests.

### Prerequisites

To use this node, you need the following details from your Adalo app:

1. **App ID (`appId`)**:
   * Found directly in your browser's address bar when editing your app:
   * `https://app.adalo.com/apps/<appId>/screens`
   * Example: `3c3bd2cb-c2e5-4c4f-bde7-f568d5b3c27e`
2. **API Key (`apiKey`)**:
   * In Adalo, open the left sidebar and navigate to **Settings ⚙️** > **App Access**.
   * Under the **API Key** section, click **Generate Key** and copy the generated bearer token.
3. **Collection ID (`collectionId`)**:
   * In Adalo, open the left sidebar and click **Database 🗄️**.
   * Click the three dots (**...**) next to the target collection (e.g. *Users*, *Tasks*) and select **`<> API Documentation`**.
   * Copy the internal collection ID from the endpoint URL (typically starts with `t_`, e.g. `t_31acb5f8f06a4a28882e419610685a6b`).

### Use Cases

- **User Onboarding & Sync:** Automatically register new users in Adalo collections when they sign up on a website or CRM.
- **Task & Inventory Automation:** Fetch pending tasks or products from Adalo, process them with AI or backend logic, and update their statuses.
- **Data Cleanup:** Periodically archive or delete expired records and test data from Adalo tables.
- **Cross-Platform Integration:** Sync records between Adalo and external platforms (e.g. Mattermost, Slack, Google Sheets, or SQL databases).

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `listRecords` | The operation to perform: `listRecords`, `getRecord`, `createRecord`, `updateRecord`, or `deleteRecord`. |
| `apiKey` | `string` | ✅ Yes | — | Adalo secret API key (Bearer token). Supports expressions. |
| `appId` | `string` | ✅ Yes | — | Adalo Application UUID. Supports expressions. |
| `collectionId` | `string` | ✅ Yes | — | Target collection ID (e.g. `t_31acb5f8f...`). Supports expressions. |
| `recordId` | `string` | ⚠️ Conditional | — | Record identifier. **Required** for `getRecord`, `updateRecord`, and `deleteRecord`. Supports expressions. |
| `fields` | `string` | ⚠️ Conditional | — | JSON string representing the column key-value pairs to set. **Required** for `createRecord` and `updateRecord`. Supports expressions. |
| `limit` | `string` | ❌ No | `"20"` | Maximum number of records to return when using `listRecords`. Supports expressions. |

---

### Operations Reference

#### 1. `listRecords`
Fetches a list of records from the specified collection.
* **HTTP Endpoint:** `GET https://api.adalo.com/v0/apps/{appId}/collections/{collectionId}?limit={limit}`
* **Required Parameters:** `apiKey`, `appId`, `collectionId`
* **Optional Parameters:** `limit` (default: `20`)

#### 2. `getRecord`
Retrieves a single record by its unique ID.
* **HTTP Endpoint:** `GET https://api.adalo.com/v0/apps/{appId}/collections/{collectionId}/{recordId}`
* **Required Parameters:** `apiKey`, `appId`, `collectionId`, `recordId`

#### 3. `createRecord`
Inserts a new record into the collection with the supplied fields.
* **HTTP Endpoint:** `POST https://api.adalo.com/v0/apps/{appId}/collections/{collectionId}`
* **Required Parameters:** `apiKey`, `appId`, `collectionId`, `fields`

#### 4. `updateRecord`
Modifies an existing record in the collection.
* **HTTP Endpoint:** `PUT https://api.adalo.com/v0/apps/{appId}/collections/{collectionId}/{recordId}`
* **Required Parameters:** `apiKey`, `appId`, `collectionId`, `recordId`, `fields`

#### 5. `deleteRecord`
Permanently removes a record from the collection.
* **HTTP Endpoint:** `DELETE https://api.adalo.com/v0/apps/{appId}/collections/{collectionId}/{recordId}`
* **Required Parameters:** `apiKey`, `appId`, `collectionId`, `recordId`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream data stream. Values can be mapped dynamically into `recordId`, `fields`, or other parameters using expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the Adalo API returns a successful response (HTTP 200 or 204). |
| `error` | `object` | Emitted when validation, authentication, network, or Adalo API errors occur. |

---

### Output Schemas

#### `listRecords` Output

```json
{
  "records": [
    {
      "id": 1,
      "Email": "alice.smith@example.com",
      "Username": "alice",
      "Full Name": "Alice Smith",
      "created_at": "2026-10-08T12:49:00.563Z",
      "updated_at": "2026-10-08T12:49:00.563Z"
    },
    {
      "id": 2,
      "Email": "bob.jones@example.org",
      "Username": "bobj",
      "Full Name": "Bob Jones",
      "created_at": "2026-10-08T12:49:00.900Z",
      "updated_at": "2026-10-08T12:49:00.900Z"
    }
  ]
}
```

#### `getRecord`, `createRecord`, and `updateRecord` Output

```json
{
  "id": 6,
  "Email": "test.automation@example.com",
  "Username": "testuser",
  "Full Name": "Test Automation User",
  "created_at": "2026-10-08T15:13:48.247Z",
  "updated_at": "2026-10-08T15:13:48.247Z"
}
```

#### `deleteRecord` Output

```json
{
  "success": true
}
```

---

### Configuration Examples

#### 1. List Records with Pagination
```json
{
  "operation": "listRecords",
  "apiKey": "YOUR_ADALO_API_KEY",
  "appId": "3c3bd2cb-c2e5-4c4f-bde7-f568d5b3c27e",
  "collectionId": "t_31acb5f8f06a4a28882e419610685a6b",
  "limit": "10"
}
```

#### 2. Create a Record
```json
{
  "operation": "createRecord",
  "apiKey": "YOUR_ADALO_API_KEY",
  "appId": "3c3bd2cb-c2e5-4c4f-bde7-f568d5b3c27e",
  "collectionId": "t_31acb5f8f06a4a28882e419610685a6b",
  "fields": "{\n  \"Email\": \"user@example.com\",\n  \"Username\": \"johndoe\",\n  \"Full Name\": \"John Doe\"\n}"
}
```

#### 3. Update a Record Dynamically
```json
{
  "operation": "updateRecord",
  "apiKey": "YOUR_ADALO_API_KEY",
  "appId": "3c3bd2cb-c2e5-4c4f-bde7-f568d5b3c27e",
  "collectionId": "t_31acb5f8f06a4a28882e419610685a6b",
  "recordId": "{{ $json.userId }}",
  "fields": "{\n  \"Full Name\": \"John Doe Updated\"\n}"
}
```

#### 4. Delete a Record
```json
{
  "operation": "deleteRecord",
  "apiKey": "YOUR_ADALO_API_KEY",
  "appId": "3c3bd2cb-c2e5-4c4f-bde7-f568d5b3c27e",
  "collectionId": "t_31acb5f8f06a4a28882e419610685a6b",
  "recordId": "6"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve and log Adalo records
```

### Sample Workflow: Fetch User from Adalo and Log Details

In this workflow, a **Manual Trigger** initiates execution, the **Adalo** node queries the `Users` collection using `getRecord`, and a **Log** node records the returned user profile:

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger"
    },
    {
      "id": "function",
      "type": "function",
      "config": {
        "code": "return {\n  userId: \"1\"\n};"
      }
    },
    {
      "id": "adalo",
      "type": "adalo-action",
      "config": {
        "operation": "getRecord",
        "apiKey": "YOUR_ADALO_API_KEY",
        "appId": "3c3bd2cb-c2e5-4c4f-bde7-f568d5b3c27e",
        "collectionId": "t_31acb5f8f06a4a28882e419610685a6b",
        "recordId": "{{ $json.userId }}"
      }
    },
    {
      "id": "log",
      "type": "log"
    }
  ]
}
```

### Execution Flow

1. **Manual Trigger** kicks off the workflow.
2. **Function** node prepares target record parameters (e.g. `userId: "1"`).
3. **Adalo** node resolves the `recordId` expression, sends a secure authenticated `GET` request to Adalo's REST API, and receives the JSON record object.
4. **Log** node captures the record fields (`Email`, `Full Name`, timestamps) for inspection and downstream workflow routing.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Adalo API error 404: Request failed with status code 404`
**Cause:** The `collectionId` or `appId` does not exist in Adalo, or the collection name was entered instead of the technical ID.
**Solution:**
- Do not use the plain collection name (e.g. `"Users"`). Use the technical ID found in **Database > ... > API Documentation** (e.g. `t_31acb5f8f...`).
- Verify that `appId` matches the UUID in your Adalo app URL.

#### `Adalo API error 401: Unauthorized`
**Cause:** The provided `apiKey` is invalid, expired, or missing.
**Solution:**
- Go to **Settings > App Access** in Adalo and generate a new API key.
- Ensure the API key is passed without extra whitespace or quotes.

#### `Validation failed with 1 error(s) - recordId: Invalid string`
**Cause:** A numeric type was passed to `recordId` instead of a string.
**Solution:**
- Ensure expressions or input parameters provide `recordId` as a string value (e.g. `"1"` instead of `1`).

#### `fields must be a valid JSON string`
**Cause:** The JSON supplied in the `fields` parameter contains a syntax error (e.g. missing double quotes, unclosed braces).
**Solution:**
- Format `fields` as valid JSON: `{"PropertyName": "Value"}`.

---

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `apiKey is required` | The `apiKey` field was left blank | Supply your Adalo API Key. |
| `appId is required` | The `appId` field was left blank | Supply your Adalo App UUID. |
| `collectionId is required` | The `collectionId` field was left blank | Supply the technical Collection ID (`t_...`). |
| `recordId is required for <operation>` | An operation requiring a specific record was executed without `recordId` | Provide a valid `recordId`. |
| `fields is required for <operation>` | `createRecord` or `updateRecord` was called without payload data | Provide valid JSON fields. |
| `Adalo API error 404` | Non-existent app or collection endpoint | Verify `appId` and `collectionId`. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Prepare and format dynamic payloads and record IDs for Adalo
- [Manual Trigger](./manual-trigger.md) – Trigger Adalo database automations on demand
- [Webhook](./webhook.md) – Receive webhooks from third-party services to create records in Adalo
- [Log](./log.md) – Inspect Adalo query outputs and debugging logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-08 | Initial release with full CRUD support (`listRecords`, `getRecord`, `createRecord`, `updateRecord`, `deleteRecord`), dynamic expression mapping, JSON payload validation, and responsive SVG branding. |

<!-- /SECTION: changelog -->

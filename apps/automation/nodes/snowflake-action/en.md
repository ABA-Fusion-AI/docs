---
node_id: "snowflake-action"
title: "Snowflake Action"
description: "Execute queries and manage Snowflake data warehouse objects via the Snowflake SQL REST API."
category: "peer-only"
subcategory: "Integrations"
version: "1.1.0"
language: "en"
last_updated: "2026-10-05"
author: "Fusion Team"
tags:
  - snowflake
  - sql
  - data-warehouse
  - database
  - integration
  - peer-only
related_nodes:
  - snowflake
  - http-request
  - function
---

<!-- SECTION: overview -->
# Snowflake Action

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Run SQL statements and explore databases, schemas, tables, and columns in [Snowflake](https://www.snowflake.com) through the [Snowflake SQL REST API](https://docs.snowflake.com/en/developer-guide/sql-api/index) (`/api/v2/statements`). The node authenticates with an OAuth token, a key-pair JWT, or a programmatic access token (PAT). Most fields support expressions.

### Use Cases

- **Query the warehouse from a workflow:** Run a `SELECT` and pass the rows to downstream nodes (Function, If/Else, AI Chat, notifications).
- **Write results back:** Insert or update rows from data produced earlier in the workflow, using bindings for the values.
- **Schema discovery:** List databases, schemas, tables, and the columns of a table before building a query.
- **Long-running statements:** Check a statement's status and get its result with its `statementHandle`.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `executeQuery` | The action to perform. See operations below. |
| `account` | `string` | Yes | — | Snowflake account identifier, e.g. `MYORG-MYACCOUNT`. Used to build `https://<account>.snowflakecomputing.com`. Only letters, digits, `.`, `_` and `-` are accepted. Supports expressions. |
| `accessToken` | `string` | Yes | — | OAuth access token, key-pair JWT, or programmatic access token. Supports expressions. |
| `tokenType` | `enum` | No | `OAUTH` | Token type sent in the `X-Snowflake-Authorization-Token-Type` header: `OAUTH`, `KEYPAIR_JWT`, or `PROGRAMMATIC_ACCESS_TOKEN`. |
| `warehouse` | `string` | No | — | Warehouse sent with every statement. Needed for any statement that reads or writes table data, unless the user has a default warehouse. Supports expressions. |
| `database` | `string` | No | — | Database sent with every statement; required for `listSchemas` and `listTables`. Supports expressions. |
| `schema` | `string` | No | — | Schema sent with every statement; required for `listTables`. Supports expressions. |
| `sql` | `string` | No | — | SQL statement to run. Required for `executeQuery`. Supports expressions. |
| `bindings` | `string` (JSON) | No | `"{}"` | JSON object of values for `?` placeholders in `sql`. Supports expressions. See [Security](#security). |
| `acknowledgeRisk` | `boolean` | No | `false` | Must be `true` to run `executeQuery`. |
| `tableName` | `string` | No | — | Table for `listColumns`, e.g. `MY_DB.PUBLIC.ORDERS` (each part is quoted separately). Supports expressions. |
| `statementHandle` | `string` | No | — | Handle returned by a previous `executeQuery`. Required for `getQueryStatus`. Supports expressions. |

### Available Operations

| Operation | Description | Snowflake request | Required Parameters |
|-----------|-------------|-------------------|---------------------|
| `executeQuery` | Run a custom SQL statement. | `POST /api/v2/statements` | `sql`, `acknowledgeRisk: true` |
| `listDatabases` | List the databases visible to the token's role. | `SHOW DATABASES` | — |
| `listSchemas` | List the schemas of a database. | `SHOW SCHEMAS IN DATABASE "<database>"` | `database` |
| `listTables` | List the tables of a schema. | `SHOW TABLES IN SCHEMA "<database>"."<schema>"` | `database`, `schema` |
| `listColumns` | Describe the columns of a table. | `DESCRIBE TABLE "<db>"."<schema>"."<table>"` | `tableName` |
| `getQueryStatus` | Get the status / result of a statement by its handle. | `GET /api/v2/statements/<statementHandle>` | `statementHandle` |

All operations except `getQueryStatus` are sent as `POST /api/v2/statements` with `timeout: 60` (seconds) and the `warehouse`, `database`, and `schema` values when they are set.

### Parameters Used by Operation

All parameters are shown in the editor for every operation; this table shows which ones each operation actually uses.

| Parameter | `executeQuery` | `listDatabases` | `listSchemas` | `listTables` | `listColumns` | `getQueryStatus` |
|-----------|:---:|:---:|:---:|:---:|:---:|:---:|
| `account`, `accessToken`, `tokenType` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `sql`, `acknowledgeRisk`, `bindings` | ✅ | — | — | — | — | — |
| `warehouse` | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| `database` | ✅ | ✅ | ✅ (required) | ✅ (required) | ✅ | — |
| `schema` | ✅ | ✅ | ✅ | ✅ (required) | ✅ | — |
| `tableName` | — | — | — | — | ✅ | — |
| `statementHandle` | — | — | — | — | — | ✅ |

### Getting a Token (Programmatic Access Token)

1. In Snowsight, open a SQL worksheet as `ACCOUNTADMIN`.
2. A PAT is only accepted for a user that is subject to a network policy (or whose authentication policy relaxes this requirement). For a quick test you can attach a policy that allows all IPs:

   ```sql
   CREATE NETWORK RULE MY_DB.PUBLIC.ALLOW_ALL MODE = INGRESS TYPE = IPV4 VALUE_LIST = ('0.0.0.0/0');
   CREATE NETWORK POLICY FUSION_ALLOW_ALL ALLOWED_NETWORK_RULE_LIST = ('MY_DB.PUBLIC.ALLOW_ALL');
   ALTER USER "my_user" SET NETWORK_POLICY = FUSION_ALLOW_ALL;
   ```

   In production, restrict `VALUE_LIST` to the IP ranges of your Fusion environment.
3. Generate the token and copy the secret (it is shown only once):

   ```sql
   ALTER USER "my_user" ADD PROGRAMMATIC ACCESS TOKEN fusion
     ROLE_RESTRICTION = 'MY_ROLE' DAYS_TO_EXPIRY = 30;
   ```

4. In the node, set `account` (Snowsight → account menu → **View account details** → *Account identifier*), `accessToken`, and `tokenType: PROGRAMMATIC_ACCESS_TOKEN`.

> Usernames created in lowercase must be quoted (`"my_user"`), otherwise Snowflake looks for `MY_USER`.

### Migrating from Basic Authentication

Version 1.1.0 replaced the legacy `username` and `password` fields with `accessToken` and `tokenType`. Workflows that still use Basic authentication must be updated with a token before deployment.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. The incoming data is not used directly; use expressions in the parameters (e.g. `sql`, `bindings`, `tableName`) to pass upstream values. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The raw Snowflake SQL API response for the operation. If Snowflake returns a non-JSON body, it is wrapped as `{ "result": "<text>" }`. |
| `error` | `Error` | Validation errors, authentication failures, network issues, or Snowflake API errors. |

The response is returned as Snowflake sends it: column definitions are in `resultSetMetaData.rowType` and rows are in `data` as **arrays of strings** (numbers included).

### Output Examples

#### `executeQuery` — `SELECT ID, NAME FROM ITEMS ORDER BY ID`

```json
{
  "resultSetMetaData": {
    "numRows": 2,
    "format": "jsonv2",
    "rowType": [
      { "name": "ID", "type": "fixed", "nullable": true },
      { "name": "NAME", "type": "text", "nullable": true }
    ]
  },
  "data": [
    ["1", "alpha"],
    ["2", "beta"]
  ],
  "code": "090001",
  "message": "Statement executed successfully.",
  "statementHandle": "01c786c7-0002-2510-0000-0002d55e9149",
  "statementStatusUrl": "/api/v2/statements/01c786c7-0002-2510-0000-0002d55e9149",
  "sqlState": "00000"
}
```

#### `listTables` — `database: FUSION_TEST`, `schema: PUBLIC`

```json
{
  "resultSetMetaData": { "numRows": 1, "rowType": [{ "name": "created_on" }, { "name": "name" }, { "name": "..." }] },
  "data": [
    ["1791200000.000", "ITEMS", "..."]
  ],
  "code": "090001",
  "message": "Statement executed successfully."
}
```

The object name is the second column (`name`) of each row for `listDatabases`, `listSchemas`, and `listTables`.

#### `listColumns` — `tableName: FUSION_TEST.PUBLIC.ITEMS`

```json
{
  "resultSetMetaData": { "numRows": 2, "rowType": [{ "name": "name" }, { "name": "type" }, { "name": "..." }] },
  "data": [
    ["ID", "NUMBER(38,0)", "..."],
    ["NAME", "VARCHAR(16777216)", "..."]
  ],
  "code": "090001",
  "message": "Statement executed successfully."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Snowflake in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Snowflake Action Node:** Runs the configured operation (e.g. `executeQuery`) with the account, token, and query parameters.
3. **Log Node:** Displays the Snowflake response.

### Example Configurations

#### Run a query

```json
{
  "operation": "executeQuery",
  "account": "MYORG-MYACCOUNT",
  "accessToken": "{{ secrets.SNOWFLAKE_TOKEN }}",
  "tokenType": "PROGRAMMATIC_ACCESS_TOKEN",
  "warehouse": "MY_WH",
  "database": "FUSION_TEST",
  "schema": "PUBLIC",
  "sql": "SELECT ID, NAME FROM ITEMS ORDER BY ID",
  "acknowledgeRisk": true
}
```

#### Query with a bound value

```json
{
  "operation": "executeQuery",
  "warehouse": "MY_WH",
  "database": "FUSION_TEST",
  "schema": "PUBLIC",
  "sql": "SELECT NAME FROM ITEMS WHERE ID = ?",
  "bindings": "{\"1\":{\"type\":\"FIXED\",\"value\":\"1\"}}",
  "acknowledgeRisk": true
}
```

#### List the columns of a table

```json
{
  "operation": "listColumns",
  "warehouse": "MY_WH",
  "tableName": "FUSION_TEST.PUBLIC.ITEMS"
}
```

#### Check a statement by handle

```json
{
  "operation": "getQueryStatus",
  "statementHandle": "01c786c7-0002-2510-0000-0002d55e9149"
}
```

### Common Patterns

- **Discovery chain:** `listDatabases` → `listSchemas` → `listTables` → `listColumns` to explore an unknown warehouse.
- **Query → Function → Notification:** Run a report query, turn `data` rows into objects in a Function node, then send a summary.
- **Webhook → Insert:** Receive an event and insert it with `executeQuery` and bindings (never concatenate input into `sql`).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `accessToken` in Fusion's **Secrets** system. Never paste tokens directly into workflow parameters or commit them to version control.

Custom SQL requires `acknowledgeRisk: true`. Put dynamic values in bindings rather than concatenating them into SQL:

```json
{
  "sql": "SELECT * FROM orders WHERE customer_id = ?",
  "bindings": "{\"1\":{\"type\":\"TEXT\",\"value\":\"customer-1\"}}",
  "acknowledgeRisk": true
}
```

- Binding positions must be positive integers (`"1"`, `"2"`, …).
- Binding values must be strings (or arrays of strings).
- Supported binding types: `TEXT`, `FIXED`, `REAL`, `BOOLEAN`, `BINARY`, `DATE`, `TIME`, `TIMESTAMP_NTZ`, `TIMESTAMP_LTZ`, `TIMESTAMP_TZ`.
- `listSchemas`, `listTables`, and `listColumns` quote object identifiers safely. Account identifiers and statement handles are validated before being used in URLs.
- Snowflake does not support bindings in multi-statement requests — send one statement per execution.
- Use a least-privilege role (`ROLE_RESTRICTION` on the PAT) and a short token expiry. Restrict the network policy to your Fusion environment's IPs in production.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Snowflake API error 401` — `390432 Fail : Network policy is required.`
- **Cause:** The token is a PAT and the Snowflake user has no network policy.
- **Solution:** Attach a network policy to the user (see [Getting a Token](#getting-a-token-programmatic-access-token)) or set an authentication policy with `PAT_POLICY = (NETWORK_POLICY_EVALUATION = ENFORCED_NOT_REQUIRED)`.

#### `Snowflake API error 401` — other codes
- **Cause:** Wrong, expired, or revoked token, or a key-pair JWT older than one hour.
- **Solution:** Generate a new token and check that `tokenType` matches it.

#### `Snowflake API error 422` — `391920 … You must specify the warehouse to use`
- **Cause:** `warehouse` is empty, misspelled, or the warehouse does not exist / is not usable by the token's role. Snowflake ignores an unknown warehouse name and then fails with this error.
- **Solution:** Run `executeQuery` with `SHOW WAREHOUSES` (it works without a warehouse) and use one of the returned names.

#### `executeQuery requires acknowledgeRisk to be true`
- **Cause:** `acknowledgeRisk` is `false` (the default).
- **Solution:** Set `acknowledgeRisk` to `true` after reviewing the SQL.

#### `database is required for listSchemas` / `schema is required for listTables` / `tableName is required for listColumns`
- **Cause:** A required field for the selected operation is empty.
- **Solution:** Fill in the field listed in [Available Operations](#available-operations).

#### `Snowflake bindings must be valid JSON` / `Invalid binding position` / `Unsupported Snowflake binding type`
- **Cause:** `bindings` is not a JSON object of `{ "<position>": { "type": "...", "value": "..." } }`.
- **Solution:** Fix the JSON; see the example in [Security](#security).

#### `Invalid Snowflake account identifier` / `Invalid qualified Snowflake identifier`
- **Cause:** `account` contains a URL or other characters, or `tableName` has an empty part (e.g. `DB..TABLE`).
- **Solution:** Use only the account identifier (e.g. `MYORG-MYACCOUNT`) and a `DB.SCHEMA.TABLE` table name.

#### Query returns only a handle and no rows
- **Cause:** The statement did not finish within the 60-second timeout the node sends, so Snowflake answered asynchronously.
- **Solution:** Call `getQueryStatus` with the returned `statementHandle` until the result is available.

#### Object does not exist or not authorized
- **Cause:** The database, schema, or table does not exist, or the token's role has no privilege on it. Identifiers are quoted, so names are case-sensitive (`items` ≠ `ITEMS`).
- **Solution:** Check names with the list operations and the role's grants.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Snowflake](./snowflake.md) – Earlier version with `executeQuery`, `getQueryResults`, `listDatabases`, and `listSchemas`
- [HTTP Request](./http-request.md) – Call other Snowflake REST endpoints
- [Function](./function.md) – Convert Snowflake `data` rows into objects
- [Cron](./cron.md) – Schedule recurring queries

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-10-05 | Full documentation: parameters, operations, output examples, token setup, troubleshooting |
| 1.1.0 | 2026-08-05 | Token-based authentication (`accessToken`, `tokenType`) replaces Basic authentication; bindings |

<!-- /SECTION: changelog -->
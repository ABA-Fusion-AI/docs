---
node_id: "mssql"

title: "SQL Server"

description: "Connect to Microsoft SQL Server and perform Insert, Update, Delete, Select, and raw SQL Query operations."

category: "Database / SQL"

version: "1.0.0"

language: "en"

last_updated: "2026-09-08"

author: "Fusion Team"

tags:

- mssql

- sql-server

- database

- sql

- insert

- update

- delete

- select

- raw-query

related_nodes:

- function

- if

- postgres

- mysql

---

**# SQL Server**

> **\*\*Category:\*\*** database-nodes | **\*\*Type:\*\*** Action Node

Connect to **\*\*Microsoft SQL Server\*\*** and execute structured database operations or validated raw SQL queries.

The **\*\*SQL Server\*\*** node exposes five operations: `Insert`, `Update`, `Delete`, `Select`, and `SQL Query`.

**### Supported Features**

\- Connect to Microsoft SQL Server using username and password authentication

\- Configure server, database, port, encryption, and certificate trust

\- Insert records

\- Update records using required WHERE conditions

\- Delete records using required WHERE conditions

\- Select all or specific columns

\- Apply equality and `IS NULL` filters

\- Limit Select results using SQL Server `TOP`

\- Execute validated raw SQL statements with parameter bindings

\- Return SQL Query results as objects or arrays

\- Validate table, schema, column, and record-key identifiers

\- Reuse and close a SQL Server connection pool

**### Use Cases**

\- Store workflow data in SQL Server

\- Retrieve records for downstream workflow processing

\- Update or delete existing database records

\- Execute custom parameterized SQL queries

\- Integrate local or remote Microsoft SQL Server databases with Fusion workflows

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `user` | `string` | ✅ Yes | — | SQL Server username. |
| `password` | `string` | ✅ Yes | — | Password for the username. |
| `server` | `string` | ✅ Yes | `"localhost"` | SQL Server hostname or IP. |
| `database` | `string` | ✅ Yes | — | Database name. |
| `port` | `number` | ❌ No | — | Optional SQL Server TCP port. |
| `encrypt` | `boolean` | ❌ No | `true` | Enable TLS encryption. |
| `trustServerCertificate` | `boolean` | ❌ No | `true` | Accept self-signed certificates. |
| `operation` | `enum` | ✅ Yes | — | `Insert`, `Update`, `Delete`, `Select`, or `SQL Query`. |
| `schemaName` | `string` | ❌ No | — | Optional SQL schema. |
| `table` | `string` | ✅ Yes* | — | Table for Insert, Update, Delete, and Select operations. |

**### Insert Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `insertData` | `record` | ✅ Yes | — | Column/value object to insert. |

**### Update Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `updateParams.data` | `record` | ✅ Yes | — | Columns and new values. |
| `updateParams.where` | `record` | ✅ Yes | — | Required WHERE conditions. |

**### Delete Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `deleteParams.where` | `record` | ✅ Yes | — | Required WHERE conditions. |

**### Select Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `selectParams.columns` | `string[]` | ❌ No | — | Columns to retrieve. Defaults to `*` behavior when empty. |
| `selectParams.where` | `record` | ❌ No | — | Optional WHERE conditions. |
| `selectParams.limit` | `number` | ❌ No | — | Positive row limit implemented with `TOP`. |

**### SQL Query Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `sqlQueryParams.statement` | `string` | ✅ Yes | — | SQL statement. |
| `sqlQueryParams.binds` | `record` | ❌ No | — | Named SQL parameter values. |
| `sqlQueryParams.acknowledgeRisk` | `boolean` | ❌ No | `false` | Confirm that the raw SQL statement is trusted. |
| `sqlQueryParams.outFormat` | `enum` | ❌ No | `"object"` | Output as `object` or `array`. |

**---**

**## Operations**

| Operation | SQL | Description |
| --------- | --- | ----------- |
| `Insert` | `INSERT INTO` | Insert one record. |
| `Update` | `UPDATE ... SET ... WHERE` | Update matching records. |
| `Delete` | `DELETE FROM ... WHERE` | Delete matching records. |
| `Select` | `SELECT` | Retrieve rows with optional columns, conditions, and limit. |
| `SQL Query` | Custom SQL | Execute a validated raw SQL statement. |

**---**

**## Request Body Construction**

**### Insert**

```sql
INSERT INTO <table> (name, email) VALUES (@value_1, @value_2)
```

Values are registered as SQL parameters.

**### Update**

```sql
UPDATE <table> SET name = @set_1 WHERE id = @where_1
```

**### Delete**

```sql
DELETE FROM <table> WHERE id = @where_1
```

**### Select**

Without selected columns:

```sql
SELECT * FROM <table>
```

With a limit:

```sql
SELECT TOP (10) id, name FROM <table>
```

**### WHERE Clause Construction**

Normal values:

```sql
column = @where_1
```

Null values:

```sql
column IS NULL
```

Undefined values are ignored. Multiple conditions are joined using `AND`.

**### SQL Query**

Bindings are registered using `request.input(name, value)` before the statement is executed.

**---**

**## Inputs & Outputs**

**### Inputs**

The node does not use incoming workflow data directly. Operations are controlled by node configuration.

**### Outputs**

| Operation | Output |
| --------- | ------ |
| `Insert` | `{ rowsAffected: result.rowsAffected }` |
| `Update` | `{ rowsAffected: result.rowsAffected }` |
| `Delete` | `{ rowsAffected: result.rowsAffected }` |
| `Select` | `recordset` or `[]` |
| `SQL Query` object format | Rows as objects |
| `SQL Query` array format | Rows as arrays |
| `SQL Query` without recordset | `{ rowsAffected: result.rowsAffected }` |

**### Output Example**

```json
{
  "rowsAffected": [1]
}
```

Select example:

```json
[
  {
    "id": 1,
    "name": "Yassine"
  }
]
```

**---**

**## Configuration Examples**

**### Insert**

```json
{
  "user": "fusion_user",
  "password": "YOUR_PASSWORD",
  "server": "localhost",
  "database": "FusionTest",
  "port": 1433,
  "operation": "Insert",
  "schemaName": "dbo",
  "table": "Users",
  "insertData": {
    "name": "Yassine",
    "email": "yassine@example.com"
  }
}
```

**### Select**

```json
{
  "user": "fusion_user",
  "password": "YOUR_PASSWORD",
  "server": "localhost",
  "database": "FusionTest",
  "operation": "Select",
  "schemaName": "dbo",
  "table": "Users",
  "selectParams": {
    "columns": ["id", "name"],
    "where": {
      "id": 1
    },
    "limit": 10
  }
}
```

**### SQL Query**

```json
{
  "user": "fusion_user",
  "password": "YOUR_PASSWORD",
  "server": "localhost",
  "database": "FusionTest",
  "operation": "SQL Query",
  "sqlQueryParams": {
    "statement": "SELECT id, name FROM dbo.Users WHERE id = @userId",
    "binds": {
      "userId": 1
    },
    "acknowledgeRisk": true,
    "outFormat": "object"
  }
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → SQL Server (`Insert`)

\- SQL Server (`Select`) → Function

\- SQL Server (`Select`) → If

\- SQL Server (`Update`) → Notification

\- SQL Server (`SQL Query`) → Function

**---**

**## Error Handling**

**### Missing Table**

```text
`table` is required for this operation
```

**### Invalid Identifier**

```text
Invalid <fieldName>. Only letters, numbers, and underscore are allowed.
```

**### Missing Required Record**

```text
`<fieldName>` is required
```

**### Invalid Record**

```text
`<fieldName>` must be an object
```

**### Empty Required Record**

```text
`<fieldName>` must contain at least one key
```

**### Unsafe Update**

```text
Update requires at least one where clause condition
```

**### Unsafe Delete**

```text
Delete requires at least one where clause condition
```

**### Invalid SQL Statement**

```text
`statement` must be a non-empty SQL string
```

**### Missing Connection Pool**

```text
SQL Server connection pool is null, make sure setup has run
```

**### Unsupported Operation**

```text
Unsupported operation: <operation>
```

**---**

**## Troubleshooting**

**### "Invalid object name 'dbo.TableName'."**

**\*\*Cause\*\***

The configured schema or table does not exist in the selected database.

**\*\*Solution\*\***

Verify `database`, `schemaName`, and `table`.

---

**### Update or Delete Rejects the WHERE Configuration**

**\*\*Cause\*\***

Update and Delete require at least one usable condition.

**\*\*Solution\*\***

Provide an explicit condition such as:

```json
{
  "where": {
    "id": 1
  }
}
```

---

**### Filtering NULL Values**

Use:

```json
{
  "where": {
    "deleted_at": null
  }
}
```

The generated condition is:

```sql
deleted_at IS NULL
```

**---**

**## Security**

Structured table, schema, column, and record-key identifiers are validated before SQL generation.

Structured values are bound as SQL parameters instead of being directly interpolated into generated statements.

Raw SQL bind names are checked by `validateMssqlParameterNames()`, and raw statements are passed through `validateMssqlRawQuery()`.

For production workflows:

\- Store database credentials securely

\- Use least-privilege SQL Server accounts

\- Use TLS for remote database connections

\- Review raw SQL before acknowledging its risk

\- Configure certificate trust according to the environment

**---**

**## Notes**

The node creates the SQL Server connection pool during `setup()` and closes it during `stop()`.

If `schemaName` is supplied, the target table becomes:

```text
schemaName.table
```

If no schema is supplied, only `table` is used.

For WHERE clauses:

\- `undefined` is ignored

\- `null` becomes `IS NULL`

\- Other values become parameterized equality conditions

\- Multiple conditions use `AND`

Select uses:

```text
TOP (<limit>)
```

only when the limit is a positive integer.

The raw-query implementation can normalize a statement from a string or from an object containing `statement`, `sql`, or `query`, although the exposed schema defines `statement` as a string.

The node does not:

\- Automatically create databases or tables

\- Automatically migrate schemas

\- Automatically retry failed SQL operations

\- Automatically wrap operations in transactions

\- Automatically paginate Select results

\- Add implicit WHERE conditions to Update or Delete

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-08 | Initial release |

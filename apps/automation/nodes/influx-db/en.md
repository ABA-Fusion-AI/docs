---
node_id: "influx-db"
title: "InfluxDB"
description: "Write time-series points and query InfluxDB using a Flux builder or raw Flux."
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-09-28"
author: "Fusion Team"
tags: [influxdb, flux, time-series, database]
---

<!-- SECTION: overview -->
# InfluxDB

> **Category:** Data / Databases | **Type:** Action Node

Write a time-series point, build a query from measurement and filter settings, or execute a trusted Flux query. The node creates an InfluxDB client during setup and displays its running status while executing.

Typical uses include storing sensor readings, recording application metrics, and retrieving historical measurements for downstream processing.

This page describes the supplied `InfluxDBNode` implementation. The imported `InfluxDBFluxBuilder` and `formatResponse` implementations were not supplied, so their exact query-generation and output-formatting behavior is not specified here. Query operations require an endpoint that supports the Flux query API used by this client.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection and operation

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | `string` | Yes | None | Nonempty InfluxDB server URL, for example `http://localhost:8086`. Supports expressions. |
| `token` | `string` | Yes | None | Nonempty API token. Supports expressions. |
| `org` | `string` | Yes | None | Nonempty organization name. Supports expressions. |
| `bucket` | `string` | Yes | None | Nonempty bucket name. Required by the schema for all operations, including raw Flux. Supports expressions. |
| `operation` | `enum` | Yes | None | `write`, `query`, or `queryFlux`. |
| `debug` | `boolean` | No | `false` | Enables debug logging through the imported logging helper. |

### Write parameters

These settings apply to `operation: "write"`.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `measurement` | `string` | At runtime | None | Measurement name; a falsy value is rejected. Supports expressions. |
| `tags` | `array` | No | Unset | Tag entries with a nonempty string `name` and string `value`. Both support expressions. |
| `fields` | `array` | At runtime | Unset | At least one field entry is required. See conversion rules below. |
| `timestampMode` | `enum` | No | `Heure actuelle` | `Heure actuelle` uses current-time behavior; `Date personnalisée` uses an explicit timestamp. |
| `timestamp` | `string` | In custom-date mode | None | Nonempty ISO timestamp with a timezone. Supports expressions. The schema attaches conditional visibility to write/custom-date mode. |

Each field has a nonempty string `name`, a `value` that is a string, number, or boolean, and a `type` of `string`, `number`, or `boolean` (default: `string`). Field names and values support expressions.

| Field type | Conversion | Point method |
|------------|------------|--------------|
| `string` | `String(value)` | `stringField` |
| `number` | `Number(value)` | `floatField` |
| `boolean` | `Boolean(value)` | `booleanField` |

Use actual boolean values: the string `"false"` converts to `true` because it is nonempty. Numbers are written as floating-point fields; there is no integer field type in this schema. Invalid numeric strings can convert to `NaN`; the node performs no explicit finite-number validation before calling the client.

### Timestamp behavior

- `Heure actuelle`: the node does not set an explicit point timestamp. Any stale value in `timestamp` is ignored.
- `Date personnalisée`: `timestamp` must contain a valid calendar date, a time including seconds, and `Z` or an offset such as `+01:00`. Example: `2026-09-28T12:00:00Z`.
- The parser trims whitespace and checks month lengths and leap years. Date-only strings, timestamps without timezones, numeric epochs, and invalid dates are rejected.
- Fractional seconds are accepted, but the value is converted to a JavaScript `Date`, so custom timestamps retain only millisecond precision.
- The write handler also accepts a timestamp when the mode is null or missing and the timestamp is nonblank. This is a legacy fallback; schema defaulting may set `Heure actuelle` before the handler runs.

### Query-builder parameters

These settings apply to `operation: "query"`.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `measurement_query` | `string` | At runtime | None | Measurement passed to `builder.from(...)`. Supports expressions. |
| `fields_query` | `string[]` | No | Unset | When nonempty, passed to `builder.filter(...)`. |
| `where` | `array` | No | Unset | Conditions passed one at a time to `builder.where(...)`. |
| `range` | `object` | No | Unset | When supplied, passes `start` and optional `stop` to `builder.range(...)`. |

Each `where` condition contains:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `column` | `string` | Yes | Nonempty column name. |
| `operator` | `enum` | Yes | One of `==`, `!=`, `<`, `<=`, `>`, `>=`, `=~`, `!~`. |
| `value` | `string`, `number`, or `boolean` | Yes | Comparison value. |
| `logicalOperator` | `enum` | No | `and` or `or`. |

Within a supplied `range`, `start` defaults to `"-1h"`; `stop` is optional. The entire `range` object has no default. If it is omitted, the handler does not call `builder.range(...)`. The builder determines the resulting range behavior, supported time syntax, escaping, and how conditions are combined. Use an explicit range in configurations instead of assuming an implicit last-hour query.

### Raw Flux parameters

These settings apply to `operation: "queryFlux"`.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fluxQuery` | `string` | At runtime | None | Complete Flux program. Supports expressions. A falsy value is rejected. |
| `acknowledgeRisk` | `boolean` | Must be `true` | `false` | Confirms that the raw Flux program is trusted. |
| `range` | `object` | No | Unset | Visible for this operation in the schema, but ignored by the raw Flux handler. Put the range directly in `fluxQuery`. |

Raw Flux is passed directly to `queryRows(...)`; the node does not add a bucket, measurement, or time range. Use trusted Flux and a token with permissions appropriate to the intended operation. The configured `bucket` does not constrain the buckets referenced by raw Flux.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

The node runs from its configuration. Its `handleTick()` method accepts no input argument and does not read the incoming payload directly. Configure values or use the framework's expression-enabled fields to supply dynamic settings.

A successful operation returns `formatResponse(result)`. Because that helper is not included in the supplied code, the final output envelope cannot be confirmed from this implementation alone.

### Write result before formatting

```json
{
  "success": true,
  "message": "Data written successfully"
}
```

Each write execution constructs one point, calls `writePoint(...)`, and awaits `writeApi.close()` before returning this result.

### Query result before formatting

Both query modes collect records into an array. Each row is converted with `tableMeta.toObject(row)`. The array resolves only when the query completes; an empty result is `[]` before formatting. Record fields depend on the Flux query and returned table metadata. Results are buffered in memory rather than emitted one row at a time.

Errors are logged and rethrown. Query callback errors reject the operation; partially collected rows are not returned as a successful result.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

Replace the illustrative connection settings and token placeholder with your own configuration.

### Write a sensor reading

```json
{
  "url": "http://localhost:8086",
  "token": "<your-api-token>",
  "org": "example-org",
  "bucket": "telemetry",
  "operation": "write",
  "measurement": "temperature",
  "tags": [{ "name": "room", "value": "office" }],
  "fields": [
    { "name": "celsius", "value": 22.5, "type": "number" },
    { "name": "occupied", "value": false, "type": "boolean" }
  ],
  "timestampMode": "Heure actuelle"
}
```

To write an explicit time, replace `timestampMode` with `"Date personnalisée"` and add `"timestamp": "2026-09-28T12:00:00Z"`.

### Query through the builder

```json
{
  "url": "http://localhost:8086",
  "token": "<your-api-token>",
  "org": "example-org",
  "bucket": "telemetry",
  "operation": "query",
  "measurement_query": "temperature",
  "fields_query": ["celsius"],
  "where": [
    { "column": "room", "operator": "==", "value": "office" }
  ],
  "range": { "start": "-1h" }
}
```

This supplies the measurement, selected field, room condition, and range to the builder. The generated Flux is logged when debug logging is enabled.

### Execute trusted raw Flux

Use the same connection settings, set `operation` to `"queryFlux"`, set `acknowledgeRisk` to `true`, and set `fluxQuery` to:

```flux
from(bucket: "telemetry")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "temperature")
  |> filter(fn: (r) => r._field == "celsius")
```

The bucket and range are part of this program. Changing the separate `range` setting has no effect on raw Flux execution.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Query recent measurements and inspect the result
```

Connect **Manual Trigger → InfluxDB → Log**. Apply the query-builder example to InfluxDB, supply your connection settings, and run the trigger. Inspect the formatted query result in Log. The preview contains only node wiring; configure parameters and credentials before running it.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Explanation and action |
|------------------|------------------------|
| `InfluxDB client not initialized` | Setup has not initialized the client, or `stop()` has cleared it. Ensure setup completes before execution. |
| `Client not initialized` | A write/query helper found no client. Check the node lifecycle. |
| `measurement is required for write` | Provide a nonempty write measurement. |
| `fields are required for write` | Supply at least one field. |
| `timestamp must be a valid ISO 8601 date with a timezone` | Use a valid full timestamp with seconds and a timezone in custom-date mode. |
| `measurement_query is required` | Supply a measurement for the query builder. |
| `fluxQuery is required` | Supply a nonempty raw Flux program. |
| `Raw Flux requires acknowledgeRisk to be true` | Review the Flux program and set `acknowledgeRisk: true` when it is trusted. |
| `Unknown operation: <operation>` | Use `write`, `query`, or `queryFlux`; normal schema validation restricts the field to these values. |
| A boolean field is unexpectedly true | A nonempty string, including `"false"`, converts to `true`. Use the boolean `false`. |
| Raw query ignores configured range or bucket | Those settings are not injected into raw Flux. Specify them in the program itself. |
| Query or write fails at the server | Check the server URL, organization, token permissions, bucket, and the server error. Errors are rethrown without a new wrapper message. |

Debug logging receives the URL and organization during setup, write measurement/tags/fields, and generated builder Flux. It does not explicitly receive the token in this code. Enable it with awareness of the data contained in those values.

`stop()` only clears the client reference; it does not explicitly cancel in-flight requests. The node defines no custom timeout or retry policy; any client defaults depend on the installed library. Client construction alone is not proof of server connectivity.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-28 | Initial documentation based on the supplied InfluxDBNode implementation. |

<!-- /SECTION: changelog -->

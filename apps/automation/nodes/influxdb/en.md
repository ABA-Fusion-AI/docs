---
node_id: "influxdb"
title: "InfluxDB"
description: "Write time-series data and execute protected Flux queries."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [influxdb, flux, time-series, database]
---

# InfluxDB

Writes points and queries InfluxDB using either the structured Flux builder or trusted raw Flux.

## Security

Prefer the `query` operation. Bucket, measurement, fields, column names, string values, regular expressions, and timestamps are safely encoded by the builder. Range values accept only negative Flux durations or RFC 3339 UTC timestamps.

The `queryFlux` operation executes complete Flux programs and therefore requires `acknowledgeRisk: true`. Never construct raw Flux from webhook or other untrusted input.

## Workflow example

See [example.workflow.json](./example.workflow.json).

---
node_id: "dokku"
title: "Dokku"
description: "Run supported Dokku application commands over SSH."
category: "developer-tools"
subcategory: "deployment"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [dokku, ssh, deployment]
---

# Dokku

Runs a restricted set of Dokku commands over SSH: `apps:list`, `apps:create`, `ps:rebuild`, and `config:set`.

## Security

The command is selected from a fixed list. Application names are validated, configuration entries must use `KEY=VALUE`, and every argument is shell-quoted before being sent over SSH. Command arguments are not returned in node output, preventing configuration secrets from appearing in workflow results.

For configuration values containing spaces, quote the complete assignment, for example `my-app TOKEN="value with spaces"`.

## Workflow example

See [example.workflow.json](./example.workflow.json).

---
node_id: "azure-sql-database-admin"
title: "Azure SQL Database Admin"
description: "Manage Azure SQL databases and firewall rules"
category: "integrations"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - azure
  - sql
  - admin
related_nodes:
  - azure-sql-backup-restore
---

# Azure SQL Database Admin

> **Category:** Integrations | **Type:** Action Node

Operations:

- `listDatabases`, `getDatabase`, `createDatabase`, `updateDatabase`, `deleteDatabase`
- `pauseDatabase`, `resumeDatabase`
- `listFirewallRules`, `getFirewallRule`, `setFirewallRule`, `deleteFirewallRule`

## Configuration

Required:

- `subscriptionId`
- `credentials`: `{ tenantId, clientId, clientSecret }`
- `resourceGroupName`
- `serverName`
- `operation`

Optional (operation-specific):

- `databaseName`
- `parameters` (database payload)
- `firewallRuleName`
- `firewallRule` (`startIpAddress`, `endIpAddress`)

Outputs are Azure SDK responses, or status objects like `{ deleted: true }`, `{ paused: true }`, `{ resumed: true }`.

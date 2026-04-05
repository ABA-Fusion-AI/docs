---
node_id: "azure-sql-backup-restore"
title: "Azure SQL Backup / Restore"
description: "Create restore points and restore Azure SQL databases"
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
  - backup
  - restore
related_nodes:
  - azure-sql-database-admin
---

# Azure SQL Backup / Restore

> **Category:** Integrations | **Type:** Action Node

Actions:

- `createRestorePoint`
- `restoreDatabase`
- `restoreLongTermBackup`

## Configuration

Required:

- `subscriptionId`
- `credentials`: `{ tenantId, clientId, clientSecret }`
- `resourceGroupName`, `serverName`, `databaseName`
- `action`

Action-specific:

- `restorePointLabel` for restore point creation
- `restoreOptions` for database restore (target names, create mode, restore time)
- `longTermRetention` for long-term backup restore payload

Outputs are Azure SDK responses (or restore point identifiers).

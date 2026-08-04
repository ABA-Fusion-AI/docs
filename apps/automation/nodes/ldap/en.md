---
node_id: "ldap"
title: "LDAP"
description: "Search, authenticate, and manage LDAP directory entries and group memberships."
category: "security-networking"
subcategory: "identity-access"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [ldap, directory, identity, groups]
related_nodes: [keycloak, ms-entra-id]
---

<!-- SECTION: overview -->
# LDAP

> **Category:** Security & Networking&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Bind to an LDAP directory with the maintained `ldapts` client. Search and read entries, create/update/delete entries, authenticate users, and manage group membership.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `host` | string | Yes | — | LDAP server hostname. |
| `port` | number | No | `389` | LDAP port. |
| `dn`, `password` | string | Yes | — | Service bind credentials. |
| `baseDn` | string | Yes | — | Base distinguished name. |
| `operation` | enum | No | `search` | Directory operation. |
| `filter` | string | No | `(objectClass=*)` | LDAP search filter. |
| `attributes` | string | No | — | Comma-separated returned attributes. |
| `entryDn`, `attributes_json`, `objectClass` | string | Conditional | Entry-management fields. |
| `groupDn`, `memberDn` | string | Conditional | Group-membership fields. |
| `authDn`, `authPassword` | string | Conditional | User authentication credentials. |
| `sizeLimit` | number | No | `100` | Maximum search results. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Success:** Search entries, entry data, authentication status, or mutation confirmation.
- **Error:** Connection, bind, validation, or LDAP protocol error.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search an LDAP directory
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store bind and authentication passwords as workflow secrets. Use a least-privilege service account and a protected network connection.
<!-- /SECTION: security -->

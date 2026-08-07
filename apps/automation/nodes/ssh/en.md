---
node_id: "ssh"
title: "SSH"
description: "Execute commands on a remote server via SSH"
category: "devops"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - ssh
  - remote
  - devops
  - command
related_nodes:
  - http-request
  - function
  - docker
---

<!-- SECTION: header -->
# SSH

> **Category:** DevOps | **Type:** Action Node

Execute shell commands on a remote server over SSH and pass the result into downstream workflow steps.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **SSH** node connects to a remote host and runs a command using SSH authentication. It is useful for server administration, deployment automation, and remote command execution inside workflows.

### Key Features

- **Remote Command Execution:** Run shell commands on a target server
- **Authentication Support:** Connect with username and password credentials
- **Workflow Friendly:** Use outputs in later automation steps
- **Operational Automation:** Support common administrative tasks such as listing files or checking system state

### Use Cases

- Run remote maintenance commands
- Inspect directories or files on a server
- Trigger deployment or diagnostics tasks
- Integrate server operations into business workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | ✅ Yes | — | SSH host or hostname |
| `username` | `string` | ✅ Yes | — | SSH username |
| `password` | `string` | ✅ Yes | — | SSH password or authentication secret |
| `command` | `string` | ✅ Yes | — | Shell command to execute on the remote server |

### Example

```text
host: "test.rebex.net"
username: "demo"
password: "password"
command: "ls -la"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data that can be used to supply command values |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Command execution result |
| `error` | `object` | Error information if execution fails |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: List Files on a Remote Server

```text
host: "test.rebex.net"
username: "demo"
password: "password"
command: "ls -la"
```

**Result:**

```text
Directory contents returned by the remote shell
```

### Example: Run a Remote Diagnostics Command

```text
command: "uname -a"
```

<!-- /SECTION: examples -->

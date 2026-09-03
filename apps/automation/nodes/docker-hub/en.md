---
node_id: "docker-hub"
title: "Docker Hub"
description: "Get repository information from Docker Hub API."
category: "devops-cloud-management"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - "docker"
  - "docker-hub"
  - "repository"
  - "container"
  - "devops"
related_nodes:
  - "docker"
  - "npm"
  - "ssh"
---

<!-- SECTION: header -->

# Docker Hub

> **Category:** DevOps & Cloud Management | **Type:** Action Node

Get repository information from Docker Hub API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Docker Hub** node retrieves information about a repository from the Docker Hub API using a user or namespace and a repository name.

### Key Features

- Retrieve Docker Hub repository information
- Access repository metadata
- Retrieve pull and star counts
- Check repository privacy and automation status
- Retrieve repository description and last update information
- Return repository permissions when available
- Generate a direct Docker Hub repository URL

### Processing Flow

1. Configure the Docker Hub user or namespace.
2. Configure the repository name.
3. Send a request to the Docker Hub API.
4. Validate the API response.
5. Return the repository information.

### Use Cases

- Retrieve Docker image repository information
- Check whether a Docker Hub repository exists
- Inspect public repository metadata
- Retrieve repository statistics
- Integrate Docker Hub information into DevOps workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `user` | String | Yes | Docker Hub user or namespace, such as `library`. |
| `repo` | String | Yes | Docker Hub repository name, such as `nginx`. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The node uses the configured `user` and `repo` parameters to identify the Docker Hub repository.

Example:

```json
{
  "user": "library",
  "repo": "nginx"
}
```

### Outputs

On success, the node returns repository information including:

- `success`
- `user`
- `repository`
- `namespace`
- `repository_type`
- `status`
- `description`
- `full_description`
- `is_private`
- `is_automated`
- `star_count`
- `pull_count`
- `last_updated`
- `is_migrated`
- `affiliation`
- `permissions`
- `docker_hub_url`
- `note`

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Retrieve Repository Information

Use the following configuration to retrieve information about the official Nginx repository:

```json
{
  "user": "library",
  "repo": "nginx"
}
```

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: Docker Hub Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Repository Not Found

**Cause:** The specified Docker Hub namespace or repository does not exist.

**Solution:** Verify the `user` and `repo` values and confirm that the repository exists on Docker Hub.

### Missing User or Repository

**Cause:** The `user` or `repo` parameter is empty.

**Solution:** Provide both required parameters before executing the node.

### Docker Hub API Error

**Cause:** Docker Hub returned an HTTP error or the service could not be reached.

**Solution:** Verify connectivity to Docker Hub and retry the workflow.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **Docker** — Interact with Docker containers and images.
- **NPM** — Work with npm-related resources.
- **SSH** — Interact with remote systems through SSH.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-03` | Initial documentation. |

<!-- /SECTION: changelog -->
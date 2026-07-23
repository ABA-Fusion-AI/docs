---
node_id: "cloudflare-pages"
title: "Cloudflare Pages"
description: "List Cloudflare Pages projects or create a deployment"
category: "infrastructure"
subcategory: "hosting"
language: "en"
tags: [cloudflare, pages, deployment, hosting]
related_nodes: [vercel, netlify, firebase-hosting]
---

<!-- SECTION: header -->
# Cloudflare Pages

> **Category:** Infrastructure | **Type:** Action Node

Lists Cloudflare Pages projects or creates a deployment for an existing Pages project.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Required | Default | Description |
|---|---:|---:|---|
| `apiToken` | Yes | — | Cloudflare API token with **Pages Read** or **Pages Write** permission. |
| `accountId` | Yes | — | Cloudflare account ID. |
| `operation` | No | `list_projects` | Choose `list_projects` or `create_deployment`. |
| `projectName` | For deployment | — | Existing Cloudflare Pages project name. |
| `branch` | No | `main` | Branch used for the deployment. |

Cloudflare Pages deployments require a project already connected to a source repository or configured for direct uploads.
<!-- /SECTION: configuration -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Cloudflare Pages projects
```
<!-- /SECTION: examples -->

---
node_id: "kobotoolbox-form"
title: "KoboToolbox Form"
description: "Manage KoboToolbox forms, schemas, survey definitions, and project deployments."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-12"
author: "Fusion Team"
tags: [kobotoolbox, forms, survey, deployment, peer-only]
related_nodes: ["kobotoolbox-submission", "kobotoolbox-file", "kobotoolbox-hook", "kobotoolbox"]
---

<!-- SECTION: overview -->
# KoboToolbox Form

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **KoboToolbox Form** node provides complete project and form management for the KoboToolbox platform via API v2. It allows workflows to discover available forms, inspect questions and schema metadata, and programmatically redeploy updated surveys.

### Key Capabilities
- **Get Many Forms (`getMany`)**: List all projects with support for search queries (`q`), ordering, and pagination.
- **Get Form (`get`)**: Fetch complete asset details, questions definition, survey settings, and translation languages.
- **Redeploy Form (`redeploy`)**: Trigger automated redeployment of an updated form to make changes live for field enumerators.
<!-- /SECTION: overview -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Configuration Parameters
- **Operation (`operation`)**: `getMany`, `get`, or `redeploy` (Default: `getMany`).
- **API Token (`apiToken`)**: KoboToolbox authentication token (supports dynamic expressions).
- **Base URL (`baseUrl`)**: Server domain (Defaults to `https://kf.kobotoolbox.org`).
- **Form ID (`formId`)**: Unique asset identifier (required for `get` and `redeploy`).
- **Search Query (`q`)**: Filter string for searching forms by title or metadata in `getMany`.
- **Ordering (`ordering`)**: Sort criteria, e.g., `-date_modified` or `name`.
- **Start / Limit**: Pagination offset and batch size controls.

### Outputs
- **Success Output**:
  - `getMany`: Array of forms with metadata and deployment statuses.
  - `get`: Full asset object including survey schema and question definitions.
  - `redeploy`: Deployment status confirming active version deployment.
- **Error Output**: Standard error output detailing API error messages and HTTP status codes.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve Form Details and Redeploy
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

- Authenticate using API Tokens with restricted permissions.
- Never hardcode personal credentials; store your KoboToolbox API token in Fusion Credential Manager or secret environment variables.
<!-- /SECTION: security -->

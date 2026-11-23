---
node_id: "orthanc"
title: "Orthanc PACS"
description: "Upload, search, download, anonymize, modify, delete and route DICOM medical images with an Orthanc PACS server."
category: "peer-only"
subcategory: "Healthcare & Life Sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-11-22"
author: "Fusion Team"
tags:
  - orthanc
  - pacs
  - dicom
  - medical-imaging
  - healthcare
  - c-store
related_nodes: []
---

<!-- SECTION: overview -->
# Orthanc PACS

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Work with medical images stored in an **Orthanc** PACS server through its REST API: **upload** DICOM files, **search** patients, studies, series and instances, **download** a DICOM file or a preview image, **anonymize** or **modify** tags, **delete** resources and **send** images to another DICOM node (C-STORE). The node offers **9 operations**:

| Resource | Operations |
|----------|------------|
| Images | `upload_instance`, `get_file` |
| Search & details | `find_resources`, `get_resource` |
| Transformation | `anonymize`, `modify` |
| Management | `delete_resource`, `send_to_modality` |
| Server | `system_info` |

### Use Cases

- **Image ingestion:** Upload DICOM files received from another system into the PACS.
- **Research datasets:** Find studies and anonymize them before sharing them.
- **Routing:** Send new studies to a viewer, an AI service or another PACS.
- **Data correction:** Fix wrong tags (e.g. a study description) on a study.
- **Previews:** Get a PNG/JPEG preview of an instance for a report or a notification.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `orthancUrl` | `string` | Yes | Orthanc base URL (default `http://localhost:8042`). |
| `username` | `string` | If auth is enabled | Orthanc user (often `orthanc`). |
| `password` | `string` | If auth is enabled | Orthanc password. Credentials are sent only when both `username` and `password` are set. |
| `operation` | `enum` | Yes | Operation to perform (default `find_resources`). |
| `level` | `enum` | For `find_resources`, `delete_resource` | `Patient`, `Study` (default), `Series` or `Instance`. |
| `query` | `string` | No | `find_resources` filter as JSON, e.g. `{"PatientName": "JONES*", "StudyDate": "20230101-"}`. Empty = all resources of the level. |
| `expand` | `boolean` | No | Return full details instead of IDs only (default `true`). |
| `resourceId` | `string` | For get / file / delete / anonymize / modify | Orthanc ID of the resource (the `ID` returned by `find_resources` or `upload_instance`). |
| `fileUrl` | `string` | For `upload_instance` | Public URL of the `.dcm` file to upload. |
| `fileFormat` | `enum` | No | For `get_file`: `dicom` (default), `preview_png` or `preview_jpeg`. |
| `modifications` | `string` | No | JSON rules for `modify` / `anonymize` (see below). |
| `targetModality` | `string` | For `send_to_modality` | Name of a modality declared in Orthanc. |
| `resourcesToSend` | `string` | For `send_to_modality` | JSON array of Orthanc IDs, e.g. `["<study-id>"]`. |

> Fields are shown only for the operations that use them.

### Operations

| Operation | Endpoint | Required | Notes |
|-----------|----------|----------|-------|
| `system_info` | `GET /system` | — | Server version, DICOM AET and ports. |
| `upload_instance` | `POST /instances` | `fileUrl` | The node downloads the file from the URL, then sends it to Orthanc. |
| `find_resources` | `POST /tools/find` | `level` | Optional `query`, `expand`. |
| `get_resource` | `GET /studies|series|instances|patients/{id}` | `resourceId` | The level is detected automatically (tried in this order). |
| `get_file` | `GET /instances/{id}/file`, `/preview`, `/rendered` | `resourceId` | **Instance ID** only. Returns the file as Base64. |
| `delete_resource` | `DELETE /{level}/{id}` | `resourceId`, `level` | Permanent. |
| `anonymize` | `POST /studies|series/{id}/anonymize` | `resourceId` | **Study or Series ID.** Creates a new anonymized copy. |
| `modify` | `POST /studies|series/{id}/modify` | `resourceId`, `modifications` | **Study or Series ID.** Creates a new modified copy. |
| `send_to_modality` | `POST /modalities/{name}/store` | `targetModality`, `resourcesToSend` | DICOM C-STORE to another node. |

### Modification rules

- **modify:** `{"Replace": {"StudyDescription": "Corrected"}, "Remove": ["0010-0020"]}`
- **anonymize:** empty for the default anonymization, or for example `{"Keep": ["SeriesDescription"]}` to keep some tags.

> `anonymize` and `modify` do not change the original: Orthanc creates a **new** study or series and returns its `ID` and `Path`.

### Connection

- **Self-hosted:** run Orthanc with Docker:
  ```bash
  docker run -d --name orthanc -p 8042:8042 -p 4242:4242 jodogne/orthanc-plugins
  ```
  The default credentials of this image are `orthanc` / `orthanc`. If Fusion cannot reach your machine directly, expose port 8042 with a tunnel and use the public URL.
- **Modalities (for `send_to_modality`):** declare the target DICOM node in Orthanc, for example with the REST API:
  ```bash
  curl -u orthanc:orthanc -X PUT http://localhost:8042/modalities/target \
    -d '{"AET":"ORTHANC","Host":"orthanc2","Port":4242}'
  ```
  Then use `target` as `targetModality`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. an ID from a previous search) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `array` | JSON response of Orthanc; `get_file` returns `{ "fileType", "data" }` with `data` in Base64; `delete_resource` returns `{ "success": true, "level", "resourceId" }`. |
| `error` | `Error` | Emitted on missing fields, invalid JSON or HTTP errors (`Orthanc API Error (<status>): ...` / `Orthanc Error: Not Found`). |

### Output Examples

#### `upload_instance`

```json
{
  "ID": "f689ddd2-662f8fe1-8b18180d-ec2a2cee-937917af",
  "ParentPatient": "fa558bce-587a86d3-ad0da9b3-9d043d9d-4f5c5718",
  "ParentSeries": "93034833-163e42c3-bc9a428b-194620cf-2c5799e5",
  "ParentStudy": "8a8cf898-ca27c490-d0c7058c-929d0581-2bbf104d",
  "Path": "/instances/f689ddd2-662f8fe1-8b18180d-ec2a2cee-937917af",
  "Status": "Success"
}
```

#### `find_resources` (level `Study`, query `{"PatientID": "FUSION001"}`)

```json
[
  {
    "ID": "32d2ea17-fe55e783-e93aaa94-8d50d139-db1ed569",
    "Type": "Study",
    "MainDicomTags": {
      "AccessionNumber": "ACC001",
      "StudyDate": "20261122",
      "StudyDescription": "Fusion test study"
    },
    "PatientMainDicomTags": {
      "PatientID": "FUSION001",
      "PatientName": "Test^Fusion"
    },
    "ParentPatient": "0d895fcb-91e17c46-464ac59f-6bf33c17-10c18ed4",
    "Series": ["3f67aa72-19476a00-8c6371f5-c91c2fef-56b362d3"]
  }
]
```

#### `anonymize` / `modify`

```json
{
  "Description": "REST API",
  "ID": "3a1e45a2-ad3d1fc2-5c4f4900-0607d860-6e4d6ab2",
  "IsAnonymization": true,
  "InstancesCount": 1,
  "FailedInstancesCount": 0,
  "ParentResources": ["32d2ea17-fe55e783-e93aaa94-8d50d139-db1ed569"],
  "Path": "/studies/3a1e45a2-ad3d1fc2-5c4f4900-0607d860-6e4d6ab2"
}
```

#### `send_to_modality`

```json
{
  "Description": "REST API",
  "LocalAet": "ORTHANC",
  "RemoteAet": "ORTHANC",
  "InstancesCount": 1,
  "FailedInstancesCount": 0,
  "ParentResources": ["8a8cf898-ca27c490-d0c7058c-929d0581-2bbf104d"]
}
```

#### `get_file`

```json
{ "fileType": "preview_png", "data": "iVBORw0KGgoAAAANSUhEUgAA..." }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Orthanc PACS in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Orthanc PACS Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Find a patient's studies

```json
{
  "orthancUrl": "https://pacs.example.com",
  "username": "{{ secrets.ORTHANC_USER }}",
  "password": "{{ secrets.ORTHANC_PASSWORD }}",
  "operation": "find_resources",
  "level": "Study",
  "query": "{\"PatientID\": \"{{ input.patientId }}\"}"
}
```

#### Anonymize a study

```json
{
  "operation": "anonymize",
  "resourceId": "{{ input.studyId }}",
  "modifications": "{\"Keep\": [\"SeriesDescription\"]}"
}
```

#### Route a study to another PACS

```json
{
  "operation": "send_to_modality",
  "targetModality": "target",
  "resourcesToSend": "[\"{{ input.studyId }}\"]"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `username` and `password` in Fusion's **Secrets** vault. Do not commit them into version control.

- DICOM files contain **patient health data**: use HTTPS, keep the server private, and anonymize studies before sharing them outside the care team.
- Always set a username and password on the Orthanc server: without them, anyone who can reach the URL can read and delete images.
- `delete_resource` is permanent: protect it from repeated triggers.
- `upload_instance` downloads the file from `fileUrl`: only use URLs you trust.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The output is `{"success": true}` for `system_info` or `find_resources`
- **Cause:** The server requires authentication and `username` / `password` are empty. Orthanc answers 401 with an empty body.
- **Solution:** Fill in both `username` and `password`.

#### `Orthanc Error: Not Found` on `get_file`
- **Cause:** `resourceId` is a study or series ID; `get_file` needs an **instance** ID.
- **Solution:** Run `find_resources` with level `Instance`, or read `ID` from `upload_instance`.

#### `Resource ID not found in Patients, Studies, Series, or Instances.`
- **Cause:** Wrong `resourceId`, or the resource was deleted (or wrong credentials).
- **Solution:** Get the ID again with `find_resources`.

#### `Failed to anonymize` / `Failed to modify`
- **Cause:** The ID is not a study or series ID, or the rules are invalid.
- **Solution:** Use a study/series ID and check the JSON rules.

#### `Failed to download file from: ...`
- **Cause:** `fileUrl` is not publicly reachable.
- **Solution:** Use a public direct link to the `.dcm` file.

#### `Invalid JSON in query` (or `modifications`, `resourcesToSend`)
- **Cause:** The field is not valid JSON (e.g. single quotes).
- **Solution:** Use double quotes, e.g. `["id1", "id2"]`.

#### `send_to_modality` fails or `FailedInstancesCount` > 0
- **Cause:** The modality is not declared, or the target refuses the connection (host, port or AET).
- **Solution:** Check `GET /modalities` and the target's DICOM settings.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-22 | Initial documentation: 9 operations, parameters, connection, modalities, output examples, troubleshooting |

<!-- /SECTION: changelog -->

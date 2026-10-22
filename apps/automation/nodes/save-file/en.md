---
  node_id: "save-file"
  title: "Save file"
  description: "Keep a file a step made as one of your files, in a folder you choose, for your other workflows to use."
  category: "storage"
  subcategory: "files"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-10-22"
  author: "Fusion Team"
  tags:
    - file
    - save
    - keep
    - storage
  related_nodes:
    - write-file
    - read-file
---

<!-- SECTION: overview -->
  # Save file

  > **Category:** Storage & Files&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Files a run makes (a webhook upload, a **Write file** output) are removed with the run's outputs after a while. **Save file** keeps one as one of your files: it appears in the explorer, in the folder you chose, and any of your workflows can use it. Only you see your files.

### Use Cases

- Keep the report a workflow builds every month, replacing last month's
- Collect uploads from a form into a folder
- Hand a file from one workflow to another

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `file` | `file` | ✅ Yes | — | The file to keep. Usually an earlier step's file in expression mode, e.g. `$files.file` of a **Write file** step. |
| `folder` | `folder` | ❌ No | beside this workflow | **Choose folder** opens your drives and folders. |
| `name` | `string` | ❌ No | the file's own | The name to keep it under. No `/ \ : * ? " < > |`, at most 255 bytes, not starting or ending with a space or a dot. |
| `ifExists` | `enum` | ✅ Yes | `rename` | When the name is taken there: `rename` keeps both, numbering this one `name (2).ext`; `replace` gives that file this content (workflows pointing at it get the new content); `fail` stops with an error. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

**Success output:** the kept file, `{ "$files": { "file": { "id", "name", "mimeType", "size" } } }`, shown as a file card.

**Error output:** the file is not available to this workflow; the name is not allowed; the name is taken and `ifExists` is `fail`; the file is larger than the platform's limit.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Example

**Interval → Write file → Save file**: every month, **Write file** builds `report.csv`; **Save file** keeps it in *Reports* with `ifExists: replace`, so a **Read file** in another workflow always reads the latest.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

- **"There is already a file named … here."** The name is taken and `ifExists` is `fail`.
- **"File … is not available to this workflow"**: the file was removed, or is not this workflow's to read.
- **"Choose a folder to save the file in."** The workflow is not filed anywhere; choose a folder.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- **Write file**: make a file from data
- **Read file**: read one of your files, or one an earlier step attached

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

- **1.0.0** (2026-10-22): first version.

<!-- /SECTION: changelog -->

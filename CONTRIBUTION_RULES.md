# Contribution Rules

## General Principles
- Keep changes small and focused
- Follow the existing project structure and conventions
- Avoid unnecessary complexity
- Always check for project-specific contribution rules (if they exist) and follow them first

## Code Quality
- Ensure code passes all build checks before submitting
- Run the project locally before opening a pull request
- Use clear and meaningful commit messages

## Node Documentation

A node's page lives at `apps/automation/nodes/<node_id>/en.md`, with an optional
`icon.svg` beside it. **`<node_id>` must match the key the package exports in its
`nodes` map exactly** — the platform looks the folder up by that id, so a page
filed under a different spelling silently resolves to nothing.

Copy an existing page (`merge` is a good template) and keep the `<!-- SECTION: … -->`
markers paired. To show the node in the builder's palette, also add its id to the
right group → subgroup `nodes` array in `apps/automation/groups.json`.

### Every node must ship an example workflow

**This is required, not optional.** Whoever writes the node writes the example.
Configuration tables tell a reader what the fields are; they don't tell them what
the node is *for* or what to wire it to. The example does.

Put an exported graph next to the page and reference it with a `fusion-workflow`
fence:

````markdown
### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate a CSV and inspect it
```
````

`src` resolves relative to the node's own doc folder. `title` is optional but
worth setting — it's what the reader sees above the preview.

Guidelines:

- **Keep it small.** Two to five nodes. It should be legible at a glance, not a
  production workflow.
- **Show the node in context**, with whatever feeds it and whatever consumes its
  output — a node alone on the canvas teaches nothing.
- **Every node needs a `position`**, or the preview can't lay the graph out.
- **Connection handles must exist on the node.** A `sourceHandle` or
  `targetHandle` that doesn't match a real output/input is dropped silently and
  you get half a diagram.
- **Strip credentials before committing.** A raw builder export carries
  `secrets`, `variables` and every node's `data.parameters` — for some nodes that
  means real tokens. The docs repo is served openly, so an unedited export
  publishes them. The preview ignores those fields, but the file still contains
  them.

## Security
- Do not commit secrets, tokens, or sensitive data
- Use `.env` files for environment variables
- Never commit a raw workflow export without stripping `secrets`, `variables`
  and node `parameters` first (see Node Documentation above)

## Review Expectations
- Ensure changes are tested and functional
- Be ready to update your changes based on review feedback

## Development Setup
See [DEVELOPMENT_REQUIREMENTS.md](./DEVELOPMENT_REQUIREMENTS.md) for environment and tooling requirements.

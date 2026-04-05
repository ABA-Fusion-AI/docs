# src/routes/graph-route.ts

Express router exposing CRUD operations for "graphs" with authentication.

Export
- `graphRouter`: Express.Router

Purpose
- Provide REST endpoints to create, read, update and delete graph resources.
- All operations are scoped to authenticated users via `req.session.user.id`.
- Handlers delegate persistence to a `db` module.

Endpoints
- GET /
    - Behavior: return all graphs for the authenticated user
    - Success: 200 JSON array of graph objects
    - Auth: requires `req.session.user.id`

- POST /export
    - Body: `{ name?: string, ids: string[] }` (JSON)
    - Behavior: export multiple graphs as downloadable JSON file
    - Validates input with validator schema
    - Filename format: `{name}_{YYYY-MM-DD}.json` or `{graphName}_{YYYY-MM-DD}.json` for single graph
    - Success: 200 JSON file download with `Content-Disposition: attachment`
    - Error: 404 if no graphs found to export
    - Auth: filters graphs by `req.session.user.id`

- POST /import
    - Body: array of graph objects `TGraphDbEntry[]` (JSON)
    - Behavior: bulk import graphs for the authenticated user via `db.addGraph(graph, userId)` for each graph
    - Success: 201 JSON array of created graphs
    - Error: 401 if user not authenticated
    - Auth: requires `req.session.user.id`, imports scoped to user

- GET /export-code/:id
    - Params: `id` — graph identifier
    - Behavior: generate and download executable code as ZIP file via `generateCode(graph)`
    - Filename format: `exported-code-{id}.zip`
    - Success: 200 ZIP file download
    - Error: 404 if graph not found or unauthorized, 500 if code generation fails
    - Auth: requires graph ownership (`graph.userId === req.session.user.id`)

- POST /
    - Body: graph object (JSON)
    - Behavior: create a new graph for authenticated user via `db.addGraph(graph, userId)`
    - Success: 201 JSON created graph
    - Error: 401 if user not authenticated, 400 for invalid input
    - Auth: requires `req.session.user.id`

- GET /:id
    - Params: `id` — graph identifier
    - Behavior: return graph by ID via `db.getGraphById(id)`
    - Success: 200 JSON graph object
    - Error: 404 if graph not found or unauthorized
    - Auth: requires graph ownership (`graph.userId === req.session.user.id`)

- PATCH /:id
    - Params: `id` — graph identifier
    - Body: updated graph object (JSON)
    - Behavior: persist updates via `db.updateGraph(graphId, updatedGraph)`
    - Success: 200 JSON updated graph object
    - Error: 404 if graph not found or unauthorized
    - Auth: requires graph ownership

- DELETE /delete-many
    - Body: `{ ids: string[] }` (JSON)
    - Behavior: bulk delete graphs via `db.deleteGraphsByIds(ids, userId)`
    - Success: 200 JSON `{ deletedCount: number }`
    - Auth: scoped to user's graphs only

- DELETE /:id
    - Params: `id` — graph identifier
    - Behavior: delete graph via `db.deleteGraph(id)`
    - Success: 204 No Content on successful deletion
    - Error: 404 if graph not found or unauthorized
    - Auth: requires graph ownership

Notes
- Expected db API:
    - `db.getGraphs(userId: string, ids?: string[]) => Promise<Graph[]>`
    - `db.addGraph(graph: Graph, userId: string) => Promise<Graph>`
    - `db.getGraphById(id: string) => Promise<Graph | null>`
    - `db.updateGraph(graphId: string, graph: Graph) => Promise<Graph | null>`
    - `db.deleteGraph(id: string) => Promise<boolean>`
    - `db.deleteGraphsByIds(ids: string[], userId: string) => Promise<number>`
- All routes require session authentication with `req.session.user.id`
- Uses custom error classes: `NotFoundError`, `UnauthorizedError`
- Input validation handled by `validator` package (imported as `v`)
- Request bodies must be parsed with `express.json()` (or equivalent)

# src/routes/template-route.ts

Minimal router scaffold for template-related endpoints.

Export
- `templateRouter`: Express.Router

Purpose
- Placeholder router to group template CRUD routes.
- Intended to be extended with route handlers (GET, POST, PUT, DELETE) for template resources.

Usage
- Mount on an Express app:
    - `app.use('/templates', templateRouter)`

Implementation notes / TODO
- Add handlers and validation for:
    - GET / (list templates, optional filters)
    - POST / (create template)
    - GET /:id (read template)
    - PUT /:id (update template)
    - DELETE /:id (remove template)
- Apply middleware as needed (body parsing, auth, input validation).
- Define and use a `db` or service module for persistence similar to `graph-route`.
- Include consistent success and error response shapes (use `error` property for failures).

Type hints
- Document or import a `Template` type/interface describing the resource shape for consistent validation and typing.
- Ensure route handlers return appropriate HTTP status codes: 200/201/204 on success, 400/404/500 on errors.

Example (one-line)
- Add a basic GET: `templateRouter.get('/', async (req, res) => { /* fetch templates and res.json */ })`
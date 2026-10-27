# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Small Express REST API (users + health check) used as the base project for the Claude Code course.

## Commands

- `npm run dev` — start the API on http://localhost:3000 with auto-reload (`node --watch`); `PORT` overrides the port
- `npm test` — run all tests with Node's built-in runner (`node --test`)
- `node --test --test-name-pattern="404" tests/users.test.js` — run a single test by name
- `npm run lint` — ESLint (v8, `.eslintrc.json`); CI runs lint then test on Node 22

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`/users`, `/health`), and exports `app`. It only calls `listen()` when run directly (`require.main === module`), so tests import `app` without opening a port.
- `routes/<resource>.js` — one `express.Router()` per resource. A new resource needs a new file here plus an `app.use()` line in `server.js`.
- `db/store.js` — the only data access layer: an in-memory array that resets on restart. Routes call store functions; they never hold data themselves.

## Conventions

- CommonJS only (`require` / `module.exports`); ESLint is configured with `sourceType: "script"`, so `import`/`export` will fail lint.
- Data access goes through `db/store.js` functions, not direct array manipulation in routes.
- Errors are returned as `res.status(code).json({ error: "message" })`, with an early `return`: 400 for missing/invalid input, 404 for a missing resource, 201 for a successful create.
- Route `:id` params are strings — convert with `Number(req.params.id)` before calling the store.
- Tests use `node:test` + `node:assert` + `supertest` against the exported `app` (not Jest/Mocha). The store is shared module state across tests in a file, so don't assume fixed IDs or counts after a `POST`.
- Unused `req`, `res`, `next`, or `_` params are allowed by lint; other unused vars warn.

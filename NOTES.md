# Notes: setting up Claude Code on this project

## What's in CLAUDE.md, and what I left out

I kept four short sections: a one-line description, the commands I actually run (including how to run a single test, which isn't in the README), the architecture (how `server.js`, `routes/` and `db/store.js` connect, and what adding a new resource involves), and conventions. Each convention prevents a specific mistake Claude would otherwise make here: writing ES `import` syntax (fails lint because ESLint uses `sourceType: "script"`), writing Jest tests (only `node:test` + `supertest` are installed), forgetting `Number(req.params.id)`, or assuming fixed user IDs in tests even though the in-memory store is shared across tests.

I deliberately left out the endpoint list and file tree (Claude can read `routes/` directly), the install and setup steps from the README (one-off, not needed every session), and anything about `.env` or real config values (sensitive, and the app doesn't use secrets). Anything Claude can discover in a few seconds just costs context on every session.

## Permission rules

- **allow:** `npm test`, `npm run lint`, `node --test`, and read-only git (`status`, `diff`, `log`). These are safe, run constantly, and prompting for them every time adds nothing.
- **ask:** `git push` and `npm install`, because one publishes outside my machine and the other changes dependencies and `package-lock.json`.
- **deny:** `Read(./.env)` and `Bash(cat .env:*)`, `git push --force` / `-f`, and `rm -rf`.

Without the deny rules, Claude could read `.env` while debugging config and echo real secrets into the conversation, or "fix" a rejected push by force-pushing and overwriting other people's commits. `Read(./.env)` alone only blocks the Read tool, which is why I also denied `cat .env`. That is still best-effort (other shell commands could read the file), so the real protection is keeping `.env` out of git.

While setting up, I found that `.gitignore` had leading spaces on most lines. Git treats those spaces as part of the pattern, so `.env` and `.claude/settings.local.json` were **not** being ignored. I removed the spaces and checked with `git check-ignore`.

## Verification

<!-- TODO: fill in after running these in a fresh session -->
- `/memory`: CLAUDE.md shows as loaded — _confirmed / not yet_
- `/permissions`: allow / ask / deny rules listed — _confirmed / not yet_
- Asked "How do I run the tests here?" — Claude answered from CLAUDE.md — _confirmed / not yet_

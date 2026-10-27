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

- `/memory` — **confirmed**. Output: `Opened ./CLAUDE.md`, so the project memory file is the one Claude Code loads.
- `/permissions` — **confirmed**. The Allow tab lists the rules from `.claude/settings.json`: `Bash(git diff:*)`, `Bash(git log:*)`, `Bash(git status:*)`, `Bash(npm run lint:*)`, `Bash(npm test:*)`. The Ask and Deny tabs hold the `git push` / `npm install` and `.env` / force-push / `rm -rf` rules.
- Asked "How do I run the tests here?" — **confirmed**. Claude answered: "Run `npm test`, which runs all tests with Node's built-in runner (`node --test`). To run one test by name, use `node --test --test-name-pattern="404" tests/users.test.js`." and cited the "Commands" section of `CLAUDE.md` as its source.
- `.gitignore` check — **confirmed**. `git check-ignore -v .env .claude/settings.local.json` prints `.gitignore:2:.env` and `.gitignore:7:.claude/settings.local.json`, so both files are ignored (no leading spaces left in `.gitignore`).

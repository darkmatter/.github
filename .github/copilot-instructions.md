# Darkmatter — GitHub Copilot Instructions

## Skills and ADRs

Reusable agent skills and architecture decision records live in [darkmatter/skills](https://github.com/darkmatter/skills).

- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

## Architecture decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| ADR-0001 | Use `bd` (beads) for task tracking and agent memory. No `TodoWrite` or `MEMORY.md` for cross-session state. |
| ADR-0002 | Every repo exposes `install`, `setup`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>` |
| ADR-0003 | Cross-language types use Protobuf + `buf`. Default transport: ConnectRPC. Commit generated code. |
| ADR-0004a | No reinvention — check for existing libraries before implementing |
| ADR-0004b | One typed `src/settings.<ext>` per binary; no scattered `process.env` reads; fail fast at startup |
| OTel | App code imports only OTel SDKs; provider wiring lives in shared packages |

## Always apply

- `coding-standards` on any code task
- `brainstorming` before implementing anything non-trivial
- `test-driven-development` before writing implementation code
- `systematic-debugging` before proposing fixes
- `verification-before-completion` before claiming work is done

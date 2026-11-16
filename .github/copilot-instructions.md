# Darkmatter — GitHub Copilot & Codex Instructions

## Skills and ADRs

Reusable agent skills and architecture decision records live in [darkmatter/skills](https://github.com/darkmatter/skills).

- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

## Architecture decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| ADR-0001 | Use `bd` (beads) for task tracking and agent memory. No `TodoWrite` or `MEMORY.md` for cross-session state. `bd prime` at session start. |
| ADR-0002 | Every repo exposes `install`, `setup`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| ADR-0003 | Cross-language types use Protobuf + `buf`. Default transport: ConnectRPC. Commit generated code. `buf lint` + `buf breaking` in CI. |
| ADR-0004 | No reinvention — check for existing libraries before implementing. A dependency beats private code. |
| ADR-0005 | One typed `src/settings.<ext>` per binary. Only place that reads raw env vars. Validates at startup. Secret values use redacted wrappers — never plain strings. |
| ADR-0006 | READMEs follow Standard Readme. Required: title, install (copy/paste-able), usage with quickstart, ADR-0002 command surface, config/secrets docs, verify command, contributing, license. No unexplained placeholders. |
| ADR-0007 | No inline SQL in TypeScript — no `sql<Row>\`...\`` or raw strings. Use Kysely (preferred) or Drizzle. Improve the abstraction before falling back to inline SQL. |
| OTel | App code imports only OTel SDKs; provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

## Always apply

- `coding-standards` — any TypeScript/JS/React/Node code task
- `brainstorming` — before implementing anything non-trivial
- `test-driven-development` — before writing implementation code
- `systematic-debugging` — before proposing fixes
- `diagnose` — hard bugs and performance regressions (reproduce → minimise → hypothesise → instrument → fix)
- `verification-before-completion` — before claiming work is done
- `definition-of-done` — any complex multi-step task

## Key skills by category

**Task management:** `beads-setup` (no `.beads/`?), `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `finishing-a-development-branch`, `triage`, `handoff`

**Design & planning:** `grill-me` (stress-test a plan), `grill-with-docs` (grill + persist decisions to CONTEXT.md/ADRs), `prototype` (throwaway logic or UI prototype), `zoom-out` (map modules and callers)

**Code quality:** `requesting-code-review`, `receiving-code-review`, `codebase-cleanup`, `improve-codebase-architecture`, `end-of-turn-review`, `writing-skills`

**Architecture:** `effect-typescript`, `alchemy`, `nix-flake-organization`, `sops-secret-access`, `repository-organization`, `rust-best-practices`

**UI/Frontend:** `frontend-design`, `ui-ux-pro-max`, `vercel-react-best-practices`, `nextjs-to-rwsdk-migration`, `kickoff-dm-design`, `shadcn-registry-first`, `ui-component-architecture`, `run-ui-registry-variations`

**Browser automation:** `browser-use` (Python, persistent sessions), `agent-browser` (CDP, Node/Rust)

**Communication:** `caveman`, `caveman-commit` (compact commit messages), `caveman-review` (compact reviews), `compress`

**Domain:** `neon-postgres`, `openchronicle-setup`, `hl-funding-analysis`

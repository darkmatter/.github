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
| ADR-0006 | READMEs follow Standard Readme: title, background if needed, TOC over 100 lines, copy-pasteable install + usage, documented command surface (ADR-0002), config/secrets docs, verification command, contributing, license last. |
| ADR-0007 | TypeScript MUST NOT embed SQL as inline strings/template literals (incl. `sql<Row>` tags). Use a type-checked query builder — preference order Kysely > Drizzle > comparable alternatives. |
| ADR-0008 | App code imports only OTel SDKs; provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

## Always apply

- `brainstorming` — before implementing anything non-trivial
- `test-driven-development` — before writing implementation code (`tdd` is an overlapping, equally valid entry point)
- `definition-of-done` — any complex multi-step task

Note: earlier versions of this file also listed `coding-standards`, `systematic-debugging`, and `verification-before-completion` as always-apply skills. None of these exist as a skill directory in darkmatter/skills — see "Known gaps" in [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md). Don't invoke them by name until they're built.

## Key skills by category

**Task management:** `beads-setup` (no `.beads/`?), `finishing-a-development-branch`, `handoff`

**Code quality:** `diagnose`, `requesting-code-review`, `receiving-code-review`, `codebase-cleanup`, `end-of-turn-review`, `writing-skills`, `improve-codebase-architecture`

**Architecture:** `effect-typescript`, `alchemy`, `nix-flake-organization`, `sops-secret-access`, `repository-organization`, `choose-dev-entrypoints`

**UI/Frontend:** `ui-ux-pro-max`, `vercel-react-best-practices`, `nextjs-to-rwsdk-migration`, `ui-component-architecture`, `shadcn-registry-first`, `kickoff-dm-design` (manual), `run-ui-registry-variations` (manual)

**Browser automation:** `agent-browser` (CDP, Node/Rust)

**Communication:** `caveman-commit` (compact commit messages)

**Domain:** `openchronicle-setup`, `rust-best-practices`, `run-meeting-summary` (manual)

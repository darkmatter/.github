# Darkmatter — GitHub Copilot & Codex Instructions

## Skills and ADRs

Reusable agent skills and architecture decision records live in [darkmatter/skills](https://github.com/darkmatter/skills).

- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

## Architecture decisions (apply to all repos)

> **ADR-0001 removed.** "Use `bd` (beads)" was retracted upstream (`darkmatter/skills` commit `f5f5227`, 2026-07-09) along with the `beads-setup`/`beads-linear-sync` skills and all `.beads/` state. There is no org-standard tracker right now — use whatever native task tracking Codex/Copilot exposes, or the project's own documented tool.

| ADR | Rule |
|-----|------|
| ADR-0002 | Every repo exposes `install`, `setup`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| ADR-0003 | Cross-language types use Protobuf + `buf`. Default transport: ConnectRPC. Commit generated code. `buf lint` + `buf breaking` in CI. |
| ADR-0004 | No reinvention — check for existing libraries before implementing. A dependency beats private code. |
| ADR-0005 | One typed `src/settings.<ext>` per binary. Only place that reads raw env vars. Validates at startup. Secret values use redacted wrappers — never plain strings. |
| ADR-0006 | READMEs follow Standard Readme: title/description, TOC over 100 lines, copy/paste-able install + usage, dev command surface (ADR-0002), config/secrets, verification command, contributing, license last. |
| ADR-0007 | No inline SQL in TypeScript (including `sql<Row>\`...\``). Use a type-checked query builder — Kysely preferred, Drizzle allowed but not preferred for complex joins. |
| ADR-0008 | Preferred per-language conventions live as exemplar code in `references/<lang>/` (`rust`, `go`, `typescript`) in `darkmatter/skills`, not just prose. Precedence: project `.agent/` → `references/` → general idiom. |
| OTel | App code imports only OTel SDKs; provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

## Always apply

- `definition-of-done` — any complex multi-step task
- `diagnose` — before proposing fixes for bugs or performance regressions
- `tdd` — when the user explicitly asks for a TDD/red-green workflow

## Key skills by category

**Task and workflow:** `choose-dev-entrypoints`, `codebase-cleanup`, `finishing-a-development-branch`, `find-skills`, `grill-me`, `grill-with-docs`, `handoff`, `improve-codebase-architecture`, `prototype`, `repository-organization`, `writing-skills`, `zoom-out`

**Architecture:** `effect-typescript`, `alchemy`, `nix-flake-organization`, `sops-secret-access`, `darkmatter-gitops-conventions`, `darkmatter-ts-toolchain`

**UI/Frontend:** `ui-ux-pro-max`, `vercel-react-best-practices`, `ui-component-architecture`, `shadcn-registry-first`, `run-ui-registry-variations`, `nextjs-to-rwsdk-migration`

**Browser automation:** `agent-browser` (CDP, Node/Rust)

**Other:** `rust-best-practices`, `session-context-pipeline`

As of 2026-07-27, these are no longer shipped upstream — don't suggest them: `beads-setup`, `beads-linear-sync`, `brainstorming`, `caveman`/`caveman-commit`/`caveman-review`/`compress`, `continuous-learning`, `dm-skill-creator`, `end-of-turn-review`, `kickoff-dm-design`, `receiving-code-review`, `requesting-code-review`, `run-meeting-summary`, `test-driven-development`, `triage`, `coding-standards`, `frontend-design`, `browser-use`, `neon-postgres`, `openchronicle-setup`, `hl-funding-analysis`.

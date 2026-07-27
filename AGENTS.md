# Darkmatter — Org-Level Agent Context

This file provides context for all AI agents (Claude Code, GitHub Copilot, Codex) working in darkmatter repositories.

Full details: [darkmatter/skills](https://github.com/darkmatter/skills)
- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

---

## Architecture Decisions (apply to all repos)

> **ADR-0001 removed.** "Beads is the standard task tracker" was retracted upstream (`darkmatter/skills` commit `f5f5227`, 2026-07-09) along with the `beads-setup`/`beads-linear-sync` skills and all `.beads/` state. There is no org-standard tracker right now — use the harness's native task tracking or the project's own documented tool.

| ADR | Rule |
|-----|------|
| [0002](https://github.com/darkmatter/skills/blob/main/docs/adr/0002-standard-project-command-surface.md) | Every repo has `install`, `setup`, `server`/`run`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| [0003](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md) | Cross-language types use **Protobuf + `buf`**. Default transport: **ConnectRPC**. Generated code is committed. `buf lint` + `buf breaking` in CI. |
| [0004](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md) | **No reinvention.** Check for existing libraries before implementing. A dependency beats a private reimplementation. |
| [0005](https://github.com/darkmatter/skills/blob/main/docs/adr/0005-typed-settings-module-decoupled-from-provider.md) | **One typed `src/settings.<ext>`** per binary. Only place that reads raw env. Validates at startup. Decoupled from provider. Secret values must use redacted wrappers (`Config.redacted`, `SecretStr`, `secrecy::Secret<T>`). |
| [0006](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md) | READMEs follow **Standard Readme**: title/description, TOC over 100 lines, copy/paste-able install + usage, dev command surface (ADR-0002), config/secrets notes, verification command, contributing, license last. |
| [0007](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md) | **No inline SQL in TypeScript** (including `sql<Row>\`...\``). Use a type-checked query builder — **Kysely** preferred, Drizzle allowed but not preferred for complex joins. |
| [0008](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md) | Preferred per-language conventions live as exemplar code in [`references/<lang>/`](https://github.com/darkmatter/skills/tree/main/references) (`rust`, `go`, `typescript`), not just prose. Precedence: project `.agent/` → `references/` → general idiom. |
| OTel | App code imports only **OpenTelemetry SDKs**. Provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

---

## Skills to apply proactively

Current as of 2026-07-27 — reflects what's actually shipped from `darkmatter/skills/skills/` (excludes `inactive/` and the archived `docs/00-inbox/skills/`).

**Task and workflow:** `choose-dev-entrypoints`, `codebase-cleanup`, `definition-of-done`, `diagnose` (bug/perf diagnosis), `find-skills`, `finishing-a-development-branch`, `grill-me`, `grill-with-docs`, `handoff`, `improve-codebase-architecture`, `prototype`, `repository-organization`, `tdd`, `writing-skills`, `zoom-out`

**Architecture:** `effect-typescript`, `alchemy`, `nix-flake-organization`, `sops-secret-access`, `darkmatter-gitops-conventions`, `darkmatter-ts-toolchain`

**UI:** `ui-ux-pro-max`, `vercel-react-best-practices`, `ui-component-architecture`, `shadcn-registry-first`, `run-ui-registry-variations`, `nextjs-to-rwsdk-migration`

**Browser:** `agent-browser` (CDP/Node/Rust)

**Other:** `rust-best-practices`, `session-context-pipeline`

**Runtime policies (auto):** `using-superpowers` (session start), `strategic-compact` (long autonomous sessions)

**No longer shipped — don't reference as available:** `beads-setup`, `beads-linear-sync`, `brainstorming`, `caveman`/`caveman-commit`/`caveman-review`/`compress`, `continuous-learning`, `dm-skill-creator`, `end-of-turn-review`, `kickoff-dm-design`, `receiving-code-review`, `requesting-code-review`, `run-meeting-summary`, `test-driven-development`, `triage`, `coding-standards`, `frontend-design`, `browser-use`, `neon-postgres`, `openchronicle-setup`, `hl-funding-analysis`, `systematic-debugging`, `verification-before-completion`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`.

---

## Per-repo agent context

Every darkmatter project repo has its own `AGENTS.md` and `.agent/` directory with project-specific state and decisions. This org-level file is the floor; project-level files are the ceiling.

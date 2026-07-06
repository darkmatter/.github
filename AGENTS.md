# Darkmatter — Org-Level Agent Context

This file provides context for all AI agents (Claude Code, GitHub Copilot, Codex) working in darkmatter repositories.

Full details: [darkmatter/skills](https://github.com/darkmatter/skills)
- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

---

## Architecture Decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| [0001](https://github.com/darkmatter/skills/blob/main/docs/adr/0001-beads-as-task-tracker-and-agent-memory.md) | **Beads (`bd`)** for task tracking and agent memory. `bd prime` to load context; `bd remember` to save; `bd create`/`bd close` for tasks. No `TodoWrite` or `MEMORY.md` for cross-session state. |
| [0002](https://github.com/darkmatter/skills/blob/main/docs/adr/0002-standard-project-command-surface.md) | Every repo has `install`, `setup`, `server`/`run`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| [0003](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md) | Cross-language types use **Protobuf + `buf`**. Default transport: **ConnectRPC**. Generated code is committed. `buf lint` + `buf breaking` in CI. |
| [0004](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md) | **No reinvention.** Check for existing libraries before implementing. A dependency beats a private reimplementation. |
| [0005](https://github.com/darkmatter/skills/blob/main/docs/adr/0005-typed-settings-module-decoupled-from-provider.md) | **One typed `src/settings.<ext>`** per binary. Only place that reads raw env. Validates at startup. Decoupled from provider. Secret values must use redacted wrappers (`Config.redacted`, `SecretStr`, `secrecy::Secret<T>`). |
| [0006](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md) | READMEs follow **Standard Readme**: title, background if needed, TOC over 100 lines, copy-pasteable install + usage, documented command surface (ADR-0002), config/secrets docs, a verification command, contributing, license last. |
| [0007](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md) | TypeScript MUST NOT embed SQL as inline strings/template literals (including `sql<Row>` tags). Use a **type-checked query builder**, preference order **Kysely > Drizzle > other builders with comparable compile-time checking**. |
| [0008](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-decouple-telemetry-concerns.md) | App code imports only **OpenTelemetry SDKs**. Provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

---

## Skills to apply proactively

The following are confirmed present as `skills/<name>/` directories in darkmatter/skills. If you're tempted to invoke `coding-standards`, `systematic-debugging`, `verification-before-completion`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `frontend-design`, `browser-use`, `caveman`, `caveman-review`, `compress`, `neon-postgres`, or `hl-funding-analysis` — none of these exist as a skill; see "Known gaps" in [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md).

**Always-on:** `brainstorming`, `test-driven-development` (or its overlapping twin `tdd`), `definition-of-done`

**Task management:** `beads-setup` (when `.beads/` missing), `beads-linear-sync`, `finishing-a-development-branch`, `handoff`

**Debugging & code quality:** `diagnose`, `requesting-code-review`, `receiving-code-review`, `codebase-cleanup`, `repository-organization`, `end-of-turn-review`, `writing-skills`, `improve-codebase-architecture`

**Architecture:** `effect-typescript`, `alchemy`, `nix-flake-organization`, `sops-secret-access`, `choose-dev-entrypoints`

**Workflow:** `dm-skill-creator`, `find-skills`, `grill-me`, `grill-with-docs`, `triage`, `zoom-out`, `prototype`

**UI:** `ui-ux-pro-max`, `vercel-react-best-practices`, `ui-component-architecture`, `shadcn-registry-first`, `kickoff-dm-design` (manual), `run-ui-registry-variations` (manual)

**Platform:** `nextjs-to-rwsdk-migration`, `openchronicle-setup`, `rust-best-practices`

**Browser:** `agent-browser` (CDP/Node/Rust)

**Communication:** `caveman-commit`

**Manual-invocation only:** `run-meeting-summary`, `kickoff-dm-design`, `run-ui-registry-variations`

**Runtime policies (auto):** `using-superpowers` (session start), `continuous-learning` (session end), `strategic-compact` (long autonomous sessions)

---

## Per-repo agent context

Every darkmatter project repo has its own `AGENTS.md` and `.agent/` directory with project-specific state and decisions. This org-level file is the floor; project-level files are the ceiling.

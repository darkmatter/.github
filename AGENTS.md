# Darkmatter — Org-Level Agent Context

This file provides context for all AI agents (Claude Code, GitHub Copilot, Codex) working in darkmatter repositories.

Full details: [darkmatter/skills](https://github.com/darkmatter/skills)
- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

---

## Architecture Decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| [0002](https://github.com/darkmatter/skills/blob/main/docs/adr/0002-standard-project-command-surface.md) | Every repo has `install`, `setup`, `server`/`run`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| [0003](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md) | Cross-language types use **Protobuf + `buf`**. Default transport: **ConnectRPC**. Generated code is committed. `buf lint` + `buf breaking` in CI. |
| [0004](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md) | **No reinvention.** Check for existing libraries before implementing. A dependency beats a private reimplementation. |
| [0005](https://github.com/darkmatter/skills/blob/main/docs/adr/0005-typed-settings-module-decoupled-from-provider.md) | **One typed `src/settings.<ext>`** per binary. Only place that reads raw env. Validates at startup. Decoupled from provider. Secret values must use redacted wrappers (`Config.redacted`, `SecretStr`, `secrecy::Secret<T>`). |
| [0006](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md) | **README minimum standard.** Follow [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) structure: title, install, usage, dev commands (aligns with ADR-0002), config/secrets, testing, contributing, license last. Copy/paste-able commands. |
| [0007](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md) | **Type-checked SQL in TypeScript.** No inline SQL strings or tagged templates (`sql<Row>\`...\``). Use a schema-derived query builder/ORM. Prefer **Kysely**; **Drizzle** allowed when already present. |
| [0008](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md) | **Per-language reference codebases** under `references/` (currently `rust/`, `go/`, `typescript/`). Skills carry prose; references carry code. Precedence: project `.agent/` → `references/` → general idiom. |
| OTel | App code imports only **OpenTelemetry SDKs**. Provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

---

## Skills to apply proactively

**Always-on:** `coding-standards`, `brainstorming`, `test-driven-development`, `diagnose`, `definition-of-done`

**Architecture:** `effect-typescript`, `alchemy`, `darkmatter-ts-toolchain`, `darkmatter-gitops-conventions`, `nix-flake-organization`, `sops-secret-access`, `choose-dev-entrypoints`, `rust-best-practices`, `repository-organization`, `zoom-out`, `improve-codebase-architecture`

**Code quality:** `requesting-code-review`, `receiving-code-review`, `codebase-cleanup`, `end-of-turn-review`, `writing-skills`

**Workflow:** `session-context-pipeline`, `finishing-a-development-branch`, `handoff`, `triage`, `grill-me`, `grill-with-docs`, `dm-skill-creator`, `find-skills`, `run-meeting-summary`

**UI:** `frontend-design`, `ui-ux-pro-max`, `shadcn-registry-first`, `ui-component-architecture`, `vercel-react-best-practices`, `nextjs-to-rwsdk-migration`, `kickoff-dm-design`, `prototype`

**Platform:** `openchronicle-setup`, `neon-postgres`

**Browser:** `browser-use` (Python/persistent), `agent-browser` (CDP/Node/Rust)

**Communication:** `caveman`, `caveman-commit`, `caveman-review`, `compress`

**Runtime policies (auto):** `using-superpowers` (session start), `continuous-learning` (session end), `strategic-compact` (long autonomous sessions)

---

## Per-repo agent context

Every darkmatter project repo has its own `AGENTS.md` and `.agent/` directory with project-specific state and decisions. This org-level file is the floor; project-level files are the ceiling.

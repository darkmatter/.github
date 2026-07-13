# Darkmatter — Org-Level Agent Context

This file provides context for all AI agents (Claude Code, GitHub Copilot, Codex) working in darkmatter repositories.

Full details: [darkmatter/skills](https://github.com/darkmatter/skills)
- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

---

## Architecture Decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| [0002](https://github.com/darkmatter/skills/blob/main/docs/adr/0002-standard-project-command-surface.md) | Every repo has `install`, `setup`, `server`/`run`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. Exception: `turbo run <target>` in monorepos. |
| [0003](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md) | Cross-language types use **Protobuf + `buf`**. Default transport: **Connect (ConnectRPC)**. Generated code is committed. `buf lint` + `buf breaking` in CI. Exempt: libraries, services with <5 endpoints + single language + single first-party client, schema-as-code setups with no cross-language typed consumers. |
| [0004](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md) | **No reinvention.** Prefer structured tool output over parsing human-readable text. Search before writing anything >~20 lines in a well-known domain (encoding, parsing, crypto, date/time, protocols). A dependency beats a private reimplementation. |
| [0005](https://github.com/darkmatter/skills/blob/main/docs/adr/0005-typed-settings-module-decoupled-from-provider.md) | **One typed `src/settings.<ext>`** per binary. Only place that reads raw env (`process.env`, `Bun.env`, `os.environ`, `std::env::var`, …). Validates at startup, fails fast with all errors at once. Description/provider/runtime stay decoupled. Secrets MUST use redacted wrappers (`Config.redacted`, `SecretStr`, `secrecy::Secret<T>`) — plain `string` for a secret is a defect. |
| [0006](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md) | READMEs follow **Standard Readme** as the default shape: title/description, install, usage/quickstart, standard command surface (ADR-0002), config/secrets, verification, contributing, license last. TOC required over 100 lines. |
| [0007](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md) | TypeScript MUST NOT embed inline SQL (including tagged `sql<Row>\`...\`` templates) for application queries. Use a type-checked query builder — **Kysely preferred**; Drizzle allowed but weaker inference for complex query-heavy code. |
| [0008](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md) | Preferred per-language conventions live as exemplar code under [`references/<language>/`](https://github.com/darkmatter/skills/tree/main/references) (rust, go, typescript) — code shows, skills tell. Precedence when conventions conflict: project `.agent/` → `references/` → general language idiom. |
| OTel-only observability | App code depends only on OpenTelemetry SDKs. Provider-specific packages (`@sentry/*`, PostHog, Datadog) never appear in `apps/*`; provider wiring is isolated in shared packages. *(Team convention; not yet tracked as a numbered ADR in `darkmatter/skills`.)* |

**Note on task tracking:** Several skills and hooks (`darkmatter-gitops-conventions`, `.codex/hooks.json`) reference **Beads (`bd`)** as the task tracker / agent-memory convention (`bd prime`, `bd create`, `bd remember`). There is currently **no `docs/adr/0001-*` file for this** in `darkmatter/skills` — the ADR that used to document it is missing or was never finalized. Treat `bd` usage as an active team convention, not a citable ADR, until that gap is resolved upstream.

---

## Skills to apply proactively

Ground truth is the 43 skills actually shipped under [`skills/`](https://github.com/darkmatter/skills/tree/main/skills) in `darkmatter/skills`, per [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md).

**Always-on / process:** `brainstorming` (before creative work), `test-driven-development` (before implementation), `definition-of-done` (multi-step tasks), `tdd` (explicit red-green-refactor requests)

**Debugging & orientation:** `diagnose` (reproduce → minimise → hypothesise → fix), `zoom-out` (map callers/modules before diving in), `improve-codebase-architecture` (deepen shallow modules), `choose-dev-entrypoints` (Nix/Just/Bun/Turbo responsibility boundaries), `triage` (issue state machine)

**Code quality & review:** `codebase-cleanup` (multi-pass refactor sweep), `repository-organization` (layout, README, ADR placement), `requesting-code-review`, `receiving-code-review`, `end-of-turn-review` (second-opinion pass), `writing-skills`, `ui-component-architecture`, `rust-best-practices`

**Architecture & infrastructure:** `effect-typescript`, `alchemy` (Cloudflare/AWS via Alchemy v2), `nix-flake-organization`, `sops-secret-access`, `darkmatter-ts-toolchain` (Bun/tsgo/vitest/oxlint/Effect/Alchemy contract), `darkmatter-gitops-conventions` (darkmatter/gitops specifically)

**UI/Frontend:** `ui-ux-pro-max`, `vercel-react-best-practices`, `shadcn-registry-first`, `run-ui-registry-variations` (manual), `nextjs-to-rwsdk-migration`, `kickoff-dm-design` (manual)

**Browser automation:** `agent-browser` (Chrome/Chromium via CDP — Node/Rust)

**Workflow & collaboration:** `finishing-a-development-branch`, `dm-skill-creator`, `find-skills`, `run-meeting-summary` (manual), `handoff`, `grill-me`, `grill-with-docs`, `session-context-pipeline`, `prototype`

**Communication:** `caveman-commit` (ultra-compressed commit messages)

**Domain-specific:** `openchronicle-setup` (macOS local-first agent memory)

**Runtime policies (auto-applied by agent client):** `using-superpowers` (session start), `continuous-learning` (session end / Stop hook), `strategic-compact` (long autonomous sessions)

> Skills previously listed here that do **not** currently exist in `darkmatter/skills` (`skills/coding-standards`, `skills/browser-use`, `skills/caveman`, `skills/caveman-review`, `skills/compress`, `skills/frontend-design`, `skills/neon-postgres`, `skills/hl-funding-analysis`, plus `beads-setup`, `beads-linear-sync`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `systematic-debugging`, `verification-before-completion`) have been removed from this list. If any of these ship again, re-add them here and cite `docs/catalog.md`.

---

## Per-repo agent context

Every darkmatter project repo has its own `AGENTS.md` and `.agent/` directory with project-specific state and decisions. This org-level file is the floor; project-level files are the ceiling.

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
| [0006](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md) | Every non-trivial README follows **Standard Readme** structure. Must include: title, install, usage/quickstart, command surface (ADR-0002), config/secrets, verify command, contributing, license. All commands copy/paste-able from repo root. |
| OTel | App code imports only **OpenTelemetry SDKs**. Provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

---

## Skills to apply proactively

**Always-on:** `coding-standards`, `brainstorming`, `test-driven-development`, `diagnose` (hard bugs/regressions), `systematic-debugging`, `verification-before-completion`, `definition-of-done`

**Task management:** `beads-setup` (when `.beads/` missing), `beads-linear-sync`, `writing-plans`, `executing-plans`, `triage` (issue workflow), `handoff` (session continuity)

**Agent orchestration:** `subagent-driven-development`, `dispatching-parallel-agents`

**Code quality:** `requesting-code-review`, `receiving-code-review`, `codebase-cleanup`, `repository-organization`, `end-of-turn-review`, `writing-skills`

**Architecture:** `effect-typescript`, `alchemy`, `nix-flake-organization`, `sops-secret-access`, `improve-codebase-architecture`, `zoom-out`

**Design & discovery:** `grill-me`, `grill-with-docs`, `prototype`

**Workflow:** `finishing-a-development-branch`, `dm-skill-creator`, `find-skills`, `run-meeting-summary`

**UI:** `frontend-design`, `ui-ux-pro-max`, `vercel-react-best-practices`, `kickoff-dm-design`, `shadcn-registry-first`, `run-ui-registry-variations`, `ui-component-architecture`

**Platform:** `nextjs-to-rwsdk-migration`, `openchronicle-setup`, `neon-postgres`

**Browser:** `browser-use` (Python/persistent), `agent-browser` (CDP/Node/Rust)

**Communication:** `caveman`, `caveman-commit`, `caveman-review`, `compress`

**Runtime policies (auto):** `using-superpowers` (session start), `continuous-learning` (session end), `strategic-compact` (long autonomous sessions)

---

## Per-repo agent context

Every darkmatter project repo has its own `AGENTS.md` and `.agent/` directory with project-specific state and decisions. This org-level file is the floor; project-level files are the ceiling.

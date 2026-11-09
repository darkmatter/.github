# Darkmatter — Org-Level Agent Context

This file provides context for all AI agents (Claude Code, GitHub Copilot, Codex) working in darkmatter repositories.

Full details: [darkmatter/skills](https://github.com/darkmatter/skills)
- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)
- Shared instruction topics: [`docs/agents/`](https://github.com/darkmatter/skills/tree/main/docs/agents) (flake output `agentsMd`)

---

## Architecture Decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| [0002](https://github.com/darkmatter/skills/blob/main/docs/adr/0002-standard-project-command-surface.md) | Every repo has `install`, `setup`, `server`/`run`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| [0003](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md) | Cross-language types use **Protobuf + `buf`**. Default transport: **ConnectRPC**. Generated code is committed. `buf lint` + `buf breaking` in CI. |
| [0004](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md) | **No reinvention.** Check for existing libraries before implementing. A dependency beats a private reimplementation. |
| [0005](https://github.com/darkmatter/skills/blob/main/docs/adr/0005-typed-settings-module-decoupled-from-provider.md) | **One typed `src/settings.<ext>`** per binary. Only place that reads raw env. Validates at startup. Decoupled from provider. Secret values must use redacted wrappers (`Config.redacted`, `SecretStr`, `secrecy::Secret<T>`). |
| [0006](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md) | **README minimum standard.** Follow [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) structure: title, install, usage, dev commands (aligns with ADR-0002), config/secrets, testing, contributing, license last. Copy/paste-able commands. |
| [0007](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md) | **Type-checked SQL in TypeScript** *(superseded by [0015](https://github.com/darkmatter/skills/blob/main/docs/adr/0015-cohesive-modules.md))* — banned inline SQL in favor of Kysely/Drizzle. 0015 keeps typed builders in place and allows parameterized SQL inside its owning adapter with row validation. |
| [0008](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md) | **Per-language reference codebases** under `references/` in darkmatter/skills. Skills carry prose; references carry code. Precedence: project `.agent/` → `references/` → general idiom. |
| [0009](https://github.com/darkmatter/skills/blob/main/docs/adr/0009-curate-default-agent-skill-bundle.md) | **Curate the default skill bundle** *(superseded by 0010).* Home Manager enabled a small explicit allowlist; runtime hooks lived under `presets/`. Replaced by ADR-0010. |
| [0010](https://github.com/darkmatter/skills/blob/main/docs/adr/0010-install-all-catalogued-agent-skills.md) | **Install all catalogued skills.** Home Manager installs every top-level `skills/` directory — the catalog is the inventory, not an allowlist. Client runtime assets remain under `presets/<client>/runtime/`. |
| [0011](https://github.com/darkmatter/skills/blob/main/docs/adr/0011-sops-files-as-json.md) | **SOPS payloads are JSON** (`*.sops.json`). No new `*.sops.yaml` / `.env.sops`; convert when touched. `.sops.yaml` creation rules are out of scope. |
| [0012](https://github.com/darkmatter/skills/blob/main/docs/adr/0012-ops-scripts-in-typescript.md) | **Ops scripts in TypeScript-only repos are TypeScript**, run with Bun. No new `.sh` beyond a runtime trampoline. |
| [0013](https://github.com/darkmatter/skills/blob/main/docs/adr/0013-shared-ui-is-its-own-package.md) | **Shared UI is its own package; the name is per-repo.** Screens import primitives from it; components stay presentational. |
| [0014](https://github.com/darkmatter/skills/blob/main/docs/adr/0014-named-config-files-over-flags-and-env.md) | **Named config files over flags/env.** Interactive runs pick a named config from a list; flags and env vars only override individual keys, read through `effect/Config`. |
| [0015](https://github.com/darkmatter/skills/blob/main/docs/adr/0015-cohesive-modules.md) | **Cohesive modules.** Keep each capability together; split only when it improves understanding. Canonical module rules live in the **Readability and module design** section of `docs/agents/60-readability.md` (ownership moved by [0017](https://github.com/darkmatter/skills/blob/main/docs/adr/0017-skill-and-instruction-ownership.md)). Supersedes 0007's blanket SQL ban. |
| [0015](https://github.com/darkmatter/skills/blob/main/docs/adr/0015-hostnames-by-audience-workers-by-path.md) | **Hostnames by audience, Workers by path.** `api.dm.sh` is the single service hostname; Workers attach plural path prefixes via zone routes. No new hostname per Worker. *(Two accepted ADRs share the number 0015 — cite by title.)* |
| [0016](https://github.com/darkmatter/skills/blob/main/docs/adr/0016-gherkin-acceptance-for-assigned-tasks.md) | **Gherkin acceptance for assigned tasks.** Every assigned Linear task or GitHub issue carries at least one Given/When/Then scenario before implementation; trivial mechanical tasks may note an exception. |
| [0017](https://github.com/darkmatter/skills/blob/main/docs/adr/0017-skill-and-instruction-ownership.md) | **Skill and instruction ownership.** Each policy domain has exactly one owner; other artifacts reference the owner by name. Do not recreate a deleted skill to hold a rule. Partially supersedes 0015's ownership/pointer decision only. |
| [0018](https://github.com/darkmatter/skills/blob/main/docs/adr/0018-centaur-consumes-skills-directly.md) | **Centaur consumes the root skills catalog.** `skills/` at the repo root is the only source; no sandbox manifest, generated subtree, or sync script. |

---

## Skills to apply proactively

**Always-on:** `definition-of-done`, `when-to-write-tests` (default is no new test; test observable public behavior)

**Architecture:** `effect-typescript`, `alchemy`, `darkmatter-ts-toolchain`, `darkmatter-repo-setup`, `nix-flake-organization`, `sops-secret-access`, `repository-organization`, `choose-dev-entrypoints`

**Code quality:** `codebase-cleanup` (scoped cleanup and the optional eight-pass audit), `advisor` (advisor-facing review guidance)

**UI:** `darkmatter-design-system`, `shadcn-registry-first`, `ui-component-architecture`, `run-ui-registry-variations` (manual invocation only)

**Externally installed** (declared in root `package.json` `skills`; upstream-maintained, not vendored): `agent-browser` (`vercel-labs/agent-browser`), `find-skills` (`vercel-labs/skills`), `rust-best-practices` (`apollographql/skills`), `test-driven-development` (`obra/superpowers`, opt-in when the user asks for TDD), `ui-ux-pro-max` (`nextlevelbuilder/ui-ux-pro-max-skill`), `vercel-react-best-practices` (`vercel-labs/agent-skills`)

**Client runtimes (opt-in, not task skills — ADR-0010):** `presets/claude/runtime/session-context-pipeline` (Claude hook bundle: session summarizer, doc injection, end-of-turn checklist), `presets/opencode/runtime/continuous-learning` (stop hook), `presets/opencode/runtime/strategic-compact.md` (auto-compaction contract)

---

## Per-repo agent context

Every darkmatter project repo has its own `AGENTS.md` and `.agent/` directory with project-specific state and decisions. Project `AGENTS.md` files are generated from the shared [`docs/agents/`](https://github.com/darkmatter/skills/tree/main/docs/agents) topics (`inputs.darkmatter-skills.agentsMd`) plus the repo's own `docs/AGENTS.repo.md` — see the `darkmatter-repo-setup` skill. This org-level file is the floor; project-level files are the ceiling.

# Darkmatter — GitHub Copilot & Codex Instructions

## Skills and ADRs

Reusable agent skills and architecture decision records live in [darkmatter/skills](https://github.com/darkmatter/skills).

- Skills catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md)
- ADRs: [`docs/adr/`](https://github.com/darkmatter/skills/tree/main/docs/adr)

## Architecture decisions (apply to all repos)

| ADR | Rule |
|-----|------|
| ADR-0002 | Every repo exposes `install`, `setup`, `server`/`run`, `test`, `build`, `ci`, `console` via `./scripts/<name>` or `just <name>`. Bootstrap: `./scripts/install && ./scripts/setup`. Pre-PR: `./scripts/ci`. |
| ADR-0003 | Cross-language types use Protobuf + `buf`. Default transport: ConnectRPC. Commit generated code. `buf lint` + `buf breaking` in CI. |
| ADR-0004 | No reinvention — check for existing libraries before implementing. A dependency beats private code. |
| ADR-0005 | One typed `src/settings.<ext>` per binary. Only place that reads raw env vars. Validates at startup. Secret values use redacted wrappers — never plain strings. |
| ADR-0006 | README minimum standard — follow [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) structure: title, install, usage, dev commands, config/secrets, testing, contributing, license last. Copy/paste-able commands. |
| ADR-0007 | Type-checked SQL in TypeScript *(superseded by ADR-0015, cohesive modules)* — originally banned inline SQL in favor of Kysely/Drizzle. 0015 keeps typed builders in place and allows parameterized SQL inside its owning adapter with row validation. |
| ADR-0008 | Per-language reference codebases under `references/` in darkmatter/skills. Skills carry prose; references carry code. Precedence: project `.agent/` → `references/` → general idiom. |
| ADR-0009 | Curate the default skill bundle *(superseded by 0010)*. Home Manager enabled a small explicit allowlist; runtime hooks under `presets/`. Replaced by ADR-0010. |
| ADR-0010 | Install all catalogued skills. Home Manager installs every top-level `skills/` directory — the catalog is the inventory, not an allowlist. Client runtime assets remain under `presets/<client>/runtime/`. |
| ADR-0011 | SOPS payloads are JSON, named `*.sops.json`. No new `*.sops.yaml` / `.env.sops`; convert when touched. |
| ADR-0012 | Ops scripts in TypeScript-only repos are TypeScript, run with Bun. No new `.sh` beyond a runtime trampoline. |
| ADR-0013 | Shared UI is its own package; the package name is per-repo. Screens import primitives from it. |
| ADR-0014 | Configuration comes from named config files picked from a list; flags and env vars only override individual keys, read through `effect/Config`. |
| ADR-0015 | Cohesive modules: keep each capability together; split only when it improves understanding. Canonical rules live in the `codebase-design` skill. |
| ADR-0015 | Hostnames by audience, Workers by path: `api.dm.sh` is the single service hostname; Workers attach plural path prefixes via zone routes. *(Two accepted ADRs share 0015 — cite by title.)* |
| ADR-0016 | Gherkin acceptance: assigned Linear tasks and GitHub issues carry at least one Given/When/Then scenario before implementation; trivial mechanical tasks may note an exception. |
| OTel | App code imports only OTel SDKs; provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

## Always apply

- `diagnose` — before proposing fixes
- `definition-of-done` — any complex multi-step task
- `when-to-write-tests` — before writing or requesting tests; default is no new test, test observable public behavior

## Key skills by category

**Architecture:** `effect-typescript`, `alchemy`, `darkmatter-ts-toolchain`, `darkmatter-gitops-conventions`, `darkmatter-repo-setup`, `nix-flake-organization`, `sops-secret-access`, `repository-organization`, `domain-organization`, `codebase-design`, `choose-dev-entrypoints`, `rust-best-practices`

**Code quality:** `codebase-cleanup`, `keep-codebase-maintainable`, `test-driven-development` (opt-in, when the user asks for TDD)

**Workflow:** `advisor`, `flue`, `find-skills`

**UI/Frontend:** `darkmatter-design-system`, `ui-ux-pro-max`, `shadcn-registry-first`, `ui-component-architecture`, `vercel-react-best-practices`, `run-ui-registry-variations`

**Browser automation:** `agent-browser` (CDP, Node/Rust)

**Client runtimes (opt-in, not task skills — ADR-0010):** `presets/claude/runtime/session-context-pipeline` (Claude hook bundle: session summarizer, doc injection, end-of-turn checklist), `presets/opencode/runtime/continuous-learning` (stop hook), `presets/opencode/runtime/strategic-compact.md` (auto-compaction contract)

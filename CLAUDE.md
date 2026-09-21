# Darkmatter — Claude Code Organizational Context

This file provides org-level instructions and context for Claude Code agents working across all darkmatter repositories. It applies to every repo in the darkmatter GitHub organization.

For full skills and ADR details: [darkmatter/skills](https://github.com/darkmatter/skills)

---

## What Darkmatter builds

Darkmatter is a small, polyglot engineering team shipping developer tools, crypto/DeFi infrastructure, and AI-native applications. Primary stack: TypeScript/Bun (most apps), Effect (functional TS), Rust (performance services), Python (tooling), Nix (dev environments and system config), React/Next.js (frontend).

---

## Architecture Decisions

These decisions apply to **every** darkmatter project repo unless the project explicitly documents an exception. Full ADR text lives in [darkmatter/skills/docs/adr](https://github.com/darkmatter/skills/tree/main/docs/adr).

### ADR-0002: Standard command surface
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0002-standard-project-command-surface.md)

Every darkmatter repo exposes these commands as `./scripts/<name>` or `just <name>`:

| Command | Purpose |
|---------|--------|
| `install` | Idempotent host-level toolchain installer (runtimes, compilers, system tools) |
| `setup` | Per-checkout: install deps from lockfiles, DB init, decrypt secrets, run codegen |
| `server` / `run` | Start the app (`server` for services, `run` for CLIs/one-shots) |
| `test` | Run the test suite |
| `build` | Build the deployable artifact |
| `ci` | Full pre-PR gate: lint + typecheck + test + build + drift checks |
| `console` | Interactive shell / REPL / devshell |

Bootstrap any darkmatter repo:
```sh
git clone <project>
./scripts/install
./scripts/setup
./scripts/server  # or: just server
```
Before opening a PR: `./scripts/ci`

Nix repos SHOULD wrap scripts to enter the devshell (`nix develop -c <command>`).

### ADR-0003: Protobuf is the source of truth for service and type definitions
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md)

Any codebase sharing types across a language boundary MUST use Protobuf as the IDL. Tooling: `buf`. Default transport: ConnectRPC.

- `.proto` files live at `proto/<package>/` in the service repo
- Generated code is **committed** (enables grep, reviewable diffs, no build-time dep for consumers)
- `buf lint` + `buf breaking` run in CI; breaking changes against main fail the build
- Add `./scripts/proto-gen` (or `just proto-gen`) for the codegen entrypoint
- CI runs `buf generate` then `git diff --exit-code` to catch drift

Exemptions: pure libraries, services with ≤5 endpoints + single language + single first-party client, schema-as-code setups where all typed consumers are in the same language.

### ADR-0004: No reinvention
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md)

Before writing an implementation, check whether it's a solved problem.

1. **Prefer structured output over output parsing.** Check for machine-readable flags before parsing human-readable output.
2. **Search before writing.** If it would exceed ~20 lines or touches a well-known domain (encoding, escaping, formatting, parsing, cryptography, date/time, protocols), assume a library exists.
3. **A dependency is preferable to a private reimplementation.** Libraries have tests, handle edge cases, and receive upstream fixes.

### ADR-0005: Application config in one typed settings module
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0005-typed-settings-module-decoupled-from-provider.md)

Every binary has exactly one settings module (`src/settings.<ext>`) that:
- Declares all configuration inputs in a typed struct/service
- Is the **only** place that reads raw provider APIs (`process.env`, `os.environ`, `std::env::var`, `Bun.env`, etc.)
- Validates at startup and fails fast with structured errors listing every problem at once
- Is decoupled from its provider — swapping from env vars to SOPS to a secret manager touches only the runtime entrypoint, not the description or consumers

Three layers: **description** (typed schema), **provider** (swappable source), **runtime** (wiring point).

Effect TypeScript reference:
```ts
// src/settings.ts — description layer only
export class Settings extends Effect.Service<Settings>()("Settings", {
  effect: Effect.gen(function* () {
    const port = yield* Config.integer("PORT").pipe(Config.withDefault(8080))
    const databaseUrl = yield* Config.string("DATABASE_URL")
    const stripeKey = yield* Config.redacted("STRIPE_KEY")
    return { port, databaseUrl, stripeKey } as const
  }),
}) {}
```

Secret values MUST be typed as redacted wrappers (`Config.redacted`, Pydantic `SecretStr`, Rust `secrecy::Secret<T>`). Plain string typing for a secret is a defect.

ADR-0014 makes named config files the primary source: an interactive run picks a named config from a list, and flags/env vars only override individual keys.

### ADR-0006: README minimum standard
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md)

Every darkmatter project README follows [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) as the default structure. Required sections: title + short description, install (copy/paste-able), usage (copy/paste-able quickstart), development command surface (aligns with ADR-0002), configuration/secrets, testing/verification, contributing, and license last. Copy/paste-able means commands run as written from the repo root.

### ADR-0007: Type-checked SQL in TypeScript *(superseded)*
**Status:** Superseded by ADR-0015 (cohesive modules) | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md)

Originally banned inline SQL in favor of schema-derived query builders (**Kysely** preferred, **Drizzle** allowed). ADR-0015 supersedes the blanket ban: existing typed builders and ORMs may stay in place, and parameterized SQL may live inside its owning adapter alongside bindings and row conversion, with returned rows validated and observable query behavior verified against the database.

### ADR-0008: Per-language reference codebases
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md)

Per-language reference codebases (under `references/` in `darkmatter/skills`) hold exemplar code showing preferred conventions. Skills carry prose guidance; references carry code. Precedence when conventions conflict: project `.agent/` rules → `references/` exemplars → general language idiom.

### ADR-0009: Curate the default agent skill bundle *(superseded)*
**Status:** Superseded by ADR-0010 | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0009-curate-default-agent-skill-bundle.md)

A small explicit allowlist of team-wide skills was enabled by Home Manager; client-runtime hooks lived under `presets/<client>/runtime/`. Replaced by ADR-0010, which installs every catalogued skill.

### ADR-0010: Install all catalogued agent skills
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0010-install-all-catalogued-agent-skills.md)

Home Manager installs **every** top-level directory in `skills/`. The module derives the enabled skill IDs from the source directory — the catalog is the human-readable inventory, not an allowlist. Adding a new skill directory includes it automatically. Client runtime assets remain under `presets/<client>/runtime/` and are not installed as task skills.

### ADR-0011: SOPS-encrypted files are stored as JSON
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0011-sops-files-as-json.md)

SOPS-encrypted secret documents MUST be JSON, named `*.sops.json`. No new `*.sops.yaml`, `.env.sops`, or other non-JSON payloads; convert existing ones when touched. The SOPS creation-rules file (`.sops.yaml`) is SOPS configuration, not an encrypted document, and stays as-is. JSON payloads import directly into TypeScript and Nix (`lib.importJSON`) and stay inspectable with `jq` without decrypting.

### ADR-0012: TypeScript-only projects write ops scripts in TypeScript
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0012-ops-scripts-in-typescript.md)

In a TypeScript-only project, utility and operations scripts (`scripts/`, codegen, migrate/seed, release, doctor, git hooks, CI steps beyond invoking one command) MUST be TypeScript, run with Bun (`bun scripts/ci.ts`). Allowed dispatchers: `just` recipes, package/Turbo scripts, and CI YAML that only exec the TS entrypoint. Narrow POSIX exception: a trampoline that only bootstraps the runtime or `exec`s into `nix develop`. No new `.sh` files; convert when touched.

### ADR-0013: Shared UI lives in its own package; the name is per-repo
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0013-shared-ui-is-its-own-package.md)

Reusable React UI MUST live in its own package, separate from app screens, routes, and business logic. The package path and import alias are per-repo — discover them from the workspace; do not require `@repo/ui` or any other name. Graduate a visual unit into the package on second use or when it is a presentational primitive; keep components dumb (no routes, stores, or API clients inside).

### ADR-0014: Named config files over flags and env
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0014-named-config-files-over-flags-and-env.md)

Configuration comes from named, committed, complete configuration files. There is no selector flag: an interactive run picks a named config from a list; a non-interactive run takes the name from the environment. Flags and environment variables override individual keys — they are not the primary source. Everything is read through `effect/Config` (the ADR-0005 settings module), so the source stays interchangeable. Scope is configuration, not invocation arguments.

### ADR-0015: Cohesive modules and small public interfaces
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0015-cohesive-modules.md)

Keep each capability together, make it simple to use, and split it only when the split improves understanding. The rules in the `codebase-design` skill are the canonical module convention. Group source by domain, then by useful role; keep an operation's helpers, queries, bindings, and decoding nearby. Public interfaces expose complete operations including ordering, failure, and cleanup. Retain thin public package entries and real external-interface adapters; remove internal forwarding chains that add no contract. When introducing a file limit, use 300 nonblank, noncomment lines. Supersedes ADR-0007's blanket SQL ban; ADR-0013's UI package boundary remains in force.

### ADR-0015: Hostnames by audience, Workers by path
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0015-hostnames-by-audience-workers-by-path.md)

Cloudflare Workers get hostnames by audience, not per feature: `dm.sh` / `darkmatter.io` serve the marketing site; `api.dm.sh` is the single hostname for service Workers, each addressed by a plural path prefix via a zone route. `dm.sh/api/*` is reserved for same-origin proxies. No new hostname per Worker without an ADR-level reason. Only the production Alchemy stage attaches DNS records and routes.

*(Note: two accepted ADRs share the number 0015 — cite and index by title, not number.)*

### ADR-0016: Assigned tasks include Gherkin acceptance scenarios
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0016-gherkin-acceptance-for-assigned-tasks.md)

Every Linear task or GitHub issue assigned to a person or agent MUST include at least one Gherkin acceptance scenario (Given/When/Then with concrete inputs and observable outcomes) before implementation begins. A trivial, mechanical task may omit the scenario by stating the expected result and noting the exception; behavior changes always require one. Complete the task when acceptance conditions are verified and the evidence recorded.

### OTel-only observability
**Status:** Accepted

App code depends only on OpenTelemetry SDKs. Provider-specific packages (`@sentry/*`, PostHog, Datadog) never appear in `apps/*`. Provider wiring is isolated in shared packages.

---

## Skills catalog

Team-wide skills distribute from [darkmatter/skills](https://github.com/darkmatter/skills) via Nix Home Manager, which installs every top-level `skills/` directory (ADR-0010). Full catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md).

### Apply on every task

| Skill | When |
|-------|------|
| `diagnose` | Before proposing fixes for bugs or failures |
| `definition-of-done` | Complex, multi-step tasks |
| `when-to-write-tests` | Before writing or requesting tests — default is no new test; test observable public behavior |

### Architecture & infrastructure

| Skill | Use for |
|-------|--------|
| `effect-typescript` | Effect services, Layers, typed errors, Schema, Alchemy deploys |
| `alchemy` | Alchemy v2 infrastructure (Cloudflare/AWS providers) |
| `darkmatter-ts-toolchain` | Org TS toolchain contract: Bun, vitest/oxlint, Effect, Alchemy deploys, changesets |
| `darkmatter-gitops-conventions` | Safe-change playbook for `darkmatter/gitops` (validation, sha-pinned images, SOPS, rollback) |
| `darkmatter-repo-setup` | Set up or onboard a repo to darkmatter standards: toolchain, Nix devshell, ops surface, CI, AGENTS.md |
| `nix-flake-organization` | Thin `flake/` public layer + `src/` implementation |
| `sops-secret-access` | SOPS-encrypted config, private registries (JSON payloads, ADR-0011) |
| `repository-organization` | Repo layout, Standard README, ADR placement, agent context |
| `domain-organization` | Organize source by domain owner, role, and module; plan structural moves |
| `codebase-design` | Module boundaries and cohesive capabilities — canonical conventions (ADR-0015) |
| `choose-dev-entrypoints` | Choose responsibility boundaries across Nix, Just, Bun, Turborepo, scripts |
| `rust-best-practices` | Idiomatic Rust: borrowing, error handling, linting, performance, testing |

### Task and workflow

| Skill | Use for |
|-------|--------|
| `advisor` | Acting as an advisor to another agent |
| `flue` | Working with the Flue framework |
| `codebase-cleanup` | Multi-pass refactor sweep (8 specialist subagents) |
| `keep-codebase-maintainable` | Cleanup and maintainability passes, not feature work |
| `test-driven-development` | Opt-in red-green-refactor when the user asks for TDD; ordinary test requests use `when-to-write-tests` |
| `find-skills` | Discover and install agent skills from the open ecosystem |

### UI/Frontend

| Skill | Use for |
|-------|--------|
| `darkmatter-design-system` | Canonical darkmatter UI design system — tokens, theming, components; prefer over generic shadcn/ui |
| `ui-ux-pro-max` | Design system intelligence (styles, palettes, fonts, UX guidelines) |
| `shadcn-registry-first` | Bias UI work toward existing shadcn registry components before hand-rolling |
| `ui-component-architecture` | Keep reusable React UI in its own per-repo package (ADR-0013); screens stay thin |
| `vercel-react-best-practices` | React/Next.js performance |
| `run-ui-registry-variations` | Build three UI variations from shadcnblocks, Aceternity, or the Darkmatter registry (manual invocation) |

### Browser automation

| Skill | Use for |
|-------|--------|
| `agent-browser` | Chrome/Chromium via CDP — browser automation for Node.js/Rust workflows |

### Client runtimes (not task skills)

These are **not task skills** (ADR-0010). They are opt-in hook bundles under `presets/<client>/runtime/`:

| Item | Client | What |
|-------|--------|------|
| `session-context-pipeline` | Claude | Opt-in hook bundle: session summarizer, library doc injection, end-of-turn checklist |
| `continuous-learning` | OpenCode | Opt-in continuous-learning stop hook |
| `strategic-compact` | OpenCode | Auto-compaction contract implemented by the OpenCode plugin |

---

## Working conventions

1. **Use the standard command surface.** `./scripts/setup` before working; `./scripts/ci` before PRs. In TypeScript-only repos the scripts themselves are TypeScript, run with Bun (ADR-0012).
2. **Reference ADRs when making architectural decisions.** Surface conflicts before proceeding.
3. **Secrets use SOPS.** Apply `sops-secret-access` skill; never print decrypted contents. Encrypted documents are JSON, named `*.sops.json` (ADR-0011).
4. **Effect is the default for TypeScript services.** See `effect-typescript` skill and ADR-0005.
5. **Protobuf when crossing language boundaries.** Use `buf`, commit generated code (ADR-0003).
6. **One settings module per binary.** No scattered `process.env` reads (ADR-0005). Configuration comes from named config files; flags and env vars only override individual keys (ADR-0014).
7. **Cohesive modules.** Keep each capability together; parameterized SQL may live in its owning adapter with row validation (ADR-0015, which supersedes ADR-0007's blanket SQL ban).
8. **READMEs meet the minimum standard.** Follow Standard Readme structure with required onboarding anchors (ADR-0006).
9. **Shared UI is its own package.** The package name is per-repo; screens import primitives from it (ADR-0013).
10. **One service hostname.** Service Workers attach to `api.dm.sh` by plural path prefix; no new hostname per Worker (ADR-0015).
11. **Assigned tasks carry Gherkin acceptance scenarios** before implementation begins (ADR-0016).

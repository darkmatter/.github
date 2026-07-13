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

### ADR-0006: README minimum standard
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0006-readme-minimum-standard.md)

Every darkmatter project README follows [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) as the default structure. Required sections: title + short description, install (copy/paste-able), usage (copy/paste-able quickstart), development command surface (aligns with ADR-0002), configuration/secrets, testing/verification, contributing, and license last. Copy/paste-able means commands run as written from the repo root.

### ADR-0007: Type-checked SQL in TypeScript
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md)

TypeScript code MUST NOT embed SQL as inline strings or template literals (including tagged templates like `` sql<Row>`...` ``). Use a type-checked query builder or ORM that derives query types from the database schema. **Kysely** is preferred; **Drizzle** is allowed when already present. If a query can't be expressed through the typed surface, improve the abstraction — do not fall back to inline SQL.

### ADR-0008: Per-language reference codebases
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md)

The `references/` section of `darkmatter/skills` holds per-language reference codebases (currently `rust/`, `go/`, `typescript/`) with exemplar code showing preferred conventions. Skills carry prose guidance; references carry code. Precedence when conventions conflict: project `.agent/` rules → `references/` exemplars → general language idiom.

### OTel-only observability
**Status:** Accepted

App code depends only on OpenTelemetry SDKs. Provider-specific packages (`@sentry/*`, PostHog, Datadog) never appear in `apps/*`. Provider wiring is isolated in shared packages.

---

## Skills catalog

Team-wide skills distribute from [darkmatter/skills](https://github.com/darkmatter/skills) via Nix Home Manager. Full catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md).

### Apply on every task

| Skill | When |
|-------|------|
| `coding-standards` | Any TypeScript/JS/React/Node code authoring or review |
| `brainstorming` | Before any non-trivial implementation |
| `test-driven-development` | Before writing implementation code |
| `diagnose` | Before proposing fixes for bugs or failures |
| `definition-of-done` | Complex, multi-step tasks |

### Architecture & infrastructure

| Skill | Use for |
|-------|--------|
| `effect-typescript` | Effect services, Layers, typed errors, Schema, Alchemy deploys |
| `alchemy` | Alchemy v2 infrastructure (Cloudflare/AWS providers) |
| `darkmatter-ts-toolchain` | Org TS toolchain contract: Bun, vitest/oxlint, Effect, Alchemy deploys, changesets |
| `darkmatter-gitops-conventions` | Safe-change playbook for `darkmatter/gitops` (validation, sha-pinned images, SOPS, rollback) |
| `nix-flake-organization` | Thin `flake/` public layer + `src/` implementation |
| `sops-secret-access` | SOPS-encrypted config, private registries |
| `repository-organization` | Repo layout, Standard README, ADR placement, agent context |
| `choose-dev-entrypoints` | Choose responsibility boundaries across Nix, Just, Bun, Turborepo, scripts |
| `rust-best-practices` | Idiomatic Rust: borrowing, error handling, linting, performance, testing |
| `zoom-out` | Map modules and callers using domain glossary vocabulary — read-only orientation |
| `improve-codebase-architecture` | Surface architectural friction and propose deepening opportunities for testability |

### Task and workflow

| Skill | Use for |
|-------|--------|
| `session-context-pipeline` | Hook-driven session summarizer, library doc injection, end-of-turn checklist |
| `finishing-a-development-branch` | Merge, PR, or cleanup after implementation |
| `handoff` | Compact conversation into a handoff document for a fresh agent |
| `triage` | Triage issues through category/state roles; write agent briefs for ready issues |
| `grill-me` | Interview the user relentlessly about a plan until reaching shared understanding |
| `grill-with-docs` | Grilling session that challenges a plan and updates CONTEXT.md and ADRs inline |
| `dm-skill-creator` | Create a new team-wide skill |
| `requesting-code-review` | Dispatch code-reviewer subagent before merge |
| `receiving-code-review` | Evaluate review feedback rigorously before implementing |
| `codebase-cleanup` | Multi-pass refactor sweep (8 specialist subagents) |
| `end-of-turn-review` | GPT second-opinion pass over diffs or plans at end of turn |
| `writing-skills` | TDD applied to process documentation — create, edit, verify skills |
| `find-skills` | Discover and install agent skills from the open ecosystem |
| `run-meeting-summary` | Resolve meeting artifacts and draft approved Obsidian summaries |

### UI/Frontend

| Skill | Use for |
|-------|--------|
| `frontend-design` | Distinctive, production-grade UI |
| `ui-ux-pro-max` | Design system intelligence (styles, palettes, fonts, UX guidelines) |
| `shadcn-registry-first` | Bias UI work toward existing shadcn registry components before hand-rolling |
| `ui-component-architecture` | Keep React screens thin; reuse `@repo/ui` primitives, avoid div-soup |
| `vercel-react-best-practices` | React/Next.js performance |
| `nextjs-to-rwsdk-migration` | Port Next.js App Router to RedwoodSDK on Cloudflare Workers |
| `kickoff-dm-design` | Design-room kickoff: Linear ticket + Slack post from a Claude Design URL |
| `prototype` | Throwaway prototype to answer a design question (terminal app or UI variations) |

### Browser automation

| Skill | Use for |
|-------|--------|
| `browser-use` | Browser automation via `browser-use` CLI with persistent sessions (Python) |
| `agent-browser` | Chrome/Chromium via CDP — prefer for Node.js/Rust workflows |

### Communication & compression

| Skill | Use for |
|-------|--------|
| `caveman` | Ultra-compressed communication (~75% token savings) |
| `caveman-commit` | Ultra-compressed conventional commit messages (subject ≤50 chars) |
| `caveman-review` | Ultra-compressed code review comments (one line per finding) |
| `compress` | Compress natural-language memory files into caveman format |

### Domain-specific

| Skill | Use for |
|-------|--------|
| `neon-postgres` | Neon Serverless Postgres |
| `openchronicle-setup` | Local-first agent memory (macOS) |
| `hl-funding-analysis` | Hyperliquid perp funding rate analysis |

### Runtime policies (auto-applied by agent client)

These are **not task skills** — they are consumed by the agent runtime to configure session behavior.

| Skill | When |
|-------|------|
| `using-superpowers` | Session start — establishes skill discovery and invocation protocol |
| `continuous-learning` | Session end (Stop hook) — extracts reusable patterns into new skills |
| `strategic-compact` | Long autonomous sessions with auto-compaction enabled |

---

## Working conventions

1. **Use the standard command surface.** `./scripts/setup` before working; `./scripts/ci` before PRs.
2. **Reference ADRs when making architectural decisions.** Surface conflicts before proceeding.
3. **Secrets use SOPS.** Apply `sops-secret-access` skill; never print decrypted contents.
4. **Effect is the default for TypeScript services.** See `effect-typescript` skill and ADR-0005.
5. **Protobuf when crossing language boundaries.** Use `buf`, commit generated code (ADR-0003).
6. **One settings module per binary.** No scattered `process.env` reads (ADR-0005).
7. **Type-checked SQL in TypeScript.** No inline SQL strings — use Kysely or Drizzle (ADR-0007).
8. **READMEs meet the minimum standard.** Follow Standard Readme structure with required onboarding anchors (ADR-0006).

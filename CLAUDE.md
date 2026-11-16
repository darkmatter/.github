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

Nix repos SHOULD wrap scripts to enter the devshell (`nix develop -c <command>`). `turbo run <target>` is the one accepted substitute, for monorepos where turbo's cache/dependency-graph is the point.

### ADR-0003: Protobuf is the source of truth for service and type definitions
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0003-protobuf-as-service-source-of-truth.md)

Any codebase sharing types across a language boundary MUST use Protobuf as the IDL. Tooling: `buf`. Default transport: Connect (ConnectRPC).

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

Darkmatter READMEs follow [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) as the default shape. Every non-trivial project README MUST include: title + short description, background (when needed), a table of contents (required over 100 lines), copy/paste-able **install**, copy/paste-able **usage**/quickstart, the standard command surface from ADR-0002 (or a link to it), configuration/secrets pointers (no secret values), a verification/test command, contributing guidance, and license last.

### ADR-0007: Type-checked SQL in TypeScript
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md)

TypeScript MUST NOT embed SQL as inline strings or template literals for application queries — including tagged templates like `sql<Row>\`...\``, which only assert a row shape without checking table/column names, joins, or nullability.

Use a type-checked query builder or ORM that derives types from the schema:
1. **Kysely** preferred for query-heavy code — strongest inference across selects, joins, aliases, nullability.
2. **Drizzle** allowed when already present, but not preferred for complex query-heavy code (weaker inference).
3. Other tools acceptable only with comparable compile-time checking.

Plain `.sql` files for external DB tooling (validated by migration/schema tooling, not embedded in TS) are outside scope; TypeScript migration files are still covered.

### ADR-0008: Per-language reference codebases
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md)

Preferred conventions for a language/framework live as runnable exemplar code under [`darkmatter/skills/references/<language>/`](https://github.com/darkmatter/skills/tree/main/references) (currently `rust/`, `go/`, `typescript/`), each with a `README.md` index — prose guidance stays in skills, code exemplars live in `references/`.

Precedence when conventions conflict: project `.agent/` rules → `references/` exemplars → general language idiom.

`references/` is **not** distributed by the Nix Home Manager module (which syncs only `skills/`); consumers need the repo checkout.

### OTel-only observability
**Status:** Accepted (team convention; not yet a numbered ADR)

App code depends only on OpenTelemetry SDKs. Provider-specific packages (`@sentry/*`, PostHog, Datadog) never appear in `apps/*`. Provider wiring is isolated in shared packages.

### Note: task tracking / Beads (`bd`)

Several skills and hooks in `darkmatter/skills` (`darkmatter-gitops-conventions`, `.codex/hooks.json`) actively reference **Beads (`bd`)** — `bd prime`, `bd create`, `bd remember`, `bd codex-hook` — as the task-tracker / agent-memory convention. However, `darkmatter/skills/docs/adr/` currently has **no `0001-beads-as-task-tracker-and-agent-memory.md`** file; the ADR that historically documented this appears to be missing or was never finalized upstream. Until that's resolved, treat `bd` as an actively-used team convention rather than something to cite as an accepted ADR. Flag this gap if you're deciding whether to keep building on `bd`.

---

## Skills catalog

Team-wide skills distribute from [darkmatter/skills](https://github.com/darkmatter/skills) via Nix Home Manager, or as a [Claude Code plugin](https://github.com/darkmatter/skills#claude-code-plugin-marketplace) (`/plugin marketplace add darkmatter/skills`). Full catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md). The list below reflects the 43 skills that actually exist under `skills/` as of this writing — verify against the catalog before citing a skill that isn't here.

### Apply on every task

| Skill | When |
|-------|------|
| `brainstorming` | Before any non-trivial implementation |
| `test-driven-development` | Before writing implementation code |
| `tdd` | Explicit red-green-refactor / test-first requests (local alternative to the always-on skill above) |
| `definition-of-done` | Complex, multi-step tasks |

### Debugging & orientation

| Skill | Use for |
|-------|--------|
| `diagnose` | Reproduce → minimise → hypothesise → instrument → fix → regression-test |
| `zoom-out` | Read-only orientation before diving into unfamiliar code |
| `improve-codebase-architecture` | Surface architectural friction, deepen shallow modules |
| `choose-dev-entrypoints` | Pick responsibility boundaries across Nix/Just/Bun/Turbo/scripts |
| `triage` | State-machine triage for bugs/enhancements through review |

### Architecture & infrastructure

| Skill | Use for |
|-------|--------|
| `effect-typescript` | Effect services, Layers, typed errors, Schema, Alchemy deploys |
| `alchemy` | Alchemy v2 infrastructure (Cloudflare/AWS providers) |
| `nix-flake-organization` | Thin `flake/` public layer + `src/` implementation |
| `sops-secret-access` | SOPS-encrypted config, private registries |
| `darkmatter-ts-toolchain` | Org TS toolchain contract: Bun-only, tsgo/vitest/oxlint, Effect, Alchemy |
| `darkmatter-gitops-conventions` | Safe-change playbook specific to `darkmatter/gitops` |
| `repository-organization` | Repo layout, Standard README, ADR placement, agent context |

### Code quality & review

| Skill | Use for |
|-------|--------|
| `codebase-cleanup` | Multi-pass refactor sweep (8 specialist subagents) |
| `requesting-code-review` | Dispatch code-reviewer subagent before merge |
| `receiving-code-review` | Evaluate review feedback rigorously before implementing |
| `end-of-turn-review` | Second-opinion pass over diffs or plans at end of turn |
| `writing-skills` | TDD applied to process documentation — create, edit, verify skills |
| `ui-component-architecture` | Keep React screens thin, graduate reusable units into `packages/ui` |
| `rust-best-practices` | Idiomatic Rust — ownership, error handling, testing, type-state |

### UI/Frontend

| Skill | Use for |
|-------|--------|
| `ui-ux-pro-max` | Design system intelligence (styles, palettes, fonts, UX guidelines) |
| `vercel-react-best-practices` | React/Next.js performance |
| `shadcn-registry-first` | Bias UI work toward existing shadcn registry components |
| `run-ui-registry-variations` | **Manual.** Build 3 registry-backed UI variations |
| `nextjs-to-rwsdk-migration` | Port Next.js App Router to RedwoodSDK on Cloudflare Workers |
| `kickoff-dm-design` | **Manual.** Design-room kickoff: Linear ticket + Slack post from a Claude Design URL |

### Browser automation

| Skill | Use for |
|-------|--------|
| `agent-browser` | Chrome/Chromium via CDP — Node.js/Rust workflows, direct CDP control |

### Workflow & collaboration

| Skill | Use for |
|-------|--------|
| `finishing-a-development-branch` | Merge, PR, or cleanup after implementation |
| `dm-skill-creator` | Create a new team-wide skill |
| `find-skills` | Discover and install agent skills from the open ecosystem |
| `run-meeting-summary` | **Manual.** Resolve meeting artifacts and draft approved Obsidian summaries |
| `handoff` | Compact a long session into a handoff doc for the next agent |
| `grill-me` | Interview the user relentlessly to stress-test a plan or design |
| `grill-with-docs` | `grill-me`, plus updates CONTEXT.md and creates ADRs inline |
| `session-context-pipeline` | Hook-driven background session summarizer + doc-brief injector |
| `prototype` | Throwaway prototype to answer a design/state/UI question |

### Communication

| Skill | Use for |
|-------|--------|
| `caveman-commit` | Ultra-compressed conventional commit messages (subject ≤50 chars) |

### Domain-specific

| Skill | Use for |
|-------|--------|
| `openchronicle-setup` | Local-first agent memory (macOS) |

### Runtime policies (auto-applied by agent client)

These are **not task skills** — they are consumed by the agent runtime to configure session behavior.

| Skill | When |
|-------|------|
| `using-superpowers` | Session start — establishes skill discovery and invocation protocol |
| `continuous-learning` | Session end (Stop hook) — extracts reusable patterns into new skills |
| `strategic-compact` | Long autonomous sessions with auto-compaction enabled |

### Removed / not currently shipped

The following were previously listed in this file but do **not** exist under `darkmatter/skills/skills/` as of this writing: `coding-standards`, `browser-use`, `caveman`, `caveman-review`, `compress`, `frontend-design`, `neon-postgres`, `hl-funding-analysis`, `beads-setup`, `beads-linear-sync`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `systematic-debugging`, `verification-before-completion`. If these reappear in `docs/catalog.md`, restore them here.

---

## Working conventions

1. **Use the standard command surface.** `./scripts/setup` before working; `./scripts/ci` before PRs (ADR-0002).
2. **Reference ADRs when making architectural decisions.** Surface conflicts before proceeding.
3. **Secrets use SOPS.** Apply `sops-secret-access` skill; never print decrypted contents.
4. **Effect is the default for TypeScript services.** See `effect-typescript` skill and ADR-0005.
5. **Protobuf when crossing language boundaries.** Use `buf`, commit generated code (ADR-0003).
6. **One settings module per binary.** No scattered `process.env` reads (ADR-0005).
7. **No inline SQL in TypeScript.** Use Kysely (preferred) or Drizzle (ADR-0007).
8. **READMEs follow Standard Readme.** See ADR-0006 and `repository-organization`.

# Darkmatter — Claude Code Organizational Context

This file provides org-level instructions and context for Claude Code agents working across all darkmatter repositories. It applies to every repo in the darkmatter GitHub organization.

For full skills and ADR details: [darkmatter/skills](https://github.com/darkmatter/skills)

---

## What Darkmatter builds

Darkmatter is a small, polyglot engineering team shipping developer tools, crypto/DeFi infrastructure, and AI-native applications. Primary stack: TypeScript/Bun (most apps), Effect (functional TS), Rust (performance services), Python (tooling), Nix (dev environments and system config), React/Next.js (frontend).

---

## Architecture Decisions

These decisions apply to **every** darkmatter project repo unless the project explicitly documents an exception. Full ADR text lives in [darkmatter/skills/docs/adr](https://github.com/darkmatter/skills/tree/main/docs/adr).

> **ADR-0001 removed.** The prior "Beads is the standard task tracker" decision was retracted upstream (`darkmatter/skills` commit `f5f5227`, 2026-07-09) — the `beads-setup`/`beads-linear-sync` skills, `.beads/` state, and the ADR itself were all deleted. There is currently **no org-standard task tracker ADR**. Don't tell agents to use `bd`/beads; use the harness's native task tracking (e.g. `TodoWrite`/`TaskCreate`) or a project's own documented tracker instead.

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

Project READMEs MUST follow [Standard Readme](https://github.com/RichardLitt/standard-readme/blob/main/spec.md) as the default structure. At minimum: title + one-line description, a table of contents for READMEs over 100 lines, copy/paste-able **install** and **usage** blocks, the development command surface (ADR-0002), configuration/secrets notes, a verification command, contributing guidance, and license last. Documentation-only repos may skip install/usage only if they say so explicitly.

### ADR-0007: Type-checked SQL in TypeScript
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0007-type-checked-sql-in-typescript.md)

TypeScript MUST NOT embed SQL as inline strings or tagged templates (including `sql<Row>\`...\``) for application queries — a `Row` generic is a type assertion, not query checking. Use a type-checked query builder/ORM that derives types from the schema. **Kysely** is preferred for query-heavy code; **Drizzle** is allowed but not preferred for complex joins. Plain `.sql` files for external migration/schema tooling (not embedded in TypeScript) are outside scope.

### ADR-0008: Per-language reference codebases
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0008-per-language-reference-codebases.md)

[`darkmatter/skills/references/`](https://github.com/darkmatter/skills/tree/main/references) holds one directory per language (`rust/`, `go/`, `typescript/` today) with exemplar code showing preferred conventions — layout, error handling, testing, tooling, preferred libraries. Skills carry prose guidance; `references/` carries the code that shows it. Precedence when conventions conflict: project `.agent/` rules → `references/` exemplars → general language idiom. Not distributed via Home Manager (skills are); needs a repo checkout.

### OTel-only observability
**Status:** Accepted

App code depends only on OpenTelemetry SDKs. Provider-specific packages (`@sentry/*`, PostHog, Datadog) never appear in `apps/*`. Provider wiring is isolated in shared packages.

---

## Skills catalog

Team-wide skills distribute from [darkmatter/skills](https://github.com/darkmatter/skills) via Nix Home Manager. Full catalog: [`docs/catalog.md`](https://github.com/darkmatter/skills/blob/main/docs/catalog.md). This section lists only skills currently shipped from `skills/` (not `inactive/` or the archived `docs/00-inbox/skills/`) as of 2026-07-27.

### Task and workflow

| Skill | Use for |
|-------|--------|
| `choose-dev-entrypoints` | Deciding where dev/codegen/install commands belong (Nix vs Just vs Bun vs scripts) |
| `codebase-cleanup` | Multi-pass refactor sweep (8 specialist subagents) |
| `definition-of-done` | Any complex, multi-step task that may invite shortcuts |
| `diagnose` | Disciplined bug/perf-regression diagnosis loop (reproduce → minimise → hypothesise → instrument → fix → regression-test) |
| `find-skills` | Discover and install agent skills from the open ecosystem |
| `finishing-a-development-branch` | Merge, PR, or cleanup once implementation is complete |
| `grill-me` | Interview the user relentlessly on a plan/design until shared understanding is reached |
| `grill-with-docs` | `grill-me`, but updates CONTEXT.md/ADRs inline as decisions crystallize |
| `handoff` | Compact the current conversation into a handoff doc for a fresh agent |
| `improve-codebase-architecture` | Surface architectural friction and deepening opportunities |
| `prototype` | Throwaway prototype (terminal app or radically different UI variations) to answer a design question |
| `repository-organization` | Repo layout, Standard README, agent context, ADR placement |
| `tdd` | User-requested TDD workflow with interface planning and vertical red-green slices |
| `writing-skills` | TDD applied to process documentation — create, edit, verify skills |
| `zoom-out` | Map relevant modules/callers using domain vocabulary before diving into unfamiliar code |

### Architecture & infrastructure

| Skill | Use for |
|-------|--------|
| `effect-typescript` | Effect services, Layers, typed errors, Schema, Alchemy deploys |
| `alchemy` | Alchemy v2 infrastructure (Cloudflare/AWS providers) |
| `nix-flake-organization` | Thin `flake/` public layer + `src/` implementation |
| `sops-secret-access` | SOPS-encrypted config, private registries |
| `darkmatter-gitops-conventions` | Safe changes to `darkmatter/gitops`: validation trio, sha-pinned images, SOPS rules, rollback via revert |
| `darkmatter-ts-toolchain` | Org TS toolchain contract: Bun-only, tsgo/vitest/oxlint, Effect for I/O, Alchemy deploys, changesets |

### UI/Frontend

| Skill | Use for |
|-------|--------|
| `ui-ux-pro-max` | Design system intelligence (50+ styles, palettes, fonts, UX guidelines) |
| `vercel-react-best-practices` | React/Next.js performance |
| `ui-component-architecture` | Keep screens thin; graduate reusable units into `packages/ui` |
| `shadcn-registry-first` | Install from configured shadcn registries before hand-rolling UI |
| `run-ui-registry-variations` | Manual: build 3 UI variations from shadcnblocks/Aceternity/Darkmatter registries |
| `nextjs-to-rwsdk-migration` | Port Next.js App Router to RedwoodSDK on Cloudflare Workers |

### Browser automation

| Skill | Use for |
|-------|--------|
| `agent-browser` | Chrome/Chromium via CDP — Node.js/Rust workflows |

### Other

| Skill | Use for |
|-------|--------|
| `rust-best-practices` | Idiomatic Rust (Apollo GraphQL handbook): borrowing/cloning, error handling, testing, generics |
| `session-context-pipeline` | Hook-driven background pipeline: session summarizer, doc-brief injector, end-of-turn checklist reviewer |

### Runtime policies (auto-applied by agent client)

These are **not task skills** — they are consumed by the agent runtime to configure session behavior.

| Skill | When |
|-------|------|
| `using-superpowers` | Session start — establishes skill discovery and invocation protocol |
| `strategic-compact` | Long autonomous sessions with auto-compaction enabled |

**Removed since the last import:** `beads-setup`, `beads-linear-sync`, `brainstorming`, `caveman-commit`, `continuous-learning`, `dm-skill-creator`, `end-of-turn-review`, `kickoff-dm-design`, `receiving-code-review`, `requesting-code-review`, `run-meeting-summary`, `test-driven-development`, `triage` (moved to `inactive/` upstream — not currently shipped). `coding-standards`, `frontend-design`, `browser-use`, `neon-postgres`, `openchronicle-setup`, `hl-funding-analysis`, `caveman`, `caveman-review`, `compress`, `systematic-debugging`, `verification-before-completion`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents` no longer exist in `darkmatter/skills` at all (some are archived, unshipped, under `docs/00-inbox/skills/`). Don't reference any of these as available skills until they reappear upstream.

---

## Working conventions

1. **Use the standard command surface.** `./scripts/setup` before working; `./scripts/ci` before PRs.
2. **Reference ADRs when making architectural decisions.** Surface conflicts before proceeding.
3. **Secrets use SOPS.** Apply `sops-secret-access` skill; never print decrypted contents.
4. **Effect is the default for TypeScript services.** See `effect-typescript` skill and ADR-0005.
5. **Protobuf when crossing language boundaries.** Use `buf`, commit generated code (ADR-0003).
6. **One settings module per binary.** No scattered `process.env` reads (ADR-0005).
7. **Follow Standard Readme.** See ADR-0006 before writing or reviewing a README.
8. **No inline SQL in TypeScript.** Use Kysely/Drizzle per ADR-0007.
9. **No org-standard task tracker right now.** ADR-0001 (Beads) was retracted; use the harness's native tracking or the project's own documented tool.

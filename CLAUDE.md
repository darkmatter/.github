# Darkmatter — Claude Code Organizational Context

This file provides org-level instructions and context for Claude Code agents working across all darkmatter repositories. It applies to every repo in the darkmatter GitHub organization.

For full skills and ADR details: [darkmatter/skills](https://github.com/darkmatter/skills)

---

## What Darkmatter builds

Darkmatter is a small, polyglot engineering team shipping developer tools, crypto/DeFi infrastructure, and AI-native applications. Primary stack: TypeScript/Bun (most apps), Effect (functional TS), Rust (performance services), Python (tooling), Nix (dev environments and system config), React/Next.js (frontend).

---

## Architecture Decisions

These decisions apply to **every** darkmatter project repo unless the project explicitly documents an exception. Full ADR text lives in [darkmatter/skills/docs/adr](https://github.com/darkmatter/skills/tree/main/docs/adr).

### ADR-0001: Beads is the standard task tracker and agent memory store
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0001-beads-as-task-tracker-and-agent-memory.md)

Use `bd` (beads) for all task tracking and persistent agent memory. **Do not** use `TodoWrite`, `TaskCreate`, `MEMORY.md`, `TODO.md`, or `NOTES.md` for state that must survive a session.

| Command | Purpose |
|---------|--------|
| `bd prime` | Load task + memory context at session start |
| `bd create "task"` | Create a task |
| `bd ready` | List tasks with all blockers closed |
| `bd remember "insight"` | Store a memory |
| `bd memories <keyword>` | Query stored memories |
| `bd close <id>` | Close a task |
| `bd linear sync` | Sync bidirectionally with Linear |

When a repo lacks `.beads/`, apply the `beads-setup` skill before creating tasks.

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

### ADR-0004a: No reinvention
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-no-reinvention.md)

Before writing an implementation, check whether it's a solved problem.

1. **Prefer structured output over output parsing.** Check for machine-readable flags before parsing human-readable output.
2. **Search before writing.** If it would exceed ~20 lines or touches a well-known domain (encoding, escaping, formatting, parsing, cryptography, date/time, protocols), assume a library exists.
3. **A dependency is preferable to a private reimplementation.** Libraries have tests, handle edge cases, and receive upstream fixes.

### ADR-0004b: Application config in one typed settings module
**Status:** Accepted | [Full ADR](https://github.com/darkmatter/skills/blob/main/docs/adr/0004-typed-settings-module-decoupled-from-provider.md)

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
| `systematic-debugging` | Before proposing fixes for bugs or failures |
| `verification-before-completion` | Before claiming work is done |
| `definition-of-done` | Complex, multi-step tasks |

### Architecture & infrastructure

| Skill | Use for |
|-------|--------|
| `effect-typescript` | Effect services, Layers, typed errors, Schema, Alchemy deploys |
| `alchemy` | Alchemy v2 infrastructure (Cloudflare/AWS providers) |
| `nix-flake-organization` | Thin `flake/` public layer + `src/` implementation |
| `sops-secret-access` | SOPS-encrypted config, private registries |
| `repository-organization` | Repo layout, Standard README, ADR placement, agent context |

### Task and workflow

| Skill | Use for |
|-------|--------|
| `beads-setup` | Onboard a repo onto `bd` (run when `.beads/` is missing) |
| `beads-linear-sync` | Configure Beads ↔ Linear sync |
| `writing-plans` | Plan before implementation |
| `subagent-driven-development` | Execute plans via dispatched subagents |
| `finishing-a-development-branch` | Merge, PR, or cleanup after implementation |
| `dm-skill-creator` | Create a new team-wide skill |
| `requesting-code-review` | Dispatch code-reviewer subagent before merge |
| `receiving-code-review` | Evaluate review feedback rigorously before implementing |
| `codebase-cleanup` | Multi-pass refactor sweep (8 specialist subagents) |

### UI/Frontend

| Skill | Use for |
|-------|--------|
| `frontend-design` | Distinctive, production-grade UI |
| `ui-ux-pro-max` | Design system intelligence (styles, palettes, fonts, UX guidelines) |
| `vercel-react-best-practices` | React/Next.js performance |
| `nextjs-to-rwsdk-migration` | Port Next.js App Router to RedwoodSDK on Cloudflare Workers |

### Domain-specific

| Skill | Use for |
|-------|--------|
| `neon-postgres` | Neon Serverless Postgres |
| `openchronicle-setup` | Local-first agent memory (macOS) |
| `hl-funding-analysis` | Hyperliquid perp funding rate analysis |

---

## Working conventions

1. **Check for Beads first.** No `.beads/`? Apply `beads-setup` before creating tasks.
2. **Use the standard command surface.** `./scripts/setup` before working; `./scripts/ci` before PRs.
3. **Reference ADRs when making architectural decisions.** Surface conflicts before proceeding.
4. **Secrets use SOPS.** Apply `sops-secret-access` skill; never print decrypted contents.
5. **Effect is the default for TypeScript services.** See `effect-typescript` skill and ADR-0004b.
6. **Protobuf when crossing language boundaries.** Use `buf`, commit generated code (ADR-0003).
7. **One settings module per binary.** No scattered `process.env` reads (ADR-0004b).

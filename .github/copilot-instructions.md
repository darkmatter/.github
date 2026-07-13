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
| ADR-0007 | Type-checked SQL in TypeScript — no inline SQL strings or tagged templates. Use a schema-derived query builder/ORM. Prefer Kysely; Drizzle allowed when already present. |
| ADR-0008 | Per-language reference codebases under `references/` (currently `rust/`, `go/`, `typescript/`). Skills carry prose; references carry code. Precedence: project `.agent/` → `references/` → general idiom. |
| OTel | App code imports only OTel SDKs; provider wiring (`@sentry/*`, PostHog, etc.) lives in shared packages only. |

## Always apply

- `coding-standards` — any TypeScript/JS/React/Node code task
- `brainstorming` — before implementing anything non-trivial
- `test-driven-development` — before writing implementation code
- `diagnose` — before proposing fixes
- `definition-of-done` — any complex multi-step task

## Key skills by category

**Architecture:** `effect-typescript`, `alchemy`, `darkmatter-ts-toolchain`, `darkmatter-gitops-conventions`, `nix-flake-organization`, `sops-secret-access`, `choose-dev-entrypoints`, `rust-best-practices`, `repository-organization`, `zoom-out`, `improve-codebase-architecture`

**Code quality:** `requesting-code-review`, `receiving-code-review`, `codebase-cleanup`, `end-of-turn-review`, `writing-skills`

**Workflow:** `session-context-pipeline`, `finishing-a-development-branch`, `handoff`, `triage`, `grill-me`, `grill-with-docs`, `dm-skill-creator`, `find-skills`, `run-meeting-summary`

**UI/Frontend:** `frontend-design`, `ui-ux-pro-max`, `shadcn-registry-first`, `ui-component-architecture`, `vercel-react-best-practices`, `nextjs-to-rwsdk-migration`, `kickoff-dm-design`, `prototype`

**Browser automation:** `browser-use` (Python, persistent sessions), `agent-browser` (CDP, Node/Rust)

**Communication:** `caveman`, `caveman-commit` (compact commit messages), `caveman-review` (compact reviews), `compress`

**Domain:** `neon-postgres`, `openchronicle-setup`, `hl-funding-analysis`

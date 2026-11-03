# user-tailored-planner

> Kiroween 2026 — **Skeleton Crew** entry. The **Planner**: a task-planning app that
> adopts the shared [user-tailored-ui](https://github.com/taylor-made-labs/user-tailored-ui)
> foundation to prove it in a real workflow.

**Status:** vertical slice implemented (spec tasks 1–9). Task workflow, policy, agenda
customization, the four rejected-violation fixtures, and save/reopen/undo are built and
tested; CI is the source of truth that it runs. Task 10 (vendoring a runnable copy into
`user-tailored-ui/apps/planner/`) is a later, separate step. We do not overstate
reliability — see [Limitations](#limitations).

## What the Planner is

A task-planning app. Users **create, edit, organize, and complete tasks** — a real,
persistent workflow — and then use an **in-app AI agent** to **customize a board into a
focused daily agenda**, in natural language. The agent's proposal is **untrusted**: it is
validated against the creator's policy before anything is applied. A violating request is
**rejected with an explanation**, and the last valid view is preserved.

```
request → agent proposal → deterministic validation → preview → apply → save → reopen → undo
```

Task identities, contents, actions, and the required complete-task control are preserved
across any customization.

## How it fits the foundation (dogfooding)

The Planner **adopts the foundation through its public exports and ports only** — it does
not reach into foundation internals and does not copy the Inventory app. The shared
customization logic, domain contracts, and the three ports (inference, storage, UI
capability) live in the foundation repo.

### How the foundation is consumed (public API only)

The `@taylor-made-labs/user-tailored-ui` package is **not published to npm yet**, and CI
runs `npm ci` in an isolated runner with no access to the sibling checkout — so a `file:`
dependency cannot resolve there. To keep CI self-contained while still consuming **only**
the foundation's public surface, this repo **vendors the foundation's public entrypoint**:

- `src/foundation/` is a faithful copy of the foundation's own `src/index.ts` and exactly
  the modules it re-exports (the customization types, policy, validator, orchestrator, the
  three ports, and the two labeled test-double adapters). **No foundation internals** are
  copied.
- `src/ports/foundation.ts` is the **single seam**: every Planner module imports the
  foundation through this one file, never from `src/foundation/**` directly. When the
  foundation publishes, this file becomes `export * from '@taylor-made-labs/user-tailored-ui'`
  and `src/foundation/` is deleted — **no other Planner source changes**. That is the point
  of the port.

This matches the spec's allowance to vendor *just the public entrypoint* behind a port.

### Architecture (ports & adapters)

| Layer | Location | Depends on |
|---|---|---|
| Domain (pure) | `src/domain/task.ts` | nothing (no React/model/storage) |
| Creator policy + views | `src/customization/` | foundation types only |
| Business persistence (tasks) | `src/ports/task-store.ts` + `src/adapters/task-store-*.ts` | IndexedDB (browser) / in-memory (tests) |
| Customization history | foundation `CustomizationHistoryStore` (in-memory double; browser-repos later) | foundation port |
| UI capability (board/agenda/task-card) | `src/adapters/ui-capability.ts` | foundation `UiCapability` port |
| Orchestration | `src/customization/service.ts` | foundation `CustomizationOrchestrator` |
| React 18 + Vite UI | `src/App.tsx`, `src/ui/**` | the layers above |

Business task data and customization state use **separate stores** and never mix. Model
output is **data** (validated config + tokens + approved-component refs) — never executed.

## How it fits the project (5 repos)

| Repo | Role |
|---|---|
| [user-tailored-ui](https://github.com/taylor-made-labs/user-tailored-ui) | Reusable skeleton, shared customization, contracts, main submission |
| [TailoredUI](https://github.com/taylor-made-labs/TailoredUI) | Fork — reusable UI capability (MIT, upstream: Aman Singh) |
| [browser-repos](https://github.com/taylor-made-labs/browser-repos) | IndexedDB-backed Git library (independent, reusable) |
| **user-tailored-planner** (this) | Task-planning app (first dogfooding app) |
| [user-tailored-inventory](https://github.com/taylor-made-labs/user-tailored-inventory) | Inventory app |

## Tech surface

TypeScript (strict) + React 18 + Vite for the web app surface; `vitest` for tests.
Dependencies are pinned to exact versions with a committed lockfile.

## Getting started

Build/test is verified in **GitHub Actions CI** (the dev sandbox has no public network).

```bash
npm ci
npm run typecheck
npm run lint
npm run test        # deterministic domain + foundation-integration suite
npm run build       # tsc --noEmit + vite build
npm run dev         # runnable web app (board + agent panel)
npx tsx src/demo.ts # headless end-to-end demo (offline, no secrets)
```

The default agent is the **labeled, offline test-double** (`providerId: "test-double"`,
`simulated: true`) — no API key, deterministic, safe for CI and demos. The real Anthropic
adapter is a later, port-compatible swap and needs no Planner code change. Every simulated
result is labelled **SIMULATED** in the UI.

### Try the agent

In the agent panel, ask for a *"focused daily agenda"* to apply a valid customization, or
use the quick-pick buttons to trigger each of the four policy violations and see the
inline, cited rejection while the last valid view is kept.

## Licensing

Dual-licensed under either **MIT** ([LICENSE-MIT](LICENSE-MIT)) or
**Apache-2.0** ([LICENSE-APACHE](LICENSE-APACHE)), at your option. Third-party and upstream
attributions are in [NOTICE](NOTICE).

## Limitations

- Customization enforcement covers an explicit, **finite** set of invariants (allowed
  actions, the required complete control, the component allow-list, and the font-scale /
  density token limits) — **not** universal correctness.
- The default agent is the **offline test-double**: deterministic keyword matching, not
  real natural-language understanding. The real Anthropic adapter (variable, networked) is
  a later gated step. Validation is deterministic and is the actual guarantee regardless of
  which agent produced the proposal.
- The customization-history store is the in-memory double in this slice; the
  browser-repos / IndexedDB-backed store is a later, port-compatible swap. Business **task**
  data already persists in IndexedDB.
- Multi-user accounts/auth are out of scope; user separation is shown via scoped ids.

## Kiro & Kiroween 2026

Built with [Kiro](https://kiro.dev) using a spec-driven workflow for **Kiroween 2026**.
Specs, steering, and development evidence live in the public [`.kiro/`](.kiro/) directory.

# user-tailored-planner

> Kiroween 2026 — **Skeleton Crew** entry. The **Planner**: a task-planning app that
> adopts the shared [user-tailored-ui](https://github.com/taylor-made-labs/user-tailored-ui)
> foundation to prove it in a real workflow.

**Status:** seed scaffold. This first commit is the project skeleton (structure, licensing,
CI, public `.kiro/`). The actual Planner build lands in a later task; sections marked
_(planned)_ are not yet implemented. We do not overstate reliability — see
[Limitations](#limitations).

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

> **Note on the foundation dependency.** The `@taylor-made-labs/user-tailored-ui` package is
> **not published yet**, so this seed does **not** declare a dependency on it. Once the
> foundation publishes a versioned package, the Planner will consume it as a **pinned**
> dependency behind its interfaces. Until then this repo only carries the app scaffold.

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

## Getting started _(planned)_

Build/test is verified in **GitHub Actions CI** (the dev sandbox has no public network).

```bash
npm ci
npm run test       # deterministic
npm run typecheck
npm run build
npm run dev        # runnable web app (planned)
```

## Licensing

Dual-licensed under either **MIT** ([LICENSE-MIT](LICENSE-MIT)) or
**Apache-2.0** ([LICENSE-APACHE](LICENSE-APACHE)), at your option. Third-party and upstream
attributions are in [NOTICE](NOTICE).

## Limitations

- This is a **seed**: no task workflow or customization is implemented yet.
- Once built, customization enforcement will cover an explicit, finite set of invariants —
  **not** universal correctness. The agent's natural-language interpretation is real and
  therefore variable; validation is deterministic and is the actual guarantee.

## Kiro & Kiroween 2026

Built with [Kiro](https://kiro.dev) using a spec-driven workflow for **Kiroween 2026**.
Specs, steering, and development evidence live in the public [`.kiro/`](.kiro/) directory.

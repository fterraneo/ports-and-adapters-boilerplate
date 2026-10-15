# Agent Guidelines

Guidelines for any agent or model developing on this repository. This file is the
single source of truth for the development principles below. Keep it updated: when a
decision on these topics changes, append an entry to the [Decision Log](#decision-log).

- [Commands](#commands)
- [Commit rules](#commit-rules)
- [Code principles](#code-principles)
- [Architecture](#architecture)
- [Testing](#testing)
- [TypeScript](#typescript)
- [Decision Log](#decision-log)

## Commands

| Command             | Purpose                                  |
| ------------------- | ---------------------------------------- |
| `npm test`          | Run the full test suite                  |
| `npm run test:w`    | Run tests in watch mode                  |
| `npm run tc`        | Typecheck (`tsc --noemit`)               |
| `npm run tc:w`      | Typecheck in watch mode                  |
| `npm run build`     | Clean and compile to `.build`            |
| `npx eslint .`      | Lint (flat config in `eslint.config.js`) |
| `npx prettier .`    | Format check                             |

Run `npm test` and `npm run tc` before every commit. A unit of work is only complete
when both pass.

## Commit rules

### Microcommits

- Every commit is a single, atomic, self-contained change: one logical change per
  commit. Never bundle unrelated changes.
- Commit as soon as a unit of work is complete. A unit of work is the smallest change
  that leaves the codebase in a working state (tests green, typecheck clean).
- Prefer many small commits over few large ones. If a change can be split into steps
  that each leave the codebase working, split it.
- Do not mix formatting or refactoring churn with a behaviour change.

### Message format

Use the [Conventional Commits](https://www.conventionalcommits.org/) standard:

```
type(scope): imperative summary
```

- `type` is one of: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`,
  `build`, `ci`, `style`, `revert`.
- The summary is imperative mood and resumes **what changed**, not what it does in
  the past tense: use `add cache eviction`, not `added cache eviction` or `adds cache
  eviction`.
- Scopes are optional and are not enforced by any hook. Use one only when it adds
  clarity.

### Co-author

Append the current agent/model as a co-author on every commit:

```
Co-authored-by: github-copilot/claude-haiku-5.5 <noreply@github.com>
```

### Pushing

**Mandatory: do not push.** Never run `git push`, `git push --force`, `git push
--atomic`, or any other command that publishes commits to a remote. Do not open pull
requests, create releases, or comment on issues via `gh`. Only the human pushes to
the remote repository.

Agents may stage and commit locally. Leave the commits on the local branch for the
human to review and push.

## Code principles

- **SOLID:** each principle applies to the layer you are working in. In particular,
  favour small interfaces with a single caller-driven reason to change (Interface
  Segregation) and depend on abstractions, not on concretions (Dependency Inversion).
- **KISS:** the simplest solution that satisfies the requirement wins. Avoid
  speculative generality, premature abstraction, and clever indirection. YAGNI
  applies: do not build what the current requirement does not need.
- Keep functions small and at one level of abstraction. Name things after what they
  mean in the domain, not how they are implemented.

## Architecture

We use **Hexagonal Architecture** (Ports and Adapters) to keep the core domain with
zero infrastructure dependencies.

- The **core** (domain and application layer) must have no infrastructure
  dependencies: no framework, no driver, no SDK, no database client, no filesystem,
  no clock or random source provided by the runtime, no direct network calls.
- Use the **Dependency Inversion** principle: define interfaces (**ports**) at the
  core domain level, expressed in the language of the domain. Adapters live at the
  edge and implement those ports. The core depends only on the ports it declares.
- Infrastructure enters the application only through an adapter, and is wired at the
  outermost composition root. The core never names an adapter, and never imports
  from one.
- Keep the folder structure free-form for now: no canonical layout is prescribed.
  When you introduce a structure, record it in the Decision Log first.

## Testing

We follow **Outside-In TDD** (double-loop): start from the outside with large
acceptance tests, then drop inside for unit tests where they are needed.

- Start with an acceptance test that describes the behaviour from the outside. It
  drives the whole system through its public surface.
- Acceptance tests **never** use real infrastructure. They use in-memory adapters
  implementing the same ports the production adapters implement. No network, no
  filesystem, no real database or clock in a test.
- Add unit tests only when a unit needs to be driven or verified on its own. Do not
  write unit tests for trivial delegation or for anything the acceptance test already
  pins down.
- Unit tests are colocated next to the code under test and named `*.test.ts`
  (for example `src/orders/order-service.test.ts`).
- Use [`testdouble`](https://github.com/testdouble/testdouble.js) for all test
  doubles: `td.function()`, `td.replace()`, `td.object()`. Prefer real in-memory
  implementations of your own ports over mocked internals; use testdouble when you
  need to isolate a single collaborator.

### Running tests

Jest is configured with `injectGlobals: false`, so test files must import their
globals:

```ts
import { describe, expect, it, afterEach } from "@jest/globals";
import * as td from "testdouble";

afterEach(() => td.reset());
```

Call `td.reset()` in an `afterEach` so state does not leak between tests.

## TypeScript

- `tsconfig.json` must keep `"strict": true`.
- Do not introduce `any`. Use `unknown` and narrow it when the type is genuinely not
  known. If `any` is unavoidable, document why in the code and note it in the
  Decision Log.
- Prefer types that make invalid states unrepresentable.

## Decision Log

Append-only record of decisions taken on the topics above (commit rules, code
principles, architecture, testing, TypeScript). Newest entries at the bottom.

When you take or change a decision in one of these areas, add an entry in this
format: `- YYYY-MM-DD — <topic> — <what was decided and why>`. Do not rewrite or
remove past entries; supersede them with a new one.

- 2026-10-15 — repository — Adopted `AGENTS.md` as the canonical agent guideline file,
  with `CLAUDE.md` importing it, so all agents and models share one context.
- 2026-10-15 — commit rules — Microcommits plus Conventional Commits, every commit
  carrying `Co-authored-by: github-copilot/claude-haiku-5.5 <noreply@github.com>`.
  Scopes are optional and not enforced by any hook.
- 2026-10-15 — commit rules — Pushing is forbidden to agents: only the human pushes to
  the remote repository.
- 2026-10-15 — architecture — Hexagonal Architecture with ports defined in the core
  and adapters at the edge; no canonical folder structure is prescribed yet.
- 2026-10-15 — testing — Outside-In TDD with acceptance tests driven by in-memory
  adapters, colocated `*.test.ts` unit tests, and `testdouble` for doubles.

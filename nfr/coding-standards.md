# Coding Standards

> Owner: Vervian · Last reviewed: 2026-07-18
>
> **Scope:** All Vervian product code. Default stack is TypeScript with React (Next.js) on the front end and Node on the back end. Where a product uses something else, the principles still apply; the language-specific rules adapt.

This document exists for one reason: **the code Vervian ships is built with heavy AI assistance, and AI-written, AI-reviewed code rots into "dark code" that nobody actually understands.** That is the failure mode we design against. The codebase must stay clean, reused, and documented well enough that a human can pick up any block and know exactly what it does, who calls it, and where its output goes.

---

## The two unbreakable rules

These are not suggestions. A change that violates either is wrong and does not merge.

### 1. Reuse before you create. Never reinvent.

- **Never build a new component, hook, utility, pattern, or abstraction if one already exists.** Look to the design system and the existing codebase first, every time.
- New UI is composed from existing design-system primitives. If a primitive is missing, you add it **to the design system** (with owner approval), not inline in an app.
- Apps contain page routes, API handlers, app-specific compositions, and business logic. They do **not** redefine buttons, inputs, cards, typography, layout primitives, or any other atom.
- Before introducing any new dependency or pattern, state what already-present option you considered and why it did not fit. "I didn't check" is not an answer.

See [`design-system.md`](../strategy/design-system.md) and the reuse rule in `agent-guide.md`.

### 2. Atomic design. One change cascades everywhere.

We use atomic design so that **a single edit propagates across the entire application.** That is the whole point: it minimizes the load of the codebase and keeps every surface consistent.

- UI atoms and tokens live in one place (the design system / `packages/ui` style package). Apps import them; they never copy or fork them.
- **Zero hardcoded design values.** Every color, font size, font weight, spacing, padding, margin, gap, border radius, shadow, line height, and z-index references a canonical token. No magic numbers, no one-off hex values.
- If two components of the same kind use different values for the same thing, that is a bug. Fix it at the token/primitive level, not by patching one instance.
- No file-copy sync scripts between the design system and apps. The shared package is the mechanism; copying defeats it.

---

## Before you write code: the build-decision ladder

Every change starts at the cheapest option that works and only moves down a rung when the one above genuinely does not fit. Walk it in order, and state in the PR where you landed:

1. **Does this need to exist?** If the requirement can be met without new code, meet it without new code. The best change is often no change.
2. **Already in this codebase?** Reuse it. This is rule 1 above.
3. **Standard library?** Use it before adding anything.
4. **Native platform feature?** Prefer the platform over a new dependency.
5. **An already-installed dependency?** Use what is already here before pulling in something new.
6. **One line?** If a one-liner does it, write the one line, not an abstraction around it.
7. **Only then:** the minimum that works.

A new dependency is the second-to-last resort, not the first move. Adding one requires the same "what already-present option did you consider, and why did it not fit" answer that rule 1 demands (and it carries the checklist's license/SBOM obligations, see [`00-pr-nfr-review-checklist.md`](00-pr-nfr-review-checklist.md)).

**The ladder trims solutions, never safety.** These are out of scope for reduction, always. Do not cut them to make a diff smaller:

- Trust-boundary and input validation (see [`security.md`](security.md)).
- Error handling and data-loss handling.
- Security controls (see [`security.md`](security.md)).
- Accessibility (see [`accessibility.md`](accessibility.md)).

**Lazy about the solution, never about reading.** The ladder runs *after* you understand the problem, not instead of it. Read the code the change touches and trace the real flow before picking a rung. A small diff built on a misread of the code is worse than no change.

---

## No dark code: the documentation standard

Every meaningful block of code carries documentation that lets a human understand it without reverse-engineering it. This is the direct, deliberate counter to dark code.

**Every module, and every non-trivial function/component, documents:**

1. **What it does**, in one plain-English sentence. No jargon for its own sake.
2. **Consumers** who calls this / what relies on it.
3. **Inputs** what data comes in, and from where.
4. **Outputs and side effects** what it returns, what it writes, and where that goes next (the downstream).
5. **Why**, when the reason is not obvious from the code.

Use this header shape (adapt to the language; this is the TS/JS form):

```ts
/**
 * Formats a raw receipt record into the shape the report view renders.
 *
 * Consumers:  ReportView (apps/web), report PDF generator (worker)
 * Inputs:     RawReceipt from the receipts table (via getReceipt)
 * Outputs:    ReportLine[]; pure, no side effects
 * Why:        the view and the PDF must show identical lines, so the mapping lives here once
 */
```

Rules for the documentation itself:

- It describes **intent and contract**, not a line-by-line restatement of the code. Comments that just echo the code are noise.
- It stays true. A comment that has drifted from the code is worse than none. Update docs in the same change as the code.
- If you cannot clearly explain what a block does and who depends on it, the block is **not done.** Unexplained code does not ship.
- Public/exported functions, React components, API route handlers, and shared utilities always get a header. Trivial one-line helpers can be self-documenting by name.

---

## Don't repeat yourself

- No duplicated logic. If the same thing exists in two places, extract it to one and reference it.
- If you find existing duplication while working nearby, flag it. Removing it may belong in a separate change, but it does not get ignored. Duplicate-code detection is part of code review (see [`00-pr-nfr-review-checklist.md`](00-pr-nfr-review-checklist.md) and `/vervian-code-review`), which sits alongside these standards rather than only inside them.

---

## React / Next.js

- Components are small, composable, and single-purpose. A component doing three jobs is three components.
- Design-system-first (rule 1). Compose primitives; do not restyle them inline.
- Respect the server/client boundary. Keep components server-rendered by default; reach for `"use client"` only when interactivity requires it. Never leak secrets into client components or `NEXT_PUBLIC_` values (see [`security.md`](security.md)).
- Follow the rules of hooks. No conditional hooks, no hooks outside components.
- Keep state close to where it is used. Lift only when genuinely shared. Avoid prop-drilling sprawl and avoid reaching for global state when local will do.
- Props are typed. No untyped `any` props.
- Accessibility is part of "done" for any UI, see [`accessibility.md`](accessibility.md).

## Node / back end

- Route handlers stay thin: validate, authorize, delegate to a service, return. Business logic lives in services, not in the handler.
- Validate all input at the boundary (schema validation). Never trust the client. See [`security.md`](security.md).
- Authorize every protected operation explicitly. Authentication is not authorization.
- Handle errors deliberately. No silently swallowed errors, no leaking internals to the client.
- Structured logging, and **never log PII or secrets** (see [`privacy-gdpr.md`](privacy-gdpr.md)).
- No secrets in code or client bundles. Configuration comes from the environment.

## TypeScript

- `strict` on. `any` requires a written reason; prefer `unknown` plus narrowing.
- Share types from one source; do not redeclare the same shape in two files (rule 1 again).
- Make illegal states unrepresentable where the type system allows it.

---

## How we verify

- The build-decision ladder was walked: the change is the smallest option that works, and any new dependency states why the platform, stdlib, or an existing dependency did not fit. Safety (validation, error handling, security, accessibility) was not cut to shrink the diff.
- The change reads as part of the surrounding code: same naming, same patterns, same idioms.
- Lint and type-check pass. No new warnings.
- Every new exported function, component, handler, and shared utility has the documentation header above.
- No new hardcoded design values; tokens only.
- No new duplication; reuse was checked and stated.
- `/vervian-code-review` and the [`PR NFR review checklist`](00-pr-nfr-review-checklist.md) are the gate.

## Sources

- Atomic design (Brad Frost) for the component model.
- The reuse-first and no-dark-code posture is Vervian's, see `core-beliefs.md` and [`product-management.md`](../strategy/product-management.md).
- The build-decision ladder and the "trims solutions, never safety" carve-out are adapted from ponytail (github.com/DietrichGebert/ponytail, MIT), reframed to Vervian's reuse-first posture. Its benchmark claims are not adopted as commitments.

# Testing Strategy

> Owner: Vervian · Last reviewed: 2026-06-05
>
> **Status:** Stub. Sets the **org-level** testing posture every Vervian product follows. Detailed per-product test suites (unit-test architecture, specific frameworks, fixtures, regression coverage) live in product repos. This document declares the floor, not the ceiling.

---

## Scope

The minimum testing every Vervian product is expected to maintain to ship. Where a product has tighter requirements, they go into that product's override or its own per-product test strategy doc.

---

## 1. Layers and minimums

Every shipped Vervian product maintains:

- **Unit tests** for the core domain logic of its codebase. Coverage is not the goal; coverage of the *risk* is. A green test suite where the failure modes are untested is worse than honesty about the gaps.
- **Integration tests** for the boundaries that matter: file I/O, external service calls, integrations, IPC between modules.
- **End-to-end / acceptance tests** for the documented critical journeys (see § 4).
- **Performance benchmarks** for the documented hot paths (see [`performance.md`](performance.md#5-measurement-and-evidence)).
- **Accessibility checks** for in-scope UI surfaces: automated (axe-core or equivalent) plus per-release manual keyboard / screen-reader sweep (see [`accessibility.md`](accessibility.md)).
- **Security scanning**: dependency vulns and secret scanning at minimum, run in CI (see [`security.md`](security.md#6-dependencies-and-supply-chain)).

What's not on this list is also intentional. We don't mandate end-to-end coverage of every UI path or 100 % unit-test coverage; those are decisions that should fit the product.

## 2. Risk-based, not coverage-based

The org doesn't set a coverage percentage. It sets an expectation that **the test suite reflects the risk profile of the code**. Hot paths, integration boundaries, security-relevant code, and critical paths all have stronger coverage than internal utility helpers.

The PR review (per [`00-pr-nfr-review-checklist.md`](00-pr-nfr-review-checklist.md)) asks "what could this change break, and how would we know?" Testing answers that question.

## 3. Test environments

> *To be defined per product.* Discuss: local, CI, staging/QA env, production-mirror env. Where does each test layer run? What's the data story (per [`data-classification-retention.md`](data-classification-retention.md): no real customer data in non-production environments without explicit handling).

## 4. Critical journeys

Each product names its **critical journeys** (5 to 10 user paths, written down once per product) that get the focused testing. Suggested journey categories:

- Login / licensing / entitlement check.
- Primary creation / authoring flow (one per major persona).
- Save / load / export of customer data.
- Settings change with side effects (for example account, payment, sharing).
- Error recovery (lost connection, invalid input, expired session).

The journey list is the canon for accessibility manual passes, end-to-end automated tests, and pre-release sanity checks.

## 5. Critical paths get more

Anything declared critical per [`reliability-availability.md`](reliability-availability.md#1-the-critical-path-principle) is tested under failure conditions, not just happy path. Network down, third-party gone, connection lost, disk full, mid-session restart attempted. The test suite includes the unhappy cases that would cause data loss, break a core flow, or trigger an irreversible action.

## 6. AI-generated test code

Tests proposed by AI assistants are reviewed by a human. AI-written tests that pass a broken implementation are common; the reviewer's job is to verify the test actually exercises the failure mode it claims to.

## 7. What's tested informs what's documented

A "tested" claim in a release note or NFR document is a load-bearing claim. If we say something is tested, the test exists, runs, and would fail if the behaviour regressed. If we don't have such a test, we say "verified by manual inspection at release"; honesty preserves trust.

## 8. Performance and accessibility are tests, not afterthoughts

Both are first-class test layers, not gates appended at the end. A performance regression on a documented hot path or an accessibility regression on a critical journey is the same severity of break as a logic regression.

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Test layers present | Per-product test strategy doc names what runs where |
| Critical journeys defined | Listed in product's own test doc; referenced from this strategy |
| Risk coverage | Reviewed per release; gaps recorded as known limitations |
| Critical-path failure tests | Chaos / failure-condition test runs documented for relevant products |

## Sources

- [`performance.md`](performance.md), [`accessibility.md`](accessibility.md), [`security.md`](security.md), [`reliability-availability.md`](reliability-availability.md): the NFRs this strategy supports.
- Industry: the Test Pyramid as a starting point, not a religion.

## Open items

- [ ] Fill in § 3 (test environments) once tooling is settled.
- [ ] Each product names its critical journeys and links here.
- [ ] Define org-level CI baseline (what jobs run on every PR vs every nightly vs pre-release).
- [ ] Reconcile with [`release-management.md`](release-management.md) gates: duplicate-of-truth checks.

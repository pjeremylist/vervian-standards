# Release & Change Management

> Owner: Vervian · Last reviewed: 2026-06-05
>
> **Status:** Stub. Several NFRs already reference release behaviour (signed installers, autoupdate respecting a do-not-disturb mode, rollback paths, hotfix policy) but no document owns the end-to-end flow of how a change becomes a customer release. This document fills that gap. The numbered sections below are the structure to be filled in.

---

## Scope

How a code change moves from PR-merged to in-customer-hands at Vervian. Applies to every shipped product (web/SaaS surfaces, installed desktop, anything else). Each product may layer specifics on top, but the cadence, gates, and approval pattern come from here.

---

## 1. Release cadence

> *To be defined per product.* Examples to discuss:
> - Web/SaaS surfaces: continuous deployment with feature-flagged rollout.
> - Installed / desktop products: regular cadence with a hotfix lane.
> - Cadence aligned with customer planning where the deployment context demands it.

The cadence is a product decision but is **declared** so customers and the team can plan against it.

## 2. Pre-release gates

Every release must clear:

- [ ] All NFR checklist items from [`00-pr-nfr-review-checklist.md`](00-pr-nfr-review-checklist.md) for the cumulative changes in the release.
- [ ] Test suite per [`testing-strategy.md`](testing-strategy.md).
- [ ] Security scan (dependencies, secrets) per [`security.md`](security.md#6-dependencies-and-supply-chain).
- [ ] Accessibility audit per [`accessibility.md`](accessibility.md) on UI-touching changes.
- [ ] Performance benchmarks recorded per [`performance.md`](performance.md#5-measurement-and-evidence).
- [ ] Connectivity-model claims still hold (see [`reliability-availability.md`](reliability-availability.md#2-connectivity-model)).
- [ ] Release notes drafted.

## 3. Approval

This is a single-owner repo. The owner commits directly or opens a PR and merges it. There is no two-approver gate. The discipline is in the checklist (§ 2), not in a sign-off ceremony.

General guidance that still applies: do not push directly to protected branches; let CI run the gates before a change ships.

## 4. Signing and distribution

- Installers signed per [`security.md`](security.md#4-code-signing-and-installers).
- Distribution channels (auto-update servers, download portals, app stores) documented per product.
- Pre-release / beta channels clearly labelled and not auto-promoted.

## 5. Rollout

> *To be defined.* Discuss: phased rollout vs all-at-once, percentage rollout for SaaS, customer notification, change windows for products deployed in sensitive contexts.

## 6. Rollback

Every release has a documented rollback path. **A release that can't be rolled back without manual customer intervention has not cleared the gates.** Tested before the release ships, not when it's needed.

## 7. Hotfix flow

When a regression or incident requires shipping faster than the regular cadence:

> *To be defined.* Critical questions: how is "hotfix" declared? Which gates can be deferred (none in § 2 are skipped) and which run in an abbreviated form? The release still passes the gates before it ships.

## 8. Do-not-disturb interaction

Releases do not interrupt active customer use. Where a product's deployment context demands it, the "do not disturb" semantics in [`reliability-availability.md`](reliability-availability.md#5-update-mechanism) govern when an update may prompt, download, or install. A product whose context requires this declares it in its product override (see `product-overrides/`).

## 9. Communication

- **Internal:** release notes drafted alongside the change, surfaced before the release goes out.
- **External:** customer-facing notes published on the documented channel (changelog page, in-product banner, email, per product).
- **Incidents:** any release that causes a customer-impacting incident triggers the post-release flow in [`security.md`](security.md#9-incident-response) and/or [`reliability-availability.md`](reliability-availability.md).

## 10. Versioning

> *To be defined per product.* Semver vs date-based vs build-number, decided per product context.

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Pre-release gates cleared | Release checklist artifact attached to the release record |
| Rollback path | Tested in pre-release; recorded |
| Hotfix flow | Reviewed periodically with a tabletop exercise |
| Cadence honoured | Release history matches declared cadence (drift is a flag, not a defect) |

## Sources

To be filled in as this document is fleshed out. Likely sources:

- Past Vervian releases that inform current practice (linked once consolidated). TBD.
- Industry release-management references (release trains, progressive delivery) where relevant to Vervian's mix of installed and SaaS products.

## Open items

- [ ] Fill in sections 1, 5, 7, 10 (TBD currently).
- [ ] Confirm gates in § 2 cover the actual current pre-release checks; add anything missing.
- [ ] Document the per-product release cadence.

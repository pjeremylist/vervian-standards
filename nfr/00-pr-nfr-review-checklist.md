# PR Non-Functional Review Checklist

> Owner: Vervian · Last reviewed: 2026-06-05

This checklist is the **gate** every pull request goes through. Skim it on every PR. For any item that *might* apply, click into the linked NFR or strategy doc and confirm.

If you are an AI assistant proposing a PR, walk this checklist explicitly in the PR description: list which items applied, which didn't, and why.

---

## How to use this

- The **author** runs the checklist before opening or merging the PR.
- This is a single-owner repo: the owner commits directly or opens a PR for non-trivial changes. There is no required second approver, so the author is also the reviewer. Run the checklist with that in mind.
- Items marked **(blocking)** must be resolved before merge. Other items are flags for discussion.
- A "no, doesn't apply" answer is fine. What's not fine is silence.

---

## Security · see [`security.md`](security.md)

- [ ] **(blocking)** No secrets, tokens, API keys, or credentials in the diff (including test fixtures and example configs).
- [ ] **(blocking)** Any new dependency has a known license class (see `legal-compliance.md`) and is in the SBOM.
- [ ] **(blocking)** Any new network exposure (port, endpoint, listener) has documented auth and authorization.
- [ ] Input from untrusted sources (network, file, user) is validated before use (see [`security.md`](security.md#11-application-security)).
- [ ] New public or per-user-costly endpoints have rate limiting / abuse controls.
- [ ] Any feature where an LLM or agent processes untrusted content treats that content as data, not instructions, and documents its prompt-injection exposure (see [`security.md`](security.md#10-ai-and-generated-code)).
- [ ] Logs do not contain secrets, tokens, or PII (see [`privacy-gdpr.md`](privacy-gdpr.md)).
- [ ] If the change touches the installer/update path: signed correctly, no privilege escalation regression.
- [ ] If the change touches CI/CD or release workflows: pipeline-integrity rules hold (see [`security.md`](security.md#12-build-pipeline-and-release-integrity)).

## Privacy · see [`privacy-gdpr.md`](privacy-gdpr.md)

- [ ] **(blocking)** No new personal-data field is collected without documented lawful basis and a retention rule.
- [ ] **(blocking)** No personal data is sent to a new third party (analytics, error reporting, AI provider) without DPA review.
- [ ] Telemetry additions can be disabled by the customer where required.
- [ ] DSAR (access / erasure / portability) flows still work for any new personal-data field.

## Data lifecycle · see [`data-classification-retention.md`](data-classification-retention.md)

- [ ] New data has a classification (Public / Internal / Confidential / Restricted).
- [ ] Retention rule defined for any new persisted data.
- [ ] Backups/exports include or exclude the new data consistent with its classification.
- [ ] Deletion paths cover the new data (account erasure, soft-delete, hard-delete).

## Performance · see [`performance.md`](performance.md)

- [ ] No regression in latency budgets relevant to the surface being changed.
- [ ] **(blocking for critical paths)** End-to-end latency target still met for any change on a latency-sensitive path.
- [ ] Bundle / install size impact stated in PR description if non-trivial.
- [ ] Resource use (CPU, memory, network) noted if the change is on a hot path.

## Reliability · see [`reliability-availability.md`](reliability-availability.md)

- [ ] **(blocking for critical paths)** A change does not introduce a new network dependency that breaks operation where offline or isolated operation is expected.
- [ ] Failure modes are explicit. The change either degrades gracefully or fails loudly, not silently.
- [ ] Autosave / recovery behaviour preserved for any change in the document/state pipeline.
- [ ] Update/installer changes do not introduce mid-session restart requirements.

## Architecture · see `architecture.md`

- [ ] **(blocking)** The deployment gate holds: this change does not put more than one distinct product on the same repo, app, process, machine, or database, and a failure in it cannot degrade an unrelated product sharing infrastructure.
- [ ] **(blocking)** No new paid infrastructure (app, machine, database, billed third-party service) is provisioned without the owner's approval.
- [ ] Any new service, deploy unit, or stack element has its decision recorded (what, why, alternatives, blast radius).
- [ ] Services touched still own their data stores exclusively; no new cross-service database access.

## Accessibility · see [`accessibility.md`](accessibility.md)

- [ ] New UI elements have keyboard equivalents.
- [ ] Colour is not the only carrier of meaning.
- [ ] Focus is managed (visible focus, logical order, returned focus after dialog dismissal).
- [ ] Contrast meets WCAG 2.2 AA where the product override doesn't carve out otherwise.

## Release & testing · see [`release-management.md`](release-management.md) and [`testing-strategy.md`](testing-strategy.md)

- [ ] Change is covered by tests appropriate to its risk (unit / integration / e2e).
- [ ] Release/versioning impact is understood and noted where relevant.

## Code quality & documentation · see [`coding-standards.md`](coding-standards.md)

- [ ] **(blocking)** Reuse was checked. No new component, hook, utility, or pattern was created where one already exists.
- [ ] **(blocking)** No hardcoded design values. Colours, spacing, type, radii, shadows, z-index reference tokens.
- [ ] **(blocking)** Every new exported function, component, API handler, and shared utility has the documentation header (what it does, consumers, inputs, outputs/side effects, why). No dark code.
- [ ] No duplicated logic introduced. Existing duplication found nearby is flagged.
- [ ] `strict` types hold. No unjustified `any`.

## Design system · see [`design-system.md`](../strategy/design-system.md)

- [ ] UI changes reuse existing system components and tokens rather than introducing one-offs or hard-coded values.

## Product overrides · see `product-overrides/`

- [ ] If applying a deviation from the baseline, the relevant override is cited by name.
- [ ] If introducing a *new* deviation, a PR to the standards repo is opened in parallel. **Do not** ship the deviation in product code alone.

---

## Related strategy docs

- [`design-system.md`](../strategy/design-system.md)
- [`product-management.md`](../strategy/product-management.md)
- `marketing.md` (for changes to marketing surfaces or copy)

---

## PR description template

Paste this into the PR description and fill it in:

```
## What
<one line>

## Why
<incident / ticket / customer / decision link>

## NFR checklist
- Security:   <which items applied, links to deeper notes>
- Privacy:    ...
- Data:       ...
- Performance:...
- Reliability:...
- Accessibility:...
- Overrides cited: <product-overrides/<product>.md section, or "none">

## Blast radius
<which products / customers / surfaces are affected>
```

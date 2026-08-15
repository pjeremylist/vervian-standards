# Security

> Owner: Vervian · Last reviewed: 2026-08-14

## Scope

Applies to every Vervian product, every internal service, every build pipeline. Where a product override exists in `product-overrides/`, the override applies *in addition to* (never instead of) these rules unless explicitly noted.

Vervian products run in environments where a compromise is not merely a data risk but a risk to core user workflows and to irreversible actions (data destroyed by ransomware, a compromised pipeline corrupting outputs, a leaked customer data file). The framing throughout is "a compromise on the critical path is the worst case, so design for it."

A **critical path** here means any flow where failure causes data loss, breaks a core user flow, or triggers an irreversible action.

### Separation of duties, and what replaces it

Vervian is one person. Several controls below assume two sets of hands: an author and an independent reviewer, an operator and an auditor. Where that separation does not exist, the control is **not waived**. It is met by a compensating control, and the substitution is written down rather than assumed.

| Control that assumes separation | Compensating control at single-owner scale |
| :--- | :--- |
| Independent review before merge | Automated gates that cannot be self-approved: required CI checks, SAST, secret scanning, branch protection. An AI assistant may review; it is never the approver (see [section 10](#10-ai-and-generated-code)). |
| Independent access review | Written credential list, reviewed on a fixed date, with the date and the changes recorded (see [section 2](#2-authorization)). |
| Segregation between the person who deploys and the person who approves | Release artifacts are built and deployed only by the release pipeline, never from a developer machine (see [section 12](#12-build-pipeline-and-release-integrity)). The pipeline is the second pair of hands. |
| Independent audit of whether controls actually operate | Annual self-audit (`/vervian-security-audit`) with findings tracked to closure, plus the evidence retained under [How we verify](#how-we-verify). |

Two consequences, both deliberate:

- **Vervian states the honest position.** A customer reading this sees compensating controls, not an implied team.
- **The same test applies to anyone Vervian assesses.** A small organisation is not non-compliant because it lacks role separation. The question is whether a compensating control exists and can be evidenced. Absence of headcount is not a finding. Absence of a control is.

### Client engagements

On a client engagement, the client's own security policy, tooling, and approved-AI list govern the work inside their environment. This document is the floor Vervian brings, not a replacement for their standard. Where the two differ, the stricter applies. Where Vervian's own systems are used on an engagement, `ai-tooling-policy.md` sets the boundary.

---

## 1. Authentication

- All authenticated surfaces use industry-standard auth: OAuth 2.0 / OIDC for web/SaaS, or signed license entitlements for installed products. No bespoke cryptographic schemes.
- Passwords (where used): minimum 12 characters, hashed with Argon2id or bcrypt (cost >= 12). Never stored or transmitted in plaintext.
- Failed-login throttling on every password or token-exchange endpoint: progressive delay or temporary lockout after repeated failures from the same account or source. The chosen mechanism and thresholds are documented per product.
- MFA **required** for all Vervian staff accounts (administrative, developer, and any account with production access). MFA *offered* to end customers on SaaS surfaces.
- Session tokens are scoped, time-bound, and revocable. No long-lived bearer tokens in shipped code. Default bounds (a product override may tighten or, with rationale, relax): access tokens expire within 1 hour where refresh tokens are used, within 24 hours otherwise; refresh tokens expire within 90 days; revocation takes effect within 15 minutes.

## 2. Authorization

- Default deny. Every action checks authorization at the server / authoritative layer, never client-only.
- Role separation between **end user** (uses the product), **administrator** (configures the deployment), and **Vervian support** (incident response).
- Every path that grants elevated privilege (role change, support impersonation, admin grant) is listed in the product's architecture doc and emits an audit-log event (per [section 8](#8-logging-and-audit)).
- **Access lifecycle:** keep a current, written list of every system and third-party tool that holds a Vervian credential (cloud, repos, secret stores, vendor accounts). *When Vervian has staff or contractors:* revoke their access within 24 hours of departure and review grants at least every 6 months. Single-owner today, so this is a forward rule; the credential list still gets maintained now.

## 3. Secrets management

- **No secrets in repositories.** Pre-commit and CI checks scan for high-entropy strings and common secret patterns.
- Production secrets live in a managed secret store (specific tool: TBD. Likely CI/CD pipeline variables for build-time, a vault product for runtime).
- Customer-supplied secrets (API keys for third-party integrations) are encrypted at rest with per-customer keys.
- Rotation cadence: production access credentials rotated at least every 90 days; signing keys rotated annually or on staff turnover.

## 4. Code signing and installers

Vervian ships installed desktop software for some products. The installer is itself a security boundary.

- **(blocking)** All shipped installers are signed with Vervian's production code-signing certificate.
- **(blocking)** The private signing key lives in a managed key store (cloud KMS, HSM, or the platform's certificate vault), never on a developer machine and never in a repository. Signing happens only from the release pipeline (see [section 12](#12-build-pipeline-and-release-integrity)).
- **(blocking)** Auto-update mechanisms verify signatures before applying updates. No update is applied from an unsigned source.
- Update servers are over HTTPS with certificate pinning where the platform supports it.
- Installers do not silently elevate privileges or install background services without explicit user consent at install time.
- See also [`reliability-availability.md`](reliability-availability.md). Updates must not interrupt an active critical-path operation.

## 5. Network exposure

- Every listening port / endpoint is documented in the product's architecture doc.
- No unauthenticated network surface in shipped products. Local-only ports (e.g. for inter-process comms on a single machine) must bind to loopback.
- TLS 1.2 minimum; TLS 1.3 where the platform supports it. No SSLv3, no TLS 1.0/1.1.
- Certificate validation is on by default and cannot be disabled by configuration in shipped products.

## 6. Dependencies and supply chain

- Every shipped product maintains a current **SBOM**, generated as part of the release pipeline.
- Dependency vulnerability scanning runs in CI; critical/high CVEs block release until triaged.
- License compliance per `legal-compliance.md`.
- Lockfiles committed. No floating versions in production builds.
- New dependencies require justification in the PR ("why is this better than what we have").

## 7. Third-party integrations

When connecting to an external service (analytics, error reporting, AI provider, payment, vendor API):

- Inventoried in the product's third-party processor registry (see [`privacy-gdpr.md`](privacy-gdpr.md#5-third-party-processors)).
- DPA in place if any personal data is transmitted.
- Failure of the third party does not break critical paths (see [`reliability-availability.md`](reliability-availability.md)).
- Outbound credentials use least-privilege scopes.

## 8. Logging and audit

- Security-relevant events (auth success/failure, privilege change, configuration change, data export) are logged with timestamp, actor, and outcome.
- Logs do **not** contain secrets, tokens, full credit card numbers, or unredacted personal data.
- Production logs are retained per the retention policy in [`data-classification-retention.md`](data-classification-retention.md).
- Logs are tamper-evident (append-only or signed) for security-investigation purposes.

## 9. Incident response

- Every Vervian product has a designated security contact. For routine matters the contact responds within one business day. For a suspected incident involving customer data, triage starts within 1 hour of the report, matching the breach flow in [`privacy-gdpr.md`](privacy-gdpr.md#7-breach-notification).
- Confirmed incidents involving customer personal data trigger the breach notification flow in [`privacy-gdpr.md`](privacy-gdpr.md#7-breach-notification) (72-hour clock under GDPR).
- Keep a one-page written incident plan per product: who to notify, how to contain, how to restore, and the breach clock. Review it at least annually. A formal tabletop exercise is a team practice for later, not a solo requirement.
- Post-incident: write-up within two weeks, NFR updates within four weeks if the incident revealed a gap.

## 10. AI and generated code

Two distinct exposures live here and they need separating: **AI used to build the software**, and **AI shipped inside the software**. The rules differ.

### 10a. AI used to build the software

- **(blocking)** All AI-generated code, configuration, and infrastructure-as-code is treated as untrusted input. It passes the same review, linting, and security scanning as human-written code before merge. No auto-merge of AI output.
- Code suggested by AI assistants is reviewed by a human before merge. AI is not an approver.
- Secrets, customer data, and proprietary algorithms are not pasted into external AI tools that don't carry Vervian's enterprise data-handling terms.
- Which AI systems are sanctioned, what data each may touch, and when each may be used are recorded in `ai-tooling-policy.md`. A system not on that register is not authorised for Vervian or customer data.

### 10b. AI agents as an attack surface

Agents (assistants with tool, file, or network access, in development or embedded in a product) are an attack surface, not just a data-egress risk:

- **(blocking)** Agents never hold or use production credentials. An agent that needs production data works through an audited, least-privilege, read-only path approved per case.
- Untrusted content an agent processes (customer tickets, uploaded files, web pages, third-party API responses) is treated as **data, not instructions**.
- Tools and MCP servers granted to an agent follow least privilege: only the tools the task needs, write access only where the task writes.
- Actions an agent takes against a Vervian system are logged with actor, tool, and outcome, per [section 8](#8-logging-and-audit).
- **(blocking)** An agent does not deploy to production, delete data, or take any other irreversible action without a human-approved gate.
- See `agent-guide.md` for AI assistant rules.

### 10c. AI shipped inside a product

Applies to any product feature backed by a model: chat, summarisation, extraction, retrieval-augmented generation, embedded agents.

- **(blocking) Feature record.** Before an AI-backed feature is approved, its design records four things: what data leaves the environment, which provider and model receive it, what actions the model is authorised to trigger, and the data classification of both inputs and outputs (per [`data-classification-retention.md`](data-classification-retention.md#classification-tiers)). Kept in the product's architecture doc.
- **(blocking) Prompt injection.** The feature documents its prompt-injection exposure and mitigations: input isolation, tool allow-lists, output validation, human confirmation for irreversible actions.
- **(blocking) Output handling.** Model output is never executed, rendered, or passed to a downstream sink raw. It is validated and encoded exactly like any other untrusted input, per [section 11](#11-application-security).
- **System prompts are not a security boundary.** No secrets, credentials, API keys, or security-critical rules are placed in a system prompt. Assume the prompt is extractable. Access control and filtering are enforced in the application layer, outside the model.
- **Retrieval and vector stores.** Where a product embeds documents for retrieval: enforce tenant isolation on the vector store, apply the same access control as the source documents, and sanitise content before embedding. A poisoned or cross-tenant retrieval is a data breach, not a relevance bug.
- **Model provenance.** Base models, fine-tunes, and embedding models are version-pinned and sourced from vetted providers. A model version is a dependency, and [section 6](#6-dependencies-and-supply-chain) applies to it.
- **Consumption limits.** Every model-backed endpoint has input and output token ceilings, a request timeout, and rate limiting (per [section 11](#11-application-security)). Cost and call-volume anomalies raise an alert. Unbounded model consumption is both a denial-of-service vector and a direct financial loss.
- **Adversarial testing before release.** Any AI-facing feature is tested against a prompt-injection and jailbreak suite before first release and before any change to its system prompt, tools, or model version. Failures at critical or high severity block release. The suite is maintained per product and grows with each real finding.

## 11. Application security

Baseline: **OWASP ASVS Level 1** for every product. A product that handles regulated data (health, payment, children's data) may be held to **Level 2** through its product override, where the extra effort is justified, not by default for everything. The rules below are the controls most often violated; ASVS is the full catalogue behind them.

**Design-time.** Insecure design is not fixable by code review, so it is caught before the code exists. Every new service, externally reachable feature, or AI-backed capability gets a written threat model before development starts. At Vervian's scale this is a short structured note, not a workshop: what an attacker would want here, how they would reach it, what stops them, and what is left unmitigated. Anything left unmitigated is either fixed, or recorded as an accepted risk in the product override with a review date. The note lives with the feature's design and is revisited when the design materially changes.

- **(blocking)** All input from untrusted sources (network, user, file, third-party API) is validated before use: type, length, range, format.
- **(blocking)** Database access uses parameterized queries or an ORM. No string-built SQL/NoSQL from user input.
- Output encoding appropriate to the sink (HTML, attribute, URL, shell). No user input reaches a shell command or `eval`-like construct without strict allow-listing.
- Outbound requests built from user-supplied URLs (webhooks, importers, link previews) are SSRF-guarded: scheme and host allow-listing, no redirects to private address ranges, no access to cloud metadata endpoints.
- File uploads: validate type by content (not extension alone), enforce size limits, store outside the web root with non-executable permissions, and never serve back with a user-controlled content type.
- Public, unauthenticated, or per-user-costly endpoints (login, signup, password reset, export, AI inference) are rate-limited. Limits documented per product.
- CSRF protection on every state-changing browser-facing endpoint that uses cookie-based auth.
- Data classified Confidential or Restricted is encrypted at rest, using platform-managed or customer-managed keys per [`data-classification-retention.md`](data-classification-retention.md#classification-tiers). Roll-your-own crypto is prohibited; use vetted libraries and platform primitives.

## 12. Build pipeline and release integrity

The scope section names "a compromised pipeline corrupting outputs" as a worst case. These controls address it:

- Default branches of product repos are protected against force-push and history rewrite. Agents and unattended automation never push to them directly; humans may (per `GOVERNANCE.md`). Required status checks where CI exists.
- CI/CD tokens and deploy credentials are least-privilege and scoped per repo/environment. No org-wide tokens in pipelines.
- Third-party CI actions/plugins are pinned to a commit SHA or verified version, not a floating tag.
- Release artifacts are built only by the release pipeline from a clean checkout. No artifacts built on developer machines are shipped.
- The release pipeline records what it built (source SHA, dependency lockfile hash, SBOM) so any shipped artifact can be traced to its inputs.
- Accounts with write access to product repos or this standards repo use MFA (per [section 1](#1-authentication)); commits to this standards repo are signed where tooling allows.

## 13. Backup security

[`reliability-availability.md`](reliability-availability.md#7-backup-and-recovery) covers restore testing and recovery objectives. This section covers the ransomware case the scope names: backups must survive a compromise of production.

- Backups are encrypted at rest.
- Backup storage is access-separated from production: credentials that administer production systems cannot delete or overwrite backups.
- At least one backup copy is immutable (object-lock / WORM) or offline for the duration of the backup retention window.
- Backup access is logged and restricted to named accounts.

## 14. Vulnerability management

- Dependency CVEs: handled at release time per [section 6](#6-dependencies-and-supply-chain).
- Target resolution times once a vulnerability in Vervian's own shipped or deployed code is confirmed: **critical** within 7 days (out-of-band update if the product auto-updates), **high** within 30 days, **medium/low** into normal releases. These are targets; the clock starts at confirmation.
- Static analysis (SAST) with security rules enabled runs in CI for every product repo.
- Each product runs the standard security self-audit (`/vervian-security-audit` or equivalent) at least annually and before any major release; findings are tracked to closure.
- **Disclosure channel:** vulnerabilities can be reported to **jlist@vervian.com**, published in the product docs and, for web surfaces, in a `security.txt`. Reports get an acknowledgement within one business day.

---

## How we verify

| Requirement area | Evidence |
| :--- | :--- |
| Secret scanning | Pre-commit hook + CI job result |
| Dependency vulns | SBOM + scan report attached to each release |
| Signing | Build pipeline signs; release notes record cert thumbprint |
| Auth/authorization | Test suite covers default-deny + role boundaries; throttling covered by test or manual check per release |
| Access lifecycle | Current written list of credential-holding systems; 6-month review and 24h offboarding apply once there is staff |
| Network exposure | Listening ports/endpoints in the architecture doc match a scan of the deployed product, checked per release |
| TLS configuration | External scan run at least quarterly |
| Logging | Log sample reviewed quarterly for accidental PII / secrets |
| Incident response | One-page written incident plan per product, reviewed annually. The plan names at least one AI-specific scenario (prompt injection leading to data exposure, agent taking an unauthorised action) |
| Separation of duties | The compensating-control table above is reviewed annually and updated when headcount changes |
| AI / agent rules | Agent credential and tool grants reviewed alongside the access review |
| AI feature records | Feature record (data out, provider/model, authorised actions, classification) present in the architecture doc for every AI-backed feature |
| AI adversarial testing | Prompt-injection and jailbreak suite results attached to the release, per AI-facing feature |
| AI consumption limits | Token ceilings and rate limits configured; cost/volume alert exists and has been triggered at least once in test |
| Threat modelling | Written threat-model note attached to each new service, external feature, or AI capability |
| Application security | SAST in CI; annual `/vervian-security-audit` report per product |
| Pipeline integrity | Branch-protection settings + pinned-action check, reviewed annually |
| Backup security | Immutability/access-separation config reviewed annually with the restore exercise in [`reliability-availability.md`](reliability-availability.md#how-we-verify) |
| Vulnerability SLAs | Tracker timestamps from confirmation to fix, sampled annually |

## Exceptions

None by default. Product-specific exceptions live in `product-overrides/` with stated rationale and review date.

## Sources

- OWASP ASVS 4.x (the underlying control catalogue; levels set in [section 11](#11-application-security)).
- OWASP Top 10 (the design-time and application-security rules in sections [10](#10-ai-and-generated-code) and [11](#11-application-security)).
- OWASP Top 10 for LLM Applications, 2025 revision (all of [section 10c](#10c-ai-shipped-inside-a-product): prompt injection, improper output handling, supply chain, system prompt leakage, vector and embedding weaknesses, excessive agency, unbounded consumption).
- NIST SP 800-218 (SSDF) and SLSA (pipeline integrity rules in [section 12](#12-build-pipeline-and-release-integrity)).
- NCSC ransomware guidance (offline/immutable backup rule in [section 13](#13-backup-security)).
- GDPR Article 32 (security of processing).
- 2026-06-10 internal review of this standard (gaps: application security, pipeline and backup controls absent despite being named in scope; incident-response triage timing contradicted `privacy-gdpr.md`).
- 2026-08-14 review against a client AI-DLC cybersecurity policy (client-confidential, held outside this repo). Gaps it exposed and this revision closed: no design-time threat modelling; no record of what an AI feature sends where; no controls for system prompt leakage, retrieval/vector isolation, model pinning, or unbounded consumption; no adversarial testing gate for AI-facing features; separation of duties assumed rather than stated. See also `control-framework-mapping.md`.
- TBD: prior internal incidents that motivated specific rules. To be linked once the historical record is reviewed.

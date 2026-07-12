# Security

> Owner: Vervian · Last reviewed: 2026-06-10

## Scope

Applies to every Vervian product, every internal service, every build pipeline. Where a product override exists in `product-overrides/`, the override applies *in addition to* (never instead of) these rules unless explicitly noted.

Vervian products run in environments where a compromise is not merely a data risk but a risk to core user workflows and to irreversible actions (data destroyed by ransomware, a compromised pipeline corrupting outputs, a leaked customer data file). The framing throughout is "a compromise on the critical path is the worst case, so design for it."

A **critical path** here means any flow where failure causes data loss, breaks a core user flow, or triggers an irreversible action.

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

- Code suggested by AI assistants is reviewed by a human before merge. AI is not an approver.
- Secrets, customer data, and proprietary algorithms are not pasted into external AI tools that don't carry Vervian's enterprise data-handling terms.

AI **agents** (assistants with tool, file, or network access, in development or embedded in a product) are an attack surface, not just a data-egress risk:

- **(blocking)** Agents never hold or use production credentials. An agent that needs production data works through an audited, least-privilege, read-only path approved per case.
- Untrusted content an agent processes (customer tickets, uploaded files, web pages, third-party API responses) is treated as **data, not instructions**. Product features built on LLMs document their prompt-injection exposure and the mitigations (input isolation, tool allow-lists, output validation, human confirmation for irreversible actions).
- Tools and MCP servers granted to an agent follow least privilege: only the tools the task needs, write access only where the task writes.
- See `agent-guide.md` for AI assistant rules.

## 11. Application security

Baseline: **OWASP ASVS Level 1** for every product. A product that handles regulated data (health, payment, children's data) may be held to **Level 2** through its product override, where the extra effort is justified, not by default for everything. The rules below are the controls most often violated; ASVS is the full catalogue behind them.

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
| Incident response | One-page written incident plan per product, reviewed annually |
| AI / agent rules | Agent credential and tool grants reviewed alongside the access review; prompt-injection exposure documented per LLM-backed feature |
| Application security | SAST in CI; annual `/vervian-security-audit` report per product |
| Pipeline integrity | Branch-protection settings + pinned-action check, reviewed annually |
| Backup security | Immutability/access-separation config reviewed annually with the restore exercise in [`reliability-availability.md`](reliability-availability.md#how-we-verify) |
| Vulnerability SLAs | Tracker timestamps from confirmation to fix, sampled annually |

## Exceptions

None by default. Product-specific exceptions live in `product-overrides/` with stated rationale and review date.

## Sources

- OWASP ASVS 4.x (the underlying control catalogue; levels set in [section 11](#11-application-security)).
- OWASP Top 10 for LLM Applications (prompt injection and agent rules in [section 10](#10-ai-and-generated-code)).
- NIST SP 800-218 (SSDF) and SLSA (pipeline integrity rules in [section 12](#12-build-pipeline-and-release-integrity)).
- NCSC ransomware guidance (offline/immutable backup rule in [section 13](#13-backup-security)).
- GDPR Article 32 (security of processing).
- 2026-06-10 internal review of this standard (gaps: application security, pipeline and backup controls absent despite being named in scope; incident-response triage timing contradicted `privacy-gdpr.md`).
- TBD: prior internal incidents that motivated specific rules. To be linked once the historical record is reviewed.

# Data Classification & Retention

> Owner: Vervian · Last reviewed: 2026-06-05

## Scope

Every piece of data Vervian stores (in production systems, backups, logs, exports, support tools, developer machines) has a classification and a retention rule. This document defines the tiers and the defaults.

When the classification of a new data field isn't obvious, the PR description states the chosen tier and the rationale.

---

## Classification tiers

| Tier | Definition | Example data | Handling |
| :--- | :--- | :--- | :--- |
| **Public** | Information Vervian has chosen to publish or that has no confidentiality value. | Product marketing pages, published documentation, release notes. | No special handling. |
| **Internal** | Routine company information not intended for external release but with low individual harm if disclosed. | Internal wiki content, non-sensitive metrics, build-pipeline logs without secrets. | Access limited to Vervian staff. |
| **Confidential** | Information whose disclosure would harm Vervian, a customer, or an individual. | Customer lists, sales pipeline, source code, support tickets, customer-uploaded files, employee records. | Access on a need-to-know basis. Encryption in transit and at rest. Logged access for the most sensitive subsets. |
| **Restricted** | Information whose disclosure would cause serious harm: regulated, security-critical, or contractually protected. | Production secrets/keys, payment data, personal data of minors (if any), legal-hold material. | Strong access controls (MFA, audit, named approvers). Encryption with managed keys. Defined retention with documented deletion. |

Data inherits the highest tier of any field it combines. A dataset joining "Internal" telemetry with "Confidential" customer IDs becomes Confidential.

---

## Default retention rules

These are defaults. Specific datasets may have shorter retention required by privacy law, longer retention required by contract or legal hold, or specific retention specified in a product override.

| Data type | Default retention | Notes |
| :--- | :--- | :--- |
| Active customer account data | Life of the customer contract + 90 days | Then anonymise or delete unless legal hold |
| Customer payment / billing records | 7 years from transaction | Tax / accounting obligation |
| Customer support tickets | 3 years from ticket close | Personal data within tickets minimised per [`privacy-gdpr.md`](privacy-gdpr.md#10-logs-and-pii) |
| Customer-uploaded files (user content, crash dumps with customer data) | 90 days after last access, unless retained for active support | Customers can request earlier deletion |
| Application logs (no PII) | 90 days | Longer if needed for an active investigation |
| Security audit logs | 1 year minimum, 2 years for sensitive systems | Tamper-evident; see [`security.md`](security.md#8-logging-and-audit) |
| Error / crash telemetry | 90 days | Identifiers stripped or hashed where diagnostic purpose allows |
| Backups | 35 days (rolling) | Long enough for recovery, short enough to bound DSAR fulfilment |
| Employee records | Per employment law in the jurisdiction; typically 7 years post-departure | HR-owned |

---

## Exports

Any feature that exports data (to a customer, to a partner, to disk for support) must:

- Match or exceed the source data's classification (an export can be more restrictive, never less).
- Carry the classification label in the artifact where the format supports it.
- Be logged for Restricted data.
- Honour DSAR portability formats (machine-readable, structured) where the export is a DSAR response.

## Deletion

- **Soft delete** (flagging a record as deleted but keeping the bytes) does **not** satisfy GDPR erasure. Erasure means the record is unrecoverable through normal operations within the timeline.
- Backup-based recovery may temporarily resurrect deleted records. Backups must roll past within the documented retention window, and we do not selectively restore deleted records from backup without re-applying the deletion.
- **Hard deletion verified**: after an erasure request, a sample-based verification confirms the record no longer appears in production queries, exports, or accessible logs.

## Anonymisation

When data is retained for analytics or operational learning beyond the personal-data retention rule, it must be **genuinely anonymised**, not pseudonymised. Hashed IDs with the original keys still around are pseudonymised, not anonymised, and remain personal data.

When in doubt, treat as pseudonymised.

## Cross-references

- Lawful basis and DSAR mechanics: [`privacy-gdpr.md`](privacy-gdpr.md).
- Encryption and access control: [`security.md`](security.md).
- Customer contractual overrides: `legal-compliance.md`.

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Every new field has a classification | PR description records it |
| Retention runs actually delete | Quarterly verification job; results recorded |
| DSAR erasure works end-to-end | Annual rehearsal across primary systems and backups |
| Exports honour classification | Spot-check on customer-facing export features |

## Exceptions

- Legal hold: deletion is suspended for data under hold, recorded centrally.
- Active incident investigation: relevant logs/records are preserved until the investigation closes.
- Product-specific exceptions live in `product-overrides/`.

# Privacy & GDPR

> Owner: Vervian · Last reviewed: 2026-06-05

## Scope

Applies to every Vervian product and internal system that processes personal data, regardless of the data subject's jurisdiction. The strictest applicable regime (typically GDPR) sets the bar. We do not maintain separate code paths per jurisdiction unless commercially required.

See `legal-compliance.md` for the regulatory landscape.

---

## 1. What counts as personal data

Anything that identifies, or can identify, a natural person, directly or indirectly. For Vervian this includes (non-exhaustive):

- Customer account fields: name, email, phone, company, billing address.
- End-user identifiers in product use: user IDs, IP addresses, device IDs, support ticket content.
- User-generated content and uploaded files where they contain names or other identifiers.
- Telemetry / crash reports if they include identifiers or free-text user input.

When in doubt, treat as personal data.

## 2. Principles (data minimisation, etc.)

Vervian applies all GDPR Article 5 principles. The operational implications:

- **Minimisation:** collect the least personal data needed to deliver the feature. New collection requires a justified entry in the PR description.
- **Purpose limitation:** data collected for purpose A is not used for purpose B without renewed lawful basis.
- **Storage limitation:** every personal-data field has a retention rule (see [`data-classification-retention.md`](data-classification-retention.md)).
- **Accuracy:** customers can correct their own data; product features must support this.
- **Integrity and confidentiality:** see [`security.md`](security.md).
- **Accountability:** record-keeping in the Article 30 processing register (location: TBD).

## 3. Lawful basis

Every processing activity must record a lawful basis. The common bases for Vervian:

| Activity | Likely basis |
| :--- | :--- |
| Account creation, license entitlement, support | Contract |
| Marketing emails | Consent (opt-in, separately captured) |
| Security logging, abuse prevention | Legitimate interest (balancing test on file) |
| Crash telemetry tied to a user | Legitimate interest, with opt-out |

Adding a new processing activity without a recorded lawful basis is a blocking PR finding.

## 4. Data Subject Rights (DSARs)

Vervian honours the GDPR Article 15-22 rights for any data subject regardless of jurisdiction. Operational rules:

| Right | Timeline | How it's delivered |
| :--- | :--- | :--- |
| Access (Art. 15) | 30 days from verified request | Machine-readable export covering all Vervian systems holding the subject's data |
| Rectification (Art. 16) | 30 days | In-product where possible; support flow otherwise |
| Erasure (Art. 17) | 30 days | Hard-delete across primary stores; queued anonymisation in derived data; documented exceptions where retention is legally required |
| Restriction (Art. 18) | 30 days | Flag in the customer record; processing systems honour the flag |
| Portability (Art. 20) | 30 days | Same export as Access, in a structured machine-readable format |
| Objection (Art. 21) | 30 days | Cease processing for the objected purpose unless overriding legitimate ground |
| Automated decisions (Art. 22) | n/a. Vervian does not currently take solely-automated decisions affecting individuals | Flag any new feature that would change this |

Identity verification before fulfilling a DSAR is mandatory and documented per request.

## 5. Third-party processors

A processor is anyone who handles personal data on Vervian's behalf (cloud hosting, error reporting, analytics, AI providers, payment processors, support tools).

- **(blocking)** Every processor is in Vervian's processor registry (location: TBD) **before** customer data flows.
- **(blocking)** DPA (Data Processing Agreement) in place. Standard Contractual Clauses where data crosses borders out of the EEA.
- Sub-processors disclosed transparently to customers; customers given notice of changes per their contract.
- Adding a new processor mid-development requires sign-off from the privacy owner (Vervian).

## 6. International transfers

Where personal data leaves the EEA, Vervian relies on an Article 46 transfer mechanism (SCCs in most cases, adequacy decisions where they apply). Engineering practice:

- Hosting-region selection considered for any new service: prefer EU/UK regions when serving EU/UK customers.
- Document the transfer chain in the processor registry.
- Transfer Impact Assessments (TIAs) recorded for high-risk transfers. Coordinate with the privacy owner.

## 7. Breach notification

A personal-data breach starts a **72-hour clock** for notification to the relevant supervisory authority under GDPR.

Operational flow:

1. **T+0:** any engineer or staff member who suspects a personal-data breach informs the security contact (channel: TBD) immediately.
2. **T+1 hour:** security and privacy triage scope and confirm whether personal data is involved.
3. **T+24 hours:** scope of affected data subjects estimated; containment underway.
4. **T+72 hours:** notification to supervisory authority if the threshold is met (risk to data subjects).
5. **Without undue delay:** notification to affected data subjects where the breach is likely to result in a high risk to their rights and freedoms.

The full incident-response playbook lives alongside the security incident workflow. See [`security.md`](security.md#9-incident-response).

## 8. DPIA (Data Protection Impact Assessment)

A DPIA is required when a new processing activity is likely to result in a high risk to data subjects' rights. The privacy owner must be involved before scope is finalised for any such activity.

DPIAs are required for: large-scale processing of special categories, systematic monitoring of individuals or public areas, new technology where the privacy impact isn't well-understood.

## 9. Telemetry, error reporting, AI provider integrations

When sending data outside Vervian's systems for these purposes:

- Strip or hash identifiers wherever possible.
- Customer opt-out available where the basis is not strict-necessity.
- Document what's sent, to whom, and under what basis in the user-facing privacy notice.
- AI provider integrations: only providers with enterprise data-handling terms (no training on Vervian data). See [`security.md`](security.md#10-ai-and-generated-code).

## 10. Logs and PII

Logs are not a loophole for retention. Personal data in logs counts as personal data:

- Logs do not contain raw PII beyond what's strictly necessary for diagnostics.
- IP addresses in logs are truncated or hashed where the diagnostic purpose allows.
- Log retention rules match the data classification (see [`data-classification-retention.md`](data-classification-retention.md)).

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Lawful basis recorded | Article 30 register entry per processing activity |
| Processor registry | Registry up to date as of latest release |
| DSAR timeliness | Sampled audit of DSAR ticket turnaround |
| Breach readiness | Written breach plan reviewed annually (see [`security.md`](security.md#9-incident-response)); formal tabletop once there is a team |
| Telemetry minimisation | Reviewed quarterly by the privacy owner |

## Exceptions

Customer-specific contractual provisions may impose stricter requirements (see `legal-compliance.md`). They are followed for that customer's deployment.

## Sources

- GDPR (Regulation (EU) 2016/679), particularly Articles 5, 6, 13-22, 30, 32, 33, 34, 35, 46.
- UK GDPR.
- ICO and CNIL guidance.
- PIPEDA (Canada).

# Reliability & Availability

> Owner: Vervian · Last reviewed: 2026-06-05

## Scope

Failure behaviour, offline operation, recovery, and update mechanics across Vervian products. The unifying frame: failure on a critical path is qualitatively different from a failure anywhere else. When a failure can cause data loss, break a core user flow, or trigger an irreversible action, it warrants stricter rules than a cosmetic or recoverable failure.

---

## 1. The critical-path principle

A path is **critical** if its failure would cause data loss, break a core user flow, or trigger an irreversible action the user cannot easily undo. Critical paths include (non-exhaustive):

- Saving, syncing, or exporting user data.
- Authentication, entitlement, and payment flows.
- Any action that mutates external state irreversibly (sending, publishing, deleting).

Critical paths follow stricter rules than the rest of the product:

- **No new network dependency** without explicit design review.
- **No silent failure.** A critical-path failure must be visible to the user immediately.
- **No automatic updates mid-action.** Update mechanisms must respect a "do not disturb" mode where the product's context demands it.
- **Tested under failure conditions** (network down, third-party gone, disk full) before each release.

## 2. Connectivity model

Different Vervian products operate in fundamentally different connectivity contexts, and the offline-availability bar differs accordingly. Some products are web/SaaS and online-only by design, some are desktop or local-first, and some need to keep working without a network. **Every product must declare its connectivity model in its product override**, choosing from:

- **Offline-capable**: the product is expected to operate fully without a network. Core paths work offline; network-only features are exceptional and named. Local-first and desktop products typically fall here.
- **Online-first, offline-available**: the product is designed primarily for connected operation (cloud-backed compute, AI-assisted features, central data) but exposes a documented subset of capabilities when offline.
- **Online-only**: the product does not function offline. Acceptable for web/SaaS products with no offline requirement, and stated explicitly.

Whichever model applies, these rules hold:

- **Authentication and licensing tolerate intermittent connectivity** where the model is not strictly online-only. Entitlements remain valid for a documented duration without phoning home.
- **Failure modes when offline are explicit, not silent.** A feature unavailable due to lack of network displays a clear "unavailable offline" state with guidance, never a hang, never a silent failure.
- **Graceful degradation, not blocking.** A product's offline subset must remain usable on its own terms; lost cloud features do not block the rest of the product.

Each product declares its model in its product-overrides file (see `product-overrides/`).

## 3. Graceful degradation

When a non-critical dependency fails (third-party service, optional integration, telemetry endpoint), the product:

- Continues to operate on its core paths.
- Surfaces a non-blocking indicator that the dependency is unavailable.
- Does not retry indefinitely in a way that consumes resources or generates noise.
- Recovers automatically when the dependency returns.

When a critical dependency fails (the path is critical per § 1), the failure is loud, immediate, and includes guidance for the user.

## 4. Crash recovery and autosave

For products that hold user state:

- Autosave at a cadence appropriate to the product (sub-minute for active editing surfaces).
- Crash recovery on next launch: offered, not silent. The user decides whether to restore.
- Backup copies of recent state retained for at least the last N saves (N defined per product, default 5).
- No data loss from a clean process termination. From a hard kill, loss is bounded by the autosave cadence.

## 5. Update mechanism

- Updates are **opt-in or scheduled**, never silent during use.
- A "do not disturb" flag is honoured by the update mechanism where the product's context demands it. While set, no update prompts, no background downloads that compete for resources.
- Update rollback path documented and tested. A broken update can be undone without manual file surgery.
- Update artifacts signed and verified (see [`security.md`](security.md#4-code-signing-and-installers)).
- Pre-release builds clearly labelled and never auto-promoted to general-availability channels.

## 6. SaaS availability (where applicable)

For Vervian-operated services, availability is a tracked commitment, not just an internal concern (see `vision.md`). Regressions are visible, not buried.

| Metric | Target |
| :--- | :--- |
| Uptime SLO | TBD per service; refine per service criticality |
| Planned-maintenance notification | TBD; advance notice for customer-facing systems |
| Recovery objectives (RTO/RPO) per service | Stated in the service architecture doc |
| Incident transparency | Status page kept current during incidents; post-mortem within a defined window |

## 7. Backup and recovery

- Production data backed up per its classification and operational criticality (see [`data-classification-retention.md`](data-classification-retention.md)).
- Restore procedures **tested at least annually**. An untested backup is not a backup.
- Backup integrity verified (checksums, sample restores).
- Recovery objectives documented per service: RPO (how much data can we lose) and RTO (how long to restore).
- Backups must survive a compromise of production (encryption, access separation, immutability): see [`security.md`](security.md#13-backup-security).

## 8. Failure modes: what's acceptable

Some failures are acceptable; others aren't. The product owner writes down which is which, per product.

| Failure | Acceptable? | Required behaviour |
| :--- | :--- | :--- |
| Telemetry endpoint unreachable | Yes | Buffer locally, retry, never block UI |
| AI / cloud-assist feature unreachable | Yes | Disable the feature with a notice; rest of product unaffected |
| License / entitlement server unreachable | Yes, up to documented offline grace period | Continue operation; warn when grace nears expiry |
| Connection lost mid-action on a critical path | **Acceptable only if** the user is alerted immediately and the system holds state or fails loud as designed | See product override |
| Silent data corruption | **Never** | Loud, visible, recoverable |

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Offline operation | Smoke suite run against a network-disabled deployment per release (for offline-capable / offline-available products) |
| Crash recovery | Automated test that kills the process and confirms recovery on next launch |
| Update rollback | Pre-release rollback drill |
| Backup restore | Annual restore exercise; results recorded |
| Failure-mode matrix | Documented per product; reviewed on incident |

## Exceptions

Per-product overrides live in `product-overrides/`.

## Sources

- Past incidents (TBD: link to incident archive once consolidated).
- "Site Reliability Engineering" (Google) for the SLI/SLO/SLA framing.

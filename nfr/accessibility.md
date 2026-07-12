# Accessibility

> Owner: Vervian · Last reviewed: 2026-06-05

## Scope

The accessibility baseline for every Vervian product surface a user can interact with: web admin consoles, marketing properties, in-product dialogs and configuration panels, support flows. Applies across web, SaaS, and desktop surfaces.

Where a strict WCAG criterion is genuinely incompatible with the demands of a specialised surface, the product team **must** record the carve-out in their product override, with rationale and any compensating measure.

Regulatory drivers (AODA, European Accessibility Act, ADA Title III) are summarised in `legal-compliance.md`.

---

## 1. Baseline

**WCAG 2.2, Level AA**, across all in-scope surfaces. The most operationally relevant criteria:

| Area | Requirement |
| :--- | :--- |
| **Keyboard** | Every interactive control reachable and operable via keyboard. No keyboard traps. Logical tab order. |
| **Focus visibility** | Focus state always visible. Sufficient contrast on the focus indicator. |
| **Focus management** | Dialogs trap focus while open and return focus to the trigger when dismissed. |
| **Contrast** | Text at least 4.5:1 against its background (at least 3:1 for large text). UI components and graphical objects at least 3:1 against adjacent colours. |
| **Colour as carrier** | Colour is never the only means of conveying information, indicating state, or distinguishing elements. |
| **Target size** | Interactive targets at least 24x24 CSS px (WCAG 2.2 SC 2.5.8). Exceptions for inline targets in flowing text. |
| **Forms** | Labels present, errors identified in text, programmatically associated. |
| **Motion** | No content that flashes more than 3 times per second. Animations respect `prefers-reduced-motion`. |
| **Text alternatives** | Meaningful images have alt text; decorative images marked as such. |
| **Headings & structure** | Logical heading hierarchy; landmarks used so assistive tech can navigate. |
| **Language** | Page/document language set programmatically. |

## 2. Screen-reader support

Web/admin surfaces are tested with at least one screen reader per major platform:

- **Windows:** NVDA (primary), JAWS where customers require it.
- **macOS:** VoiceOver.
- **iOS / Android:** VoiceOver / TalkBack, where the product has mobile surfaces.

Test coverage focuses on: form completion, navigation between landmarks, dialog open/close, status announcements after async actions, error recovery.

## 3. Specialised surfaces

Some product surfaces (data-dense canvases, real-time visualisations, low-latency tooling) are not general-purpose document/form UIs. WCAG written for typical document/form interfaces does not always map cleanly to them.

**The carve-out rule:**

1. Even on specialised surfaces, the baseline still applies to **anything that isn't the specialised core**: menus, file dialogs, settings panels, configuration screens, error toasts, login flows. Those follow WCAG 2.2 AA without negotiation.
2. The specialised core may need to deviate. When it does, the product override records:
   - Which criterion is being carved out.
   - Why the standard approach is incompatible with the workflow.
   - The **compensating measure** (alternative keyboard path, alternative output, documentation for assistive tech users, etc.).
3. A carve-out is never "we don't have time." It's a documented design decision with a review date.

## 4. Native platform accessibility

Where a desktop product runs in an installed environment and the platform exposes accessibility APIs (UI Automation on Windows, AX on macOS), product surfaces use the native idioms, not custom drawing for things that have a native control equivalent.

## 5. Marketing and customer-facing web surfaces

Vervian websites, the customer portal, support documentation: WCAG 2.2 AA without carve-outs. These surfaces are first-impression contact and any regression here is a regulatory exposure under AODA and EAA.

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Automated checks | axe-core (or equivalent) integrated into the build for in-scope web surfaces |
| Keyboard / focus | Manual sweep on the documented critical journeys, per release |
| Screen reader | Targeted manual testing on a representative subset, per release |
| Contrast / colour | Design-token audit + automated check |
| Reduced motion | Spot-check at release |

Each in-scope product names its critical journeys (login, primary task completion, settings, error recovery, sign-out). These are the journeys that get the manual sweep.

## Exceptions

Carve-outs for specialised surfaces are recorded in `product-overrides/` per product. **Marketing surfaces have no carve-outs.**

## Sources

- WCAG 2.2 Level AA (W3C).
- AODA (Accessibility for Ontarians with Disabilities Act).
- European Accessibility Act (EU) 2019/882.
- ADA Title III (US, applied to commercial websites in current case law).

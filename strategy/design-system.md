# Design System

> Owner: Vervian · Last reviewed: 2026-06-05
>
> **Status:** A baseline pointer. This document holds the shared design-system principles. Per-product design systems implement them. Canonical URLs are TBD.

---

## What the design system is

A shared visual and interaction language that product teams build on, so Vervian products feel consistent without each one reinventing primitives. The intent:

- **Reuse first.** A single set of primitive components and tokens, consumed rather than re-implemented per product.
- **Tokens, not hard-coded values.** Colours, spacing, radii, typography, and shadows live as named tokens.
- **Accessibility wired in.** Primitives ship with focus states, ARIA attributes, and keyboard support already handled.

This is a standards baseline, not a single implementation. Each product may maintain its own design-system app or package; this document is the shared frame they conform to. Implementation details (framework, component library, tooling) are decided per product and are TBD at the baseline level.

---

## Where it lives

The shared baseline is this document. Concrete implementations (token files, primitive components, product-specific component sets) live in the relevant product repo or a shared package once one exists.

- **Canonical home (target):** a published design-system reference. URL TBD. Once it exists, consumers (engineers, designers, AI assistants) follow the live reference, not a file path.
- **Structure** (the durable shape): tokens in a single token layer; primitive components in one place; product-specific component sets layered on top. Expansion lands on this scaffold.
- **Owner-maintained:** Vervian. Significant changes to tokens or shared primitives go through the design-system owner.

---

## The rules

These apply to every Vervian product touching UI:

1. **Use the system. Don't build parallel components.** If a primitive exists, that is the version used. Forks live inside the design-system project, not in product code.
2. **Tokens, not hard-coded values.** Colours, spacing, radii, typography, shadows reference tokens. Hard-coded hex values, raw pixel measurements, and ad-hoc font sizes are defects.
3. **Variants over one-offs.** When a new visual treatment is needed, it lands as a variant on the existing primitive, not a new component three folders over.
4. **Brand voice in UI copy** follows the rules in [`product-management.md`](product-management.md#speak-the-customers-language). The system enforces typography; the team enforces voice.
5. **Accessibility is part of the system, not an afterthought.** Components ship with focus states, ARIA attributes, and keyboard support already wired. WCAG 2.2 AA per [`accessibility.md`](../nfr/accessibility.md). Specialised surfaces are still subject to the carve-out rules in `README.md`, not the design-system primitives.

---

## How product repos consume it

The consumption model is **TBD** and decided per product. Options on the table include a versioned shared package that product repos pin to and bump explicitly, or a copy-into-consumer pattern. Whichever is chosen, the rules above (reuse-first, tokens, variants, accessibility) hold regardless of mechanism.

Where products maintain their own design systems, those implement this baseline rather than diverge from it.

---

## Cross-references

- [`accessibility.md`](../nfr/accessibility.md): accessibility requirements the system implements.
- [`product-management.md`](product-management.md#speak-the-customers-language): UI copy / language conventions.
- `vision.md`: the strategic frame.
- `core-beliefs.md`: consistency and quality applied most visibly through this system.

---

## Open items

- [ ] **Publish a canonical reference** for the shared baseline and replace the "URL TBD" placeholders above.
- [ ] Decide the consumption model for product repos (shared package, copy-via-CLI, or per-product systems conforming to this baseline). Document the chosen pattern.
- [ ] Define the change flow for tokens vs primitives vs product-specific components. Token changes have broader blast radius than a single product-only component.
- [ ] Cross-check each implementing design system against the WCAG 2.2 AA bar in [`accessibility.md`](../nfr/accessibility.md) and record the audit.
- [ ] Document the brand / voice / tone layer that sits above the implementation (logos, brand colours, marketing-vs-product copy distinctions). Currently absent.

# Performance

> Owner: Vervian · Last reviewed: 2026-06-05

## Scope

Latency, throughput, and resource budgets for Vervian products and supporting services. Performance is anchored in concrete realities, not abstractions:

1. **User-facing interaction responsiveness** is the difference between a tool that gets adopted and one that gets abandoned. This applies to any interactive product.
2. **Backend and API responsiveness** matters wherever a product depends on server-side or SaaS surfaces.
3. **Developer iteration speed** matters for the team. A 10-minute build is a tax on every change, every day.

Numerical targets below are **baseline defaults**. Products with tighter or looser requirements specify them in `product-overrides/`.

---

## 1. User-facing interaction latency (interactive products)

| Operation class | Target | Notes |
| :--- | :--- | :--- |
| Direct input feedback (cursor, scroll, drag) | < 16 ms (60 fps) typical, < 33 ms (30 fps) worst | Anything visibly choppy is a regression |
| Discrete UI command (button click, menu select, edit) | < 100 ms p95 | If it can't be done in 100 ms, show progress within 100 ms |
| File / record open / load | < 2 s p95 for typical cases; loading indicator for larger | Define "typical" per product |
| Application startup | < 5 s p95 to interactive | Cold start on representative hardware |

These are visible-to-the-user budgets. They are measured on **representative customer hardware**, not developer machines.

## 2. Throughput and resource use

- CPU and memory budgets on shipped products are stated in the product's architecture doc. Regressions of > 10 % on the hot path are flagged in the PR.
- Network bandwidth: products must run usefully on their declared connectivity model. See [`reliability-availability.md`](reliability-availability.md#2-connectivity-model). Bandwidth-heavy features must respect that model (degrade or defer where the model is offline-capable; degrade visibly when the model is online-first).
- Install / bundle footprint: documented per product; meaningful increases are noted in the PR.

## 3. Server-side / SaaS latency (where applicable)

For any web/SaaS surface Vervian operates:

| Metric | Target |
| :--- | :--- |
| API p95 latency | < 500 ms |
| API p99 latency | < 2 s |
| Time-to-first-byte (web) | < 800 ms p95 |
| Core Web Vitals (LCP / INP / CLS) | "Good" thresholds per the current Web Vitals definition |

## 4. Developer iteration

Developer productivity is a non-functional requirement.

| Metric | Target |
| :--- | :--- |
| Local incremental build / hot reload | < 5 s for the change-feedback loop on representative repos |
| Full CI build (PR check) | < 15 min p95 |
| Test suite (unit + most integration) | < 10 min locally |

When a CI run grows past these thresholds, treat it as a regression to investigate, not a fact of life.

## 5. Measurement and evidence

Each product is responsible for:

- **Defining the hot paths** it cares about (roughly 5 to 10 user journeys per product).
- **Benchmarking** them on representative hardware on a defined cadence (at least each release).
- **Recording** the benchmark results so regressions are visible across releases, not just in the latest run.

Specific tools are per-product (load-test framework, profiler, Web Vitals collector). The standard is: a PR reviewer can ask "show me the benchmark for this change" and get a real answer.

---

## How we verify

| Requirement | Evidence |
| :--- | :--- |
| Interaction budgets | Profiler runs against the documented hot paths; recorded per release |
| Server latency | Synthetic monitoring + production p95/p99 dashboards |
| CI duration | Trended in the CI dashboard; flagged on regression |

## Exceptions

Product-specific tighter targets and looser ones (for example one-time imports that aren't on a hot path) live in `product-overrides/`.

## Sources

- Nielsen Norman 100ms / 1s / 10s response-time guidelines.
- Web Vitals (web.dev).
- TBD: product-specific latency targets recorded as commitments are made.

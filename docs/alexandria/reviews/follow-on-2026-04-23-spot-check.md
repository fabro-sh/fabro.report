# Follow-On Batch Spot-Check — 2026-04-23

**Reviewer:** Conan
**Date:** 2026-04-23
**Scope:** Phases 1 (`c66ddba`) + 2 (`021a4a7`). Six-part verification. Final gate before implementation-readiness.

**Files reviewed:**
- 6 new cards: A5, A7, A8, A12, A13, A14
- 14 modified cards (CLI rename cascade + backlink updates)
- 4 source files (`storage-and-keys.md`, `cli-interaction-patterns.md`, `accessibility.md`, `tech-stack-and-deployment.md`)
- Manifest (19 cards, 2 blocked on visual direction)

---

## Part 1 — CLI Rename Cascade Correctness

**Verdict: PASS_WITH_NITS**

Every `fabro debug` reference as a CLI command is eliminated. Every `fabro report` as a CLI command is eliminated. Surviving hits fall in three legitimate categories:

| Hit Location | Type | Legitimacy |
|---|---|---|
| `fabro.report` (domain) throughout all cards | Domain | Kept — it's the domain, not the command. |
| `fabro dump` (producer command) in F2 / A6 / A7 | New canonical | Correct post-rename. |
| F2 "Canonical CLI Commands" subsection — "pre-2026-04-23 names `fabro debug` ... and `fabro report` ... are **superseded**" | Explicit supersession note | Correct — this is where the old names should live. |
| Source files (`product-vision.md` at L14, L25, L55, L58) | Pre-rename source | Not a card — source files are frozen, correct to leave. |

**F2 Canonical CLI Commands subsection (new):** Table-based, with the existing "Usage Rules" numbered list following. Lives in the HOW section (lines 55–65) alongside the other authoritative tables, using the same voice. Includes explicit supersession note. No new vocabulary invented. Clean.

**Command-choice semantics spot-check (sampled usage):**

| Card | Usage | Assessment |
|---|---|---|
| F1 Validation Criteria | "A Reporter can run `fabro dump --upload`, receive a URL, and share it." | Correct — primary path. |
| F4 HOW-section "Success output" | "A successful `fabro dump --upload` (or `fabro upload`)" | Correct — both paths named. |
| A1 Example 1 | "`fabro dump --upload -m 'cargo install hangs on CI'`" | Correct. |
| A3 Lifecycle 1 | "`fabro dump --upload` (or `fabro upload` against a previously-dumped tarball)" | Correct. |
| F5 HOW | "The `fabro dump --upload` (and standalone `fabro upload`) CLI..." | Correct. |
| F6 Paired with F3 clause | "dump-and-upload" shorthand | Correct — covers both. |

---

## Part 2 — A5 / A7 / A8 Quality

**Verdict: PASS**

| Card | Grade |
|---|---|
| A5 `System - Retention, Storage, and Key Scheme` | A |
| A7 `Standard - Interaction Patterns (CLI + Delete)` | A |
| A8 `Standard - Accessibility Policy` | A- |

### A5 — Source fidelity: faithful

All numerics present (`us-east-1`, `fabro-report-bundles`, 32 hex chars, 128 bits, 30 days, `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` + `<id>/metadata.json`). Load-bearing S3 lifecycle mechanism, per-Report directory keys, and 128-bit entropy all present. Cross-references (A9, A10, A11, F2) specific to mechanism-vs-policy distinction. No inventions beyond source.

### A7 — Source fidelity: faithful

Presigned URLs, 50MB cap, `NO_COLOR`-awareness, client-side delete confirmation all present with source-consistent rationale. Full four-step presigned-URL flow in HOW/Upload Architecture. Byte-path property named explicitly. Belt-and-suspenders 50MB cap (pre-flight + S3-side). Cross-references (F4, F2, A1, A4, A9, A10, A8) all present.

### A8 — Source fidelity: faithful

WCAG 2.2 AA, `NO_COLOR`, keyboard-first, screen-reader HTML, `/ui` skill workflow hint all present. Two-surface framing explicitly named load-bearing. Delete-confirmation modality specified. Cross-references (F4, A4, A7, C2) present; C2 correctly flagged as blocked.

**Minor nit (A8-N1):** One mild expansion — naming `--plain` anti-example. Source open-questions flag asks about `--plain` as open question (L44); A8 resolves it with a narrow rule. Small forward call, consistent with F4 and the register, defensible. Not fix-worthy.

---

## Part 3 — A12 / A13 / A14 Quality

**Verdict: PASS**

| Card | Grade |
|---|---|
| A12 `Decision - Service Stack` | A |
| A13 `Decision - Sentry for Service Errors, Not Bundle Processing` | A |
| A14 `System - Service Topology` | A- |

### A12 — Rejected Alternatives Credibility

| Option | Rejection Rationale | Credibility |
|---|---|---|
| A: Bun / TypeScript on Fly.io | (1) Borrowing bun.report's stack re-imports drift at implementation layer; (2) Fly doesn't simplify S3 access. | Real — two independent specific grounds. |
| B: Rust on Fly.io | IAM role from Fargate to S3 vs. long-lived AWS keys in Fly secrets or cross-cloud federation. | Strongest — concrete, specific, named. |
| C: Cloudflare Workers + R2 | (1) Diverges from team's AWS tooling; (2) Terraform support for Workers weaker than for Fargate. | Real — specific operational arguments. |
| D: Rust + Fargate (chosen) | Dominant S3 path simplicity; language alignment with Fabro; Terraform maturity. | Coherent. |

All rejections specific. No strawman.

### A13 — Tension Resolution

Explicitly distinguishes Sentry-for-service-errors from Sentry-as-bundle-processor. Does NOT contradict F3 — WHERE section states "The drift rejection here stays unchanged in substance; nothing about bundle-contents forwarding is softened." "No error tracking at all" framed as real alternative (Option A) and rejected with specific reason: "a 30-day-retention product gives a narrow window to catch operational bugs before they matter." Clean tension resolution.

### A14 — Distinctness Audit

| Comparison | Finding |
|---|---|
| A14 vs A12 | **Distinct.** A12 = "why these pieces, not others" (comparative rationale, 4 options considered). A14 = "what the pieces are and how they connect" (operational topology). A14's HOW section has Request Flow / Secret-injection Flow / IAM Flow — none belong in a Decision card. Not collapse-worthy. |
| A14 vs A5 | **Clean pairing.** A5 owns bucket layout, keys, URLs, lifecycle, sidecar metadata. A14 owns Fargate task, Route53, IAM role, Secrets Manager, Terraform repo. Each has explicit "not restated" language. Neither redundant. |
| Territory coverage | Complete — every topology piece covered. |

**Minor nit (A14-N1):** Properties-This-Topology-Guarantees section and Anti-Examples section have overlapping "what it is not" content in two shapes. Not padded exactly, but repetition noticeable on careful read. Not fix-worthy.

---

## Part 4 — F3 Inline Sentry Clarifying Note

**Verdict: PASS**

The note appears in F3's Drift Examples list under the existing Sentry entry.

| Check | Finding |
|---|---|
| Note weakens F3's drift list? | No. Drift rejection stands as first-class item. Note explicitly states "The drift rejection here stays unchanged in substance; nothing about bundle-contents forwarding is softened." |
| Note points at A13? | Yes. Wikilink to A13 at end of clarifying paragraph. |
| Load-bearing phrase used | "Sentry-as-downstream-bundle-processor" — matches A13's own phrasing. |
| Structural placement | Sits inside the existing HOW → Drift Examples bulleted list, scoped to its entry. No new top-level section; no re-ordering. |
| F3 WHERE section references A13? | Yes, with specific anchor ("the paired rejection... together they close both escape hatches"). |

Excellent containment.

---

## Part 5 — F1 Stale-Reference Fix

**Verdict: PASS**

F1 now references `[[Principle - Narrow Forever]]` as a proper wikilink in the "Strategy and scope commitment" subsection of WHERE (line 19). No plain-text "(manifest F6, not yet drafted)" fossil remaining.

**Full-library audit for remaining stale forward references:** All remaining plain-text references (A7/A8 `Standard - CLI Copy`, A8/A4/A6 `Standard - Design System` C2, F4 `Standard - CLI Copy`) resolve to cards that do not exist (C2 blocked, `Standard - CLI Copy` not in manifest). Correctly kept as plain-text.

---

## Part 6 — Backlink Audit (29 Edits)

**Verdict: PASS_WITH_NITS**

### F6 "Referenced by" entries

All six F6 backlinks have specific, non-generic anchors (scope-at-storage-layer, scope-at-command-surface, scope-at-accessibility, stack-as-permanent-shape, observability-expansion distinction, topology-at-infrastructure-layer). No "this card is referenced by F6" phrasing.

### F3 "Referenced by" entries

All three F3 backlinks tied to specific drift shapes (stack-layer drift, Sentry clarification, topology-layer drift).

### A5's "Implemented by" entries from A9, A10, A11

All three reverse-anchors substantive. A9's unguessability requirement ↔ A5's 128-bit entropy mechanism. A10's no-dedup ↔ A5's random-ID choice. A11's 30-day retention ↔ A5's Terraform-declared S3 lifecycle. Sam not inventing framing.

### A12 / A13 Referenced-by Claims vs. Upstream Anchors

Forward/reverse match on F5/A12 (Strategy's payoff is an operational architecture that makes team retrieval reliable) and F5/A13 (Service-side reliability is load-bearing for retrieval).

**Nit (Backlink-N1):** F4's "Referenced by" entry for A13 is slightly richer than A13's forward-link to F4 (which goes via A7). Framing is true but one spot where reverse-link prose is a half-step ahead of forward-link prose. No fix needed.

---

## Additional Checks

| Check | Finding |
|---|---|
| `Standard - CLI Copy` speculative reference in A7 and A8 | Plain-text in both cards, explicitly marked "not in manifest; referenced as the future home for exact strings." Correct — not wikilinks. |
| First `systems/` subfolder (A5 + A14) | Clean. Two System cards of clearly distinct scope. Pattern parallels `decisions/` and `standards/`. No structural concern. |
| Word-count trajectory | All new cards exceed 700-word soft cap (accepted per manifest). Sampled dense sections — each is load-bearing. A14's "Properties This Topology Guarantees" is closest to repetition; still earns its lines. |
| Source citation accuracy (2 per new card) | All 12 sampled citations verified accurate. |

---

## Cascade Completeness Verification

Sampled CLI rename propagation through all 14 Phase-1-touched cards: F1, F2, F3, F4, F5, F6, A1, A2, A3, A4, A6, A9, A10, A11, C1, C4 — old verbs absent or only in supersession notes; new verbs correct everywhere. Clean cascade.

---

## Summary Table

| Part | Verdict |
|---|---|
| 1. CLI Rename Cascade | PASS_WITH_NITS (no action needed) |
| 2. A5 / A7 / A8 Quality | PASS |
| 3. A12 / A13 / A14 Quality | PASS |
| 4. F3 Inline Sentry Note | PASS |
| 5. F1 Stale-Reference Fix | PASS |
| 6. Backlink Audit (29 edits) | PASS_WITH_NITS (one half-step prose mismatch, no fix needed) |

---

## Overall Verdict

**APPROVED.**

The 2026-04-23 follow-on batch is implementation-ready. The CLI rename cascade landed cleanly across 14 cards; 6 new Amplifier cards (A5, A7, A8, A12, A13, A14) are source-faithful, load-bearing, distinct, and correctly cross-referenced; F3's Sentry clarifying note closes the drift-vs-operational-use escape hatch without weakening F3; F1's stale reference is resolved; and 29 backlink edits carry substantive anchors rather than generic pairings.

**The library is now ready for Factory-mode implementation work on `fabro.report` the service.**

Implementation can begin on:
- **Infrastructure:** Terraform-declared Fargate + Route53 + IAM role + Secrets Manager / SSM + S3 bucket (A5 + A12 + A14 are sufficient spec)
- **CLI:** `fabro dump` / `fabro dump --upload` / `fabro upload` with presigned-URL upload path (A7 is sufficient spec)
- **Service (Rust):** Upload coordination endpoint, metadata-page render endpoint, delete endpoint (A6 + A7 + A14 sufficient spec; Sentry wiring per A13)
- **Report Metadata Page HTML structure:** A4 + A8 sufficient for structure + accessibility. Visual direction remains deferred.

## Deferred / Permanently-Blocked Cards

| Card | Status | Implication |
|---|---|---|
| C2 `Standard - Design System (Receipt Page + CLI Output)` | BLOCKED on visual direction | Blocks visual implementation; does NOT block structural scaffolding (A4 + A8 carry the structural contract). Can be built as unstyled / minimally-styled HTML first. |
| C3 `Standard - Receipt Page Sketch` | BLOCKED on visual direction | Same blocker as C2. Draft together. |
| `Standard - CLI Copy` | Not in manifest (future) | Exact CLI success/error/progress strings remain downstream of A7's register spec. Implementation can make reasonable first-pass strings calibrated against `bun`. |
| 5.2 + 5.4 Institutional Memory | No card planned | Accumulates after first Decision revision or first Health Check finding. |

**Permanent acknowledgments:** 2 of 19 manifested cards remain blocked on product-owner visual direction. This is the only outstanding dependency; none of the six new cards are on its critical path.

---

**Status: DONE**

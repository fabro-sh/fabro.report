# Source Assessment: Area 1.1 Product Vision

**Source reviewed:** `/Users/bhelmkamp/p/fabro-sh/fabro.report/docs/alexandria/sources/product-vision.md`
**Config:** Factory × Low × Low, 22-area pool; area 1.1 is Foundation tier.
**Assessor:** Conan
**Date:** 2026-04-22

This is raw source material, not a card. Assessment evaluates trustworthiness, scope preservation, and card-candidate inventory. No grading.

---

## Trustworthiness: HIGH (with one caveat)

The document is rigorously sourced and self-disciplined about provenance:

- Section "Confirmed Product Decisions" (L74–88) enumerates 8 specific decisions with crisp boundaries. Each is decision-shaped, not aspiration-shaped.
- Section "Raven's Inferences (NOT confirmed)" (L113–119) isolates three inferred framings and flags them explicitly. This separation is done correctly and is the single most important trust signal in the document.
- "Open Questions" (L121–128) lists four unresolved items with no false confidence.
- Cross-referencing with `alexandria-config.json` → `discovery.confirmed_entities` matches: Report, Debug Bundle, Reporter, Triager, Report Metadata Page are consistent between the two docs. No drift.

**Caveat — single source:** The source is one conversation with Bryan. Nothing in the repo corroborates any of this. That is appropriate for greenfield Factory mode, but it means every "confirmed decision" rests on a single point of evidence. Sam should not treat this as equivalent to decisions that also appear in shipped code or written specs. Low risk; note it and move on.

---

## Decision vs Inference Separation: CLEAN

| Layer | Where | Verdict |
|---|---|---|
| Confirmed decisions | L74–88 | 8 items, each testable, each narrow enough to ship. Clean. |
| Inferences flagged as such | L113–119 | 3 items with plausible-but-unconfirmed reasoning. Correctly quarantined. |
| Open questions | L121–128 | 4 items. None block Vision card draft; all worth resolving before final. |

The "near-headless, not literally headless" note (L119) is exactly the right kind of epistemic hygiene — it catches a word Bryan used, shows why literal reading would mislead downstream agents, and proposes the substitute ("minimal public surface"). Keep it.

---

## Scope-Distinction Preservation (load-bearing): STRONG

The narrow-scope distinction vs `bun.report` is handled with appropriate weight:

- L33–48 "What `fabro.report` Is Not": bullet-list-negated against a concrete comparison product, with a direct quote calling out that only the intake URL shape is borrowed.
- L48 explicitly names the cascade risk: "Factory-mode builder agents, told this product is 'like `bun.report`,' will drift toward the fuller `bun.report` shape unless the narrower scope is explicit in the library."
- L100–105 "Template Relationship" restates the same distinction from the template side.
- L137 closes with the one-sentence load-bearing claim: *this product is intentionally a pickup point, not a triage surface.*

This is the single most important thing the source had to get right, and it is stated three times in three registers (features list, template relationship, closing thesis). Strong.

One minor risk: the Anti-Patterns card (area 3.5) is referenced at L48 as the place where drift will be named directly. Conan should ensure the Anti-Patterns source material, when written, actually repeats this distinction — do not let the Vision be the only place it lives.

---

## Coverage by Dimension

| Dimension | Coverage | Notes |
|---|---|---|
| WHAT | High | L18–32 flow enumeration is crisp and testable. Users section (L51–64), Security Model (L67–72), Confirmed Entities (L91–98) all concrete. |
| WHY | High for Vision; thin on strategic WHY | The narrow-scope WHY is excellent (stated as cascade-risk prevention). But the *strategic* WHY — why Fabro needs this service at all — lives only in Raven's inferences (L117 "reducing friction in bug reporting"). For a Foundation card, strategic WHY should be confirmed by Bryan before the Vision card is finalized. |
| WHERE | High | Parent product context (L108–111), template reference (L100–105), CLI-command-not-yet-in-Fabro note (L111) all well-placed. |
| HOW | Medium | End-to-end flow (L22–31) is present. S3 storage, capability-URL model, metadata shape all named. Missing: what the Report ID looks like (UUID? short slug?), what makes the URL "short," delete mechanics (immediate? soft?). Acceptable for Vision; will need HOW fill-in at product-layer cards. |
| WHEN | Medium | 30-day retention stated. `fabro report` subcommand is new — flagged. No roadmap signal beyond "narrow forever" implication (L127 flags this as unresolved). Acceptable. |

---

## Card Candidates from This Source

The source explicitly notes (L132–137) that not everything here belongs in the Vision card. Conan's inventory agrees:

| Candidate Card | Type | Zone | Source Section | Priority |
|---|---|---|---|---|
| Product Thesis - fabro.report is a Pickup Point, Not a Triage Surface | Product Thesis | /rationale/ | L33–48, L100–105, L137 | Foundation — build first |
| Principle - Narrow Scope vs bun.report | Principle | /rationale/ | L33–48, L100–105 | Foundation — may collapse into Thesis; decide at Inventory |
| Principle - Capability URLs Are the Entire Access Control Surface | Principle | /rationale/ | L67–72 | Core |
| Standard - Report Metadata Shape | Standard | /rationale/ | L80–84, L94 | Core (feeds Noun Vocabulary area 2.2) |
| Standard - Retention Policy (30-day default) | Standard | /rationale/ | L84 | Amplifier |
| Domain - fabro.report Intake Service | Domain | /product/ | L18–32 overall | Core |
| Component - Report | Product Entity | /product/ | L93–94 | Core (area 2.3) |
| Component - Debug Bundle | Product Entity | /product/ | L95 | Amplifier (out-of-scope internals) |
| Component - Report Metadata Page | Product Entity | /product/ | L98 | Core |
| Agent - Reporter | Persona/Agent | /product/ | L53–60 | Foundation (area 1.3 JTBD input) |
| Agent - Triager | Persona/Agent | /product/ | L61–64 | Foundation (area 1.3 JTBD input) |
| Decision - No Dedup (bundles expire anyway) | Decision Log Entry | /rationale/ (or 5.1) | L84 | Amplifier |
| Decision - Capability URLs over Auth | Decision Log Entry | /rationale/ (or 5.1) | L67–72 | Amplifier |
| Anti-Pattern - "Like bun.report" Drift | Anti-Pattern | /rationale/ (area 3.5) | L48, L100–105 | Foundation — must exist |

Notes for Inventory (Job 1):
- The Product Thesis and the narrow-scope Principle may collapse into one card. Decide at Inventory time based on whether Sam can state the Thesis without restating the scope distinction inline.
- Decisions (no-dedup, capability-URLs, 30-day retention) should land in the Key Decisions Log (area 5.1), not in the Vision card. Source already flags this at L135–136. Honor it.
- Reporter/Triager cards will feed area 1.3 (JTBD) and are user-shaped, not feature-shaped. Keep them thin here and let 1.3 inflate them.

---

## Standard Candidates

Extract-ready specification content, in priority order:

| Content | Source Location | Extraction Notes |
|---|---|---|
| Report metadata fields (ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata, retention deadline) | L94, L80–84 | Ready. Becomes the spine of Standard - Report Metadata Shape; also feeds Noun Vocabulary (2.2). |
| 30-day retention default | L84, L30 | Ready. One-line Standard or a field in Standard - Report Metadata Shape. |
| Capability URL access model (view metadata + delete on possession; bundle contents never exposed via service) | L67–72 | Ready. Becomes Standard - Capability URL Security Model. |
| Metadata-page scope (ID + timestamp + size + delete button; no bundle contents) | L32, L98 | Ready. Becomes Standard - Report Metadata Page Surface. |
| No-dedup rule + rationale | L84 | Ready as Decision Log entry, not as Standard (no ongoing spec to enforce beyond "don't add dedup"). |
| No-auth rule on public surface | L67–72, L85 | Ready as Principle or Decision. |

---

## Anti-Pattern Content: PRESENT AND LOAD-BEARING

| Found | Location |
|---|---|
| Drift toward full `bun.report` shape when told "like `bun.report`" | L33–48 (explicit feature negation list); L48 (cascade-risk rationale); L105 (template-relationship restatement) |
| "Headless" overclaim — product is near-headless, not literally | L119 |
| Treating the service as a triage surface rather than a pickup point | L137 |

This is strong anti-pattern material — not merely present but specifically framed as cascade prevention. The Anti-Patterns card at area 3.5 should inherit this directly.

---

## Source Gaps

### Critical (blocks Vision card finalization — not blocking draft)
- **Strategic WHY is only inferred.** L117 ("reducing friction in bug reporting") is in the Inferences section. For a Foundation-tier Vision card, Bryan should confirm or replace this motivation before the card is signed off. Acceptable to draft without it; not acceptable to ship without it.

### Addressable (proceed with caution)
- **Success criteria missing.** L125 flags this. Vision card can be drafted without success metrics; Strategy card (1.2) and Roadmap card (5.3) cannot.
- **"Narrow forever" is implied, not stated.** L127. Affects Roadmap tone and whether the Vision card should carry a "and this will not broaden" clause. Ask Bryan before finalizing.
- **Managed-Fabro relationship undefined.** L128. If a hosted Fabro offering ever exists, do those bundles land here? Low-priority but worth a one-line answer to prevent future agent guesswork.

### Nice to have
- **Report ID/URL shape.** Not specified. Short slug? UUID? Human-readable? Fine for Vision; will be needed for Standard - Report Metadata Shape.
- **Delete semantics.** Immediate S3 object deletion, or soft-delete with grace period? Needed at product-layer card time.
- **Domain ownership** for `fabro.report` DNS (L126). Orthogonal to Vision; log somewhere.

---

## Issues Sam Should Address (Follow-up Revision, Optional)

These are small improvements. None block proceeding to Vision card draft.

1. **L119 "headless" note could be promoted.** It currently sits in the Inferences section. The observation ("near-headless, not literally") is a clean epistemic catch that deserves its own line in a "Vocabulary Cautions" sub-section or similar. Low priority — the current placement works.
2. **L111 CLI-subcommand note is load-bearing but buried.** The fact that `fabro report` does not yet exist in Fabro's CLI surface and this project implies adding it is a cross-repo dependency. Worth surfacing as either its own bullet under "Confirmed Product Decisions" or a flagged item for Institutional Memory (5.2+5.4).
3. **No explicit statement of what the user *doesn't* see between upload and URL return.** E.g., is there a progress indicator? Error surface? This is HOW-dimension fill-in and probably belongs at product-layer card time, not here. Log only.

---

## Readiness: READY_FOR_CARDS

The source material is trustworthy, cleanly separates confirmed decisions from inferences, preserves the load-bearing scope distinction in triplicate, and enumerates card candidates with enough fidelity to drive Inventory. Known gaps are all flagged in the source itself, which is the right behavior.

Vision card can be drafted now. Finalization (not drafting) should wait on:
- Bryan confirming strategic WHY (L117)
- Bryan confirming "narrow forever" posture (L127)

Neither blocks the Inventory job.

---

## Recommendation

**READY_FOR_CARDS** — proceed to Job 1 (Inventory) for the full 22-area pool using this source plus any others Sam has produced.

**Next agent action:** Conan runs Job 1 (Inventory) to manifest the full card list and build order. The candidates table above is the starting input, not the final manifest.

**Parallel human action (non-blocking):** Bryan confirms strategic WHY and the "narrow forever" posture before the Vision card is finalized (not before it is drafted).

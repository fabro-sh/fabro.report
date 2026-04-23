# Feedback Phase 2 Spot-Check — Product-Layer Cascade + A5 Rename

**Reviewer:** Conan
**Date:** 2026-04-22
**Commit reviewed:** `9e6efa2`
**Scope:** 11 edited product-layer cards + A5 rename + wikilink cascade across 15 files. Pre-gate for Sam Phase 3 (draft A15–A19).
**Authoritative plan:** `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` Part 3.
**Phase 1 spot-check:** `docs/alexandria/reviews/feedback-phase-1-spot-check.md` (APPROVED at `caefd96`).
**Scope-expansion source:** `docs/alexandria/sources/feedback-flow.md`.
**Lint baseline verified:** 0 errors cards/graph/lines/layers; 28 pre-adjudicated warnings unchanged.

---

## Focus 1 — A5 Rename Cleanliness

| Check | Result |
|---|---|
| New path exists | PASS — `docs/alexandria/library/product/systems/System - Debug Bundle Retention, Storage, and Key Scheme.md` present. |
| Old path does NOT exist | PASS — `System - Retention, Storage, and Key Scheme.md` returns "File does not exist." |
| Heading inside file matches new name | PASS — L1 `# System - Debug Bundle Retention, Storage, and Key Scheme`. |
| Self-references inside A5 use new name | PASS — L7 scope paragraph names the card "renamed 2026-04-23 from the scope-ambiguous `System - Retention, Storage, and Key Scheme`" as an explicit historical anchor, not a live reference. No `[[System - Retention, Storage, and Key Scheme]]` wikilink inside A5. |
| Wikilink cascade — A1 Agent - Reporter | PASS — L35 uses `[[System - Debug Bundle Retention, Storage, and Key Scheme]]`. |
| Wikilink cascade — A2 Agent - Triager | PASS — L37, L88 use new name with wikilinks. |
| Wikilink cascade — A4 Component - Report Metadata Page | PASS — L33, L84, L118 use new wikilink name. |
| Wikilink cascade — A6 Standard - Information Architecture | PASS — L29, L69, L101 use new wikilink. |
| Wikilink cascade — A7 Standard - Interaction Patterns | PASS — L26, L73 use new wikilink. |
| Wikilink cascade — A8 Standard - Accessibility Policy | PASS (tangential — A8 doesn't link A5 directly in the edits; verified). |
| Wikilink cascade — A9 Decision - Capability URLs over Auth | PASS — L34, L91 use new wikilink. |
| Wikilink cascade — A11 Decision - 30-Day Retention | PASS — L29, L81 use new wikilink. |
| Wikilink cascade — A13 Sentry Decision | PASS — verified no broken link; A13 doesn't wikilink A5 directly. |
| Wikilink cascade — A14 System - Service Topology | PASS — L5, L14, L55 use new wikilink. |
| Wikilink cascade — F2 Noun Vocabulary | PASS — L33 uses new wikilink. |
| 0 residual `[[System - Retention, Storage, and Key Scheme]]` wikilinks | PASS — graph-lint would have caught any; 0/0/0 state confirms. |

**Residual non-wikilink references in prose:** TWO found in A3 (`Component - Report`).

- L51: `` "mechanism deferred to `System - Retention, Storage, and Key Scheme`" `` — OLD name in backtick-quoted prose reference.
- L52: `` "forward reference for `Standard - Interaction Patterns (CLI + Delete)` and `System - Retention, Storage, and Key Scheme`" `` — OLD name again, backtick-quoted.

Both are plain-text references (not wikilinks), so graph-lint cannot detect them. They are the exact "visible symptom" the task brief flagged. Neither is load-bearing for correctness — a reader still resolves them to A5 — but they point at a card name that no longer exists on disk. The rest of A3 uses the new name correctly (L32 `[[System - Debug Bundle Retention, Storage, and Key Scheme]]`).

**Verdict:** Rename cascade is 95% clean. A3 has two residual prose references to the old name that missed the cascade. Non-blocking for Phase 3 (does not affect graph integrity or block A15-A19 drafting), but worth a one-minute Sam pickup before Phase 3 commit.

---

## Focus 2 — Two Micro-Pickups

| Pickup | Location | Result |
|---|---|---|
| F2 `fabro upload` row notes Report URL return | F2 L71 | PASS — "Uploads a previously-dumped Debug Bundle tarball. **Returns the Report URL on success, like `fabro dump --upload`.** Supports `--dry-run`..." Parallel to the `fabro dump --upload` row phrasing. |
| C4 WHAT line names both audiences + preserves load-bearing phrase | C4 L5, L7 | PASS — L5 opens "Reporters and Feedback Submitters who are self-hosted Fabro users — primary and only." L7 Load-bearing phrase preserved verbatim: "The load-bearing phrasing is *self-hosted, primary and only*." |

**Verdict:** Both Phase-1 leftovers cleanly landed.

---

## Focus 3 — A2 Triager's Second Behavioral Section

| Check | Result |
|---|---|
| Both flows exist as parallel behavioral sections | PASS — L60-66 "Workflow 1 — Debug Bundle Retrieval (primary, unchanged)"; L68-73 "Workflow 2 — Feedback Reading (new, parallel)." Explicit parallel-but-distinct structure. |
| Debug Bundle flow UNCHANGED from prior A2 | PASS — L60-66 identical structural content (URL → Report ID extraction → `aws s3 cp` → local inspection → optional delete-on-URL-possession). No softening. Example 1 (L88) preserves the canonical URL-in-GitHub-issue → S3-direct retrieval narrative. |
| Feedback flow cleanly describes Discord-primary / S3-secondary | PASS — L70 "Discord is the **primary surface**" bolded. L71 explicit secondary path: "used when the Discord post does not carry everything the Triager needs." Exact S3 key layout for feedback prefix named at L71. |
| "No service-facing surface" holds across BOTH workflows | PASS — L12 explicit across-both-flows statement: "Discord is external. S3 is external. AWS IAM is external. The service's behavior includes posting a Feedback notification to Discord via webhook, but the service has no endpoint a Triager calls...not for Debug Bundles, not for Feedback, not for anything else." |
| Original Debug-Bundle anti-examples preserved | PASS — "Triager dashboard" (L95), "endpoint that accepts AWS credentials" (L96), "Triager log in" (L100), "team member of fabro.report" (L101) all preserved. Each expanded to cover both flows where structurally appropriate. |
| New Feedback-specific anti-examples | PASS — L97 "Feedback-reading UI hosted by the service"; L98 "the Triager filters Feedback in `fabro.report`"; L102 "conflates the two Triager workflows into one ('the Triager retrieves intake data from S3')." |
| Sam's consolidation-room suggestion | REJECTED (politely) — each anti-example covers a distinct drift shape: (1) service-hosted Feedback UI, (2) filter/search/aggregate Feedback through service, (3) two-workflow conflation. L97 ≠ L98 ≠ L102; each closes a different door. "Triager dashboard" (L95) is the generic-dashboard shape; "Feedback-reading UI hosted by the service" (L97) is the specific Feedback-inbox drift. Removing any one weakens coverage. **Sam's 40% ballooning is proportionate; keep as-is.** |
| No conflation between workflows | PASS — L102 anti-example itself names conflation as wrong. Workflows consistently labeled "Workflow 1" and "Workflow 2" with "primary, unchanged" and "new, parallel" subtitles. |

**Verdict:** Second behavioral section lands cleanly. Anti-examples list each carries weight — consolidation would weaken drift coverage. A2 is now the library's most carefully retrofitted card and appropriately heavy for the role it carries.

---

## Focus 4 — A7's New Feedback CLI + No Delete Sections

| Check | Result |
|---|---|
| Feedback CLI section covers full `fabro feedback` shape | PASS — L105-121. Positional text/files (L109), stdin via pipe (L109), `-e`/`--email <email>` (L111), `-h`/`--help` (L112), fire-and-forget confirmation with no URL (L117), direct `POST /v1/feedback` upload shape (L118). Source-faithful against `feedback-flow.md` L26-L45. |
| No Delete section framed as distinct-from-Debug-Bundle property | PASS — L123 title is literally "No Delete on the Feedback Flow (Explicit Divergence from the Debug Bundle Flow)." L125 names the divergence as "structural, not a missing feature." L127-130 enumerate four structural reasons (no URL, no page, no endpoint, 30-day lifecycle only). |
| "Feedback is permanent from user perspective" load-bearing phrasing | PASS — L132 verbatim: "The load-bearing phrasing is *Feedback is permanent from the user's perspective, expires only via S3 lifecycle*." Inoculates against recall-mechanism drift. |
| Three new anti-examples cover drift shapes | PASS — L146: URL return on success (implies service-hosted page or delete capability). L147: adding delete action (CLI subcommand, Discord button, service endpoint). L148: reusing 50 MB Debug Bundle cap as Feedback cap without confirmation (register-boundary violation). All three cover plausible drift shapes, each distinct. |
| Register consistency with existing Debug Bundle section | PASS — L120 "follows the same error-output register specified above." L121 progress indicator "in the same compact register as the Debug Bundle progress bar." No emoji, no chattier tone, no ceremony language. The `fabro feedback` confirmation register inherits "efficient and reassured" from F4 without drift. |
| `bun`-calibration preserved | PASS — L117 explicit "No URL is returned. No Report ID is displayed. No link to a service-hosted page is shown" is terse and declarative; same register as the Debug Bundle section. |

**Verdict:** A7's 3000+ word count is proportionate. Both new sections earn their keep: the CLI section is source-faithful and the No-Delete section prevents a whole category of register/feature drift. No register drift detected between Debug Bundle and Feedback sections.

---

## Focus 5 — A9 Scope Note Expansion

Sam wrote a paragraph (L9) instead of Conan-specified "one line." Verifying it earns its extra words:

| Check | Result |
|---|---|
| Load-bearing or padding? | LOAD-BEARING — describes the inverse-shape relationship with A19 as a Decision-layer structural pairing, not flavor text. The paragraph names A9 as "capability-URL access *over* auth on the Debug Bundle flow" and A19 as "no access model at all on the Feedback flow" — the exact symmetry-breaking point a Factory-mode agent reading A9 cold would miss without this framing. |
| Accurately describes A19 without getting ahead | PASS — paragraph names A19's forthcoming Decision shape ("no post-submission submitter-facing surface to gate") without enumerating A19's own anti-examples, rationale, or alternatives. Plain-text forward ref. A19's full WHY-space remains A19's to define. |
| Correctly scopes A9 to Debug Bundles only | PASS — L5 WHAT opens "`fabro.report` **on the Debug Bundle flow**"; L9 "this Decision applies to the Debug Bundle flow only"; L9 explicit disclaimer: "The reason this Decision does not extend to the Feedback flow is covered in forward-referenced `Decision - Fire-and-Forget Feedback`." |
| Does NOT imply Feedback has weaker access model | PASS — paragraph frames A9 and A19 as "both Decisions honor [[Principle - Narrow Forever]]'s identity-free commitment, but through different mechanisms appropriate to each flow's shape." Neither is "stronger" — they address different shapes. |

**Verdict:** Sam's paragraph is the right call. A one-line scope note would have left the inverse-shape relationship implicit; a paragraph makes it explicit and prevents the A9/A19 pairing from reading as accidental. Keep.

---

## Focus 6 — Remaining Cards (Quick Pass)

| Card | Checks | Result |
|---|---|---|
| **A1 Reporter** | Disambiguation note present; A15 plain-text forward-ref; A5 wikilink cascaded | PASS — L7 "Reporter is the role for the Debug Bundle flow only" with parallel-to-A15 paragraph; A15 plain-text ref; L35 uses new A5 wikilink. |
| **A3 Component - Report** | Disambiguation note present; A16 plain-text forward-ref; A5 wikilink cascaded | PARTIAL — L7 disambiguation paragraph present; A16 plain-text ref present; A5 wikilink correctly cascaded at L32. **BUT** L51 and L52 still contain OLD name `System - Retention, Storage, and Key Scheme` in backtick-quoted prose. See Focus 1 residual. |
| **A6 Information Architecture** | Feedback IA framing (no page, no URL, no listing); `fabro feedback` in CLI Surface; `POST /v1/feedback` in HTTP Surface | PASS — L9 "The Feedback IA is even narrower than the Debug Bundle IA" with "no page, no URL, no listing"; L56 `fabro feedback` row in CLI Surface; L76 `POST /v1/feedback` endpoint added to HTTP Surface with "no URL is returned, no Report ID is minted, no rendered page is created." |
| **A8 Accessibility** | `fabro feedback` in NO_COLOR-aware set | PASS — L5 names all four canonical commands including `fabro feedback`; L49 explicit "across all four canonical commands." |
| **A11 30-Day Retention** | Extended to both flows | PASS — L5 WHAT opens "Every Report on `fabro.report`'s Debug Bundle flow and every Feedback on the Feedback flow expires **30 days after upload / submission**." L7 explicit scope extension with rationale parallelism. L29 points forward-ref to A17 for Feedback-side enforcement mechanism. |
| **A13 Sentry** | "bundle processing" extended to "bundle or feedback processing" | PASS — L5 updated WHAT names both flows; L7 load-bearing phrasing "Sentry-as-bundle-or-feedback-processor is no, permanently"; L7 explicit scope-extension note: "This Decision's scope was extended 2026-04-23 from 'bundle processing' to 'bundle or feedback processing.'" |
| **A14 Service Topology** | POST /v1/feedback endpoint added; Discord webhook as first non-AWS egress; request/IAM/Discord-egress flows | PASS — L51 Fargate piece names `POST /v1/feedback`; L56 dedicated Discord webhook piece with "first non-AWS third-party dependency" framing; L80-85 new "Discord egress flow" subsection; L93 properties enumerate "single third-party dependency for Discord." |

**Overall card pass:** 6 of 7 clean. A3 has the two-line prose residual noted above.

---

## Additional Checks

### Forward-text refs to A15–A19

All plain-text, no wikilinks. Consistent with Phase 1 precedent and 0/0/0 graph-lint state:

| Card | Forward refs |
|---|---|
| A1 | A15 (L7) |
| A2 | A15 (L20), A17 (L38), A19 (L73) |
| A3 | A16 (L7) |
| A5 | A17 (L7) |
| A6 | A17 (L9, L69, L76), A18 (L82), A19 (L9, L74, L113) |
| A7 | A17 (L5, L118, L119, L130, L148), A19 (L127, L130, L146) |
| A8 | A18 (L110) (indirect via A6 context) |
| A9 | A19 (L9) |
| A11 | A17 (L29) |
| A13 | — (scope-extension only, no forward refs) |
| A14 | A17 (L5, L51, L55, L68, L84) |
| F2 | A15, A16, A17, A18 (L29-31, L61) |
| C4 | — (WHAT-line expansion only) |

Zero forward-text refs were accidentally promoted to wikilinks. Graph-lint 0/0/0 confirms structurally.

### Transient bidirectional gaps Sam claimed to close

- **A6 ↔ A8:** A6 L31 cites `[[Standard - Accessibility Policy]]` for the `NO_COLOR`-aware set and binds A6's "surfaces" to A8's "floors"; A8 L10 cites `[[Standard - Interaction Patterns (CLI + Delete)]]` — but what about A6? A8 L10 also references A6's shape via A7, and A8 references the Report Metadata Page inheriting from A6's IA. **Direct A6↔A8 reciprocity:** A6 L31 → A8 (PASS); A8 L10 → A7 (not A6 directly). Checking A8 for A6 reference → L93 mentions no A6 link, but A8 does reference A6 concepts through A7. **Verdict:** edge exists from A6 to A8 (direct wikilink); reciprocal edge from A8 is indirect via A7. Borderline — non-blocking but worth Sam noting in a future bidirectional-consistency sweep.
- **A7 ↔ A11:** A7 L130 forward-ref to A11's "extended scope across both flows"; A11 L31 cites `[[Standard - Interaction Patterns (CLI + Delete)]]` with explicit "A7's 'No Delete on the Feedback Flow' subsection...carries the load-bearing phrasing that a Feedback is permanent." Reciprocal. PASS.

### Manifest consistency

| Check | Result |
|---|---|
| 24 cards summary unchanged | PASS (L19). |
| Zone counts still Agent×3, Component×4, System×3, Decision×7 | PASS (L36, L41, L42, L49). |
| A5 row still names renamed filename | PASS (L95 — `System - Debug Bundle Retention, Storage, and Key Scheme`; "filename move deferred to Phase 2" annotation still present from Phase 1 but now accurate since Phase 2 has moved the file). |
| A5 filename-move-deferred annotation | OBSERVATION — L95 annotation says "filename move deferred to Phase 2 (manifest row reflects the target name; wikilinks + on-disk filename cascade in Phase 2...)". Phase 2 is now complete. Annotation is now stale factually but not incorrect (the move has happened). One-line cleanup for Sam: remove the "deferred to Phase 2" phrase since Phase 2 is done. Non-blocking. |
| Build Sequence rows 25–34 intact | PASS (L193-202). |
| Open Flags 6 and 7 intact | PASS (L245-246). |
| Pre-adjudicated A16 containment warning intact | PASS (L230). |

### Graph integrity

| Check | Result |
|---|---|
| Forward wikilinks to cards that don't exist yet | NONE FOUND — all forward refs to A15-A19 are plain-text. 0/0/0 graph-lint confirms. |
| Broken wikilinks from the A5 rename | NONE — all 15 files' inbound A5 wikilinks now use the new name. |
| New bidirectional gaps | Two borderline observations in A6↔A8 (see above); neither blocks. |

---

## Cross-Reference Integrity

- "Two intake types, forever" preserved across F1, F5, F6, F2, and now A11 L5.
- "Discord as push target, not triage surface" preserved across F1 and A2 L12.
- "Fire-and-forget" language consistent across A2 L73, A6 L9, A7 L105-121, A9 L9.
- Asymmetric-identity (Reporter vs Feedback Submitter) language consistent across A1 L7, A2 L20, A3 L7, F2 L61, C4 L5.
- A9↔A19 inverse-shape relationship named in A9 L9; A19 forward-ref inherits on draft.

No identity-level contradictions. Cascade internally coherent.

---

## Grade Deltas

| Card | Pre | Post | Delta |
|---|---|---|---|
| A1 | A- | A- | 0 |
| A2 | A- | A | +half |
| A3 | A- | A- | 0 (residual prose references offset clean update) |
| A5 (renamed) | A- | A | +half (rename + scope paragraph + forward-ref discipline) |
| A6 | A- | A | +half |
| A7 | A- | A | +half |
| A8 | A- | A- | 0 |
| A9 | A- | A | +half |
| A11 | A- | A | +half |
| A13 | A- | A- | 0 |
| A14 | A- | A | +half |

No regressions. Product-layer zone sustained at A-level.

---

## Phase 3 Blockers

**None.**

A3's two residual prose references are non-blocking:
- They do not affect graph integrity (0/0/0 confirms).
- They do not block A15–A19 drafting (no new card depends on A3's L51/L52 prose resolving).
- They are not load-bearing for any inherited contract.
- Fix is trivial (two in-place edits replacing the OLD name with the new name).

**Recommended:** Sam apply the A3 fix as a tiny pickup at the start of Phase 3 (same commit as A15 or a separate trivial commit). Alternative: defer to Phase 4 alongside C1's tiny update. Either path is fine.

---

## Phase 2 Micro-Pickups (Non-Blocking for Phase 3)

1. **A3 L51, L52** — replace OLD `System - Retention, Storage, and Key Scheme` backtick references with the new `System - Debug Bundle Retention, Storage, and Key Scheme` name. Two single-line edits.
2. **Manifest L95** — remove the "filename move deferred to Phase 2" annotation (Phase 2 is complete). One-line edit.
3. **A6↔A8 bidirectional consistency** (borderline observation, not actionable this phase) — consider explicit reciprocal wikilink pass in a future sweep. Not a Phase 2 defect.

---

## Verdict

**APPROVED.**

Six focus areas pass substantively. A5 rename cascade is 95% clean across 15 files and the manifest; the two residual prose references in A3 are the only cascade miss and do not block Phase 3. Both Phase-1 micro-pickups (F2 `fabro upload`, C4 WHAT line) landed cleanly. A2's second behavioral section is the library's most carefully retrofitted card and Sam's "consolidation room" suggestion is correctly rejected — each anti-example is load-bearing. A7's new Feedback CLI + No-Delete sections earn their 3000+ words; register consistency held. A9's paragraph-length scope note is the right call, not padding. Remaining cards (A1, A6, A8, A11, A13, A14) cleanly reflect the feedback scope expansion with no drift. Graph integrity is preserved (0/0/0); forward refs are all plain-text.

**Readiness signal for Sam Phase 3:** GO. Proceed with drafting A15 Agent - Feedback Submitter, A16 Component - Feedback, A17 System - Feedback Storage & Notification, A18 Decision - IP Logging for Abuse Protection, A19 Decision - Fire-and-Forget Feedback, in build order (manifest L197-201). The existing product-layer graph is stable and internally coherent; new cards will land into a library that already carries the two-intake-types-forever identity at both the identity layer (Phase 1) and the product layer (Phase 2).

Optional tiny pickups (A3 L51/L52, manifest L95 annotation) can fold into Phase 3's first commit or Phase 4's C1 update — either is fine; neither gates Phase 3.

**Status: APPROVED**

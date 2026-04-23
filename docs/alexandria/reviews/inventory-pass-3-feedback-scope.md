# Inventory Pass 3 — Feedback Scope Expansion

**Job:** Conan Job 1 — Inventory (incremental, scope-expansion pass)
**Date:** 2026-04-23
**Prior state:** 17 of 19 cards ratified (commit `c7194da`); C2/C3 still blocked on visual direction. Inventory Pass 2 captured at `docs/alexandria/reviews/inventory-pass-2-tech-stack.md`.
**Triggering sources:**
- `docs/alexandria/sources/feedback-flow.md` (new, 2026-04-23) — full capture of the `fabro feedback` intake
- `docs/alexandria/sources/product-vision.md` (updated 2026-04-23) — adds scope-expansion + absolute-ceiling decisions
**Product owner confirmation (2026-04-23):** exactly two intake types forever (Debug Bundles + Feedback). No third without an identity-change conversation.

---

## Part 1 — Identity-Level Impact Assessment

The library's load-bearing identity was one-intake. It is now two-intake. That is not a decoration change; that is a change to the noun that F1/F6 protect. Below, each identity card gets a single honest read and a minimum-surgery verdict.

### F1 — Product Thesis: "Pickup Point, Not Triage Surface"

**Survives?** Yes. The "pickup point" framing is the deepest correct framing we have. Feedback is also a pickup — the Feedback Submitter drops a message, the service captures it, Triagers retrieve it out-of-band (Discord, optionally S3). The Triager still does not interact with the service itself. The service still has no triage surface.

**What breaks?** The card's prose is Debug-Bundle-centric throughout. Nouns (Report, Debug Bundle, Reporter) dominate; the "flow" section describes exactly one flow; the out-of-scope list reads as though one intake is the identity.

**Minimum-surgery fix:** Broaden, don't rewrite. Add:
1. A one-paragraph framing that `fabro.report` has **two pickup shapes** (Debug Bundles; Feedback), both serving the same identity ("pickup point, not triage surface"), with different post-pickup mechanics.
2. A parallel flow description for Feedback alongside the existing Debug Bundle flow.
3. A clarifying sentence that Discord is a push-notification target, not a triage surface hosted by the service — the service still has no team-facing UI.
4. Preserve every existing load-bearing sentence about capability URLs, S3-direct retrieval, and the narrow scope. Those all apply equally to Debug Bundles.

### F3 — Principle: Reject "Like bun.report" Drift

**Survives?** Yes. The drift-prevention function is intact. `bun.report`'s fuller shape — symbolication, rendering, dashboards, post-upload feedback forms — remains rejected.

**What breaks?** The "feedback collection" drift entry is now ambiguous. In its original context, it rejected *post-upload feedback forms on the Debug Bundle flow* (the pattern bun.report uses). It reads today as though it rejects any feedback-shaped intake — which would contradict the now-confirmed `fabro feedback` command.

**Minimum-surgery fix:** Apply the Sentry-distinction pattern (the one Conan already approved for A13 and the inline F3 note about Sentry). Replace the "feedback collection" bullet with a bullet that distinguishes two things:
- **Rejected:** post-upload feedback forms attached to the Debug Bundle flow (the bun.report pattern — "how was this bundle?").
- **In scope:** the standalone `fabro feedback` CLI, which is a separate intake shape, not a bundle-flow extension. See `Component - Feedback` + `System - Feedback Storage & Notification`.

Also add a note to the "scope-drift enumeration" framing that the new ceiling is "two intake types, forever" — so the drift list now rejects any *third* intake surface with the same force it used to reject any second. Tightening + clarifying, not wholesale rewrite.

### F5 — Principle: Strategy, Support Effectiveness for Self-Hosted Users

**Survives?** Yes, cleanly. The Strategic WHY was "support effectiveness for self-hosted users via team retrieval of data the team needs to help." Feedback serves exactly that WHY via a different data shape (voice / bug-reports / feature-requests rather than reproducibility data). The strategy is unchanged; the *mechanisms* now include two shapes.

**What breaks?** Prose emphasis. The card likely reads as though Debug Bundles are the only mechanism. A reader today could conclude "voice capture isn't part of the strategy" — but it clearly is.

**Minimum-surgery fix:** One small prose expansion. A sentence or short paragraph that names Feedback as the second mechanism serving the same strategic WHY: users on their own systems need multiple low-friction ways to get relevant signal to the team; Debug Bundles carry reproducibility data, Feedback carries voice. Both serve "make it easy to help self-hosted users." No paired principle card needed — that would duplicate F5.

### F6 — Principle: Narrow Forever

**Survives?** Yes, but requires the most careful retrofit of the four. F6 was written as a one-intake identity commitment. The commitment now needs to be visibly and permanently two-intake.

**What breaks?** Two things:
1. The card's identity framing needs explicit elevation to "two intake types, exactly, forever." Without that, a future agent reads F6, sees "narrow forever — bundles only," and treats `fabro feedback` as drift.
2. The out-of-scope list is mostly still correct (symbolication, rendering, team dashboards on the service, triage UIs, per-user accounts, cross-upload identity tracking) — all apply equally to both flows — but should be restated as applying across both.

**Minimum-surgery fix:** Explicit rewrite of the identity sentence + retrofit of the out-of-scope list framing. Specifically:
1. New identity line: "exactly two intake types — Debug Bundles and Feedback — permanently. A third intake surface is an identity change, not a feature."
2. Convert the out-of-scope list's framing from "beyond intake" to "across both intake types": symbolication of either, rendering of either, dashboards of either, etc.
3. Add a "confirmed-and-bounded expansion" entry logging the 2026-04-23 scope change: feedback is in, ceiling is now two, commitment is otherwise intact.

This is the one card where "just broaden the prose" is insufficient — the commitment's shape changed, so the card's identity statement must visibly change. Still minimum-surgery (no new card), just the largest edit of the four.

### Verdict across F1/F3/F5/F6

No new Foundation card is required. The "two-intake-types-forever" commitment lives in F1 + F6 as a paired statement, not in a third card. A separate `Principle - Two Intake Types Forever` card would be decorative — it would restate what F1 names (the identity) and what F6 protects (the ceiling). The library doesn't need a third copy.

---

## Part 2 — New Cards Needed

Discipline filter: the library is currently 19 manifested cards at Factory × Low × Low. Adding 6 cards is 31% growth. Each new card must justify its existence as load-bearing, not decorative. The Sentry-distinction precedent (A13 + inline F3 note) shows that a Decision card is warranted when a new *category* enters the library, not every time a detail is worth pinning.

| Candidate | Verdict | Reasoning |
|---|---|---|
| **Agent - Feedback Submitter** | **BUILD** | Load-bearing. A new persona with asymmetric identity vs. Reporter (email optional + IP logged vs. no identity at all). Absorbing into A1 would misrepresent the Reporter's identity model. Parallel-to-A1 structure is already the library's pattern. |
| **Component - Feedback** | **BUILD** | Load-bearing. A first-class entity, parallel to `Component - Report`. Has its own fields, its own ID shape (UUIDv7 vs 32-hex-char), its own lifecycle. Absorbing into A3 would create a card describing two entities, which is exactly the atomicity violation Conan rejects elsewhere. |
| **System - Feedback Storage & Notification** | **BUILD** | Load-bearing. A5 (`System - Retention, Storage, and Key Scheme`) is Debug-Bundle-specific in its key scheme, metadata shape, and object layout. Feedback has a different object shape (`feedback.json` + `feedback.txt`), different ID shape, different prefix, and a notification side (Discord webhook + failure self-reporting) A5 has no reason to carry. Two Systems is cleaner than one bloated System. Absorbs the Discord-webhook mechanics and the IP-logging mechanics inline (see below). |
| **Decision - Discord Webhook for Feedback Notification** | **SKIP (absorb into System card)** | Not load-bearing as a standalone Decision. The rationale ("Discord is a notification target, not a downstream bundle processor — distinct from the Sentry rejection") is one paragraph. It belongs inside `System - Feedback Storage & Notification` as the notification section, with a cross-reference from F3's clarifying note. Adding a full Decision card here is decorative — the decision isn't controversial, doesn't have multiple rejected alternatives worth documenting, and doesn't change across the library. |
| **Decision - IP Logging for Abuse Protection** | **BUILD (thin)** | Borderline, but build. This is the library's first capture of Reporter-adjacent identity data, and it introduces an identity-data pattern the library has been deliberately clean of until now. A future agent needs to find "why do we log IPs?" and not have to dig it out of a System card's implementation section. Keep it thin — rationale, scope of use, retention, what it's *not* used for (no analytics, no cross-submission tracking). Parallel in shape to A10 (`Decision - No Content Dedup`) which is also thin. |
| **Decision - Fire-and-Forget Feedback** | **BUILD (thin)** | Load-bearing. This is the single most confusable divergence from the Debug Bundle flow — a future agent looking at Component - Report's capability URL + delete button will want to know why Component - Feedback has neither. The rationale ("Feedback Submitter's involvement ends at 'sent'; no URL, no delete, because there's no view or capability to gate") is worth its own card so it can be linked from both A3 and Component - Feedback. Parallel in shape to A9 (`Decision - Capability URLs over Auth`) in the inverse. |

**Summary:** 5 new cards — 1 Agent, 1 Component, 1 System, 2 Decisions. Not 6. The Discord Decision folds into the System card.

**Growth:** 19 → 24 cards (+26%). Still proportionate to the scope change (new intake type, new persona, new third-party integration).

---

## Part 3 — Updates to Existing Cards

Cascade map. "Update" means Sam touches the card; "no change" means the card is scoped tightly enough to one flow that no retrofit is needed.

| Card | Update? | What Changes |
|---|---|---|
| **F1 Product Thesis** | YES (medium) | Broaden pickup-point framing to two intake types; add parallel feedback flow description; name Discord as push target, not triage surface. See Part 1. |
| **F2 Noun Vocabulary** | YES (medium) | Add `Feedback` and `Feedback Submitter` entries (parallel shape to Report + Reporter). Update the CLI Commands table to include `fabro feedback` alongside `fabro dump --upload` / `fabro upload`. Do NOT add `Discord channel` or `Feedback webhook` as first-class nouns — they're implementation details of System - Feedback Storage & Notification. |
| **F3 Reject "Like bun.report" Drift** | YES (small, targeted) | Rewrite the "feedback collection" drift bullet using the Sentry-distinction pattern. Add a note to the scope-drift enumeration section that the ceiling is now "two intake types, forever." See Part 1. |
| **F4 Minimal Public Surface Feels Calm** | YES (tiny) | Current card covers CLI output + error messages + receipt page tone. Add that `fabro feedback`'s confirmation message and Discord post format inherit the same calm tone — minimal, efficient, no celebration language. One-paragraph addition; no structural rewrite. |
| **F5 Strategy** | YES (small) | One-paragraph expansion naming Feedback as the second mechanism serving the same Strategic WHY. See Part 1. |
| **F6 Narrow Forever** | YES (medium) | Largest identity-card edit. Rewrite identity line to "exactly two intake types, forever"; retrofit out-of-scope list framing to "across both flows"; add confirmed-expansion log entry for 2026-04-23. See Part 1. |
| **C1 Competitive Context** | YES (tiny) | Note that `bun feedback` is the template shape for `fabro feedback`, parallel to how `bun.report`-the-URL-shape is the template for the Debug Bundle flow. One sentence. Does not change the card's claim that no broader competitor survey is load-bearing. |
| **C4 Decision - Who This Service Is For** | NO | Audience is unchanged — self-hosted Fabro users. Feedback Submitters and Reporters are both members of that same audience. |
| **A1 Agent - Reporter** | YES (tiny) | Add a "not to be confused with Feedback Submitter" disambiguation sentence + wikilink. The Reporter's identity model (none; Fabro version only) is unchanged. |
| **A2 Agent - Triager** | YES (medium) | Add a second behavioral section for the Feedback retrieval workflow. Debug Bundle workflow (S3 direct with own credentials) unchanged. Feedback workflow: read Discord channel → optionally retrieve S3 object for full submission. The Triager is now one role with two workflows, not two roles. |
| **A3 Component - Report** | YES (tiny) | Add a disambiguation note + wikilink to `Component - Feedback`. No field or behavior changes. |
| **A4 Component - Report Metadata Page** | NO | Debug-Bundle-specific; feedback has no metadata page. Disambiguation is implicit via A3's note. |
| **A5 System - Retention, Storage, and Key Scheme** | YES (small) | Rename scope: explicitly state this card covers Debug Bundle storage only. Cross-reference `System - Feedback Storage & Notification` for the parallel feedback system. Alternatively, rename the card to `System - Debug Bundle Retention, Storage, and Key Scheme` — Conan recommendation: rename, because F2's naming convention prefers entity-qualified names when a parallel exists. |
| **A6 Standard - Information Architecture** | YES (small) | IA section needs a parallel "feedback submission has no page, no URL, no listing" statement. The minimal-IA identity is intact — it's now two parallel minimal-IAs. |
| **A7 Standard - Interaction Patterns** | YES (medium) | Add the `fabro feedback` CLI interaction as a parallel section to `fabro dump --upload` / `fabro upload`. Covers: the CLI shape (text / files / piped stdin), the `--email` flag, confirmation output, no-URL-returned, no delete flow. The existing Debug Bundle interaction section is unchanged. |
| **A8 Standard - Accessibility Policy** | YES (tiny) | Add `fabro feedback` CLI to the NO_COLOR-aware set. The receipt page policy is unchanged (no feedback page exists). |
| **A9 Decision - Capability URLs over Auth** | YES (tiny) | Add a one-line scope note: this Decision applies to the Debug Bundle flow only. Feedback has no capability URL model because it has no post-submission access — see `Decision - Fire-and-Forget Feedback`. |
| **A10 Decision - No Content Dedup** | NO | Scope was already bundle-specific. Feedback dedup is not a question anyone asks — the content isn't reproducibility data. Leave alone. |
| **A11 Decision - 30-Day Retention Default** | YES (tiny) | Extend scope to cover both flows — feedback also retains 30 days per the source. One-sentence expansion; the rationale (bundles/feedback expire anyway) holds. |
| **A12 Decision - Service Stack** | NO | Rust on Fargate covers both endpoints. No change. |
| **A13 Decision - Sentry for Service Errors, Not Bundle Processing** | YES (tiny) | Note that "bundle processing" language extends to "feedback processing" — Sentry is still a service-error tracker, not a downstream processor of *either* intake. One-sentence expansion. |
| **A14 System - Service Topology** | YES (small) | Add the feedback endpoint to the topology description. Add Discord webhook as an egress dependency alongside existing AWS dependencies. Note that Discord is the first non-AWS third-party dependency. |
| **C2 Design System** | STILL BLOCKED | No feedback-specific visual direction needed (no page). |
| **C3 Receipt Page Sketch** | STILL BLOCKED | Debug-Bundle-specific; no feedback equivalent. |

**Cascade size:** 17 existing cards get touched, ranging from tiny (1 sentence) to medium (several paragraphs). 4 cards untouched (A4, A10, A12, C4). 2 cards still blocked (C2, C3).

**Recommended rename:** A5 → `System - Debug Bundle Retention, Storage, and Key Scheme`. This makes the parallel to the new `System - Feedback Storage & Notification` visible at the filename level and prevents future agents from mistaking A5 as the storage-System-for-everything.

---

## Part 4 — Manifest Updates

Proposed manifest edits. Sam can apply these as part of the next batch; Conan can apply directly if preferred for speed.

### New Summary Line

> **Summary:** 24 cards to build (19 prior + 5 added 2026-04-23 in Inventory Pass 3 for the feedback scope expansion). 6 Foundation, 4 Core, 14 Amplifier, 0 Deprioritized. 2 cards remain blocked on source (C2, C3 — visual direction). 22 cards draftable from existing source.

Note: "6 Foundation" only if no new Foundation card is added, which is Conan's recommendation. If the human disagrees and wants a dedicated "Two Intake Types Forever" Foundation Principle, this becomes 7 Foundation / 24 cards.

### New Card Entries

Add these rows to the Amplifier tier table:

| # | Card | Type | Area | Zone | Build Order | Source Status | Intent |
|---|------|------|------|------|-------------|---------------|--------|
| A15 | `Agent - Feedback Submitter` | Agent | 1.3 | `/product/` | 25 | EXISTS (feedback-flow.md) | Persona card, parallel to A1 Reporter. Identity model: optional email + server-logged IP + Fabro version metadata. No post-submission interaction. |
| A16 | `Component - Feedback` | Component | 2.3 | `/product/` | 26 | EXISTS | First-class entity, parallel to A3 Report. UUIDv7 ID, message text, file attachments manifest, optional email, IP, Fabro version/platform/arch, timestamp, retention deadline. |
| A17 | `System - Feedback Storage & Notification` | System | 2.4 | `/product/` | 27 | EXISTS | Parallel to A5 for Debug Bundles. Covers: feedback.json + feedback.txt S3 layout under `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`, Discord webhook posting with component-v2 payload shape, failure self-reporting. Absorbs the Discord-webhook rationale inline. |
| A18 | `Decision - IP Logging for Abuse Protection` | Decision | 5.1 | `/rationale/` | 28 | EXISTS | First identity-data capture in the library. Why IPs are logged, scope of use (abuse protection + basic attribution only — no analytics, no cross-submission tracking), retention (same 30 days). |
| A19 | `Decision - Fire-and-Forget Feedback` | Decision | 5.1 | `/rationale/` | 29 | EXISTS | Why feedback returns no URL, has no delete, and has no capability-URL model. Pairs with A9 (capability URLs for Debug Bundles) as the inverse decision for the other flow. |

### Card Type Mapping Row Additions

Add these to the area → type mapping table:

| Area | Type in this library | Reasoning |
|---|---|---|
| 1.3 Personas / JTBD | Agent (×3) | Reporter + Triager + Feedback Submitter. Triager is one role; Reporter vs Feedback Submitter is the asymmetric-identity split. |
| 2.3 Product Entities | Component (×3) | Report, Debug Bundle, Report Metadata Page, Feedback. Note: manifest says ×3 today; update to ×4 when adding A16. |
| 2.4 System Design | System (×3) | A5 (Debug Bundle storage) + A14 (Service Topology) + A17 (Feedback Storage & Notification). Three systems reflects the real separation of concerns. |
| 5.1 Key Decisions Log | Decision (×7) | A9, A10, A11, A12, A13, A18, A19. |

(The current manifest shows "Component (×3)" for area 2.3 — update to ×4. Current "System (×2)" → ×3. Current "Decision (×5)" for 5.1 → ×7.)

### Rename in Existing Card Rows

- A5 filename: `System - Retention, Storage, and Key Scheme` → `System - Debug Bundle Retention, Storage, and Key Scheme`. Update the row accordingly. Update every inbound wikilink across the library (cascade — this is a small cascade).

### New "Accepted Lint Warnings" Adjudications Anticipated

- `sweep4.missing_containment` on `Component - Feedback` (A16) — same rationale as A3/A4; this library uses F1 as the de facto container for all product-layer cards. Pre-adjudicate in the manifest rather than re-surfacing during spot-check.
- `sweep2.word_count` on new cards — likely, per library-wide precedent. Pre-accepted.

### New Open Flags

Add to the Open Flags section:
- **6. Discord webhook Decision absorbed into A17.** Conan recommends no standalone `Decision - Discord Webhook for Feedback Notification` card; the rationale lives inside `System - Feedback Storage & Notification`. If the human wants it as a standalone Decision, promote to A20.
- **7. F6 identity-line rewrite.** The "narrow forever" identity sentence in F6 changes shape, not just prose. Flagged so the spot-check verifies the rewrite didn't narrow or broaden the commitment unintentionally.

### Build Sequence Table

Add a "Inventory Pass 3 Follow-On (2026-04-23)" section after the existing "Follow-On Build Sequence (2026-04-23)" section:

| Order | Card | Depends On |
|---|------|------------|
| 25 | Phase 1 cascade (F1, F3, F4, F5, F6) — identity edits | Existing F1/F3/F4/F5/F6 cards ratified |
| 26 | Phase 1 cascade (F2) — add nouns + CLI row | After identity edits |
| 27 | Phase 2 cascade (A1, A3, A5 rename, A9, A11, A13 — tiny updates + A5 rename) | After Foundation cascade |
| 28 | Phase 2 cascade (A2, A6, A7, A8, A14 — medium updates) | After tiny updates |
| 29 | Phase 3 — A15 `Agent - Feedback Submitter` | F2 updated |
| 30 | Phase 3 — A16 `Component - Feedback` | F2 updated; A15 drafted |
| 31 | Phase 3 — A17 `System - Feedback Storage & Notification` | A16 drafted; A14 updated |
| 32 | Phase 3 — A18 `Decision - IP Logging for Abuse Protection` | A17 drafted |
| 33 | Phase 3 — A19 `Decision - Fire-and-Forget Feedback` | A16 drafted; A9 updated |
| 34 | Phase 4 — C1 tiny update (bun feedback template note) | Independent; pairs anywhere |

---

## Part 5 — Sam's Work Phases

The scope change is large enough to phase. Conan recommends **three Sam dispatches**, not four — Phase 4 from the brief's sketch collapses into Phase 2 because updating A2 is not structurally different from updating A7 or A14 (they're all "add a parallel section to reflect the second flow"). Phases are ordered for cascade-safety: identity first so new cards inherit the correct frame; existing cards cascade before new cards so new cards link into a stable graph.

### Phase 1 — Foundation Identity Cascade

**Goal:** Land the new two-intake-types-forever identity everywhere in the Foundation layer before any new cards or product-layer retrofits touch it.

**Scope:** F1, F2, F3, F4, F5, F6 + C1 (tiny) + C4 (no-op, confirm only).

**Key edits:**
- F1: broaden pickup-point framing; parallel flow description; Discord-as-push-not-triage-surface sentence.
- F2: add Feedback + Feedback Submitter noun entries; add `fabro feedback` row to CLI Commands table.
- F3: Sentry-distinction-pattern rewrite of the "feedback collection" drift bullet; scope-drift-ceiling note.
- F4: one-paragraph addition extending calm-tone policy to feedback confirmation + Discord post format.
- F5: one-paragraph addition naming Feedback as the second mechanism serving the same Strategic WHY.
- F6: identity-line rewrite to "exactly two intake types, forever"; out-of-scope list framing retrofit; 2026-04-23 confirmed-expansion log entry.
- C1: one sentence noting `bun feedback` as template shape for `fabro feedback`.
- C4: verify no change needed (audience unchanged).

**Handoff to Conan:** Spot-check Phase 1 before Phase 2 starts. This is the identity-change pass — worth verifying the commitment's shape matches the product owner's confirmation before any new cards inherit it.

**Estimated effort:** 90–120 minutes for Sam.

### Phase 2 — Product-Layer Cascade (Existing Cards)

**Goal:** Retrofit every existing product-layer card that needs to reflect the second flow, **before** new cards are added. This keeps the graph internally consistent at every intermediate commit.

**Scope:** A1 (tiny), A2 (medium — second behavioral section), A3 (tiny), A5 (rename + scope note — triggers wikilink cascade), A6 (small), A7 (medium — parallel CLI section), A8 (tiny), A9 (tiny), A11 (tiny), A13 (tiny), A14 (small — feedback endpoint + Discord dependency).

**Key edits:**
- A5 rename: `System - Retention, Storage, and Key Scheme` → `System - Debug Bundle Retention, Storage, and Key Scheme`. Update all inbound wikilinks. This is the largest mechanical piece.
- A2 second behavioral section: the Triager's feedback workflow (Discord → optional S3). Explicit note that feedback does not use the S3-direct-with-own-credentials primary path — Discord is the primary surface for feedback.
- A7 parallel CLI section: `fabro feedback` shape, `--email` flag, confirmation output, no-URL, no-delete.
- All the "tiny" updates are 1–3 sentences each; group into one pass.

**Handoff to Conan:** Spot-check Phase 2 before Phase 3 starts. Verify the cascade didn't miss any inbound links to A5 after rename, and that disambiguation notes (A1↔A15, A3↔A16) are consistent.

**Estimated effort:** 90–120 minutes for Sam.

### Phase 3 — New Card Creation

**Goal:** Add the 5 new cards. Graph is stable from Phase 1+2 — new cards link into a library that already carries the two-intake-types-forever frame.

**Scope:** A15, A16, A17, A18, A19 — in that order (build order 29–33 per the updated manifest).

**Key edits:**
- A15 `Agent - Feedback Submitter`: persona card. Parallel shape to A1. Identity model explicit: optional email + server-logged IP + Fabro version metadata. No post-submission interaction.
- A16 `Component - Feedback`: first-class entity. Parallel shape to A3. UUIDv7 ID, message text, attachments manifest, optional email, IP, version/platform/arch, timestamp, retention deadline.
- A17 `System - Feedback Storage & Notification`: parallel to A5. S3 layout, Discord webhook posting, failure self-reporting. Absorbs the Discord-webhook rationale inline (why Discord, what gets posted, how failures self-report).
- A18 `Decision - IP Logging for Abuse Protection`: thin Decision. Why, scope of use, what it's not used for, retention.
- A19 `Decision - Fire-and-Forget Feedback`: thin Decision. Why no URL, why no delete, why no capability-URL model. Links to A9 as the inverse-shape decision for Debug Bundles.

**Handoff to Conan:** Full spot-check of all 5 new cards. This is the pass where novel cards land in the library; standards apply harder than on existing-card retrofits.

**Estimated effort:** 2–3 hours for Sam across all 5 cards.

### Phase-Boundary Rationale

Why three phases instead of four:
- A2 ("second behavioral section") is mechanically identical to A7 and A14 ("add a parallel section"). Splitting it into its own phase creates phase overhead without a cascade-safety benefit.
- Identity must land before product layer (Phase 1 → Phase 2).
- Existing cascade must land before new cards (Phase 2 → Phase 3) so new cards link into a stable, internally consistent graph.

Why not one phase:
- Identity edits are the hardest to get right and most expensive to undo. Worth a spot-check before proceeding.
- Phase 2's rename cascade (A5 → `System - Debug Bundle Retention, Storage, and Key Scheme`) touches many wikilinks; verification before new cards are added prevents broken links at commit boundaries.

---

## Prioritized Next-Action List for Sam

1. **Phase 1 dispatch — Foundation identity cascade.** Edit F1, F2, F3, F4, F5, F6 per Part 3 / Part 5. Also apply C1 tiny update and confirm C4 is no-op.
2. **Return to Conan for Phase 1 spot-check.** Verify the identity-change shape is correct before proceeding.
3. **Phase 2 dispatch — Product-layer cascade.** 11 card updates + the A5 rename + wikilink sweep.
4. **Return to Conan for Phase 2 spot-check.** Verify the rename cascade caught all inbound links; verify disambiguation notes are consistent.
5. **Phase 3 dispatch — 5 new cards (A15, A16, A17, A18, A19) in build order.**
6. **Return to Conan for Phase 3 full spot-check.** Novel-card standards.
7. **(Parallel / optional, any time)** Bryan provides visual direction to unblock C2/C3. Not on this critical path.

---

## Manifest Edits I Want Applied

Applied by: **Sam, as the first action of Phase 1**, so subsequent phases reference the updated manifest. Conan confirms edits during the Phase 1 spot-check.

1. Update header: "Last incremental update: 2026-04-23 (Inventory Pass 3 — feedback scope expansion; five new cards A15–A19 manifested; identity cascade for two intake types; A5 renamed — see `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md`)"
2. Add source-inputs entry: `docs/alexandria/sources/feedback-flow.md` (feeds A15, A16, A17, A18, A19, cascade to F1/F2/F3/F5/F6/A2/A7/A14).
3. Update Summary line from "19 cards" to "24 cards" per Part 4.
4. Update Zone & Type Conventions table: 1.3 Agent (×3), 2.3 Component (×4), 2.4 System (×3), 5.1 Decision (×7).
5. Append Amplifier tier rows A15–A19 per Part 4.
6. Rename A5 filename/row to `System - Debug Bundle Retention, Storage, and Key Scheme`.
7. Append "Inventory Pass 3 Follow-On (2026-04-23)" sub-section with build orders 25–34 per Part 4.
8. Add pre-adjudicated "Accepted Lint Warnings" entries per Part 4 (A16 containment + word count).
9. Add Open Flags 6 and 7 per Part 4.

---

**Status: DONE**

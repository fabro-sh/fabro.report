# Spot-Check: Batch C1 / A3 / A4 / A6

**Job:** Conan Job 2.5 — Spot-Check (not full grading)
**Date:** 2026-04-22
**Scope:** 4 new cards + 9 backlink edits across 8 existing cards
**Reviewer:** Conan
**Config:** Factory × Low × Low (22-area pool)
**Lint state (human-verified):** `cards` 0/20 (word-count only); `graph` 0/0/0; `lines` 0/0/0; `layers` 3 warnings (1 empty `experience/`, 2 containment on A3/A4).

---

## Verdict at a Glance

| Card | Type | Scope fit | Citations | Vocabulary | Grade proxy |
|---|---|---|---|---|---|
| C1 `Standard - Competitive Context` | Standard | Stays one-pager, closes survey | OK (header-vs-content pattern persists) | Clean | B+/A- |
| A3 `Component - Report` | Component | Complete; honors A9/A10/A11 | OK | Clean | A- |
| A4 `Component - Report Metadata Page` | Component | Structural-only; defers visual | OK | Clean (one soft drift: see §5) | A- |
| A6 `Standard - Information Architecture` | Standard | Positive framing of absence holds | OK | Clean | A- |

No blockers. Two small fixes + two convention calls. Backlink audit clean.

---

## 1. C1 — "One-Pager Closing the Survey" Check (Task Point 1)

**Question:** Does C1 close Open Flag 3 as a one-pager, or does it expand into a broader survey?

**Finding:** It closes. This is a one-pager in the exact shape recommended.

Evidence:
- The WHAT explicitly names itself a one-pager and explicitly states "the purpose of this card is to *close* the competitive-analysis question for this library, not to open it" (L5).
- The WHEN section cites "Open Flag 3" by name and cites Conan's recommendation ("skip the paste-services / support-bundle-uploaders survey") (L30). This is the exact close-this-flag move.
- The "Adjacent Categories (Named To Close, Not To Survey)" section (L50–L59) explicitly does **not** survey paste services / support-bundle uploaders / file-sharing / crash-report products. Each gets a single bullet of "share DNA + not-borrowed list" — not four individual competitive write-ups. That is the correct discipline.
- The HOW/Test section (L63) hands an agent a mechanical yes/no question ("is the feature in the intake URL shape?") rather than a survey-comparison framework. That is close-the-question shape, not open-the-survey shape.

**No scope creep detected.** The card earns its tier-2 Core placement by being the explicit foreclosure of an area that would otherwise recur as drift.

Minor note: Section "What Is Not Borrowed From `bun.report`" (L38–L48) partially overlaps with F3's drift list. C1 frames this overlap explicitly ("this list mirrors and anchors [[Principle - Reject Like bun.report Drift]]'s drift examples at the competitive-analysis layer" — L40). Mirroring is intentional and the framing calls it out; not a defect.

---

## 2. A3 — Completeness Check (Task Point 2)

**Checklist:**

| Required element | Present? | Where |
|---|---|---|
| Authoritative field list — ID | Yes | HOW L49 |
| URL | Yes | HOW L50 |
| Upload timestamp | Yes | HOW L51 |
| Bundle size | Yes | HOW L52 |
| Optional reporter message | Yes | HOW L53 |
| Fabro version | Yes | HOW L54 |
| Arbitrary key/value metadata tags | Yes | HOW L55 |
| Retention deadline | Yes | HOW L56 |
| Pointer to Debug Bundle in S3 | Yes | WHAT L5, WHERE L17 (explicit "reference to a Debug Bundle… opaque-tarball payload in S3") |
| Lifecycle — created | Yes | HOW L62 |
| Lifecycle — visited | Yes | HOW L63 |
| Lifecycle — deletable | Yes | HOW L64 |
| Lifecycle — auto-expires | Yes | HOW L65 |
| Not-the-Debug-Bundle distinction | Yes | WHY L37 ("the load-bearing coupling is *the Report is not the Debug Bundle*"); Key Couplings L71; Anti-Example L77 |
| No mutable state | Yes | HOW L67 ("No modification after creation is possible"); Anti-Example L78 |
| No content hash (ties to A10) | Yes | WHERE L16 (cites A10 explicitly); Anti-Example L80 |
| Access semantics inherit from A9 | Yes | WHERE L12; Key Couplings L72; cites A9 three times |
| Retention inherits from A11 | Yes | WHERE L13; HOW L56; cites A11 three times |

**Finding:** Complete. Every required element is present and the A9/A10/A11 inheritance is explicit rather than implied.

The load-bearing statement "the Report is not the Debug Bundle" is foregrounded in WHY (L37) and reinforced in Key Couplings (L71). Correct emphasis.

The anti-example at L81 correctly forwards the "only four elements on the Metadata Page" commitment to A4, preventing this Component card from silently growing the Page's visible-field list.

**One nuance worth naming (not a fix):** A3 says the Report "is otherwise immutable: there is no modification operation after creation, only deletion" (L5). Strictly speaking, deletion *is* a state transition — the Report is terminal-deleted. A3 handles this in HOW ("Deletion is terminal — there is no undelete, no archive" L64) so the immutability claim is consistent in context, but a future reader could catch it as a micro-contradiction. Leave it; fixing reduces clarity.

---

## 3. A4 — Structural-Only Scope (Task Point 3)

**Checklist:**

| Required element | Present? | Where |
|---|---|---|
| Four elements: Report ID | Yes | HOW L54 |
| Upload timestamp | Yes | HOW L55 |
| Bundle size | Yes | HOW L56 |
| Delete button | Yes | HOW L57 |
| Access semantics inherit from A9 | Yes | WHERE L15; HOW L74–L81; cites A9 six times |
| Expiration behavior inherits from A11 | Yes | WHERE L17; HOW "Expired-Report Behavior" L83–L87 |
| Tone register inherits from F4 | Yes | WHERE L19; HOW "Tone Inheritance" L89–L98; cites F4 three times |
| No visual invention | **Verified** — see below |

**Visual-invention check:**
- The WHAT paragraph explicitly names the card as "a **structural** Component card, not a visual one" (L7) and defers visual direction to the blocked C2.
- HOW L52 says the four elements render "in the visual arrangement specified by `Standard - Design System` (blocked)."
- Timestamp L56: "Displayed in a format the Reporter can read (exact format deferred to the Design System card)." Good.
- Bundle size L56: "Displayed in a human-readable unit (exact format deferred to the Design System card)." Good.
- Anti-Example L108: "Adding a visual direction ('let's make it dark-mode and monospace') to this card rather than to `Standard - Design System`." Good — names the anti-pattern of the card drifting into C2's territory.
- Section "Deferred to Other Cards" (L110–L117) is exhaustive: visual direction → C2, delete flow → A7, accessibility → A8, URL shape → A5. Each deferral is explicit with the blocker named.

**No visual invention detected.** Sam was disciplined about this.

The "What the Page Does Not Show" section (L62–L71) is load-bearing and correctly enumerates the `bun.report`-adjacent shapes the page rejects (contents, Reporter identity, Triager data, recent-uploads, feedback, symbolicated stack). This is the right place for that enumeration.

---

## 4. A6 — Positive Framing of Absence (Task Point 4)

**Question:** Does A6 name explicit absences and frame each as deliberate rather than omitted?

**Finding:** Yes, and this is the card's structural center of gravity.

Evidence of deliberate-absence framing:
- WHAT L5: "This Standard is captured here specifically so it *stays* minimal: an IA that is not written down invites future agents to invent navigation…" — frames absence as commitment, not gap.
- WHY L33–L35: "The reason this Standard exists at all, then, is to *foreclose* IA growth by stating the IA shape explicitly and naming the categories of IA the product does not have." — explicit meta-statement of the positive-absence frame.
- HOW "What the IA Is Not" (L96–L105) — seven explicit absences, each with a named rationale:

| Absence named | Why absent (framed positively) |
|---|---|
| No user account pages | "Reporters have no identity on the service" |
| No team dashboards | "Triagers retrieve from S3 directly; the service has no team-facing IA" |
| No "my uploads" view | "URL possession is the only handle; lost URLs are not recoverable, by design" |
| No browse endpoint | "Not for Reporters (who have no identity to scope by), not for Triagers (who use S3 directly)" |
| No search | No justification at this item — inherits from the section-level rationale |
| No multi-page site | "The Report Metadata Page is the *only* rendered page" |
| No documentation surface | "Fabro's documentation lives in the Fabro repo or wherever Fabro publishes docs" |

- HOW L78–L80 covers the explicitly-asked-for absences ("no dashboard, no search, no listing, no navigation, no home page with recent Reports feed") across multiple sections; no home-page-with-recent-Reports claim appears as a future-work item anywhere.
- HOW "HTTP Surface" (L68–L74) also lists endpoint-level absences: no listing endpoint, no browse, no account, no second rendered page, no health/status.

**Positive-framing holds end-to-end.** Every absence is either named as a commitment ("by design," "permanently," "forever") or is tied back to an upstream Decision/Principle that makes the absence coherent. There is no "we don't have this yet" phrasing anywhere.

One small nuance: "No search" (L103) is the one absence that does not individually name its rationale. It is covered by the section-level framing, but a single-line "because there is no content surface to search over" would make the symmetry perfect. Not a fix — cosmetic.

---

## 5. Vocabulary Compliance (Task Point 5)

Scanned all four cards for canonical-name drift per F2 and F3-1-shape rules.

| Card | "Report Metadata Page" shortened? | `bun.report` vocabulary imported? | Other noun drift? |
|---|---|---|---|
| C1 | No | No (uses "Crash Report," "Stack Trace," "Feedback Form" only in explicit drift-list contexts) | None |
| A3 | No | No | None |
| A4 | No — always fully spelled out in running prose | No (uses "dashboard" only in explicit anti-example) | **One soft note:** WHAT L5 uses "landing surface" as a synonym for the page. Not a canonical-name violation (the card then immediately uses the canonical name), but "landing" is one of the shapes F2 specifically warns against. See below. |
| A6 | No | No | None |

**A4 "landing surface" note:** L5 opens with "The **Report Metadata Page** is the one-page landing surface served at a Report's URL." F2 L58 prohibits shortening to "Landing Page" — A4 uses "landing surface," which is not literally on F2's prohibited-shortening list, and A4 never uses "landing" as a standalone synonym. It is a descriptor phrase ("landing surface" = "the surface something lands at"), not a renaming. Call this acceptable but worth flagging: if Sam tweaks A4 for any other reason, tightening "landing surface" to "one-page surface" or "one rendered surface" would remove the soft drift vector. **Not a blocker; optional polish.**

F3-1-shape drift (the "page card is where vocabulary drift first shows up" risk): **not present.** A4 is disciplined.

---

## 6. Source Citations (Task Point 6)

Per-card spot-check of 1–2 citations each:

### C1
- `product-vision.md` L33–L48 "What `fabro.report` Is Not" — L33 is the heading "## What `fabro.report` Is Not (load-bearing)"; content is L35–L48. Header-vs-content pattern persists but the claim holds (the section is genuinely that range). Acceptable by the A1/A2 high-water-mark (they did the same thing).
- `product-vision.md` L102–L107 "Template Relationship" — L102 is heading, L103–L107 are the bullets. Same pattern. Content-accurate.
- Manifest citation "Open Flag 3" — Verified present in `card-manifest.md` L179. Accurate.

### A3
- `product-vision.md` L82–L88 metadata captured — L82 is bullet header "Metadata captured at upload (outside the bundle):", L83–L85 are the three metadata items. L86–L88 cover Dedup, Auth, Bundle content rendering — those are adjacent Confirmed Decisions but not the "metadata captured" content per se. **Mild over-citation:** the cited range extends two lines past the metadata content into unrelated Decisions. Not a correctness defect (those lines are all Confirmed Decisions and all feed A3's field semantics) but the phrasing "L82–L88 Confirmed Product Decisions → metadata captured at upload" implies a tighter match than exists. Acceptable; consider tightening to "L82–L85 (metadata items); L86 (dedup); L87 (auth); L88 (no content rendering)" on any future edit.
- `product-vision.md` L96 "Confirmed Entities → Report" — L96 is "- **Report** — the upload event…". Accurate.

### A4
- `product-vision.md` L31 receipt page description — L31 verified as the exact sentence quoted in A4 ("a minimal one-page metadata receipt showing the Report ID, upload timestamp, and bundle size, plus a delete button…"). Accurate.
- `product-vision.md` L100 "Report Metadata Page" in Confirmed Entities — L100 verified. Accurate.

### A6
- `product-vision.md` L20–L32 end-to-end flow — L20 is the opening prose of "What `fabro.report` Is"; L22–L29 are the six-step flow; L31 is the receipt-page sentence. Range is slightly wider than strictly necessary but covers what's cited.
- `product-vision.md` L111–L113 Parent Product Context — L111 is the Fabro bullet, L112 is the fabro debug bullet, L113 is the "`fabro report` is new" bullet. A6 cites this range for "`fabro report` as new Fabro subcommand" and uses the framing at HOW L54. Accurate.

**Source-citation grade across batch:** matches A1/A2's high-water-mark. The header-vs-content pattern recurs but is consistent with the library's established convention. A3's metadata-range over-citation is the one callable item, and it is minor.

---

## 7. Containment-Warning Convention (Task Point 7)

Two new `sweep4.missing_containment` warnings on A3 and A4. No Section/Template/Governance card exists in the library.

### Conan's Recommendation: **Path B — Accept the warnings permanently; document the acceptance.**

**Why:**

- **Path A (add a Section card) overshoots for this library's scale.** A Section card to contain two Components in a 16-card library is structurally load-bearing only if it carries meaningful content — and the content available (an `fabro.report` intake service description) duplicates F1 (the Product Thesis), which already operates as the de facto container for every product-layer card. A Section card here would be either (i) a thin wikilink stub whose only job is to satisfy the linter — which is bureaucracy, not architecture — or (ii) a duplicate of F1's framing under a different type label. Neither earns its place.
- **Path C (defer to A5) is backwards.** A5 is a System card, not a containment card in the taxonomy sense. Even if A5's structure makes A3/A4 look connected, the linter's containment rule is about Section/Template/Governance parenting of Components, not about structural adjacency to a sibling System. Waiting for A5 will not resolve the warning.
- **Path B fits the library's scale.** This is a 16-card intake-service library. Two containment warnings on two Components is a structural artifact of the product shape (the product *is* the intake service; there is no enclosing "service" concept above it that is not already F1). Naming this in the manifest's conventions section so the warnings are known-and-accepted rather than re-surfaced each build is the correct discipline.

**Concrete action for Path B:** Add a new subsection to `card-manifest.md` — something like "## Accepted Lint Warnings" with a row for the two `sweep4.missing_containment` warnings on A3 and A4, citing this spot-check as the adjudication. That is a Sam-executed manifest edit, not a card change. Small blast radius; no cascade.

**If Bryan disagrees and prefers Path A**, the Section card would be `Section - fabro.report Intake Service` under `/product/sections/`, and A3/A4 would gain a "contained by" wikilink. Blast radius: 1 new card + 2 card edits + 1 manifest edit. Not large, but also not free. Conan still recommends Path B.

---

## 8. C1 Filename Convention (Task Point 8)

**File:** `Standard - Competitive Context.md`
**Manifest row header:** `Standard - Competitive Context: What to Borrow, What to Skip`
**All 16 prior wikilinks:** `[[Standard - Competitive Context]]` (short form)

**Conan's call: Agree with Sam. Keep the short form.**

Rationale:
- The short form is what the 16 existing wikilinks resolve to. The long form would require a rename + 16 edits + any future wikilinks that have already been drafted in the backlinks Sam just closed. Blast radius is real and the benefit is low.
- The long form's colon + comma (`: What to Borrow, What to Skip`) is a filename-unfriendly pattern on case-insensitive filesystems and in cross-tool rendering. The short form is cleaner.
- The manifest's long form is a descriptive subtitle, not a rename. `card-manifest.md` treating a card's manifest row title as a descriptive handle while the card's actual filename is the canonical identifier is consistent with how F1's manifest row handles F6 ("Principle - Narrow Forever (Roadmap as Commitment)" — file is `Principle - Narrow Forever.md`). Sam's judgment matches an established pattern.
- **Small editorial consistency fix:** Update the manifest row for C1 to read `Standard - Competitive Context` (short form) as the canonical card reference, optionally retaining "What to Borrow, What to Skip" as a description in the Intent column. This removes the appearance of drift between manifest and filename.

---

## 9. Backlink Audit (Task Point 9)

Nine edits across eight cards. Audit:

### F1 — Removed stale forward-ref to `Domain - fabro.report Intake Service`

Verified: F1's current ecosystem section no longer references `Domain - fabro.report Intake Service`. F1 now lists (current state): Standards (F2), Anti-Patterns (F3), Strategy + Scope (F5, F6), Tonal (F4), and a full set of Decision + Agent + Component + Standard referenced-by backlinks.

**Gap check:** Did the removal of the stale `Domain` ref create a conceptual coverage gap in F1?
- F1 does not otherwise claim a "Domain" parent anywhere in its current content.
- F1's WHY (L36–L40) and HOW (L52–L72) cover the pickup-point framing, the Validation Criteria, and the Decision Heuristic without needing a Domain handle.
- F1's Ecosystem WHERE section is now complete (covers vocabulary, anti-patterns, strategy, scope, tone, decisions, agents, components, standards, and template reference).
- **No gap detected.** The `Domain - fabro.report Intake Service` card never existed; the removed forward-ref was stale from an earlier manifest draft that was never realized. Removing it is a correctness fix, not a coverage loss.

### F4 — Upgraded two forward-refs to wikilinks for A4

Verified: F4 line 30 now contains `[[Component - Report Metadata Page]]` as an explicit referenced-by backlink (previously this was a plain-text forward reference per the prior manifest). The anchor prose ("Must carry the same register as the CLI. Not a starker aesthetic, not a warmer one. Visual direction for the page belongs in a Core-tier card (manifest C2), but tone consistency with this Principle is non-negotiable.") is correct against A4's **structural** scope — F4 is pairing tonally with A4's text-string surface, not prescribing visuals.

**Scope-match check:** F4's backlink anchor does not claim A4 carries any visual commitment. A4 is structural-only; F4 cites it for "every user-facing string on that page — field labels, delete button copy, delete confirmation flow, any state copy during the delete action — inherits this Principle's register" (L30). That is exactly the text-string surface, not the visual surface. Clean.

A4 → F4 backlink is also clean: A4's WHAT (L5) cites F4 as "efficient and reassured, calibrated against `bun`" and WHERE "Tone" (L19) cites F4 as the register-inheritance source. Bidirectional and correctly scoped.

### F3 — Added backlinks to all four new cards

Each must tie to a specific drift shape F3 enumerates, not generic pairing.

| New card | F3's backlink prose | Specific drift shape F3 enumerates | Match? |
|---|---|---|---|
| C1 `Standard - Competitive Context` | "the competitive-analysis one-pager that mirrors this Principle's drift list at the competitor-survey layer; the not-borrowed list in C1 is the positive-frame anchor of F3's negative-frame enumeration" (F3 L25) | F3's full Drift Examples list (L48–L56) | Yes — C1 mirrors F3's drift items at the competitor-survey layer |
| A3 `Component - Report` | "field enrichment on the Report (investigation status, symbolication state, triage assignee, content hash) is the drift shape this Principle rejects at the data-model layer" (F3 L23) | F3 L55 "per-team scoping, user accounts, or authentication" + F3 L49 "symbolication" | Yes — enumerates data-model-layer drift shapes |
| A4 `Component - Report Metadata Page` | "the single most named drift shape in this Principle is page enrichment (bundle-contents tab, symbolicated stack render, feedback prompt, team-assignment status)" (F3 L24) | F3 L48 "make the Report Metadata Page more useful" — the paradigmatic drift | Yes — correctly names A4 as the defense against F3's most-named drift |
| A6 `Standard - Information Architecture` | "IA growth (dashboards, listings, filters, nav, second rendered pages) is a specific drift vector this Principle enumerates; the IA Standard is the upstream minimal-IA commitment that closes the 'add a route' escape hatch feature by feature" (F3 L26) | F3 L53 "team-facing dashboard" + L54 "per-team scoping" + L56 implicit | Yes — IA-layer drift enumerated explicitly |

**All four F3 backlinks are tied to specific enumerated drift shapes, not generic "this Principle applies."** Clean audit.

### F2, F5, F6, C4, A9 — Other backlink closures

- **F2 (Noun Vocabulary)** → closes backlinks to C1, A3, A4, A6. Each cites F2 as "authoritative name source." Verified at F2 L22–L24. Clean.
- **F5 (Strategy)** → closes backlink to C1 at F5 L34 ("the one-pager closes the competitive-analysis question because no competitive-differentiation work is needed when this Strategy is the entire purpose"). Specific, not generic. Clean.
- **F6 (Narrow Forever)** → closes backlinks to C1, A3, A4, A6 (L26–L29). Each tied to a specific scope shape: C1 to "re-surveying competitors," A3 to "Report-field expansion," A4 to "page-content expansion," A6 to "IA growth." Clean.
- **C4 (Who This Service Is For)** → closes backlink to C1 at L26 ("adjacent products serving different populations… are structurally unhelpful comparisons"). Specific. Clean.
- **A9 (Capability URLs)** → closes backlinks to C1 and A6 at L27 and L28. C1 backlink ties to "enumerates per-user accounts / per-team scoping as features explicitly not borrowed"; A6 backlink ties to "the IA Standard names URL possession as coextensive with the product's access model." Specific. Clean.

**No generic pairings, no smuggled claims, no scope creep.** Sam's backlink closures are correctly scoped.

---

## 10. Small Findings & Optional Polish

Captured for the record. None are blockers.

| # | Card | Finding | Recommended action |
|---|---|---|---|
| 1 | A3 | Citation range L82–L88 is slightly wider than the specific "metadata captured" content (L82–L85) | Tighten on any future edit; leave for now |
| 2 | A4 | "landing surface" (L5) as descriptor — soft proximity to F2's prohibited "Landing Page" shortening | Optional tighten to "one-page surface"; not a fix |
| 3 | A6 | "No search" absence (L103) does not individually name its rationale | Optional one-line addition; cosmetic |
| 4 | Manifest | C1 row header uses long form while filename + 16 wikilinks use short form | Editorial: align manifest row header to short form |
| 5 | Manifest | Two `sweep4.missing_containment` warnings not yet documented as accepted | Add "Accepted Lint Warnings" subsection per §7 Path B |

---

## 11. What the Library State Is Now

All EXISTS-source cards are drafted. The 2026-04-22 build has produced:

- **Foundation (5/5):** F1, F2, F3, F4 (unblocked via tone source), F5, F6 — all ratified.
- **Core (3/4):** C1 (this batch), C4 — ratified. C2 blocked on visual direction. C3 blocked on same.
- **Amplifier (7/11):** A1, A2, A3 (this batch), A4 (this batch), A6 (this batch), A9, A10, A11 — ratified. A5, A7, A8 blocked on product-owner source (retention mechanism / CLI UX / accessibility floor).

**Remaining work is blocked on Bryan's source input:**

| Blocked card | Needs |
|---|---|
| C2 `Standard - Design System` | Visual direction (typography, color, layout, chrome) |
| C3 `Standard - Receipt Page Sketch` | Paired with C2 — sketch + caption |
| A5 `System - Retention, Storage, and Key Scheme` | Retention mechanism (S3 lifecycle vs cron); key-scheme decision |
| A7 `Standard - Interaction Patterns (CLI + Delete)` | CLI UX specifics (progress, errors); delete confirmation flow |
| A8 `Standard - Accessibility Policy` | Accessibility floor (WCAG AA + NO_COLOR, or alternative) |

**This batch closes the draftable-from-current-source work.** Next action is source capture from Bryan, not further card drafting.

---

## Verdict

**APPROVED_WITH_FIXES.**

The four cards pass spot-check on structure, scope, vocabulary, and citation. The fixes are small:

1. **Required:** Document the two `sweep4.missing_containment` warnings as accepted in `card-manifest.md` (Path B per §7). Small Sam edit.
2. **Required:** Align C1's manifest row header to the short filename form (per §8). Small Sam edit.
3. **Optional (polish):** A3 citation tightening; A4 "landing surface" tightening; A6 "no search" one-liner.

No card content changes required. The four new cards and all nine backlink edits are structurally sound and do not smuggle claims.

Library state: all EXISTS-source cards drafted; remaining work (C2, C3, A5, A7, A8) is blocked on Bryan's source input for visual direction, CLI UX, retention mechanism, and accessibility floor.

**Status: APPROVED_WITH_FIXES**

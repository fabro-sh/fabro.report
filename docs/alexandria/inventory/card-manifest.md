# Alexandria Card Manifest — `fabro.report`

**Job:** Conan Job 1 — Inventory
**Date:** 2026-04-22
**Last incremental update:** 2026-04-23 (Inventory Pass 3 — feedback scope expansion; five new cards A15–A19; identity cascade; A5 renamed — see `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md`)
**Config:** Factory × Low Novelty × Low Complexity — 22-area pool (5 Foundation / 4 Core / 9 Amplifier / 4 Deprioritized)
**Source inputs:**
- `docs/alexandria/alexandria-config.json`
- `docs/alexandria/initialize-output.md`
- `docs/alexandria/sources/product-vision.md` (updated — strategic WHY and "narrow forever" both confirmed; CLI command rename confirmed 2026-04-23)
- `docs/alexandria/sources/tone-and-emotional-goals.md` (feeds F4)
- `docs/alexandria/sources/tech-stack-and-deployment.md` (feeds A12, A13, A14)
- `docs/alexandria/sources/storage-and-keys.md` (feeds A5)
- `docs/alexandria/sources/cli-interaction-patterns.md` (feeds A7)
- `docs/alexandria/sources/accessibility.md` (feeds A8)
- `docs/alexandria/sources/feedback-flow.md` (feeds A15–A19 + cascade to F1/F2/F3/F5/F6/A2/A7/A14)
- `docs/alexandria/reviews/product-vision-source-assessment.md` (prior candidate list, 14 cards)

**Summary:** 24 cards to build (19 prior + 5 added 2026-04-23 in Inventory Pass 3 for the feedback scope expansion). 6 Foundation, 4 Core, 14 Amplifier, 0 Deprioritized (all 4 Deprioritized areas omitted as not-needed). 2 cards remain blocked on source (C2, C3 — visual direction). 22 cards draftable from existing source.

---

## Zone & Type Conventions

Zones used (proposals — flag if project diverges):
- `/rationale/` — WHY layer: Product Thesis, Principle, Standard, Anti-Pattern, Decision
- `/product/` — product layer: Domain, Component, Agent, System
- `/experience/` — experience-over-time: Journey, Experience Goal (not used here; one journey is Deprioritized, Experience Goals live as a Principle in /rationale/ for a service this small)

Card type mapping to areas:

| Area | Type in this library | Reasoning |
|---|---|---|
| 1.1 Product Vision | Product Thesis | The anchor WHY. |
| 1.2 Product Strategy | Principle | Strategy for a service this narrow is one Principle, not a suite. |
| 1.3 Personas / JTBD | Agent (×3) | Reporter + Triager + Feedback Submitter. Triager is one role with two workflows; Reporter vs Feedback Submitter is the asymmetric-identity split added 2026-04-23. |
| 1.4 Competitive Analysis | Standard | "What to borrow / what to skip" is a spec-shaped recurring reference. |
| 1.5 Market Requirements | Decision | Single decision-shaped artifact: "who we're for." |
| 2.1 Information Architecture | Standard | Trivial IA captured as a spec so it stays trivial. |
| 2.2 Noun Vocabulary | Standard | Canonical names + definitions. |
| 2.3 Product Entities | Component (×4) | Report, Debug Bundle, Report Metadata Page, Feedback (added 2026-04-23 in Inventory Pass 3). |
| 2.4 System Design | System (×3) | Debug Bundle Retention, Storage, and Key Scheme (A5) + Service Topology (A14, added 2026-04-23 — Fargate/Route53/IAM/Terraform) + Feedback Storage & Notification (A17, added 2026-04-23 in Inventory Pass 3 — feedback.json/feedback.txt S3 layout + Discord webhook). Separate change frequencies, separate concerns. |
| 3.2 Emotional / Aesthetic Goals | Principle | Tone anchor for a minimal-surface product. |
| 3.5 Anti-Patterns | Principle | The load-bearing "not bun.report" card. (Taxonomy note: linter does not recognize "Anti-Pattern" as a canonical type, so this card is typed Principle with negative-framing language. Renamed 2026-04-22.) |
| 4.1 Design System | Standard | Receipt-page conventions, explicit. |
| 4.3 Prototypes / Mockups | Standard (with sketch) | Sketch + caption as a spec. |
| 4.2 Interaction Patterns | Standard | CLI upload + delete interaction. |
| 4.4 Accessibility Standards | Standard | Factory-mandated policy, even if minimal. |
| 5.1 Key Decisions Log | Decision (×7) | Seven confirmed decisions worth individual cards: Capability URLs over Auth (A9), No Content Dedup (A10), 30-Day Retention Default (A11), Service Stack (A12 — Rust/Fargate, added 2026-04-23), Sentry for Service Errors Not Bundle Processing (A13 — added 2026-04-23), IP Logging for Abuse Protection (A18 — added 2026-04-23 in Inventory Pass 3), Fire-and-Forget Feedback (A19 — added 2026-04-23 in Inventory Pass 3). |
| 5.2+5.4 Institutional Memory | — | Empty at start; accumulates over time. No card yet. |
| 5.3 Roadmap | Principle | "Narrow forever" — Roadmap for this product is a commitment, not a plan. |

---

## Tier 1 — Foundation (5 areas, 6 cards)

Build these first. Nothing else can be written well without them. Foundation cards are the anchor; product-layer cards inherit their frames.

| # | Card | Type | Area | Zone | Build Order | Source Status | Intent |
|---|------|------|------|------|-------------|---------------|--------|
| F1 | `Product Thesis - Pickup Point, Not Triage Surface` | Product Thesis | 1.1 | `/rationale/` | 1 | EXISTS | The anchor. "Intake-only, forever; bundle retrieval is out-of-band via S3." Absorbs the narrow-scope-vs-bun.report framing as its WHY. |
| F2 | `Standard - Noun Vocabulary` | Standard | 2.2 | `/rationale/` | 2 | EXISTS | Canonical names + definitions for Report, Debug Bundle, Reporter, Triager, Report Metadata Page. Feeds every downstream card. |
| F3 | `Principle - Reject "Like bun.report" Drift` | Principle | 3.5 | `/rationale/` | 3 | EXISTS | Names the specific failure mode: agents told "like bun.report" will add symbolication, rendering, feedback, dashboards. Cascade-prevention card. Originally manifested as "Anti-Pattern"; renamed 2026-04-22 for linter-taxonomy alignment. |
| F4 | `Principle - Minimal Public Surface Feels Calm` | Principle | 3.2 | `/rationale/` | 4 | PARTIAL | Emotional/aesthetic anchor. Tone of CLI output, error messages, receipt page. Source has "near-headless" and "minimal public surface" but no tone guidance yet. |
| F5 | `Principle - Strategy: Support Effectiveness for Self-Hosted Users` | Principle | 1.2 | `/rationale/` | 5 | EXISTS | Strategic WHY as a standalone principle. Explains why we optimize for "team can get the bundle" over e.g. "user sees rich diagnostics." |
| F6 | `Principle - Narrow Forever (Roadmap as Commitment)` | Principle | 5.3 | `/rationale/` | 6 | EXISTS | Paired with F1. Treats Roadmap as a no-expansion commitment. Covers what's permanently out-of-scope. Built in Foundation because every downstream card references it. |

**Note on F5/F6 placement:** Areas 1.2 (Strategy) and 5.3 (Roadmap) are not Foundation-tier in the config (1.2 is Foundation; 5.3 is Amplifier). F5 is Foundation per config. F6 is promoted to Foundation by Conan because the "narrow forever" commitment is load-bearing for every other card and Bryan just confirmed it as an identity decision. Flagged as a Conan promotion — revert to Amplifier if the human disagrees.

---

## Tier 2 — Core (4 areas, 4 cards)

Build after Foundation. These enrich the Foundation frame with structure and concrete specs.

| # | Card | Type | Area | Zone | Build Order | Source Status | Intent |
|---|------|------|------|------|-------------|---------------|--------|
| C1 | `Standard - Competitive Context` | Standard | 1.4 | `/rationale/` | 7 | EXISTS (closed as one-pager per Conan Open Flag 3; filename short form, subtitle "What to Borrow, What to Skip" lives inside the card) | bun.report as template; paste services + support-bundle uploaders named-to-close, not surveyed. Ratified 2026-04-23. |
| C2 | `Standard - Design System (Receipt Page + CLI Output)` | Standard | 4.1 | `/rationale/` | 8 | ABSENT | Typography, spacing, color, copy conventions for receipt page. CLI output conventions. Source has no visual direction. |
| C3 | `Standard - Receipt Page Sketch` | Standard | 4.3 | `/rationale/` | 9 | ABSENT | One-page sketch with labels. Source describes the page content but not its layout. |
| C4 | `Decision - Who This Service Is For` | Decision | 1.5 | `/rationale/` | 10 | EXISTS | Self-hosted Fabro users, primary and only. Hosted-Fabro bundles are explicitly deferred. Source has this — just needs to be a proper Decision card. |

---

## Tier 3 — Amplifier (9 areas; 6 cards planned, 3 areas deferred)

Build after Core. Several Amplifier areas collapse into single cards or pair with Decisions.

| # | Card | Type | Area | Zone | Build Order | Source Status | Intent |
|---|------|------|------|------|-------------|---------------|--------|
| A1 | `Agent - Reporter` | Agent | 1.3 | `/product/` | 11 | EXISTS | Thin persona card. Fabro user with a bug; no identity; uses CLI; expects to share URL out-of-band. |
| A2 | `Agent - Triager` | Agent | 1.3 | `/product/` | 12 | EXISTS | Thin persona card. Fabro team member; uses own AWS credentials; never touches intake service. |
| A3 | `Component - Report` | Component | 2.3 | `/product/` | 13 | EXISTS | The upload event and its public identity. Links to Noun Vocabulary (F2). |
| A4 | `Component - Report Metadata Page` | Component | 2.3 | `/product/` | 14 | EXISTS | The one-page receipt at the URL. Links to Design System (C2) once it exists. |
| A5 | `System - Debug Bundle Retention, Storage, and Key Scheme` | System | 2.4 | `/product/` | 15 | EXISTS (as of 2026-04-23 via `storage-and-keys.md`) | Renamed 2026-04-23 from `System - Retention, Storage, and Key Scheme` (on-disk filename + 41 wikilinks cascaded in Phase 2 at commit `9e6efa2`) so A17's parallel `System - Feedback Storage & Notification` naming pattern is visible at filename level. 30-day expiration via S3 lifecycle rule; date-partitioned keys `<YYYY>/<MM>/<DD>/<id>/`; sidecar JSON metadata at `<id>/metadata.json`; bundle at `<id>/bundle.tar.gz`; 32-hex-char Report IDs (128 bits entropy); bucket `fabro-report-bundles` (env-overridable), region `us-east-1` (env-overridable). |
| A6 | `Standard - Information Architecture` | Standard | 2.1 | `/rationale/` | 16 | EXISTS | Minimal IA: one page, one URL per Report, delete-on-URL. Captured so it stays minimal. |
| A7 | `Standard - Interaction Patterns (CLI + Delete)` | Standard | 4.2 | `/rationale/` | 17 | EXISTS (as of 2026-04-23 via `cli-interaction-patterns.md`) | `fabro dump --upload` / `fabro upload --dry-run` (renamed from `fabro report` 2026-04-23); presigned S3 URLs so bytes never flow through service; progress bar; `NO_COLOR`-aware colored errors; client-side delete confirmation; 50MB cap. |
| A8 | `Standard - Accessibility Policy` | Standard | 4.4 | `/rationale/` | 18 | EXISTS (as of 2026-04-23 via `accessibility.md`) | WCAG 2.2 AA for Report Metadata Page; `NO_COLOR` for CLI; keyboard + screen-reader HTML best practices; `/ui` skill workflow note for page implementation. |
| A9 | `Decision - Capability URLs over Auth` | Decision | 5.1 | `/rationale/` | 19 | EXISTS | Why no auth; why URL-as-capability is the whole access model. |
| A10 | `Decision - No Content Dedup` | Decision | 5.1 | `/rationale/` | 20 | EXISTS | Why no dedup; rationale is "bundles expire anyway." |
| A11 | `Decision - 30-Day Retention Default` | Decision | 5.1 | `/rationale/` | 21 | EXISTS | Why 30 days; what happens at expiration. |
| A12 | `Decision - Service Stack` | Decision | 5.1 | `/rationale/` | 22 | EXISTS (as of 2026-04-23 via `tech-stack-and-deployment.md`) | Rust on AWS Fargate, Docker image, Terraform-provisioned. Rejects Bun/Fly (bun.report template) and Rust/Fly (Fabro's deployment model). Chosen for AWS-native IAM-role-based S3 access. Added 2026-04-23. |
| A13 | `Decision - Sentry for Service Errors, Not Bundle Processing` | Decision | 5.1 | `/rationale/` | 23 | EXISTS (as of 2026-04-23 via `tech-stack-and-deployment.md`) | Distinguishes Sentry-as-service-error-tracker (chosen) from Sentry-as-downstream-bundle-processor (rejected by F3). Pairs with an inline clarifying note in F3. Added 2026-04-23. |
| A14 | `System - Service Topology` | System | 2.4 | `/product/` | 24 | EXISTS (as of 2026-04-23 via `tech-stack-and-deployment.md`) | Fargate task, Route53 DNS (`fabro.report` registered), S3 bucket, IAM role, Terraform-provisioned. Separate from A5 (data topology) — different change frequencies, different concerns. Added 2026-04-23. |
| A15 | `Agent - Feedback Submitter` | Agent | 1.3 | `/product/` | 25 | EXISTS (feedback-flow.md) | Persona card, parallel to A1 Reporter. Identity model: optional email + server-logged IP + Fabro version metadata. No post-submission interaction (no URL, no delete). Added 2026-04-23 in Inventory Pass 3. |
| A16 | `Component - Feedback` | Component | 2.3 | `/product/` | 26 | EXISTS (feedback-flow.md) | First-class entity, parallel to A3 Report. UUIDv7 ID, message text, file attachments manifest, optional email, IP, Fabro version/platform/arch, timestamp, retention deadline. Added 2026-04-23 in Inventory Pass 3. |
| A17 | `System - Feedback Storage & Notification` | System | 2.4 | `/product/` | 27 | EXISTS (feedback-flow.md) | Parallel to A5 for Debug Bundles. Covers: `feedback.json` + `feedback.txt` S3 layout under `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`, Discord webhook posting with component-v2 payload shape, failure self-reporting. Absorbs the Discord-webhook rationale inline (see Open Flag 6). Added 2026-04-23 in Inventory Pass 3. |
| A18 | `Decision - IP Logging for Abuse Protection` | Decision | 5.1 | `/rationale/` | 28 | EXISTS (feedback-flow.md) | First identity-data capture in the library. Why IPs are logged, scope of use (abuse protection + basic attribution only — no analytics, no cross-submission tracking), retention (same 30 days). Thin Decision, parallel in shape to A10. Added 2026-04-23 in Inventory Pass 3. |
| A19 | `Decision - Fire-and-Forget Feedback` | Decision | 5.1 | `/rationale/` | 29 | EXISTS (feedback-flow.md) | Why feedback returns no URL, has no delete, and has no capability-URL model. Pairs with A9 (capability URLs for Debug Bundles) as the inverse decision for the other flow. Thin Decision. Added 2026-04-23 in Inventory Pass 3. |

**Deferred Amplifier area:**

| Area | Reason | When to revisit |
|---|---|---|
| 5.2+5.4 Institutional Memory | Nothing to remember yet. This area accumulates over time as decisions are revisited, bugs are found, agent drift is caught. | When the first Decision card gets revised or reversed, or when Conan's first Health Check surfaces something worth logging. |

---

## Tier 4 — Deprioritized (4 areas, 0 cards)

All four areas omitted as not-needed for this product.

| Area | Why Omitted |
|---|---|
| 2.5 Full GDD / PRD | Foundation + Core already carry more than this product needs. A GDD would be redundant. |
| 3.1 User Journey Maps | One journey: run `fabro report` → get URL → paste it. A Journey map would restate the Flow section of the Product Thesis in a different format. Not worth the duplication. |
| 3.3 Engagement Loops | Product is used once per bug by design. Loops are anti-pattern here. |
| 3.4 Progression / Mastery | No progression to design. Agent tempted to invent one should be told to re-read F1 and F3. |

If the human wants any of these for tier-completeness reasons, flag it and Conan will re-plan.

---

## Source Gaps — What Needs to Be Captured Before Blocked Cards Can Be Drafted

Cards marked PARTIAL or ABSENT in the tables above are blocked on source material that does not yet exist in `docs/alexandria/sources/`. The gaps, grouped by who can resolve them:

### Requires Bryan (product owner input)

| Gap | Feeds Card(s) | One-line Ask |
|---|---|---|
| Tone guidance for CLI output, errors, and receipt page | F4 (Emotional Goals) | "What should a Reporter *feel* when they run `fabro report` and get back a URL? Efficient? Reassured? Invisible?" |
| Receipt page visual direction | C2, C3, A4 | "Dark mode? Monospace? Terminal-aesthetic? Or plain system fonts? Any reference product we want it to feel like?" |
| CLI UX specifics | A7 | "Progress indicator during upload? What does an upload-failure error look like? Delete confirmation — one click or two?" |
| Retention/storage mechanism | A5 | "S3 lifecycle rule for the 30-day expiration, or app-side cron? Key scheme — UUID, short slug, hierarchical?" |
| Accessibility floor | A8 | "Beyond 'meets WCAG AA,' is there anything specific we care about? CLI NO_COLOR support?" |

### Requires Bryan or light research

| Gap | Feeds Card(s) | One-line Ask |
|---|---|---|
| Short survey of paste services / support-bundle patterns | C1 | Maybe 30 minutes of Raven-or-Bryan research. Could also be punted — the bun.report comparison alone may suffice for this product's size. |

### Already covered by existing source

Cards F1, F2, F3, F5, F6, C4, A1, A2, A3, A4, A6, A9, A10, A11 (14 of 16) can be drafted by Sam from `product-vision.md` today.

---

## Recommended Build Sequence (first 5 cards in order)

| Order | Card | Why This First |
|---|---|---|
| 1 | F1 `Product Thesis - Pickup Point, Not Triage Surface` | The anchor. Every other Foundation card references this. |
| 2 | F2 `Standard - Noun Vocabulary` | Every downstream card uses these nouns. Must exist before Components, Agents, or Systems are drafted. |
| 3 | F3 `Anti-Pattern - "Like bun.report" Drift` | Paired with F1. Names the specific cascade risk. Must exist before product-layer cards so Sam has the negative frame. |
| 4 | F5 `Principle - Strategy: Support Effectiveness for Self-Hosted Users` | The strategic WHY. Explains what the product is optimizing for, which governs downstream trade-offs. |
| 5 | F6 `Principle - Narrow Forever (Roadmap as Commitment)` | Pairs with F1 + F3. Turns "narrow scope" from a current fact into a permanent commitment. Every Decision card and every product-layer card benefits from this being upstream. |

Then F4 (blocked on tone source), then Core tier.

## Follow-On Build Sequence (2026-04-23)

After the 12 cards ratified through commit `3b4fd06`, and after the CLI rename cascade (mechanical edit pass across F2/A1/F1/F3/F4/F5/F6/A2/A3/A4/A6/A9/A10/A11/C1/C4 — see `docs/alexandria/sources/cli-interaction-patterns.md` for scope), the remaining build order is:

| Order | Card | Depends On |
|---|---|---|
| 17 | A5 `System - Retention, Storage, and Key Scheme` | Cascade done; F2 canonical names current |
| 18 | A7 `Standard - Interaction Patterns (CLI + Delete)` | Cascade done |
| 19 | A8 `Standard - Accessibility Policy` | Cascade done |
| 20 | A12 `Decision - Service Stack` | Independent |
| 21 | A13 `Decision - Sentry for Service Errors, Not Bundle Processing` | Pairs with inline F3 clarifying note in same pass |
| 22 | A14 `System - Service Topology` | Cites A5, A12, A13 — drafts last |

C2 and C3 remain blocked on visual direction.

## Inventory Pass 3 Follow-On Build Sequence (2026-04-23)

Phased cascade for the feedback scope expansion. Phase boundaries are cascade-safety boundaries: identity lands before product layer; existing-card retrofits land before new cards so new cards link into a stable graph.

| Order | Card | Depends On |
|---|---|---|
| 25 | Phase 1 cascade (F1, F3, F4, F5, F6) — identity edits | Existing F1/F3/F4/F5/F6 cards ratified |
| 26 | Phase 1 cascade (F2) — add nouns + CLI row | After identity edits |
| 27 | Phase 2 cascade (A1, A3, A5 rename, A9, A11, A13 — tiny updates + A5 filename move) | After Foundation cascade |
| 28 | Phase 2 cascade (A2, A6, A7, A8, A14 — medium updates) | After tiny updates |
| 29 | Phase 3 — A15 `Agent - Feedback Submitter` | F2 updated |
| 30 | Phase 3 — A16 `Component - Feedback` | F2 updated; A15 drafted |
| 31 | Phase 3 — A17 `System - Feedback Storage & Notification` | A16 drafted; A14 updated |
| 32 | Phase 3 — A18 `Decision - IP Logging for Abuse Protection` | A17 drafted |
| 33 | Phase 3 — A19 `Decision - Fire-and-Forget Feedback` | A16 drafted; A9 updated |
| 34 | Phase 4 — C1 tiny update (`bun feedback` template note) | Independent; pairs anywhere |

C2 and C3 remain blocked on visual direction.

---

## Blocked Cards

| Card | Blocker | Unblock By |
|---|---|---|
| C2 `Standard - Design System` | No visual direction captured | Bryan provides receipt-page visual direction; source-material file created. |
| C3 `Standard - Receipt Page Sketch` | No sketch exists | Same blocker as C2 — draft together. |

*Resolved 2026-04-23 (sources captured, unblocked):* F4, A5, A7, A8.
*Resolved 2026-04-22:* C1 (closed as one-pager per Open Flag 3 — no survey needed).

**Only 2 of 19 cards remain blocked,** both on visual direction for the Report Metadata Page.

---

## Accepted Lint Warnings

These warnings are structural properties of this library at its current scale, not defects to fix. They are expected to persist and should not be re-surfaced as new drift in future builds.

| Warning | Scope | Rationale | Adjudicated |
|---|---|---|---|
| `sweep4.missing_containment` on `Component - Report` | A3 | No Section/Template/Governance card exists in this library. A Section card here would duplicate F1's framing or be a thin stub; a separate Domain card is not load-bearing for a 16-card intake-service library. F1 operates as the de facto container for every product-layer card. Revisit if the library grows past ~30 cards. | Conan, 2026-04-23 (`docs/alexandria/reviews/batch-c1-a3-a4-a6-spot-check.md`) |
| `sweep4.missing_containment` on `Component - Report Metadata Page` | A4 | Same rationale as A3 — the product-layer cards in this library use F1 as their de facto structural anchor. | Conan, 2026-04-23 |
| `sweep4.missing_containment` on `Component - Feedback` | A16 | Same rationale as A3 and A4 — this library uses F1 as the de facto container for all product-layer cards. Pre-adjudicated in Inventory Pass 3 so it does not re-surface during spot-check. | Conan, 2026-04-23 (`docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` Part 4) |
| `sweep4.underpopulated_layer` on `experience/` | library-wide | Manifest deliberately deprioritizes all Experience-layer areas (3.1 User Journey Maps, 3.3 Engagement Loops, 3.4 Progression / Mastery). Zero Experience cards is a positive design choice for a narrow intake service, not an omission. | Conan, 2026-04-23 |
| `sweep2.word_count` (currently 20 instances) | library-wide | Every card exceeds the 700-word soft cap because each card is calibrated for standalone Factory-mode agent readability. Compression would undermine that property. Accepted in every prior spot-check (F1-F6, Decisions, Agents, this batch). | Conan across all spot-checks |

Future linter runs should treat these as known-accepted, not re-flag as new drift. If a new warning of these shapes appears on a new card, adjudicate it on its own merits rather than treating it as a default-accept.

---

## Open Flags for Human Judgment

1. **F6 Foundation promotion.** Conan promoted `Principle - Narrow Forever` from the Amplifier-tier 5.3 Roadmap area into Foundation because the "narrow forever" commitment is structurally load-bearing for every downstream card. If you disagree, move it to Amplifier position after A11.
2. **Zone paths.** Proposed `/rationale/`, `/product/`, `/experience/` based on standard Alexandria conventions. The repo has no existing library cards to pattern-match against. If your project convention differs, flag and Conan will re-emit.
3. **C1 scope.** "Short survey of paste services / support-bundle uploaders" may be disproportionate effort for this product's size. Conan's recommendation: skip the external survey; make C1 a one-page card that states "bun.report is our template for intake shape only; no other competitor analysis is load-bearing at this scale" and close the area.
4. **Institutional Memory (5.2+5.4).** No card planned now. Revisit after first Health Check or first Decision reversal.
5. **Two Agents in area 1.3.** Config allocates one Amplifier slot to "User Personas / JTBD." Splitting into Agent - Reporter and Agent - Triager produces two cards. Both are thin. If you prefer one card (`Agent - Personas`), collapse — Conan recommends two because the asymmetry (no-identity Reporter vs credentialed Triager) is load-bearing and deserves separate framing. **Update 2026-04-23:** Inventory Pass 3 adds a third Agent, `Agent - Feedback Submitter` (A15), for the same reason — asymmetric identity vs Reporter (optional email + server-logged IP vs. no identity at all).
6. **Discord webhook Decision absorbed into A17.** Conan recommends no standalone `Decision - Discord Webhook for Feedback Notification` card; the rationale (why Discord, what gets posted, how failures self-report) lives inside `System - Feedback Storage & Notification` (A17) as the notification section, with a cross-reference from F3's clarifying note. If the human wants it as a standalone Decision, promote to A20 — this flag is the place that reopens.
7. **F6 identity-line rewrite.** The "narrow forever" identity sentence in F6 changes *shape*, not just prose, in Phase 1 of the 2026-04-23 cascade. New identity: "exactly two intake types (Debug Bundles + Feedback), forever." Flagged so the Phase 1 spot-check verifies the rewrite didn't narrow or broaden the commitment unintentionally — in particular, the "no third intake type, forever" identity-level ceiling must be visibly preserved and every prior load-bearing phrase ("narrow forever", "intake-only", "pickup point", "permanently out-of-scope") must survive intact.

---

## Recommendation: Next Agent Action

**Sam: draft card F1 `Product Thesis - Pickup Point, Not Triage Surface`.**

Then, in order:
- **Sam: draft card F2 `Standard - Noun Vocabulary`.**
- **Sam: draft card F3 `Anti-Pattern - "Like bun.report" Drift`.**

After F1–F3 are written, return to Conan for Spot-Check. Do not build F4–F6 before Spot-Check — the first three Foundation cards set the pattern for everything else and should be validated before scale.

Parallel to Sam's drafting: **Bryan provides tone guidance** (unblocks F4) and **visual direction for the receipt page** (unblocks C2/C3). These can be captured into new source files in `docs/alexandria/sources/` whenever convenient; they are not on the critical path for Foundation F1–F3.

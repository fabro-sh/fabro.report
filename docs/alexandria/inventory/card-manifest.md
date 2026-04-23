# Alexandria Card Manifest — `fabro.report`

**Job:** Conan Job 1 — Inventory
**Date:** 2026-04-22
**Config:** Factory × Low Novelty × Low Complexity — 22-area pool (5 Foundation / 4 Core / 9 Amplifier / 4 Deprioritized)
**Source inputs:**
- `docs/alexandria/alexandria-config.json`
- `docs/alexandria/initialize-output.md`
- `docs/alexandria/sources/product-vision.md` (updated — strategic WHY and "narrow forever" both confirmed)
- `docs/alexandria/reviews/product-vision-source-assessment.md` (prior candidate list, 14 cards)

**Summary:** 16 cards to build. 5 Foundation, 4 Core, 7 Amplifier, 0 Deprioritized (all 4 Deprioritized areas omitted as not-needed). 6 cards blocked on missing source material; 10 cards draftable from existing source. First 3 draftable: Product Thesis, Noun Vocabulary, Anti-Patterns.

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
| 1.3 Personas / JTBD | Agent (×2) | Reporter + Triager are Agents in the library taxonomy. |
| 1.4 Competitive Analysis | Standard | "What to borrow / what to skip" is a spec-shaped recurring reference. |
| 1.5 Market Requirements | Decision | Single decision-shaped artifact: "who we're for." |
| 2.1 Information Architecture | Standard | Trivial IA captured as a spec so it stays trivial. |
| 2.2 Noun Vocabulary | Standard | Canonical names + definitions. |
| 2.3 Product Entities | Component (×3) | Report, Debug Bundle, Report Metadata Page. |
| 2.4 System Design | System | Retention, S3 layout, key scheme. |
| 3.2 Emotional / Aesthetic Goals | Principle | Tone anchor for a minimal-surface product. |
| 3.5 Anti-Patterns | Principle | The load-bearing "not bun.report" card. (Taxonomy note: linter does not recognize "Anti-Pattern" as a canonical type, so this card is typed Principle with negative-framing language. Renamed 2026-04-22.) |
| 4.1 Design System | Standard | Receipt-page conventions, explicit. |
| 4.3 Prototypes / Mockups | Standard (with sketch) | Sketch + caption as a spec. |
| 4.2 Interaction Patterns | Standard | CLI upload + delete interaction. |
| 4.4 Accessibility Standards | Standard | Factory-mandated policy, even if minimal. |
| 5.1 Key Decisions Log | Decision (×3) | Three confirmed decisions worth individual cards. |
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
| C1 | `Standard - Competitive Context: What to Borrow, What to Skip` | Standard | 1.4 | `/rationale/` | 7 | PARTIAL | bun.report as template; paste services + support-bundle uploaders as analogues. Source has the bun.report comparison in triplicate; lacks an actual survey of paste services / support-bundle patterns. |
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
| A5 | `System - Retention, Storage, and Key Scheme` | System | 2.4 | `/product/` | 15 | PARTIAL | 30-day expiration mechanics, S3 key layout, bundle storage. Source has policy but not mechanism (TTL via S3 lifecycle? cron? soft-delete?). |
| A6 | `Standard - Information Architecture` | Standard | 2.1 | `/rationale/` | 16 | EXISTS | Minimal IA: one page, one URL per Report, delete-on-URL. Captured so it stays minimal. |
| A7 | `Standard - Interaction Patterns (CLI + Delete)` | Standard | 4.2 | `/rationale/` | 17 | PARTIAL | `fabro report` CLI UX (progress, errors, URL output); delete-button confirmation flow. Source missing: progress indicator? error surface? delete confirmation? |
| A8 | `Standard - Accessibility Policy` | Standard | 4.4 | `/rationale/` | 18 | ABSENT | Factory-mandated minimum. Source says "almost irrelevant for a headless-ish receipt" — card should be explicit about the minimum floor (WCAG AA on receipt page, CLI output respects NO_COLOR, etc.). |
| A9 | `Decision - Capability URLs over Auth` | Decision | 5.1 | `/rationale/` | 19 | EXISTS | Why no auth; why URL-as-capability is the whole access model. |
| A10 | `Decision - No Content Dedup` | Decision | 5.1 | `/rationale/` | 20 | EXISTS | Why no dedup; rationale is "bundles expire anyway." |
| A11 | `Decision - 30-Day Retention Default` | Decision | 5.1 | `/rationale/` | 21 | EXISTS | Why 30 days; what happens at expiration. |

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

---

## Blocked Cards

| Card | Blocker | Unblock By |
|---|---|---|
| F4 `Principle - Minimal Public Surface Feels Calm` | No tone guidance in source | Bryan provides tone direction; Sam extracts to a source-material file, Conan assesses it, then drafts card. |
| C1 `Standard - Competitive Context` | Partial — bun.report covered; paste services / support-bundle uploaders not surveyed | Bryan decides whether a survey is worth the effort; if yes, Raven or Bryan produces the survey. |
| C2 `Standard - Design System` | No visual direction captured | Bryan provides receipt-page visual direction; source-material file created. |
| C3 `Standard - Receipt Page Sketch` | No sketch exists | Same blocker as C2 — draft together. |
| A5 `System - Retention, Storage, and Key Scheme` | Mechanism (vs policy) not specified | Bryan picks S3-lifecycle vs app-cron; decides key scheme. |
| A7 `Standard - Interaction Patterns` | CLI UX specifics + delete-confirmation flow undefined | Bryan answers the CLI-UX questions. |
| A8 `Standard - Accessibility Policy` | Floor not specified | Bryan confirms "WCAG AA + NO_COLOR" is the floor or provides alternative. |

**Claim the build is not blocked on these.** Sam can draft 14 of 16 cards today. The 7 blocked cards are Amplifier/Core tier; they wait their turn regardless.

---

## Open Flags for Human Judgment

1. **F6 Foundation promotion.** Conan promoted `Principle - Narrow Forever` from the Amplifier-tier 5.3 Roadmap area into Foundation because the "narrow forever" commitment is structurally load-bearing for every downstream card. If you disagree, move it to Amplifier position after A11.
2. **Zone paths.** Proposed `/rationale/`, `/product/`, `/experience/` based on standard Alexandria conventions. The repo has no existing library cards to pattern-match against. If your project convention differs, flag and Conan will re-emit.
3. **C1 scope.** "Short survey of paste services / support-bundle uploaders" may be disproportionate effort for this product's size. Conan's recommendation: skip the external survey; make C1 a one-page card that states "bun.report is our template for intake shape only; no other competitor analysis is load-bearing at this scale" and close the area.
4. **Institutional Memory (5.2+5.4).** No card planned now. Revisit after first Health Check or first Decision reversal.
5. **Two Agents in area 1.3.** Config allocates one Amplifier slot to "User Personas / JTBD." Splitting into Agent - Reporter and Agent - Triager produces two cards. Both are thin. If you prefer one card (`Agent - Personas`), collapse — Conan recommends two because the asymmetry (no-identity Reporter vs credentialed Triager) is load-bearing and deserves separate framing.

---

## Recommendation: Next Agent Action

**Sam: draft card F1 `Product Thesis - Pickup Point, Not Triage Surface`.**

Then, in order:
- **Sam: draft card F2 `Standard - Noun Vocabulary`.**
- **Sam: draft card F3 `Anti-Pattern - "Like bun.report" Drift`.**

After F1–F3 are written, return to Conan for Spot-Check. Do not build F4–F6 before Spot-Check — the first three Foundation cards set the pattern for everything else and should be validated before scale.

Parallel to Sam's drafting: **Bryan provides tone guidance** (unblocks F4) and **visual direction for the receipt page** (unblocks C2/C3). These can be captured into new source files in `docs/alexandria/sources/` whenever convenient; they are not on the critical path for Foundation F1–F3.

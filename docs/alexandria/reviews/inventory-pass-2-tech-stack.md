# Inventory Pass 2 — Tech Stack, Storage, CLI, Accessibility Sources

**Job:** Conan Job 1 — Inventory (incremental pass, not a full re-manifest)
**Date:** 2026-04-23
**Scope:** Four new source files captured 2026-04-23:
- `docs/alexandria/sources/tech-stack-and-deployment.md`
- `docs/alexandria/sources/storage-and-keys.md`
- `docs/alexandria/sources/cli-interaction-patterns.md`
- `docs/alexandria/sources/accessibility.md`
**Library state going in:** 16 of 16 manifested cards drafted + spot-check-approved. 5 cards blocked on source (C2, C3, A5, A7, A8).
**Config:** Factory × Low × Low (22-area pool) — unchanged.
**Assessor:** Conan

---

## Part 1 — A5 / A7 / A8 Unblocking Summary

All three sources are complete enough to move the corresponding cards from `PARTIAL`/`ABSENT` to `EXISTS`. Each source was written to the level of specificity Sam needs to draft a load-bearing card without speculating.

### A5 `System - Retention, Storage, and Key Scheme` — UNBLOCKED (EXISTS)

**Feeding source:** `docs/alexandria/sources/storage-and-keys.md`

**What Sam has concretely:**

| Required element (per manifest PARTIAL list) | Source coverage | Load-bearing? |
|---|---|---|
| Retention mechanism (lifecycle vs cron) | S3 lifecycle rule, Terraform-declared, `expiration.days = 30`. Zero cron. | Yes — explicitly named as drift-preventing against "app-side sweeper" anti-example. |
| S3 key layout | `<YYYY>/<MM>/<DD>/<id>.tar.gz`, date-partitioned, daily granularity | Yes — includes rationale (browsability, lifecycle correlation). |
| Bucket model | New dedicated bucket, private, IAM-role write, Triager-credential read | Yes — isolation rationale stated. |
| URL format | `fabro.report/r/<32-hex-chars>` | Yes — entropy floor (128 bits) tied back to A9 "non-guessable URLs are load-bearing." |
| ID generation | Cryptographically random, not sequential / not timestamp-derived | Yes — anti-examples already enumerated. |
| Metadata-bundle lifecycle coupling | Both expire on same day; sidecar JSON as Raven's best-guess | Flagged as unresolved in source's Open Questions. |

**Gaps (flagged, non-blocking):**
- Metadata storage location (sidecar JSON vs object user-metadata vs datastore) — source Open Question 1. Raven recommends sidecar JSON; the recommendation is load-bearing enough to carry into A5 as a default-with-flag. A5 can be drafted with "sidecar JSON co-located with the bundle" as the working assumption, flagged for Bryan confirmation.
- Bucket name — source Open Question 2. Trivial; A5 can use a placeholder (`fabro-report-bundles`) or leave the name unspecified.
- Region — not specified in either storage source or tech-stack source. Flagged in `tech-stack-and-deployment.md` Open Question 1. Not blocking for A5.

**Verdict:** EXISTS. Sam can draft A5 today. The card will be load-bearing, not speculative, provided it treats the metadata-storage choice as a flagged default rather than a silent assumption.

### A7 `Standard - Interaction Patterns (CLI + Delete)` — UNBLOCKED (EXISTS)

**Feeding source:** `docs/alexandria/sources/cli-interaction-patterns.md`

**What Sam has concretely:**

| Required element (per manifest PARTIAL list) | Source coverage | Load-bearing? |
|---|---|---|
| Progress indicator | Progress bar during upload; client-rendered from HTTP progress callback | Yes — tied to F4 tone register ("silent upload is anxiety-inducing"). |
| Error surface | Plain English, name the problem + suggest next action, red-for-error color-when-available, NO_COLOR-honoring | Yes — calibrated against `bun` error output per F4. |
| Delete confirmation | Client-side "are you sure?" in a small amount of JS; rationale for not doing a server roundtrip given | Yes — explicit rejection of the two-step server-side alternative. |
| CLI command structure | `fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro upload --dry-run` | Yes — supersedes earlier `fabro report` / `fabro debug` references; cascade scope enumerated in source. |
| Upload architecture | Presigned S3 URLs; CLI never POSTs bundle bytes to the service | Yes — structurally enforces F3's "Debug Bundle contents are opaque to the service" claim. Architectural, not just UX. |
| Max bundle size | 50 MB hard cap; pre-upload local size check AND presigned-URL `Content-Length` constraint | Yes — both enforcement paths named; prevents "CLI reported OK, S3 rejected" failure mode. |

**Gaps (flagged, non-blocking):**
- Exact error message strings — source explicitly defers to a future downstream `Standard - CLI Copy` card. A7 carries the *register*, not the strings. Correct discipline.
- Whether `fabro upload` takes a path argument or uses a conventional location — Open Question; "likely both" guidance is sufficient for A7 to skate past.
- Success output shape (URL only, or URL + confirmation line) — source resolves this inline ("confirmation line plus the URL" per F4's anti-silent-success rule).

**Verdict:** EXISTS. Sam can draft A7 today. Source is unusually rich — it does double duty as a UX spec AND an architectural spec (presigned URLs).

### A8 `Standard - Accessibility Policy` — UNBLOCKED (EXISTS)

**Feeding source:** `docs/alexandria/sources/accessibility.md`

**What Sam has concretely:**

| Required element (per manifest ABSENT list) | Source coverage | Load-bearing? |
|---|---|---|
| Web accessibility floor | WCAG 2.2 AA (not AAA, not 2.1) — with rationale for each dimension choice | Yes — floor is explicit. |
| CLI accessibility | `NO_COLOR` env var honored per no-color.org | Yes — named as table-stakes for F4's `bun`-calibrated register. |
| Keyboard surface | Keyboard-operable page, logical focus order, visible focus indicators | Yes — explicit delete-button-and-confirmation call-out. |
| Screen reader | Semantic HTML (`<button>`, `<time>`, heading hierarchy, `aria-label` where needed) | Yes — enumerated. |
| Implementation workflow | `/ui` skill named as the implementation tool | Yes — a workflow hint, not a card-scope requirement; A8 should note it as an off-ramp. |
| Anti-examples | Hard-coded ANSI without NO_COLOR guard; mouse-only delete; icon-only buttons; color-only error state; custom modal without focus trap when `confirm()` would do | Yes — directly extractable. |

**Gaps (flagged, non-blocking):**
- No documented accessibility-testing workflow (Axe/WAVE/Lighthouse/manual SR). Source Open Question 1. Does not block A8 — the card states the floor, not the verification procedure.
- Explicit accessibility statement / link on the Report Metadata Page — Source Open Question 3. Minor; A8 can state "no formal accessibility statement required; this product is a small internal tool."
- `--plain` flag for scripting contexts beyond `NO_COLOR` — Open Question 2. Not blocking; can defer to a future CLI card.

**Verdict:** EXISTS. Sam can draft A8 today. Source is small but complete.

---

## Part 2 — New Card Candidates

`tech-stack-and-deployment.md` is the only source that contains content not mapped to an existing manifest entry. The source itself suggests four candidates. Conan's per-candidate adjudication:

### Candidate 1 — `Decision - Rust on AWS Fargate over Bun on Fly`

| Field | Verdict |
|---|---|
| **In manifest?** | **YES** |
| **Tier** | Amplifier |
| **Zone** | `/rationale/` |
| **Area ID** | 5.1 Key Decisions Log (no new area needed — slots alongside A9/A10/A11/C4) |
| **New card ID** | A12 |

**Reasoning — in:**
- This is a genuine, decision-shaped, rejected-alternatives-present product decision. Bryan confirmed the stack explicitly; the rejected alternatives (Bun/Fly matches `bun.report` template; Rust/Fly matches Fabro) are load-bearing *because* this product sits at a structural boundary between those two reference points.
- The "why Fargate and not Fly" rationale (AWS-native posture for IAM-role-based S3 access) is non-obvious and worth preserving. An agent asked "should this move to Fly.io to match Fabro?" needs a durable answer — not just implied-by-current-state.
- Factory-mode agents will treat the chosen stack as a default when generating code for *any* system card. Without A12, they might regenerate a Bun-flavored service because `bun.report` is the stated shape template (C1). C1 says "intake URL shape only, not downstream processing"; A12 extends that discipline to the implementation stack.

**Reasoning for 5.1 placement:** area 5.1 Key Decisions Log already carries three Amplifier-tier cards (A9, A10, A11); adding a fourth is not a distribution-breaking move. The Decision type is exactly right: a past, not-to-be-relitigated choice with rejected alternatives.

### Candidate 2 — `Decision - Sentry for Service Errors, Not Bundle Processing`

| Field | Verdict |
|---|---|
| **In manifest?** | **YES** |
| **Tier** | Amplifier |
| **Zone** | `/rationale/` |
| **Area ID** | 5.1 Key Decisions Log |
| **New card ID** | A13 |

**Reasoning — in:** See Part 3 below for the full argument. Short version: F3 explicitly rejects "Sentry integration" in its drift list. Bryan's confirmed Sentry usage is for service operational errors. A future agent searching the library for "Sentry" will hit F3's rejection and A13's acceptance; the *distinction* between the two uses is what makes both true simultaneously. An in-place F3 note handles the local contradiction; a dedicated Decision card handles the structural searchability (Factory agents look by decision type, not by cross-referencing clarifying notes inside Principles). Both reinforce the other. See Part 3 for recommendation: (c) both.

### Candidate 3 — `System - Service Topology`

| Field | Verdict |
|---|---|
| **In manifest?** | **YES** |
| **Tier** | Amplifier |
| **Zone** | `/product/` |
| **Area ID** | 2.4 System Design (shares area with A5) |
| **New card ID** | A14 |

**Reasoning — in:**
- Area 2.4 (System Design) currently has one card (A5 — retention / storage / keys). The new card covers a *different* dimension of system design: operational topology (Fargate task, Route53, IAM role, Terraform repo) versus data topology (S3 bucket, keys, lifecycle) in A5.
- Both are System-type cards, and both are genuinely load-bearing, but they are not the same card. A5 is "where Reports live"; A14 is "what serves them."
- Splitting is correct because a future agent asking "how does the service deploy?" vs "how does storage work?" gets clean, non-overlapping answers. Merging would overload A5 with unrelated concerns.
- Cross-refs A14 → A5 (storage), A12 (stack decision), A13 (Sentry usage) give the topology card its natural position as the operational integrator.

**Why not a Domain card instead:** a Domain card would claim containment over product-layer cards. The topology is implementation-layer (Fargate, IAM, Terraform) — it doesn't contain Reports or Pages. A System card fits better.

**Scope discipline check:** This adds an implementation-layer System card to a library that is Factory × Low × Low for a narrow intake service. Is it justified?
- A14 carries information a Factory agent building or modifying the service genuinely needs. Without it, the only source of topology truth is the Terraform repo itself — which is the implementation, not the library-level truth about why the topology looks this way.
- A14 is not a re-description of Terraform. It captures *why* the topology is shaped this way (single-region AWS-native, IAM role over baked-in creds, Fargate over EC2/Lambda/Fly) and the structural implications for related cards (A5's IAM boundary, A13's Sentry error flow, A12's deployment target).

**Verdict stands: in.**

### Candidate 4 — `Standard - Observability`

| Field | Verdict |
|---|---|
| **In manifest?** | **NO** |
| **Tier** | — |
| **Zone** | — |
| **Area ID** | — |
| **Rejected because** | Under-specified + scope-creep risk |

**Reasoning — out:**
- The confirmed content is thin: "Sentry for errors; CloudWatch logs by default; metrics and tracing deferred." That's three bullets, two of which are default Fargate behavior.
- The Sentry-for-errors content is fully absorbed by A13 (the new Decision card). The CloudWatch content is implicit-from-Fargate. The "metrics and tracing deferred" content is a non-decision — it's the absence of a decision.
- A Standard card here would be a thin stub at best, or a speculative expansion at worst (tempting Sam to invent metric names, SLO targets, tracing vendor choices that have not been confirmed). This is precisely the drift shape F3 warns against: adding capability because the slot exists.
- If metrics / tracing / structured-logging decisions get made later, they can land as a new Standard card at that point — with actual content, not placeholders.

**Scope-discipline check:** this library is 16 (now manifesting ~19) cards for a narrow intake service. Standards covering near-default behavior would bloat the library without load-bearing content.

**Verdict: out. Not in manifest.**

### Summary of Manifest Additions

| New card | Tier | Zone | Area | Source |
|---|---|---|---|---|
| A12 `Decision - Rust on AWS Fargate over Bun on Fly` | Amplifier | `/rationale/` | 5.1 | `tech-stack-and-deployment.md` |
| A13 `Decision - Sentry for Service Errors, Not Bundle Processing` | Amplifier | `/rationale/` | 5.1 | `tech-stack-and-deployment.md` (+ F3 reinforcement per Part 3) |
| A14 `System - Service Topology` | Amplifier | `/product/` | 2.4 | `tech-stack-and-deployment.md` |

Net new cards: +3. Manifest grows from 16 to 19. Still well within the Factory × Low × Low intent (the 22-area pool, with 4 deprioritized areas omitted, permits up to 18 cards by tier-count arithmetic; A12/A13 are additions to an area that already carries three cards, and A14 is an addition to an area that already carries one — neither violates area-distribution discipline).

---

## Part 3 — The Sentry F3 Tension: Recommendation

**Framing:** F3 `Principle - Reject Like bun.report Drift` enumerates a drift list that includes "If asked to integrate with Sentry (or any crash-reporting SaaS): do NOT…" The rejection is about Sentry-as-downstream-bundle-processor (the `bun.report` use case: parsing Debug Bundle contents as crash payloads and forwarding them into Sentry). Bryan's confirmed Sentry usage is for the `fabro.report` service's own operational errors (panics, upload failures in the Rust service). **These are not contradictory, but a Factory-mode agent reading F3 cold will see an apparent contradiction.**

### Conan's Recommendation: **(c) Both — in-place F3 note AND a new Decision card (A13)**

**Reasoning:**

**Why (a) alone is insufficient:** An in-place note in F3 works for a reader who reads F3 top-to-bottom. But Factory-mode agents often search by artifact type: "show me the Decision cards about observability" → they look in 5.1 Decisions, not in F3 Principles. Without A13, that search turns up empty and the agent is left to infer from F3 — exactly the cold-read failure mode we're trying to prevent. Relying on the reader to always encounter F3 before encountering any code change that touches Sentry is a structural bet that will fail under agent retrieval patterns.

**Why (b) alone is insufficient:** Without an F3 note, F3 reads as flatly rejecting Sentry. A reader hitting F3 and *not* also hitting A13 (because they searched by Principle type, not Decision type) gets the wrong answer. F3's drift list is the first artifact any agent reading "what to reject from the `bun.report` template" will encounter; leaving it looking like it forbids Sentry across the board, with the correction living only in a sibling file, is brittle.

**Why (c) — belt and suspenders — is right:**
- F3's enumerated drift list stays intact (minimal surgery, preserves the concrete-drift-shape discipline F3 earned). The note is additive: one short paragraph, probably one subsection header, distinguishing the two Sentry use cases and pointing at A13.
- A13 carries the distinction as a first-class Decision card, discoverable by Factory agents searching 5.1 for "decisions about error handling" or "decisions about observability."
- A13 → F3 backlink and F3 → A13 forward-link create a bidirectional handle. An agent hitting either card resolves the distinction by following one link.
- The duplication is about 3 sentences of content across the two cards. Cheap. Structurally worth it.

**Concrete shape of the F3 note (for Sam to write when drafting A13):**
- Location: probably at the end of F3's "Drift Examples" section, as a short "Clarification — Sentry" subsection, or inline in the drift list with an explicit "Note: this does not reject Sentry for the service's own operational errors — see [[Decision - Sentry for Service Errors, Not Bundle Processing]]" pointer.
- Length: 2–4 sentences. Does not rewrite F3's drift-list entries; adds a distinguishing note.
- Scope: names the two uses (downstream bundle processor vs service operational error reporting) and points to A13 for the full Decision framing.

**Blast radius of the F3 edit:** F3 itself + A13 new card + 1 backlink in A13 → F3 + 1 backlink in F3 → A13. No cascade across other cards (no other card currently cites F3's Sentry-rejection item specifically).

---

## Part 4 — CLI Rename Cascade Scope

**Cascade scope confirmed from `cli-interaction-patterns.md` L18–L24 and `product-vision.md` L80.** The rename is:

- `fabro debug` (command) → `fabro dump` (everywhere in prose)
- `fabro report` (command) → `fabro dump --upload` (one-step path) or `fabro upload` (two-step path) as context dictates (everywhere in prose)
- **NOT CHANGING:** the product domain `fabro.report`, the noun "Report," the capitalized entity `Report Metadata Page`, or any wikilink targets. Only the bash-rendered CLI verbs move.

### Per-Card Cascade Load

Conan cannot grep the card contents from tools available in this pass; the load estimates below are derived from the manifest's intent column + spot-check-observed prose + the source-vision's explicit references. Sam should treat these as starting estimates and confirm per-card during the edit pass.

| Card | File (per manifest + spot-check conventions) | Expected occurrences | Edit load | Notes |
|---|---|---|---|---|
| **F1** `Product Thesis - Pickup Point, Not Triage Surface` | `/rationale/Product Thesis - Pickup Point, Not Triage Surface.md` | Multiple (flow description in WHAT; validation criteria in HOW) | Medium — 2–4 prose mentions | F1 carries the end-to-end flow; `fabro report` appears in the "Reporter runs X, gets URL" narrative. Replace with `fabro dump --upload` (primary path). |
| **F2** `Standard - Noun Vocabulary` | `/rationale/Standard - Noun Vocabulary.md` | Potentially heavy — this card defines the canonical vocabulary and may have a CLI-verb section | **HIGH — likely 3–6+ occurrences** | F2 is cited by the CLI source as a cascade-target explicitly. If F2 currently defines `fabro report` / `fabro debug` as canonical verbs, Sam needs to update both the canonical-name list AND any cross-reference prose. **If F2 has an explicit "CLI verbs are canonical" subsection, this is the single highest-load edit of the cascade.** |
| **F3** `Principle - Reject Like bun.report Drift` | `/rationale/Principle - Reject Like bun.report Drift.md` | 1–2 | Low — 1–2 prose mentions | Any references in F3's drift-list rationale. Also note: F3 gets the new Sentry clarifying paragraph in the same Sam pass (Part 3). |
| **F4** `Principle - Minimal Public Surface Feels Calm` | `/rationale/Principle - Minimal Public Surface Feels Calm.md` | 1–2 | Low — 1 prose mention | Per the cli-interaction source F4 is cited for tone-register calibration ("efficient and reassured" applied to CLI output). Any `fabro report` mentions in F4's CLI-tone examples → `fabro dump --upload`. |
| **F5** `Principle - Strategy: Support Effectiveness for Self-Hosted Users` | `/rationale/Principle - Strategy - Support Effectiveness for Self-Hosted Users.md` (short or long form per Sam convention) | 0–1 | None–Low | Strategic WHY card; may mention the CLI verb in the "how a user reaches the service" summary. |
| **F6** `Principle - Narrow Forever (Roadmap as Commitment)` | `/rationale/Principle - Narrow Forever.md` | 0–1 | None–Low | Roadmap-as-commitment card. Likely only passing reference if any. |
| **C1** `Standard - Competitive Context` | `/rationale/Standard - Competitive Context.md` | 0–1 | None–Low | Competitive one-pager. May use CLI verbs in "intake URL shape" description. |
| **C4** `Decision - Who This Service Is For` | `/rationale/Decision - Who This Service Is For.md` | 0–1 | None–Low | Audience decision. May mention CLI path in defining "self-hosted Fabro users." |
| **A1** `Agent - Reporter` | `/product/Agent - Reporter.md` | **HIGH — 2–4+ occurrences** | **HIGH** | The Reporter persona explicitly runs the CLI. `fabro report` / `fabro debug` are central to defining the Reporter's action set. Expect mentions in the persona description, JTBD, and the "what the Reporter does" section. **Second-highest-load edit of the cascade, after F2.** |
| **A2** `Agent - Triager` | `/product/Agent - Triager.md` | 0–1 | None–Low | Triagers don't run the CLI. Any mention is probably contextual ("the Reporter ran `fabro report`, then the Triager…"). |
| **A3** `Component - Report` | `/product/Component - Report.md` | 1 | Low | Component card; one passing reference to the verb that produced the Report, if any. Spot-check confirms A3 references `fabro report` in its lifecycle/creation narrative. |
| **A4** `Component - Report Metadata Page` | `/product/Component - Report Metadata Page.md` | 0–1 | None–Low | Structural-only scope; may mention the CLI verb in the "how a Reporter reaches this page" context. |
| **A6** `Standard - Information Architecture` | `/rationale/Standard - Information Architecture.md` | 0–1 | None–Low | IA card. May mention the CLI in the "access path" section. |
| **A9** `Decision - Capability URLs over Auth` | `/rationale/Decision - Capability URLs over Auth.md` | 0–1 | None–Low | Auth decision. CLI verb may appear in the "how a Reporter gets a URL" context. |
| **A10** `Decision - No Content Dedup` | `/rationale/Decision - No Content Dedup.md` | 0 | None | Dedup decision; unlikely to cite CLI verb. |
| **A11** `Decision - 30-Day Retention Default` | `/rationale/Decision - 30-Day Retention Default.md` | 0 | None | Retention decision; unlikely to cite CLI verb. |
| **A5** (new draft) | `/product/System - Retention, Storage, and Key Scheme.md` | 0–1 | None–Low | Storage-layer card. Uses CLI verb if at all only in the "what produces the bundle" framing. |
| **A7** (new draft) | `/rationale/Standard - Interaction Patterns (CLI + Delete).md` | **Every occurrence is authoritative — draft with new verbs natively** | N/A | A7 is new; Sam drafts directly against `cli-interaction-patterns.md`'s new verb set. No retrofit. |
| **A8** (new draft) | `/rationale/Standard - Accessibility Policy.md` | Uses new verbs natively | N/A | Same — draft with new verbs. |
| **A12, A13, A14** (new drafts per Part 2) | (new) | Use new verbs natively | N/A | New cards. |

### Cascade Mechanics — Sam's Instructions

1. **Mechanical substitutions** — apply these as literal string replacements first, then re-read for contextual correctness:
   - `fabro debug` → `fabro dump` (command reference only; leave the noun "debug bundle" alone — it refers to the artifact, not the command)
   - `fabro report` (when used as a CLI command — i.e., in code fences or as "run `fabro report`") → `fabro dump --upload` (primary one-step path) OR `fabro upload` (two-step; contextual call)
   - **NOT CHANGING:** the product domain `fabro.report`; the noun "Report"; wikilink targets; `Report Metadata Page`; anything where "report" is used as an English noun (e.g., "the report expires after 30 days").

2. **Contextual-correctness pass** — for each replaced occurrence, verify:
   - If the replacement is `fabro dump --upload`, the surrounding prose should make sense with "one-step dump + upload" semantics.
   - If the replacement is `fabro upload`, the surrounding prose should make sense with "upload a previously-dumped tarball" semantics.
   - If the context describes "the user runs one command and gets a URL," use `fabro dump --upload`. If the context describes a staged upload (e.g., CI pipeline reuse, `--dry-run` pre-flight), use `fabro upload`.

3. **`Debug Bundle` noun is unchanged.** The artifact is still called a Debug Bundle. Only the command that produces it (`fabro dump`, formerly `fabro debug`) changed. A Debug Bundle remains a Debug Bundle.

4. **Reporter persona (A1) — special attention.** A1 has the Reporter running the CLI. Every reference to the CLI verb should move to `fabro dump --upload` as the primary path (this is the "efficient one-step" path Bryan confirmed). Secondary reference to `fabro upload` belongs only if A1 enumerates the two-step advanced path; if it does not, don't invent it.

5. **F2 — special attention.** Noun Vocabulary is the canonical-names card. If F2 currently lists CLI verbs as part of its canonical vocabulary, the list needs editing AND any prose that uses the old verbs needs updating. Expect this to be the single heaviest per-card edit.

6. **F3 — same pass, separately.** F3 needs the Sentry clarifying note (Part 3). Apply that edit in the same Sam pass as the CLI rename to minimize re-reviews.

### Cascade Ordering Recommendation

Apply the cascade as a single Sam pass, ordered from highest-load to lowest so Sam is most focused when the stakes are highest:

1. F2 `Standard - Noun Vocabulary` — canonical-name editing
2. A1 `Agent - Reporter` — heaviest persona-card rewrite
3. F1, F3, F4 — Foundation cards with CLI prose
4. A3 — Component - Report (lifecycle mentions)
5. F5, F6, C1, C4, A2, A4, A6, A9, A10, A11 — light-touch remaining edits
6. Draft A5, A7, A8 new cards — natively with new verbs, no retrofit needed
7. Draft A12, A13, A14 new cards — natively with new verbs
8. F3 Sentry clarifying note — in the same Sam pass, layered on top of the CLI rename

This ordering also coincides with "touch F2 first" discipline — every other card's CLI-verb edit should be auditable against F2's updated canonical list.

---

## Part 5 — Manifest Edits

### Edit A — New card entries (Part 2 additions)

Add three rows to the Tier 3 Amplifier table:

| # | Card | Type | Area | Zone | Build Order | Source Status | Intent |
|---|------|------|------|------|-------------|---------------|--------|
| A12 | `Decision - Rust on AWS Fargate over Bun on Fly` | Decision | 5.1 | `/rationale/` | 22 | EXISTS | Stack + deployment choice with rejected alternatives (Bun/Fly matches `bun.report` template; Rust/Fly matches Fabro; Rust/Fargate chosen for AWS-native IAM-role S3 access). |
| A13 | `Decision - Sentry for Service Errors, Not Bundle Processing` | Decision | 5.1 | `/rationale/` | 23 | EXISTS | Distinguishes the service's Sentry usage (operational errors) from the `bun.report`-style Sentry-as-bundle-processor pattern F3 rejects. Prevents a future cold-read contradiction. |
| A14 | `System - Service Topology` | System | 2.4 | `/product/` | 24 | EXISTS | Fargate task, Route53, S3 bucket, IAM role, Terraform. Operational topology, paired with A5's data topology. |

### Edit B — Update Source Status for A5 / A7 / A8

In the Tier 3 Amplifier table:

| # | Card | Source Status (was → is) |
|---|------|--------------------------|
| A5 | `System - Retention, Storage, and Key Scheme` | PARTIAL → **EXISTS** |
| A7 | `Standard - Interaction Patterns (CLI + Delete)` | PARTIAL → **EXISTS** |
| A8 | `Standard - Accessibility Policy` | ABSENT → **EXISTS** |

### Edit C — Update Blocked Cards table

Remove A5, A7, A8 from the "Blocked Cards" table. Remaining blocked cards are C2, C3, and F4 (flagged as PARTIAL pending tone source; confirm current state in Sam's next pass). C2/C3 remain blocked on visual direction — unchanged.

### Edit D — Update "Summary" line

`Summary:` at the top of the manifest currently says **"16 cards to build."** Update to:
> **Summary:** 19 cards planned. 5 Foundation (F1–F6), 4 Core (C1–C4), 10 Amplifier (A1–A11 + A12–A14 new), 0 Deprioritized. 2 cards remain blocked on source (C2, C3); 17 cards draftable or drafted.

### Edit E — No changes required to "Accepted Lint Warnings" table

The three new cards (A12, A13, A14) are:
- A12, A13 — Decision cards in `/rationale/`; no containment requirement. No new containment warnings expected.
- A14 — System card in `/product/`. Same containment status as A5 (existing System card). If A5 doesn't trigger `sweep4.missing_containment` (confirm via lint after A14 lands), A14 should pattern-match. If A5 does, and A14 also does, add A14 to the existing containment-warning acceptance row (same rationale as A3/A4/A5).

**Pre-land action:** Sam runs `ax lint` after A14 is drafted and reports back. If A14 generates a new warning shape not already accepted, Conan re-adjudicates before the manifest pins it as accepted.

### Edit F — Re-order Recommended Build Sequence

The current "Recommended Build Sequence (first 5 cards in order)" section is historical and describes Foundation. No re-ordering needed within Foundation.

Add a new subsection after the Foundation-build-sequence:

> **## Follow-On Build Sequence (2026-04-23 incremental pass)**
>
> | Order | Card | Why This Order |
> |---|---|---|
> | 17 | A5 `System - Retention, Storage, and Key Scheme` | Longest-blocked card; storage source unblocks it cleanly. Builds before A7/A8 because A7 references A5's URL format and A8 references A5's page surface. |
> | 18 | A7 `Standard - Interaction Patterns (CLI + Delete)` | Depends on A5 (presigned-URL architecture) and references F4 (tone). Build after A5. |
> | 19 | A8 `Standard - Accessibility Policy` | Two-surface (web + CLI); references A4, A7, C2 (C2 still blocked but A8 can be drafted referencing the blocked card's eventual shape). |
> | 20 | A12 `Decision - Rust on AWS Fargate over Bun on Fly` | Independent decision card; no blockers. Build anytime after F-tier + A5. |
> | 21 | A13 `Decision - Sentry for Service Errors, Not Bundle Processing` | Pair with F3 clarifying-note edit in the same Sam pass. |
> | 22 | A14 `System - Service Topology` | Builds last; cites A5, A12, A13 as dependencies. |
>
> **Before drafting any of these:** apply the CLI rename cascade across the existing 16 cards (Part 4). Drafts 17–22 then use the new verbs natively.

### Edit G — Update date header

Current manifest header: **Date: 2026-04-22**. Add a line below: **Last incremental update: 2026-04-23 (Inventory Pass 2 — tech stack / storage / CLI / accessibility sources absorbed).**

---

## Prioritized Next-Action List for Sam

Apply in this order, in a single coordinated pass:

1. **Apply the CLI rename cascade (Part 4) across F2, A1, F1, F3, F4, A3 first (high-load cards); then F5, F6, C1, C4, A2, A4, A6, A9, A10, A11 (low-load).** Do this before drafting any new cards — the new cards should use the new verbs natively, and the existing cards must be consistent with them before the cascade creates split-convention drift.

2. **Draft A5 `System - Retention, Storage, and Key Scheme`** from `storage-and-keys.md`. Use the manifest-identified anti-examples (sequential IDs, cron retention, mismatched lifecycle). Flag metadata-storage-location (sidecar JSON default) as an inline "pending Bryan confirmation" note.

3. **Draft A7 `Standard - Interaction Patterns (CLI + Delete)`** from `cli-interaction-patterns.md`. Uses new CLI verbs natively. Include presigned-URL architectural note; carries register-not-strings for errors (F4-calibrated).

4. **Draft A8 `Standard - Accessibility Policy`** from `accessibility.md`. Two-surface coverage (CLI `NO_COLOR` + web WCAG 2.2 AA + keyboard + SR). Notes `/ui` skill as implementation workflow.

5. **Draft A12 `Decision - Rust on AWS Fargate over Bun on Fly`** from `tech-stack-and-deployment.md`. Rejected-alternatives section is the load-bearing content.

6. **Draft A13 `Decision - Sentry for Service Errors, Not Bundle Processing`** from `tech-stack-and-deployment.md`. **In the same pass, add the F3 clarifying note** per Part 3 recommendation (c).

7. **Draft A14 `System - Service Topology`** from `tech-stack-and-deployment.md`. Cites A5, A12, A13. This is last because it's the integrator.

8. **Run `ax lint`** after all drafts land. Report back to Conan with any new warning shapes.

9. **Return to Conan for Spot-Check of A5, A7, A8** (unblocking-pass spot-check, same shape as the C1/A3/A4/A6 batch spot-check). Separate spot-check for A12/A13/A14 because they are a different tier of addition (new-territory cards, not unblocks).

---

## Manifest Edits — Ready to Apply

Conan can apply the manifest edits directly in this pass (Edits A–G are all mechanical: new rows, status-column updates, header changes). Sam need not touch `card-manifest.md` for these.

**Recommendation:** Conan applies Edits A–G in a single manifest update immediately, so Sam's next pass starts from a current manifest. Sam's pass then focuses on cards, not on manifest reconciliation.

**Alternative:** If the human prefers Sam to apply manifest edits (for consistency with "Sam is the only agent that writes library cards"), note that `card-manifest.md` lives in `docs/alexandria/inventory/`, not in `library/`. It is inventory metadata, not a library card. Conan-direct is the standard pattern for inventory/meta files.

---

## Open Flags for Human Judgment

1. **A12 title specificity.** `Decision - Rust on AWS Fargate over Bun on Fly` names all four axes in the title. Alternative: `Decision - Service Stack` (shorter) or `Decision - Rust on AWS Fargate` (drops the rejected-alternative from the title). Conan's recommendation: keep the long form — rejected-alternative-in-the-title is load-bearing for Factory-mode agents scanning Decision titles, and the title length matches A9/A10/A11 conventions.

2. **A13 title length.** `Decision - Sentry for Service Errors, Not Bundle Processing` is long. Alternative: `Decision - Sentry Scope` (thin). Conan's recommendation: keep the long form — the distinction IS the content; a thin title would re-create the cold-read ambiguity A13 exists to prevent.

3. **A14 pairing with A5.** A14 is a new System card alongside A5 in area 2.4. Consider: does this library benefit from a single System card (`System - Topology and Storage` merged) or two (A5 data + A14 operational)? Conan recommends two — the merger would overload one card with two concerns that have different change frequencies (storage layout is stable; Fargate task config evolves). Flag for human.

4. **Metadata storage (A5 Open Question 1).** Sidecar JSON vs object user-metadata vs DynamoDB. Raven recommends sidecar JSON. If Bryan has a preference, capture it before A5 drafts or A5 will state the recommendation as a flagged-default.

5. **AWS region** (flagged across tech-stack source and storage source). Not blocking any card draft, but worth a one-line confirmation so A14 can state it authoritatively rather than "assumed us-east-1 or us-west-2."

---

**Status: DONE_WITH_CONCERNS** — three new cards added to the manifest; CLI rename cascade inventoried; F3/Sentry tension resolved via belt-and-suspenders (both F3 note + new A13). Concerns: (1) cascade edit counts are estimates — Sam should confirm per-card load when starting the edit pass; (2) metadata-storage choice for A5 remains a flagged default pending Bryan confirmation; (3) `ax lint` must run after A14 to catch any new containment-warning shapes before manifest pins them as accepted.

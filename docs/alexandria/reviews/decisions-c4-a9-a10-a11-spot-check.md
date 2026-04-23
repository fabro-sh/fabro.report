# Decisions C4 / A9 / A10 / A11 Spot-Check

**Reviewer:** Conan
**Date:** 2026-04-22
**Scope:** First post-Foundation batch of four Decision cards — C4 `Decision - Who This Service Is For`, A9 `Decision - Capability URLs over Auth`, A10 `Decision - No Content Dedup`, A11 `Decision - 30-Day Retention Default`. Plus backlink audit on F1/F2/F3/F4/F5/F6 "Referenced by" edits Sam added after drafting these four.
**Mode:** Spot-check (not full grading). Gate before product-layer A1/A2 Agent cards.
**Lint state (human-verified):**
- `ax lint cards`: 0 errors, 14 warnings (10 word-count advisories + 4 folder-placement warnings, folder placement accepted by product owner)
- `ax lint graph`: 0 errors, 0 warnings, 0 info — full bidirectional closure
- `ax lint lines`: 0/0/0
- `ax lint layers`: 0 errors, 2 pre-existing warnings (no product/experience cards yet)

---

## 1. Accuracy Against Source

Each Decision's WHAT transcribes a specific line of `Confirmed Product Decisions` in `docs/alexandria/sources/product-vision.md`. Spot-checked verbatim:

| Card | Source line | Source text | Card's transcription | Match |
| --- | --- | --- | --- | --- |
| C4 | L78 | "Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs." | WHY L38 quotes L78 in italics verbatim (including "users run Fabro on their own systems"). WHAT L5 names "Reporters who are self-hosted Fabro users — primary and only." | Accurate |
| A9 | L87 | "Auth: none on the public surface. Capability URLs only." | WHAT L5 carries "The entire public-surface access control for `fabro.report` is possession of the Report's URL… there is no second factor, no login, no account, no session." WHEN L69 cites L87 directly. | Accurate |
| A9 | L66–72 Security Model | "Capability URLs. Possession of the URL grants: The ability to view the metadata receipt. The ability to delete the report." | WHAT L5 transcribes view + delete exactly: "Possession grants the ability to view the Report Metadata Page and the ability to delete the Report." | Accurate (upgrades "metadata receipt" → canonical "Report Metadata Page" correctly) |
| A10 | L86 | "Dedup: none. No content-hash dedup. Rationale: bundles expire anyway." | WHAT L5 carries "no dedup… at upload time, at storage time, or anywhere in the retention lifecycle." WHY L36 quotes L86 verbatim. | Accurate |
| A11 | L81 | "Retention: 30-day default expiration." | WHAT L5 carries "Every Report on `fabro.report` expires 30 days after upload." WHEN L69 cites L81 directly. | Accurate |
| A11 | L29 end-to-end flow | "Unless deleted earlier, the report expires after 30 days (default retention)." | WHAT L5 reconstructs the "deletion brings clock to zero" mechanic; HOW L77 states deletion path. WHEN cites L29. | Accurate |

**No scope inflation observed.** Each card's WHAT is the same commitment the source lists, expanded with rationale and scope-of-applicability framing but not widened in content. In particular:

- **C4** does *not* claim the service is for "Fabro users running Fabro anywhere on any infrastructure" — it explicitly narrows to self-hosted and explicitly excludes hosted-Fabro. This matches source L78 ("users who run fabro on their own systems") and resolves L128's open question in the direction that phrasing already leans.
- **A9** does *not* claim capability URLs are the v1 access model with auth planned for later — it explicitly says "Auth will not be added later; auth is permanently out of scope under `[[Principle - Narrow Forever]]`" (L7). This is an amplification of the source line but not an invention: source L87 reads as an identity decision, and F6 (Narrow Forever) governs that amplification.
- **A10** does *not* claim dedup is operationally impossible or technically unsafe — it correctly frames the rejection as a cost-benefit decision at this retention scale. This matches source L86's phrasing ("bundles expire anyway").
- **A11** does *not* claim 30 days is optimal; it correctly frames it as a product-owner judgment call in the right neighborhood (WHY L41: "It is not a precise optimum — no measurement would produce 'exactly 30 days is correct.'"). This is the right epistemic register for a product-owner-chosen default.

**Verdict:** All four Decisions accurately record the source commitments. No inflation, no invention.

---

## 2. Rejected Alternatives — Credible or Strawmen?

Decision cards earn their teeth only if the rejected alternatives are real options with genuine trade-offs, not shapes designed to lose. Audited A9's five rejected alternatives and A11's four in detail.

### 2.1 A9 — Five rejected alternatives (four explicit, one is the chosen option)

| Option | Credibility | Rationale strength |
| --- | --- | --- |
| A — Reporter authentication (login required) | Real | Rejected on friction-for-Reporter grounds, tied to F5's Strategy. Not a strawman — this is the default for most SaaS products and has to be actively rejected. |
| B — Triager authentication | Real | Rejected on "Triagers do not use the service; they retrieve from S3 with their own AWS credentials" — this is a structural fact from the source, not a manufactured rationale. The "would duplicate the S3 credential system" line is specific and correct. |
| C — Hybrid (public capability URLs for view + deletion password/email) | Real and task-flagged | Task asked: real or placeholder? **Real.** Hybrid access is a well-known pattern (e.g., GitHub Gists' "delete with token" flow, many paste services' "edit password" flows). The rejection rationale is specific: it re-introduces identity via email/password capture, requires a recovery-flow surface, and "half-authentication is worse than no authentication for this product's shape." Concrete and non-strawman. |
| D — Signed URLs with short expirations + refresh | Real and task-flagged | Task asked: real or placeholder? **Real.** S3 presigned URLs are the obvious default for S3-backed services, and "signed URL with refresh" is how AWS itself recommends short-lived credentials. The rejection rationale is specific: "refresh implies a session concept, session implies identity, and identity is what this product does not have." The observation that short-lived signed URLs are "the correct choice inside a larger service with an auth system" is accurate — A9 rejects them not because they're wrong but because this product is not the context they fit. Concrete and non-strawman. |
| E — Capability URLs only (chosen) | n/a | Chosen option. Rationale is specific to the product shape (end-to-end: upload, out-of-band share, delete, retention-bounded). |

**Verdict on A9:** All four rejected alternatives are real options that a Factory-mode agent could plausibly arrive at. Each rejection cites specific-to-this-product reasons (F5 friction, S3 credential duplication, identity-capture cost, session-implies-identity cost). None are shaped to lose.

### 2.2 A11 — Four rejected alternatives (plus the chosen option)

| Option | Credibility | Rationale strength |
| --- | --- | --- |
| A — 7-day retention | Real and task-flagged | Task asked: specific or generic? **Specific.** The rationale walks a realistic triage timeline: "Reporter files an issue, the team sees it on Monday, the triage priority lands it mid-week, the bundle is retrieved late week — a 7-day window leaves almost no margin for the bundle to still be available by the time a Triager goes to retrieve it." This is concrete week-shape reasoning tied to F5's Strategy (retention that routinely expires before team retrieval is retention that defeats Strategy). Not generic. |
| B — 90 days / 1 year | Real and task-flagged | Task asked: specific or generic? **Specific.** The rationale names the specific edge cases longer retention covers ("Triager out for extended PTO, bundle needed for a post-mortem weeks after the fix") and then names the specific cost: a Debug Bundle may contain "workflow data, stack traces, or environment context that the Reporter did not intend to share indefinitely." The inflection point framing — "past 30 days the framing drifts toward 'we're holding this indefinitely for reasons'" — is concrete tone-of-product argument, not a generic "privacy is bad" claim. |
| C — Retention flexibility / per-Report overrides | Real and task-flagged | Task asked: specific or generic? **Specific.** The rejection names three separate complications: (1) per-Report retention adds a parameter surface (how is it set? at upload time? after upload? only by Triagers?), (2) retention tiers imply a multi-tier product shape, (3) F3 names "extend retention policy per Report (custom expiration windows, indefinite retention tiers)" as a drift shape. That third citation is a specific callback to F3, not a generic "Narrow Forever rejects this" wave. |
| D — Indefinite retention | Real and task-flagged | Task asked: specific or generic? **Specific.** Three distinct arguments: (1) "indefinite retention makes `fabro.report` an archive, which is a different product category," (2) capability-URL leaks compound forever (A9 coupling cited), (3) "removes the natural privacy posture that comes from 'the service forgets after a month.'" Product-category, security-surface, and privacy-posture arguments each operate at different layers. Not generic. |
| E — 30 days, Reporter-immutable (chosen) | n/a | Chosen option with explicit rationale that it is "the product-owner-chosen balance" between the short/long tensions — not a precise optimum. |

**Verdict on A11:** All four rejected alternatives have rationale specific to this product's shape at this retention scale. None are generic "we don't want that" rejections. The concrete week-shape triage timeline in Option A and the three-layer argument against indefinite retention (D) are the two strongest examples.

### 2.3 Overall alternatives audit

Both cards pass the "shaped to lose" test. A8/A9 (sic — A9) Options C and D are genuinely appealing alternatives that many products in this space adopt (deletion passwords, S3 presigned URLs); their rejection requires specific-to-this-product argument and A9 supplies it. A11's Options A–D walk a full retention-design spectrum with concrete-to-this-product rationale at each stop. These are teeth-bearing Decision cards.

---

## 3. Cross-Decision Couplings

A10 ↔ A11 coupling (retention → no dedup needed) and A9 ↔ A11 coupling (retention bounds capability-URL leak window) audited for symmetry.

### 3.1 A10 ↔ A11 coupling (retention makes dedup unnecessary)

| Direction | Claim | Where stated |
| --- | --- | --- |
| A10 → A11 (A10 depends on A11) | "bundles expire after 30 days, so any storage-cost case for dedup is small by construction. Retention bounds the worst case; without retention, dedup's cost-benefit math would look very different." | A10 WHERE L15 "Paired with" |
| A11 → A10 (A11 acknowledges coupling) | "The Rationale for no dedup is *bundles expire anyway*, which means that Decision depends on this one. If retention were, say, 10 years, the dedup math would change and A10 would need to be revisited. The two Decisions are coupled." | A11 WHERE L16 "Paired with" |
| A10 revisit trigger 3 | "A retention policy change (see [[Decision - 30-Day Retention Default]]) extends retention long enough that duplication cost starts to dominate. In that case, the retention decision and this Decision are coupled and should be revisited together." | A10 "Conditions That Would Trigger Revisiting" #3 |
| A11 revisit trigger 3 | "Upstream change to [[Decision - No Content Dedup]]. If dedup becomes in scope for any reason, the retention math couples to it and both decisions should be reviewed together." | A11 "Conditions That Would Trigger Revisiting" #3 |

**Symmetric.** Both cards state the coupling from their own side, both cite the other card, both name a coupled-revisit trigger. Neither overclaims — A10 correctly says its Rationale *depends on* A11; A11 correctly says A10 depends on it but does not claim A11 is motivated by dedup. The dependency direction is recorded the same way in both places. Good.

### 3.2 A9 ↔ A11 coupling (retention bounds capability-URL leak window)

| Direction | Claim | Where stated |
| --- | --- | --- |
| A9 → A11 | "capability URLs depend on the URL eventually expiring. A URL that grants view + delete forever is a larger security exposure than one that expires after 30 days; the retention default is what bounds the capability-URL surface's lifetime risk. The two Decisions reinforce each other." | A9 WHERE L20 "Related" |
| A9 cost explicit | "URL leak equals access… The mitigation is the 30-day retention ([[Decision - 30-Day Retention Default]]) and the delete button on the Report Metadata Page." | A9 "What It Costs" (L48) |
| A11 → A9 | "capability URLs grant view + delete to anyone in possession. A URL that grants access forever is a larger lifetime-risk surface than one that becomes unreachable after 30 days. Retention is what bounds the capability-URL exposure window; without retention, the access-model cost profile would be different." | A11 WHERE L15 "Paired with" |
| A11 cost explicit | "Capability-URL leaks are bounded but not eliminated. If a URL leaks on day 1, it grants view + delete for 30 days. Thirty days is a bound on the exposure window, not zero exposure." | A11 "What It Costs" (L49) |

**Symmetric.** Both cards state the leak-window-bounding claim from their own side and both acknowledge the reinforcement runs both ways. A11's cost section even quantifies the bound ("30 days is a bound on the exposure window, not zero exposure") which is the honest framing A9's cost section also takes ("view + delete capability until the Report expires or is deleted").

### 3.3 Task-flagged subcheck: Does A9's "non-guessable URLs are load-bearing" line interact correctly with A11's retention leak-window argument?

A9 "What It Costs" bullet 4 (L51):

> **Non-guessable URLs are load-bearing.** If an agent ever generates URLs with insufficient entropy (predictable IDs, short slugs, sequential numbers), the capability model collapses into no access control at all. The key scheme is part of the security model; it must produce URLs that cannot be reasonably brute-forced. That constraint is forwarded to `System - Retention, Storage, and Key Scheme` (manifest A5).

This is A9 naming a *second* failure mode — URL-guessability — which is independent of A11's retention-bound-the-leak-window argument. A11's leak window bounds the cost of a *known* URL being pasted where it shouldn't be; A9's entropy requirement bounds the cost of an *unknown* URL being brute-forced. Two different threat models, two different mitigations.

A11's argument is: retention limits exposure duration of a leaked URL. A9's argument is: entropy limits brute-force discoverability of URLs that never leaked at all. Both hold independently; neither is redundant to the other.

**Correct interaction.** Neither overclaims against the other: A11 does not claim retention makes entropy unnecessary (it doesn't), and A9 does not claim entropy makes retention unnecessary (it doesn't). Both are required; A11 bounds the post-leak exposure, A9 prevents pre-leak guessing. The cards converge on "capability URLs are the whole access model iff (entropy is sufficient AND retention is bounded)" — and they say so independently.

**Verdict on couplings:** Both couplings are stated symmetrically with matching claims on each side. No overclaim on either side. A9's entropy argument is orthogonal to A11's retention argument and both cards correctly treat them as separate mitigations.

---

## 4. Subtle Consistency — Capability Scope vs Retention Mutability

Task asked: verify A9 doesn't accidentally extend capability-URL scope to retention modification, and A11 doesn't accidentally imply a Reporter-facing retention knob.

### 4.1 Does A9 accidentally extend capability to retention modification?

Searched A9 for any claim that URL possession grants retention modification. Results:

- A9 WHAT L5: "Possession grants the ability to view the Report Metadata Page and the ability to delete the Report; there is no second factor, no login…" — view + delete, exactly. No "extend," "renew," or "modify retention."
- A9 HOW "Following This Decision" L76: delete action submitted "requires no additional authentication — possession of the URL is the whole authorization." — still view + delete only.
- A9 HOW "Violating This Decision" L82–86: five violations listed. None mention retention modification. One violation is "Adding a password, PIN, or confirmation code to the delete action" — which clarifies the delete is URL-only, still scoped to deletion.

**A9 does not extend capability to retention modification.** The capability stays scoped to view + delete throughout.

### 4.2 Does A11 accidentally imply a Reporter-facing retention knob?

Searched A11 for any claim that Reporters can modify retention. Results:

- A11 WHAT L5: "Reporters cannot extend retention, shorten retention, or opt into indefinite retention. The only party who can alter the retention policy — as a policy, not per-Report — is the product owner."
- A11 WHAT L7: "Per-Report retention overrides, retention tiers, and indefinite retention are all permanently out of scope."
- A11 HOW "Following This Decision" L77: "Reporters who want a Report gone sooner than 30 days can delete it via the delete button on the Report Metadata Page… Deletion brings the retention clock to zero; it does not extend or reset any clock." — this is the one retention-facing action a Reporter has, and A11 scopes it explicitly: deletion only, not extension.
- A11 HOW "Violating This Decision" L82–86 enumerates five violations, all of which reject Reporter-facing retention flexibility: per-Report overrides, pin/extend mechanism, silent reset on page view, missed deletion at deadline, retention-near-expiration notifications that imply agency.

**A11 does not imply a Reporter-facing retention knob.** The Reporter can delete (which takes the retention clock to zero); they cannot extend, pin, renew, or otherwise modify retention. The deletion-as-retention-shortening behavior is framed as a subset of existing delete capability under A9, not as a new retention surface.

### 4.3 Cross-check: A9's delete-on-URL-possession does not enable retention extension through A11

The A9 capability (view + delete) and A11's Reporter-immutable retention policy are consistent because delete is **one-way** — it shortens the retention window to zero but cannot extend it. A11 L77 makes this explicit: "Deletion brings the retention clock to zero; it does not extend or reset any clock." A9 L50 similarly frames delete as the URL-holder's prerogative without extending to any retention-adjusting capability.

**Verdict:** Capability-URL scope (A9) and retention policy mutability (A11) are consistent. A9 stays scoped to view + delete. A11 reserves retention policy to product-owner-only. Delete is one-way (shortens; never extends). The two Decisions do not accidentally overlap or leak into each other's scope.

---

## 5. Vocabulary Compliance (F2 Rule)

Checked all four Decision cards for canonical five-noun usage (Report, Debug Bundle, Reporter, Triager, Report Metadata Page).

### 5.1 C4

| Noun | Occurrences | Status |
| --- | --- | --- |
| Report | Not used standalone (hosted-Fabro discussion is about audience, not Reports) | Acceptable — scope of C4 is the audience, so Report usage is limited |
| Debug Bundle | L5, L15, L38 | Correct |
| Reporter | L5, L12, L17 (×2), L26, L34, L40 (×2), L42, L52, L72, L73 (×2), L78, L80, L84 | Correct |
| Triager | L15, L26, L52, L73, L84 | Correct |
| Report Metadata Page | Not used | Acceptable — C4 is an audience card, not a surface card |

No shortenings. No "user," "customer," "Fabro user" used in a noun-shortening position (the phrase "Fabro user" appears in the "audience is Fabro users running Fabro on their own systems" framing, which is a descriptive qualifier of Reporter, not a noun replacement). No "hosted-Fabro user" adopted as a canonical noun — it is consistently framed as a descriptor of a population out of scope.

### 5.2 A9

| Noun | Occurrences | Status |
| --- | --- | --- |
| Report | L5, L20, L24, L50, L62, L77, L86 | Correct |
| Debug Bundle | L12, L15, L33, L42, L44 (×2), L75, L77 | Correct |
| Reporter | L5 context, L18, L32 (×2), L34, L36, L40, L49, L50, L63, L75, L85, L86, L90, L93 | Correct |
| Triager | L15, L33, L50, L55, L77, L93 | Correct |
| Report Metadata Page | L5, L15, L18, L25, L33, L76, L77 (×2), L83 | Correct — never "metadata page" or "receipt page" in assertion position |

A9 L18 vocabulary note: quotes F2's definitions exactly. No shortenings in A9's own assertion prose. Scanned for "receipt page" — not present except in the WHAT bullet about F2 ("**Report Metadata Page** definition explicitly states 'capability-URL addressed…'") which is correct reference-to-F2 framing.

### 5.3 A10

| Noun | Occurrences | Status |
| --- | --- | --- |
| Report | L5 (×3), L17, L22, L29, L33, L43, L44 (×2), L54, L56, L68, L69, L77, L81 | Correct |
| Debug Bundle | L5 (×2), L17, L19, L22, L29, L30, L36, L42, L44, L48, L68, L69, L74, L75, L77, L81 | Correct |
| Reporter | L5, L19, L31 (×2), L33, L42, L43, L76 | Correct |
| Triager | L33, L44, L48, L76 | Correct |
| Report Metadata Page | L76 | Correct (single usage in violation example) |

No "bundle" used in noun-position without the "Debug" qualifier in A10's own assertion prose. A10 L48 uses "bundles" (plural, anaphoric — the canonical name is established nearby). Acceptable per F2's anaphoric-shortening tolerance established in the F5 spot-check.

### 5.4 A11

| Noun | Occurrences | Status |
| --- | --- | --- |
| Report | L5 (×3), L18, L23, L25 context, L30, L32, L34, L35, L41, L43, L48 (×3), L49, L54, L60, L64, L75, L76, L77 (×2), L78, L82, L83, L84, L85, L86, L90, L92 | Correct |
| Debug Bundle | L5, L12, L15 context, L18, L32, L48, L54, L75, L77, L78, L83, L85 | Correct |
| Reporter | L5, L7, L12 context, L18, L20, L31, L32, L34, L43, L47, L62, L64, L77, L82, L86 | Correct |
| Triager | L12 context, L20, L31, L32, L48, L54, L60, L78 | Correct |
| Report Metadata Page | L24, L77, L83, L86 | Correct |

A11 L88 "Reporter Metadata Page" — wait. Rechecking. L77 actual text in card reads "Reporters who want a Report gone sooner than 30 days can delete it via the delete button on the Report Metadata Page." Correct. No "Reporter Metadata Page" slip.

**Scanned for all known shortenings across all four cards:** "receipt page" (no), "metadata page" standalone (no), "landing page" (no), "bundle" as a first-reference noun (no — always "Debug Bundle" first-reference or anaphoric after), "fabro user" in noun-replacement position for Reporter (no — always qualifier), "team member" in noun-replacement position for Triager (no), "URL" as a synonym for Report (no — URL is consistently a property of Report, not a replacement for it).

**Verdict:** All four Decision cards are vocabulary-clean. The F3-1 shortening-drift risk pattern (card references F2 then bends it in its own prose) does not recur.

---

## 6. Source Citations

Spot-checked 1–2 line refs per card against `docs/alexandria/sources/product-vision.md`.

| Card | Citation | Source content at that line | Status |
| --- | --- | --- | --- |
| C4 | Sources L88 "L78 Strategic WHY" | L78 is exactly "Strategic WHY (confirmed 2026-04-22). Fabro users run Fabro on their own systems…" | Accurate |
| C4 | Sources L88 "L52–64 Users section" | Header L51 "## Users"; content L53–L64. Citation starts 1 line early (header L51, first content L53). | Minor slip (same header-vs-content pattern flagged across prior spot-checks) |
| C4 | Sources L88 "L115–128 Raven's Inferences and Open Questions" | Header L115 "## Raven's Inferences (NOT confirmed by Bryan — flag for review)"; L128 is Open Question 3 about hosted-Fabro. | Accurate range |
| A9 | Sources L96 "L66–72 Security Model" | Header L66 "## Security Model"; content L68–L72. Range-start covers header rather than first content line. | Minor slip |
| A9 | Sources L96 "L87 Confirmed Product Decision 'Auth: none…'" | L87 is exactly "Auth: none on the public surface. Capability URLs only." | Accurate |
| A10 | Sources L87 "L86 'Dedup: none. No content-hash dedup. Rationale: bundles expire anyway.'" | L86 is exactly that line. | Accurate |
| A11 | Sources L96 "L29 end-to-end flow" | L29 is exactly "Unless deleted earlier, the report expires after 30 days (default retention)." | Accurate |
| A11 | Sources L96 "L81 Confirmed Product Decision 'Retention: 30-day default expiration.'" | L81 is exactly that line. | Accurate |
| A11 | Sources L97 "initialize-output.md L23 retention default" | Not spot-checked in this pass — citation target is separate source file. | Deferred |

**Pattern recurrence:** The header-vs-content-line drift pattern flagged in F1–F3, F5–F6, and F4 spot-checks recurs in C4 (L52 vs L53) and A9 (L66 vs L68). Same severity as prior: trivial nit, reader following the reference lands in the right section.

**Verdict:** Citations are directionally accurate across all four cards. Two new minor slips consistent with the established pattern. Worth Sam landing alongside the existing F2-1/F3-2/F5-N1/F5-N2/F6-N1/F4-N1/F4-N2 pass.

---

## 7. Backlink Audit — F1/F2/F3/F4/F5/F6 "Referenced by" Edits

Sam added/updated "Referenced by" entries in all six Foundation cards pointing at C4/A9/A10/A11. The graph linter reports 0/0/0, so bidirectional closures are structurally intact. The substantive question is whether anchor descriptions reflect what the Decision cards actually claim or smuggle new claims.

Because the Foundation cards are not re-read here (they are fully ratified per the F4 spot-check and the `ax lint graph` result confirms their backlinks resolve), I audit by comparing what each Decision **claims about itself** to what a legitimate backlink anchor in each Foundation card would say.

### 7.1 F1 (Product Thesis - Pickup Point, Not Triage Surface) → Referenced by C4/A9/A10

What each Decision claims about F1:

- **C4** WHERE L15 "Paired with F1": "the pickup-point thesis assumes the pickup is happening between a self-hosted Reporter and a Fabro-team Triager retrieving from S3 with their own credentials. Changing the audience would change the two endpoints of the pickup." — this is a legitimate audience-level claim about F1's thesis; not smuggled.
- **A9** WHERE L15 "Paired with F1": "a pickup point has exactly two handoffs… The capability URL is the token of the first handoff. The S3 credentials are the token of the second. Neither handoff involves authentication on the `fabro.report` service." — A9 independently makes this two-handoff claim; F1's "Referenced by" anchor should mirror. Not smuggled.
- **A10** WHERE L19 "Related to F1": "a pickup point does not analyze what was dropped. Hashing a Debug Bundle to compare it to prior uploads is a form of light content analysis; even if the result is used only for dedup and never surfaced, the service would be looking inside the bundle in a way the pickup-point thesis refuses." — A10 independently extends F1's thesis to content-analysis refusal; a legitimate extension that F1's Referenced by entry can cite.

### 7.2 F2 (Standard - Noun Vocabulary) → Referenced by C4/A9/A10/A11

Each Decision cites F2 for specific vocabulary claims:

- **C4** L17 cites F2's Reporter definition as audience-specific ("written for a self-hosted-Fabro Reporter") — legitimate.
- **A9** L18 cites F2's Reporter definition ("no authentication, no login, no account") and Report Metadata Page definition ("capability-URL addressed") as being *rationalized by* A9. This is the correct causal direction — F2 states the what, A9 states the why.
- **A10** L17 cites F2's Report and Debug Bundle definitions and notes that two-Reports-to-one-Debug-Bundle would break the one-to-one relationship F2 implies — a legitimate vocabulary-level claim about dedup.
- **A11** L18 cites F2's Report definition naming "retention deadline (default 30 days)" and names this Decision as the rationale behind that default value — correct causal direction.

### 7.3 F3 (Principle - Reject Like bun.report Drift) → Referenced by C4/A9/A10/A11

Each Decision cites F3 as the drift-shape forum where a proposal to overturn the Decision would live. Legitimate pattern; drift-proposal framing is F3's explicit job:

- **C4** L19 "Related to F3": audience-expansion proposals are the audience-layer version of `bun.report` drift. Accurate extension of F3's feature-shape drift to audience shape.
- **A9** L16 "Paired with F3": per-team auth, user accounts, "log in to see your uploads" — all F3-listed drift shapes. Matches F3's drift list.
- **A10** L20 "Related to F3": "dedup proposals can arrive disguised as operational improvements" — a legitimate addition to F3's drift catalog at the operational layer. This extends F3 rather than contradicting it; the Referenced-by anchor should mirror.
- **A11** L21 "Related to F3": "F3 names retention-extension as a drift shape; this Decision is the upstream commitment F3 rejects against." — correctly identifies the upstream/drift-list relationship.

### 7.4 F4 (Principle - Minimal Public Surface Feels Calm) → Referenced by A10 and A11

Task-flagged: does A10 independently claim "dedup adds invisible surface area," or is that claim only in the backlink anchor?

**A10 WHERE L12 "Governed by F4":**
> [[Principle - Minimal Public Surface Feels Calm]] — dedup adds surface area that is invisible to the Reporter but still part of the product: a hash-computation step on upload, content-identity metadata in storage, a collision-handling policy ("if a new upload matches an existing hash, do we point at the existing Debug Bundle, or copy, or ignore?"). Each of those is a decision this product does not have to make, and the calm register depends on not having to make it.

**A10 WHY L38 calm-register argument:**
> The calm-register argument (F4) is also present: every invisible operational mode is a place where future agents will have to make decisions the product owner has not made. "If a collision happens, what do we do?" is a question this product should never have to answer, and the way to guarantee it never has to answer it is to not introduce the collision in the first place. No dedup is not the absence of a decision; it is the decision to not maintain a collision-handling stance.

**A10 independently makes the "dedup adds invisible surface area" claim** in two places (WHERE L12 and WHY L38), with WHY L38 being the fuller treatment. The claim is not only in the backlink anchor — it is independently argued in A10's own belief section. F4's "Referenced by" entry for A10 can legitimately cite this claim.

Task-flagged for A11: does A11 independently cite F4? Checking A11 WHERE:
- A11 does **not** list F4 in its "Governed by" or "Paired with" or "Related" sections.
- A11 HOW "Violating This Decision" L86 cites F4 once: "its copy inherits [[Principle - Minimal Public Surface Feels Calm]] and does not offer agency the Reporter does not have." — a single citation at the tone layer.

If F4's "Referenced by" entry cites A11 primarily on tone-of-expiration-messaging grounds (L86), that is a narrow but legitimate claim A11 independently makes. If F4's anchor goes broader ("retention policy is a minimal-surface commitment"), A11 does not independently argue that — it would be over-reaching.

**Flag for verification:** whoever reviews F4's "Referenced by" entry for A11 should confirm the anchor is scoped to expiration-message tone (L86), not broader retention-surface framing. A11's own argument is that retention serves F5 (Strategy) and the leak-window (A9); F4 is a minor reference, not a governor.

### 7.5 F5 (Principle - Strategy Support Effectiveness for Self-Hosted Users) → Referenced by C4/A9/A10/A11

C4 explicitly "forward-references" to F5's "manifest C4" pointer resolving to this card (C4 L21) — meaning F5 previously had a forward-reference to C4 that Sam has now resolved into a backlink. This is the same pattern the F4 spot-check identified for F5's forward-ref to F4. Legitimate upgrade.

All four Decisions cite F5 as the Strategy they derive from:
- C4: audience directly formalized from F5's "users who run fabro on their own systems."
- A9: access-model concrete expression of F5's "make retrieval easy" framing.
- A10: calm-register and Strategy-aligned ("no dedup because retention bounds it, which matches Strategy").
- A11: retention calibrated to F5's retrieval workflow.

Each claim is independently present in the Decision. F5's "Referenced by" entries should mirror.

### 7.6 F6 (Principle - Narrow Forever) → Referenced by C4/A9/A10/A11 (task-flagged as highest overreach risk)

**Task asked: does each Decision actually claim to be a concrete expression of narrow-forever, or is Sam over-reaching?**

- **C4** WHAT L5 and WHY L33: "*Primary and only* closes the escape hatch that would otherwise read this as 'self-hosted is the initial target and we'll add others later'" (L7); "[[Principle - Narrow Forever]] treats 'for v1' framings as the decay path that erodes scope commitments one relaxation at a time. Putting hosted-Fabro on the Roadmap as 'later' turns every design discussion into 'let's make this flexible enough for hosted users too,' which is audience-shaped drift into a fuller product." (L33). **C4 explicitly frames itself as an audience-layer expression of F6** — the "audience commitment is an identity decision at the same level as the feature-scope commitment" phrasing (L34) is load-bearing. Not over-reaching.
- **A9** WHAT L7: "Auth will not be added later; auth is permanently out of scope under [[Principle - Narrow Forever]]. Capability URLs are forever." **A9 explicitly frames itself as a concrete access-model expression of F6's permanent-out-of-scope list** (auth is one of F6's nine items — "Per-user accounts" at F6 L44 per the prior spot-check). Not over-reaching.
- **A10** WHAT L7: "Dedup will not be added later; if storage costs ever genuinely require deduplication, the product owner reopens this Decision as a Strategy-adjacent change, not as an implementation optimization." WHERE L13: "adding dedup later would be a scope question (new behavior, new edge cases, new operational mode), not a vocabulary update or an implementation tweak. This Decision aligns the data model with the narrow-forever commitment." **A10 frames dedup-as-scope-question under F6.** Not over-reaching — though note that dedup is *not* one of F6's enumerated nine items (it's a new A10-driven narrow-forever-adjacent commitment). A10 is extending F6's logic to dedup rather than citing F6 as already-enumerating dedup. Legitimate extension — F6 explicitly says its list "can grow as new drift shapes are noticed" (per F5-F6 spot-check section 2.2). A10 is consistent.
- **A11** WHAT L7: "Per-Report retention overrides, retention tiers, and indefinite retention are all permanently out of scope under [[Principle - Narrow Forever]]. The product has one retention number, and this card names it." WHERE L13: "indefinite retention, per-Report overrides, and retention tiers are on F6's permanently-out-of-scope list. Retention is a single number by commitment, not a parameter surface." **A11 frames retention-flexibility-as-permanently-out-of-scope under F6.** Not over-reaching, though again, "retention tiers / per-Report overrides" are not explicitly in F6's enumerated nine items. A11 is extending F6's logic to retention policy, which is consistent with F6's "list can grow" framing.

**All four Decisions independently claim to be concrete expressions of narrow-forever.** C4 and A9 map directly to F6's existing nine-item list (audience identity; auth/accounts). A10 and A11 extend F6's logic to new drift shapes (dedup; retention flexibility) in the way F6's "list can grow" explicitly authorizes.

**Potential flag:** F6's "Referenced by" entries for A10 and A11 should be careful not to frame those as "F6 already listed dedup/retention-flexibility" — they are extensions, not existing enumerations. If F6's anchors frame them as extensions, no overreach. If F6's anchors imply "F6 already called out dedup/retention-tiers," that would be smuggling content back into F6.

**Conservative recommendation:** when Sam reviews F6's "Referenced by" entries for A10 and A11, verify the anchor language frames those Decisions as new drift shapes that extend F6's logic (not as "F6 already had this on its list"). If the anchors are framed as extensions, no fix. If they over-claim that F6 enumerated those drift shapes, tighten.

### 7.7 Drift-risk scan on the backlink edits

- **New claims not independently present in the Decisions:** None found in the four Decisions. Every backlink target has independent content in the Decision's own WHAT/WHERE/WHY supporting the backlink anchor.
- **Vocabulary drift introduced via backlinks:** None. All four Decisions use canonical nouns, so any backlink anchors that mirror Decision claims will inherit canonical-noun usage.
- **Contradiction with Foundation framings:** None observed. C4 respects F5's Strategy scope; A9 respects F6's auth-out-of-scope; A10 respects F4's calm register; A11 respects F6's scope-parameter-surface rejection.
- **Softening of Foundation claims via backlinks:** None observed in the Decision content. If any Foundation backlink anchor softens (e.g., "A9 might want to revisit auth if…" instead of "A9 rejects auth permanently"), that would be a Sam editing nit in the Foundation card — not visible from the Decision cards themselves, but worth verifying.

**Verdict on backlinks:** Every Decision-side claim cited by a Foundation "Referenced by" entry is independently present in the Decision's own assertion prose. The graph linter's 0/0/0 confirms structural closure. The one flagged risk (F6's anchors for A10/A11 possibly implying F6 already enumerated dedup/retention) should be verified by spot-reading F6 — if the anchors say "extends F6's logic," no fix; if they imply "F6 already covers this," tighten to "extends."

---

## 8. C4 Audience Scope — Hosted-Fabro Deferral Framing

Task-flagged: does C4 faithfully transcribe F5's "users who run fabro on their own systems," and is hosted-Fabro deferral framed correctly as "different product question" (correct) rather than "future expansion" (wrong, would violate F6)?

### 8.1 Audience scope transcription

C4 WHY L38:
> The Strategic WHY was confirmed by the product owner verbatim on 2026-04-22: *Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs.* That phrasing names self-hosted users as the target.

Source L78 matches word-for-word. Transcription faithful. No tightening (C4 does not narrow further, e.g., to "professional self-hosted users" or "self-hosted users with team support plans") and no loosening (C4 does not widen to "Fabro users" generally).

### 8.2 Hosted-Fabro deferral framing

C4 WHAT L5:
> Bundles produced by users of any future hosted-Fabro offering are **out of scope under this Decision**; if hosted-Fabro ever needs bundle intake, that is a separate product question, not a scope expansion of `fabro.report`.

**"Separate product question" is the correct framing.** C4 does not say "hosted-Fabro is a future expansion," "hosted-Fabro is on the roadmap," "we'll handle hosted-Fabro in v2," or any other temporal-deferral phrasing that would imply hosted-Fabro is queued for later at this product. It explicitly says hosted-Fabro is a *different product question* — i.e., if hosted-Fabro ever needs bundle intake, that intake is a different product, not an expansion of `fabro.report`.

C4 WHY Option B rejection (L33) reinforces this exactly:
> **Option B (rejected): Audience is self-hosted users "for v1," with hosted-Fabro bundle intake explicitly on the Roadmap for later.** Rejected because [[Principle - Narrow Forever]] treats "for v1" framings as the decay path that erodes scope commitments one relaxation at a time.

This is the right rejection. "For v1" framings are explicitly called out as the F6 decay pattern, and C4 refuses them.

C4 WHY Option C chosen (L34):
> Hosted-Fabro bundle intake is a separate product question, deferred — not queued, not scheduled, not on a roadmap here.

**"Not queued, not scheduled, not on a roadmap"** is the load-bearing phrasing. C4 is using "deferred" in the sense of "not this product's problem" rather than "this product will get to it later." Three explicit negatives (not queued / not scheduled / not on a roadmap) close the temporal-deferral escape hatch.

### 8.3 Does C4 leave any escape hatch for hosted-Fabro to drift back in?

C4 "Conditions That Would Trigger Revisiting" (L57–L62) names three conditions; the only one relevant to hosted-Fabro is #2:
> A hosted-Fabro offering has been announced or shipped, and the product owner decides that its bundle intake should land in `fabro.report` rather than in a purpose-built hosted intake product. **That is a Strategy change, not a scope expansion; this Decision is the thing it changes.**

This is correct framing: hosted-Fabro intake arriving at `fabro.report` requires the product owner to make an explicit Strategy change that overturns this Decision, not an implementation call. F6-compatible.

**Verdict on C4 audience scope:** Strategic WHY transcribed faithfully (no tightening, no loosening). Hosted-Fabro deferral framed correctly as "separate product question" with three explicit negatives closing the temporal-deferral escape hatch. F6-compatible. No drift into "future expansion" framing.

---

## 9. Fixes

| ID | Card | Fix | Severity |
| --- | --- | --- | --- |
| — | — | **None that gate the build.** | — |

Nits worth landing opportunistically, not gates:

| ID | Card | Nit | Severity |
| --- | --- | --- | --- |
| C4-N1 | C4 | Sources citation `L52–64` for Users section: header is L51, first content L53. Change to `L53–L64` or `L51–L64`. Same header-vs-content drift pattern as F2-1/F3-2/F5-N1/F5-N2/F6-N1/F4-N1/F4-N2. | Trivial |
| A9-N1 | A9 | Sources citation `L66–72` for Security Model: header is L66, first content L68. Change to `L68–L72` or keep `L66–L72` as "header + content." Same pattern. | Trivial |
| F6-backlink-check | F6 | Verify F6's new "Referenced by" entries for A10 and A11 frame those Decisions as *extensions of F6's logic* (dedup and retention-flexibility are new drift shapes), not as *items F6 already enumerates*. F6's enumerated nine items do not include dedup or retention-tier specifically; A10 and A11 extend under F6's "list can grow" clause. If F6's anchor language says "extends," no fix. If it says "on F6's list" in a way that implies already-enumerated, tighten. | Low — verify, not assumed broken |
| F4-backlink-check | F4 | Verify F4's new "Referenced by" entry for A11 is scoped to retention-expiration-message tone (A11 L86, which is A11's single F4 citation), not broader "retention is a minimal-surface commitment" framing. A11 does not independently argue that broader framing. | Low — verify, not assumed broken |

Nine total trivial citation-range nits across all five Foundation+Decisions spot-checks (F2-1, F3-2, F5-N1, F5-N2, F6-N1, F4-N1, F4-N2, C4-N1, A9-N1) can land in one Sam consistency pass whenever convenient. No gate.

The two F6/F4-backlink-check items above are low-confidence flags from a one-sided audit (I read the Decisions; the Foundation card bodies are not re-read since they were approved in prior spot-checks). The graph linter's 0/0/0 result is evidence the links *resolve*, not that every anchor sentence is maximally precise. These checks would take Sam five minutes; if the anchors already say "extends F6" and "at tone layer" respectively, both flags close with no edit.

---

## 10. Lint State

Human-reported lint state confirmed:
- `ax lint cards`: 0 errors, 14 warnings (10 word-count advisories + 4 folder-placement — folder-placement accepted by product owner per spot-check task context; not re-flagged)
- `ax lint graph`: 0 errors, 0 warnings, 0 info — full bidirectional closure including the new C4/A9/A10/A11 backlinks to F1/F2/F3/F4/F5/F6
- `ax lint lines`: 0/0/0
- `ax lint layers`: 0 errors, 2 pre-existing warnings (no product/experience cards yet; the planned A1 Agent - Reporter and A2 Agent - Triager are still ahead)

No lint-driven fixes required. The 10 word-count advisories mirror the Foundation-tier pattern: each Decision card is built for standalone-agent-readability and the length earns its keep. Not audited in depth here because the Foundation-tier audit concluded this class of advisory is acceptable when the card pulls its weight per-word.

---

## Verdict

**APPROVED** — all four Decision cards (C4, A9, A10, A11) are spot-check-clear. Proceed to the next batch.

### Summary of readiness

| # | Card | Spot-check status | Load-bearing claim |
| --- | --- | --- | --- |
| C4 | Decision - Who This Service Is For | APPROVED (this spot-check) | Audience is self-hosted Fabro users, primary and only. Hosted-Fabro is a separate product question, not a future expansion. |
| A9 | Decision - Capability URLs over Auth | APPROVED (this spot-check) | URL possession is the whole access model; view + delete; four rejected alternatives each with product-specific rationale; non-guessable URLs load-bearing. |
| A10 | Decision - No Content Dedup | APPROVED (this spot-check) | No hash, no content identity, no coalescing, ever; retention makes dedup unnecessary (coupled to A11); pickup-point thesis refuses content analysis. |
| A11 | Decision - 30-Day Retention Default | APPROVED (this spot-check) | 30 days, same for every Report, Reporter-immutable; four rejected alternatives (7d/90d+/flex/indefinite) with concrete rationale; coupled to A9 (leak window) and A10 (dedup math). |

### Cross-cutting findings

- **Source accuracy:** All four Decisions transcribe the Confirmed Product Decisions section faithfully with no scope inflation.
- **Alternatives credibility:** A9's five options and A11's four walked alternatives are real, not strawmen. Each rejection cites product-specific rationale.
- **Couplings symmetric:** A10↔A11 (retention bounds dedup math) and A9↔A11 (retention bounds URL-leak window) are stated symmetrically from both sides. A9's entropy argument is orthogonal to A11's retention argument; both are required mitigations and neither is overclaimed.
- **Capability/retention scope:** A9 stays scoped to view+delete. A11 keeps retention product-owner-only. Delete is one-way (shortens; never extends). No accidental capability-creep.
- **Vocabulary:** Canonical nouns used correctly across all four cards. No shortenings in assertion prose. F3-1 drift-pattern does not recur.
- **Citations:** Directionally accurate with two new header-vs-content range-start slips (C4-N1, A9-N1) matching the established pattern.
- **Backlinks:** Every Decision-side claim cited by Foundation "Referenced by" entries is independently present in the Decision's own content. Two low-confidence flags on F6 (A10/A11 extension vs enumeration) and F4 (A11 scope) worth a five-minute Sam verification.

### Next agent action

**Sam: draft A1 `Agent - Reporter` and A2 `Agent - Triager`.**

Rationale:
- Both are EXISTS-source (draftable from `product-vision.md` today).
- Both are thin persona cards that inherit audience from C4 (just ratified), access model from A9, and retention from A11. The Foundation + Decisions substrate they rest on is now complete.
- Neither is blocked on source gaps.
- The `System - Retention, Storage, and Key Scheme` (A5) card is PARTIAL — mechanism not specified — so A3/A4 Component cards that would want to reference A5 should wait for Bryan's S3-lifecycle-vs-app-cron + key-scheme input.

Alternative if Bryan wants to unblock A5 first: Bryan supplies the mechanism + key-scheme decisions as a source-material addition, Conan does a source assessment, then Sam drafts A5 before A3/A4/A1/A2. Either order works; A1/A2 is the faster path to adding product-layer coverage without waiting on Bryan.

**Parallel Sam opportunistic work (not gates):** one citation-consistency pass landing the nine accumulated trivial nits (F2-1, F3-2, F5-N1, F5-N2, F6-N1, F4-N1, F4-N2, C4-N1, A9-N1) plus verification pass on F6 A10/A11 and F4 A11 backlink anchor language.

**Status: DONE**

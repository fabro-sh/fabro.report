# Agents A1 / A2 Spot-Check (First Product-Layer Batch)

**Reviewer:** Conan
**Date:** 2026-04-22
**Scope:** A1 `Agent - Reporter`, A2 `Agent - Triager`. Plus backlink audit on the six Foundation cards (F1/F2/F3/F4/F5/F6) + two Decision cards (C4/A9) Sam updated to close the graph around A1/A2.
**Mode:** Spot-check (not full grading). First `/product/` cards. Gate before the next draftable batch (C1 + A3 + A4 + A6).
**Lint state (human-verified):**
- `ax lint cards`: 0 errors, 16 warnings (14 pre-existing accepted + 2 new word-count A1/A2 advisories)
- `ax lint graph`: 0/0/0 — full bidirectional closure
- `ax lint lines`: 0/0/0
- `ax lint layers`: 0 errors, 2 pre-existing warnings (experience/ empty, product/ has only A1/A2 — expected)

---

## 1. Asymmetric-Identity Model — Both Sides, Honestly Carried?

**Yes.** The asymmetry is the load-bearing structural property of the pair, and both cards state their own side *and* name the other side in the same paragraph where the asymmetry is established.

| Asymmetric property | A1 Reporter side | A2 Triager side |
| --- | --- | --- |
| Identity model | WHAT L5: "The Reporter has no identity on this service — no account, no login, no user record — and is identified only by the Fabro version captured as metadata at upload time." | WHAT L5: "the Triager never interacts with the `fabro.report` service itself. They retrieve Debug Bundles directly from S3 using their own AWS credentials, not through any endpoint, API, dashboard, or login on this service." |
| Access vector into Debug Bundle | WHAT L5: "Their entire interaction with the service is the `fabro report` CLI: one command, one Debug Bundle upload, one short URL returned on stdout." (Note: Reporter *cannot* access bundle contents through the service at all — confirmed HOW L59.) | WHAT L5: "The only piece of the service a Triager ever touches is the same short URL a Reporter shared with them — out-of-band, in a GitHub issue or Slack or email — from which they copy the Report ID and use it to locate the Debug Bundle in S3." |
| Cross-reference to the other endpoint | WHERE L14 "Counterpart role" entry: "the asymmetry between them — no-identity Reporter on the service, credentialed Triager retrieving from S3 — is load-bearing for the product shape. A Reporter never becomes a Triager within this service; the two roles are structurally separate." | WHERE L12 "Counterpart role" entry: "The asymmetric-identity model is load-bearing: a Reporter has no identity on the service and interacts with the CLI only; a Triager has full AWS credentials and interacts with S3 only. The two roles sit on opposite sides of the service and neither is ever both." |

Both cards name the asymmetry *as an asymmetry* (not just "Reporter has no account" and separately "Triager uses AWS credentials"). Both name the other side's property when stating their own. Both close the "can a Reporter become a Triager" escape hatch — A1 explicitly ("A Reporter never becomes a Triager within this service; the two roles are structurally separate"), A2 explicitly ("The two roles sit on opposite sides of the service and neither is ever both"). Symmetric framing from both sides.

**Verdict:** Asymmetric-identity model honestly carried on both cards. The pair reads as one structural commitment expressed from two viewpoints, not two independent role sketches that happen to reference each other.

---

## 2. A2's "Never Interacts with the Intake Service Itself" — Load-Bearing?

**Yes. A2 carries this clause in exactly the right place with exactly the right reinforcement pattern.**

### 2.1 Position: WHAT, early, not buried

A2 WHAT L5 names it inside the load-bearing framing sentence — not after a buildup, not in a subclause, not in WHERE:

> The Triager's single most counterintuitive property — and the one most likely to be eroded by a builder agent working from a "bug-intake service" intuition — is this: **the Triager never interacts with the `fabro.report` service itself.**

Bolded. Named as *the single most counterintuitive property*. Flagged as *the one most likely to be eroded*. This is the tripwire placement that signals to a Factory-mode agent reading A2 in isolation: this is the property you cannot let slip.

### 2.2 Reinforcement across WHERE / WHY / HOW

The clause appears in distinct forms across the card, each operationalizing a different failure mode:

| Location | Form of the clause | What it defends against |
| --- | --- | --- |
| WHERE L10 "Identity and vocabulary" | "**never interacts with the intake service itself**… the service has no Triager-facing surface" — named as vocabulary-level, not implementation-level | Drift via vocabulary (if F2 loosens, A2 loosens) |
| WHERE L16 "Scope commitment — this role is the single most load-bearing case of it" | "The service exists for Triagers' benefit and gives them no surface, and that is not a v1 simplification — it is the product's identity." | Drift via "v1 only" relaxation |
| WHERE L18 "Scope coherence" (F1 pairing) | "no part of the team's investigation workflow depends on an endpoint of this service. Any card that describes a Triager-facing surface is evidence the thesis has been abandoned." | Drift via downstream Triager-facing endpoints |
| WHY L30 (Thesis) | "If Triagers had a surface here, this service would be a triage product with an intake front-door rather than a pickup point" | Drift via implicit category shift |
| WHY L33 "Driver" | "Without this card, downstream product-layer cards risk inventing a Triager-facing surface by implication: a key scheme that assumes team listing, a retention policy that assumes team visibility, a metadata field that only makes sense on a dashboard." | Drift via downstream-card implication |
| HOW "What a Triager Does Not Do" L55–L60 | Six negative bullets, each beginning with "Does **not**" in bold. Login, dashboard, listing, filter, symbolicate/render, notifications — all permanently refused. | Drift via feature enumeration |
| HOW Anti-Example 1 L69 | "A card describes a 'Triager dashboard' that lists recent Reports. This is the single most common drift vector this card names and is permanently out-of-scope by [[Principle - Narrow Forever]]." | Drift via dashboard proposal |
| HOW Anti-Example 2 L70 | "An endpoint is added to `fabro.report` that accepts AWS credentials and returns a bundle (a 'convenience wrapper' over S3)." | Drift via "convenience wrapper" framing |

Eight reinforcements across WHAT/WHERE/WHY/HOW, each operationalizing a different drift vector. Not one is a repeat of another; each adds a specific failure-mode that a Factory-mode agent could plausibly arrive at.

### 2.3 The "incidental capability" carve-out

HOW L51 names the *one* case where a Triager does touch the service — URL-possession-based delete — and frames it explicitly as *not* a Triager-specific privilege:

> Optionally, if the Reporter shared the URL with them and asks for the Report to be deleted, the Triager can visit the Report Metadata Page and use the delete button. **This is the only way a Triager ever touches the service, and it is an incidental capability borrowed from URL possession — not a privileged Triager operation.**

This is the right handling. Without this carve-out, a future agent could point at Example 2 (L65) and say "see, Triagers do use the service." With it, the carve-out is framed as URL-possession (A9) incidentally exposing a surface to anyone holding the URL, not as a Triager-targeted feature. Example 2 itself (L65) reinforces the framing: "This interaction uses the Triager's URL possession as the authorization, not any Triager-specific privilege — any Reporter in possession of the URL could have done the same."

**Verdict:** "Never interacts with the intake service itself" is load-bearing throughout A2. Placement, reinforcement, and carve-out are all calibrated correctly. This is the property A2 was drafted to defend, and the card defends it.

---

## 3. JTBD Shape — Role / Goals / Context / Constraints / Anti-Patterns

Both cards carry the five expected JTBD dimensions. Outcome specificity is the dimension most likely to degenerate into generic "use the product successfully" language, so it earns its own spot-check.

### 3.1 A1 Reporter JTBD dimensions

| Dimension | Where in A1 | Specificity |
| --- | --- | --- |
| Role identity | WHAT L5 ("a self-hosted Fabro user who hit a bug and ran `fabro report`") | Tight — audience scope (self-hosted), trigger (ran CLI), state (hit a bug) all named |
| Goals / outcomes | WHAT L5: "get the Debug Bundle to the Fabro team, get the URL into the right place (GitHub issue, Slack, email), and get back to the work the bug interrupted." | **Specific, not generic.** Task-required outcome phrasing ("get the Debug Bundle to the Fabro team fast, get back to work") is present: "simple and time-bounded" frames the outcome, and the three-step outcome chain (team-has-bundle / URL-shared / back-to-work) matches the task requirement exactly. |
| Context / when-active | HOW "Context: When a Reporter Arrives" L46 | Specific — "already been interrupted by a failure, already lost context on whatever they were actually trying to do, and are looking to spend the minimum possible time on the reporting path" |
| Constraints | HOW "What the Reporter Does" L50–L54 — five concrete actions, each with specific behavior | Specific — each action is a named activity (runs CLI, reads success output, copies URL, optionally visits page, returns to work) |
| Anti-patterns | HOW "Anti-Examples" L72–L75 — four wrongs, each with rationale + linked upstream card | Specific — each anti-pattern names a concrete feature shape (feature that presumes return-to-read-contents, "user" framing, recover-my-URL flow, post-upload feedback prompt) |

### 3.2 A2 Triager JTBD dimensions

| Dimension | Where in A2 | Specificity |
| --- | --- | --- |
| Role identity | WHAT L5 ("a member of the Fabro team (currently Bryan + team) who retrieves Debug Bundles for bug investigation") | Tight — team scope (Fabro team), current composition (Bryan + team), purpose (bug investigation) all named |
| Goals / outcomes | WHY L31 "every Triager-side design choice is checked against 'does this help the team retrieve the Debug Bundle?'" + HOW "Context: When a Triager Acts" L43: "The Triager's job is then to get from that URL to the Debug Bundle, reproduce the bug, and fix it." | **Specific, not generic.** Task-required outcome phrasing ("get the right Debug Bundle fast so they can reproduce and fix the bug") is present: the "get from that URL to the Debug Bundle, reproduce the bug, and fix it" chain matches the task requirement. Not "manage support tickets" or "handle bug reports." |
| Context / when-active | HOW "Context: When a Triager Acts" L43 | Specific — "prompted into action by seeing a `fabro.report` URL in a channel the Fabro team monitors — a GitHub issue filed against Fabro, a Slack message in a team or support channel, an email" |
| Constraints | HOW "What a Triager Does" L47–L51 — five concrete actions | Specific — each action names a concrete step (sees URL, extracts Report ID, retrieves from S3 with own credentials, works with bundle in own environment, optionally deletes) |
| Anti-patterns | HOW "Anti-Examples" L69–L73 — five wrongs, each with rationale + linked upstream card | Specific — each anti-pattern names a concrete drift shape (dashboard, convenience S3 wrapper, Triager definition widening, login flow, "team member of fabro.report" framing) |

### 3.3 Outcome specificity — both cards avoid the degeneration to generic

A1 L5 closes with: "*simple and time-bounded: get the Debug Bundle to the Fabro team, get the URL into the right place (GitHub issue, Slack, email), and get back to the work the bug interrupted.*" This is a three-clause outcome chain — transit the bundle, share the URL, resume interrupted work. Not "use `fabro.report` successfully."

A2 L43 closes with: "*The Triager's job is then to get from that URL to the Debug Bundle, reproduce the bug, and fix it. Each of those three steps happens outside this service.*" This is also a three-clause chain — URL-to-bundle, reproduce, fix. Not "manage bug reports" or "review support tickets." The "each of those three steps happens outside this service" sentence is a load-bearing reinforcement of section 2 (never-interacts) applied to the outcome itself.

**Verdict:** Both cards carry all five JTBD dimensions. Outcome phrasing specifically matches the task requirement — A1 avoids "use the product successfully" and A2 avoids "manage support tickets." The outcome chains are three-clause, concrete, and scoped to what the product actually affords.

---

## 4. A1 Emotional State — Does It Tie to F4?

**Yes. Load-bearingly and in exactly the place F4 is calibrated against.**

### 4.1 A1's emotional-context framing

HOW "Context: When a Reporter Arrives" L46 is the single paragraph doing this work, and it does it as the card's first HOW subsection — not buried, not in an appendix:

> The Reporter is not a product-evangelist persona; they arrive because something failed. A self-hosted Fabro user was using Fabro to run a workflow, Fabro broke, and the user ran `fabro debug` to get a tarball and then ran `fabro report` to upload it. By the time the Reporter sees the CLI output, they have already been interrupted by a failure, already lost context on whatever they were actually trying to do, and are looking to spend the minimum possible time on the reporting path so they can either (a) resume their original work, (b) share the URL with the Fabro team and wait, or (c) try a workaround. The CLI's output shape is calibrated to that emotional state — this is why [[Principle - Minimal Public Surface Feels Calm]] specifies *efficient and reassured* and not, say, *welcoming* or *thorough*.

Three properties of this paragraph:

1. **Frustrated-by-prior-bug state is stated.** "not a product-evangelist persona; they arrive because something failed." Not "a Fabro user who wants to share information" or "a user interested in helping." The frustration is the entry condition.
2. **Interruption framing is stated.** "already been interrupted by a failure, already lost context on whatever they were actually trying to do" — this is the F4-load-bearing premise. F4's "efficient and reassured" is calibrated against exactly this emotional state.
3. **The tie to F4 is explicit and directional.** The paragraph closes with "this is why [[Principle - Minimal Public Surface Feels Calm]] specifies *efficient and reassured* and not, say, *welcoming* or *thorough*." A1 explains *why* F4 reads as it does by pointing at the Reporter's emotional state. This is the correct causal direction — F4's tone choice is justified by the Reporter's state, and A1 is the card that holds that state.

### 4.2 Reinforcement in WHERE

WHERE L18 "Tone of the interaction" makes the same tie:

> [[Principle - Minimal Public Surface Feels Calm]] — the emotional goal for the Reporter is **efficient and reassured**. The Reporter is already frustrated by the bug that sent them to `fabro report`; the CLI and the Report Metadata Page must confirm the upload cleanly and step out of the way. Warmth that slows the handoff is worse than no warmth at all.

"Already frustrated by the bug that sent them to `fabro report`" is a direct claim about the Reporter's emotional state at service-entry time, and it is framed as the explanation for F4's register choice. This is the passage that would have been most likely to drift into generic "wants efficient handoff" framing — and it didn't. Frustration is named as state, not as preference.

### 4.3 F4 ↔ A1 causal direction

F4's L29 "Referenced by" entry reads:

> [[Agent - Reporter]] — the Reporter is the audience of every user-facing string this Principle governs; "efficient and reassured" is calibrated specifically to the Reporter's emotional state (already frustrated by the bug that sent them to `fabro report`) as described in that Agent card.

A1's L18 (quoted above) reads in the mirror direction: A1 holds the emotional state and F4 derives register from it.

Both cards attribute the register *to* the emotional state, and both name the same emotional state (frustrated-by-prior-bug). F4 does not claim to own the emotional state; A1 does not claim to own the register. Causal direction is clean.

**Verdict:** A1 carries the Reporter's emotional state load-bearingly. Tie to F4 is explicit, directional, and reciprocally mirrored in F4's Referenced-by entry. The emotional state is framed as the justification for F4's specific register choice, which is exactly what the task required.

---

## 5. Vocabulary Compliance (F2 Rule)

Both cards use canonical five-noun usage (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) in assertion prose.

### 5.1 A1 canonical-name usage

| Noun | Occurrences | Status |
| --- | --- | --- |
| Report | L5, L16, L27 (×2), L28, L30, L50 (forward ref A3/A4 context), L53, L67, L68 (×2), L72 | Correct |
| Debug Bundle | L5 (×3), L27, L28, L50, L53, L59, L67, L68, L72, L73 | Correct |
| Reporter | L5 (×4) and throughout — card subject | Correct |
| Triager | L14, L62, L67, L73 | Correct |
| Report Metadata Page | L5, L16, L18, L28, L53, L67, L68, L72, L75 | Correct |

Scanned for shortenings and drift shapes:
- "user" as noun-replacement for Reporter: L73 Anti-Example 2 explicitly **rejects** this drift ("Treating the Reporter as 'the user'… This product has no 'user' concept — only Reporters (who have no identity) and Triagers"). The word "user" appears in A1 as either (a) a qualifier on "Fabro user" in audience-descriptor framing (matches Standard rule), or (b) in the Anti-Example forbidding the shortening. Never as a first-class noun replacement for Reporter.
- "receipt page" / "landing page" / "dashboard": none in assertion prose.
- "bundle" without "Debug" qualifier: L67 uses "bundle" anaphorically after establishing "Debug Bundle" nearby ("a GitHub issue with a one-line description of the bug. They close the terminal… The Fabro team picks up the URL in the issue, extracts the Report ID, and retrieves the Debug Bundle from S3"). Acceptable per F2's anaphoric-shortening tolerance established in the F5 and prior spot-checks. L68 uses "Debug Bundle" at first reference, then stays with it.
- "upload" as synonym for Report: not present — "upload" used only as a verb or as a qualifier of the Report (e.g., "upload timestamp," "past uploads"), never as a noun replacement.

### 5.2 A2 canonical-name usage

| Noun | Occurrences | Status |
| --- | --- | --- |
| Report | L5, L12, L14, L24, L25, L51, L57, L58, L60, L64, L65, L69 | Correct |
| Debug Bundle | L5 (×2), L12, L14, L17, L19, L21, L25, L26, L43, L49, L50, L59, L64, L70 | Correct |
| Reporter | L5, L12, L17, L22, L23 context, L24 context, L43, L51, L57, L60, L65 (×2), L71, L73 | Correct |
| Triager | L5 (×5) and throughout — card subject | Correct |
| Report Metadata Page | L5, L51, L65, L69 | Correct |

Scanned for shortenings and drift shapes:
- "team member" as noun-replacement for Triager: L73 Anti-Example 5 explicitly **rejects** this drift ("A CLI string, a card, or a log line refers to a Triager as a 'team member of fabro.report' or 'maintainer.'"). The phrase "team member" appears in A2 as either (a) a descriptor of what Triager *is* ("a member of the Fabro team" at WHAT L5), which matches the F2 definition exactly, or (b) in the Anti-Example forbidding the shortening. Never as a replacement noun.
- "maintainer" / "responder": rejected in the Anti-Example above; not used elsewhere.
- "dashboard" / "team dashboard": appears only in negative framing (L16, L56, L69, and the "Does not visit a team dashboard" negative bullet) — never as a proposed surface.
- "bundle" without "Debug" qualifier: A2 is mostly disciplined. L50 uses "Debug Bundle"; L50 contains "untar it, inspect its contents" where "it" is anaphoric to Debug Bundle. L51 uses "Debug Bundle." L64 uses "`aws s3 cp s3://fabro-reports-bucket/<key-derived-from-Report-ID> ./bundle.tar.gz`" — the `bundle.tar.gz` filename is a shell command artifact (filename, not noun usage), and L64 uses "Debug Bundle" in the prose immediately. Acceptable — shell filename literal is not a vocabulary assertion.
- "credentials" for AWS: used correctly throughout; not drifted into "Triager account" or "Triager credentials on the service."

### 5.3 Vocabulary notes for both cards

Both cards reference F2 as the authoritative definition source (A1 L10, A2 L10) and explicitly position F2's definitions as *what this card expands* — not as *what this card invents or revises*. This is the correct framing.

**Verdict:** Both cards are vocabulary-clean. The F3-1 shortening-drift risk pattern (card references F2 then bends it in its own prose) does not recur. Both anti-example sections include explicit rejections of the most likely shortening drifts for each role ("user" for Reporter, "team member of fabro.report" for Triager), which turns each card into a tripwire against its own vocabulary drift.

---

## 6. Source Citations

Spot-checked the cited ranges against `docs/alexandria/sources/product-vision.md`:

| Card | Citation | Source content at that range | Status |
| --- | --- | --- | --- |
| A1 | Sources L79 `L52–L59 Users section → Reporter definition` | L52 "Two roles, with asymmetric identity:" (preamble). L54 "### Reporter" (subsection header). L55–L59 content (five bullets: Fabro user, no auth, Fabro version metadata, optional CLI message, share URL out-of-band). | **Accurate** — range starts at preamble L52 and covers Reporter subsection through L59 content. |
| A1 | Sources L79 `L22–32 end-to-end flow` | L22 "The flow, end-to-end:"; L24–L29 six-step numbered list; L31 metadata-receipt line ("The URL itself resolves to a minimal one-page metadata receipt…"). L32 is blank. | Accurate — covers flow numbered list plus trailing framing. |
| A1 | Sources L79 `L74–88 Confirmed Product Decisions — identity, auth, dedup` | L74 "## Confirmed Product Decisions" header. L78 Strategic WHY. L87 Auth. L86 Dedup. L88 bundle-content-rendering. | Accurate — range covers header + the identity/auth/dedup decisions named in the citation. |
| A1 | Sources L79 `L92–L98 Confirmed Entities — Reporter at L98` | L92 "## Confirmed Entities" header (blank L91). L96 Report. L97 Debug Bundle. L98 Reporter. L99 Triager. L100 Report Metadata Page. | **Minor slip** — citation says "Reporter at L98" and L98 is indeed Reporter. Range ends at L98, truncating before Triager and Report Metadata Page; this is intentional since A1 cites this range *for the Reporter definition only*. Acceptable. Header `L92–L98` matches the established header-vs-content pattern across prior spot-checks. |
| A2 | Sources L77 `L61–L64 Users section → Triager definition` | L61 "### Triager" (subsection header). L62 "A member of the Fabro team. Currently: Bryan + team." L63 "**Does not interact with the intake service itself.** They retrieve bundles directly from S3 using their own AWS credentials." L64 "The service's public surface is effectively write-only-from-CLI plus a metadata receipt; triage happens entirely out-of-band." | **Accurate** — range starts at subsection header L61 (includes header + content L62–L64). |
| A2 | Sources L77 `L22–32 end-to-end flow, step 5 Triager retrieval path` | L28 is step 5 ("A Fabro team member ('Triager') uses their own AWS credentials to retrieve the bundle directly from S3 for investigation."). | Accurate — L28 is exactly the Triager retrieval step cited. |
| A2 | Sources L77 `L74–89 Confirmed Product Decisions — Triager access path via direct S3 with own credentials at L89` | L89 exactly: "Triager access path: direct S3, using triager's own credentials. Not through this service." | Accurate — L89 is exactly the Triager-access-path decision cited. |
| A2 | Sources L77 `L92–L99 Confirmed Entities — Triager at L99` | L99 is exactly "**Triager** — a Fabro team member who retrieves bundles via direct S3 access." | Accurate — L99 is exactly the Triager entity cited. |

**Pattern observation:** the header-vs-content-line drift pattern flagged across F1–F6 spot-checks (F2-1, F3-2, F5-N1, F5-N2, F6-N1, F4-N1, F4-N2, C4-N1, A9-N1) does **not** recur in A1/A2. Both cards cite ranges that either (a) start at a header inclusively (A2 L61–L64 includes the `### Triager` header at L61 as part of the referenced section) or (b) include preamble + header + content (A1 L52–L59 includes the preamble at L52, the header at L54, and content L55–L59).

This is the correct shape for these citations because the Reporter/Triager definitions *depend* on the asymmetric-identity preamble at L52 — citing only L55–L59 for Reporter would strip the asymmetric framing, which is load-bearing for both cards. Sam's choice to include the preamble is correct. Same logic applies to A2's L61–L64 including the subsection header.

**Verdict:** Citations are accurate. No new header-vs-content slips introduced. A1's inclusion of the L52 preamble in the Reporter range is a deliberate improvement over the "content-only" pattern because the preamble carries the asymmetric-identity framing both cards need.

---

## 7. Sam's Flagged Convention Questions

### 7.1 No parent Domain link — acceptable for Agents?

**Yes, acceptable as written. Conan-approved for this batch.**

The manifest (`card-manifest.md` L34) lists `Component - Report`, `Component - Report Metadata Page`, `Component - Debug Bundle` as the planned Component cards, and `Domain - fabro.report Intake Service` is forward-referenced in F1 L30 but not manifested as its own card. A1 and A2 are the first `/product/` cards drafted; there is no parent Domain card to link to yet.

Three reasons this is acceptable:

1. **Agent cards are role-shaped, not surface-shaped.** Agents describe *who* interacts with the product, not *what surfaces* they interact with. A Domain card would aggregate Components, Systems, and surface areas — it does not aggregate Agents. The natural parent for an Agent is the Product Thesis (which names the role in product-shape terms) and the Noun Vocabulary Standard (which defines the role as an entity), and both A1 and A2 link up to F1 and F2 load-bearingly.
2. **The Domain card, when drafted, will naturally reference these Agents, not vice-versa.** If `Domain - fabro.report Intake Service` is eventually drafted (and the manifest does not commit to it — F1 L30 calls it a forward reference "planned in manifest… not yet drafted"), it will describe the intake service as a product and list the Agents that interact with it. A1 and A2 are the upstream definitions; the Domain card is a downstream aggregation.
3. **F1 already performs the Domain-card role at the Thesis level for this product.** F1 is a Plank-tier Thesis and functionally carries the domain-level framing for `fabro.report` ("the entire public product is a CLI-driven intake…"). A separate Domain card is not load-bearing for a product this small.

**Recommendation:** Do not block A1/A2 on a missing Domain card. When the next Component cards (A3, A4) are drafted, re-evaluate whether a Domain card is load-bearing. If A3/A4 can stand alone against F1/F2/F5/F6 without a Domain parent, the Domain card can stay deferred or be omitted entirely. The manifest should be updated (during Downstream Sync) if the Domain card is formally omitted.

### 7.2 First `/product/` folder — structural expectations?

**No folder README or index.md needed at this scale.**

Sam's folder choice — `docs/alexandria/library/product/agents/` — matches the canonical type-to-folder mapping (`Agent` → `agents/` subfolder under the zone root). This is the correct structural shape. The `ax lint layers` result (0 errors, 2 pre-existing warnings for empty experience/ and near-empty product/) confirms the linter accepts the placement.

Three reasons no folder README / index is needed:

1. **The manifest is the index.** `docs/alexandria/inventory/card-manifest.md` already lists every planned card with type, zone, and manifest ID. Adding a folder-level index duplicates the manifest without adding retrieval value for agents.
2. **Wikilinks are the navigation.** Each card's WHERE section enumerates the cards it references and is referenced by. A folder README would be redundant with the graph — and the graph is machine-checked by `ax lint graph`, which no README is.
3. **Factory × Low × Low scale.** For a 22-area library with 16 planned cards, folder-level scaffolding is overhead that buys nothing. This convention can be revisited if the library grows past ~30 cards and folder navigation becomes a retrieval bottleneck, but for now the graph + manifest + zone folders are sufficient.

**Recommendation:** Proceed without folder READMEs. Canonical type-to-folder mapping stands as the only structural convention. If a future product-layer folder has ambiguous placement (e.g., an Artifact that spans two types), flag at that point.

---

## 8. Backlink Edits Audit

Graph linter reports 0/0/0, confirming structural closure. The substantive question is whether the "Referenced by" anchor text smuggles claims beyond what A1/A2 actually say.

### 8.1 F6 "single most load-bearing out-of-scope item" anchor for A2 — overclaim?

**Checked. Not an overclaim — A2 independently makes this claim with matching strength.**

F6 "Referenced by" entry for A2 (F6 L25):
> [[Agent - Triager]] — Triager-facing UI (dashboards, per-team scoping, listing endpoints, any in-service triage surface) is the single most load-bearing permanently-out-of-scope item; the Triager role's "never interacts with the intake service itself" shape is the structural commitment this Principle exists to defend.

A2's own claim about F6 is at WHERE L16 ("Scope commitment — this role is the single most load-bearing case of it") — note the section header itself names the claim:
> [[Principle - Narrow Forever]] — Triager-facing UI (a team dashboard, a list of recent Reports, a filter by Fabro version, a triage UI, any Triager-facing endpoint) is on the **permanently-out-of-scope list**. The service exists for Triagers' benefit and gives them no surface, and that is not a v1 simplification — it is the product's identity. This is the single property most likely to be eroded by a builder agent's assumption that "of course the team needs a dashboard."

A2 says *the single most load-bearing case of [narrow-forever]*. F6's backlink anchor says *the single most load-bearing permanently-out-of-scope item*. Same claim, same strength, both from the structural side.

Cross-check: does F6 enumerate Triager-facing UI as one of the nine items on its permanently-out-of-scope list? F6 L51 (bullet 7) is "**Triage UIs.** No Triager-facing screens of any kind on the intake service. Triagers do not interact with the service itself." F6 enumerates it; A2 claims it is the most load-bearing of those items; F6's backlink reflects that claim. The "most load-bearing" framing is A2's independent assessment (not a pre-existing F6 enumeration), but F6's anchor carries it cleanly.

### 8.2 F1 backlinks for A1/A2 — generic, or substantive?

**Substantive. Both anchors describe what A1/A2 actually carry.**

F1 "Referenced by" entry for A1 (F1 L27):
> [[Agent - Reporter]] — the drop-off endpoint of the pickup; interacts with the service only via `fabro report` CLI and optionally the Report Metadata Page. The Reporter's narrow shape is the thesis expressed at the role layer on the upload side.

F1 "Referenced by" entry for A2 (F1 L28):
> [[Agent - Triager]] — the pickup endpoint; retrieves the Debug Bundle from S3 with their own AWS credentials and never interacts with the intake service. The Triager's no-service-surface shape is the structural condition that makes "pickup, not triage" coherent.

A1's own claim about F1 is at WHERE L20:
> [[Product Thesis - Pickup Point, Not Triage Surface]] — the Reporter drops a Debug Bundle at the pickup point and leaves. They are not an audience for in-browser rendering, symbolication, or bundle inspection.

A2's own claim about F1 is at WHERE L18:
> [[Product Thesis - Pickup Point, Not Triage Surface]] — the Triager is the picker-upper. The thesis says "the service's job ends at 'URL exists,'" which structurally requires that no part of the team's investigation workflow depends on an endpoint of this service.

F1's anchors describe each Agent as one side of the pickup handoff (drop-off endpoint / pickup endpoint). Each anchor names a specific property of the Agent (CLI-only / own-AWS-credentials). The framing "the thesis expressed at the role layer on the upload side" (A1) and "the structural condition that makes 'pickup, not triage' coherent" (A2) both describe what the Agent cards actually do for the Thesis — they are the role-layer expression of it. Not generic.

### 8.3 C4 forward-reference resolutions

C4 "Referenced by" entries for A1/A2 (C4 L24, L25):
> [[Agent - Reporter]] — inherits the self-hosted-only audience from this Decision; the Reporter role as written describes a Fabro user running Fabro on their own systems, not a hosted-Fabro user or a generic "anyone with a Debug Bundle" Reporter.
> [[Agent - Triager]] — the Triager's retrieval model (own AWS credentials, direct S3 access) is also tied to the self-hosted audience shape; widening the Triager definition (e.g., to third-party support partners, or to hosted-Fabro team members) is an audience change under this Decision.

A1's own claim about C4 is at WHERE L12:
> [[Decision - Who This Service Is For]] — the Reporter described here is a **self-hosted Fabro user, primary and only**. A hosted-Fabro offering's users are out of scope under that Decision; this card inherits that audience shape and does not describe a generic "anyone with a Debug Bundle" Reporter.

A2's own claim about C4 is at WHERE L23:
> [[Decision - Who This Service Is For]] — the audience is self-hosted Fabro users (Reporter side); the Triager side is the Fabro team as it exists today, currently Bryan + team, with no role hierarchy or Triager subtypes. Widening the Triager definition (e.g., to third-party support partners, or to hosted-Fabro team members if that offering ever exists) is an audience change under that Decision, not an implementation change here.

C4's anchor for A1 cites the same "generic 'anyone with a Debug Bundle'" phrasing A1 uses (L12 and L20). C4's anchor for A2 cites the same "third-party support partners / hosted-Fabro team members" expansion examples A2 uses (L23). Anchors mirror the Agent cards' own framing exactly. Not smuggled.

### 8.4 A9 backlinks for A1/A2 — access model framing

A9 "Referenced by" entries for A1/A2 (A9 L23, L24):
> [[Agent - Reporter]] — the capability URL returned by `fabro report` is the Reporter's entire access surface to the service after upload; there is no login, no "my uploads" view, no recovery flow — the Reporter role as written is the role-layer expression of this Decision.
> [[Agent - Triager]] — Triagers do not "unlock" the Debug Bundle through the URL; they use AWS IAM for S3 retrieval and URL possession only to optionally delete a Report on the Reporter's request. The Triager role's no-service-auth shape is the team-side expression of this Decision.

A1's own claim about A9 is at WHERE L16:
> [[Decision - Capability URLs over Auth]] — the short URL returned by `fabro report` is the Reporter's **entire access surface** to the service after upload. Possession of the URL grants view of the Report Metadata Page and the ability to delete the Report. There is nothing else the Reporter can do on this service; there is no login, no "my uploads" view, no recovery flow.

A2's own claim about A9 is at WHERE L14:
> [[Decision - Capability URLs over Auth]] — the capability URL the Reporter shares is a view+delete capability, not a retrieval capability. Triagers do not "unlock" the Debug Bundle through the URL; they locate the Debug Bundle via the Report ID encoded in the URL and retrieve it from S3 with their own AWS identity. The service has no access-control layer for Triagers because Triagers have no request path through the service. AWS IAM is the Triager's access-control layer, and it is orthogonal to this service.

A9's anchors use exact phrases from each card ("no 'my uploads' view, no recovery flow" for A1 L16; "Triagers do not 'unlock' the Debug Bundle through the URL" and "AWS IAM" for A2 L14). Anchors are quoted-from-card, not inferred-beyond-card. Mirror-match.

### 8.5 F2 forward-reference resolutions

F2 "Referenced by" entries for A1/A2 (F2 L19, L20):
> [[Agent - Reporter]] — must use "Reporter"; not "User," "Uploader," "Submitter." The Agent card expands the canonical definition into role context, goals, and anti-patterns, but inherits the exact wording here.
> [[Agent - Triager]] — must use "Triager"; not "Team Member," "Maintainer," "Responder." The Agent card expands the canonical definition into role context including the load-bearing "never interacts with the intake service itself" property recorded here.

A1's own claim about F2 at WHERE L10:
> [[Standard - Noun Vocabulary]] — the canonical **Reporter** definition (no identity, no login, identified only by Fabro version metadata; may optionally attach a short message via CLI flag; expected to share URL out-of-band) is the authoritative definition this card expands; any drift between the two is a defect in one of them.

A2's own claim about F2 at WHERE L10:
> [[Standard - Noun Vocabulary]] — the canonical **Triager** definition (a member of the Fabro team who retrieves bundles for bug investigation; **never interacts with the intake service itself**; retrieves bundles directly from S3 using their own AWS credentials; the service has no Triager-facing surface) is the authoritative definition this card expands. The no-service-surface property is vocabulary-level, not implementation-level.

F2's anchors say "inherits the exact wording here" (A1) and "expands the canonical definition into role context" (A2). Both cards say the same thing from their own side ("authoritative definition this card expands"). Bidirectional framing is consistent — F2 is authoritative, A1/A2 expand.

F2's anchor for A2 also names "never interacts with the intake service itself" as the load-bearing property A2 inherits — A2 confirms at L5 and L10 and throughout. Mirror-match.

### 8.6 F3 and F5 backlinks for A1/A2

F3 "Referenced by" entries (F3 L21, L22):
> [[Agent - Reporter]] — post-upload Reporter-facing enrichment (bundle-contents view, symbolicated stack, markdown-rendered logs, feedback prompts) is the paradigmatic drift shape; the Reporter's narrow post-upload surface as defined there is the role-layer expression of this Principle's rejection list.
> [[Agent - Triager]] — Triager-facing UI (team dashboards, listing endpoints, per-team scoping, in-service triage surfaces) is the drift shape this Principle enumerates most concretely; the Triager's no-service-surface shape as defined there is the role-layer commitment those rejections rely on.

A1 WHERE L21 and A2 WHERE L19 both cite F3 with matching framing (`bun.report`-shaped drift at the role layer). A1's claim — "`bun.report`'s Reporter-equivalent is a richer role with a post-upload web experience (markdown-rendered crash data, feedback prompts). `fabro.report`'s Reporter is deliberately thinner; any proposal to enrich the Reporter's post-upload surface is the paradigmatic drift this Principle enumerates." — matches F3's "paradigmatic drift shape" phrasing exactly. Mirror-match.

F5 "Referenced by" entries (F5 L32, L33):
> [[Agent - Reporter]] — the role this Strategy serves (self-hosted Fabro user handing off a Debug Bundle); the Reporter's narrow post-upload surface is the Strategy's concrete expression on the Reporter side of the pickup.
> [[Agent - Triager]] — the role this Strategy exists for (Fabro team member retrieving bundles); the Triager's no-service-surface shape — retrieving directly from S3 — is the Strategy's concrete expression on the team side of the pickup.

A1 WHERE L23 and A2 WHERE L21 both cite F5 with matching framing. Mirror-match.

### 8.7 Drift-risk scan on backlink edits

- **New claims not independently present in A1/A2:** None found. Every backlink target has independent content in the Agent's own WHAT/WHERE/WHY/HOW supporting the backlink anchor.
- **Vocabulary drift introduced via backlinks:** None. All anchors use canonical nouns.
- **Softening of Foundation claims via backlinks:** None observed. If anything, A2 *strengthens* F6 by explicitly calling itself "the single most load-bearing case of" F6's commitment — and F6's backlink anchor carries that strengthening faithfully rather than softening it.

**Verdict on backlinks:** All eight backlink-target cards (F1/F2/F3/F4/F5/F6/C4/A9) carry anchors that mirror the Agent cards' own framing. No smuggled claims, no generic filler, no overclaim. The F6-as-most-load-bearing-for-A2 anchor — the task-flagged highest-risk entry — is carried with exactly the strength A2 claims for itself.

---

## 9. Card-Size Observations (Word-Count Advisories)

Two new word-count warnings (A1 and A2) added to the 14 pre-existing accepted. Consistent with the Foundation-tier pattern:

- A1 and A2 each have WHAT + WHERE + WHY + WHEN + HOW sections with substantive content in each.
- A2's WHERE section carries ten distinct relationships (Identity and vocabulary, Counterpart role, Access boundary, Scope commitment, Scope coherence ×2, Strategic why, Audience commitment, Forward references ×2). Each is load-bearing for a different failure mode. Compressing would lose the specific drift-resistance the section is built to carry.
- A1's "What the Reporter Does" and "What the Reporter Does Not Do" subsections are six bullets each. Each bullet is a specific behavior or a specific non-behavior with a citation to the upstream card that governs it. Trimming would strip the citations and break the load-bearing link graph.

Standalone-readability and drift-resistance earn the length, consistent with the prior Foundation and Decisions cards. No trim recommended.

**Verdict:** Word-count advisories acceptable. Pattern-matches prior ratified length/density trade-offs.

---

## 10. Lint State

Human-reported lint state confirmed:
- `ax lint cards`: 0 errors, 16 warnings (14 pre-existing accepted + 2 new A1/A2 word-count advisories)
- `ax lint graph`: 0/0/0 — full bidirectional closure
- `ax lint lines`: 0/0/0
- `ax lint layers`: 0 errors, 2 pre-existing warnings (experience/ empty — expected; product/ has only A1/A2 — expected, will resolve as A3/A4/A5 are drafted)

No lint-driven fixes required.

---

## 11. Fixes

| ID | Card | Fix | Severity |
| --- | --- | --- | --- |
| — | — | **None that gate the build.** | — |

No nits worth chasing in this pass. A1 and A2 are clean on source citations (no new header-vs-content slips; the inclusion of L52 preamble in A1's Reporter range is a deliberate improvement, not a slip), clean on vocabulary, clean on JTBD dimensions, clean on backlink closure. The nine accumulated trivial citation-range nits from prior spot-checks (F2-1, F3-2, F5-N1, F5-N2, F6-N1, F4-N1, F4-N2, C4-N1, A9-N1) remain unresolved from prior passes but are still not gates.

---

## 12. Verdict

**APPROVED** — A1 `Agent - Reporter` and A2 `Agent - Triager` are spot-check-clear. Proceed to the next batch.

### Summary of readiness

| # | Card | Spot-check status | Load-bearing claim |
| --- | --- | --- | --- |
| A1 | Agent - Reporter | APPROVED (this spot-check) | Self-hosted Fabro user with no identity on the service; CLI-only interaction; three-clause outcome chain (team-has-bundle / URL-shared / back-to-work); emotional state (frustrated-by-prior-bug) is the justification F4's "efficient and reassured" register derives from. |
| A2 | Agent - Triager | APPROVED (this spot-check) | Fabro team member; **never interacts with the intake service itself** (load-bearing, eight reinforcements across WHAT/WHERE/WHY/HOW); retrieves Debug Bundles from S3 with own AWS credentials; single most load-bearing case of F6's narrow-forever commitment. |

### Cross-cutting findings

- **Asymmetric-identity model:** Honestly carried on both cards, with bidirectional cross-references that frame the asymmetry itself as the load-bearing structural property — not just "Reporter has no account + Triager uses AWS" as two independent facts.
- **A2's never-interacts clause:** Placed correctly (WHAT L5, named as "single most counterintuitive property" and tripwired as "most likely to be eroded"), reinforced eight times across the card with no repetition, and carve-out for URL-possession-delete is framed as *incidental* (not Triager-privileged).
- **JTBD outcomes:** Both cards avoid the generic degeneration ("use the product successfully," "manage support tickets"). A1's three-clause chain and A2's three-clause chain are specific to the product affordance.
- **A1 emotional state ↔ F4 tie:** Load-bearing, explicit, directional. A1 holds the emotional state; F4 derives register from it. Both cards attribute the register to the state; F4 does not claim to own the state and A1 does not claim to own the register.
- **Vocabulary:** Both cards are clean. Anti-Example sections explicitly reject the most likely shortening drifts for each role ("user" for Reporter; "team member of fabro.report" for Triager). F3-1 pattern does not recur.
- **Source citations:** Accurate. A1's inclusion of the L52 asymmetric-identity preamble in the Reporter-definition range is a deliberate improvement (the Reporter definition depends on that preamble for load-bearing context). No new header-vs-content slips introduced.
- **Backlinks:** All eight targets (F1/F2/F3/F4/F5/F6/C4/A9) carry anchors that mirror A1/A2's own framing. No smuggled claims. The F6-as-most-load-bearing-for-A2 anchor carries exactly the strength A2 claims for itself.
- **Folder convention:** `product/agents/` is the correct canonical type-to-folder placement. No folder README needed at this scale.
- **Domain parent:** Acceptable absence — Agents are role-shaped, not surface-shaped, and F1/F2 perform the upstream-anchor role for these cards. Re-evaluate when A3/A4 Components are drafted.

### Next draftable batch

**Sam: draft C1, A3, A4, A6 (all EXISTS-source, parallel-draftable).**

Specifically:
- **C1 `Standard - Competitive Context: What to Borrow, What to Skip`** — `bun.report` as template for intake-URL shape only; paste services and support-bundle uploaders covered as analogues at one-page scale (per manifest Open Flag 3, skip the external survey).
- **A3 `Component - Report`** — the upload event and its public identity. Inherits Report definition from F2, access semantics from A9, retention from A11.
- **A4 `Component - Report Metadata Page`** — the one-page surface at the returned URL. Inherits tone from F4, access semantics from A9. Blocked on C2 for visual direction — but C2 blocker does not prevent A4's structural draft (WHAT/WHERE/WHY/WHEN + HOW without visual spec); Sam should draft with an explicit "visual direction deferred to C2" note and close the gap when C2 ratifies.
- **A6 `Standard - Information Architecture`** — minimal IA (one page, one URL per Report, delete-on-URL). This card captures the IA so it stays minimal; trivial in content but load-bearing in scope commitment.

### Rationale for the batch shape

- All four cards are EXISTS-source (draftable from `product-vision.md` today) per the manifest.
- All four depend on the same upstream stack (F1/F2/F5/F6 + A9/A11) which is now fully ratified.
- A3 and A4 are Component cards; A1 and A2 forward-referenced them from `/product/agents/` — drafting A3/A4 closes those forward references into bidirectional wikilinks.
- C1 is Core-tier and currently the only un-drafted Core card (C2 and C3 are blocked on visual direction). Drafting C1 completes Core-tier coverage outside the visual-direction blocker.
- A6 is minimal content but closes the IA Standard referenced implicitly by A4 and explicitly by the Product Thesis.

**Parallel Sam opportunistic work (not gates):** the accumulated nine trivial citation-range nits from prior spot-checks are still unresolved; they can land in a single consistency pass whenever convenient. No gate.

### Maintenance flow note

Per Conan's auto-trigger rule: because A1/A2 introduced the first `/product/` folder, a Downstream Sync pass is auto-triggered to verify meta-files (retrieval profiles, agent definitions that reference library structure) stay consistent with the new `product/agents/` folder. However, since no meta-files currently enumerate library folders (the retrieval profile generator reads the folder tree dynamically), the Downstream Sync pass is expected to be a no-op — verify and close. This should happen before C1/A3/A4/A6 drafting so the meta-state is known-clean when the next batch lands.

**Status: DONE**

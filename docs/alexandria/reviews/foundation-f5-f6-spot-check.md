# Foundation F5–F6 Spot-Check (+ F1/F2/F3 Backlink Audit)

**Reviewer:** Conan
**Date:** 2026-04-22
**Scope:** F5 `Principle - Strategy Support Effectiveness for Self-Hosted Users`, F6 `Principle - Narrow Forever`, plus the proactive backlink edits Sam made in F1, F2, and F3.
**Mode:** Spot-check, not full grading. Gate before F4 (blocked on tone source) and Core tier.
**Lint state (verified by human):** cards 0 errors / 5 word-count warnings (advisory); graph 0/0/0, bidirectional closures complete.

---

## 1. F5 — Does It Carry the Strategic WHY Without Drift?

**Yes, verbatim, with the three anti-goals and the load-bearing trade explicit.**

### 1.1 Verbatim quote check

F5 WHY paragraph 1 quotes the Strategic WHY directly:

> *Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs. `fabro.report` exists to make it easy for users to get that data to the team.*

Matches Bryan's 2026-04-22 confirmation at `docs/alexandria/sources/product-vision.md` L78 word-for-word, including the placement of backticks around `fabro.report`. F5 explicitly frames the quote as "not a summary; it is the thing itself" and instructs downstream agents not to elaborate it — this is the right framing for a Strategy card; elaboration is how Strategy decays.

### 1.2 Three anti-goals, named explicitly

F5 WHAT section L7–L11 enumerates the three things Strategy is NOT optimizing for, each with its own bullet and a one-sentence rationale that names the adjacent concept being rejected:

| Anti-goal (F5 framing)        | Source wording (L78)                | Status    |
| ----------------------------- | ----------------------------------- | --------- |
| Not a formal support SLA      | "not support-SLA formalization"     | Match     |
| Not compliance posture        | "not compliance"                    | Match     |
| Not marketing telemetry       | "not marketing telemetry"           | Match     |

The ordering differs from the source (source order: SLA, telemetry, compliance; F5 order: SLA, compliance, telemetry). Not a drift — the ordering in L78 is rhetorical, not canonical. Each bullet in F5 names the adjacent purpose concretely enough that an agent proposing a feature-shaped like the anti-goal will recognize the match and stop. "Not a ticketing product" (L9), "not an audit artifact … not a regulated-data handling surface" (L10), "not a growth funnel … not a dataset about who uses Fabro" (L11) are the right negative specificity for a Factory-mode agent.

### 1.3 Load-bearing trade

F5 WHAT L13 states the trade load-bearingly:

> this Strategy optimizes for *the team can retrieve the Debug Bundle* over *the user sees rich diagnostics in-browser*.

That is the exact framing Bryan's Strategic WHY implies and the exact framing the spot-check task required. F5 also operationalizes the trade in HOW "The Strategy Test" (L62–L67) as two sequential questions, which keeps the trade from degrading to advisory prose.

### 1.4 No scope creep or adjacent-purpose smuggling

I looked for:
- Any mention of "also for X" where X is an adjacent purpose — none.
- Any softening of the anti-goals ("not *primarily* a support SLA," "not *yet* a telemetry feed") — none.
- Any claim to hosted-Fabro users as in-scope — F5 WHERE L26 explicitly marks hosted-Fabro as out of scope and frames expansion as a Strategy change, not an implementation change. This is the correct handling of the source's open question at L128.

**Verdict:** F5 carries the Strategic WHY verbatim, names all three anti-goals, and makes the load-bearing trade explicit and testable. No drift.

---

## 2. F6 — Is "Narrow Forever" a Permanent Identity Commitment?

**Yes, with the inversion (Roadmap as commitment, not plan) and the enumerated permanent out-of-scope list.**

### 2.1 Identity framing, not v1 framing

F6 opens (WHAT L5) with:

> The intake-only scope of `fabro.report` is a permanent identity decision, not a v1 simplification.

and reinforces this across four locations: WHAT L7 ("narrow forever … means exactly what it says: assume forever"), WHY L23 ("The commitment is phrased as identity rather than as strategy because identity changes require a different kind of approval than strategy changes"), HOW "Commitment vs Plan Framing" (L50–L55), and the closing paragraph (L75) which instructs agents to not soften "narrow forever" to "intentionally narrow," "narrow for this release," or "narrow for the foreseeable future."

The card correctly identifies the decay path ("narrow for v1 → narrow for now → narrow but we'll revisit → narrowness commitment is gone" at WHAT L7) and positions "narrow forever" as the short-circuit against that decay. This is the identity framing the task required, not a v1 simplification framing.

### 2.2 Enumerated permanently-out-of-scope list

F6 HOW "Permanently Out-of-Scope List (The Roadmap-as-Rejection)" (L33–L47) enumerates nine items. Mapping against the task checklist:

| Task-required item                   | F6 bullet                                  | Present |
| ------------------------------------ | ------------------------------------------ | ------- |
| Symbolication                        | L37 Symbolication                          | Yes     |
| Sentry integration                   | L38 Sentry integration                     | Yes     |
| Bundle content rendering             | L39 Debug Bundle content rendering         | Yes     |
| Markdown rendering                   | L40 Markdown rendering of structured crash | Yes     |
| Feedback collection                  | L41 Feedback collection                    | Yes     |
| Team dashboards                      | L42 Team dashboards                        | Yes     |
| Triage UIs                           | L43 Triage UIs                             | Yes     |
| Per-user accounts                    | L44 Per-user accounts                      | Yes     |
| Cross-upload identity tracking       | L45 Cross-upload identity tracking         | Yes     |

All nine required items present, each with a one-sentence rationale that names the specific drift shape (not a generic "out of scope"). The list is framed as closed-but-growable — "the list can grow as new drift shapes are noticed, but items do not leave the list" (L35) — which is the correct commitment shape (asymmetric: add yes, remove no).

### 2.3 Roadmap-as-commitment inversion

F6 HOW "Commitment vs Plan Framing" L50–L55 states the inversion cleanly:

> A Roadmap is usually read as "features planned for the future." This product's Roadmap — this Principle — is read as "features named for the future in order to commit to not building them." That inversion matters.

and operationalizes it with three concrete consequences: no "eventually we'll need X" answers, no "maybe later" pile, and explicit citeability of the Principle against scope-expansion proposals at any stage.

The three-card enforcement frame (F5 + F6 + F3, L57–L65) is explicit and non-overlapping: F5 rules out direction, F6 rules out timing ("maybe later"), F3 rules out feature shape. This is the correct division of labor.

### 2.4 No accidental v1 softening

I looked for:
- Any "for now," "initially," "at launch," "for the current release" softeners — none.
- Any "we may revisit" language — none. The only revision path named (L29) is "explicit product-owner identity-change conversation," which is the correct escape hatch (narrow, high-bar, named as identity-level).
- The word "forever" appears five times in F6 and survives in every context; this is load-bearing and Sam is holding the line.

**Verdict:** F6 makes "narrow forever" a permanent identity commitment, enumerates the required nine out-of-scope items, and frames the Roadmap as a list of rejections rather than a plan. No softening observed.

---

## 3. Complementarity — Do F1, F3, F5, F6 Do Four Distinct Jobs?

**Yes. Four cards, four distinct loads.**

| Card | Load-bearing job                                            | What it owns that the others don't                                        |
| ---- | ----------------------------------------------------------- | ------------------------------------------------------------------------- |
| F1   | Identity thesis: "pickup point, not triage surface"         | The product-shape framing. The counter-thesis. Validation criteria.       |
| F3   | Concrete rejection list with drift signals and tests        | The enumerated drift examples and early-warning signals in feature terms. |
| F5   | Strategic WHY and the three anti-goals                      | The purpose statement. The anti-goals. The two-question Strategy Test.    |
| F6   | Permanent scope commitment (identity-level, not strategy)   | The "forever" framing. The nine-item permanent out-of-scope list.         |

### 3.1 Overlap check — F3 drift list vs F6 out-of-scope list

There is significant surface overlap: both lists name symbolication, Sentry, bundle rendering, markdown rendering, feedback collection, team dashboards, per-user/per-team accounts, and cross-upload identity. I inspected this carefully because it looks duplicative at first glance.

The distinction is load:
- **F3 frames each item as a drift pattern** ("If asked to symbolicate: do NOT add a symbolicator…"). Voice is imperative-negative. The bullets are early-stage tripwires for an agent midway through a feature proposal.
- **F6 frames each item as a permanent commitment** ("Symbolication. No server-side crash parser…"). Voice is declarative. The bullets are the settled-decision reference F3 appeals to.

F6 itself makes this explicit at L13: "Without F6, F3 would read as 'here are features we're choosing not to build right now,' which is an argument that loses. With F6, F3 reads as 'here are features we have permanently committed not to build,' which is not an argument at all — it is a reference to a settled decision."

This is the right division of labor. F3 is the drift-detection card (runtime tripwire); F6 is the commitment card (upstream decision). The surface overlap is structurally necessary — if F3 listed different examples than F6, agents would have two conflicting lists. The voice difference carries the functional difference.

### 3.2 Overlap check — F1 counter-thesis vs F5 anti-goals

F1's counter-thesis (L9) names the "`bun.report`-shaped objection" — the Reporter-richer-experience shape. F5's anti-goals (L9–L11) name three categorically different adjacent purposes (SLA, compliance, telemetry). Zero overlap in content. F1 rejects one drift direction; F5 rejects three different ones. They complement rather than duplicate.

### 3.3 Overlap check — F5 Strategy vs F6 scope commitment

F5 says *what the product is optimizing for*. F6 says *the scope that serves it is permanent*. F6 itself articulates the relationship at L14–L15: "Narrow Forever locks in that scope is the permanent enforcement of that Strategy, not a v1 staging of a fuller product that might later pursue the anti-goals." This is the correct upstream/downstream relationship — F5 is the purpose, F6 is the commitment to not betray that purpose through gradual scope expansion. No duplication.

### 3.4 One of the four could be collapsed into another — check

Could F5 + F6 be one card? **No.** F5's anti-goals are about *purpose* ("we are not also a telemetry product"); F6's out-of-scope list is about *features* ("we do not render bundle contents"). They reject at different layers. A single card would either blur the purpose/feature distinction or become a monster.

Could F3 + F6 be one card? **No.** F3 is voice-negative and runtime; F6 is voice-declarative and upstream. F3 catches drift in motion; F6 is the standing reference. Collapsing would lose the voice difference that carries the function.

Could F1 + F5 be one card? **No.** F1 is product-shape identity ("pickup point"); F5 is strategic purpose ("team retrieval of bundles from self-hosted users"). An agent can recognize a pickup-point violation without having internalized the strategic purpose, and vice versa. Splitting is correct.

**Verdict:** Four cards, four distinct jobs, no collapsible pair. Overlaps are structural (intentional shared list in F3/F6 with voice differentiation) not duplicative.

---

## 4. Vocabulary Compliance

Checked F5 and F6 for canonical five-noun usage per F2 (Report, Debug Bundle, Reporter, Triager, Report Metadata Page).

### 4.1 F5 vocabulary

| Noun                 | Usage in F5                                                  | Status |
| -------------------- | ------------------------------------------------------------ | ------ |
| Report               | L5, L29 context, L29 heuristic                               | Correct |
| Debug Bundle         | L5, L13, L24, L47, L50, L62, L64                             | Correct (always "Debug Bundle," never "bundle" in noun position without qualifier) |
| Reporter             | L5 context, L11, L13, L48–L50, L54, L58, L65                 | Correct |
| Triager              | L24                                                          | Correct |
| Report Metadata Page | L13, L48, L54                                                | Correct |

Shortenings or invented nouns searched and not found: no "receipt page," no "metadata page" (standalone), no "landing page," no "upload," no "submission," no "payload," no "crash report," no "stack trace," no "symbolicator."

F5 L13 uses "in-browser" as a descriptor ("user sees rich diagnostics in-browser") which is not a noun and not a vocabulary concern. F5 L11 uses "dataset" — descriptor of what the product is *not* building, not a product entity. F5 L49 uses "tarball" once in parenthetical ("Fabro version, optional reporter message, arbitrary key/value tags") — wait, that's metadata not tarball. Rechecking — no "tarball" in F5 body; good.

F5 L47 says "team can retrieve the bundle from S3" — "bundle" here is used with "the" as an anaphoric reference to the Debug Bundle already named in the same sentence. This is acceptable anaphoric shortening, not a canonical-name breach. F2's rule is against naming the thing with a non-canonical name in a first-reference or standalone position; it does not forbid "the bundle" when the canonical name is established in-context. No fix.

### 4.2 F6 vocabulary

| Noun                 | Usage in F6                                                 | Status |
| -------------------- | ----------------------------------------------------------- | ------ |
| Report               | L17, L42                                                    | Correct |
| Debug Bundle         | L5, L15, L17, L37–L39, L42, L61                             | Correct |
| Reporter             | L5 (in list), L41, L44, L45                                 | Correct |
| Triager              | L37, L42, L43                                               | Correct |
| Report Metadata Page | L39, L41                                                    | Correct |

Shortenings or invented nouns: no "receipt," no "landing page," no "metadata page" standalone, no "bundle contents view" as a coined noun (it appears as "Debug Bundle content rendering" in the bullet header L39, which is fine — it's describing the rejected feature, not coining a canonical name for a real entity).

F6 L37 says "tarball" — "retrieving Debug Bundles from S3 directly. Symbolication belongs to Triagers with their own tooling…" — wait, let me recheck. L37 actual text: "Symbolication. No server-side crash parser, stack-frame resolver, or symbolication service. Symbolication belongs to Triagers with their own tooling, retrieving Debug Bundles from S3 directly." No "tarball." Good.

F6 L38: "Debug Bundles are opaque tarballs to this service." Here "tarball" is used descriptively ("bundles are tarballs") — it names the file format, not coining "Tarball" as a canonical product entity. This matches F2's treatment of Debug Bundle as a tarball in its own definition (F2 L39 "The compressed tarball produced by `fabro debug`"). Acceptable.

F6 L5 uses the phrase "`bun.report`-shaped product" in prose — parallel to F1's and F3's use. Acceptable; it is naming the drift template, not importing its vocabulary.

**Verdict:** F5 and F6 are vocabulary-clean. No shortenings of canonical names, no borrowed `bun.report` nouns, no invented product entities.

---

## 5. Source Citations

Spot-checked each card's Sources section against the actual source file line numbers.

| Card | Citation                                            | Actual content                                                                  | Status                |
| ---- | --------------------------------------------------- | ------------------------------------------------------------------------------- | --------------------- |
| F5   | L78 Strategic WHY                                   | L78 is the exact Strategic WHY bullet                                           | Accurate              |
| F5   | L51–64 Users section                                | Header L50 "## Users"; content L52–L64                                          | Acceptable nit        |
| F5   | L12 Vision Summary                                  | Header L12 "## Vision Summary"; actual sentence is L14                          | Minor slip            |
| F5   | L115–128 Raven's Inferences and Open Questions      | L115 is Raven's Inferences header; L128 is Open Question 3 (hosted-Fabro)       | Accurate range        |
| F6   | L79 "Narrow forever" Confirmed Product Decision     | L79 is the exact "Narrow forever" bullet                                        | Accurate              |
| F6   | L33–48 "What `fabro.report` Is Not"                 | Header L33 "## What `fabro.report` Is Not (load-bearing)"; content L35–L48      | Minor slip            |
| F6   | L102–107 "Template Relationship: `bun.report`"      | Header L102 "## Template Relationship: `bun.report`"; content L104–L107         | Accurate              |

Both cards also cite `docs/alexandria/inventory/card-manifest.md` for their manifest entries. F6 specifically cites the Conan-promoted-to-Foundation note; the manifest does carry that note at its F6 row (verified). F6 credits the promotion as product-owner approved on 2026-04-22; this matches the task context.

The range-start slips here (F5 L12, F5 L51, F6 L33) are the same pattern as the F2-1/F3-2 slips from the earlier spot-check — header-vs-first-content-line ambiguity. Not blockers; flag for consistency alongside the earlier fixes if Sam is already editing citation ranges. No new fix ID because the pattern is already captured.

**Verdict:** Citations accurate. Minor range-start slips consistent with the earlier F1–F3 pattern; not gate-blocking.

---

## 6. Backlink Audit — F1, F2, F3 Proactive Edits

Sam added three backlink edits after F5 and F6 were drafted. Each claim checked against the source card's content.

### 6.1 F1 WHERE — new "Strategy and scope commitment" subsection

F1 now reads at L17–L19:

> - Strategy and scope commitment:
>   - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the strategic WHY this thesis serves: the product exists to make it easy for self-hosted Fabro users to get Debug Bundles to the Fabro team.
>   - [[Principle - Narrow Forever]] — the paired commitment that turns this thesis's scope into a permanent posture rather than a v1 simplification; names the permanently-out-of-scope list.

Check: does this subsection stay true to F1's original framing and to F5/F6's actual content?

- The F5 description ("the strategic WHY this thesis serves") is an accurate one-line summary of F5's own WHAT (L5) — the thesis is the product-shape expression of the strategy, and F1's identity thesis does serve F5's strategic purpose.
- The F6 description ("the paired commitment that turns this thesis's scope into a permanent posture rather than a v1 simplification; names the permanently-out-of-scope list") is an accurate one-line summary of F6's own WHAT (L5) and HOW (L33–L47).
- F1's prior WHERE structure grouped links into "Standards," "Anti-Patterns," "Forward references," "Template reference." The new "Strategy and scope commitment" subsection is a coherent category addition and does not displace or contradict the prior groups.

No smuggling. The description of F6 as "paired commitment" is consistent with F6's own self-description as "paired with" F1 at F6 L12.

### 6.2 F2 WHY — upgraded F5 forward ref to wikilink, added F6 anchor

F2 WHY now reads at L23–L25 (previously forward-referenced as plain text):

> - Strategy anchor: [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — a narrow service has no room for synonym drift; every noun earns its place by naming a thing a Reporter, Triager, or Fabro engineer actually interacts with in the service of team retrieval of Debug Bundles.
> - Scope anchor: [[Principle - Narrow Forever]] — the closed five-noun set is itself an expression of narrow-forever; new nouns imply scope expansion and require Strategy-change approval, not a vocabulary update.

Check on the F5 anchor: accurate — F5 does establish team retrieval of Debug Bundles as the Strategic purpose (F5 L5, L13), and "synonym drift" in a narrow service does threaten that purpose (unclear nouns slow retrieval). Reasonable anchor.

Check on the F6 anchor (the task flagged this specifically): **does F6 actually support "closed five-noun set is an expression of narrow-forever; new nouns imply scope expansion"?**

F6 L16–L17 states:

> [[Standard - Noun Vocabulary]] — the canonical five nouns (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) are the entire entity surface this product will ever have. New nouns added to the product imply scope expansion; under this Principle, that is a Strategy change, not a vocabulary update. The Noun Vocabulary is effectively closed, not growing.

So yes — F6 itself says the noun set is closed and that adding nouns is scope expansion. F2's anchor is not smuggling a new claim under backlink guise; it is reflecting a claim F6 independently makes. Both cards converge on the same framing (the noun set as a closed expression of Narrow Forever), which is the correct outcome when two sibling cards reference each other — they should be internally consistent on the bridge claim.

This is a correct bidirectional closure, not a drift.

### 6.3 F3 WHERE — upgraded F6 forward text to wikilink, added F5 "Serves" entry

F3 WHERE now reads at L10–L13 (F6 upgrade):

> - Paired with:
>   - [[Product Thesis - Pickup Point, Not Triage Surface]] …
>   - [[Principle - Narrow Forever]] — the roadmap commitment this Principle defends. Each drift example below is, by that commitment, permanently out of scope — not deferred, not v2. Narrow Forever is the upstream commitment; this card is the concrete drift list it rules out.

And at L12–L13 (F5 addition under new "Serves" subsection):

> - Serves:
>   - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — each drift example below trades team-retrieval investment against richer-Reporter-experience investment, which the Strategy explicitly rejects. Rejecting these drifts is how this card enforces the Strategy.

Check on F6 upgrade: matches F6's own L13 framing ("Without F6, F3 would read as 'here are features we're choosing not to build right now,' which is an argument that loses. With F6, F3 reads as 'here are features we have permanently committed not to build,' which is not an argument at all"). Mutually consistent.

Check on F5 "Serves" entry: matches F5's own WHAT L13 trade ("team can retrieve the Debug Bundle over user sees rich diagnostics in-browser"). F3's bullet accurately frames the drift list as each item trading the wrong direction of F5's load-bearing trade. Not smuggling.

F3's prior WHERE had "Paired with," "Referenced by," "Template being misread," "Contrast with" subsections. Adding a "Serves" subsection is a category addition, not a restructure. No existing F3 framing is displaced.

### 6.4 Drift-risk scan on the backlink edits

Specifically checked for:
- **New claims not present in original F1/F2/F3 content:** none found. Every new backlink description either summarizes F5/F6's own content or restates an F1/F2/F3 claim that was already present as forward-reference text.
- **Vocabulary drift introduced via backlinks:** none. All three backlink additions use the canonical five nouns correctly.
- **Contradiction with original F1/F2/F3 framings:** none. F1 remains the identity thesis, F2 remains the noun lookup, F3 remains the concrete drift list. The backlinks contextualize, they do not overwrite.
- **Softening of load-bearing phrasing:** none. F1's "pickup point, not triage surface" is intact. F3's imperative-negative voice in Drift Examples is intact. F2's canonical-noun rules are intact.

**Verdict:** Backlink edits are legitimate bidirectional-closure work. No smuggled claims; all new anchor descriptions are supported by the target card's independent content.

---

## 7. Fixes

| ID | Card | Fix | Severity |
| --- | --- | --- | --- |
| — | — | None that gate the build. | — |

Nits worth landing opportunistically, not gates:

| ID | Card | Nit | Severity |
| --- | --- | --- | --- |
| F5-N1 | F5 | Sources: `L12` cites the Vision Summary header; actual sentence is L14. Consider `L14` or `L12–L14`. | Trivial |
| F5-N2 | F5 | Sources: `L51–64` cites Users section; header is L50, content L52–L64. Consider `L50–L64` or `L52–L64`. | Trivial |
| F6-N1 | F6 | Sources: `L33–48` cites "What fabro.report Is Not"; header L33, content L35–L48. Consider `L35–L48`. | Trivial |

Same range-start-slip pattern as F2-1/F3-2 in the prior spot-check. Low-priority; flag if Sam is doing a citation-consistency pass.

---

## 8. Lint State

Human-reported state confirmed by spot-check:
- `ax lint cards`: 0 errors, 5 word-count warnings. Sam's "standalone quality requires length" position continues to hold — both F5 and F6 pull their weight per-word, and the three-card enforcement frame (F5 + F6 + F3) would lose teeth if any of them were compressed below current length.
- `ax lint graph`: 0 errors, 0 warnings, 0 info — bidirectional closures complete. This is the gate that matters for this spot-check, and it passed. The backlink edits in Section 6 are why.

No lint-driven fixes required.

---

## Verdict

**APPROVED** — proceed to F4 once the tone source lands (F4 remains blocked on source material, not on F5/F6), or proceed to Core tier (C1/C2/C3/C4) where source is available.

The Foundation ratification is complete for the five built cards:
- F1 (Product Thesis): identity thesis, load-bearing.
- F2 (Noun Vocabulary): canonical nouns, vocabulary anchor.
- F3 (Reject bun.report drift): drift list and tripwires.
- F5 (Strategy): Strategic WHY and three anti-goals, verbatim and complete.
- F6 (Narrow Forever): permanent scope commitment with nine-item out-of-scope list.

The four-card enforcement frame (F1 + F3 + F5 + F6) is coherent, each card does distinct work, and the F2 vocabulary holds across all four. Backlink edits are legitimate; no smuggled claims.

F6's Foundation promotion (manifest decision 2026-04-22, product-owner approved) is validated by the cascade analysis — F6's framing is upstream-referenced by F1 (WHERE), F2 (WHY), and F3 (WHERE), and the "identity-level commitment" framing is load-bearing for every downstream Decision and product-layer card. Promotion was correct.

Three trivial citation-range nits (Section 7) can be landed alongside the existing F2-1/F3-2 nits or deferred to Sam's next citation-consistency pass. They do not affect substantive accuracy.

**Status: DONE**

# Foundation F4 Spot-Check (Final Foundation Gate)

**Reviewer:** Conan
**Date:** 2026-04-22
**Scope:** F4 `Principle - Minimal Public Surface Feels Calm`, plus backlink edits Sam applied to F1/F2/F3/F5/F6 to close the graph bidirectionally.
**Mode:** Spot-check (not full grading). This is the last Foundation gate — after this, Foundation is either fully ratified or returns for fixes.
**Lint state (verified):**
- `ax lint cards`: 0 errors, 6 word-count advisories (F4 is 2611 words — largest so far)
- `ax lint graph`: 0 errors, 0 warnings, 0 info — full bidirectional closure
- `ax lint lines`: 0/0/0
- `ax lint layers`: 0 errors, 2 pre-existing warnings (no product/experience cards drafted yet)

---

## 1. Does F4 Carry "Efficient and Reassured" as the Load-Bearing Pairing?

**Yes, and the pairing survives unsoftened across every location where it matters.**

The phrase "efficient and reassured" appears in WHAT L5 with both words bolded and named as "the load-bearing tone anchor and they are paired on purpose." WHAT L7–L8 then carries separate definitions for each word, and WHAT L10 restates the pairing dynamic explicitly:

> Efficient alone slides into terse and cold. Reassured alone slides into chatty and worried. Together, they describe output that confirms success cleanly and steps out of the Reporter's way.

This is the exact decay-path framing the source supplied at L20 ("efficient alone slides into terse/cold; reassured alone slides into chatty/worried"). F4 reproduces the dynamic, not just the phrase.

Further checks for the failure modes the task flagged:

| Decay risk | Occurs in F4? | Evidence |
| --- | --- | --- |
| Slides into "calm" alone | No | "Calm" appears 6 times and always as a structural/consequent claim ("a minimal surface is the *only* kind of product that can afford to feel this calm"), never as a replacement for the pairing. |
| Slides into "friendly" alone | No | Word "friendly" does not appear in F4. |
| Slides into "efficient" alone | No | WHAT L10 explicitly names this as a failure. WHY L34 uses "efficient and reassured" together again in italics. HOW Decision Heuristic L95–97 runs both tests separately and a third calibration test — a string that passes efficiency alone still fails the heuristic if it is not also reassured. |

The WHEN section L46 even names the drift-vector explicitly: "If this Principle is weakened — softened to 'friendly,' 'approachable,' or 'professional' — each of those surfaces loses its binding register." This is Sam installing a tripwire against the exact drift the task flagged.

**Verdict:** Pairing holds. Load-bearing. Not softened.

---

## 2. Is Warmth Positioned as a Modifier of Efficiency, Not a Competitor?

**Yes. The subordination is stated explicitly in three places and operationalized in the Decision Heuristic.**

WHAT L12 states it word-for-word:

> "Warm" modifies "efficient"; it is *how* the product is efficient (in a human register), not whether it is efficient.

This matches source L28 verbatim in framing. F4 then reinforces the subordination:

- **WHY L36:** The `bun` pairing is deliberate *because* warmth-as-register is checkable against a reference CLI; "warm" does not become a coequal adjective to "efficient."
- **HOW Decision Heuristic (L93–L97):** The three questions run in order — *efficient* first, *reassured* second, `bun`-register match third. Warmth is implicit in the third question as the calibration test, not as a first-class requirement competing with efficiency. A string that is warm but inefficient fails question 1 and is rewritten; warmth does not rescue it.
- **Drift examples (HOW L72, L77):** "Ceremony creep" and "chatty follow-ups" are both framed as warmth overtaking efficiency and rejected on that basis. Terse minimalism (L78) is the opposite drift and is also rejected — but notice the framing: "efficient without reassured." The card treats *reassurance* (not warmth) as the dimension that complements efficiency. Warmth is the register in which both efficiency and reassurance are expressed.

Zero occurrences of "warm and efficient" or "warm AND efficient" as coequal goals. Zero occurrences of "warmth is equally important as efficiency." The subordination is preserved.

**Verdict:** Warmth stays subordinated. No drift into coequal framing.

---

## 3. Is the `bun` CLI Reference Concrete and Actionable?

**Yes, and the concreteness is defended at three escalating levels.**

### 3.1 The card keeps the reference runnable

WHERE L25 gives an exact path and two runnable commands:

> `bun` at `~/p/oven-sh/bun.report`'s dependency — any installed version of `bun` on the Fabro team's systems. Run `bun install` or `bun run dev` and read the output; that register is the target.

This is the source's L32/L34 direction rendered as an instruction an agent can actually execute, not as an adjective cluster.

HOW "Calibration: Use `bun` as the Reference" (L81–L89) then names three specific invocations with the observable outcome for each:

| Command (F4 L85–L87) | What to observe |
| --- | --- |
| `bun install` on a project with a lockfile | "compact success output, a checkmark or two, real English" |
| `bun install` on a project with a resolution error | "direct error message, names the problem, suggests an action, no blame prose" |
| `bun run dev` | "progress indicators, state changes rendered without ceremony" |

Each invocation targets a different part of the tone surface (success, error, progress). That coverage is the reason the reference is useful as a calibration tool — not a general "feel," but a checkable register for each specific string shape.

### 3.2 The card defends concreteness as a property

WHY L36 explicitly contrasts the concrete reference against the adjective-list degeneracy:

> A tone direction that resolves to adjectives ("warm," "direct," "calm") is infinitely re-interpretable by a Factory-mode builder agent; each agent will land the adjectives slightly differently, and the product's tone will drift one commit at a time. A tone direction that resolves to a concrete, installed, runnable CLI — "read the output of `bun install` and match the register" — is checkable.

This is the source's L67 instruction ("keep the reference concrete and checkable") rendered as card content. Sam treated it as load-bearing rather than informational.

### 3.3 No degradation into "bun-like feel" prose

Scanning for degenerate uses of the reference — adjective-cluster substitutions, vague feel-claims, "bun-like" as a standalone descriptor: none found. Every invocation of the `bun` reference in F4 is coupled to either (a) a runnable command, (b) an observable output property, or (c) a mechanical test ("compare the draft against equivalent `bun` output"). The calibration stays calibration.

**Verdict:** Reference is concrete, runnable, and defended as concrete. No degradation.

---

## 4. Does F4 Extend the Tone Surface Beyond the Happy Path?

**Yes, and the extension is structured as its own HOW subsection so downstream agents cannot miss it.**

HOW "The Tone Surface Extends Beyond the Happy Path" (L60–L68) is its own named subsection — not buried, not implicit. Five specific non-happy-path surfaces are enumerated:

| Task-required surface | F4 coverage | Status |
| --- | --- | --- |
| Error messages during upload | L64 — explicit, with network/S3/invalid-bundle subcases and a concrete copy example ("That didn't upload. Check your network and try again.") | Covered |
| Report Metadata Page | L65 — inheritance of CLI register, explicit "not starker, not warmer" guidance, tonal vs visual split stated | Covered |
| Retention expiration messaging | L66 — "if any ever exists," acknowledging the Narrow Forever constraint, with concrete anti-examples ("Your retention window is approaching expiration; please take action." and "Your Report is about to expire — renew it now!") | Covered |
| Delete confirmations | L67 — one-line, explicit | Covered |
| Install/setup copy for the `report` subcommand | L68 — covered as a "just in case" surface | Covered (exceeds source requirement) |

The source only required errors, Report Metadata Page, and retention-expiration-if-any to inherit the register. F4 adds delete confirmations (called out in the task prompt) and install/setup copy (not in the task prompt but a natural extension). No gaps; mild surface-expansion that stays within the source's load-bearing framing ("every user-facing string inherits this register").

WHY L38 also restates the non-happy-path coverage as the explicit reason this Principle is Foundation-tier rather than Amplifier-tier, which prevents a future editor from pruning the section as optional detail.

**Verdict:** Tone surface extends past the happy path, structurally. All task-required surfaces covered with concrete copy examples or anti-examples.

---

## 5. Vocabulary Compliance (F2 Rule)

### 5.1 Canonical noun usage across F4

| Canonical Noun | F4 usage | Status |
| --- | --- | --- |
| Report | L5, L27, L61 bullet header, L65, L66 ("Your Report is about to expire" shown as anti-example) | Correct |
| Debug Bundle | L5, L8, L17, L64 | Correct (always capitalized, never "bundle" in first-reference noun position) |
| Reporter | L5, L7, L8, L12 context, L17, L20, L34 (×3), L56, L64 (×2), L66, L67, L96, L97 | Correct |
| Triager | Not used in F4 | Acceptable — F4 is tone-facing, and Triagers never interact with the intake service; their absence is correct per F5's framing |
| Report Metadata Page | L5, L10, L27, L38, L58, L62 header, L65 bullet, L67, L72 | Correct |

No "receipt page" used as a positive reference anywhere in F4. Every shortening ("receipt," "metadata," "the page") appears only inside explicit anti-examples.

### 5.2 The three intentional anti-example shortenings

The F4 draft summary flagged three intentional anti-examples where shortened forms appear. I found four anti-example shortenings in F4 — each clearly framed as a rejection, not a use:

| Location | Shortened form(s) | Framing | Status |
| --- | --- | --- | --- |
| L58 | "receipt," "bundle," "you," "maintainer," "page" | "Warm register does not license [these]" — explicit negative list | Correctly framed as anti-examples |
| L79 | "Receipt page." "Your bundle." "The metadata." | Labeled "Shortened canonical names" as a drift-example heading; each bullet in the surrounding list is a drift pattern being rejected | Correctly framed as anti-examples |
| L23 | "receipt page" (in context of "does not license shortening 'Report Metadata Page' to 'receipt page'") | Quoted inside a negative clause about what tone-register *does not* license | Correctly framed as anti-example |

All shortenings appear inside explicit rejection framing. None are inadvertent breaches. The F3-1-shaped risk pattern (a card referencing a vocabulary standard while bending it in its own prose) does not occur here — F4 uses shortenings only in quoted-anti-example position and never in its own assertion prose.

The card also actively defends the canonical names in two independent places:
- WHERE L23 ("tone does not override canonical names")
- HOW L58 ("Canonical names, exactly")

Double-coverage is deliberate; vocabulary compliance is explicitly load-bearing for F4 because tone drift and vocabulary drift are each the earliest symptom of the other.

**Verdict:** Vocabulary clean. No inadvertent breaches. Anti-example treatment is rigorous.

---

## 6. Backlink Audit — F1/F2/F3/F5/F6 Edits

Each backlink edit checked against the source card's original framing and against F4's independent content. The bar: does the backlink anchor text reflect claims F4 makes on its own, or does it smuggle new content into the source card?

### 6.1 F1 — "Tonal expression" subsection

F1 WHERE L20–L21 adds:
> - Tonal expression:
>   - [[Principle - Minimal Public Surface Feels Calm]] — the aesthetic consequence of this thesis. A pickup point should *feel* like a pickup point: the Reporter drops a Debug Bundle, gets a URL back, and leaves. "Efficient and reassured" is the emotional shape of that interaction.

**Check:** F4 WHERE L17 independently frames itself as the tonal expression of F1 ("this Principle is the tonal expression of the pickup-point thesis. A pickup point should *feel* like a pickup point"). The F1 backlink mirrors F4's own self-description. The phrase "Efficient and reassured is the emotional shape of that interaction" is an accurate summary of F4's WHAT L5.

No smuggling. The "aesthetic consequence" framing is consistent with F4's positioning as emotional/aesthetic anchor in the manifest.

### 6.2 F2 — "Referenced by" entry (tone does not override canonical names)

F2 WHERE L13–L14:
> - Referenced by:
>   - [[Principle - Minimal Public Surface Feels Calm]] — the tone Principle depends on this Standard. Warm / conversational register does not override canonical names; user-facing strings must still use "Report," "Debug Bundle," "Reporter," "Triager," and "Report Metadata Page" as written here.

**Check (task-flagged as potential overreach):** Does F4 independently support "tone does not override canonical names"?

F4 WHERE L23:
> [[Standard - Noun Vocabulary]] — tone does not override canonical names. CLI copy refers to a **Report**, a **Debug Bundle**, a **Reporter**, a **Triager**, and the **Report Metadata Page**, using those exact terms. "Warm / conversational" register does not license shortening "Report Metadata Page" to "receipt page" in user-facing strings; the canonical-name rule still holds.

F4 HOW L58:
> **Canonical names, exactly.** "Report," "Debug Bundle," "Reporter," "Triager," "Report Metadata Page" — as written in [[Standard - Noun Vocabulary]]. Warm register does not license "receipt," "bundle," "you," "maintainer," or "page."

Both F4 locations independently make the claim F2's backlink cites. Not smuggled — F2's anchor reflects F4's own load-bearing content. Bidirectional closure correct.

### 6.3 F3 — "Referenced by" entry (tone drift is earliest vector)

F3 WHERE L14–L16:
> - Referenced by:
>   - [[Standard - Noun Vocabulary]] — …
>   - [[Principle - Minimal Public Surface Feels Calm]] — tone drift is a vector for `bun.report` drift. `bun.report`'s chattier web-product voice ("We've received your crash report!") is a tonal shape this Principle rejects on the same grounds it rejects feature-shape drift; tone drift is the earliest symptom of the cumulative failure mode this card names.

**Check:** Does F4 independently claim that tone drift is the earliest symptom?

F4 WHY L40:
> Tone is also the leading indicator of scope drift. Vocabulary drift from [[Standard - Noun Vocabulary]] … is the structural symptom of drift; *tone* drift toward `bun.report`'s chattier web-product voice ("Thanks for your submission! We've received your crash report and our team will take a look.") is the earliest symptom. Tone drift arrives before feature drift because copy is cheap to write and features are expensive to build. Catching tone drift early catches the scope drift that would otherwise follow.

F4 WHERE L19:
> [[Principle - Reject Like bun.report Drift]] — tone drift is a vector for `bun.report` drift… Tone drift is not a separate failure mode from feature-shape drift; it is the earliest one.

Both F4 locations say this explicitly. F3's anchor reflects F4's independent claim. Not smuggled.

### 6.4 F5 — "Referenced by" entry (warmth must serve team retrieval)

F5 WHERE L27–L28:
> - Referenced by:
>   - [[Principle - Minimal Public Surface Feels Calm]] — the tonal expression of this Strategy. "Efficient and reassured" is the emotional shape of a Reporter who pastes the URL into the right place and gets back to work — which is how the Fabro team gets the Debug Bundle. Warmth that slows the handoff trades Reporter-facing polish against team-retrieval effort, which this Strategy rejects.

**Check:** Does F4 independently make the "warmth serves retrieval" claim?

F4 WHERE L20:
> [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — warmth must serve team retrieval. A Reporter who feels *efficient and reassured* pastes the URL into the right place and returns to work, which is how the Fabro team gets the Debug Bundle. Warmth that slows the handoff — effusive success messages, post-upload feedback prompts, chatty follow-ups — trades Reporter-facing polish against team-retrieval effort, in the exact direction [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] explicitly rejects.

F5's anchor is a compression of F4's own prose. Bidirectional closure correct; not smuggled. This also validates the task-flagged claim that the F5 edit "replaced F4 forward reference with resolved Referenced by entry" — before F4 existed, F5 could only forward-reference F4; now that F4 is drafted and makes the claim independently, F5's anchor stops being speculative and becomes an accurate summary.

### 6.5 F6 — "Referenced by" entry (minimal public surface = structural condition for affordable calm)

F6 WHERE L18–L19:
> - Referenced by:
>   - [[Principle - Minimal Public Surface Feels Calm]] — "minimal public surface" is the structural condition that makes the product's calm tone affordable. Calm is cheap when the surface is a CLI plus one Report Metadata Page; it gets expensive fast if the surface grows. Every feature this Principle keeps permanently out of scope is a feature whose tone does not have to be designed or kept consistent.

**Check (task-flagged as highest overreach risk):** Does F4 independently make this claim?

F4 WHAT L10:
> This Principle exists because the product has a permanently minimal public surface — a CLI and a one-page Report Metadata Page — and a minimal surface is the *only* kind of product that can afford to feel this calm. If the surface grew, keeping the tone calm would get harder. The narrow shape is what makes the calm affordable.

F4 WHERE L21:
> [[Principle - Narrow Forever]] — "minimal public surface" is the structural condition this Principle depends on. Calm is cheap when the surface is a CLI plus one page. Calm is expensive when the surface is a CLI, a page, a dashboard, a feedback flow, and a team triage UI. Every feature kept off the public surface under [[Principle - Narrow Forever]] is a feature whose tone does not have to be designed, defended, or kept consistent with the rest. The narrow-forever commitment is what makes this Principle sustainable.

F4 states this claim in its WHAT section (the card's load-bearing content) and restates it in WHERE with matching mechanics. F6's anchor is a close paraphrase. Not smuggled — in fact, F4 carries this claim *more strongly* than F6's anchor does (F4 frames it as the reason the Principle exists, not as a peripheral note).

This is actually the most load-bearing of the five backlinks, because it's the only one that establishes *why F4 is possible at all*. The causal direction is F6 → F4 (Narrow Forever's scope commitment makes F4's calm tone structurally affordable). F6's anchor reflects that direction correctly.

### 6.6 Drift-risk scan on the backlink edits

Checked for:
- **New claims not independently present in F4:** none found. Every backlink anchor description reflects a claim F4 makes in its own WHAT/WHERE/WHY content.
- **Vocabulary drift introduced via backlinks:** none. All five anchors use canonical nouns correctly.
- **Contradiction with original card framings:** none. F1's identity thesis, F2's canonical-noun rules, F3's imperative-negative drift list, F5's Strategy verbatim, and F6's identity-level commitment are all intact.
- **Softening of existing load-bearing language:** none. The new "Tonal expression" category in F1, the "Referenced by" additions in F2/F3/F6, and the resolved-forward-ref in F5 are all category-additions or forward-to-resolved upgrades, not restructures.

**Verdict:** All five backlink edits are legitimate bidirectional-closure work. No smuggled claims; every new anchor description is supported by F4's independent content.

---

## 7. Size — Does 2611 Words Earn Its Keep?

**Yes, but with one specific trimmable section and one acceptable-but-at-the-ceiling section.**

F4 is 2611 words — the largest Foundation card. The length has to justify itself against standalone-agent-readability; a Factory-mode agent reading only F4 must be able to calibrate CLI copy, error copy, Report Metadata Page copy, delete confirmation copy, and retention-expiration copy (if any) without access to F3/F5/F6 for context.

### 7.1 Length audit by section

| Section | Approx word share | Weight-pulling? | Notes |
| --- | --- | --- | --- |
| WHAT (L3–L12) | ~11% | Yes | Core framing (efficient/reassured, warm modifies efficient, minimal surface as structural condition). Each sentence carries a distinct load. No cuttable fat. |
| WHERE (L14–L30) | ~17% | Yes | Five distinct WHERE categories (Implements / Paired with / Vocabulary / Reference CLI / Forward references) each with non-trivial anchor prose. The pairing descriptions for F3, F5, F6 have some repetition with F4's own later HOW section — see 7.2. |
| WHY (L32–L40) | ~14% | Yes | Three paragraphs: tone-as-first-emotional-surface, bun-as-calibration-vs-adjective-drift, Foundation-tier-inheritance. Each paragraph is a distinct argument. Paragraph 4 (L40, tone-drift-as-leading-indicator) is load-bearing for the F3 pairing; can't be cut without breaking that bridge. |
| WHEN (L42–L46) | ~4% | Yes | Tight. Names the three confirmed source lines and flags revision-authority. |
| HOW Register (L50–L58) | ~13% | Yes | Seven bullets each naming a specific surface-level prescription with an anti-example. Each bullet is load-bearing; removing any of them lets a downstream agent re-invent that surface. |
| HOW Tone Surface (L60–L68) | ~10% | Yes | The non-happy-path section flagged as structurally required. Five bullets, each a distinct surface. Cannot compress without losing coverage. |
| HOW Drift Examples (L70–L79) | ~13% | Yes | Eight drift patterns. This is the tripwire section for agents — each pattern is a recognizable drift shape with a concrete anti-example. Compressing would let patterns slip past. |
| HOW Calibration (L81–L89) | ~9% | Yes | Three `bun` invocations + interpretation guide. Concrete-reference section; its value is proportional to its specificity. |
| HOW Decision Heuristic (L91–L99) | ~5% | Yes | Three-question decision tree. Terse already. |
| Sources (L101–L105) | ~3% | Yes | Citations only. |

### 7.2 The one specific trimmable section

**WHERE "Paired with" F3 entry (L19) and WHY L40** overlap noticeably on the "tone drift is the earliest symptom" claim. Both paragraphs make the same argument — tone drift arrives before feature drift because copy is cheap and features are expensive. The WHY version (L40) is the stronger of the two because it's in the card's belief section and has the full argument with the `bun.report` anti-example; the WHERE F3-pairing anchor could be tightened to one line that just cites the claim rather than re-stating it.

Specifically, F4 L19 could drop from ~80 words to ~30 by replacing:

> "[[Principle - Reject Like bun.report Drift]] — tone drift is a vector for `bun.report` drift. `bun.report` has a chattier, more web-product voice ('We've received your crash report!') because it is a chattier, more web-product-shaped product. If success copy for `fabro report` starts reading like that, the drift is tonal but it is still a drift, and F3 rejects it on the same grounds it rejects feature-shape drift. Tone drift is not a separate failure mode from feature-shape drift; it is the earliest one."

with something like:

> "[[Principle - Reject Like bun.report Drift]] — tone drift is the earliest vector for `bun.report` drift; F3 rejects tonal shape on the same grounds it rejects feature shape. See WHY L40 for the full argument."

Savings: ~50 words, and the load moves to WHY (which is the section that owns belief claims anyway). This is a low-priority nit — not a gate. Sam can take it or leave it.

### 7.3 Sections I considered cutting but would not

- **HOW "Decision Heuristic" (L91–L99):** Only 5% of the card; three-question ladder is already minimal. Cutting would remove the operationalization.
- **HOW "Calibration: Use `bun` as the Reference" (L81–L89):** 9% of the card. Three specific `bun` invocations with observable outcomes. The card's entire argument in WHY L36 is that the reference must be concrete and checkable; this section is that concreteness. Cutting would contradict the card's own WHY.
- **HOW "Drift Examples" (L70–L79):** 13% of the card. Eight drift patterns. Each one is a tripwire the card needs to remain load-bearing as agents write CLI copy. Cutting would let agents re-invent drift shapes.
- **HOW "The Tone Surface Extends Beyond the Happy Path" (L60–L68):** 10% of the card. Task-flagged as explicitly required. Cannot compress without losing coverage of the surfaces the source named.

**Verdict:** Length earns its keep with one specific ~50-word trim opportunity in WHERE L19. The trim is optional, not gating.

---

## 8. Source Citations

Spot-checked line references against `docs/alexandria/sources/tone-and-emotional-goals.md` and `docs/alexandria/sources/product-vision.md`.

| Card | Citation | Actual content | Status |
| --- | --- | --- | --- |
| F4 | tone source L16 "efficient and reassured" | L15 `**Efficient and reassured.**` (L16 is blank — the cited line is actually L15) | Minor slip — off by 1 |
| F4 | tone source L24 "warm / conversational" | L24 `**Warm / conversational.**` | Accurate |
| F4 | tone source L32 `bun` as reference CLI | L32 `**bun.**` | Accurate |
| F4 | tone source L34 register description | L34 is the full register-description paragraph | Accurate |
| F4 | tone source L42 Report Metadata Page inherits register | L42 is the Receipt-page-inherits bullet | Accurate |
| F4 | tone source L44 silent success is not an option | L44 is the "Silence is not an option" bullet | Accurate |
| F4 | tone source L45 emoji / symbol discipline | L45 is the emoji/symbol bullet | Accurate |
| F4 | tone source L66–69 notes for F4 | L66 load-bearing phrase; L67 `bun` calibration; L68 F3 pairing; L69 F5 pairing | Accurate range |
| F4 | product-vision L120 "near-headless" vs "minimal public surface" | L120 is the "near-headless" inference bullet | Accurate |
| F4 | product-vision L33–48 "What `fabro.report` Is Not" | Header L33, content L35–L48 | Minor slip — range-start covers the header rather than first content line (same pattern as F2-1, F3-2, F5-N1, F5-N2, F6-N1 from prior spot-checks) |

### Citation nits

| ID | Fix | Severity |
| --- | --- | --- |
| F4-N1 | Sources: `tone-and-emotional-goals.md (L16 ...)` should be `L15` — the emphasized phrase sits on L15, not L16 (L16 is blank). | Trivial |
| F4-N2 | Sources: `product-vision.md (L33–48)` should be `L35–L48` (L33 is the section header, L34 blank). Same pattern as prior spot-checks' range-start slips. | Trivial |

Both are off-by-one-or-two header-vs-content-line nits. Neither affects substantive accuracy — a reader following either reference lands within the right section. Flag alongside the existing F2-1, F3-2, F5-N1, F5-N2, F6-N1 citation-consistency pass if Sam is already in that pass; do not block on either.

**Verdict:** Citations directionally accurate. Two minor slips consistent with the pattern flagged three times now.

---

## 9. Lint State

Human-reported lint state confirmed:
- `ax lint cards`: 0 errors, 6 word-count advisories. Advisories are acceptable — each card's length has been individually audited across the three spot-checks (F1–F3, F5–F6, F4) and each pulls its weight. F4 at 2611 words is the largest, and Section 7 above confirms the length is earned with one optional ~50-word trim.
- `ax lint graph`: 0/0/0 — full bidirectional closure. The backlink edits in Section 6 are why.
- `ax lint lines`: 0/0/0 — clean.
- `ax lint layers`: 0 errors, 2 pre-existing warnings (product/experience cards not yet drafted). Not F4's problem.

No lint-driven fixes required.

---

## 10. Fixes

| ID | Card | Fix | Severity |
| --- | --- | --- | --- |
| — | — | **None that gate the build.** | — |

Nits worth landing opportunistically, not gates:

| ID | Card | Nit | Severity |
| --- | --- | --- | --- |
| F4-N1 | F4 | Sources: `tone-and-emotional-goals.md` citation for "efficient and reassured" cites L16; actual emphasized text is L15. Change `L16` → `L15`. | Trivial |
| F4-N2 | F4 | Sources: `product-vision.md (L33–48)` header-vs-content slip; change to `L35–L48`. | Trivial |
| F4-N3 | F4 | Optional: trim WHERE L19 (F3-pairing anchor) from ~80 words to ~30 by referring the tone-drift-is-earliest argument to WHY L40 rather than restating it. Saves ~50 words; not load-bearing. | Optional |

Same range-start-slip pattern observed across the three Foundation spot-checks (F2-1, F3-2, F5-N1, F5-N2, F6-N1, and now F4-N1/F4-N2). Worth Sam doing one citation-consistency pass after Foundation closes, landing all seven nits together. Not a gate for moving to Core.

---

## Verdict

**APPROVED** — Foundation tier fully ratified. Proceed to Core tier (C1/C2/C3/C4) once source gaps are filled.

### Summary of Foundation ratification

All six Foundation cards have been built and spot-checked:

| # | Card | Spot-check status | Load-bearing claim |
| --- | --- | --- | --- |
| F1 | Product Thesis - Pickup Point, Not Triage Surface | APPROVED_WITH_FIXES (F1–F3 spot-check) | Identity thesis: intake-only, forever |
| F2 | Standard - Noun Vocabulary | APPROVED_WITH_FIXES (F1–F3 spot-check) | Five canonical nouns, closed set |
| F3 | Principle - Reject "Like bun.report" Drift | APPROVED_WITH_FIXES (F1–F3 spot-check) | Concrete drift list with tripwires |
| F4 | Principle - Minimal Public Surface Feels Calm | APPROVED (this spot-check) | Tone anchor: efficient and reassured, `bun`-calibrated, inherited across every user-facing surface |
| F5 | Principle - Strategy Support Effectiveness for Self-Hosted Users | APPROVED (F5–F6 spot-check) | Strategic WHY + three anti-goals |
| F6 | Principle - Narrow Forever | APPROVED (F5–F6 spot-check) | Permanent scope commitment, nine-item out-of-scope list |

### Five-card enforcement frame is coherent

The Foundation cards form a coherent five-card enforcement frame (F1 identity + F3 drift list + F4 tone + F5 Strategy + F6 commitment) with F2 (Noun Vocabulary) as the cross-cutting vocabulary anchor. Each card does distinct work, and F4 specifically closes the last scope-enforcement gap: tone drift — the earliest symptom of the cumulative failure mode F3/F6 catch in their feature-shape forms.

F4's load-bearing contribution to the frame is threefold:
1. Makes "efficient and reassured" the testable tone anchor across every user-facing string on the product (CLI, page, errors, retention messaging, delete flow).
2. Ties the affordability of that tone to F6's Narrow Forever commitment — calm is cheap *because* the surface is permanently minimal.
3. Pre-empts tone drift as the earliest `bun.report`-drift vector, tightening F3's coverage from feature-shape-only to feature-shape-plus-register.

### Backlink edits are legitimate

All five backlink edits (F1 "Tonal expression," F2 "Referenced by," F3 "Referenced by," F5 resolved "Referenced by," F6 "Referenced by") reflect claims F4 makes independently in its WHAT/WHERE/WHY content. No smuggled claims; no contradictions with original card framings; no vocabulary drift introduced via backlinks. The `ax lint graph` 0/0/0 result reflects that Sam has closed the graph correctly.

### Next steps

**Foundation is closed.** Sam can proceed to Core tier (C1/C2/C3/C4) once the outstanding source gaps are filled:

| Core card | Source gap | Unblock by |
| --- | --- | --- |
| C1 `Standard - Competitive Context` | PARTIAL — `bun.report` covered; paste services / support-bundle uploaders not surveyed. Manifest Flag 3 recommended skipping the external survey; Bryan to confirm. | Bryan decision. |
| C2 `Standard - Design System` | ABSENT — no receipt-page visual direction. | Bryan provides receipt-page visual direction; source file created. |
| C3 `Standard - Receipt Page Sketch` | ABSENT — no sketch exists. | Paired with C2 unblock. |
| C4 `Decision - Who This Service Is For` | EXISTS — self-hosted primary; hosted-Fabro deferred. | No blocker. |

C4 is unblocked and can be drafted immediately. C1 awaits Bryan's scope decision (Conan's recommendation at manifest Flag 3 is still to skip the external survey and produce a one-page "bun.report template only, no other competitor analysis load-bearing at this scale" card). C2 and C3 are blocked together on Bryan's visual direction input.

The seven trivial citation-range nits accumulated across the three Foundation spot-checks (F2-1, F3-2, F5-N1, F5-N2, F6-N1, F4-N1, F4-N2) can be landed opportunistically by Sam in one citation-consistency pass. They do not gate Core.

**Status: DONE**

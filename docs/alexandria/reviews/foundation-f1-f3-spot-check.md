# Foundation F1–F3 Spot-Check

**Reviewer:** Conan
**Date:** 2026-04-22
**Scope:** F1 Product Thesis, F2 Standard - Noun Vocabulary, F3 Principle - Reject "Like bun.report" Drift
**Mode:** Spot-check (not full grading) to clear the gate before F4/F5/F6 drafting
**Lint state:** cards 0 errors / 3 word-count warnings; graph clean; wikilinks bidirectional

---

## 1. Trio Coherence (F1 thesis ↔ F2 vocabulary ↔ F3 negative frame)

The three cards form a coherent trio. Division of labor is clean:

| Card | Load-bearing job | Overlap with siblings |
| --- | --- | --- |
| F1 Product Thesis | States the positive identity ("pickup point, not triage surface") and the scope heuristic. | Names F3 as paired negative frame (WHERE + HOW). Uses F2 vocabulary exactly. |
| F2 Standard - Noun Vocabulary | Fixes the five canonical nouns and their definitions, with drift examples in Common Mistakes. | References F3 at the drift-symptom level only; does not restate the thesis. |
| F3 Principle | Enumerates the specific drift features rejected, plus early-warning signals and a test. | References F1 for "why narrow" and F2 for "vocabulary drift as a symptom." |

**Vocabulary consistency across the three:** "Report," "Debug Bundle," "Reporter," "Triager," "Report Metadata Page" — all five nouns used identically in all three cards. No synonym slip. F1's "receipt page" usage in WHY is framed as counter-thesis language (quoted) and is not a vocabulary breach. F3 uses "receipt page" once in its opening prose ("a receipt-page UI that only exists to confirm an upload") — this is acceptable as descriptive prose (not claiming it as a canonical name) but is mildly drift-adjacent given F2 forbids the shortening. See Fix F3-1 below.

**Framing alignment:** F1's "pickup point, not triage surface" is the load-bearing phrase and all three cards orient around it. F3's drift examples map 1:1 to F1's "What Violating This Looks Like" list but at higher resolution (F3 adds Sentry, auth, retention-per-Report which F1 does not). No contradiction — F3 is the denser version of F1's negative list, which is the correct relationship.

**One minor redundancy:** F1's "Decision Heuristic" second paragraph ("if the answer starts with 'well, `bun.report` does X,' stop") duplicates F3's "Early-Warning Signals" bullet 3 ("A design discussion framed as 'well, `bun.report` does it this way'…"). This is acceptable — F1 is stating it as a heuristic for Thesis readers, F3 is cataloguing it as a drift signal for Principle readers. Different contexts, same sentence shape. Not a fix.

**Verdict:** Coherent trio. No drift between cards.

---

## 2. Is F3 Still Load-Bearing as a Principle?

**Yes.** The rename from Anti-Pattern to Principle did not soften the card. The negative-framing content is intact and positioned correctly:

- **Drift Examples section:** Still structured as "If asked to X: do NOT …" — imperative negative. Not diluted into "prefer Y" framing. Each bullet names a concrete feature from `bun.report` and rejects it by name. This reads as a Principle-with-teeth, not a watered-down best-practices list.
- **Early-Warning Signals:** Kept as its own subsection. Vocabulary-import signal, Triager-endpoint signal, "well `bun.report` does it" signal, Report Metadata Page feature-creep signal. All five signals are recognizable symptoms, not abstract advice.
- **"What This Principle Does Not Forbid" carve-out:** Present and specific. Four items: better CLI copy, operational/retention work, metadata-tag additions, `fabro debug` bundle-structure work. This prevents the Principle from being weaponized against legitimate in-scope improvements. Essential to the card's load-bearing role.
- **Test section:** Two concrete tests ("Does this feature exist in `bun.report`?" / "Who is the audience?"). Sharp, actionable, hard to misapply.

**Negativity is preserved because the content is negative.** The type label is "Principle" but the framing is "reject X, reject Y, reject Z." A Factory-mode agent reading only this card will come away with a clear list of features not to build. That is exactly the load F3 was intended to bear. The rename is cosmetic (linter alignment) and the functional frame survived intact.

**One observation:** the card's WHAT opens with "Reject the specific drift pattern …" which is a strong negative opening sentence — good. The only risk point would be if a future edit smooths that opening into "Favor narrow scope over …" which would soften the frame. Flag for Sam: if F3 is ever edited, the imperative-negative voice is load-bearing and should be preserved.

**Verdict:** Load-bearing under the Principle label. No softening observed.

---

## 3. Does F1 Preserve the `bun.report` Scope Distinction Without Restating F3?

**Yes, with one minor redundancy worth noting.**

F1's bun.report distinction lives in three places:
- **WHAT (L9):** Counter-thesis paragraph names the `bun.report`-shaped objection and rejects it. One paragraph, no feature enumeration.
- **WHERE (L23):** Template reference line — names the borrowed scope ("intake URL shape only") and the not-borrowed scope ("downstream processing, rendering, symbolication, feedback, triage UI"). One line.
- **HOW Decision Heuristic 2nd paragraph (L61):** "If the answer starts with 'well, `bun.report` does X,' stop." One sentence, plus a link to F3.
- **Validation Criteria "Invalidated by":** Names feature-request shapes (rendering, symbolication, feedback, triage UI) that would invalidate the thesis. Four shapes only — does not enumerate.

F1 stays at the level of *framing and heuristic*; F3 owns the *enumerated drift list*. F1 does not repeat F3's 9 drift examples. F1 does not repeat F3's 5 early-warning signals. F1 does not repeat F3's two tests. The complementarity is working.

**The one redundancy:** F1 "What Violating This Looks Like" (five bullets in HOW) overlaps structurally with F3's Drift Examples (nine bullets in HOW). Both lists name concrete violations. F1's five are a subset of F3's nine plus one framing difference (F1 includes "team dashboard" with team-scope rationale; F3 has that plus per-team-auth which F1 folds into the dashboard bullet).

Is this a problem? **No, on reflection.** F1 needs some concrete violation examples in its HOW to be standalone-useful (a builder agent reading only F1 needs at least a few "do not build this" examples to make the Thesis actionable). F3 is the canonical enumerated list; F1 carries an illustrative subset. Sam's judgment here is correct: duplication is small, both cards stay standalone-useful, and the Thesis would lose teeth if its HOW section pointed only to F3.

**Verdict:** Scope distinction preserved. Complementarity clean. No fix needed.

---

## 4. Does F2's Common-Mistakes Section Pull Weight on bun.report Drift?

**Yes.** F2 Common Mistakes bullet 5:

> **Wrong:** Importing `bun.report`'s vocabulary ("Crash Report," "Stack Trace," "Symbolicator," "Feedback"). These nouns name features that `fabro.report` does not have; using them signals a scope drift. See [[Principle - Reject Like bun.report Drift]].

This is the load-bearing bullet for scoped-drift risk at the vocabulary layer. It:
- Names `bun.report` explicitly (not generically "other products").
- Names four specific drift nouns, not a vague "don't use triage vocabulary."
- Explains *why* the nouns are wrong (they name features the product does not have), which prevents the rule from being cargo-culted.
- Links to F3 so an agent hitting this mistake has a path to the fuller Principle.

**Is one bullet enough?** For the Common Mistakes section's standalone-agent-readability purpose, yes. F2's job is to be the noun-lookup authority; the drift bullet is sufficient as a tripwire, and the deep treatment lives in F3. Adding a second drift-specific bullet would double-up on F3 without adding new information.

**Verdict:** Weight pulled. No fix needed.

---

## 5. Sources Section Accuracy

Spot-checked line references against `docs/alexandria/sources/product-vision.md`:

| Card | Citation | Verified content | Status |
| --- | --- | --- | --- |
| F1 | L33–48 | "What fabro.report Is Not" section — claims match (narrow-scope vs bun.report framing) | Accurate |
| F1 | L74–88 | "Confirmed Product Decisions" — Strategic WHY, "Narrow forever," Scope, Retention, etc. | Accurate |
| F1 | L134–141 | "Notes for the Future Vision Card" — load-bearing distinction statement | Accurate |
| F2 | L91–100 "Confirmed Entities" | Header is actually L92 (L91 is blank); entries L96–100. Off-by-one on range start. | Minor — see Fix F2-1 |
| F2 | L53–64 "Users" | Reporter (L54–59) and Triager (L61–64) sections | Accurate |
| F3 | L33–48 | "What fabro.report Is Not" section | Accurate |
| F3 | L100–107 | "Template Relationship: bun.report" (header L102, content L104–107) | Range start 2 lines early but content correct |
| F3 | L48 cascade-risk | L48 contains the "Factory-mode builder agents … will drift" sentence | Accurate |

Citations are directionally correct; two have minor range-start slippage (F2 L91, F3 L100 vs L102). Not enough to block — any reader following the reference lands in the right section. Flag as a Sam nit for consistency when convenient, but not a gate.

**Verdict:** Accurate with minor range-start slippage on two citations. Acceptable.

---

## 6. Word-Count Warnings

Sam's position ("standalone quality requires length") holds for all three cards. Each card is built to be read in isolation by a Factory-mode agent with no access to the others, and length earns its keep.

**However, one specific trim opportunity:**

- **F3 "WHY: Belief" second paragraph (L22–23):** The sentence "The cost of letting drift happen once is small — a single feature of the wrong shape. The cost of letting it happen repeatedly is that the product turns into a second-rate `bun.report` clone while pretending to be something narrower, and the narrowness commitment stops meaning anything because it has been renegotiated, one feature at a time, by agents acting in good faith." — this is rhetorically strong but the second half ("renegotiated, one feature at a time, by agents acting in good faith") restates the first half. Could be trimmed by ~20–30 words without losing the argument. Not a gate.

**F1 and F2:** No specific fat I'd cut. F1's HOW section is the longest but each subsection pulls weight (Following / Violating / Heuristic serve distinct standalone purposes). F2's Canonical Entities table cannot shrink and the Usage Rules are all load-bearing.

**Verdict:** Word-count warnings accepted. Optional trim on F3 WHY paragraph 2.

---

## Fixes

| ID | Card | Fix | Severity |
| --- | --- | --- | --- |
| F3-1 | F3 | In WHAT opening sentence (L5), replace "a receipt-page UI" with "the Report Metadata Page" to conform with F2's canonical vocabulary rule against the "receipt page" shortening. The card currently violates the vocabulary standard it references. | Low (drift-adjacent) |
| F2-1 | F2 | Sources section: correct `L91–100` to `L92–100` (header is L92; L91 is blank). | Trivial (nit) |
| F3-2 | F3 | Sources section: correct `L100–107` to `L102–107` (header is L102). | Trivial (nit) |
| F3-3 | F3 | Optional: trim WHY paragraph 2 second half to tighten the cost argument. | Optional |

None of the above blocks F4/F5/F6 drafting. F3-1 is the most substantive — a card that references a vocabulary standard and then bends the standard in its own prose is a small but real crack. Worth fixing before product-layer cards start quoting F3.

---

## Verdict

**APPROVED_WITH_FIXES** — proceed to F4/F5/F6 drafting in parallel with Sam applying the four fixes above. The trio is coherent, F3 is load-bearing under the Principle label, F1/F3 complement without duplicating, F2 pulls weight on drift vocabulary, and source citations are directionally accurate. F3-1 (receipt-page vocabulary slip in F3) is the only fix with any teeth and is a one-line edit.

Do not block F4/F5/F6 on these fixes — they are small enough to land alongside or after. But F3-1 should land before any product-layer card (A1–A5) quotes or links F3, to avoid propagating the vocabulary slip.

# Source Material — Tone and Emotional Goals (3.2)

**Status:** raw source material, not a card. Captures the tone direction Bryan confirmed to unblock card F4.
**Area:** 3.2 Emotional / Aesthetic Goals (Foundation tier, Experience & Feel domain)
**Captured:** 2026-04-22
**Source:** direct confirmation from Bryan Helmkamp in response to Conan's unblocking questions.
**Feeds card:** F4 `Principle - Minimal Public Surface Feels Calm`

---

## Confirmed Direction

### How a Reporter should feel when `fabro report` returns the URL

**Efficient and reassured.**

- *Efficient*: no wasted lines, no ceremony, no "we are pleased to inform you" prose. The CLI returns what the Reporter needs to share the URL and nothing that delays them.
- *Reassured*: the Reporter should close the terminal confident that the upload landed, the URL works, and the bundle will be there when the team goes to retrieve it. Silent ambiguity is worse than a small amount of confirmation.

The pairing matters: efficient alone slides into terse/cold; reassured alone slides into chatty/worried. Together they describe output that confirms success cleanly and steps out of the Reporter's way.

### Tone Register

**Warm / conversational.**

Not corporate, not bureaucratic, not aggressively minimal. Output that reads like a competent colleague relaying information, not a system log. Expect real English ("Uploaded.", "Share this with the Fabro team.") over status-code prose ("UPLOAD_COMPLETE"). Expect plain words over jargon where possible.

Warmth has limits: it does not mean effusive, chatty, or emoji-heavy for its own sake. "Warm" modifies "efficient" — it tells you *how* to be efficient (in a human register), not whether to be efficient.

### Reference CLI

**`bun`.**

The `bun` CLI is the reference for the overall tone register — ship-it voice, light visual polish (checkmarks, symbols used sparingly where they clarify), warmth in success messages without slipping into ceremony, direct but not curt in error states. When in doubt about how a given CLI string should feel, read the equivalent `bun` output and match the register.

---

## Implications Raven Observes (flag for review)

These are not confirmed decisions; they are direct implications of the three answers above that Sam should treat as defaults unless Bryan corrects them:

- **Receipt page inherits the same register.** The Report Metadata Page should read "efficient and reassured" too — not a starker aesthetic than the CLI output, not a warmer one. Visual direction for that page (Core tier card C2) will need to keep tone consistent with this register.
- **Error messages are part of the tone surface.** A failed upload must still feel efficient-and-reassured in its failure shape: tell the Reporter what happened, what to do next, no blame, no stack traces in the happy-path surface.
- **Silence is not an option in the `bun`-style reference.** The `bun` CLI does not let a successful command produce zero output; it at least marks completion. `fabro report` should do the same — a success should leave a visible confirmation, not just a URL on its own line with no framing.
- **Emoji / symbol discipline.** Follow `bun`'s restraint: a checkmark or small indicator where it genuinely clarifies state (success / failure / progress), not decorative sparkle. Excess emoji drifts away from "efficient."

These implications are Raven's inference from the three confirmed answers. If any of them overreach, correct them in F4 drafting or a follow-up.

---

## What This Source Does Not Cover

- **Visual direction for the Report Metadata Page** (color, typography, spacing, layout) — captured separately for Core cards C2 and C3.
- **Specific CLI copy strings** (exact wording of success / error / progress messages) — belongs in product-layer specs, not in the Foundation Principle card.
- **Accessibility specifics** (screen reader behavior, color contrast minimums, NO_COLOR support) — Amplifier card A8.
- **CLI UX mechanics** (progress indicator shape, delete confirmation flow, failure retry behavior) — Amplifier card A7.

F4 should state the *principle* (what tone the product has, why, how to tell when it's drifting) and leave specific strings / layouts / mechanics to downstream cards.

---

## Notes for the Future F4 Card

When this source becomes a card (`Principle - Minimal Public Surface Feels Calm`):

- The load-bearing phrase is **"efficient and reassured."** Those two words together are the tone anchor. F4 should center on that pairing.
- The `bun` CLI reference is useful as a *calibration tool* — a card reader can go run `bun install` or `bun run dev` and see the register. Keep the reference concrete and checkable.
- Pair F4 with F3 (Reject bun.report drift): tone drift is another vector for "Like bun.report" drift. If success copy starts reading like `bun.report`'s chattier web-product voice ("We've received your crash report!"), that is a tone drift and F3 territory.
- Pair F4 with F5 (Strategy): warmth should serve team retrieval, not Reporter entertainment. A Reporter who feels "efficient and reassured" is a Reporter who pastes the URL into the right place and gets back to work — which is how the team gets the Debug Bundle. Warmth that slows the handoff violates Strategy.

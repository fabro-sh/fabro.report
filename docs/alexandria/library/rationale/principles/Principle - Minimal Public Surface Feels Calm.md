# Principle - Minimal Public Surface Feels Calm

## WHAT: The Principle

The emotional goal for `fabro.report` is that a Reporter who runs `fabro report` and receives a URL feels **efficient and reassured** — and that the rest of the product's public surface (error messages, the Report Metadata Page, any retention or expiration notification that ever exists) inherits that same register. Those two words are the load-bearing tone anchor and they are paired on purpose.

- **Efficient** means no wasted lines, no ceremony, no "we are pleased to inform you" prose. The CLI returns what the Reporter needs to share the URL and nothing that delays them. Success output is compact. Error output is compact. Progress output is compact.
- **Reassured** means the Reporter closes the terminal confident that the upload landed, the URL works, and the Debug Bundle will be there when the Fabro team goes to retrieve it. Silent ambiguity is worse than a small amount of confirmation; a bare URL on stdout with no framing is *not* what this Principle asks for.

Efficient alone slides into terse and cold. Reassured alone slides into chatty and worried. Together, they describe output that confirms success cleanly and steps out of the Reporter's way. This Principle exists because the product has a permanently minimal public surface — a CLI and a one-page Report Metadata Page — and a minimal surface is the *only* kind of product that can afford to feel this calm. If the surface grew, keeping the tone calm would get harder. The narrow shape is what makes the calm affordable.

The tone register is **warm / conversational**, calibrated against the `bun` CLI. "Warm" modifies "efficient"; it is *how* the product is efficient (in a human register), not whether it is efficient. Real English ("Uploaded.", "Share this with the Fabro team.") beats status-code prose ("UPLOAD_COMPLETE"). Plain words beat jargon. A competent colleague relaying information, not a system log. Warmth that slows the handoff is not warmth this Principle is buying.

## WHERE: Ecosystem

- Implements:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — this Principle is the tonal expression of the pickup-point thesis. A pickup point should *feel* like a pickup point: a Reporter drops a Debug Bundle, gets a URL back, and leaves. The emotional shape of "efficient and reassured" is exactly what a pickup-point interaction should feel like when the Reporter closes the terminal.
- Paired with:
  - [[Principle - Reject Like bun.report Drift]] — tone drift is a vector for `bun.report` drift. `bun.report` has a chattier, more web-product voice ("We've received your crash report!") because it is a chattier, more web-product-shaped product. If success copy for `fabro report` starts reading like that, the drift is tonal but it is still a drift, and F3 rejects it on the same grounds it rejects feature-shape drift. Tone drift is not a separate failure mode from feature-shape drift; it is the earliest one.
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — warmth must serve team retrieval. A Reporter who feels *efficient and reassured* pastes the URL into the right place and returns to work, which is how the Fabro team gets the Debug Bundle. Warmth that slows the handoff — effusive success messages, post-upload feedback prompts, chatty follow-ups — trades Reporter-facing polish against team-retrieval effort, in the exact direction [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] explicitly rejects.
  - [[Principle - Narrow Forever]] — "minimal public surface" is the structural condition this Principle depends on. Calm is cheap when the surface is a CLI plus one page. Calm is expensive when the surface is a CLI, a page, a dashboard, a feedback flow, and a team triage UI. Every feature kept off the public surface under [[Principle - Narrow Forever]] is a feature whose tone does not have to be designed, defended, or kept consistent with the rest. The narrow-forever commitment is what makes this Principle sustainable.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — tone does not override canonical names. CLI copy refers to a **Report**, a **Debug Bundle**, a **Reporter**, a **Triager**, and the **Report Metadata Page**, using those exact terms. "Warm / conversational" register does not license shortening "Report Metadata Page" to "receipt page" in user-facing strings; the canonical-name rule still holds.
- Reference CLI:
  - `bun` at `~/p/oven-sh/bun.report`'s dependency — any installed version of `bun` on the Fabro team's systems. Run `bun install` or `bun run dev` and read the output; that register is the target. Checkmarks and symbols sparingly, where they clarify state. Warmth in success messages without ceremony. Direct but not curt in error states. When a CLI string's feel is unclear, the equivalent `bun` output is the calibration.
- Referenced by:
  - [[Decision - No Content Dedup]] — dedup would add an invisible operational mode (hash computation, collision handling, reference counting) whose existence undermines the "calm is cheap" structural condition this Principle depends on; the Decision refuses that surface area.
  - [[Decision - 30-Day Retention Default]] — any retention-expiration notification that ever exists inherits this Principle's register and must not offer agency the Reporter does not have; the Decision defers any such copy to this Principle.
  - [[Agent - Reporter]] — the Reporter is the audience of every user-facing string this Principle governs; "efficient and reassured" is calibrated specifically to the Reporter's emotional state (already frustrated by the bug that sent them to `fabro report`) as described in that Agent card.
- Forward references (planned in `docs/alexandria/inventory/card-manifest.md`, not yet drafted):
  - `Component - Report Metadata Page` (manifest A4) — must carry the same register as the CLI. Not a starker aesthetic, not a warmer one. Visual direction for the page belongs in a Core-tier card (manifest C2), but tone consistency with this Principle is non-negotiable.
  - `Amplifier - CLI UX Mechanics` (manifest A7) — progress indicator shape, delete confirmation flow, failure retry behavior. Must render the register this Principle specifies.
  - `Amplifier - Accessibility` (manifest A8) — screen reader behavior, `NO_COLOR` support, color contrast. Tone choices that depend on color or symbols must degrade gracefully; the register must survive accessibility accommodations.
  - `Standard - CLI Copy` or equivalent product-layer spec for exact success / error / progress strings. Exact strings are downstream; the register they must match lives here.

## WHY: Belief

Tone is the first thing a Reporter experiences and the last thing they remember. The moment the CLI returns the URL is the entire emotional surface of the product for a self-hosted Fabro user who just hit a bug — and that Reporter is already in a bad mood, because something failed. Output that is terse and cold leaves them uncertain their upload worked, which sends them back to run it again or to the terminal history to check, both of which slow the handoff. Output that is chatty and worried adds lines they have to read before they can get to the URL, which also slows the handoff. Output that is *efficient and reassured* does the job once, visibly, in one breath, and steps out of the way.

The pairing with `bun` is deliberate. A tone direction that resolves to adjectives ("warm," "direct," "calm") is infinitely re-interpretable by a Factory-mode builder agent; each agent will land the adjectives slightly differently, and the product's tone will drift one commit at a time. A tone direction that resolves to a concrete, installed, runnable CLI — "read the output of `bun install` and match the register" — is checkable. An agent proposing CLI copy can compare its draft to equivalent `bun` output and see whether the register matches. A reviewer can do the same. Calibration against a live reference collapses a subjective adjective list into a mechanical test.

The reason this Principle is Foundation-tier rather than Amplifier-tier is that *every* user-facing string on this product inherits from it: CLI success output, CLI error output, CLI progress output, the text rendered on the Report Metadata Page, any retention-expiration messaging that ever exists, copy on delete confirmation, copy on upload failure. If this Principle lives at Amplifier tier, each of those surfaces is free to re-invent tone. If this Principle lives at Foundation tier, each of those surfaces is bound to it and the register stays consistent across the whole public surface.

Tone is also the leading indicator of scope drift. Vocabulary drift from [[Standard - Noun Vocabulary]] ("Crash Report" instead of "Report," "Feedback Form" on a metadata page) is the structural symptom of drift; *tone* drift toward `bun.report`'s chattier web-product voice ("Thanks for your submission! We've received your crash report and our team will take a look.") is the earliest symptom. Tone drift arrives before feature drift because copy is cheap to write and features are expensive to build. Catching tone drift early catches the scope drift that would otherwise follow.

## WHEN: Timeline

Tone direction confirmed 2026-04-22 by the product owner in response to Conan's unblocking questions; captured in `docs/alexandria/sources/tone-and-emotional-goals.md`. Stable. The three confirmed answers — "efficient and reassured" as emotional goal (source L16), "warm / conversational" as register (source L24), `bun` as reference CLI (source L32) — are the load-bearing content of this Principle and any revision to them is a product-owner decision, not an implementation call.

This card is Foundation-tier because every downstream user-facing surface inherits from it. If this Principle is weakened — softened to "friendly," "approachable," or "professional" — each of those surfaces loses its binding register and the tone across the product drifts into whatever voice happened to write each string first. The load-bearing specifics ("efficient and reassured," `bun` as the reference) are the thing that stops that drift, and they must stay exactly as phrased.

## HOW: Application

### The Register, In Concrete Terms

- **Success output is compact and confirms.** A successful `fabro report` prints enough to make clear the upload landed and enough to share — at minimum a confirmation line and the URL. It does not print zero lines (silent success is ambiguous; see source L44). It does not print a paragraph.
- **Error output is compact and actionable.** A failed upload tells the Reporter what happened and what to do next. No blame. No stack traces on the happy-path surface. No "please try again later" filler. If retry is the answer, say retry.
- **Progress output clarifies state.** A slow upload gets a progress indicator — a spinner, a byte counter, a percentage, whatever `bun`-style restraint allows. Not decorative animation. Not ASCII art.
- **Symbols are sparse.** A checkmark where it genuinely marks success. An indicator where it genuinely marks failure. Not sparkle. Not emoji-as-decoration. If a string reads the same with the symbol removed, the symbol is probably noise.
- **Real English over status-code prose.** "Uploaded." over "UPLOAD_COMPLETE". "Share this with the Fabro team." over "HANDOFF_READY". "That didn't upload — the server returned an error." over "ERR_UPLOAD_FAILED_5XX".
- **Plain words over jargon.** A Reporter hitting a bug is not a systems engineer and may not be a CLI expert. Words that Fabro's own documentation uses in Reporter-facing contexts are fine; vocabulary imported from adjacent debugging tooling is suspect.
- **Canonical names, exactly.** "Report," "Debug Bundle," "Reporter," "Triager," "Report Metadata Page" — as written in [[Standard - Noun Vocabulary]]. Warm register does not license "receipt," "bundle," "you," "maintainer," or "page."

### The Tone Surface Extends Beyond the Happy Path

Every user-facing string on this product inherits this register, not just the success output. Specifically:

- **Error messages during upload.** A network failure, an S3 permission error, an invalid Debug Bundle — each needs copy that is efficient and reassured in its *failure shape*. Tell the Reporter what happened, what to do next, no blame. A failed upload is not an occasion for corporate apology prose; it is also not an occasion for a stack trace. "That didn't upload. Check your network and try again." is closer to the target than either extreme.
- **The Report Metadata Page.** The page at a Report's URL inherits the CLI's register. Not starker, not warmer. If the CLI is warm/conversational and the page is a Spartan table of metadata, the two do not match and the Reporter's experience breaks on the handoff from terminal to browser. Visual direction for the page lives in a Core-tier card (forward reference: `Component - Report Metadata Page`, manifest A4); *tonal* direction is here, and they must stay aligned.
- **Retention expiration messaging (if any ever exists).** The 30-day retention deadline is a product decision. Whether the product ever notifies Reporters near expiration is a product decision too, and one [[Principle - Narrow Forever]] leans against — but if any such surface ever exists, its copy inherits this register. No formal-support-SLA voice ("Your retention window is approaching expiration; please take action."), no marketing voice ("Your Report is about to expire — renew it now!"). Plain warmth.
- **Delete confirmations.** The Report Metadata Page has a delete button. Its confirmation flow is user-facing copy. Same register.
- **Any copy on CLI install or setup for the `report` subcommand.** If the Fabro binary prints anything about `fabro report` during install, that copy is in scope for this Principle too.

### What Tone Drift Looks Like

- **Ceremony creep.** "Thanks for your submission! We've received your crash report and our team will take a look." — this is `bun.report`'s chattier web-product voice and it violates this Principle *and* [[Principle - Reject Like bun.report Drift]] simultaneously. It is also out-of-band in another way: `fabro.report` does not own the team's response; "our team will take a look" is a commitment this service cannot make. Strip it.
- **Silent success.** A command that completes and prints only the URL, with no framing. Ambiguous — the Reporter cannot tell from the output whether the URL is the success or the failure. Add a confirmation line.
- **Emoji cascade.** Rockets, sparkles, party poppers on success. Decorative, not informative. A checkmark at most.
- **Corporate error prose.** "An error has occurred while processing your request. Please try again later or contact support." — no load, no specifics, no warmth, no action. All four are wrong.
- **Status-code prose.** "UPLOAD_COMPLETE". "ERR_NETWORK_TIMEOUT". The register is conversational, not log-line.
- **Chatty follow-ups.** "How was your upload experience? Tell us what happened." — post-upload feedback prompts are not just tone drift; they are scope drift forbidden by [[Principle - Reject Like bun.report Drift]]. Do not add them, not even warmly.
- **Terse minimalism.** "OK." "Done." "Fail." — efficient without reassured. A Reporter reading "OK." cannot tell what succeeded or whether the URL is trustworthy.
- **Shortened canonical names.** "Receipt page." "Your bundle." "The metadata." These violate [[Standard - Noun Vocabulary]] even when the rest of the register is correct; warm register is not a license to rename.

### Calibration: Use `bun` as the Reference

When in doubt about how a given CLI string should feel, run `bun` and compare.

- `bun install` on a project with a lockfile — compact success output, a checkmark or two, real English.
- `bun install` on a project with a resolution error — direct error message, names the problem, suggests an action, no blame prose.
- `bun run dev` — progress indicators, state changes rendered without ceremony.

A draft CLI string for `fabro report` that reads in the same register as equivalent `bun` output is probably calibrated. A draft that reads warmer (more adjectives, more gratitude, more apology) is drifting toward `bun.report`'s web-product voice. A draft that reads colder (more status codes, more silence, more minimalism) is drifting toward terse-and-cold. Adjust until it matches `bun`.

### Decision Heuristic

Before shipping any user-facing string on this product — CLI, Report Metadata Page, error copy, progress copy, delete confirmation, anywhere — ask three questions in order:

1. **Is this string *efficient*?** No wasted lines, no ceremony, no corporate prose, no filler. If there is a word that could be removed without losing information or warmth, remove it.
2. **Is this string *reassured*?** A Reporter reading it closes the terminal (or the browser) confident the operation succeeded (or failed, clearly). Silent ambiguity is wrong. Chatty anxiety is also wrong.
3. **Does it read in the same register as equivalent `bun` CLI output?** If yes, the calibration is right. If it reads warmer, compress. If it reads colder, add a beat of human voice.

A string that fails any of the three is drifting. Rewrite it, don't defend it.

## Sources

- `docs/alexandria/sources/tone-and-emotional-goals.md` (L15 "efficient and reassured" emotional goal; L24 "warm / conversational" register; L32 `bun` as reference CLI; L34 register description; L42 Report Metadata Page inherits register; L44 silent success is not an option; L45 emoji / symbol discipline; L66–69 notes for F4)
- `docs/alexandria/sources/product-vision.md` (L120 "near-headless" vs "minimal public surface" framing; L35–L48 "What `fabro.report` Is Not" — minimal public surface as structural condition)
- `docs/alexandria/inventory/card-manifest.md` (F4 entry)

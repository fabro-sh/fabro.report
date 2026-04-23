# Source Material — Accessibility Policy

**Status:** raw source material, not a card. Captures the accessibility commitments Bryan confirmed on 2026-04-23.
**Captured:** 2026-04-23
**Source:** direct confirmation from Bryan Helmkamp in response to unblock-interview Group 4.
**Feeds card:** A8 `Standard - Accessibility Policy` (manifest Amplifier tier).

---

## Confirmed Decisions

### Floor

- **WCAG 2.2 AA.** This is the compliance floor for the Report Metadata Page.
- AA rather than AAA — AAA would be overkill for a minimal one-page metadata receipt.
- 2.2 rather than 2.1 — current standard; reflects best-practice awareness.

### CLI Accessibility

- **`NO_COLOR` environment variable honored.** If `NO_COLOR` is set (per https://no-color.org/), the CLI emits no color escape codes. Colored output becomes plain monochrome output.
- Rationale: `NO_COLOR` is the standard user opt-out for CLI color and is respected by `bun`, `ripgrep`, `cargo`, and most modern CLI tools. Supporting it is table-stakes for the `bun`-calibrated tone register in F4.

### Web Accessibility

- **Keyboard accessibility** — the Report Metadata Page (and especially the delete button + its client-side confirmation) must be fully operable via keyboard. No mouse-only interactions. Focus order must be logical. Focus indicators must be visible.
- **Screen reader HTML best practices** — semantic HTML (`<button>` for buttons, `<time>` for timestamps, proper heading hierarchy, `aria-label` where semantics aren't obvious from the element alone). The page is minimal enough that these best practices should be trivial to follow.

### Design Workflow

- **Use the `/ui` skill for webpage design.** This is a workflow hint rather than a product requirement — Bryan prefers the `/ui` skill as the implementation tool when the page is actually built. Note it so Sam and the eventual builder know where to look.

## Implications Raven Observes (flag for review)

- **A one-page, four-element receipt is naturally easy to make accessible.** The page has a heading, three metadata lines (Report ID, upload timestamp, bundle size), and one button (delete). WCAG AA compliance on this surface is a low bar; implementing it correctly the first time is much easier than retrofitting later.
- **Delete-confirmation accessibility needs explicit attention.** A native browser `confirm()` dialog is accessible by default (keyboard and screen reader). A custom JavaScript modal for confirmation needs focus trap, ESC-to-cancel, focus return on close, `role="dialog"` and `aria-modal="true"`. Simpler is better for A11y — if a native `confirm()` is acceptable, use it.
- **Color contrast on error states in the CLI** is a separate concern from `NO_COLOR`. Even when color IS available, contrast against the terminal background matters; some terminal themes render "red" at low contrast. Follow `bun`'s conventions here.
- **`fabro dump` / `fabro upload` CLI output should work in CI logs.** CI log viewers often strip color codes but preserve text. `NO_COLOR` support already covers this, but keep in mind that log viewers are a real audience.

## Open Questions (flagged for human judgment)

None of these block A8 drafting:

1. **Is there a documented internal accessibility-testing workflow?** (Axe, WAVE, manual screen-reader testing, Lighthouse?) Not specified.
2. **Does the service need to produce any assistive-tech affordances for CLI output** beyond `NO_COLOR`? E.g., explicit `--plain` flag for scripting contexts?
3. **Any formal accessibility statement needed on the Report Metadata Page** (a link to an accessibility policy or contact)? Most small internal tools skip this; flag for confirmation.

## What This Source Does Not Cover

- **Exact color tokens for CLI output** (red-for-error, green-for-success, etc.). Implementation detail — lives in `Standard - CLI Copy` or A7 when drafted.
- **Exact heading hierarchy / semantic structure of the Report Metadata Page.** A4 (Component) specifies structure; the visual layout is C2 (Design System) and C3 (Sketch). A8 provides the accessibility lens on both.
- **Non-Web / non-CLI accessibility surfaces.** There are none — the product has no mobile app, no desktop app, no voice interface.

## Notes for Future A8 Card

When Sam drafts A8:

- **Load-bearing:** accessibility is a floor, not a feature. Failing the floor is a defect, not an incomplete feature.
- **Two-surface coverage:** CLI (`NO_COLOR`) and web (WCAG 2.2 AA + keyboard + screen reader).
- **Cross-refs:** F4 (tone register — the `bun`-calibrated CLI must also be `NO_COLOR`-compliant), A4 (Report Metadata Page is the only web surface needing A11y), A7 (CLI UX has its own A11y shape via `NO_COLOR`), C2 (Design System must respect contrast minimums — blocked until C2 unblocks).
- **Anti-examples to include:** hard-coded ANSI escape codes without `NO_COLOR` guard; mouse-only delete interaction; icon-only buttons without text or `aria-label`; color as the only signal for error state (fails contrast and colorblind users); a custom JavaScript modal without focus trap when a native `confirm()` would do the job.
- **Workflow note:** page implementation uses the `/ui` skill.

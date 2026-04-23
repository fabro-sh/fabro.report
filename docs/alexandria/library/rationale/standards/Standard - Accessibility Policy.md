# Standard - Accessibility Policy

## WHAT: Definition

The **accessibility floor** for `fabro.report` across its two user-facing surfaces: the CLI (Fabro's dump-and-upload flow *and* the `fabro feedback` flow) and the web (the Report Metadata Page). On the CLI, the floor is **`NO_COLOR` support across all four canonical commands** (`fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro feedback`) — when the `NO_COLOR` environment variable is set (per https://no-color.org/), the CLI emits no color escape codes, for every command in the set. On the web, the floor is **WCAG 2.2 AA** for the Report Metadata Page, including keyboard-first operation, semantic HTML, and an accessible delete-confirmation flow. The Feedback flow has no web surface (no page is rendered on `fabro feedback` success — see [[Standard - Information Architecture]]'s Feedback IA framing), so the web accessibility floor applies to the Debug Bundle flow's Report Metadata Page only; the Feedback flow's accessibility surface is the `fabro feedback` CLI's error and confirmation output, which is covered by the CLI floor above. The floor is not a feature; it is a pass/fail criterion for the product, and failing it is a defect rather than an incomplete feature. The `/ui` skill is the recommended implementation tool when the Report Metadata Page is actually built. This Standard is deliberately short because the product's accessibility surfaces are both small: a four-command CLI plus a one-page receipt with four visible elements is naturally easy to make accessible, and implementing it correctly the first time is much cheaper than retrofitting later.

## WHERE: Ecosystem

- Surfaces governed:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — A7 names `NO_COLOR`-awareness as a CLI mechanic; this Standard binds that mechanic to the accessibility floor and owns the non-color halves (keyboard operation, screen-reader semantics, delete-confirmation accessibility). The CLI surface governed by A7 must also satisfy this Standard's CLI floor.
  - [[Component - Report Metadata Page]] — the only web surface the product has. Its four-element structure (ID, timestamp, bundle size, delete button) is specified by A4; this Standard adds the accessibility lens — semantic HTML, keyboard operability, screen-reader behavior — on top of that structure.
- Tone register alignment:
  - [[Principle - Minimal Public Surface Feels Calm]] — the `bun`-calibrated CLI register specified by F4 must *also* be `NO_COLOR`-compliant. Warmth is carried by word choice, not by color; color strips gracefully without changing register. Error legibility in CI log contexts (which often strip color but preserve text) is a direct consequence of this alignment.
- Access-model coupling (delete flow):
  - [[Decision - Capability URLs over Auth]] — the delete-confirmation accessibility requirement is load-bearing specifically on the Report Metadata Page because the delete action is the whole Reporter-initiated capability on the service after upload. If the confirmation is inaccessible, the delete capability is effectively inaccessible to keyboard-only and screen-reader users.
- Audience:
  - [[Agent - Reporter]] — the Reporter is the audience of both Debug Bundle surfaces (CLI + Report Metadata Page). Accessibility on the Debug Bundle flow is accessibility for Reporters; Triagers do not use the service's surfaces (see [[Agent - Triager]]), so no Triager-facing accessibility surface exists.
  - [[Agent - Feedback Submitter]] — added 2026-04-23. The Feedback Submitter is the audience of the `fabro feedback` CLI. The Feedback flow has no web surface (no Report Metadata Page equivalent), so its accessibility floor is the CLI half of this Standard only — `NO_COLOR`-awareness, screen-reader-readable confirmation line, keyboard-invocable command.
- Scope commitments:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the pickup-point thesis keeps the accessibility surface small. One CLI, one page, one delete button; no dashboards, no listings, no multi-step flows. The narrowness is what makes WCAG 2.2 AA cheap to hit and keep.
  - [[Principle - Narrow Forever]] — the surface governed by this Standard is the surface forever. Accessibility work done at the 2.2 AA floor for the Report Metadata Page does not grow into a larger compliance program because the product does not grow additional surfaces.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — accessibility copy (ARIA labels, heading text, delete-confirmation prompts) uses canonical names exactly. "Report Metadata Page" in `<title>` and heading elements, not "receipt page" or "status page"; "Delete Report" on the button, not "Delete" or "Remove."
- Rendering posture:
  - [[Decision - Service Stack]] — A12's server-side-rendered HTML commitment is what makes the web-accessibility floor structurally cheap. There is no SPA build pipeline to audit; the Report Metadata Page is rendered by the Rust service and delivered as HTML, which is the simplest path to WCAG 2.2 AA conformance. Topology changes that introduce a client-rendered front-end would move the accessibility surface.
  - [[System - Service Topology]] — the SSR-HTML pipeline this Standard's floor applies to runs on the Fargate task named in A14. Topology changes that move rendering off Fargate (to edge workers, to a client-side SPA build) would move the accessibility surface and would need to clear this Standard explicitly.
- Blocked cross-reference:
  - `Standard - Design System` (manifest C2, blocked on visual direction) — C2 owns color choices and typography for the Report Metadata Page. Color contrast minimums (WCAG 2.2 AA: 4.5:1 for body text, 3:1 for large text and UI components) are an accessibility floor that C2's color palette must respect. This cross-reference is a plain-text forward reference until C2 is drafted.
- Forward references (planned in `docs/alexandria/inventory/card-manifest.md`, not yet drafted):
  - `Standard - CLI Copy` (not in manifest; referenced as the future home for exact strings) — exact color tokens for CLI output (red-for-error, green-for-success) are implementation detail for that downstream card; this Standard only binds the fallback-to-monochrome behavior when `NO_COLOR` is set.
- Source provenance:
  - `docs/alexandria/sources/accessibility.md` — the raw material Bryan confirmed on 2026-04-23. Authoritative for the WCAG 2.2 AA floor, `NO_COLOR` requirement, keyboard and screen-reader best practices, the delete-confirmation accessibility consideration, and the `/ui` skill workflow note.

## WHY: Rationale

Accessibility for this product is structurally cheap. A three-command CLI that honors `NO_COLOR` and a one-page receipt built with semantic HTML are both at the low-effort end of the accessibility spectrum. The reason this Standard still exists explicitly — rather than being absorbed into "good engineering" — is that *accessibility skipped at the start is expensive to retrofit later*, and retrofitting is the most common failure mode across products this size. A Standard card anchors the expectations before implementation so neither "we'll handle it later" nor "we meet it by default" becomes the drift vector that lets the floor slip.

The *two-surface* framing is load-bearing. Accessibility commitments on web products often focus on web accessibility alone and treat CLI accessibility as an afterthought (or as "not applicable"). For `fabro.report`, the CLI is the Reporter's primary surface: they touch it every time, and the Report Metadata Page only sometimes. `NO_COLOR` support is therefore as load-bearing as WCAG 2.2 AA — both are floor conditions, and either floor failing is a product defect. Naming both surfaces explicitly here prevents the CLI-is-fine mental model that otherwise quietly takes hold.

The **delete-confirmation accessibility** concern is called out separately because the delete action is the one interactive affordance on the Report Metadata Page, and an inaccessible confirmation modal effectively locks keyboard-only and screen-reader users out of the product's only Reporter-initiated capability after upload. Source material (`accessibility.md` L35) notes that a native browser `confirm()` dialog is accessible by default, and a custom modal needs focus trap, ESC-to-cancel, focus return, `role="dialog"`, and `aria-modal="true"`. The card leaves the modality choice open (A7 also leaves it open) but binds either choice to the accessibility contract.

## WHEN: Timeline

Drafted 2026-04-23 from source material captured the same day (`docs/alexandria/sources/accessibility.md`). Stable. Revisions to the floor — raising to AAA, adding a formal accessibility statement, adding an internal audit cadence — are product-owner scope decisions that feed back into the source material. Revisions to the mechanics (exact ARIA patterns, exact keyboard order, modality of the delete confirmation) are implementation-level and appropriate to land as Standard updates informed by implementation experience. The `/ui` skill workflow note is a current-practice hint; if the recommended tool changes, this Standard updates alongside.

## HOW: Specification

### CLI Accessibility

- **`NO_COLOR` support across all four canonical commands.** When the `NO_COLOR` environment variable is set (any non-empty value) per https://no-color.org/, the CLI emits no color escape codes — for every command in the canonical set (`fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro feedback`). Colored output becomes plain monochrome output; every success, error, warning, and progress state is readable as text alone, including the `fabro feedback` confirmation line and error messages specified by [[Standard - Interaction Patterns (CLI + Delete)]]'s Feedback CLI section.
- **Implication for error output:** color is never the *only* signal. Red-for-error and green-for-success are additive to plain-English error and success text. This property is automatic when the CLI-copy register from [[Principle - Minimal Public Surface Feels Calm]] is followed, but it is named here explicitly because colorblind users and `NO_COLOR`-set users need the same resilience.
- **CI log compatibility.** CI log viewers often strip ANSI color but preserve text. `NO_COLOR`-honoring output also works in those contexts for free, without a separate `--plain` or `--no-color` flag. If a `--plain` flag is ever added, it is shorthand for `NO_COLOR=1`, not a different behavior.
- **Color contrast (when color is rendered).** Terminal themes render "red," "yellow," and "green" at different contrasts against different backgrounds. The CLI follows `bun`'s color conventions (per [[Principle - Minimal Public Surface Feels Calm]]), which are calibrated for typical terminal contrast; the Standard does not pin exact RGB values because terminal rendering is user-theme-controlled.

### Web Accessibility (Report Metadata Page)

- **Floor:** WCAG 2.2 AA. AA rather than AAA because the page is a minimal one-page receipt and AAA would be overkill; 2.2 rather than 2.1 because it is the current standard and reflects best-practice awareness.
- **Keyboard-first operation.** Every interactive element (the delete button, the delete-confirmation flow, any link added later) must be fully operable via keyboard. No mouse-only interactions. Focus order is logical (heading → metadata lines → delete button → confirmation flow). Focus indicators are visible.
- **Semantic HTML.** `<button>` for buttons (not styled `<div>`). `<time>` for timestamps. Proper heading hierarchy (one `<h1>` for the page, no skipped levels). `aria-label` where element semantics are not obvious from content alone. These are trivial to follow for a four-element page; they are load-bearing because skipping them costs little up front and much later.
- **Screen reader support.** The four visible elements read in a sensible order under a screen reader: page heading, then Report ID, upload timestamp, bundle size, then the delete button. The delete-confirmation flow, whenever invoked, is announced in a way the screen reader can follow (see Delete-Confirmation Accessibility below).

### Delete-Confirmation Accessibility (Load-Bearing)

The delete action on the Report Metadata Page has a client-side confirmation per [[Standard - Interaction Patterns (CLI + Delete)]]. That confirmation is the one place on the page where accessibility can quietly fail:

- **Preferred path (simplest): native `confirm()` dialog.** Accessible by default — keyboard operable, screen-reader announced, no focus-trap or role-management work required. If the product accepts the default browser modality, this is the correct choice.
- **Alternate path (custom modal): requires focus trap, ESC-to-cancel, focus return on close, `role="dialog"`, and `aria-modal="true"`.** If a custom modal is chosen (e.g., for visual-consistency reasons from a future `Standard - Design System`), all of the above are required; any missing element is an accessibility defect.
- **Authorization is separate from accessibility.** The delete action is authorized by URL possession (per [[Decision - Capability URLs over Auth]]); accessibility covers *how* the Reporter operates the button and the confirmation, not *whether* they are allowed to.

### Workflow Hint

When the Report Metadata Page is actually implemented, use the **`/ui` skill** as the workflow tool. This is a product-owner preference captured in source (`accessibility.md` L30) rather than a product requirement — the skill is the recommended on-ramp that produces accessible defaults with less retrofit risk.

### Anti-Examples (What Fails the Floor)

- **Wrong:** Hard-coded ANSI escape codes in CLI output without a `NO_COLOR` guard. Breaks the `NO_COLOR` floor; breaks CI-log legibility.
- **Wrong:** Mouse-only delete on the Report Metadata Page (e.g., a click handler attached to a `<div>` with no keyboard focus or `Enter` handler). Fails WCAG 2.2 AA keyboard operability.
- **Wrong:** Icon-only buttons without text or `aria-label`. A delete button rendered as a trash-can glyph alone is inaccessible to screen readers.
- **Wrong:** Color as the only signal for error state. Red text on red background with no accompanying English sentence is inaccessible regardless of contrast; it fails color-independence.
- **Wrong:** A custom JavaScript modal for delete confirmation without focus trap, ESC-to-cancel, focus return, `role="dialog"`, and `aria-modal="true"`. If a native `confirm()` would satisfy the product requirement, use it — retrofitting focus management later is expensive.
- **Wrong:** Styled `<div>`s acting as buttons. Semantic HTML is part of the WCAG floor; `<div onclick=...>` fails screen-reader affordance even if it visually looks like a button.
- **Wrong:** Skipping heading levels (jumping from `<h1>` to `<h3>`). Breaks heading-hierarchy navigation for assistive tech.
- **Wrong:** A `/--plain` CLI flag that does something *different* from `NO_COLOR=1`. If `--plain` is added at all, it is a synonym for `NO_COLOR=1`, not a separate accessibility mode.

### Test

Before shipping either the CLI or the Report Metadata Page, run three checks. First: run the CLI with `NO_COLOR=1` set and verify every success, error, warning, and progress surface is legible without color. If any state becomes ambiguous, the register is carrying information in color that should be in text. Second: operate the Report Metadata Page with the keyboard alone — Tab through the focus order, Enter to activate the delete button, confirm the delete confirmation reacts to Enter and ESC, confirm focus returns sensibly after close. If any interaction requires the mouse, the page fails the keyboard-first floor. Third: run an accessibility audit (axe, WAVE, Lighthouse — whichever the team prefers) against the Report Metadata Page and verify WCAG 2.2 AA with no violations. If violations appear, they are defects, not deferred improvements.

## Sources

- `docs/alexandria/sources/accessibility.md` (full file — authoritative for WCAG 2.2 AA floor, `NO_COLOR` requirement, keyboard and screen-reader best practices, delete-confirmation accessibility, and `/ui` skill workflow note; confirmed 2026-04-23)
- `docs/alexandria/sources/cli-interaction-patterns.md` (error-register and delete-confirmation context — how the accessibility floor lands on the mechanics A7 specifies)
- `docs/alexandria/inventory/card-manifest.md` (A8 entry)

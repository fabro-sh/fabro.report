# Standard - Design System (Receipt Page + CLI Output)

## WHAT: Definition

The **visual-system spec** for `fabro.report`'s single rendered surface — the Report Metadata Page — and the cross-reference point for CLI output's visual register. This Standard fixes typography (Geist sans + JetBrains Mono, lifted from the Fabro web UI), the full palette (Fabro's three teal shades, three navy shades, three ice shades, three accent colors, and the twelve semantic tokens — all copied verbatim from `~/p/fabro-sh/fabro/apps/fabro-web/app/app.css` `@theme` block), a **dark-mode-only** posture with no `prefers-color-scheme` light branch, the brand-asset rule ("the logotype already includes the symbol — never use both together on the same page"), the atmosphere-background option carried from Fabro, and the accessibility constraints on the delete-confirmation dialog. It does **not** specify page layout (that is [[Standard - Receipt Page Sketch]]'s territory), **does not** specify structural content (that is [[Component - Report Metadata Page]]'s territory), and **does not** re-specify the CLI-output register (already owned by [[Principle - Minimal Public Surface Feels Calm]] and [[Standard - Interaction Patterns (CLI + Delete)]] at the register level; this Standard cross-references those without repeating them). The scope line is deliberately tight: C2 covers the page's *system* (fonts, colors, rules); C3 covers the page's *shape* (layout, placement); upstream cards already cover the CLI.

## WHERE: Ecosystem

- Paired sibling:
  - [[Standard - Receipt Page Sketch]] — C3 specifies the concrete layout (narrow centered column, three metadata rows, delete affordance, symbol-only footer) that this Standard's system (fonts, colors, tokens, atmosphere) renders into. C2 and C3 are built to be read together; neither is complete alone.
- Structural anchor:
  - [[Component - Report Metadata Page]] — A4 owns what the page *is* (four-element receipt, capability-URL-addressed, one rendered surface) and is deliberately visually silent; this Standard fills in the visual direction A4 defers to C2. Every visual choice on the page flows through this Standard.
- Pickup-point thesis:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the page is the pickup receipt, and this Standard carries the visual load of that receipt: narrow, terminal-flavored, confirmation-shaped. A design choice that would make the page feel like a triage dashboard (wider columns, table chrome, multi-column layouts) fails the thesis at the visual layer.
- Tone register (referenced, not owned):
  - [[Principle - Minimal Public Surface Feels Calm]] — F4 owns the *tonal* register (efficient and reassured, `bun`-calibrated) for every user-facing string on the CLI and the page; this Standard owns the *visual* register that must not contradict F4. A warm-but-visually-loud page breaks the tone handoff from terminal to browser.
- Narrowness commitment:
  - [[Principle - Narrow Forever]] — F6's "no visual-surface growth" commitment applies here directly. This Standard's palette and typography are the palette and typography forever; adding a second accent color for a future dashboard, or a second type family for marketing copy, would require expanding the product's surface. F6 rejects that expansion upstream of the visual choice.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped visual enrichment (rich charts, team-activity panels, symbolication visualizations) has no visual vocabulary in this Standard by design. The palette is calibrated for a four-element receipt; it has no secondary styles for the dashboard shapes F3 rejects.
- CLI output cross-reference:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — A7 owns the CLI's visual register (progress bar shape, error-output register, `NO_COLOR`-aware colored errors, delete-confirmation accessibility). This Standard's title mentions "CLI Output" for completeness but does not re-specify the CLI; anything about CLI colors, progress shapes, or error color-coding lives in A7 and in [[Principle - Minimal Public Surface Feels Calm]]. If page and CLI color vocabulary ever drift, the tiebreaker is F4's `bun`-calibrated register plus A7's mechanics, not this Standard.
- Delete-flow accessibility:
  - [[Standard - Accessibility Policy]] — A8's WCAG 2.2 AA floor for contrast, keyboard operability, and screen-reader semantics applies to every visual choice made in this Standard. The palette's semantic tokens were calibrated against AA contrast ratios; the delete-confirmation accessibility requirements below are C2's expression of A8's accessibility floor at the visual-component layer.
- Information-architecture coupling:
  - [[Standard - Information Architecture]] — A6's "one rendered surface, nothing else" commitment is the structural condition that makes this Standard's narrow visual vocabulary affordable. A palette calibrated for one page would break across a multi-page product; A6 guarantees it will not need to.
- Canonical names:
  - [[Standard - Noun Vocabulary]] — every label, heading, and string rendered with this Standard's typography uses canonical names exactly. The page's `<title>` and `<h1>` read "Report Metadata Page" or "Report" — never "receipt page," "status page," or "landing page."
- Brand provenance:
  - `~/p/fabro-sh/fabro/apps/fabro-web/app/app.css` (`@theme` block) — authoritative reference for fonts, palette, semantic tokens, and the `.bg-atmosphere` class. If Fabro's brand updates, propagate via a fresh copy into this Standard (the two products share a brand deliberately; see WHY below).
  - `~/p/fabro-sh/fabro/apps/marketing/public/symbol.svg` and `~/p/fabro-sh/fabro/apps/fabro-web/public/logotype.svg` — authoritative brand assets. Copy into `fabro.report`'s static assets at implementation time.
- Implementation-workflow reference:
  - The `/ui` skill is the recommended implementation tool when the Report Metadata Page is actually built — see [[Standard - Accessibility Policy]]'s workflow-hint section. This Standard is the product-level visual direction; `/ui` handles implementation-level execution.
- Source provenance:
  - `docs/alexandria/sources/receipt-page-visual-direction.md` — the raw source Bryan confirmed on 2026-04-23. Authoritative for dark-mode-only posture, typography lifted from Fabro, palette lifted from Fabro, narrow centered column, brand-asset "never both" rule, atmosphere-background carry-over, and the delete-confirmation accessibility reminders.

## WHY: Rationale

A sibling product inherits its brand. `fabro.report` exists as a companion service to Fabro — different runtime, different surface, same product family — and the honest visual direction for the companion is not "invent a new design system" but "inherit the established one and deviate only where the companion's surface demands it." Lifting typography and palette from the Fabro web UI collapses a design question into a copy operation. The copy operation is load-bearing precisely because it removes a decision surface that would otherwise accumulate drift: every downstream agent who might have reinterpreted "dark and terminal-flavored" lands on the same Geist/JetBrains Mono/navy-950/teal-500 system without a judgment call.

The **dark-mode-only** posture is honest, not restrictive. `fabro.report`'s audience is developers who arrived at the page from a CLI running in a terminal; the terminal is dark, the mental model is dark, and a light-mode branch would add implementation surface for a near-nil user population. Refusing the `prefers-color-scheme: light` media query at the Standard layer is cheaper than adding light styles now and discovering later that nobody uses them. If light-mode demand ever emerges, the decision to add it is a product-owner question that revisits [[Principle - Narrow Forever]] before it becomes a visual-direction update.

The **brand-asset rule** ("never both together") is the single rule an agent rendering the footer is most likely to violate. The logotype SVG already includes the symbol inside it; placing a separate symbol next to the logotype produces a redundant brand signature and a layout bug simultaneously. Naming this rule explicitly, and citing Bryan's 2026-04-23 confirmation, removes the possibility of the violation landing as a "harmless addition" in an implementation pass.

The **page-only scope** (with CLI coverage farmed out) is a deliberate split. A single card that tried to specify both the page's visual system and the CLI's colored-output register would either be too long to review carefully or too shallow to be authoritative on either surface. Keeping C2 tight on the page, and cross-referencing A7 + F4 for the CLI, keeps each Standard at a reviewable size and puts each specification in the card where downstream agents are most likely to look for it.

## WHEN: Timeline

Drafted 2026-04-23 from source material captured the same day (`docs/alexandria/sources/receipt-page-visual-direction.md`). Unblocks the visual-direction deferral [[Component - Report Metadata Page]] held open since A4 was drafted earlier on 2026-04-23. Stable. Revisions follow Fabro's brand: if Fabro updates fonts, palette, semantic tokens, or brand assets, this Standard updates via a fresh copy from `~/p/fabro-sh/fabro/apps/fabro-web/app/app.css`. Independent visual evolution (choosing a different teal, adopting a different type family, adding a light mode) is a product-owner decision that revisits [[Principle - Narrow Forever]] before it becomes a Standard update — this Standard explicitly refuses independent drift from Fabro at the Foundation of its specification.

## HOW: Specification

### Typography

**Font families** (copied verbatim from Fabro's `app.css` `@theme` block):

```css
--font-sans: "Geist", ui-sans-serif, system-ui, sans-serif,
  "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
--font-mono: "JetBrains Mono", ui-monospace, monospace;
```

**Font loading** (Google Fonts, matching Fabro's loading pattern):

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
<link rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Geist:wght@100..900&family=JetBrains+Mono:wght@400;500;600&display=swap" />
```

**Usage split** on the Report Metadata Page:

- **Geist (sans)** — section heading (if present), field labels, delete-button copy, footer text.
- **JetBrains Mono (mono)** — Report ID value, byte-size value, timestamp value (unless sans reads better at implementation), any terminal-flavored string.

The aesthetic is "CLI output made viewable" — mono-leaning, not mono-throughout. A page rendered in Geist only loses the receipt feel; a page rendered in JetBrains Mono only loses the warmth of Geist's sans for labels and affordances.

**Default size** (sensible default borrowed from Fabro's `diffs` component config in the same CSS file):

- Body text: `13px`
- Line height: `20px`

The implementation may scale these for the receipt's narrow column; the point is that Fabro's sibling components run at 13px/20px, and parity keeps the visual handoff between products coherent.

### Color Palette

Every color below is lifted verbatim from Fabro's `app.css` `@theme` block. Do not redefine or approximate; copy the CSS custom properties into `fabro.report`'s stylesheet unchanged.

**Primary — Teal**

```css
--color-teal-300: #B5DDEF;
--color-teal-500: #67B2D7;  /* primary accent */
--color-teal-700: #357F9E;
```

**Neutrals Dark — Navy** (dark-mode foundation)

```css
--color-navy-950: #0F1729;  /* page background */
--color-navy-800: #252C3D;  /* panel / card surface */
--color-navy-600: #4B5768;  /* muted text */
```

**Neutrals Light — Ice** (foreground on dark)

```css
--color-ice-50:  #F7F9FB;
--color-ice-100: #E8EDF3;  /* secondary text */
--color-ice-300: #A8B5C5;  /* tertiary / labels */
```

**Accents**

```css
--color-mint:  #5AC8A8;  /* success / confirmation */
--color-amber: #F0A45B;  /* warning / attention */
--color-coral: #E86B6B;  /* destructive (delete confirmation) */
```

**Semantic tokens** — the twelve load-bearing tokens the Report Metadata Page must use. Do **not** define new semantic tokens at this scale; reach for these first and flag a gap if a surface legitimately needs one that is not yet present.

```css
--color-page:          #0F1729;                      /* page background (= navy-950) */
--color-panel:         #252C3D;                      /* panel surface (= navy-800) */
--color-panel-alt:     #1a2133;                      /* alternate surface */
--color-overlay:       rgba(255, 255, 255, 0.04);    /* subtle overlay */
--color-overlay-strong: rgba(255, 255, 255, 0.10);   /* stronger overlay */
--color-fg:            #ffffff;                      /* primary text (Report ID value) */
--color-fg-2:          #E8EDF3;                      /* secondary text (timestamps, sizes) */
--color-fg-3:          #A8B5C5;                      /* tertiary text (labels) */
--color-fg-muted:      #4B5768;                      /* muted text (footer) */
--color-line:          rgba(255, 255, 255, 0.06);    /* subtle dividers */
--color-line-strong:   rgba(255, 255, 255, 0.12);    /* stronger dividers */
--color-focus:         rgba(103, 178, 215, 0.4);     /* focus ring (keyboard a11y) */
--color-on-primary:    #0F1729;                      /* text on teal-500 fills */
```

### Dark-Mode-Only Posture

The Report Metadata Page renders dark **always**. There is no `@media (prefers-color-scheme: light)` branch; there is no light-mode palette; there is no theme toggle on the page. Fabro's web UI uses the `scheme-only-dark` HTML class pattern to opt out of browser-level light-mode negotiation, and `fabro.report` follows the same pattern. An implementation that introduces a light-mode branch is a scope change, not a visual refinement — see WHEN above.

### Brand Assets (Load-Bearing Rule)

**The logotype already includes the symbol. Never use both together on the same page.** Either the symbol alone, or the logotype alone; never both. The rule exists because placing the logotype (which visually includes the symbol) next to a separate symbol file renders a redundant brand signature — the symbol appears twice.

- **`symbol.svg`** — source at `~/p/fabro-sh/fabro/apps/marketing/public/symbol.svg`. ~1 KB. The symbol alone, suitable for a subtle footer on a narrow-column receipt.
- **`logotype.svg`** — source at `~/p/fabro-sh/fabro/apps/fabro-web/public/logotype.svg`. ~8 KB, viewBox `1500×348`. The full logotype (symbol + wordmark) with the symbol baked in.

**Implementation rule:** copy both assets into `fabro.report`'s static-asset directory at implementation time. Do **not** reference them across a sibling repo path; the source-of-truth remains in the Fabro repo, but `fabro.report` carries its own copy. If the Fabro brand updates, propagate via a fresh copy.

**First-best-guess choice for the Report Metadata Page footer:** symbol alone. The symbol alone is subtler, fits a narrow centered column, and reads as "this is part of the Fabro family" without the wordmark's weight. The symbol may be linked to [Fabro's marketing site](https://fabro.sh) or the Fabro docs (TBD at implementation); it is **not** linked to a `fabro.report` homepage because there is no homepage — see [[Standard - Information Architecture]]. If a stronger brand presence is ever wanted, swap to the logotype; the "never both" rule still holds.

### Atmosphere Background

Fabro carries a `.bg-atmosphere` class in its `app.css`:

```css
.bg-atmosphere {
  background-color: var(--color-page);
  background-image:
    radial-gradient(ellipse 120% 60% at 15% 5%, rgba(53, 127, 158, 0.12) 0%, transparent 50%),
    radial-gradient(ellipse 80% 50% at 85% 90%, rgba(90, 200, 168, 0.07) 0%, transparent 45%);
  background-attachment: fixed;
}

.bg-atmosphere::before {
  /* Low-opacity SVG noise overlay — see ~/p/fabro-sh/fabro/apps/fabro-web/app/app.css for the full data-URL. */
}
```

**Recommended default:** use `.bg-atmosphere` on the Report Metadata Page for surface continuity with the Fabro web UI. The two subtle radial gradients plus low-opacity noise texture cost nothing operationally and keep the page visually continuous with Fabro.

**Judgment note (explicit):** the atmosphere recommendation is Raven's first-best-guess, not Bryan's signed-off direction. A flat `--color-page` background is an acceptable alternative if the atmosphere feels too busy for a four-element receipt. Override at implementation time if the rendered page feels over-decorated; the choice is compatible with the rest of this Standard either way.

### Delete-Confirmation Accessibility (Load-Bearing)

The Report Metadata Page has a delete button (structural per [[Component - Report Metadata Page]]; mechanics per [[Standard - Interaction Patterns (CLI + Delete)]]). Its confirmation dialog is the single interactive surface on the page and the place where accessibility most easily fails.

- **Preferred path:** a **native browser `confirm()` dialog**. Accessible by default (focus trap, ESC-to-cancel, screen-reader announcement, focus return all handled by the browser). If visual consistency is not a blocker, use this.
- **Alternate path:** a **custom modal** (e.g., for visual-consistency reasons driven by this Standard's palette). A custom modal must:
  - Trap focus inside the modal.
  - Support ESC-to-cancel.
  - Return focus to the delete button when the modal closes.
  - Use `role="dialog"`.
  - Use `aria-modal="true"`.
  - Use `--color-coral` for the destructive primary button.
  - Use `--color-focus` for the keyboard-focus ring.
- **Any missing element is an accessibility defect.** See [[Standard - Accessibility Policy]] for the accessibility floor this requirement satisfies.

### CLI Output (Cross-Reference, Not Specification)

The Standard's title includes "CLI Output" for completeness; the CLI's visual register is already specified upstream and is **not** re-specified here. Concretely:

- **Color register** — [[Standard - Interaction Patterns (CLI + Delete)]]'s "Error Output" and "Progress Indicator" sections own color-for-success, color-for-error, color-for-progress, and the `NO_COLOR`-aware fallback. Do not re-derive CLI colors from this Standard's palette.
- **Tone register** — [[Principle - Minimal Public Surface Feels Calm]]'s "efficient and reassured" register, calibrated against `bun`, governs every CLI string across all four canonical commands.
- **Accessibility register** — [[Standard - Accessibility Policy]]'s `NO_COLOR` floor applies to every colored CLI output.

If CLI output and the Report Metadata Page ever drift in visual vocabulary — e.g., the CLI renders errors in red while the page renders destructive actions in coral — the tiebreaker is the combination of F4 (tone) and A7 (CLI mechanics), not this Standard. This Standard's palette is calibrated for the page; the CLI inherits a *register*, not a specific hex value.

### Font Contrast Floor

Every foreground / background pair used on the Report Metadata Page must clear WCAG 2.2 AA contrast (4.5:1 for body text; 3:1 for large text and UI components), per [[Standard - Accessibility Policy]]. The semantic tokens in this Standard were calibrated against those ratios; implementers can pair `--color-fg` / `--color-fg-2` / `--color-fg-3` with `--color-page` / `--color-panel` / `--color-panel-alt` with confidence. New token combinations invented later are **not** pre-cleared — run the contrast check before shipping.

### Anti-Examples (What This Standard Refuses)

- **Wrong:** Defining a new teal by hex eyeballing Fabro's color ("#65b1d8" instead of `#67B2D7`). Lifted means lifted — copy the value from Fabro's `app.css`.
- **Wrong:** Adding a `@media (prefers-color-scheme: light)` branch with a light-mode palette. Violates the dark-mode-only posture; light mode is a product-owner scope question, not an implementation refinement.
- **Wrong:** Placing the logotype in the header and the symbol in the footer on the same page. Violates the "never both together" rule; the logotype already contains the symbol.
- **Wrong:** Introducing a second primary accent color (e.g., a purple for a "premium" feel). The palette is Fabro's palette; a second accent is a brand change that goes back to Fabro.
- **Wrong:** Using a sans-serif alternative to Geist (e.g., Inter) because Geist "felt heavy." Typography is lifted from Fabro; substituting a similar-but-different sans is the same category of drift as a color approximation.
- **Wrong:** Color as the only signal on a state (e.g., red text on coral hover with no text change on hover to indicate destructive). Every state must read as text without color; color is additive, not primary. See [[Standard - Accessibility Policy]].
- **Wrong:** A custom delete-confirmation modal without `role="dialog"`, `aria-modal="true"`, focus trap, ESC-to-cancel, and focus return. If visual consistency is not blocking, use native `confirm()` — it is accessible by default.
- **Wrong:** A decorative background animation (particles, drifting gradient, moving symbol). `.bg-atmosphere` is static; motion on the page is out of register with the calm receipt feel from [[Principle - Minimal Public Surface Feels Calm]].
- **Wrong:** Using Fabro's SVG graph overrides or `@pierre/diffs` brand overrides on `fabro.report`'s page. Those overrides exist for Fabro's specific components; the Report Metadata Page has neither diffs nor SVG graphs, and inheriting the overrides wholesale is a copy error, not a visual alignment.
- **Wrong:** Re-specifying CLI color tokens in this Standard (e.g., "errors render in `--color-coral`"). The CLI's color register is [[Standard - Interaction Patterns (CLI + Delete)]]'s territory — not this one.

### Test

Before shipping any visual choice for the Report Metadata Page, ask four questions in order. First: "Does this choice come from Fabro's `app.css` (palette, typography, semantic token) or from this Standard's rules (dark-mode-only, never-both, atmosphere option, confirm-dialog a11y)?" If neither, the choice has invented visual vocabulary and is drifting. Second: "Does this choice preserve WCAG 2.2 AA contrast for every foreground / background pair I'm rendering?" If not, the palette has been used in a combination it was not calibrated for; re-run the contrast check and adjust. Third: "If I replaced Fabro's brand tomorrow with a new palette, would this choice need to change mechanically or conceptually?" If conceptually, the choice is too entangled with one particular color value and should be re-framed against the semantic tokens. Fourth: "Does this visual choice contradict the tone-from-F4 or mechanics-from-A7?" If yes, the visual surface has drifted from the register the rest of the product carries, and the CLI-to-page handoff will feel wrong even though both halves look right in isolation.

## Sources

- `docs/alexandria/sources/receipt-page-visual-direction.md` (full file — authoritative for dark-mode-only posture, Geist + JetBrains Mono typography, palette and semantic tokens lifted from Fabro, narrow centered column, brand-asset "never both together" rule, atmosphere-background carry-over, delete-confirmation accessibility reminders; confirmed 2026-04-23 by Bryan)
- `~/p/fabro-sh/fabro/apps/fabro-web/app/app.css` (`@theme` block; `.bg-atmosphere` class; `@pierre/diffs` overrides reference — authoritative brand source, copy-from-here)
- `~/p/fabro-sh/fabro/apps/marketing/public/symbol.svg` (authoritative symbol asset)
- `~/p/fabro-sh/fabro/apps/fabro-web/public/logotype.svg` (authoritative logotype asset)
- `docs/alexandria/inventory/card-manifest.md` (C2 entry)

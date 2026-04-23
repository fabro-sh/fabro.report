# Source Material — Receipt Page Visual Direction

**Status:** raw source material, not a card. Captures the visual direction for the Report Metadata Page, confirmed 2026-04-23 by Bryan Helmkamp to unblock C2 and C3.
**Captured:** 2026-04-23
**Source:** direct confirmation from Bryan + inspection of the Fabro web UI at `~/p/fabro-sh/fabro/apps/fabro-web/` (authoritative brand reference).
**Feeds cards:** C2 `Standard - Design System (Receipt Page + CLI Output)` and C3 `Standard - Receipt Page Sketch`.
**Scope:** This source covers ONLY the Debug Bundle flow's Report Metadata Page. The Feedback flow has no rendered page (per A19 Fire-and-Forget Feedback).

---

## Load-Bearing Confirmations (Bryan, 2026-04-23)

- **Dark mode only.** No light-mode, no `prefers-color-scheme` respect. The page renders dark always.
- **Typography lifted from the Fabro web UI.** Fabro is Bryan's own product; `fabro.report` is a companion service and should inherit the same typography system rather than invent a new one.
- **Color palette lifted from the Fabro web UI.** Same sibling-product justification.
- **Page width: narrow (~500px, centered).** Four-line receipt; more width is wasted.
- **Chrome: content + tiny footer.** No header, no nav.
- **Branding:** use either the symbol OR the logo/wordmark from Fabro, **never both together.** (The logotype already includes the symbol; showing both is redundant and a brand-guideline violation.)
- **Metadata rows:** terminal-style `key: value` lines, left-aligned, monospace, subtle label-vs-value color contrast. *(Raven's first-best-guess, Bryan confirmed.)*
- **Delete button:** secondary text-link affordance. The destructive framing lives inside the confirmation dialog, not on the button itself. Not a big red button. *(Raven's first-best-guess, Bryan confirmed.)*
- **Report ID display:** full 32 hex chars visible, monospace, click-to-copy affordance. *(Raven's first-best-guess, Bryan confirmed.)*
- **Expired Report:** generic 404. No explicit retention-policy messaging on the page.
- **Deleted Report:** generic 404. No distinct state surface.

---

## Typography (From Fabro `app.css`)

**Authoritative reference file:** `~/p/fabro-sh/fabro/apps/fabro-web/app/app.css` (`@theme` block).

```css
--font-sans: "Geist", ui-sans-serif, system-ui, sans-serif,
  "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
--font-mono: "JetBrains Mono", ui-monospace, monospace;
```

**Font loading:** Fabro loads both from Google Fonts. For parity:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
<link rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Geist:wght@100..900&family=JetBrains+Mono:wght@400;500;600&display=swap" />
```

**Usage in the Fabro sibling (the `diffs` component config in the same file) gives a sensible default size:**
- Body text size: `13px`
- Line height: `20px`
- Sans for headers, mono for code / identifier content

**Applied to the Report Metadata Page:**
- **Sans (Geist)** for any labels, footer copy, button text.
- **Mono (JetBrains Mono)** for Report ID values, timestamps (optional), byte sizes, and anything terminal-flavored.
- A "receipt feels like CLI output made viewable" aesthetic — mono-leaning, not mono-throughout.

---

## Color Palette (From Fabro `app.css`)

**Primary — Teal**
- `--color-teal-300: #B5DDEF`
- `--color-teal-500: #67B2D7`  ← primary accent
- `--color-teal-700: #357F9E`

**Neutrals Dark — Navy** (the dark-mode foundation)
- `--color-navy-950: #0F1729`  ← page background
- `--color-navy-800: #252C3D`  ← panel / card surface
- `--color-navy-600: #4B5768`  ← muted text

**Neutrals Light — Ice** (foreground text on dark)
- `--color-ice-50: #F7F9FB`
- `--color-ice-100: #E8EDF3`  ← secondary text
- `--color-ice-300: #A8B5C5`  ← tertiary / labels

**Accents**
- `--color-mint: #5AC8A8`  ← success / confirmation
- `--color-amber: #F0A45B`  ← warning / attention
- `--color-coral: #E86B6B`  ← destructive (delete confirmation dialog)

**Semantic tokens (use these for the Report Metadata Page — do NOT define new ones):**
- `--color-page: #0F1729` (navy-950) — page background
- `--color-panel: #252C3D` (navy-800) — receipt card surface if used
- `--color-panel-alt: #1a2133` — alternate surface
- `--color-fg: #ffffff` — primary text (Report ID value)
- `--color-fg-2: #E8EDF3` — secondary text (timestamps, sizes)
- `--color-fg-3: #A8B5C5` — tertiary text (labels)
- `--color-fg-muted: #4B5768` — muted text (footer)
- `--color-line: rgba(255, 255, 255, 0.06)` — subtle dividers
- `--color-focus: rgba(103, 178, 215, 0.4)` — focus ring (keyboard a11y)
- `--color-on-primary: #0F1729` — text on teal-500 fills

**Background atmosphere (optional — carried from Fabro):**
Fabro uses a `.bg-atmosphere` class with two subtle radial gradients + low-opacity noise texture. The Report Metadata Page may inherit this for surface continuity with the Fabro web UI, or opt for flat `--color-page` for a more receipt-like feel. **Raven's read:** use atmosphere — it's visually continuous with Fabro and costs nothing operationally. Override in implementation if it feels too busy for a 4-element page.

---

## Brand Assets

**Authoritative files:**
- `~/p/fabro-sh/fabro/apps/marketing/public/symbol.svg` — the symbol only (~1 KB)
- `~/p/fabro-sh/fabro/apps/fabro-web/public/logotype.svg` — the full logotype (symbol + wordmark) (~8 KB, `1500×348` viewBox)

**Load-bearing rule (Bryan 2026-04-23):** **the logotype already includes the symbol. Never use both together on the same page.** Either one, never both.

**`fabro.report` choice:** either is acceptable. Raven's first-best-guess:

- **Use the symbol only in the footer.** The symbol alone is subtler, fits a narrow-column receipt, and reads as "this is part of the Fabro family" without the wordmark's weight. If the team later wants stronger brand presence on the receipt, swap to the logotype.
- **Symbol linked to `https://fabro.sh` or the Fabro docs** (TBD). Not to the `fabro.report` homepage — this service has no homepage.

Decide at implementation time; both are compatible with this Standard.

**Copying assets:** the assets should be copied into `fabro.report`'s repo (not referenced across a sibling path). Path likely `apps/...` depending on the Rust workspace layout, or served directly via the service's static file handling. Keep the source-of-truth in the Fabro repo; if the brand updates, propagate via a new copy.

---

## Report Metadata Page — Sketch (feeds C3)

Narrow-column receipt layout. All text left-aligned. Terminal-style metadata rows. Dark atmosphere background.

```
┌──────────────────────────────────────────┐
│                                          │
│   [page background — navy-950 + atm.]    │
│                                          │
│                                          │
│                                          │
│    Report                                │
│                                          │
│    ID          a3f8b29d01e4c76f2a1b...   │
│                (32-char mono, copy)      │
│                                          │
│    Uploaded    2026-04-23 14:32:09 UTC   │
│                (mono or sans — TBD)      │
│                                          │
│    Size        2.3 MB                    │
│                (mono, human-readable)    │
│                                          │
│                                          │
│    [ Delete this Report ]                │
│    (secondary text-link affordance)      │
│                                          │
│                                          │
│                                          │
│                                          │
│                ─────                     │
│                [ symbol ]                │
│                                          │
└──────────────────────────────────────────┘
         ~500px centered column
```

**Layout notes:**
- Page: centered single column, max-width ~500px, vertical padding generous (breathing room).
- Section heading "Report": sans, small, muted (`--color-fg-3`), or absent entirely — TBD at implementation.
- Metadata rows: label in sans + `--color-fg-3` (muted/label color); value in mono + `--color-fg` (primary text). Row layout: label left (fixed ~100px column), value flowing.
- Report ID: full 32-character string visible on a single line if width permits; otherwise wraps without breaking mid-word. Click-to-copy surfaces a small "copied" affirmation (mint-accented).
- Delete button: sans, secondary-link style (maybe coral-hinted on hover), keyboard-accessible, opens confirmation dialog (modal or inline).
- Footer: centered, small, muted. Either Fabro symbol (linked) + optional tagline, or just the symbol. No text-wordmark if symbol is shown.

**Deliberately absent from the page (reinforces F6 "Narrow Forever" + A6 IA):**
- No navigation bar.
- No "recent Reports" list.
- No logout / login / user menu.
- No search.
- No "related Reports" section.
- No share buttons.
- No comments / reactions.
- No tabs (overview vs files vs anything).

---

## Delete Confirmation Flow (feeds C2 + cross-references A7)

Per A7 `Standard - Interaction Patterns (CLI + Delete)`: delete confirmation is client-side, not a two-step server roundtrip.

**Native `confirm()` preferred** for accessibility (focus trap, ESC, screen reader support all handled by the browser). If the team later wants a custom modal for aesthetic reasons, it must:
- Trap focus
- Support ESC-to-cancel
- Return focus to the delete button on close
- Use `role="dialog"` and `aria-modal="true"`
- Contrast and colors drawn from this Standard (coral accent for the destructive primary button)

**Confirmation copy register** inherits F4 "efficient and reassured": no ceremony, no alarmism. Something like *"Delete this Report? This can't be undone."* + `[ Cancel ]` `[ Delete ]` — the delete button in the dialog is where the coral destructive color lives.

**After delete:** redirect to a generic 404. Per Bryan's confirmation, no distinct "Deleted" state page exists. The URL simply stops resolving.

---

## Error / State Pages

Per Bryan (2026-04-23):
- **Expired Report:** generic 404 page. No retention-policy messaging, no "this expired at X" note.
- **Deleted Report:** generic 404 page. Same as expired — the user can't tell from the page whether the Report expired naturally or was explicitly deleted.

**404 page itself** should inherit the design system (dark, narrow column, same typography, symbol in footer). Content: `404 Not Found` + one muted sentence. No back-link, no "try searching" — there's no search and nothing to go back to.

---

## CLI Output (the C2-Covers-Both consideration)

Note that C2 is titled `Standard - Design System (Receipt Page + CLI Output)` in the manifest. The CLI output register is already covered by F4 (`Principle - Minimal Public Surface Feels Calm`) and A7 (`Standard - Interaction Patterns (CLI + Delete)`) — efficient and reassured, `bun`-calibrated, `NO_COLOR`-aware colored errors.

**For this source's scope:** C2 primarily covers the Report Metadata Page visual system. The CLI output side is already specified in F4/A7 at the register level and doesn't need fresh visual direction — C2 just needs to cross-reference F4/A7 for the CLI side and specify the page side in detail.

---

## Implementation Workflow Note

Per `docs/alexandria/sources/accessibility.md`: when the team actually builds the Report Metadata Page, use the `/ui` skill. This source provides the product-level visual direction; the `/ui` skill handles the implementation-level execution.

---

## Implications Raven Observes (flag for review)

These are direct implications of the confirmed direction:

- **Tight coupling to Fabro's brand.** Any Fabro brand update (new colors, new font, new logo) propagates to `fabro.report`. This is the right tradeoff for a sibling product, but worth naming. If the brand diverges, the update is copy-from-Fabro, not independently-redesigned.
- **Dark-mode-only posture is honest.** The product is a CLI-first service; users arriving at the page are developers running a CLI in a terminal. Dark is the default mental model. Light-mode support would be extra work for a near-nil audience.
- **Generic 404 for both expired and deleted** is a privacy-ish posture — a Reporter who deleted a Report can't be retroactively outed by a different page for expiration. Accept as-is; don't over-engineer.
- **Click-to-copy on Report ID** is a small but real quality-of-life affordance. The "copied" confirmation should respect F4 register — subtle mint accent, no ceremony, no "✓ Copied to clipboard!" ceremony.
- **No retention-policy messaging anywhere on the page.** The 30-day policy is documented in A11 and carried in CLI output at upload time; Bryan does not want the page to re-surface it. This means a Reporter visiting the page a week before expiration has no in-context warning. Acceptable by product-owner decision; if abuse patterns emerge (Reporters surprised by expiration) we revisit.

---

## Open Questions (flagged for human judgment; non-blocking)

1. **Atmosphere vs flat background.** Raven's read is atmosphere (matches Fabro). If Bryan prefers a flatter receipt aesthetic, flag for implementation.
2. **Section heading "Report" vs no heading.** A tiny visual choice — whether to label the metadata block or let the metadata rows stand alone. Implementation decision.
3. **Footer content exact shape.** Just the symbol? Symbol + "fabro.sh" link? Something else? Implementation decision.
4. **Timestamp format.** ISO 8601 UTC (`2026-04-23T14:32:09Z`), human-readable (`April 23, 2026 at 2:32 PM UTC`), or relative (`2 days ago`)? Raven's read: ISO 8601 UTC — respects the terminal/receipt aesthetic and matches how Triagers work with timestamps. Flag for confirmation at implementation time.
5. **Byte size format.** `2.3 MB` vs `2,345,678 bytes` vs `2.3 MiB` (binary)? Raven's read: `2.3 MB` (decimal, human-readable) — the CLI probably reports the same format; keep consistency.

---

## What This Source Does Not Cover

- **Feedback flow visual direction.** Feedback has no page (A19 fire-and-forget). No visual direction needed.
- **CLI output visual register.** Already covered by F4 and A7. C2 can cross-reference those without re-specifying.
- **Email, Slack, or any other output surface.** The product has none.
- **Marketing site for `fabro.report`.** Out of scope — the product has no marketing site.

---

## Notes for Future Cards

When Sam drafts C2 + C3:

- **C2 `Standard - Design System`:** the authoritative spec for typography (Geist sans + JetBrains Mono), colors (semantic tokens lifted from Fabro), dark-mode-only posture, brand-asset rule ("never both"). Should be implementation-agnostic (not tied to a specific Rust templating engine). Cross-refs: F4 (tone register), A4 (page element list), A6 (IA — the page is the only rendered surface), A8 (accessibility contrast compliance), A7 (delete button + confirmation flow), Fabro repo (authoritative brand source).
- **C3 `Standard - Receipt Page Sketch`:** the concrete layout sketch — narrow column, metadata rows, delete affordance, footer. Likely includes a text-based ASCII sketch (like the one in this source) or a Figma/Excalidraw reference. Load-bearing: the layout is minimal forever; any addition is a scope question, not a visual-direction change.
- **Brand-asset copying action:** once C2 lands and implementation begins, copy `symbol.svg` + `logotype.svg` from the Fabro repo into `fabro.report`'s static assets. Document this as an implementation-time task, not a library concern.

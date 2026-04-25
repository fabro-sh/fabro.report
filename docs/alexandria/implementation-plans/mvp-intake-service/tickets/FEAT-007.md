---
id: FEAT-007
title: "Apply C2 design system + C3 layout to Report Metadata Page"
outcome: O-1
tier: must
enabler: false
blocked-by: [FEAT-006]
blocks: [FEAT-008]
cards: [Standard - Design System (Receipt Page + CLI Output), Standard - Receipt Page Sketch, Component - Report Metadata Page]
---

## Motivation

The Report Metadata Page is the only rendered surface in the product, so it must hit the C2/C3 visual bar in MVP rather than ship as a placeholder. A half-finished receipt page is a half-finished product surface, and there is no second page to compensate. The plain SSR from FEAT-005 is replaced here with the full Geist + JetBrains Mono / dark-mode / narrow-column treatment.

## Description

Replace the plain HTML from FEAT-005 with the full C2 + C3 visual treatment:

- **Typography:** Geist for headings/UI, JetBrains Mono for the metadata values (ID, timestamp, size). Self-host the fonts under `static/fonts/` and reference via `@font-face`; do not load from Google Fonts (network dependency on a third party for the page render is unwanted).
- **Color system:** Implement the Fabro-lifted palette as CSS custom properties in a single `:root` block. Define the semantic tokens C2 names (e.g., `--color-bg`, `--color-text`, `--color-text-dim`, `--color-coral` for destructive framing, `--color-focus` for the keyboard focus ring). Dark-mode-only — do not implement a light-mode toggle, do not respect `prefers-color-scheme: light`.
- **Layout per C3:** Single ~500px max-width column, centered horizontally on the viewport, generous vertical padding. Three terminal-style metadata rows (label-left, value-right, monospace value, dim label). Secondary text-link delete affordance (NOT a big red button) below the metadata block. Symbol-only footer (no nav links, no "About", no "Privacy") at the bottom.
- **Generic 404 state:** Same visual shell, brief calm message, no tombstone language. Used by the page handler for expired/deleted/never-existed Reports.

Inline all CSS in the `<head>` of the page (single SSR response, no external stylesheet) — the CSS is small enough that it doesn't justify a network round-trip.

## Context

- [[Standard - Design System (Receipt Page + CLI Output)]] (C2) — typography, palette, semantic tokens, dark-mode-only, brand-asset rule. WCAG 2.2 AA contrast floor (4.5:1 body, 3:1 large text and UI components).
- [[Standard - Receipt Page Sketch]] (C3) — narrow ~500px centered column, three terminal-style metadata rows, secondary text-link delete affordance, symbol-only footer, ISO 8601 UTC timestamp default, decimal `2.3 MB` size default.
- [[Component - Report Metadata Page]] — defers visual treatment to C2 + C3 + A8 (accessibility) + A7 (delete confirmation flow). Strict on the four-element rule.
- [[Principle - Minimal Public Surface Feels Calm]] — the visual treatment must read as efficient and reassured; no ceremony, no decoration that doesn't carry weight.
- The `/ui` skill is the recommended implementation tool per [[Standard - Accessibility Policy]] and [[Component - Report Metadata Page]] — use it for the actual rendering work to get a quality result quickly without inventing the design.
- Anti-patterns to avoid: light-mode support; mode toggle; loading fonts from Google Fonts CDN; adding a hero image, brand logo, or marketing copy; replacing the secondary text-link delete with a prominent red button; adding a footer with nav links; using non-Geist / non-JetBrains-Mono fonts.

## Acceptance Criteria

- [ ] Geist and JetBrains Mono fonts are self-hosted under `static/fonts/` and served by the Rust service (or via the ALB if cheaper); no external font CDN.
- [ ] The page CSS is inlined in `<head>` (no external stylesheet).
- [ ] Layout: single centered column with max-width ~500px (e.g., `max-width: 32rem`), centered horizontally, generous vertical padding.
- [ ] Three metadata rows render in terminal style (label-left dim, value-right monospace).
- [ ] Delete affordance is a secondary text-link styled below the metadata, NOT a button-shaped element. Color is `--color-coral` (destructive semantic token) or equivalent dim-red on the dark background.
- [ ] Footer is symbol-only (e.g., a small Fabro mark or wordmark only) — no text links, no copyright line.
- [ ] Generic 404 state shares the same shell and tone register; no "this Report expired" wording.
- [ ] Contrast measured with a checker hits WCAG 2.2 AA — body text ≥ 4.5:1, large text/UI ≥ 3:1.
- [ ] No client-side JavaScript yet (delete confirmation flow + accessibility audit lands in FEAT-008).
- [ ] Page renders correctly in dark mode regardless of `prefers-color-scheme`.

## Implementation Notes

- **Use the `/ui` skill** to do the actual rendering work. Brief it with the C2 / C3 cards directly; it can produce the HTML + CSS at the right design quality without inventing direction.
- Place the page template in `src/templates/receipt.rs` (or equivalent) — keep CSS in a Rust string adjacent to the template so the design lives with the page.
- Use `font-display: swap` on `@font-face` so text shows immediately during font load.
- Self-hosted fonts: download the OFL-licensed Geist and JetBrains Mono variable fonts; bundle WOFF2 only (modern browser baseline).
- Preserve the FEAT-005 `<meta name="robots" content="noindex,nofollow">`.
- Do not use Tailwind, a CSS framework, or any preprocessor — vanilla CSS is enough at this scale.

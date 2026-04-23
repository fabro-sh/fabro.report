# Spot-Check: C2 + C3 — Visual Direction Closes the Library

**Job:** Conan Job 2.5 — Spot-Check
**Date:** 2026-04-23
**Commit:** `9d38754`
**Cards:** C2 `Standard - Design System (Receipt Page + CLI Output)`, C3 `Standard - Receipt Page Sketch`
**Authoritative source:** `docs/alexandria/sources/receipt-page-visual-direction.md` (captured at `9df718f`)
**Lint state:** 0 errors; 37 pre-adjudicated warnings
**Prior reviews:** `docs/alexandria/reviews/`

---

## Verdict Summary

| Card | Grade | Status |
|------|-------|--------|
| C2 Design System | A | Ratified |
| C3 Receipt Page Sketch | A | Ratified |

Both cards meet Standard-layer fidelity to source, are paired cleanly without duplication, and integrate coherently with all 11 upstream cards. No blockers. The library closes at **24 of 24 ratified, zero blockers.**

---

## 1. C2 Quality & Source Fidelity

| Check | Finding |
|-------|---------|
| Typography: Geist stack verbatim | PASS. C2 L57–59 reproduces `--font-sans` with the full fallback chain (Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji) exactly matching source L32–33. `--font-mono` at L59 matches source L34. |
| Typography: JetBrains Mono stack verbatim | PASS. `ui-monospace, monospace` chain matches. |
| Google Fonts loading pattern | PASS. C2 L65–68 reproduces `preconnect` + `preconnect crossorigin` + `stylesheet` block exactly from source L40–43. Weights `100..900` for Geist and `400;500;600` for JetBrains Mono both preserved. |
| 12 semantic tokens | PASS. `--color-page`, `--color-panel`, `--color-panel-alt`, `--color-overlay`, `--color-overlay-strong`, `--color-fg`, `--color-fg-2`, `--color-fg-3`, `--color-fg-muted`, `--color-line`, `--color-line-strong`, `--color-focus`, `--color-on-primary` — all 12 present at C2 L124–137. Note: C2 adds `--color-overlay`, `--color-overlay-strong`, `--color-line-strong` which are load-bearing additions from the Fabro `app.css` and are not a source drift — they're legitimate completeness. |
| 9 shade tokens (teal ×3, navy ×3, ice ×3) | PASS. teal-300/500/700 (L92–94), navy-950/800/600 (L100–102), ice-50/100/300 (L108–110). All 9 present and hex values match source exactly. |
| 3 accents (mint, amber, coral) | PASS. `#5AC8A8`, `#F0A45B`, `#E86B6B` at L116–118, exactly matching source L76–78. |
| Dark-mode-only posture | PASS. Called "dark-mode-**only**" explicitly at L5, L48, L140–141, L208. Not softened to "dark-mode-first." `scheme-only-dark` HTML-class pattern named (L141). `@media (prefers-color-scheme: light)` refused explicitly (anti-example L208). |
| Brand-asset "never both" rule | PASS. L42 cites "Bryan's 2026-04-23 confirmation" explicitly; L145 states "The logotype already includes the symbol. Never use both together on the same page." Load-bearing framing preserved. |
| Delete-confirmation accessibility | PASS. L180–188: native `confirm()` preferred; custom-modal requires focus trap, ESC-to-cancel, focus return, `role="dialog"`, `aria-modal="true"`, plus `--color-coral` and `--color-focus` bindings. "Any missing element is an accessibility defect" (L189). |
| CLI cross-reference posture | PASS. L20–21 names A7 as owner of CLI visual register. L192–199 crosses to A7 (color/progress/error) and F4 (tone). No CLI color specification in C2 itself. |

### Sam's Flag (d): Tiebreaker Clause Verification

Source material does not explicitly spell out a page-vs-CLI drift tiebreaker. C2 L21 and L199 add: "If CLI output and the Report Metadata Page ever drift in visual vocabulary … the tiebreaker is the combination of F4 (tone) and A7 (CLI mechanics), not this Standard."

**Assessment: APPROVE.** This clause is a legitimate load-bearing extrapolation. The scope question ("what happens if CLI and page drift?") is a real future risk implied by the title pairing ("Receipt Page + CLI Output"), and the resolution (F4 + A7 win over C2 at the register level) is the only answer consistent with the cross-reference posture already named in source. Without this clause, a future agent reading C2 cold could conclude C2 owns CLI colors. The clause closes that escape hatch. Substance-preserving, not scope-expanding.

---

## 2. C3 Quality & Sketch Fidelity

| Check | Finding |
|-------|---------|
| ASCII sketch reproduction | PASS. C3 L51–81 reproduces the source sketch (source L121–150) with identical box shape, content placement, annotations ("32-char mono, copy", "mono or sans — TBD", "mono, human-readable", "secondary text-link affordance"), divider, and `[ symbol ]` footer. `~500px centered column` caption preserved. Byte-exact where it matters. |
| Narrow ~500px centered column | PASS. L5, L80, L87 all specify `max-width: ~500px` centered. |
| 3 metadata rows (ID, Uploaded, Size) | PASS. L90 names `ID`, `Uploaded`, `Size` as the canonical labels. |
| Secondary text-link delete affordance | PASS. L95 specifies Geist sans, body-text size, "not red", with hover hinting at `--color-coral`. |
| Symbol-only footer | PASS. L96: symbol alone (or logotype, never both), optional divider above, no text wordmark. |
| "What The Sketch Doesn't Show" absence list | PASS (12 items, exceeds source's 8). L102–113 enumerates: no nav, no recent/my uploads, no search, no tabs, no share, no related-Reports rail, no comments/reactions, no logout/login, no Debug Bundle contents, **no retention-policy messaging**, no post-upload feedback prompt, no related/next-steps link rail. C3 legitimately expanded the list from source's 8 to 12 by importing F3 and F6 drift shapes. Source-consistent expansion. |
| Expired → generic 404 | PASS. L117 — "The page served is a generic **404**. No retention-policy messaging, no 'this Report expired at X' note, no 'upload another' link." |
| Deleted → generic 404 | PASS. L119 — distinct-state explicitly rejected with Bryan 2026-04-23 citation. |
| Distinct-state rejection | PASS. L119 + anti-example L154 both reject distinct expired/deleted state pages. |
| Format defaults: ISO 8601 UTC | PASS. L129–132 names two shapes (`2026-04-23 14:32:09 UTC` and `2026-04-23T14:32:09Z`); rejects human-relative and locale-formatted. Labeled "Recommended Defaults (Implementation-Refinable)" at L125. |
| Format defaults: decimal byte size | PASS. L133 `2.3 MB`, `17.8 KB`, `890 bytes`; rejects raw bytes and binary (MiB). Labeled "Implementation-Refinable." |

---

## 3. C2 ↔ C3 Pairing (Complementarity, Not Duplication)

| Dimension | C2 Owns | C3 Owns |
|-----------|---------|---------|
| Fonts, palette, semantic tokens | Yes | References |
| Brand-asset "never both" rule | Yes (load-bearing) | References |
| Dark-mode-only posture | Yes | Inherits |
| Atmosphere background option | Yes (with Raven's judgment note) | References |
| Delete-confirmation a11y requirements | Yes (component-visual layer) | References (for affordance shape) |
| Narrow ~500px centered column | References | Yes |
| 3 metadata rows placement | References | Yes |
| Delete affordance shape (secondary text-link) | References | Yes |
| "What The Sketch Doesn't Show" absence list | — | Yes (load-bearing) |
| State-page behavior (generic 404) | — | Yes |
| Recommended default formats | — | Yes |

Cross-references are mutual and clean. C2 WHERE L10 and C3 WHERE L10 each name the other as "paired sibling" with the explicit framing "neither is complete alone." No duplicated specification found. A Factory-mode agent reading only C2 gets the system; reading only C3 gets the layout; together they specify the page.

---

## 4. Sam's Five Judgment Flags

| Flag | Description | Verdict | Reasoning |
|------|-------------|---------|-----------|
| (a) | Atmosphere-background carried as Raven's recommendation, overridable | APPROVE | Source L93 explicitly names this as "Raven's read"; C2 L173–174 preserves the honest judgment framing with the override path. Substance-preserving; does not over-commit to a Bryan-confirmed choice. |
| (b) | Symbol-alone footer as first-best-guess, logotype acceptable alternative | APPROVE | Source L106–107 + L158 carry both options with symbol-alone as first-best-guess. C2 L152 and C3 L96 both preserve this posture. Honest confidence-level flagging. |
| (c) | ISO 8601 UTC + decimal byte size as "Recommended Defaults (Implementation-Refinable)" | APPROVE | Source L230–231 explicitly flags both as Raven's reads requiring Bryan confirmation at implementation. C3 L125–135 preserves the "Implementation-Refinable" label. Anti-examples correctly reject relative/locale-formatted timestamps and binary units. |
| (d) | C2's CLI cross-reference grew a tiebreaker clause (page vs CLI color drift → F4 + A7 win) | APPROVE | See section 1 above. Legitimate load-bearing extrapolation that closes a real scope question the title "Receipt Page + CLI Output" would otherwise leave open. Not an over-reach — the clause names the tiebreaker already implied by F4/A7 ownership. |
| (e) | Asset-copy-from-Fabro rule embedded in C2 HOW (vs floating as implementation note) | APPROVE | Source L112 already stated "assets should be copied into `fabro.report`'s repo (not referenced across a sibling path)." C2 L150 makes this a Standard-level rule: "Do **not** reference them across a sibling repo path." Sam's argument is correct — this is a cross-repo hygiene rule that belongs at Standard tier, not as a floating implementation hint. Embedded correctly. |

All five flags: **APPROVE**. No TRIM or REJECT required.

---

## 5. A4 Cleanup Verification

| Check | Finding |
|-------|---------|
| No remaining "blocked on visual direction" forward-reference | PASS. A4 L7, L39–40, L52, L114, L117–124 all reference C2 and C3 as ratified paired Standards. The WHEN section (L52) explicitly states "Visual direction confirmed 2026-04-23 … unblocking the visual-direction deferral this card previously held open." No blocked-status language remains. |
| 4-element scope unchanged | PASS. A4 L58–63 still enumerates exactly: (1) Report ID, (2) Upload timestamp, (3) Bundle size, (4) Delete button. No fifth element introduced. Symbol-footer is explicitly carved out at C3 L12 as not part of A4's four structural elements because it carries no Report data. |
| C2/C3 cross-refs are substantive | PASS. A4 L39 names C2's specific contributions (typography, Fabro-lifted palette, semantic tokens, dark-mode-only posture, brand-asset rule, atmosphere option, delete-confirmation a11y). A4 L40 names C3's (narrow ~500px centered column, three metadata rows ID/Uploaded/Size, secondary text-link, symbol-only footer, generous vertical padding, absence list, 404 state-page behavior). Both substantive; neither generic. |
| Markdown-link conversion | PASS. A4 L124 `[Fabro's marketing site](https://fabro.sh)` and L128 `/ui` skill references use proper markdown. |
| HOW section cross-refs | PASS. A4 L58 cites C2 and C3 together for visual arrangement; L61–63 defer format recommendations to C3 ("per [[Standard - Receipt Page Sketch]]"); L63 defers delete-affordance shape to C3. |

---

## 6. Backlink Audit (11 Upstream Cards)

| Card | Backlink anchor to C2/C3 | Substantive? |
|------|---------------------------|---------------|
| F1 `Product Thesis - Pickup Point` | L52–53: C2 ("narrow, terminal-flavored visual system is the pickup-receipt feel expressed at the visual layer") + C3 ("narrow centered column, three metadata rows, secondary delete affordance are the pickup-receipt shape expressed at the layout layer") | Yes — each anchor names the load-bearing thesis-expression role |
| F2 `Standard - Noun Vocabulary` | L37–38: C2 ("uses 'Report Metadata Page' and 'Report' exactly") + C3 ("'Report' as the optional section heading, 'Delete this Report' as the delete-affordance text") | Yes — anchors on canonical-name enforcement at the rendered-surface layer |
| F3 `Reject bun.report Drift` | L32–33: C2 (visual-surface drift — independent brand evolution, second accent color, light-mode branch) + C3 (layout drift — bun.report content tabs, symbolication sections, distinct-state pages) | Yes — each names a different drift layer (visual vs layout) |
| F4 `Minimal Public Surface Feels Calm` | L37: C2 cross-references F4 for CLI register and inherits "warm but not loud" visual calibration. L38: C3 as "layout expression of the calm register" | Yes — tone-paired-with-visual handoff explicit |
| F6 `Narrow Forever` | L42–43: C2 (visual-surface growth — second accent color, second type family, light-mode branch) + C3 (layout growth — nav rail, tab bar, related-Reports sidebar, retention banner) | Yes — each enumerates the specific drift shapes F6 forecloses at each layer |
| A4 `Component - Report Metadata Page` | L39–40: C2 names the visual system (typography, palette, tokens, dark-mode, brand-asset rule, atmosphere option, delete a11y); C3 names the concrete shape (column, rows, affordance, footer, absence list, 404s) | Yes — structural card's paired-Standards frame is explicit |
| A6 `Information Architecture` | L39–40: C2 ("IA names the surface; C2 names how it looks"); C3 ("IA's 'no navigation, no hierarchy, no second page' commitment is visible in C3's layout — no header nav, no breadcrumbs, no tab bar") | Yes — IA-layer commitment traced to layout-layer expression |
| A7 `Interaction Patterns (CLI + Delete)` | L36–38: C2 owns visual system + delete-confirmation a11y (paired with A7 mechanics); C3 owns the secondary-text-link affordance shape (paired with A7's delete mechanics) | Yes — mechanics-vs-affordance split named cleanly |
| A8 `Accessibility Policy` | L27–29: C2 owns color choices and delete-confirmation a11y requirements paired with this Standard's calibration; C3's focus order + semantic-HTML guidance matches A8's keyboard-first + semantic-HTML requirements | Yes — a11y requirements appear in both C2 and A8 by design, with explicit attribution of which owns what |
| A9 `Capability URLs over Auth` | L38: C3 "keeps the Report ID fully visible as a selectable unit (no truncation with 'show full' toggles) precisely so URL-possession-as-capability is not obscured at the rendered layer" | Yes — access-model linkage to rendered-handle preservation |
| A11 `30-Day Retention Default` | L31: C3 "specifies the generic-404 shape for expired Reports (no retention-policy messaging, no 'this Report expired at X' note); this Decision is the retention policy that makes those URLs stop resolving, and C3's refusal to render a distinct expired state is the layout-layer expression of this Decision's 'no rendered expired surface' shape" | Yes — retention policy traced to layout-layer expression |

All 11 upstream backlinks verified substantive. No generic "this card is referenced by C2" entries.

### Specifically verified:

- **F4 (tone register paired with visual register):** C2 L15–16 names F4 as owner of tonal register, then C2 states that the visual register "must not contradict F4." F4 L37 backlink reciprocates. Handoff verified.
- **A4 (page-structure ↔ page-visual pairing):** A4 L39–40 names both C2 and C3 with specific contributions. C2 L12 and C3 L11–12 each name A4 as the structural anchor. Three-card triangulation (A4 + C2 + C3) verified.
- **A8 (accessibility contrast requirements):** A8 L28 names C2 as owner of color choices with WCAG 2.2 AA contrast floor as the calibration constraint. C2 L203 "The semantic tokens in this Standard were calibrated against [WCAG 2.2 AA] ratios." Bidirectional.

---

## Additional Checks

| Check | Finding |
|-------|---------|
| Vocabulary compliance (canonical "Report Metadata Page" in prose) | PASS. C2 prose consistently uses "Report Metadata Page" (not "Receipt Page"). Card titles use "Receipt Page" as an accepted nickname per F2 guidance, but prose stays canonical. C3 follows the same pattern. F2 L38 explicitly allows C3's title as acceptable shorthand while requiring canonical prose. |
| Source citations accurate | PASS. Both cards cite the source file as a whole (C2 L224, C3 L168) rather than specific line numbers, which is appropriate given the source's header-vs-content-line drift pattern. Explicit load-bearing items cited within WHY/HOW sections reference specific source concepts (e.g., "confirmed 2026-04-23 by Bryan") rather than L-numbers — this is the correct posture given the source's density. |
| Forward references | PASS. All wikilinked cards exist in the library: Product Thesis - Pickup Point, Principle - Minimal Public Surface, Principle - Narrow Forever, Principle - Reject bun.report Drift, Standard - Noun Vocabulary, Standard - Interaction Patterns, Standard - Accessibility Policy, Standard - Information Architecture, Component - Report Metadata Page, Decision - Capability URLs over Auth, Decision - 30-Day Retention Default. Zero dangling references. |
| Manifest consistency | PASS. Manifest L79–80 lists C2 and C3 with EXISTS status as of 2026-04-23. Build order 8 and 9 respectively. Blocked Cards section (L210) says "(None — all 24 manifested cards have source material.)" Manifest matches the current library state. |

---

## Implementation-Readiness Implications

This is the final spot-check. The library now contains:

- **6 Foundation cards** (F1–F6) — Product Thesis, canonical Noun Vocabulary, drift-rejection Principle, tone-register Principle, strategy Principle, narrow-forever Principle.
- **4 Core cards** (C1–C4) — Competitive Context, Design System (C2), Receipt Page Sketch (C3), Audience Decision.
- **14 Amplifier cards** (A1–A19, with A-numbers skipped by design) — two Agents + Feedback Submitter, three Components, three Systems, four Standards, seven Decisions.

All 24 cards ratified. Zero blocked. Zero dangling wikilinks. Zero lint errors (37 pre-adjudicated warnings, all structural-property acceptance).

**Factory-mode implementation can now proceed** against:

1. **Report Metadata Page implementation** — A4 (structure) + C2 (visual system) + C3 (layout) + A7 (delete interaction) + A8 (accessibility) together fully specify the page. The `/ui` skill is the recommended on-ramp (per A8 L73).
2. **CLI implementation** — A7 + F4 + A8 + F2 together fully specify all four canonical commands.
3. **Storage and topology** — A5 + A14 + A12 + A17 together fully specify the AWS-native Rust-on-Fargate stack and two-intake storage model.

Brand-asset copy action (copy `symbol.svg` and `logotype.svg` from `~/p/fabro-sh/fabro/` into `fabro.report`'s static assets) is the single implementation-time prerequisite named in C2 L150 and C3 L96.

No remaining product-owner inputs are required to begin implementation. Open questions flagged in the visual-direction source (atmosphere vs flat, section heading Y/N, footer exact shape, timestamp format, byte size format) are all implementation-refinable within the bounds C2 and C3 set.

---

## Verdict

**APPROVED.**

The library is now fully closed at **24 of 24 cards ratified, zero remaining blockers.** C2 and C3 meet Standard-layer fidelity to source, pair cleanly as complementary siblings (system vs shape), and integrate substantively with all 11 upstream cards. Sam's five judgment flags are all legitimate. A4 cleanup is complete with no residual blocked-status language. The 4-element scope is intact.

Factory-mode implementation is unblocked.

**Status: DONE**

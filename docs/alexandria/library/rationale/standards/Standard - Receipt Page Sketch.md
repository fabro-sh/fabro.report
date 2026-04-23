# Standard - Receipt Page Sketch

## WHAT: Definition

The **concrete layout sketch** for the Report Metadata Page — the one rendered surface of `fabro.report`. Where [[Standard - Design System (Receipt Page + CLI Output)]] specifies the *system* (fonts, colors, tokens, brand rules, dark-mode posture), this Standard specifies the *shape*: a narrow ~500px centered single column, a small section heading ("Report" or absent), three terminal-style `label: value` metadata rows (Report ID, Uploaded timestamp, bundle size), a secondary text-link delete affordance, and a symbol-only footer. The two Standards are paired — neither is complete alone. This Standard also fixes the "what the sketch deliberately omits" list (no navigation, no listing, no search, no tabs, no share buttons, no related-Reports, no comments / reactions), the state-page behavior (expired and deleted Reports both resolve to a generic 404 that inherits the design system; no distinct-state surface), and the recommended-default formats for the rendered timestamp and byte-size values. The sketch is canonical: the rendered page must hit this shape. Visual choices that keep the shape but vary details (exact row spacing, exact footer content, whether to show the "Report" heading) are implementation refinements; changes to the element set or the absence-list are scope changes that revisit the manifest.

## WHERE: Ecosystem

- Paired sibling:
  - [[Standard - Design System (Receipt Page + CLI Output)]] — C2 specifies the *system* (fonts, palette, semantic tokens, brand-asset rule, dark-mode-only posture, atmosphere background, delete-confirmation accessibility) this Standard's layout is rendered into. Read C2 and C3 together; a page built against one without the other will have the right shape with the wrong system or the right system with the wrong shape.
- Structural anchor:
  - [[Component - Report Metadata Page]] — A4 names the four structural elements (ID, upload timestamp, bundle size, delete button) this Standard arranges into a concrete layout. A4 is deliberately visually silent and defers arrangement to this Standard; this Standard inherits the element set from A4 unchanged and adds the symbol footer as the fifth visible element (the footer is not part of A4's four structural elements because it carries no Report data).
- Pickup-point thesis:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the sketch *is* the pickup receipt. Narrow centered column, generous breathing room, a single affordance — the layout shape is the visual expression of the pickup-point thesis. Any layout that would accommodate triage surface area (wider columns to fit tables, tabs for "files vs overview," a sidebar for "recent Reports") abandons the thesis at the layout layer.
- Tone register:
  - [[Principle - Minimal Public Surface Feels Calm]] — the calm, terminal-flavored receipt feel is F4's tonal register expressed in layout: breathing room, left-aligned text, monospace values for ID and size, no decorative chrome. A layout that crammed the three metadata rows together with a loud delete button would feel efficient-but-not-reassured; the generous vertical padding is what lands the "reassured" half.
- Narrowness commitment:
  - [[Principle - Narrow Forever]] — F6's "Narrow Forever" commitment is visible in this sketch as *absence*. The "What The Sketch Doesn't Show" section below is the load-bearing expression of F6 at the layout layer: every absent element is a surface F6 refuses to let grow. The absence list is prescriptive, not decorative.
  - [[Principle - Reject Like bun.report Drift]] — every `bun.report`-shaped visual enrichment (content tabs, symbolication render, feedback prompt, team-assignment banner, related-Reports rail) is specifically named absent in the "What The Sketch Doesn't Show" section. F3's drift list drove the absence list directly.
- Information architecture:
  - [[Standard - Information Architecture]] — A6's "one rendered surface, nothing else" commitment is what this Standard's layout serves. The page has no navigation because the IA has no other pages; the sketch reflects that structurally (no header nav, no breadcrumbs, no "back to list" link) and the IA commitment above is what keeps that structural choice coherent.
- Interaction coupling:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — A7 owns the delete *interaction* (client-side confirmation, authorization via URL possession, 50 MB bundle cap at upload time); this Standard owns the delete *affordance* shape (secondary text-link, not a big red button). The affordance shape is load-bearing because the destructive framing lives inside the confirmation dialog, not on the page-level button.
- Accessibility:
  - [[Standard - Accessibility Policy]] — A8's WCAG 2.2 AA floor, keyboard-first operation, and semantic-HTML requirements constrain every layout choice here. The sketch's focus order (heading → metadata rows → delete link → footer) must be the logical reading order under assistive tech; skipping a heading level or making the delete link focusable-but-not-reachable would fail the floor.
- Canonical names:
  - [[Standard - Noun Vocabulary]] — the page's heading, metadata labels, and delete copy use canonical names exactly. The heading (if present) is "Report" — not "Receipt" or "Your Upload." The delete affordance reads "Delete this Report" — not "Delete" alone or "Remove."
- Source provenance:
  - `docs/alexandria/sources/receipt-page-visual-direction.md` — the raw source Bryan confirmed 2026-04-23, including the ASCII sketch reproduced below and the absence list.

## WHY: Rationale

The sketch exists because the Report Metadata Page is the *whole* web surface of the product, and "whole web surface" deserves a canonical layout rather than a description. A Standard that said "a narrow receipt with metadata rows and a delete button" would let every implementation agent land a different arrangement — tight vs generous padding, centered vs left-gutter column, prominent vs quiet delete — and each variation would be defensibly on-brief while silently drifting from the pickup-point feel. Fixing the layout as an ASCII sketch with named element placements collapses that interpretive space to one shape.

The **absence list** carries more weight than the presence list. A layout that shows the three metadata rows and the delete button is on-brief; a layout that *also* shows a "related Reports" rail or a "files in this bundle" tab is simultaneously on-brief at the element layer and drifting at the shape layer. Agents reading only the presence list will land the shape correctly and still accumulate drift by addition. The "What The Sketch Doesn't Show" section is where this Standard does its load-bearing work: every absence is a scope choice the product has made, and naming each absence explicitly closes a growth path that would otherwise land as a "harmless addition" in some future PR.

The **state-page choice** — expired and deleted Reports both resolve to a generic 404, distinct-state explicitly rejected — is honest. A "this Report expired on X" page would be a new user-facing surface (what does its copy say? does it link to "create a new Report"? does it distinguish expired from deleted?), and each of those questions is the start of a scope expansion [[Principle - Narrow Forever]] refuses. A generic 404 is also a mild privacy posture: a Reporter who deleted a Report cannot be retroactively outed by a different page shape for expiration. The 404 inherits the design system (dark, narrow column, symbol in footer) so the state page still feels like `fabro.report`, not like a raw web-server error.

The **timestamp-and-size formats** are Raven's first-best-guess, flagged honestly as implementation-refinable. ISO 8601 UTC for timestamps respects the terminal / receipt aesthetic and matches how Triagers tend to work with timestamps in logs. Decimal byte sizes (`2.3 MB`) match typical CLI byte reporting and keep the CLI-to-page handoff feeling coherent. Both formats are recommended, not mandated, because they are the kind of detail that reasonable implementers might want to refine with real content on the page; flagging them as defaults rather than hard rules keeps this Standard honest about its confidence level.

## WHEN: Timeline

Drafted 2026-04-23 from source material captured the same day (`docs/alexandria/sources/receipt-page-visual-direction.md`). Paired with [[Standard - Design System (Receipt Page + CLI Output)]] (C2) in the same drafting pass; the two Standards unblock the visual-direction deferral [[Component - Report Metadata Page]] held open since A4. Stable. Revisions to the element set or the absence list are scope changes that revisit the manifest and the pickup-point thesis; revisions to layout details (exact column width, exact vertical padding, whether to render the "Report" heading) are implementation refinements appropriate to land as Standard updates informed by real rendered-page experience. The timestamp and byte-size formats are explicitly recommendation-grade and may be refined at implementation time without escalating to a Standard update.

## HOW: Specification

### Canonical ASCII Sketch

Narrow-column receipt layout. All text left-aligned. Terminal-style metadata rows. Dark atmosphere background (or flat `--color-page` — see [[Standard - Design System (Receipt Page + CLI Output)]] atmosphere-background section).

```text
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

This ASCII is canonical. The rendered page must hit this shape. Visual choices that preserve the shape but refine details (exact padding, exact divider style, whether the footer shows a divider at all, whether the "Report" heading is rendered) are implementation refinements that do not violate this Standard.

### Element Placement

- **Page container:** single column, `max-width: ~500px`, horizontally centered in the viewport, generous vertical padding (breathing room is load-bearing — a cramped receipt feels terse, not calm). Background: `var(--color-page)` with `.bg-atmosphere` as the recommended default per C2.
- **Section heading "Report":** Geist sans, small, muted (`--color-fg-3`), optional. An implementation may render the heading or omit it entirely; both are in-Standard. A heading larger than the metadata-value text, or in a louder color, drifts from the receipt feel.
- **Metadata rows (3):** terminal-style `label: value` lines, left-aligned, with a fixed-width label column (~100px) and a flowing value column.
  - **Label:** Geist sans, `--color-fg-3` (tertiary / muted). Specifically: `ID`, `Uploaded`, `Size`.
  - **Value:** JetBrains Mono, `--color-fg` (primary, white). Values are mono because they are identifier-shaped or number-shaped content — the page is a terminal-flavored receipt.
- **Report ID value specifics:** full 32-character hex string visible. On a narrow viewport that would break the line mid-ID, wrap without breaking mid-word — the full string remains one selectable unit. Click-to-copy affordance: clicking the ID copies it to the clipboard and surfaces a subtle "copied" affirmation in `--color-mint` (no ceremony — see [[Principle - Minimal Public Surface Feels Calm]]'s anti-emoji-cascade guidance).
- **Uploaded value specifics:** recommended default ISO 8601 UTC — `2026-04-23 14:32:09 UTC` or `2026-04-23T14:32:09Z`. Format is refinable at implementation time; see "Recommended Defaults" below.
- **Size value specifics:** recommended default decimal human-readable — `2.3 MB`, `17.8 KB`, etc. Format is refinable; see "Recommended Defaults" below.
- **Delete affordance:** secondary text-link style. Geist sans, at or near the body-text size, placed below the metadata rows with breathing room above it. The affordance reads "Delete this Report" (canonical name per [[Standard - Noun Vocabulary]] — not "Delete," "Remove," or "Delete Report" shortened). Hover state may hint at `--color-coral`; resting state is not red. The destructive framing lives inside the confirmation dialog per [[Standard - Interaction Patterns (CLI + Delete)]] and [[Standard - Design System (Receipt Page + CLI Output)]], not on the page-level affordance.
- **Footer:** centered, small, muted, with generous space above it. Shows the Fabro symbol only (or the logotype — never both, per [[Standard - Design System (Receipt Page + CLI Output)]] brand-asset rule). Recommended default: symbol alone, optionally linked to [Fabro's marketing site](https://fabro.sh) or Fabro docs. The footer may include a thin divider (`--color-line`) above the symbol; it may not include a text wordmark alongside the symbol (that combination violates the "never both" rule).

### What The Sketch Doesn't Show (Load-Bearing)

This section is the layout-layer expression of [[Principle - Narrow Forever]] and [[Principle - Reject Like bun.report Drift]]. Each absence is a deliberate choice, not an oversight. An implementation that reintroduces any item below is drifting, and the correct response is to delete the reintroduction, not to ask this Standard to add it.

- **No navigation bar.** Not at the top, not at the side. The page has no other pages to link to (see [[Standard - Information Architecture]]).
- **No "recent Reports" or "my uploads" list.** Reporters have no identity on this service; there is nothing to list (see [[Decision - Capability URLs over Auth]]).
- **No search.** No search box, no filter, no "find a Report by ID" form. The URL is the whole handle.
- **No tabs.** Not "Overview / Files / Metadata," not "Details / Contents." There is one page; it shows what it shows.
- **No share buttons.** No "copy URL," no social-share icons, no "send to Slack." Sharing is a Reporter-side action out-of-band (see [[Product Thesis - Pickup Point, Not Triage Surface]]).
- **No related-Reports rail.** No "other Reports from this team," no "similar bundle sizes," no "Reports uploaded around the same time." Reports do not compose into collections.
- **No comments / reactions.** No comment thread, no "mark as triaged," no emoji reactions. The page is a Reporter-facing confirmation surface; team-facing affordances happen out-of-band.
- **No logout / login / user menu.** No identity, no session, no user affordance of any kind (see [[Decision - Capability URLs over Auth]]).
- **No Debug Bundle contents.** Not a file listing, not a stack-frame render, not a log excerpt (see [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Reject Like bun.report Drift]]).
- **No retention-policy messaging.** No "this Report expires in X days," no "retention policy: 30 days" line, no progress-bar-to-expiration. The 30-day retention is documented in [[Decision - 30-Day Retention Default]] and communicated at CLI upload time; the page does not re-surface it (see the "Generic 404 for Expired Reports" reasoning below).
- **No post-upload feedback prompt.** No "how was your upload experience?" surface — forbidden by [[Principle - Reject Like bun.report Drift]].
- **No "related / next steps" link rail.** No "upload another Report," no "view team activity" — none of those surfaces exist.

### State Pages

**Expired Report** — the URL resolves as not-found. The page served is a generic **404**. No retention-policy messaging, no "this Report expired at X" note, no "upload another" link.

**Deleted Report** — the URL resolves as not-found. The page served is a generic **404**. **Distinct-state is explicitly rejected** (Bryan 2026-04-23): a "this Report was deleted" surface would be a new user-facing element (copy? link? contact?), and this Standard refuses that surface at the layout layer.

**Generic 404 page** — inherits the design system from [[Standard - Design System (Receipt Page + CLI Output)]]. Dark `--color-page` background (with `.bg-atmosphere` if the Report page uses it), narrow centered column, Geist sans, symbol in footer. Content: `404 Not Found` + one muted sentence (e.g., "That Report isn't here."). No back-link, no "try searching," no "upload a new Report" — the IA has nothing to link to (see [[Standard - Information Architecture]]).

The generic-404-for-both posture is load-bearing for two reasons: it closes a growth path ([[Principle - Narrow Forever]]) and it is a mild privacy posture (a Reporter who deleted a Report cannot be retroactively outed by a different page shape for expiration). Both reasons are accepted honestly; the trade-off is that a Reporter who receives a "not found" response cannot tell from the page whether their Report expired naturally or was explicitly deleted. That ambiguity is a feature, not a bug.

### Recommended Defaults (Implementation-Refinable)

Two formats are recommendation-grade — called out honestly rather than mandated, because implementation experience may refine them.

- **Timestamp format:** ISO 8601 UTC. Two acceptable shapes:
  - `2026-04-23 14:32:09 UTC` — space-separated, explicit UTC tag. Readable at-a-glance.
  - `2026-04-23T14:32:09Z` — canonical ISO 8601 with the `Z` suffix. Denser, machine-flavored.
  Rationale: terminal-flavored, matches how Triagers work with log timestamps. **Not** human-relative ("2 days ago") and **not** locale-formatted ("April 23, 2026 at 2:32 PM UTC"); relative and locale formats drift from the receipt feel and introduce locale complexity. Flag for confirmation if implementation experience suggests a different default is clearer.
- **Byte size format:** decimal, human-readable. Examples: `2.3 MB`, `17.8 KB`, `890 bytes`. One decimal place for sub-gigabyte values. Rationale: matches typical CLI byte reporting (the `fabro dump` output probably uses the same shape — keep the CLI-to-page handoff coherent). **Not** raw bytes (`2,345,678 bytes` is terminal-flavored but unreadable at receipt scale) and **not** binary units (`2.3 MiB` is technically accurate but niche; decimal is conventional outside the storage-engineering audience).

Both formats are flagged as first-best-guess; implementers are encouraged to refine based on rendered-page experience. Refinement is a Standard update, not a scope change.

### Focus Order and Semantic HTML

Per [[Standard - Accessibility Policy]], the logical focus and reading order on the page is:

1. Page heading (`<h1>` = "Report Metadata Page" — per [[Standard - Noun Vocabulary]], the canonical page name, even if the visible heading renders as "Report" or is absent).
2. Metadata rows — each `label: value` pair reads as a unit. Use `<dl>` / `<dt>` / `<dd>` or `<div>` with appropriate ARIA labels; Report ID is a `<code>`-wrapped value; Uploaded is a `<time>` element; Size is plain text.
3. Delete affordance (`<button>` or `<a>` if it navigates — `<button>` is preferred per A8).
4. Footer (symbol, possibly a link).

Every interactive element (delete affordance, symbol link, click-to-copy on the Report ID) must be keyboard-operable with visible focus state (`--color-focus` per C2's semantic tokens).

### Anti-Examples (What This Sketch Refuses)

- **Wrong:** Two-column layout with a sidebar rail for navigation or "related Reports." Violates the one-column, narrow-centered-receipt shape; introduces navigation surface the IA refuses.
- **Wrong:** Full-viewport-width layout that fills the screen. Violates the narrow-column commitment; the page is a receipt, not a document.
- **Wrong:** A large, coral-colored "Delete Report" button at the top of the page. Violates the secondary-affordance requirement; the destructive framing lives in the confirmation dialog (see [[Standard - Interaction Patterns (CLI + Delete)]] and [[Standard - Design System (Receipt Page + CLI Output)]]), not on the page.
- **Wrong:** A tab bar across the top of the page ("Overview / Files / Metadata"). Violates both the "no tabs" absence and the single-page IA from [[Standard - Information Architecture]].
- **Wrong:** A "This Report was deleted on 2026-04-20" or "This Report expired on 2026-05-23" distinct-state page. Explicitly rejected by Bryan 2026-04-23; expired and deleted both resolve to generic 404.
- **Wrong:** A retention-policy line on the Report page ("This Report expires in 23 days"). Violates the "no retention messaging on the page" commitment; retention is documented at CLI upload time and in [[Decision - 30-Day Retention Default]].
- **Wrong:** Footer with both the Fabro symbol and the Fabro logotype. Violates the "never both together" rule from [[Standard - Design System (Receipt Page + CLI Output)]].
- **Wrong:** Placing the symbol footer linked to a `fabro.report` homepage. The product has no homepage (see [[Standard - Information Architecture]]); link to [Fabro's marketing site](https://fabro.sh) or Fabro docs instead, or leave the symbol unlinked.
- **Wrong:** Showing only the last 8 characters of the Report ID ("...f2a1b...") with a "show full" toggle. The full 32 characters must be visible; the ID is the whole handle to the Report and truncation undermines copy-and-share.
- **Wrong:** Rendering the timestamp in the Reporter's local timezone ("April 23, 2026 at 2:32 PM EDT"). The default is UTC; local-timezone is locale-complexity this Standard refuses.
- **Wrong:** A "copied!" animation with confetti, a sliding toast, or a party-popper emoji on the click-to-copy affirmation. Subtle mint-accented text is sufficient; ceremony violates [[Principle - Minimal Public Surface Feels Calm]].

### Test

Before shipping the Report Metadata Page's layout, run four checks. First: "Does the rendered page hit the canonical ASCII sketch shape — narrow centered column, three metadata rows, delete affordance below them, symbol footer?" If any element has been rearranged, added, or removed, the shape has drifted; re-derive against this Standard. Second: "Is every absent item in the 'What The Sketch Doesn't Show' list actually absent from the rendered page?" If a navigation rail, a tab bar, or a related-Reports section has appeared, the absence list has been violated — delete the addition. Third: "Does the page's focus order match the reading order (heading → metadata rows → delete → footer), and is every interactive element keyboard-reachable with a visible focus ring?" If not, the page fails [[Standard - Accessibility Policy]]'s keyboard-first requirement. Fourth: "When the Report expires or is deleted, does the URL return a generic 404 rendered in the same design system, not a distinct-state page with extra copy?" If a distinct expired or deleted state has emerged, the state-page commitment has drifted — remove the distinct surface.

## Sources

- `docs/alexandria/sources/receipt-page-visual-direction.md` (full file — authoritative for narrow ~500px centered column, section heading optional, three metadata rows (ID / Uploaded / Size), secondary text-link delete affordance, symbol-only footer, generous vertical padding, ASCII sketch reproduced above, expired-and-deleted-both-404 posture, timestamp and byte-size default formats; confirmed 2026-04-23 by Bryan)
- `docs/alexandria/inventory/card-manifest.md` (C3 entry)

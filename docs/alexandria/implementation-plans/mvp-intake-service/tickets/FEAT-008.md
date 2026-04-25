---
id: FEAT-008
title: "Native confirm() delete flow + WCAG 2.2 AA accessibility audit"
outcome: O-1
tier: must
enabler: false
blocked-by: [FEAT-007]
blocks: [FEAT-014]
cards: [Standard - Accessibility Policy, Standard - Interaction Patterns (CLI + Delete), Component - Report Metadata Page]
---

## Motivation

[[Standard - Interaction Patterns (CLI + Delete)]] requires the delete affordance to be guarded by a client-side confirmation; native `confirm()` is the simplest accessible default. [[Standard - Accessibility Policy]] sets a WCAG 2.2 AA floor for the page that must be validated, not assumed. This ticket is the final O-1 ticket: after it, the Report flow is shippable.

## Description

Add a small client-side script (~10 lines) that intercepts the delete affordance click:

1. On click, call `window.confirm("Delete this report? This cannot be undone.")` (calm copy, no all-caps, no exclamation marks).
2. On confirm, issue a `fetch(window.location, { method: "DELETE" })`.
3. On 204 response, redirect to the same URL via `window.location.reload()` — the next render will be the 404 page from FEAT-005.
4. On error, display an inline error message in the calm tone register (no alert dialog, no console log to the user).

Then run a WCAG 2.2 AA audit on the page using the `/ui` skill or an automated tool (axe-core, Lighthouse). Fix any failures.

Audit checklist (from [[Standard - Accessibility Policy]]):

- Semantic HTML: the delete affordance is a `<button>` or `<a>` (not a `<div>`); the metadata rows are in a semantic structure (`<dl>` or comparable); the page has a single `<h1>`.
- Keyboard navigation: Tab reaches the delete affordance; Enter/Space activates it; focus indicator is visible per `--color-focus`.
- Screen reader: the page has a meaningful `<title>`; the delete affordance has accessible text (visible label is fine; no icon-only button).
- Contrast: re-verify ≥ 4.5:1 body, ≥ 3:1 large text/UI.
- No color-only signaling: the destructive-action color on the delete link is reinforced by the word "Delete."
- `prefers-reduced-motion`: any animation (the focus-ring transition, e.g.) respects this query.

## Context

- [[Standard - Accessibility Policy]] — WCAG 2.2 AA on the Report Metadata Page; semantic HTML; keyboard-first; screen-reader support; accessible delete-confirmation.
- [[Standard - Interaction Patterns (CLI + Delete)]] — client-side confirmation; native `confirm()` is acceptable; custom modal requires focus trap + ESC + focus return + `role="dialog"` + `aria-modal="true"`. Native `confirm()` is recommended for MVP per the planning decision.
- [[Component - Report Metadata Page]] — the page must remain at four content elements after this ticket; the inline-error display is part of the delete-affordance element, not a fifth.
- [[Principle - Minimal Public Surface Feels Calm]] — confirmation copy must be calm ("Delete this report?" not "ARE YOU SURE?!"); error messages must be plain English.
- Anti-patterns to avoid: building a custom modal when native `confirm()` works; alert dialogs on error; `console.log` to surface anything to the user; toast notifications; mouse-only delete; styled `<div>` masquerading as a button; missing `aria-label` on icon-only buttons (don't use icon-only buttons here at all).

## Acceptance Criteria

- [ ] Clicking the delete affordance triggers a native browser `confirm()` with calm copy.
- [ ] On confirm, the client issues `DELETE` to the current URL; on 204, the page reloads and renders the 404 state.
- [ ] On error, an inline error message appears next to the delete affordance in the page's tone register.
- [ ] The script is inlined in the page `<script>` tag (no external JS file).
- [ ] axe-core or Lighthouse accessibility audit reports 0 critical/serious issues for the receipt page.
- [ ] Tab reaches the delete affordance; Enter/Space activates it; focus indicator is visible.
- [ ] Screen reader (VoiceOver or NVDA) announces the delete affordance as a button/link with the word "Delete."
- [ ] `prefers-reduced-motion: reduce` disables any decorative animation.

## Implementation Notes

- Keep the script tiny and inline. No bundler, no transpiler, no NPM dependency.
- The fetch's response status check is the only happy/sad branch — no need for response body handling on success (204 has no body).
- Use the `/ui` skill for the audit step; brief it with [[Standard - Accessibility Policy]] and the rendered page output.
- Defer integration tests for the delete flow to a happy-path Playwright/test-script in a follow-up plan. A manual test in two browsers is the MVP bar.
- Consider whether the confirmation message should reference the Report ID for clarity ("Delete report `1234abcd...`?"). Recommend yes, truncated to first 8 chars, to give the Reporter one last visual confirmation they're deleting what they intended.

# Component - Report Metadata Page

## WHAT: Definition

The **Report Metadata Page** is the one-page landing surface served at a Report's URL — the service's *only* rendered surface for the public web. It shows the Report's ID, its upload timestamp, its bundle size, and a delete button. It shows nothing else: no Debug Bundle contents, no Reporter identity (there is none), no Triager-facing data, no navigation, no second page. It is capability-URL-addressed (anyone in possession of the URL sees the same page, with no auth layer; the delete action on the page is authorized solely by URL possession, per [[Decision - Capability URLs over Auth]]). It inherits its tonal register from [[Principle - Minimal Public Surface Feels Calm]] — *efficient and reassured*, calibrated against `bun` — and it exists at all only because the load-bearing pickup-point thesis of [[Product Thesis - Pickup Point, Not Triage Surface]] allows exactly one thin confirmation surface on the web side of the product. This is that surface.

This card is a **structural** Component card, not a visual one. It describes what the page *is* and what it *does*: its content, its behavior, its access model, its tone inheritance. Visual direction — typography, colors, layout, spacing, page chrome — is a separate concern deferred to the blocked `Standard - Design System` (manifest C2, ABSENT source; blocked on visual direction from the product owner). This card states the structure; the visual card will state the appearance; the two together will specify the page. Building the page without the visual card is not a blocker for the structural contract, but the visual choices remain open.

## WHERE: Ecosystem

- Canonical vocabulary:
  - [[Standard - Noun Vocabulary]] — the authoritative **Report Metadata Page** definition ("the minimal one-page landing page at a Report's URL. Shows Report ID, upload timestamp, bundle size, and a delete button. Does not expose contents.") lives there; this Component card is its structural expansion. The Standard's prohibition against shortening the name (no "Metadata Page," "Receipt Page," "Landing Page," "Status Page," "Dashboard") applies to every card, CLI string, and log line that refers to this Component.
- Entity rendered:
  - [[Component - Report]] — the Report is the entity whose fields this page renders. The page displays three fields (ID, upload timestamp, bundle size) out of the Report's authoritative field list; the other fields exist for internal and Triager use, not for Reporter-facing display.
- Access model:
  - [[Decision - Capability URLs over Auth]] — the page is rendered on any request to the URL that can reach it, regardless of who is requesting. There is no login, no session cookie, no access-check step beyond URL possession. The delete action on the page is authorized by URL possession alone; no second factor exists or will be added.
- Retention:
  - [[Decision - 30-Day Retention Default]] — during the 30-day retention window, the page renders on request. At the retention deadline, the Report and its Debug Bundle are deleted; the URL resolves as not-found; the page is not served for expired Reports (there is no "expired" state rendered — see [[Decision - 30-Day Retention Default]] HOW section, "Following This Decision").
- Tone:
  - [[Principle - Minimal Public Surface Feels Calm]] — the page inherits the same tonal register as the CLI: efficient and reassured, warm / conversational, calibrated against `bun`. F4's "tone consistency with this Principle is non-negotiable" applies to every user-facing string on this page (button label, confirmation copy, error states, field labels).
- IA position:
  - [[Standard - Information Architecture]] — this page is the only rendered surface in the product's IA; A6 captures the one-CLI-one-page-per-Report commitment that refuses home pages, listings, and navigation. The structural content of this page is reproduced from A6's IA shape.
- Scope coherence:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — this page is the *entire* rendered surface of the product. The thesis's "receipt page displays only metadata (Report ID, upload timestamp, bundle size, delete button)" in its Validation Criteria is the exact content this page carries; any field added to the page is a measurable signal of thesis drift.
  - [[Principle - Narrow Forever]] — adding fields, sections, tabs, or navigation to this page is on F6's permanently-out-of-scope list (see F6's "Early-Warning Signals" guidance, mirrored in [[Principle - Reject Like bun.report Drift]]). The page is forever a one-page four-element receipt; growth is a scope change requiring product-owner approval, not an implementation choice.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped Metadata Page enrichment (bundle-contents tab, symbolicated stack-frame render, feedback prompt, team-assignment status) is the single most named drift shape in F3. This Component card inherits the rejection; the page does not grow.
- Roles in the interaction:
  - [[Agent - Reporter]] — may optionally visit the page to confirm upload details or delete the Report. The Reporter's Example 2 in the Agent card (deletion after sharing the URL and changing their mind) is exactly the interaction this page supports.
  - [[Agent - Triager]] — may incidentally click delete if a Reporter shares the URL with them and asks for deletion (see Agent - Triager's Example 2). The Triager's delete action uses URL possession as its authorization, the same way any other visitor's would; there is no Triager-specific capability on this page. Triagers do not *use* this page for triage in any other way.
- UX, accessibility, and mechanism:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the delete confirmation flow on this page: client-side confirmation (`confirm()` or a custom modal), authorization via URL possession, and the register for any confirmation copy. The presence of a delete button is structural (this card); the confirmation flow around it is the Interaction Patterns card's concern.
  - [[Standard - Accessibility Policy]] — accessibility floor for this page (WCAG 2.2 AA, keyboard-first operation, semantic HTML, screen-reader support, delete-confirmation accessibility). Structural content is set here; accessibility constraints are set there.
  - [[System - Debug Bundle Retention, Storage, and Key Scheme]] — the URL shape this page is served at (`fabro.report/r/<32-hex-chars>`) and the S3 lifecycle mechanism that removes the page from service when a Report expires. Policy is in the Decisions cited above; mechanism is in A5.
  - [[Decision - Service Stack]] — the page is delivered as server-side-rendered HTML. A12 commits the stack to SSR-only; this Component card's content structure is rendered by the Rust service, not by a client-side framework.
  - [[System - Service Topology]] — the operational path a page-render request takes: DNS → Fargate task → (metadata JSON read from S3) → SSR HTML response. A14 names the pieces; this Component card names the content they produce.
- Forward references (not yet drafted):
  - `Standard - Design System` (manifest C2) — visual direction for this page: typography, color, layout, spacing, page chrome. **Blocked** on source material from the product owner. This Component card deliberately does not anticipate the visual choices; it states structure, and the visual card will state appearance.

## WHY: Rationale

The Report Metadata Page exists because the pickup-point thesis needs exactly one confirmation surface on the web side — the Reporter closes `fabro dump --upload`, pastes the URL somewhere, and wants to be able to click the URL once to confirm the upload really landed and to delete it if they change their mind. Without the page, the Reporter has only the CLI output and the silence of a shared URL; with the page, they have a one-click confirmation that the handoff succeeded and a one-click undo if they need it. That is the load-bearing job of the page: *confirmation of a handoff, with an exit ramp*. Nothing more.

The reason the page is *only* structural and not full-specified in this card is that the visual register is governed by a separate Design System card (C2), which is blocked on product-owner input. Writing this card without visual direction means the structural contract (four elements, no more; URL-possession auth; expired-Reports return not-found) is locked down and ready for implementation, while the appearance remains a separate, open question. Running ahead on appearance before the product owner provides direction would produce a rendered page that silently became the visual precedent; deferring is the correct posture. The deferral is explicit here so a future agent opening this card does not assume the visual direction is in play just because the structure is.

The separation between [[Component - Report]] and [[Component - Report Metadata Page]] is the separation between the *entity* and its *rendered view*. A Report exists whether or not anyone ever visits its URL; the Report Metadata Page is the page-shape it takes on when someone does. Collapsing them into one card would hide the distinction between "the thing that lives in storage" and "the thing that is served to a browser," and that distinction matters for the capability-URL model (URL-possession authorizes the page render; it does not authorize the underlying Debug Bundle retrieval, which is an AWS-IAM concern orthogonal to the page).

## WHEN: Timeline

New rendered surface, no past. Content and structural contract confirmed 2026-04-22 by the product owner in `docs/alexandria/sources/product-vision.md` (L31 "The URL itself resolves to a minimal one-page metadata receipt showing the Report ID, upload timestamp, and bundle size, plus a delete button. It does **not** expose the bundle contents." — and L100 "Report Metadata Page" in Confirmed Entities). Stable. The *visual* direction remains open pending the product owner's Design System input; that decision unblocks `Standard - Design System` (manifest C2), which is where the appearance will be specified.

## HOW: Specification

### What the Page Shows

The Report Metadata Page renders exactly four elements, in the visual arrangement specified by `Standard - Design System` (blocked) and the interaction specifics specified by `Standard - Interaction Patterns (CLI + Delete)` (blocked). Structurally, the four elements are:

1. **Report ID** — the Report's internal identifier, as carried by [[Component - Report]]. Displayed so the Reporter can recognize the Report they just uploaded, and so a Triager can read it off the page and use it to locate the Debug Bundle in S3. The Report ID is not rendered as a clickable link to anything; it is a label on the page.
2. **Upload timestamp** — the moment of successful upload, per [[Component - Report]]. Displayed in a format the Reporter can read (exact format deferred to the Design System card). Not editable.
3. **Bundle size** — the byte size of the uploaded Debug Bundle, per [[Component - Report]]. Displayed in a human-readable unit (exact format deferred to the Design System card). Not editable.
4. **Delete button** — triggers the delete action authorized by URL possession. Copy on the button and the confirmation flow around it is deferred to [[Standard - Interaction Patterns (CLI + Delete)]], with the tonal register inherited from [[Principle - Minimal Public Surface Feels Calm]] and the accessibility floor from [[Standard - Accessibility Policy]].

No other elements appear on the page. There is no navigation, no footer links beyond what the Design System card may specify for compliance or legal (that is a C2 question), no breadcrumbs, no related-Reports list, no "your other uploads" list.

### What the Page Does Not Show

This section is load-bearing because an agent reading the structural contract alone might otherwise import shapes from adjacent products. The page explicitly **does not**:

- **Show Debug Bundle contents.** Not a file listing, not a log preview, not a rendered stack frame, not any excerpt. The Debug Bundle is never exposed through this service. See [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Reject Like bun.report Drift]].
- **Show Reporter identity.** There is no Reporter identity to show. See [[Decision - Capability URLs over Auth]] and [[Standard - Noun Vocabulary]] (Reporter entry).
- **Show any Triager-facing data.** The page is the *Reporter*'s confirmation surface (with the incidental delete capability available to anyone in URL possession). There is no team dashboard, no "viewed by team" state, no assignment, no triage status. See [[Agent - Triager]].
- **Show a "recent uploads" or "my uploads" view.** There is no concept of past uploads on this service; Reporters have no identity, and URL possession is the only handle to a Report. See [[Decision - Capability URLs over Auth]].
- **Show a form, feedback prompt, or "tell us more" surface.** The optional reporter message at CLI upload time is the entire feedback surface; post-upload feedback is on the permanently-out-of-scope list. See [[Principle - Reject Like bun.report Drift]] and [[Principle - Narrow Forever]].
- **Show a symbolicated stack, rendered logs, or any content-derived information.** The service does not read bundle contents; anything derived from the contents cannot be rendered here. See [[Decision - No Content Dedup]] (on the principle that the service does not inspect bundles) and [[Principle - Reject Like bun.report Drift]].
- **Show navigation or secondary pages.** There are no other pages on this service; the IA is one-URL-per-Report and nothing else.

### Capability-URL Access Model

Access to the page is governed by [[Decision - Capability URLs over Auth]]. In practice:

- Any request to the Report's URL, regardless of its origin, IP, or identity, renders the page (during the retention window, and while the Report is not deleted).
- No session cookie, no login step, no access-check on the render path beyond "can the request reach this URL."
- The delete button submits a delete action that requires no additional authentication; possession of the URL is the authorization. See [[Decision - Capability URLs over Auth]] "Following This Decision" for the load-bearing phrasing.
- There is no "only the original Reporter can delete" concept; the product cannot distinguish the original Reporter from anyone else in URL possession, and this is named as a cost (not a bug) in [[Decision - Capability URLs over Auth]] "What It Costs."
- URL entropy (the property that makes capability URLs non-guessable) is a structural requirement on the URL / key scheme; see [[System - Debug Bundle Retention, Storage, and Key Scheme]] (32 hex characters = 128 bits).

### Expired-Report Behavior

When a Report reaches its 30-day retention deadline or is deleted earlier via the delete button, the Report Metadata Page is *no longer served* for that URL. The URL resolves as not-found; there is no "expired" state, no "this Report has been deleted" surface, no tombstone page. See [[Decision - 30-Day Retention Default]] "Following This Decision."

This is a deliberate shape, not an oversight: a rendered expired / deleted state would be a new user-facing surface (what does it look like? what does its copy say? does it link anywhere?), and every one of those questions is the beginning of a scope expansion [[Principle - Narrow Forever]] rejects. The absence of an expired-state surface is a commitment, and it is captured here so a future agent does not propose "a friendly 404 page for expired Reports" as an improvement.

### Tone Inheritance

Every user-facing string on this page inherits from [[Principle - Minimal Public Surface Feels Calm]]. That includes:

- The labels on the three visible fields (ID, upload timestamp, bundle size).
- The copy on the delete button.
- The copy in the delete confirmation flow (when specified by A7).
- Any state copy that appears during the delete action (loading, success, error).

Tone consistency between the CLI (which carries the same register) and this page is non-negotiable; a page that reads in a different register than the CLI breaks the handoff from terminal to browser. See [[Principle - Minimal Public Surface Feels Calm]] "The Tone Surface Extends Beyond the Happy Path."

### Anti-Examples (What This Page Is Not)

- **Wrong:** Adding a "Contents" tab, an "Inspect bundle" button, or any link that would expose Debug Bundle contents. This is the single most enumerated drift shape in [[Principle - Reject Like bun.report Drift]] and violates [[Product Thesis - Pickup Point, Not Triage Surface]] at its load-bearing seam.
- **Wrong:** Adding a "Fabro version" field, a "Reporter message" field, or any of the other Report-carrying fields to the page. The Reporter-facing visible field set is {ID, upload timestamp, bundle size} by design; other fields exist on the [[Component - Report]] entity for internal and Triager use. (A future product-owner decision could change this, but that is a scope change, not an implementation call.)
- **Wrong:** Building a login or "claim this Report" flow so only the original Reporter can delete. This reintroduces Reporter identity, which is out of scope under [[Decision - Capability URLs over Auth]] and [[Principle - Narrow Forever]].
- **Wrong:** Replacing the delete button with a "request deletion" flow that involves email verification or a team workflow. URL-possession authorizes deletion directly; a verification step is a hybrid-auth shape that [[Decision - Capability URLs over Auth]] rejected as Option C.
- **Wrong:** Serving a branded "expired Report" page or a "this was deleted" tombstone. Expired and deleted Reports return not-found — there is no rendered expired state. See "Expired-Report Behavior" above.
- **Wrong:** Rendering the page in a starker or warmer register than the CLI. Tone is inherited from [[Principle - Minimal Public Surface Feels Calm]]; the page and the CLI match or one of them is drifting.
- **Wrong:** Adding a visual direction ("let's make it dark-mode and monospace") to this card rather than to `Standard - Design System`. Visual direction is C2's concern. This card is deliberately visually silent.

### Deferred to Other Cards

- **Visual direction** — `Standard - Design System` (manifest C2). Typography, color, layout, spacing, page chrome. Blocked on product-owner input.
- **Delete confirmation flow** — [[Standard - Interaction Patterns (CLI + Delete)]]. Client-side confirmation; authorization by URL possession; register inherited from [[Principle - Minimal Public Surface Feels Calm]].
- **Accessibility** — [[Standard - Accessibility Policy]]. WCAG 2.2 AA, keyboard-first operation, semantic HTML, screen-reader behavior, focus handling on the delete confirmation.
- **URL shape** — [[System - Debug Bundle Retention, Storage, and Key Scheme]]. `fabro.report/r/<32-hex-chars>`, 128 bits of entropy, served from a per-Report S3 prefix.

The structural contract in this card is sufficient for a Conan review of the page's shape. Implementation requires the deferred cards to resolve first, or interim decisions from the product owner.

## Sources

- `docs/alexandria/sources/product-vision.md` (L31 receipt page description — "a minimal one-page metadata receipt showing the Report ID, upload timestamp, and bundle size, plus a delete button. It does **not** expose the bundle contents."; L66–L72 Security Model — capability-URL access; L100 Confirmed Entities → Report Metadata Page)
- `docs/alexandria/alexandria-config.json` (`discovery.confirmed_entities` → Report Metadata Page)
- `docs/alexandria/inventory/card-manifest.md` (A4 entry; visual direction deferral to C2 noted explicitly in the manifest's Source Gaps section)

# Principle - Reject "Like bun.report" Drift

## WHAT: The Principle

Reject the specific drift pattern that occurs when a Factory-mode agent is told `fabro.report` is "like `bun.report`" and silently imports `bun.report`'s downstream feature shape — symbolication, Sentry integration, in-browser rendering of crash data, markdown rendering, feedback collection, and team dashboards — as natural extensions of the Report Metadata Page, which exists only to confirm an upload. The failure mode is cumulative and cheap to trigger: each individual feature sounds reasonable in isolation ("make the receipt page more useful"), but together they convert `fabro.report` from a pickup point into a triage surface, which is the single identity decision the product has rejected. This principle names the drift so it can be caught early, and pairs with [[Product Thesis - Pickup Point, Not Triage Surface]] as the explicit negative frame around that thesis.

## WHERE: Ecosystem

- Paired with:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — this Principle is the negative frame around that thesis. Whenever the thesis says "pickup point, not triage surface," this card enumerates the specific triage-surface features that must be rejected.
  - [[Principle - Narrow Forever]] — the roadmap commitment this Principle defends. Each drift example below is, by that commitment, permanently out of scope — not deferred, not v2. Narrow Forever is the upstream commitment; this card is the concrete drift list it rules out.
- Serves:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — each drift example below trades team-retrieval investment against richer-Reporter-experience investment, which the Strategy explicitly rejects. Rejecting these drifts is how this card enforces the Strategy.
- Referenced by:
  - [[Standard - Noun Vocabulary]] — drift often enters through imported vocabulary ("Crash Report," "Stack Trace," "Feedback Form") borrowed from `bun.report`; the vocabulary standard flags those imports as a symptom.
  - [[Principle - Minimal Public Surface Feels Calm]] — tone drift is a vector for `bun.report` drift. `bun.report`'s chattier web-product voice ("We've received your crash report!") is a tonal shape this Principle rejects on the same grounds it rejects feature-shape drift; tone drift is the earliest symptom of the cumulative failure mode this card names.
  - [[Decision - Who This Service Is For]] — audience-layer drift ("let's also serve hosted-Fabro users") is the audience version of the feature-shape drift this Principle rejects; the Decision closes that escape hatch.
  - [[Decision - Capability URLs over Auth]] — per-team auth, user accounts, and team dashboards are drift shapes this Principle enumerates; the Decision is the upstream access-model commitment those rejections appeal to.
  - [[Decision - No Content Dedup]] — dedup proposals disguised as operational improvements are a drift pattern this Principle rejects; the Decision is the upstream commitment.
  - [[Decision - 30-Day Retention Default]] — retention extension (per-Report overrides, indefinite retention tiers) is a drift shape this Principle names; the Decision locks the retention policy against that drift.
  - [[Agent - Reporter]] — post-upload Reporter-facing enrichment (bundle-contents view, symbolicated stack, markdown-rendered logs, feedback prompts) is the paradigmatic drift shape; the Reporter's narrow post-upload surface as defined there is the role-layer expression of this Principle's rejection list.
  - [[Agent - Triager]] — Triager-facing UI (team dashboards, listing endpoints, per-team scoping, in-service triage surfaces) is the drift shape this Principle enumerates most concretely; the Triager's no-service-surface shape as defined there is the role-layer commitment those rejections rely on.
- Template being misread:
  - `bun.report` at `~/p/oven-sh/bun.report` — the product this drift borrows from. Only the intake URL shape is borrowed. Every feature listed in the Drift Examples section below exists in `bun.report` and must not be copied into `fabro.report`.
- Contrast with:
  - Parent product `~/p/fabro-sh/fabro` — if a feature the agent wants to build looks like triage tooling, it probably belongs in Fabro's own tooling, not in `fabro.report`.

## WHY: Belief

Low-novelty products are vulnerable to template drift because the shortest path to a design decision is pattern-matching against a familiar analogue. `bun.report` is the designated template, and it is a substantially larger product than `fabro.report` by deliberate choice. An agent told "like `bun.report`" has no mechanical way to distinguish "borrow the intake URL shape" from "borrow the full product shape" unless the distinction is stated explicitly, repeatedly, and in a form the agent will actually read before proposing features.

The cost of letting drift happen once is small — a single feature of the wrong shape. The cost of letting it happen repeatedly is that the product turns into a second-rate `bun.report` clone while pretending to be something narrower, and the narrowness commitment stops meaning anything because it has been renegotiated, one feature at a time, by agents acting in good faith. This principle prevents that by making the drift examples concrete enough that an agent reading only this card can recognize a drift proposal coming out of its own reasoning and stop it before it ships.

## WHEN: Timeline

Established 2026-04-22 alongside [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Standard - Noun Vocabulary]]. Load-bearing from day one — this card exists because the product is greenfield and every design decision that follows will encounter the `bun.report` template; the Principle must be in place before product-layer cards are drafted so each draft can check itself against the drift list. Stable; additions to the Drift Examples list are expected as new drift shapes are noticed, but the framing ("pickup point, not triage surface") does not change.

## HOW: Application

### Drift Examples (What This Principle Rejects)

Each example below is a real drift risk implied by the `bun.report` template. All of them are permanently out of scope for `fabro.report`.

- **If asked to make the Report Metadata Page more useful**: do NOT render Debug Bundle contents, file listings, or any inspection view. The Report Metadata Page shows Report ID, upload timestamp, bundle size, and a delete button. "More useful" is not a justification to expand that surface; if it were, the Product Thesis would say so. See [[Product Thesis - Pickup Point, Not Triage Surface]].
- **If asked to symbolicate crash data**: do NOT add a symbolicator, server-side crash parser, or stack-frame resolver. Symbolication is a triage concern performed by Triagers with their own tooling. `bun.report` symbolicates because its downstream audience reads crash data in-browser; `fabro.report`'s audience retrieves bundles via S3 and inspects them elsewhere.
- **If asked to integrate with Sentry (or any crash-reporting SaaS)**: do NOT add third-party integrations that expect structured crash payloads. `bun.report` has Sentry integration because it is a crash-report product; `fabro.report` is a Debug Bundle intake, and Debug Bundles are opaque tarballs to this service.
- **If asked to render logs or markdown from inside the bundle**: do NOT. The service never exposes Debug Bundle contents. Not via the Report Metadata Page, not via a secondary endpoint, not via a "convenience" server-side render.
- **If asked to collect feedback from Reporters after upload**: do NOT add a post-upload feedback form, a follow-up email, or a "tell us what happened" flow on the Report Metadata Page. An optional CLI-flag message at upload time is the entire feedback surface. Turning pickup into conversation expands scope.
- **If asked to build a team-facing dashboard (recent Reports, filtering by Fabro version, etc.)**: do NOT. There is no team-facing surface on this service. Triagers retrieve bundles directly from S3 using their own credentials. A dashboard turns `fabro.report` into a triage UI and violates the Product Thesis.
- **If asked to add per-team scoping, user accounts, or authentication**: do NOT. `fabro.report` uses capability URLs. There is no account concept, no team concept, no auth concept on the public surface. Adding any of those is a scope change requiring explicit product-owner approval, not an implementation decision.
- **If asked to track Reporter identity across uploads**: do NOT. Reporters have no identity by design. See [[Standard - Noun Vocabulary]].
- **If asked to extend retention policy per Report (custom expiration windows, indefinite retention tiers)**: do NOT without escalation. 30-day default is a product decision. "Give Triagers more time" is a Triager-side problem solvable by retrieving the bundle within the window; it is not an intake-service feature.

### Early-Warning Signals (How to Recognize Drift in Progress)

- Vocabulary from `bun.report` starts appearing in CLI strings, cards, or code: "Crash Report," "Stack Trace," "Symbolicator," "Feedback Form," "Team," "Organization," "User."
- A card proposal adds endpoints to the service that a Triager — not a Reporter — would consume.
- A design discussion framed as "well, `bun.report` does it this way" rather than as "what does the Product Thesis say?"
- A feature request routed to `fabro.report` that, rewritten, is really a request to Fabro's own internal tooling or to a separate triage tool.
- The Report Metadata Page is gaining fields, sections, tabs, or navigation. Any of those is a signal; all three combined is confirmation of drift.

### What This Principle Does Not Forbid

- Better error messages during upload, clearer failure modes, a crisper URL format, friendlier CLI copy. These help the Reporter hand off a bundle and are in scope.
- Operational improvements invisible to Reporters: retention enforcement, S3 key layout, deletion semantics, metrics for the Fabro team's own monitoring. These are `System - Retention, Storage, and Key Scheme` (manifest A5, not yet drafted) concerns, not surface-expansion.
- Adding metadata fields captured at upload time (arbitrary key/value tags already exist for this). This is explicitly allowed by the product decisions and does not expand the public surface.
- Improvements to the `fabro debug` command in the parent Fabro repo. Bundle structure is owned there; that work is not drift and not in this library's scope.

### Test

Before building or proposing any feature for `fabro.report`, ask: "Does this feature exist in `bun.report`?" If yes, ask: "Is it in the intake URL shape (upload, return URL, share URL)?" If it is intake-shape, it may be in scope — check the Product Thesis. If it is anything else — rendering, symbolication, feedback, triage, team scope, identity, inspection, analysis — it is out of scope. Permanently. Do not proceed. Route the request back to the product owner as a scope question, not as an implementation question.

A second test: "Who is the audience for this feature?" If the answer is "Triagers," the feature belongs outside this service. Triagers retrieve from S3; they have no surface here. If the answer is "Reporters, during upload or URL handoff," proceed against the Product Thesis. If the answer is "Reporters, after upload, to give them something richer to look at," stop — that is the most common drift shape.

## Sources

- `docs/alexandria/sources/product-vision.md` (L33–48 "What fabro.report Is Not"; L102–107 "Template Relationship"; L48 cascade-risk rationale)
- `docs/alexandria/inventory/card-manifest.md` (F3 entry — originally manifested as "Anti-Pattern," renamed to Principle 2026-04-22 per product owner for linter taxonomy alignment)
- Template reference: `~/p/oven-sh/bun.report` (read-only; the product being drifted toward)

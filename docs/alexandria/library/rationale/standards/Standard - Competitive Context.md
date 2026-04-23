# Standard - Competitive Context

## WHAT: Definition

`fabro.report` has exactly one template worth naming: **`bun.report`, borrowed for its intake URL shape only**. No other competitor, analogue, or adjacent product warrants deeper analysis at this product's scale. This Standard is deliberately a one-pager — the competitive landscape for a narrow-forever Debug Bundle pickup point does not earn weight proportional to the Core-tier label its area (1.4 Competitive Analysis) wears in the Factory × Low × Low configuration. The purpose of this card is to *close* the competitive-analysis question for this library, not to open it: `bun.report` is the only template that matters, and the adjacent categories below are named explicitly so a future agent does not re-open the survey they appear to invite.

## WHERE: Ecosystem

- Template:
  - `bun.report` at `~/p/oven-sh/bun.report` — the only product whose shape is borrowed. What is borrowed is the **intake URL shape** (upload a tarball, receive a short secret URL, share it out-of-band) and nothing else.
- Strategic anchor:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the thesis is what makes the borrowing stop at intake. Everything downstream of the URL belongs to the triage-product category that this thesis refuses.
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the strategic reason the borrowing stops where it does: the product is optimizing for team retrieval from self-hosted Fabro users, which is a narrower target than the population `bun.report` serves.
- Paired commitments:
  - [[Principle - Reject Like bun.report Drift]] — the negative-frame card that enumerates the specific `bun.report` features this Standard names as not-borrowed. F3 is the drift list; this Standard is the upstream "we only borrow the URL shape" commitment F3 defends.
  - [[Principle - Narrow Forever]] — anything beyond intake is on F6's permanently-out-of-scope list. Re-surveying competitors "to see if there are features worth adding" is itself a drift-shaped activity under this Principle.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the not-borrowed list in this Standard uses the canonical nouns (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) to name exactly what `fabro.report` will not import from `bun.report` or adjacent categories. Vocabulary drift into `bun.report`'s nouns ("Crash Report," "Stack Trace," "Feedback Form") is one of the earliest drift symptoms F2 calls out; this Standard inherits that vocabulary commitment.
- Audience commitment:
  - [[Decision - Who This Service Is For]] — the audience is self-hosted Fabro users, primary and only. This scopes the competitive-analysis question: comparisons to products serving different populations (managed offerings, crash-report SaaS, file-sharing services) are structurally unhelpful because their optimization targets differ.

## WHY: Rationale

Low-novelty products tempt agents to pattern-match against every adjacent category and import features under the framing of "well, X does Y." For a product with two entry points (CLI upload and a one-page metadata receipt) and one exit point (S3 retrieval by the team), the set of adjacent products that *could* be surveyed is large: paste services (GitHub gist, Pastebin, etc.), support-bundle uploaders (Vercel and Fly support-upload flows, vendor crash-report endpoints), file-sharing with expiration (WeTransfer and similar), crash-report products (Sentry, `bun.report`). Each shares some of the DNA ("upload + secret URL + expiration"), and none of them warrant deeper study, because the product shape is already decided and nothing in any of those adjacent categories would change the decision.

Surveying them would be effort in the wrong direction. The risk is not that the survey is empty — the risk is that the survey finds "interesting" features in adjacent categories and smuggles them in as "we should do this too, since X does." That is exactly the drift [[Principle - Reject Like bun.report Drift]] names. Closing the competitive-analysis question here, with an explicit one-pager, prevents the re-opening.

## WHEN: Timeline

Written 2026-04-22 per Conan's inventory recommendation (Open Flag 3 in `docs/alexandria/inventory/card-manifest.md`): skip the paste-services / support-bundle-uploaders survey; close C1 as "`bun.report` is the only template that matters at this scale." Stable. Revisiting this Standard would mean reopening scope or audience, which are governed by [[Principle - Narrow Forever]] and [[Decision - Who This Service Is For]] — not implementation-level concerns.

## HOW: Specification

### What Is Borrowed From `bun.report`

- **The intake URL shape.** Upload a tarball, receive a short secret URL, share it out-of-band. That is the entire borrowed surface.

### What Is Not Borrowed From `bun.report`

Each item below exists in `bun.report` and is **not** borrowed into `fabro.report`. This list mirrors and anchors [[Principle - Reject Like bun.report Drift]]'s drift examples at the competitive-analysis layer.

- **Symbolication** of crash reports or stack frames — `fabro.report` treats the Debug Bundle as an opaque blob.
- **Sentry integration** or any crash-reporting SaaS integration — `fabro.report` has no third-party integrations that expect structured crash payloads.
- **In-browser rendering** of bundle contents — the Debug Bundle is never exposed through the service.
- **Markdown rendering** of structured crash data — nothing from inside the bundle is rendered, ever.
- **Feedback collection** workflows — the optional CLI-flag message at upload time is the entire feedback surface.
- **Team dashboards** or in-service triage UIs — Triagers use direct S3 retrieval with their own AWS credentials; the service has no team-facing surface.
- **Per-user accounts** or per-team scoping — capability URLs are the whole access story (see [[Decision - Capability URLs over Auth]]).

### Adjacent Categories (Named To Close, Not To Survey)

Each of the following product categories shares some DNA with `fabro.report` ("upload + secret URL + expiration") and is *intentionally not surveyed further*. They are listed here so a future agent does not treat their absence as an oversight. The product shape is already decided; adding any feature sourced from these categories would be drift, not derivation.

- **Paste services** (GitHub gist, Pastebin, etc.). Share: short URL for content. Not borrowed: rendering content in-browser, syntax highlighting, revision tracking, fork / star surface area.
- **Support-bundle uploaders** (Vercel and Fly support-upload flows, vendor crash-report endpoints, SaaS support-ticket uploaders). Share: user uploads a diagnostic payload so a support team can retrieve it. Not borrowed: ticketing integration, response-time SLAs, customer-identity tracking, support-case threading.
- **File-sharing with expiration** (WeTransfer and similar). Share: upload, receive a URL, time-bounded availability. Not borrowed: download tracking, recipient notifications, large-file optimizations beyond opaque-blob storage, branded landing pages.
- **Crash-report products** (Sentry, `bun.report` itself at its fuller shape). Share: users submit crash data; a team investigates. Not borrowed: everything downstream of intake — symbolication, aggregation, alerting, dashboards, integrations.

The common property across all four categories is that **intake is the borrowed layer and everything past intake is the refused layer**. If a proposal arrives sourced from any of these categories, the response is the same one [[Principle - Reject Like bun.report Drift]] gives to a `bun.report`-sourced proposal: intake-shape features may be in scope subject to the Product Thesis; anything downstream of intake is not.

### Test

Before citing a competitor or adjacent product as justification for a feature, ask: "Is the feature in the **intake URL shape** (upload, return URL, share URL, optional delete, expiration)?" If yes, check [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Narrow Forever]]. If the feature is **anything downstream of intake** — rendering, analysis, symbolication, feedback collection, team dashboards, notifications, SLAs, identity, per-team scoping — the competitor citation is not a scope argument; it is a drift argument, and this Standard rejects it.

## Sources

- `docs/alexandria/sources/product-vision.md` (L33–L48 "What `fabro.report` Is Not"; L102–L107 "Template Relationship: `bun.report`"; L12–L14 Vision Summary — intake-only framing)
- `docs/alexandria/inventory/card-manifest.md` (C1 entry; Open Flag 3 — Conan's recommendation to close C1 as a one-pager rather than conduct an external survey)

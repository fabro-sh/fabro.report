# Component - Report

## WHAT: Definition

A **Report** is the upload event and its public identity on `fabro.report`. It is created when a Reporter runs `fabro report` and the CLI successfully uploads a Debug Bundle to S3; it exists for a fixed 30-day retention window from that creation moment; it can be deleted at any time during the window by anyone in possession of the Report's URL; and at the retention deadline it auto-deletes, its Debug Bundle is removed from S3, and the URL resolves as not-found. The Report carries a fixed set of authoritative fields — ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata tags, retention deadline — and is otherwise immutable: there is no modification operation after creation, only deletion. The [[Standard - Noun Vocabulary]] entry for Report is the authoritative definition; this Component card expands that entry with structural detail, lifecycle, and key couplings, but inherits the exact field list and naming from it.

## WHERE: Ecosystem

- Canonical vocabulary:
  - [[Standard - Noun Vocabulary]] — the authoritative Report definition (fields, relationship to Debug Bundle, distinction from Reporter) lives there; this Component card is its structural expansion. Any drift between this card's field list and the Standard's field list is a defect in one of them — the Standard is authoritative.
- Access model:
  - [[Decision - Capability URLs over Auth]] — the Report's URL is the capability token for view + delete; possession of the URL is the whole access-control mechanism. The Report carries no ACL, no owner, and no identity-bound claim; the URL is the authorization.
- Retention:
  - [[Decision - 30-Day Retention Default]] — the Report's retention deadline is exactly 30 days from upload, immutable per-Report, product-owner-adjustable only as a policy. This Component card inherits that deadline as a structural field; the *mechanism* that enforces the deadline (S3 lifecycle rule vs. application-side cron vs. other) is deferred to the forward reference below.
- Identity semantics:
  - [[Decision - No Content Dedup]] — a Report is a per-upload identity, not a per-content identity. Two Reports with byte-identical Debug Bundles are two separate Reports with separate IDs, URLs, retention deadlines, and storage objects. The Report does not carry a content hash, a canonical-bundle reference, or any content-identity metadata.
- Relationships to other entities:
  - [[Standard - Noun Vocabulary]] — the Report contains a reference to a Debug Bundle (its opaque-tarball payload in S3); Report and Debug Bundle are distinct nouns and must not be conflated. The Reporter is whoever ran `fabro report` to create the Report; the Reporter has no identity on the service and is represented only by the Fabro version field captured at upload time.
  - [[Agent - Reporter]] — creates the Report by running `fabro report`; receives the Report's URL on successful upload; may optionally attach a message via CLI flag and arbitrary key/value metadata tags. The Report is the thing the Reporter hands off.
  - [[Agent - Triager]] — retrieves the Debug Bundle referenced by the Report, using the Report ID extracted from the Report's URL as the key into the S3 bucket. The Triager never interacts with the Report through the service; the Report ID is the structural contract between Report and Triager retrieval.
- IA position:
  - [[Standard - Information Architecture]] — the Report is the only content entity on this service; its URL is the only routable address in the product's IA. A6 captures the IA shape this Component occupies: one Report per URL, no listing, no per-user view, no containment.
- Rendered surface:
  - [[Component - Report Metadata Page]] — the single rendered view of a Report on the service. The Report Metadata Page shows the Report's ID, upload timestamp, bundle size, and a delete button; it exists at the Report's URL; there is no other Report-rendering surface on this product.
- Scope commitments:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the Report is the pickup receipt. Its fields are calibrated to "confirm a Debug Bundle arrived and let the Reporter delete it if they change their mind" and nothing more; Report-level fields that serve a triage workflow (investigation state, assignee, tags pinned to team workflow stages) are out of scope.
  - [[Principle - Narrow Forever]] — adding fields to the Report is a scope question, not an implementation question. The field list in [[Standard - Noun Vocabulary]] is the complete field list; extending it requires updating that Standard (and its backing source) first.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped Report enrichment (content-derived fields, symbolication status, feedback state, team assignment) is named drift; this Component card inherits the rejection.
- Forward references (planned in `docs/alexandria/inventory/card-manifest.md`, not yet drafted):
  - `System - Retention, Storage, and Key Scheme` (manifest A5) — the mechanism that enforces the Report's retention deadline, the S3 key layout that maps a Report ID to its Debug Bundle object, and the entropy properties of the Report ID that make capability URLs non-guessable. Policy lives in [[Decision - 30-Day Retention Default]] and [[Decision - Capability URLs over Auth]]; mechanism lives there.
  - `Standard - Interaction Patterns (CLI + Delete)` (manifest A7) — the CLI UX that creates the Report (`fabro report`, flags, progress, error surface) and the delete confirmation flow on the Report Metadata Page that ends the Report's life before retention.

## WHY: Rationale

The Report has a card of its own because it is the product's single first-class entity — the thing that is created, addressed, possibly deleted, and eventually expired. Every other structural card in this library either describes the Report (this card), describes how it is rendered (the Report Metadata Page), describes who creates or retrieves it (the Reporter, the Triager), describes the policy around it (the Decisions), or describes the mechanism that enforces the policies (the forthcoming A5 System). Giving the Report a dedicated Component card surfaces its field list and lifecycle in one place so downstream implementation has a single structural reference; keeping the Component card thin — authoritative vocabulary in [[Standard - Noun Vocabulary]], policy in the Decisions, mechanism in A5 — keeps the Report card from becoming a catch-all spec that silently accumulates scope.

The load-bearing coupling is *the Report is not the Debug Bundle*. A Report is a handle (ID, URL, metadata) on a Debug Bundle (the tarball in S3); the two are separate entities with separate lifecycles on paper, even though their storage lifecycles are coupled in practice (a Report's deletion or expiration deletes the bundle). This distinction is what lets Reporters and Triagers reason about "the Report" (public identity, URL, delete button) without that phrasing implying the service exposes bundle contents. The vocabulary separation is [[Standard - Noun Vocabulary]]'s; this Component card just preserves it.

## WHEN: Timeline

New entity, no past. Fields confirmed by the product owner on 2026-04-22 (`docs/alexandria/sources/product-vision.md` L96 "Confirmed Entities → Report" and L82–L88 "Metadata captured at upload"). Lifecycle inherited from [[Decision - 30-Day Retention Default]] (retention) and [[Decision - Capability URLs over Auth]] (access model), both confirmed the same day. Stable. The Report's field list is closed under [[Standard - Noun Vocabulary]]; additions to it are vocabulary updates that go through that Standard first, not this Component card.

## HOW: Specification

### Authoritative Fields

Each field below appears in [[Standard - Noun Vocabulary]]'s Report entry and is reproduced here with its structural role. The Standard is authoritative for naming and short definitions; this card names the structural role each field plays.

- **ID** — the Report's internal identifier. Used as the S3 key (or the basis of the S3 key — mechanism deferred to `System - Retention, Storage, and Key Scheme`) for the Debug Bundle. Must have sufficient entropy that URLs constructed from it are not brute-forceable, per the security requirement in [[Decision - Capability URLs over Auth]].
- **URL** — the public short URL for the Report. Derived from the Report ID. Possession of the URL is the capability for view + delete on the Report Metadata Page. The exact URL shape — host, path, slug layout — is a forward reference for `Standard - Interaction Patterns (CLI + Delete)` and `System - Retention, Storage, and Key Scheme`; this card does not name a path shape.
- **Upload timestamp** — the moment of successful upload, measured by the service. The retention deadline is computed from this timestamp. Displayed on the [[Component - Report Metadata Page]] as one of the three pieces of visible metadata.
- **Bundle size** — the byte size of the uploaded Debug Bundle. The one property of the Debug Bundle this service observes; [[Decision - No Content Dedup]] rejects any *content* observation beyond this. Displayed on the [[Component - Report Metadata Page]] as one of the three pieces of visible metadata.
- **Optional reporter message** — a short free-form string provided at upload via CLI flag (source uses `-m "short message"`). Present when the Reporter supplied one; absent otherwise. This is the **entire** feedback surface for the product under [[Principle - Reject Like bun.report Drift]]; there is no post-upload feedback path.
- **Fabro version** — the version of the Fabro binary that produced the Debug Bundle, captured as metadata at upload time outside the tarball. Per [[Standard - Noun Vocabulary]], this is the *only* property that identifies a Reporter; the Reporter has no service-side identity.
- **Arbitrary key/value metadata tags** — free-form tags supplied at upload time (source L85: "arbitrary key/value metadata tags (no first-class 'issue link' concept — that's just one possible tag)"). These are not a schema; the service does not interpret tag contents. Team workflow hints (linked issue, triage bucket) are represented here when Reporters choose to include them, without earning first-class status.
- **Retention deadline** — upload timestamp plus 30 days, per [[Decision - 30-Day Retention Default]]. Reporter-immutable; product-owner-adjustable only as a policy, not per-Report. At the deadline, the Report and its Debug Bundle are deleted; the URL resolves as not-found.

### Lifecycle

A Report has exactly four lifecycle events, and no other states.

1. **Created at successful upload.** `fabro report` uploads a Debug Bundle to S3; the service mints a Report ID, writes the S3 storage object for the Debug Bundle, records the Report's fields (including the retention deadline), and returns the URL to the CLI. Creation is atomic with successful upload — partial uploads do not produce Reports.
2. **Possibly visited (and re-visited) via the Report Metadata Page.** Any request to the URL during the retention window renders the [[Component - Report Metadata Page]] with the Report's visible fields (ID, upload timestamp, bundle size) and a delete button. Visits do not modify the Report; there is no "viewed" state.
3. **Deletable at any time during retention, via URL possession.** The delete button on the Report Metadata Page submits a delete action authorized solely by possession of the URL. Deletion removes the Report from the service's active set and deletes the Debug Bundle from S3. Deletion is terminal — there is no undelete, no archive, no "recently deleted" view.
4. **Auto-deletes at retention deadline.** At the 30-day boundary, the Report is removed from the active set and its Debug Bundle is deleted from S3. The URL resolves as not-found; there is no rendered expired state. The mechanism that runs this deletion is deferred to `System - Retention, Storage, and Key Scheme` (manifest A5).

No modification after creation is possible. There is no edit operation for any Report field.

### Key Couplings (Load-Bearing)

- **A Report contains a reference to a Debug Bundle; they are separate entities.** This is the most important coupling the card expresses. A Report is public (URL, metadata receipt, delete button); a Debug Bundle is never public (S3-only, Triager-credentialed retrieval). See [[Standard - Noun Vocabulary]] for the distinction; see [[Product Thesis - Pickup Point, Not Triage Surface]] for why the service never collapses the two.
- **The Report's URL is the capability token.** Not the ID in isolation — the URL, in the Reporter's and Triager's possession. View and delete both flow from URL possession. See [[Decision - Capability URLs over Auth]]. (The Report ID *alone* is what a Triager uses as an S3 key for retrieval; that path is orthogonal to the capability URL and is governed by AWS IAM, not by this service.)
- **The Report's retention deadline is 30 days from creation, not adjustable.** See [[Decision - 30-Day Retention Default]]. The deadline is a structural field, not a configurable one; `fabro report --retain 90d` is a violation of that Decision and of [[Principle - Narrow Forever]].

### Anti-Examples (What a Report Is Not)

- **Wrong:** Treating the Report as the owner of the Debug Bundle's contents for any service-side operation. The service does not read, render, parse, or analyze Debug Bundle contents; the Report references the bundle, it does not embed it. See [[Decision - No Content Dedup]] and [[Principle - Reject Like bun.report Drift]].
- **Wrong:** Adding a mutable state field (investigation status, triage assignee, viewed-by flag, pinned flag). The Report has no mutable state after creation except deletion. Any proposed mutable field is a scope expansion that must go through [[Principle - Narrow Forever]] and the product owner.
- **Wrong:** Adding an ACL, owner ID, or "shared with" list. The access model is URL possession; introducing any identity-bound access concept violates [[Decision - Capability URLs over Auth]].
- **Wrong:** Adding a content hash or canonical-bundle pointer. This is the first half of dedup surface area and is rejected by [[Decision - No Content Dedup]].
- **Wrong:** Displaying a Report field on the Report Metadata Page that is not one of {ID, upload timestamp, bundle size, delete button}. Per [[Component - Report Metadata Page]], the page shows only those four elements; adding fields to the page is drift into triage-surface shape. (The other Report fields exist for internal and Triager use, not for Reporter-facing display.)

### Test

Before describing a Report in a downstream card or CLI string, check two things. First, does every field named appear in [[Standard - Noun Vocabulary]]'s Report entry? If not, that is synonym drift or field invention and must be reconciled with the Standard first. Second, does the description assume any Report capability beyond "create via upload, view receipt, delete via URL, auto-expire at 30 days"? If yes, that is a scope expansion that must clear [[Principle - Narrow Forever]] and the relevant Decision; it is not a Component-level implementation choice.

## Sources

- `docs/alexandria/sources/product-vision.md` (L26–L31 end-to-end flow — upload/URL/retrieval sequence; L82–L88 Confirmed Product Decisions → metadata captured at upload; L96 "Confirmed Entities → Report"; L66–L72 Security Model — URL as capability)
- `docs/alexandria/alexandria-config.json` (`discovery.confirmed_entities` → Report)
- `docs/alexandria/inventory/card-manifest.md` (A3 entry)

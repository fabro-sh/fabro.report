# Standard - Information Architecture

## WHAT: Definition

The **Information Architecture** of `fabro.report` is minimal by commitment. There is no sitemap, no navigation, no second page, and no hierarchy. The entire public IA is: **a small, fixed set of Fabro CLI commands (`fabro dump`, `fabro dump --upload`, `fabro upload`) and one rendered page (the Report Metadata Page) per Report, addressed by the Report's URL**. The service's endpoints — upload, render, delete, and whatever HTTP shape those resolve to — exist solely to make that CLI / one-page-per-Report contract work. This Standard is captured here specifically so it *stays* minimal: an IA that is not written down invites future agents to invent navigation, hierarchy, browse endpoints, or account surfaces; an IA that is written down as "the dump / upload CLI, one page per Report, nothing else" makes those inventions visible as scope changes rather than implementation decisions.

The load-bearing phrasing is *the IA is not hierarchical*. There is no root page, no index, no "all Reports" surface, no per-user view, no breadcrumbs. Every Report is reachable only by its URL, and URLs do not compose into a structure the Reporter or Triager navigates through. An agent reading this Standard should not come away expecting a home page, a listing page, or a two-level site structure — none exist.

## WHERE: Ecosystem

- Thesis anchor:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the thesis requires this minimal IA shape. A pickup point does not have a sitemap; it has a drop-off motion and a one-page confirmation of it. Any IA decision that adds a page, an index, or a navigational layer is evidence the thesis has been abandoned at the IA layer.
- Scope commitments:
  - [[Principle - Narrow Forever]] — extra IA (home pages, listings, per-user views, team dashboards, browse endpoints) is on F6's permanently-out-of-scope list. The IA is not "minimal for v1"; it is minimal forever, because the product is narrow forever.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped IA (a home page that explains the product, a dashboard that lists recent Reports, a per-team route) is named drift in F3. This Standard is the upstream IA commitment those rejections appeal to.
- Access model:
  - [[Decision - Capability URLs over Auth]] — URL possession is the whole access structure. That is the IA too: the set of routable surfaces is "URLs a Reporter is currently in possession of," and there is no listing, no browse, and no recovery path that would require a non-capability IA. The IA and the access model are coextensive.
- Entities the IA serves:
  - [[Component - Report]] — the Report is the only content entity on this service. The Report's URL is the only routable address. There is one rendered view of a Report (the Report Metadata Page) and no other; the IA has no cause to add further routes.
  - [[Component - Report Metadata Page]] — the one page in the IA. Its structural content (ID, upload timestamp, bundle size, delete button) is fixed by that card and is not an IA-layer decision.
- Roles that traverse the IA:
  - [[Agent - Reporter]] — the Reporter's IA traversal is: run `fabro dump --upload` (or `fabro upload`), optionally visit the Report's URL once or twice. They do not browse; they do not navigate. There is nothing to navigate.
  - [[Agent - Triager]] — the Triager does not traverse the IA at all. They see a URL in an out-of-band channel, extract the Report ID, and retrieve the Debug Bundle from S3 with their own AWS credentials. The service's IA is not part of the Triager's workflow, by design. See [[Agent - Triager]] for the structural commitment.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the IA uses canonical names exactly. The page is the **Report Metadata Page**, not the "receipt page," "landing page," "status page," or "dashboard." The URL addresses a **Report**. The CLI subcommand uploads a **Debug Bundle**.
- Mechanism and UX cards:
  - [[System - Retention, Storage, and Key Scheme]] — the concrete URL path shape (host, path layout, slug format) and the S3 key layout for Debug Bundles. This Standard names the *endpoints that must exist* and the *IA shape* they serve; the *path strings* and *key strings* are A5's territory.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the CLI UX for `fabro dump`, `fabro dump --upload`, and `fabro upload` (progress, error surface, exact flags beyond `-m` / metadata tags / `--dry-run`, success output shape) and the delete confirmation flow on the Report Metadata Page. The CLI surface in this Standard names the canonical commands and the flags confirmed in source; the detailed UX is A7's territory.
- Forward references (not yet drafted):
  - `Standard - Design System` (manifest C2) — visual direction for the one page the IA serves.

## WHY: Rationale

Information Architecture for most products is a structural concern: how the pages of a multi-page product compose into a navigable whole, how items relate to containers, how a user finds what they need. For `fabro.report`, IA is almost entirely a *commitment* concern: the structural shape is trivial (one CLI, one page per Report, nothing else), and the real risk is that the shape quietly grows. A future agent who has internalized a large-product IA mental model will, in good faith, propose a home page ("people need to know what this is"), a listing page ("teams need to see recent uploads"), or a per-user route ("Reporters might want to find their past Reports"). Each of those proposals is structurally reasonable in a different product. In this product, each is a scope expansion that [[Principle - Narrow Forever]] and [[Product Thesis - Pickup Point, Not Triage Surface]] refuse.

The reason this Standard exists at all, then, is to *foreclose* IA growth by stating the IA shape explicitly and naming the categories of IA the product does not have. The cost of writing this Standard is a few paragraphs; the cost of not writing it is that every future IA-adjacent decision has to re-derive the commitment from upstream cards, and an agent doing that re-derivation at speed will get it wrong at least once. Capturing the commitment once, in the area the linter and the library recognize as the IA area, means every future proposal lands against an explicit reference.

The S3 layout is included in this Standard (as a completeness note, not as the user-facing IA) because agents reading "Information Architecture" expect *all* structural layouts — Reporter-facing and Triager-facing — to be covered. Omitting the S3 layout would leave a gap a Triager-facing proposal could walk through ("there's no IA card that says we don't also build a team-browse surface"). Including it, with the explicit note that Triagers see S3 directly and never through a service surface, closes that gap.

## WHEN: Timeline

Established 2026-04-22 alongside the rest of the Amplifier-tier build. Stable. Revisions to this Standard are scope changes (adding a page, adding a listing, adding navigation, adding an account surface) that go through the product owner as Strategy adjustments under [[Principle - Narrow Forever]], not implementation decisions. The minimal IA is load-bearing because every downstream surface inherits from it; weakening it at the IA layer weakens every rendered surface the product will ever have.

## HOW: Specification

### CLI Surface

The CLI surface of `fabro.report` is a small, fixed set of subcommands of the Fabro binary (renamed 2026-04-23; see [[Standard - Noun Vocabulary]] "Canonical CLI Commands"):

- **`fabro dump`** — produces a Debug Bundle tarball locally. Does not upload. This is the *creation* surface for a bundle; a Reporter who wants to inspect, archive, or selectively upload the tarball starts here.
- **`fabro dump --upload`** — produces a Debug Bundle AND uploads it in one step, returning the Report's URL on stdout. Reporter's primary path and the most common creation surface for a [[Component - Report]].
- **`fabro upload`** — uploads a previously-dumped Debug Bundle tarball. Supports `--dry-run` to verify the upload path (size pre-check, connectivity, presigned-URL issuance) without actually transferring bytes.
- **Flags** — at minimum, the flags confirmed in source:
  - `-m "short message"` — supplies an optional Reporter message that becomes a field on the resulting [[Component - Report]]. Per `docs/alexandria/sources/product-vision.md` L58.
  - arbitrary key/value metadata tags — become fields on the resulting [[Component - Report]]. Per `docs/alexandria/sources/product-vision.md` L85. The exact flag syntax for these tags (e.g., `--tag key=value`) is deferred to [[Standard - Interaction Patterns (CLI + Delete)]].
  - `--dry-run` (on `fabro upload`) — verifies the upload path without transferring bytes. Per `docs/alexandria/sources/cli-interaction-patterns.md`.

The CLI subcommands live in the main Fabro binary, not in a standalone tool. This is called out in `docs/alexandria/sources/product-vision.md` L111–L113 (Parent Product Context). The `fabro dump` / `fabro dump --upload` / `fabro upload` surface is **new** to Fabro's CLI — it is not currently in Fabro's documented command set — and building this intake service implies adding those subcommands.

There are no other CLI subcommands on this product's surface. No `fabro reports list`, no `fabro report status`, no `fabro dump delete` (the delete capability is on the Report Metadata Page via URL possession, not on the CLI). No auxiliary CLI for Triagers (Triagers use `aws s3 cp` or the team's own wrappers — see [[Agent - Triager]]).

### HTTP Surface

The HTTP surface of `fabro.report` is exactly one URL shape — the **Report URL** — and a small number of endpoints behind it that make the upload / render / delete contract work. The exact path format (host, slug layout, route prefixes) is deferred to [[System - Retention, Storage, and Key Scheme]]; this Standard names the *endpoints* and their *IA roles*, not their path strings.

The endpoints that must exist:

- **Render endpoint** — serves the [[Component - Report Metadata Page]] for a valid, non-expired, non-deleted Report. Addressed by the Report URL. No auth; capability-URL-addressed, per [[Decision - Capability URLs over Auth]]. Returns not-found for expired / deleted Reports per [[Decision - 30-Day Retention Default]].
- **Delete endpoint** — handles the delete action submitted from the [[Component - Report Metadata Page]]'s delete button. Capability-URL-addressed (no additional authentication). Removes the Report from the active set and deletes the Debug Bundle from S3.
- **Upload-coordination endpoint** — accepts an upload request from `fabro dump --upload` / `fabro upload` and creates a new Report. Per [[Standard - Interaction Patterns (CLI + Delete)]], the CLI does not POST bundle bytes to the service; the service mints a Report record, returns a presigned S3 PUT URL (and the eventual Report URL) to the CLI, and the CLI uploads the Debug Bundle directly to S3. The coordination-vs-presigned split is mechanism detail governed by [[System - Retention, Storage, and Key Scheme]]; this Standard only names the endpoint's IA role (create a Report, return a URL) and does not pin the path.

No other HTTP endpoints exist on this product's surface. Specifically:

- **No listing endpoint.** There is no `GET /reports`, `GET /my-reports`, `GET /recent`, or any other route that returns multiple Reports.
- **No browse endpoint.** There is no search, filter-by-metadata, or filter-by-Fabro-version surface on the service. (Triagers who want to filter do so against the S3 bucket with their own tooling; see the "S3 Layout" note below.)
- **No account endpoint.** No login, no signup, no "my account," no profile. See [[Decision - Capability URLs over Auth]].
- **No second rendered page.** The [[Component - Report Metadata Page]] is the only rendered page. There is no home page, no product-description page, no "about" page, no docs page on this service. (Product documentation lives elsewhere — in the Fabro repo, in the Alexandria library, wherever — not on the service's public HTTP surface.)
- **No health / status / dashboard endpoint.** Operational monitoring is a Triager-side concern, handled outside this service's public surface.

### Navigation, Hierarchy, and Sitemap

There is no navigation. There is no hierarchy. There is no sitemap. Each Report is addressable by its URL and only by its URL. Reports do not compose into a parent / child structure, a category, a project, a team, or any other container. The [[Component - Report Metadata Page]] has no links to other pages on this service because there are no other pages to link to.

This is a commitment under [[Principle - Narrow Forever]] and [[Product Thesis - Pickup Point, Not Triage Surface]]. An agent who reflexively adds a logo-links-to-home element, a breadcrumb, or a footer-navigation block is drifting; the Report Metadata Page is a standalone confirmation surface, not a page in a site.

### S3 Layout (Completeness Note)

The S3 layout is named here so the IA Standard covers the full structural picture of `fabro.report`, even though the Reporter and the Reporter-facing service never touch it. Triagers retrieve Debug Bundles directly from S3 with their own AWS credentials; from the Triager's perspective, the S3 bucket's key layout is the only "IA" they traverse, and they do so outside this service's public surface.

Structurally:

- Debug Bundles live in an S3 bucket, keyed by (or derived from) the [[Component - Report]] ID.
- The key layout is designed to make Triager retrieval straightforward: given a Report ID extracted from a URL, a Triager can construct the S3 key and run `aws s3 cp s3://<bucket>/<key> ./bundle.tar.gz`.
- Key details — the bucket name, the key format, whether the key is the Report ID directly or a derivative, any prefix or sharding structure — are [[System - Retention, Storage, and Key Scheme]] concerns. This Standard does not pin those details.
- The S3 bucket is **not** part of the service's public HTTP surface. Access is via AWS IAM, not via any endpoint on `fabro.report`. See [[Decision - Capability URLs over Auth]] "Scope of Applicability."

Importantly for IA: the S3 layout does **not** imply any service-side listing, filtering, or browsing capability. Triager-side aggregation (if the team ever wants it) is done with `aws` tooling and the team's own scripts against the bucket, not through any endpoint on this service.

### What the IA Is Not

This section is explicitly load-bearing. Each item below is a shape an agent might assume because it exists in adjacent products; each is named here to foreclose the assumption.

- **No user account pages.** No "my account," "settings," "profile," or "preferences" page. Reporters have no identity on the service (see [[Decision - Capability URLs over Auth]]).
- **No team dashboards.** No team-facing surface of any kind. Triagers retrieve from S3 directly; the service has no team-facing IA. See [[Agent - Triager]] and [[Principle - Reject Like bun.report Drift]].
- **No "my uploads" view.** A Reporter has no way to list their own past uploads through the service. URL possession is the only handle; lost URLs are not recoverable, by design. See [[Decision - Capability URLs over Auth]] "What It Costs."
- **No browse endpoint.** There is no route that returns more than one Report. Not for Reporters (who have no identity to scope by), not for Triagers (who use S3 directly).
- **No search.** There is no search UI, search endpoint, or filter-by-metadata surface on the service.
- **No multi-page site.** The [[Component - Report Metadata Page]] is the *only* rendered page. No home, no docs, no about, no help, no contact, no pricing.
- **No documentation surface on the service.** Fabro's documentation lives in the Fabro repo or wherever Fabro publishes docs; `fabro.report` does not host product documentation as a user-facing web surface.

### Test

Before adding any route, page, or CLI subcommand to this product, ask: "Does this route serve the one-CLI-one-page-per-Report contract, or does it extend the IA beyond that contract?" If it serves the contract (render the page, handle the delete action, accept the upload), it is in scope subject to the Decisions. If it extends the IA (adds navigation, a listing, a browse surface, a second rendered page, a second CLI subcommand, a user-account route), it is a scope change under [[Principle - Narrow Forever]] and goes to the product owner as a Strategy-adjustment question.

A second test: "If this IA change were implemented, would a Reporter or Triager have something new to *navigate through*?" If yes, the commitment to non-hierarchical IA has been broken. The correct answer is zero net navigation surface; confirm the proposed change preserves that, or escalate it.

## Sources

- `docs/alexandria/sources/product-vision.md` (L20–L32 end-to-end flow — CLI + S3 + URL shape; L31 receipt page content; L63–L64 Triagers use own AWS credentials, not the service; L87 capability-URL access; L100 Report Metadata Page as the only rendered surface; L111–L113 Parent Product Context — dump / upload subcommands as new to Fabro's CLI; Confirmed Product Decisions 2026-04-23 — CLI rename from `fabro debug` / `fabro report` to `fabro dump` / `fabro dump --upload` / `fabro upload`)
- `docs/alexandria/sources/cli-interaction-patterns.md` (canonical command names post-2026-04-23 rename; presigned-URL upload architecture; `--dry-run` flag)
- `docs/alexandria/inventory/card-manifest.md` (A6 entry)

# Agent - Triager

## WHAT: Definition

A **Triager** is a member of the Fabro team (currently Bryan + team) who retrieves Debug Bundles for bug investigation. The Triager's single most counterintuitive property — and the one most likely to be eroded by a builder agent working from a "bug-intake service" intuition — is this: **the Triager never interacts with the `fabro.report` service itself.** They retrieve Debug Bundles directly from S3 using their own AWS credentials, not through any endpoint, API, dashboard, or login on this service. The service has no Triager-facing surface at all; the team side of the pickup happens entirely out-of-band, in AWS and in the team's own tooling. The only piece of the service a Triager ever touches is the same short URL a Reporter shared with them — out-of-band, in a GitHub issue or Slack or email — from which they copy the Report ID and use it to locate the Debug Bundle in S3. The Triager is the **pickup** side of pickup-point; this product exists for their benefit, and precisely because of that, it gives them no surface.

## WHERE: Ecosystem

- Identity and vocabulary:
  - [[Standard - Noun Vocabulary]] — the canonical **Triager** definition (a member of the Fabro team who retrieves bundles for bug investigation; **never interacts with the intake service itself**; retrieves bundles directly from S3 using their own AWS credentials; the service has no Triager-facing surface) is the authoritative definition this card expands. The no-service-surface property is vocabulary-level, not implementation-level.
- Counterpart role:
  - [[Agent - Reporter]] — the self-hosted Fabro user on the other end of the pickup. The asymmetric-identity model is load-bearing: a Reporter has no identity on the service and interacts with the CLI only; a Triager has full AWS credentials and interacts with S3 only. The two roles sit on opposite sides of the service and neither is ever both.
- Access boundary:
  - [[Decision - Capability URLs over Auth]] — the capability URL the Reporter shares is a view+delete capability, not a retrieval capability. Triagers do not "unlock" the Debug Bundle through the URL; they locate the Debug Bundle via the Report ID encoded in the URL and retrieve it from S3 with their own AWS identity. The service has no access-control layer for Triagers because Triagers have no request path through the service. AWS IAM is the Triager's access-control layer, and it is orthogonal to this service.
- Scope commitment — this role is the single most load-bearing case of it:
  - [[Principle - Narrow Forever]] — Triager-facing UI (a team dashboard, a list of recent Reports, a filter by Fabro version, a triage UI, any Triager-facing endpoint) is on the **permanently-out-of-scope list**. The service exists for Triagers' benefit and gives them no surface, and that is not a v1 simplification — it is the product's identity. This is the single property most likely to be eroded by a builder agent's assumption that "of course the team needs a dashboard."
- Scope coherence:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the Triager is the picker-upper. The thesis says "the service's job ends at 'URL exists,'" which structurally requires that no part of the team's investigation workflow depends on an endpoint of this service. Any card that describes a Triager-facing surface is evidence the thesis has been abandoned.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped triage tooling (dashboards, per-team scoping, triage UI, symbolication exposed to the team in-browser) is the most common drift vector and the one Triagers are most likely to be imagined as needing. F3 enumerates each of these; this card is the positive statement that Triagers already get what they need (an S3 path) without any of them.
- Strategic why:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the product exists for the Triager's benefit: the Fabro team can help self-hosted users only if the team can retrieve the Debug Bundle, and the Triager is the team member doing the retrieving. Strategy is optimizing for the Triager's access to the bundle, not the Triager's experience on any service. Those two things are deliberately different.
- Audience commitment:
  - [[Decision - Who This Service Is For]] — the audience is self-hosted Fabro users (Reporter side); the Triager side is the Fabro team as it exists today, currently Bryan + team, with no role hierarchy or Triager subtypes. Widening the Triager definition (e.g., to third-party support partners, or to hosted-Fabro team members if that offering ever exists) is an audience change under that Decision, not an implementation change here.
- Referenced by:
  - [[Component - Report]] — the Report carries the ID a Triager copies out of the shared URL to locate the Debug Bundle in S3. The Report's URL is the capability for view + delete on the Metadata Page (Triager's incidental use); the Report ID alone — extracted from the URL — is what the Triager uses against S3 with their own AWS credentials.
  - [[Component - Report Metadata Page]] — the only surface the Triager ever touches on the service, and only incidentally (to click delete when a Reporter shares the URL and asks for deletion). Any Triager-specific capability on this page is a violation of the no-service-surface shape defined here.
  - [[Standard - Information Architecture]] — A6 captures the commitment that the Triager does not traverse the service's IA at all; Triager-side aggregation is done with `aws` tooling against the S3 bucket, not through any endpoint on this service.
- Forward references (planned in `docs/alexandria/inventory/card-manifest.md`, not yet drafted):
  - `System - Retention, Storage, and Key Scheme` (manifest A5) — the S3 key layout determines how a Triager translates a Report ID from a URL into an S3 path for retrieval; the key scheme is the only structural contract between this service and the Triager's retrieval path.

## WHY: Rationale

- Thesis: [[Product Thesis - Pickup Point, Not Triage Surface]] — the Triager is half of what makes "pickup, not triage" a structurally coherent product. If Triagers had a surface here, this service would be a triage product with an intake front-door rather than a pickup point, and the thesis would collapse into marketing language over a different shape.
- Strategy: [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — every Triager-side design choice is checked against "does this help the team retrieve the Debug Bundle?" The current shape — direct S3 retrieval, no service surface — answers that question with the minimum possible product, which is what Strategy is buying.
- Scope: [[Principle - Narrow Forever]] — Triager-facing UI is the single most tempting drift target because "the team obviously needs to see all the Reports" is a seductive argument that disguises itself as common sense. The commitment framing of F6 — this is not deferred, not v2, not "we'll add a dashboard later" — is the upstream answer to that argument.
- Driver: Without this card, downstream product-layer cards risk inventing a Triager-facing surface by implication: a key scheme that assumes team listing, a retention policy that assumes team visibility, a metadata field that only makes sense on a dashboard. Naming the Triager once, with the no-service-surface shape load-bearing, lets every downstream card check itself against that shape before it drifts.

## WHEN: Timeline

New role, no past. Confirmed 2026-04-22 by the product owner in the Users section of `docs/alexandria/sources/product-vision.md` (L61–L64). Stable. The no-service-surface shape is load-bearing and permanent; revising it requires reopening [[Principle - Narrow Forever]] as an identity-change conversation with the product owner, not an implementation-level discussion. Role composition — currently Bryan + team, no hierarchy, no subtypes — is expected to evolve as the Fabro team grows, but the structural property (Triagers use their own AWS credentials, not a service surface) does not change with team size.

## HOW: Behavior

### Context: When a Triager Acts

A Triager is typically prompted into action by seeing a `fabro.report` URL in a channel the Fabro team monitors — a GitHub issue filed against Fabro, a Slack message in a team or support channel, an email, or any other place Reporters are likely to paste it. The URL is not delivered by the service to the Triager; it arrives through whatever out-of-band channel the Reporter chose, which is the entire social surface of the product from the team's side. The Triager's job is then to get from that URL to the Debug Bundle, reproduce the bug, and fix it. Each of those three steps happens outside this service.

### What a Triager Does

- Sees a `fabro.report` URL in a shared context (GitHub issue, Slack, email).
- Extracts the Report ID from the URL. (The Report ID is the load-bearing portion; possession of the URL is not required for retrieval once the ID is known — the Triager's access-control layer is AWS IAM, not URL possession.)
- Retrieves the Debug Bundle from S3 using their own AWS credentials, via the team's own AWS tooling (`aws s3 cp`, or whatever scripts / internal tools the team uses to work with the S3 key scheme).
- Works with the Debug Bundle in their own environment: untar it, inspect its contents, reproduce the bug, symbolicate anything that needs symbolication, render anything that needs rendering — all using Fabro's own tooling or general-purpose tools, never through `fabro.report`.
- Optionally, if the Reporter shared the URL with them and asks for the Report to be deleted, the Triager can visit the Report Metadata Page and use the delete button. This is the only way a Triager ever touches the service, and it is an incidental capability borrowed from URL possession — not a privileged Triager operation.

### What a Triager Does Not Do

- Does **not** log in to `fabro.report`. The service has no account concept for Triagers.
- Does **not** visit a team dashboard. There is none, and there will not be one. See [[Principle - Narrow Forever]].
- Does **not** browse a list of recent Reports on this service. No listing endpoint exists — Triagers discover URLs through the out-of-band channels Reporters share them in, not through the service.
- Does **not** filter Reports by Fabro version, metadata tag, or upload time through a service-side UI. Any aggregation or filtering the team wants to do is done with their own AWS tooling against the S3 bucket, not on this service.
- Does **not** symbolicate, render, or inspect Debug Bundle contents through any endpoint of this service. Debug Bundle contents are never exposed through `fabro.report`. See [[Principle - Reject Like bun.report Drift]].
- Does **not** receive notifications, alerts, or dashboards from this service about new Reports. Team awareness of Reports comes from the channels Reporters share URLs in, not from the service.

### Examples

- **Example 1 — routine retrieval.** A Reporter pastes a `fabro.report` URL into a GitHub issue titled "cargo install hangs on CI." A Triager sees the issue, copies the URL, extracts the Report ID from it, runs `aws s3 cp s3://fabro-reports-bucket/<key-derived-from-Report-ID> ./bundle.tar.gz` (or the team's equivalent wrapper) using their own AWS credentials, untars the bundle, and begins investigation. The `fabro.report` service is not consulted at any step of the retrieval; it played no role after the Reporter's upload. See `System - Retention, Storage, and Key Scheme` (manifest A5, not yet drafted) for the actual key mapping.
- **Example 2 — Reporter asks for deletion after the fact.** A Reporter DMs a Triager a day after filing the issue and asks for the Report to be deleted because the Debug Bundle contained something sensitive. The Triager visits the URL, clicks the delete button, confirms the deletion, and is done. This interaction uses the Triager's URL possession as the authorization, not any Triager-specific privilege — any Reporter in possession of the URL could have done the same. See [[Decision - Capability URLs over Auth]].

### Anti-Examples

- **Wrong:** A card describes a "Triager dashboard" that lists recent Reports. This is the single most common drift vector this card names and is permanently out-of-scope by [[Principle - Narrow Forever]]. Any feature consumed by a Triager *through this service* violates the Triager role as defined. The correct shape is S3-direct retrieval with the team's own tooling.
- **Wrong:** An endpoint is added to `fabro.report` that accepts AWS credentials and returns a bundle (a "convenience wrapper" over S3). This reintroduces a Triager-facing service surface the product explicitly refuses to have; it also duplicates AWS IAM with a second authorization system. See [[Decision - Capability URLs over Auth]] and [[Product Thesis - Pickup Point, Not Triage Surface]].
- **Wrong:** The Triager definition is widened to include "anyone with a support interest" — e.g., a third-party support partner, a hosted-Fabro team member (if that offering ever exists). This is an audience change under [[Decision - Who This Service Is For]], not an implementation change. Route it to the product owner; do not resolve it in a downstream card.
- **Wrong:** A card describes a Triager "log in" flow, a Triager account, or a Triager-specific permission layer on this service. Triagers have no service-side identity on `fabro.report`; their identity is their AWS identity, which lives in AWS IAM and does not touch this service. See [[Standard - Noun Vocabulary]] and [[Decision - Capability URLs over Auth]].
- **Wrong:** A CLI string, a card, or a log line refers to a Triager as a "team member of fabro.report" or "maintainer." The Triager is a Fabro team member who retrieves bundles from S3; the phrase "fabro.report team member" implies a service-facing role that does not exist. See [[Standard - Noun Vocabulary]].

## Sources

- `docs/alexandria/sources/product-vision.md` (L61–L64 Users section → Triager definition; L22–32 end-to-end flow, step 5 Triager retrieval path; L74–89 Confirmed Product Decisions — Triager access path via direct S3 with own credentials at L89; L92–L99 Confirmed Entities — Triager at L99)
- `docs/alexandria/inventory/card-manifest.md` (A2 entry)

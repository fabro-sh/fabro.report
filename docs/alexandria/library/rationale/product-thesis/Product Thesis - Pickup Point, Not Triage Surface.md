# Product Thesis - Pickup Point, Not Triage Surface

Type: Product Thesis (Plank — identity-shaping bet)

## WHAT: The Thesis

`fabro.report` is a pickup point, not a triage surface: the entire public product is a CLI-driven intake that uploads a Debug Bundle to S3 and hands back a short URL, and that intake plus a minimal metadata receipt is the whole service — forever. All bundle retrieval, inspection, symbolication, rendering, and triage happen out-of-band, through a Triager's own S3 credentials, never through this service.

Counter-thesis: "The URL is already in the user's hands, so the service might as well show them (and their team) something useful about the bundle — contents preview, symbolicated stack, markdown-rendered logs, a per-team dashboard." This is the `bun.report`-shaped objection, and it is explicitly rejected here: usefulness on the receipt page is worth less than keeping the service's scope and attack surface permanently narrow.

## WHERE: Ecosystem

- Standards:
  - [[Standard - Noun Vocabulary]] — canonical names (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) this thesis uses and that downstream cards inherit.
- Anti-Patterns:
  - [[Principle - Reject Like bun.report Drift]] — paired negative frame; names the specific cascade failure (symbolication, rendering, feedback collection, team dashboards) that violating this thesis looks like.
- Strategy and scope commitment:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the strategic WHY this thesis serves: the product exists to make it easy for self-hosted Fabro users to get Debug Bundles to the Fabro team.
  - [[Principle - Narrow Forever]] — the paired commitment that turns this thesis's scope into a permanent posture rather than a v1 simplification; names the permanently-out-of-scope list.
- Tonal expression:
  - [[Principle - Minimal Public Surface Feels Calm]] — the aesthetic consequence of this thesis. A pickup point should *feel* like a pickup point: the Reporter drops a Debug Bundle, gets a URL back, and leaves. "Efficient and reassured" is the emotional shape of that interaction.
- Referenced by:
  - [[Decision - Who This Service Is For]] — the pickup-point thesis assumes both endpoints of the handoff (self-hosted Reporter, Fabro-team Triager); the Decision locks the audience the thesis depends on.
  - [[Decision - Capability URLs over Auth]] — a pickup point has two handoffs (Reporter-to-URL, URL-to-S3-for-Triagers), and neither involves authentication on this service; the Decision is the access-model coherence that makes the thesis structurally work.
  - [[Decision - No Content Dedup]] — a pickup point does not analyze what was dropped; the Decision refuses the content-analysis surface area dedup would introduce.
  - [[Decision - 30-Day Retention Default]] — a pickup point is a temporary handoff location; retention is structural to the thesis, and the Decision names the duration.
  - [[Agent - Reporter]] — the drop-off endpoint of the pickup; interacts with the service only via `fabro report` CLI and optionally the Report Metadata Page. The Reporter's narrow shape is the thesis expressed at the role layer on the upload side.
  - [[Agent - Triager]] — the pickup endpoint; retrieves the Debug Bundle from S3 with their own AWS credentials and never interacts with the intake service. The Triager's no-service-surface shape is the structural condition that makes "pickup, not triage" coherent.
  - [[Component - Report]] — the first-class entity created when the pickup succeeds; the Report is what "the URL exists" means in this thesis, and its authoritative field set is calibrated to "confirm a Debug Bundle arrived and let the Reporter delete it" — nothing that would serve a triage workflow.
  - [[Component - Report Metadata Page]] — the one rendered surface the thesis allows; its four-element content (ID, upload timestamp, bundle size, delete button) is the exact measurable signal this thesis's Validation Criteria names.
  - [[Standard - Competitive Context]] — the competitive-analysis one-pager that closes the "should we survey adjacent products?" question by citing this thesis; the thesis is what makes the borrowing stop at intake shape.
  - [[Standard - Information Architecture]] — the IA Standard that expresses the thesis's minimal surface as an explicit spec: one CLI subcommand, one rendered page per Report, no navigation, no hierarchy.
- Template reference:
  - `bun.report` at `~/p/oven-sh/bun.report` — borrowed for intake URL shape only, explicitly NOT used as a template for downstream processing, rendering, symbolication, feedback, or triage UI.

## WHY: Rationale

The Fabro team needs Debug Bundles from self-hosted Fabro users in order to help them when Fabro fails. The narrowest possible product that gets a bundle from user to team is: upload, return URL, share URL, retrieve from S3. Everything beyond that — rendering, symbolication, dashboards, feedback collection — belongs to a different product category (triage tooling) that the Fabro team does not need `fabro.report` to be. Collapsing those two categories into one service trades a small convenience gain against a large, permanent expansion of attack surface, operational burden, and agent-drift risk, and the trade is not worth it.

Stated positively: this product is a pickup point. A Reporter drops a bundle; a Triager picks it up. The service's job ends at "URL exists." The load-bearing framing is *pickup*, not *triage*, because a Factory-mode builder agent reading this thesis alone must come away understanding that any feature shaped like triage belongs somewhere else — not in this codebase, not in this service's public surface, and not on any roadmap for this product.

## Validation Criteria

- **Following:** A Reporter can run `fabro report`, receive a URL, and share it. A Triager can retrieve the bundle from S3 using their own credentials. Neither workflow requires the service to render, inspect, or summarize bundle contents.
- **Invalidated by:** A feature request that can only be satisfied by the service rendering bundle contents, symbolicating crashes, offering feedback collection, or presenting a team triage UI. If such a feature is accepted rather than routed to a different product or tool, the thesis has been abandoned.
- **Measurable signals:** Receipt page displays only metadata (Report ID, upload timestamp, bundle size, delete button). CLI output communicates only what is needed to share the URL. Zero endpoints expose bundle contents via the service. Triagers retrieve bundles directly from S3.

## WHEN: Timeline

New product, no past. Confirmed as a permanent identity decision on 2026-04-22 by the product owner: "narrow forever" is an identity commitment, not a v1 simplification. No successor, no predecessor. Will pair with `Principle - Narrow Forever (Roadmap as Commitment)` (manifest F6, not yet drafted) for the commitment side of this stance.

## HOW: Application

### What Following This Looks Like

- A Reporter hits a bug, runs `fabro report`, the CLI uploads the tarball to S3, and prints a short URL. The receipt page at that URL shows Report ID, upload timestamp, bundle size, and a delete button. Nothing else.
- A Triager sees the URL shared in a GitHub issue, copies the Report ID out of it, and fetches the bundle from S3 using their own AWS credentials. The service plays no role in their triage workflow.
- A feature request arrives asking for structured display of crash metadata. It is routed to a separate triage tool (or Fabro's own internal tooling), not implemented here.

### What Violating This Looks Like

- The receipt page adds a "bundle contents" tab that lists files inside the tarball. (This service is not a bundle browser.)
- An endpoint is added that fetches and renders logs from the bundle for the Reporter to read in the browser. (The Reporter is not the audience; Triagers are, and they use S3 directly.)
- A "team dashboard" is added that lists all Reports uploaded in the last 7 days, with filtering by Fabro version. (There is no team-facing surface on this service; team triage happens in S3 and in Fabro's own tooling.)
- A feedback-collection flow is added: after receiving a URL, the Reporter is prompted to describe what went wrong. (Optional message at CLI upload time is sufficient; post-upload feedback turns pickup into conversation.)
- Symbolication of stack traces is implemented server-side so the receipt page can show human-readable frames. (Symbolication is a triage concern; it lives with Triagers, not in the service.)

### Decision Heuristic

When facing a design tradeoff, ask: "Does this feature help a Reporter hand off a bundle, or does it help a Triager inspect one?" If it helps with handoff (e.g., better error message when upload fails, a clearer URL format), it is in scope. If it helps with inspection (e.g., any kind of bundle-contents display, any per-team view, any analysis), it is out of scope — permanently — and belongs in a different tool. When in doubt, choose the narrower option; "pickup point" wins any tie against "might be useful."

A second heuristic for handling template comparisons: if the answer to "how should we design this?" starts with "well, `bun.report` does X," stop. The only thing borrowed from `bun.report` is the intake URL shape. Every other design question must be answered from this thesis and its paired cards, not from `bun.report`'s feature set. See [[Principle - Reject Like bun.report Drift]].

## Sources

- `docs/alexandria/sources/product-vision.md` (L12, L18–32, L33–48, L74–88, L100–107, L134–141)
- `docs/alexandria/inventory/card-manifest.md` (F1 entry)

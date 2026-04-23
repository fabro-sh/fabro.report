# Product Thesis - Pickup Point, Not Triage Surface

Type: Product Thesis (Plank — identity-shaping bet)

## WHAT: The Thesis

`fabro.report` is a pickup point, not a triage surface: the entire public product is a pair of CLI-driven intakes that capture either a Debug Bundle (uploaded to S3 with a short URL handed back) or a Feedback submission (stored in S3 and pushed to a team Discord channel), and those two intakes plus a minimal metadata receipt for the Debug Bundle flow are the whole service — forever. All bundle retrieval, inspection, symbolication, rendering, and triage happen out-of-band, through a Triager's own S3 credentials for Debug Bundles and through the team's Discord channel (with optional S3 retrieval) for Feedback, never through this service.

**Two intake types, forever.** Confirmed as a load-bearing identity commitment on 2026-04-23: `fabro.report` is a two-intake service — Debug Bundles and Feedback — permanently. A third intake surface is not a feature; it is a product re-founding that requires an identity-change conversation with the product owner. The two intakes are parallel but not identical: both are pickup-shaped (the submitter drops, the team picks up), and both refuse to host the triage surface that the retrieval steps belong to. See [[Principle - Narrow Forever]] for the ceiling framing.

Counter-thesis: "The URL is already in the user's hands, so the service might as well show them (and their team) something useful about the bundle — contents preview, symbolicated stack, markdown-rendered logs, a per-team dashboard." This is the `bun.report`-shaped objection, and it is explicitly rejected here: usefulness on the receipt page is worth less than keeping the service's scope and attack surface permanently narrow. The same refusal applies to the Feedback flow — Feedback is a pickup, not a conversation; the service does not host threaded replies, submitter dashboards, or feedback analytics.

### The Two Pickup Shapes

Both intakes are pickup-shaped — a submitter drops, the team picks up, out-of-band — but the mechanics differ in ways that are structural to each flow's identity:

- **Debug Bundle pickup.** A Reporter runs `fabro dump --upload` (or `fabro upload` against a previously-dumped tarball). The service uploads the tarball to S3, creates a Report, and returns a short capability URL. The Reporter shares the URL with the Fabro team out-of-band (GitHub issue, Slack, email). A Triager retrieves the Debug Bundle from S3 directly using their own AWS credentials. The service hosts only a minimal Report Metadata Page at the URL — confirming upload and offering a delete action.
- **Feedback pickup.** A Feedback Submitter runs `fabro feedback` (with optional text, files, or `--email`). The service stores the submission in S3 under a UUIDv7 key and posts a notification to a team Discord channel via webhook. The CLI returns a fire-and-forget confirmation — **no URL, no page, no delete**. The Fabro team reads the Discord channel; a Triager may optionally retrieve the full submission from S3 if the Discord post does not have everything they need.

The two flows share the same identity — pickup point, not triage surface — but differ in post-pickup mechanics. Debug Bundles give the Reporter back a URL (because the Reporter needs to share something identifying the bundle). Feedback gives the Submitter back nothing beyond confirmation (because the Submitter's involvement ends at "it was sent" and Discord is the team's surface, not the Submitter's). See [[Agent - Feedback Submitter]], [[Component - Feedback]], and [[System - Feedback Storage & Notification]] for the role, entity, and system shapes of the feedback pickup.

**Discord is a push target, not a triage surface hosted by the service.** The service itself still has no team-facing UI, no dashboard, no login, no listing endpoint. What is new on 2026-04-23 is that the service's *behavior* includes posting team notifications to an external channel — Discord is a team-facing surface, but it is hosted elsewhere, not on `fabro.report`. That distinction is load-bearing; agents who read "team-facing Discord notification" and conclude "we need a triage UI on the service" have misread the thesis. The Discord post is push-only, and the team's triage behavior (reading the post, optionally pulling the full submission from S3) is still out-of-band from the service.

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
  - [[Agent - Reporter]] — the drop-off endpoint of the pickup; interacts with the service only via the `fabro dump --upload` (or `fabro upload`) CLI path and optionally the Report Metadata Page. The Reporter's narrow shape is the thesis expressed at the role layer on the upload side.
  - [[Agent - Triager]] — the pickup endpoint; retrieves the Debug Bundle from S3 with their own AWS credentials and never interacts with the intake service. The Triager's no-service-surface shape is the structural condition that makes "pickup, not triage" coherent.
  - [[Component - Report]] — the first-class entity created when the pickup succeeds; the Report is what "the URL exists" means in this thesis, and its authoritative field set is calibrated to "confirm a Debug Bundle arrived and let the Reporter delete it" — nothing that would serve a triage workflow.
  - [[Component - Report Metadata Page]] — the one rendered surface the thesis allows; its four-element content (ID, upload timestamp, bundle size, delete button) is the exact measurable signal this thesis's Validation Criteria names.
  - [[Standard - Competitive Context]] — the competitive-analysis one-pager that closes the "should we survey adjacent products?" question by citing this thesis; the thesis is what makes the borrowing stop at intake shape.
  - [[Standard - Information Architecture]] — the IA Standard that expresses the thesis's minimal surface as an explicit spec: one CLI subcommand, one rendered page per Report, no navigation, no hierarchy.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the CLI and delete-flow mechanics that keep the pickup-point interaction narrow; dump-and-upload, a progress bar, a one-click delete, and no post-upload feedback loop are the thesis expressed as concrete UX choices.
  - [[Standard - Accessibility Policy]] — the two-surface accessibility floor (CLI `NO_COLOR`, web WCAG 2.2 AA) that keeps the thesis's minimal surface *usable* by everyone a self-hosted Fabro user might be.
  - [[System - Debug Bundle Retention, Storage, and Key Scheme]] — the data topology that makes the pickup-point thesis structurally coherent: one bucket, one lifecycle rule, per-Report directories, capability URLs; the thesis's "URL exists; service's job ends" shape in infrastructure terms.
  - [[Decision - Service Stack]] — the stack-layer commitment (Rust on AWS Fargate, Terraform-provisioned, SSR HTML) that keeps the pickup-point infrastructure as narrow as the pickup-point feature surface.
  - [[System - Service Topology]] — the operational-topology expression of the pickup-point thesis: one Fargate task, one DNS record, one IAM role, one Terraform repository. Anything that proposes a second service or a second storage surface is topology drift against the thesis.
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — a pickup point's operational monitoring asks "is the pickup itself functioning?", not "what was dropped off?". A13's operational-only Sentry scope is the observability-layer expression of the pickup-point thesis; bundle-as-crash-payload forwarding would be a triage-surface move this thesis refuses.
  - [[Standard - Design System (Receipt Page + CLI Output)]] — C2's narrow, terminal-flavored visual system is the pickup-receipt feel expressed at the visual layer; a design that would make the Report Metadata Page look like a triage dashboard fails this thesis at the visual layer.
  - [[Standard - Receipt Page Sketch]] — C3's narrow centered column, three metadata rows, and secondary delete affordance are the pickup-receipt shape expressed at the layout layer; a layout that would accommodate triage surface area (tabs, sidebars, related-Reports rails) abandons the thesis at the layout layer.
- Feedback-flow cards (added 2026-04-23 when the product scope expanded from one intake to two):
  - [[Agent - Feedback Submitter]] — the drop-off endpoint of the Feedback pickup; interacts with the service only via the `fabro feedback` CLI and receives no post-submission surface. The Feedback Submitter's asymmetric identity model (optional email + server-logged IP, distinct from the no-identity Reporter) is this thesis's two-intake-types-forever commitment expressed at the role layer on the feedback side.
  - [[Component - Feedback]] — the first-class entity created when a Feedback pickup succeeds; parallel to [[Component - Report]] but shaped by the fire-and-forget nature of the Feedback flow (UUIDv7 key, no capability URL, no rendered page).
  - [[System - Feedback Storage & Notification]] — the data-and-notification topology that makes the Feedback pickup structurally coherent: `feedback.json` + `feedback.txt` in S3 plus a Discord webhook post. The thesis's "pickup, not triage surface" shape expressed in infrastructure terms for the feedback flow — and the explicit boundary that Discord is a push target, not a service-hosted triage UI.
  - [[Decision - Fire-and-Forget Feedback]] — the access-model expression of the Feedback pickup's thesis-level shape: the Submitter drops and leaves, with no post-submission service surface because Discord (out-of-band) is where the team picks up. A19's "nothing post-submission; the confirmation is the whole contract" commitment is this thesis's pickup-shape phrased at the Decision layer.
- Template reference:
  - `bun.report` at `~/p/oven-sh/bun.report` — borrowed for intake URL shape only, explicitly NOT used as a template for downstream processing, rendering, symbolication, feedback, or triage UI.

## WHY: Rationale

The Fabro team needs data from self-hosted Fabro users in order to help them when Fabro fails. That data arrives in two shapes: **reproducibility data** (what happened technically — bundled up by `fabro dump --upload` as a Debug Bundle) and **voice** (what the user wanted to say — captured by `fabro feedback` as a Feedback submission). The narrowest possible product that gets both kinds of data from user to team is: two small CLIs, two S3-shaped pickup flows, two out-of-band team-retrieval paths (S3-direct for bundles, Discord-first for feedback). Everything beyond that — rendering, symbolication, dashboards, in-service triage UIs, feedback analytics — belongs to different product categories (triage tooling, user-research tooling) that the Fabro team does not need `fabro.report` to be. Collapsing those categories into one service trades a small convenience gain against a large, permanent expansion of attack surface, operational burden, and agent-drift risk, and the trade is not worth it.

Stated positively: this product is a two-intake pickup point. A Reporter drops a Debug Bundle and gets a URL back; a Feedback Submitter drops a message and gets a confirmation back. A Triager picks up Debug Bundles from S3 with their own credentials; the Fabro team picks up Feedback from Discord (with optional S3 retrieval). The service's job ends at "URL exists" (for Debug Bundles) and "Discord post sent" (for Feedback). The load-bearing framing is *pickup*, not *triage*, because a Factory-mode builder agent reading this thesis alone must come away understanding that any feature shaped like triage belongs somewhere else — not in this codebase, not in this service's public surface, and not on any roadmap for this product. That framing now covers **both** intakes equally.

## Validation Criteria

- **Following:** A Reporter can run `fabro dump --upload`, receive a URL, and share it. A Triager can retrieve the Debug Bundle from S3 using their own credentials. A Feedback Submitter can run `fabro feedback`, optionally with text, files, or `--email`, and receive a confirmation. The Fabro team can read the resulting Discord post and, if needed, retrieve the full submission from S3. None of those workflows requires the service to render, inspect, or summarize bundle contents; none requires the service to host a triage surface for either intake.
- **Invalidated by:** A feature request that can only be satisfied by the service rendering bundle contents, symbolicating crashes, offering post-upload feedback collection on the Debug Bundle flow, presenting a team triage UI for either intake, or hosting a submitter dashboard / feedback analytics surface for the Feedback flow. Also invalidated by: a proposal to add a third intake type (neither Debug Bundle nor Feedback) without routing it as an explicit identity-change conversation with the product owner. If any such feature is accepted rather than routed to a different product or tool, the thesis has been abandoned.
- **Measurable signals:** Receipt page displays only metadata (Report ID, upload timestamp, bundle size, delete button). `fabro feedback` returns only a confirmation line — no URL, no page rendered. CLI output for both intakes communicates only what the submitter needs (a URL to share for Debug Bundles; a confirmation for Feedback). Zero endpoints expose bundle contents or feedback contents via the service. Triagers retrieve Debug Bundles directly from S3; the Fabro team reads Feedback from Discord (with optional S3 retrieval). The count of intake types remains exactly two.

## WHEN: Timeline

New product, no past. Confirmed as a permanent identity decision on 2026-04-22 by the product owner: "narrow forever" is an identity commitment, not a v1 simplification. No successor, no predecessor. Paired with [[Principle - Narrow Forever]] for the commitment side of this stance.

**2026-04-23 scope expansion (confirmed identity update):** the thesis's pickup-point framing broadened from one intake (Debug Bundles) to two (Debug Bundles + Feedback). The load-bearing "pickup point, not triage surface" identity survives the expansion intact — both intakes are pickup-shaped and the service still hosts no triage surface. The new identity-level commitment is **"two intake types, forever"**: exactly these two, permanently, with a third intake surface requiring a product-owner identity-change conversation (see `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` for the full record of the expansion and `docs/alexandria/sources/feedback-flow.md` for the source confirmation). Discord entered the product as a **push-notification target** on the same date; it is the product's first team-facing surface in the sense that the Fabro team reads it, but it is hosted externally — the `fabro.report` service itself still has no team-facing UI.

## HOW: Application

### What Following This Looks Like

- A Reporter hits a bug, runs `fabro dump --upload`, the CLI produces the tarball and uploads it to S3, and prints a short URL. The Report Metadata Page at that URL shows Report ID, upload timestamp, bundle size, and a delete button. Nothing else.
- A Triager sees the URL shared in a GitHub issue, copies the Report ID out of it, and fetches the Debug Bundle from S3 using their own AWS credentials. The service plays no role in their triage workflow.
- A Feedback Submitter wants to share a bug report, a feature request, or general voice. They run `fabro feedback "love the new release"` (or pipe text in, or attach files, or add `--email`), the CLI returns a short confirmation, and their involvement ends. The service stores the submission in S3 and posts a notification to the team's Discord channel.
- A Fabro team member reads the Discord channel, sees a new feedback post with header (email or placeholder), sysinfo, body, and footer (feedback ID, optional S3 link), and responds out-of-band (opens a GitHub issue, replies by email if appropriate, pulls the full submission from S3 if the Discord post doesn't have everything). No in-service triage UI is involved.
- A feature request arrives asking for structured display of crash metadata, or for a feedback-analytics dashboard. It is routed to a separate triage tool, a user-research tool, or Fabro's own internal tooling, not implemented here.

### What Violating This Looks Like

- The Report Metadata Page adds a "bundle contents" tab that lists files inside the tarball. (This service is not a bundle browser.)
- An endpoint is added that fetches and renders logs from the Debug Bundle for the Reporter to read in the browser. (The Reporter is not the audience; Triagers are, and they use S3 directly.)
- A "team dashboard" is added that lists all Reports uploaded in the last 7 days, with filtering by Fabro version. (There is no team-facing surface on this service; team triage happens in S3 and in Fabro's own tooling.)
- A post-upload feedback-collection flow is added **on the Debug Bundle flow**: after receiving a URL, the Reporter is prompted to describe what went wrong. (Optional message at CLI upload time is sufficient; post-upload feedback turns pickup into conversation. The standalone `fabro feedback` CLI is a separate intake, not an extension to the Debug Bundle flow — see [[Principle - Reject Like bun.report Drift]] for the distinction.)
- A feedback-analytics or feedback-browsing surface is added on the service — a page that lists recent Feedback submissions, a dashboard aggregating feedback themes, or a submitter view of their own past submissions. (Feedback is fire-and-forget for the Submitter and read out-of-band on Discord by the team; adding a service-hosted feedback surface turns Feedback into a forum, not a pickup.)
- A third intake type is proposed — a new kind of submission that is neither a Debug Bundle nor a Feedback — and is routed as an implementation decision rather than as an identity-change conversation with the product owner. (The "two intake types, forever" commitment is the thesis-level ceiling; adding a third is a product re-founding.)
- Symbolication of stack traces is implemented server-side so the Report Metadata Page can show human-readable frames. (Symbolication is a triage concern; it lives with Triagers, not in the service.)
- The Discord webhook post is rewritten to include inline action buttons (reply, assign, resolve) or a rendered thread view — i.e., Discord is treated as a triage UI that the service is responsible for evolving. (Discord is a push target. Triage behavior on the team side is out-of-band, even when it happens in Discord; the service's responsibility ends at "post sent.")

### Decision Heuristic

When facing a design tradeoff, ask: "Does this feature help a Reporter hand off a Debug Bundle, or a Feedback Submitter hand off a Feedback — or does it help a Triager inspect one, or an in-service triage / forum experience grow?" If it helps with handoff (e.g., better error message when upload fails, a clearer URL format, a crisper `fabro feedback` confirmation line), it is in scope. If it helps with inspection, analysis, team-side triage UI, or submitter-side continued engagement after the pickup, it is out of scope — permanently — and belongs in a different tool. When in doubt, choose the narrower option; "pickup point" wins any tie against "might be useful." Both intakes answer to the same heuristic.

A second heuristic for handling template comparisons: if the answer to "how should we design this?" starts with "well, `bun.report` does X" or "well, `bun feedback` does X," check the borrowing boundary. The only things borrowed from the `bun` family are **intake shapes**: the intake URL shape for Debug Bundles (from `bun.report`) and the CLI-intake shape for Feedback (from `bun feedback`). Every other design question must be answered from this thesis and its paired cards, not from either template's feature set. See [[Principle - Reject Like bun.report Drift]].

A third heuristic for the ceiling: if the answer to "what should this product do?" involves a **third intake type** — neither a Debug Bundle nor a Feedback — stop. That is an identity-change proposal, not an implementation proposal, and it must be routed to the product owner as a Strategy question before anything is designed or built. See [[Principle - Narrow Forever]]'s "no third intake type, forever" commitment.

## Sources

- `docs/alexandria/sources/product-vision.md` (L12, L18–32, L33–48, L74–88, L100–107, L134–141)
- `docs/alexandria/sources/feedback-flow.md` (2026-04-23 scope-expansion framing; confirmed two-intake-types-forever commitment; Discord as push target)
- `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` (Part 1 identity-impact assessment for F1 and Part 3 cascade map)
- `docs/alexandria/inventory/card-manifest.md` (F1 entry)

# Principle - Strategy: Support Effectiveness for Self-Hosted Users

## WHAT: The Principle

`fabro.report` exists for exactly one strategic reason, stated in full: Fabro users run Fabro on their own systems; when Fabro fails for them, the Fabro team wants to help; to help, the team needs the data from those Fabro runs; `fabro.report` exists to make it easy for users to get that data to the team. The product is optimizing for *support effectiveness for self-hosted users* — nothing else. Every design decision on this service must trace back to "did this make it easier for the team to retrieve the Debug Bundle from a self-hosted user who hit a bug?" If the answer is no, the decision is out of scope — even if the decision itself is individually reasonable.

Three things the Strategy is explicitly NOT optimizing for, called out in the confirmed source so they cannot be silently reintroduced:

- **Not a formal support SLA.** `fabro.report` is not a ticketing product, not a commitment device, not a vehicle for response-time guarantees. It makes bundle retrieval easy; it does not manufacture obligations around the retrieval.
- **Not compliance posture.** `fabro.report` is not an audit artifact, not a regulated-data handling surface, not a GDPR/SOC-2/HIPAA-shaped product. The Reporter has no identity; the service has no account concept; there is nothing to be compliant *about* on the public surface.
- **Not marketing telemetry.** `fabro.report` does not exist to measure Fabro adoption, instrument user behavior, or feed a growth funnel. Reporter identity is deliberately absent; metadata captured at upload exists to help the team fix the specific bug, not to build a dataset about who uses Fabro.

The load-bearing trade: this Strategy optimizes for *the team can retrieve the data* over *the user sees rich diagnostics in-browser or maintains an ongoing dialog with the service*. That single trade is what makes `fabro.report` structurally different from a full crash-report product or a forum-shaped feedback product. A Reporter's in-browser experience is intentionally minimal — a Report Metadata Page with Report ID, upload timestamp, bundle size, and a delete button, and nothing else. A Feedback Submitter's post-submission experience is intentionally empty — a CLI confirmation line and nothing else. Both minimums exist because richer submitter-facing experience is not what Strategy is buying. Strategy is buying *team access to the data the team needs*. Everything else is a distraction from that.

### Two Mechanisms, Same Strategic WHY

As of the 2026-04-23 scope expansion, this Strategy is served by **two mechanisms**, not one. The Strategic WHY is unchanged — support effectiveness for self-hosted Fabro users, through team retrieval of the data the team needs to help — but the data arrives in two complementary shapes:

- **Debug Bundles carry reproducibility data.** What happened technically: crash state, logs, stack traces, environment, configuration, whatever `fabro dump` packages up. Retrieved by Triagers directly from S3 with their own AWS credentials.
- **Feedback carries voice.** What the user wanted to say: bug reports in the user's own words, feature requests, praise, complaints, general voice. Retrieved by the Fabro team from Discord (with optional S3 retrieval for the full submission).

Both mechanisms serve the same Strategic WHY — *make it easy for the team to help self-hosted users* — by giving the team different but complementary kinds of signal. A Debug Bundle without voice tells the team what broke but not what the user wanted; voice without reproducibility data tells the team what the user wanted but not how to verify. The Strategy buys both shapes by funding both intakes; it does not fund a third mechanism (user-facing diagnostics, in-service analytics, account-based history) because none of those would increase the team's ability to help a user whose data is already in hand.

The three anti-goals from the confirmed Strategic WHY — **not a formal support SLA**, **not compliance posture**, **not marketing telemetry** — apply to both mechanisms equally. Feedback is not a ticketing product any more than Debug Bundles are; the Feedback Submitter is not promised a response, does not own a case record, and does not get a queue position. IP logging on the Feedback flow (see [[Decision - IP Logging for Abuse Protection]]) is scoped strictly to abuse protection and basic attribution — **not** marketing telemetry, **not** cross-submission user tracking, **not** compliance audit-trail posture. The anti-goals hold.

## WHERE: Ecosystem

- Implements:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the pickup-point thesis is the product-shape expression of this Strategy. "Pickup point, not triage surface" is what *support effectiveness for self-hosted users* looks like when you translate Strategy into a product shape.
- Paired with:
  - [[Principle - Narrow Forever]] — Strategy says what the product is for; Narrow Forever says the scope that serves this Strategy is permanent, not a v1 simplification. Together they lock both the WHY and the scope.
- Defended by:
  - [[Principle - Reject Like bun.report Drift]] — each `bun.report`-shaped drift (symbolication, rendering, dashboards, feedback) is a drift *away* from this Strategy, because each would trade team-retrieval effort against richer-Reporter-experience effort, which this Strategy rejects.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the canonical names (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) this Principle uses. Specifically, the Triager role as defined in that Standard — "never interacts with the intake service itself" — is a direct expression of this Strategy.
- Audience scope:
  - **Self-hosted Fabro users** are the primary and only audience. Reporters in scope are those running Fabro on their own systems. A future hosted-Fabro offering's user bundles are **not in scope** under this Strategy; if hosted-Fabro users ever need bundle intake, that is a separate product question, not a scope expansion of `fabro.report`.
- Referenced by:
  - [[Principle - Minimal Public Surface Feels Calm]] — the tonal expression of this Strategy. "Efficient and reassured" is the emotional shape of a Reporter who pastes the URL into the right place and gets back to work — which is how the Fabro team gets the Debug Bundle. Warmth that slows the handoff trades Reporter-facing polish against team-retrieval effort, which this Strategy rejects.
  - [[Decision - Who This Service Is For]] — formalizes the self-hosted-only audience as a Decision card; cites this Principle as its direct governing rationale. The Decision is the audience-layer expression of this Strategy.
  - [[Decision - Capability URLs over Auth]] — the access-model expression of this Strategy; auth would add friction to the Reporter's hand-off path, which is the opposite direction from what Strategy is optimizing for.
  - [[Decision - 30-Day Retention Default]] — retention is calibrated to the realistic triage window for a self-hosted-support workflow, which is the retrieval shape Strategy is optimizing for.
  - [[Decision - Service Stack]] — the stack-layer expression of this Strategy. Rust on AWS Fargate is chosen because the IAM-role-based S3 access path makes team retrieval operationally simple; each stack piece is justified by "does this help the team retrieve a bundle reliably?"
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — the observability-layer expression of this Strategy. Service-side reliability is load-bearing for retrieval; Sentry-for-service-errors is the mechanism that preserves the reliability premise. Sentry-as-bundle-processor would trade Reporter-facing richness against team-retrieval effort, which this Strategy rejects.
  - [[System - Service Topology]] — the operational topology is as narrow as the product because narrow topology serves reliable team retrieval; every topology piece is justified against the retrieval path this Strategy optimizes for.
  - [[Agent - Reporter]] — the role this Strategy serves (self-hosted Fabro user handing off a Debug Bundle); the Reporter's narrow post-upload surface is the Strategy's concrete expression on the Reporter side of the pickup.
  - [[Agent - Triager]] — the role this Strategy exists for (Fabro team member retrieving bundles); the Triager's no-service-surface shape — retrieving directly from S3 — is the Strategy's concrete expression on the team side of the pickup.
  - [[Standard - Competitive Context]] — the one-pager closes the competitive-analysis question because no competitive-differentiation work is needed when this Strategy is the entire purpose; audiences whose intake shapes differ from self-hosted-Fabro-user retrieval are structurally outside this Strategy's scope.
- Two-intake-types-forever cards (added 2026-04-23 when the product scope expanded to carry voice as a second mechanism alongside reproducibility data):
  - [[Agent - Feedback Submitter]] — the persona that carries the voice mechanism. Also a self-hosted Fabro user, in the same audience [[Decision - Who This Service Is For]] locks. Feedback Submitters and Reporters overlap as populations (a given Fabro user may be both at different moments), but the roles serve Strategy through different data shapes.
  - [[System - Feedback Storage & Notification]] — the operational mechanism that delivers voice to the team. Discord-as-push-notification is chosen because it gets voice in front of the Fabro team with minimal service-side triage infrastructure — which is exactly what this Strategy funds: team retrieval of data, not in-service management of it.
  - [[Decision - IP Logging for Abuse Protection]] — the identity-data Decision that scopes IP logging on the Feedback flow to abuse protection only. Explicitly preserves the three anti-goals of this Strategy (no SLA, no compliance posture, no marketing telemetry); IP is logged for service-reliability reasons, not Strategy-extension reasons.
  - [[Decision - Fire-and-Forget Feedback]] — the Strategic WHY expressed in the Feedback Submitter's post-submission experience: no URL, no delete, no account. The Submitter's involvement ends at "sent" because Strategy is buying team retrieval of voice, not user-facing continued engagement.

## WHY: Belief

The Strategic WHY was captured verbatim on 2026-04-22 from the product owner in the Confirmed Product Decisions section of the source, and it is short enough to quote in full: *Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs. `fabro.report` exists to make it easy for users to get that data to the team.* That is the entire Strategy. It is not a summary; it is the thing itself.

A Principle shorter than one paragraph tempts an agent to elaborate it. Do not. Every elaboration that introduces an adjacent purpose — "also for marketing," "also for support SLA," "also for compliance" — erodes the Principle's ability to rule features out. The source names those three adjacent purposes explicitly as things the Strategy is *not* optimizing for, and that negative framing is the other half of the Principle. An agent proposing a feature should be able to cite this Principle against the feature, not around it.

The reason this Principle is Foundation-tier rather than Amplifier-tier is that every downstream design choice on this service — what to display on the Report Metadata Page, what metadata to capture at upload, whether to add any Reporter-facing endpoint, whether to integrate with any third-party tool — reduces to a trade between "team can retrieve the bundle more easily" and "Reporter sees a richer surface." The Strategy ranks the first strictly above the second. Without that ranking stated upstream, each individual trade looks like a toss-up and the product drifts into a fuller shape through many small reasonable choices. With the ranking stated, the choices become mechanical.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner, captured in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L78). Stable. Revisions to this Principle require product-owner approval; they are not implementation details. A scope expansion proposal that implicitly revises this Principle — e.g., "let's add telemetry so we know which Fabro versions are hitting bugs most" — should be surfaced as a Strategy change, not absorbed as a feature.

**2026-04-23 mechanism-set expansion:** the Strategic WHY and the three anti-goals are **unchanged** by the 2026-04-23 scope expansion. What changed is the list of mechanisms that serve this Strategy: the set grew from one (Debug Bundles) to two (Debug Bundles + Feedback). The same Strategy funds both mechanisms, by different data shapes (reproducibility data vs. voice). See `docs/alexandria/sources/feedback-flow.md` and `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` Part 1 for the scope-expansion record; Part 3 confirms Strategy is unchanged and only gains a second mechanism. Adding a **third** mechanism would require Strategy approval from the product owner; see [[Principle - Narrow Forever]]'s "no third intake type, forever" ceiling.

## HOW: Application

### What Following This Principle Looks Like

- The `fabro dump --upload` (and standalone `fabro upload`) CLI together with the service make it as frictionless as possible for a self-hosted Fabro user who hit a bug to get their Debug Bundle (reproducibility data) to the Fabro team. The upload path is short; the URL is easy to share; the team can retrieve the Debug Bundle from S3 with their own AWS credentials. Each of these is justified by "team can retrieve the reproducibility data."
- The `fabro feedback` CLI makes it as frictionless as possible for a self-hosted Fabro user to get voice (bug reports, feature requests, praise, complaints, general feedback) to the Fabro team. The submission path is fire-and-forget; the Discord webhook posts the submission to a team channel; the team reads it on Discord and optionally pulls the full submission from S3. Each of these is justified by "team can retrieve the voice."
- The Report Metadata Page shows only what the Reporter needs to confirm the upload happened and to be able to delete it if they want. `fabro feedback`'s CLI confirmation shows only what the Feedback Submitter needs to know that the submission landed. Neither surface tries to be useful beyond that because usefulness to the submitter beyond confirmation is not what Strategy is buying.
- Metadata captured at upload (Fabro version, optional reporter message, arbitrary key/value tags) is exactly the metadata the team needs to triage which Debug Bundle to look at first. Metadata captured at Feedback submission (Fabro version / platform / arch, optional email, server-logged IP) is exactly the metadata the team needs to prioritize voice and — if abuse spikes — protect the service. Neither is a telemetry feed and neither is an identity system.
- When the team wants a new capability, the first question is "does this help a team member retrieve reproducibility data or voice?" If the new capability is consumed by a Reporter or a Feedback Submitter after submission, it is probably not what Strategy is buying and should be scrutinized.

### What Violating This Principle Looks Like

- Adding analytics that measure how many Reporters complete uploads per week. (This measures product adoption; it does not help the team retrieve a specific bundle.)
- Adding a "status" indicator on the Report Metadata Page showing whether the team has viewed the bundle. (This introduces a support-SLA shape; Strategy is explicit that SLA is not what is being optimized for.)
- Adding a privacy dashboard or audit log for Reporters. (This manufactures compliance posture; Strategy is explicit that compliance is not what is being optimized for.)
- Extending the product toward hosted-Fabro users before the hosted-Fabro question has been reopened as a Strategy question. (The audience is self-hosted, primary and only; extending audience is a Strategy change, not an implementation change.)
- Building a Reporter-facing diagnostic view because "the user might want to see what they sent." (This is the paradigmatic drift — trading team-retrieval investment against Reporter-facing richness, in the exact direction Strategy rejects.)

### The Strategy Test

Before building or proposing any feature, ask two questions in order:

1. **Does this help the Fabro team retrieve reproducibility data (Debug Bundles) or voice (Feedback) from a self-hosted Fabro user?** If yes, it may be in scope — check [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Narrow Forever]]. If no, stop.
2. **If this feature exists, is its audience a Reporter or Feedback Submitter *after* submission, or the Fabro team *retrieving* data?** If submitter-after-submission, this Strategy does not fund it; the feature is trading the wrong direction and should be reconsidered. If Fabro-team-retrieving, it is a candidate for scope (still subject to Narrow Forever and the Product Thesis).

Any feature that fails either test is not a Strategy-funded feature. It may still be a good idea; it belongs in a different product or in Fabro's internal tooling, not here.

## Sources

- `docs/alexandria/sources/product-vision.md` (L78 Strategic WHY; L52–64 Users section; L12–L14 Vision Summary; L115–128 Raven's Inferences and Open Questions — self-hosted-audience scope)
- `docs/alexandria/sources/feedback-flow.md` (2026-04-23 — Feedback as voice-carrying second mechanism serving the same Strategic WHY; anti-goals preserved across both flows)
- `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` (Part 1 F5 impact assessment — Strategy survives cleanly; Part 3 cascade — two-mechanism expansion)
- `docs/alexandria/inventory/card-manifest.md` (F5 entry)

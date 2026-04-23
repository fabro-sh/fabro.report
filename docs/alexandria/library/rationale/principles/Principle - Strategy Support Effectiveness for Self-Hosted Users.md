# Principle - Strategy: Support Effectiveness for Self-Hosted Users

## WHAT: The Principle

`fabro.report` exists for exactly one strategic reason, stated in full: Fabro users run Fabro on their own systems; when Fabro fails for them, the Fabro team wants to help; to help, the team needs the data from those Fabro runs; `fabro.report` exists to make it easy for users to get that data to the team. The product is optimizing for *support effectiveness for self-hosted users* — nothing else. Every design decision on this service must trace back to "did this make it easier for the team to retrieve the Debug Bundle from a self-hosted user who hit a bug?" If the answer is no, the decision is out of scope — even if the decision itself is individually reasonable.

Three things the Strategy is explicitly NOT optimizing for, called out in the confirmed source so they cannot be silently reintroduced:

- **Not a formal support SLA.** `fabro.report` is not a ticketing product, not a commitment device, not a vehicle for response-time guarantees. It makes bundle retrieval easy; it does not manufacture obligations around the retrieval.
- **Not compliance posture.** `fabro.report` is not an audit artifact, not a regulated-data handling surface, not a GDPR/SOC-2/HIPAA-shaped product. The Reporter has no identity; the service has no account concept; there is nothing to be compliant *about* on the public surface.
- **Not marketing telemetry.** `fabro.report` does not exist to measure Fabro adoption, instrument user behavior, or feed a growth funnel. Reporter identity is deliberately absent; metadata captured at upload exists to help the team fix the specific bug, not to build a dataset about who uses Fabro.

The load-bearing trade: this Strategy optimizes for *the team can retrieve the Debug Bundle* over *the user sees rich diagnostics in-browser*. That single trade is what makes `fabro.report` structurally different from a full crash-report product. A Reporter's in-browser experience is intentionally minimal — a Report Metadata Page with Report ID, upload timestamp, bundle size, and a delete button, and nothing else — because richer in-browser experience is not what Strategy is buying. Strategy is buying *team access to the bundle*. Everything else is a distraction from that.

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
  - [[Agent - Reporter]] — the role this Strategy serves (self-hosted Fabro user handing off a Debug Bundle); the Reporter's narrow post-upload surface is the Strategy's concrete expression on the Reporter side of the pickup.
  - [[Agent - Triager]] — the role this Strategy exists for (Fabro team member retrieving bundles); the Triager's no-service-surface shape — retrieving directly from S3 — is the Strategy's concrete expression on the team side of the pickup.
  - [[Standard - Competitive Context]] — the one-pager closes the competitive-analysis question because no competitive-differentiation work is needed when this Strategy is the entire purpose; audiences whose intake shapes differ from self-hosted-Fabro-user retrieval are structurally outside this Strategy's scope.

## WHY: Belief

The Strategic WHY was captured verbatim on 2026-04-22 from the product owner in the Confirmed Product Decisions section of the source, and it is short enough to quote in full: *Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs. `fabro.report` exists to make it easy for users to get that data to the team.* That is the entire Strategy. It is not a summary; it is the thing itself.

A Principle shorter than one paragraph tempts an agent to elaborate it. Do not. Every elaboration that introduces an adjacent purpose — "also for marketing," "also for support SLA," "also for compliance" — erodes the Principle's ability to rule features out. The source names those three adjacent purposes explicitly as things the Strategy is *not* optimizing for, and that negative framing is the other half of the Principle. An agent proposing a feature should be able to cite this Principle against the feature, not around it.

The reason this Principle is Foundation-tier rather than Amplifier-tier is that every downstream design choice on this service — what to display on the Report Metadata Page, what metadata to capture at upload, whether to add any Reporter-facing endpoint, whether to integrate with any third-party tool — reduces to a trade between "team can retrieve the bundle more easily" and "Reporter sees a richer surface." The Strategy ranks the first strictly above the second. Without that ranking stated upstream, each individual trade looks like a toss-up and the product drifts into a fuller shape through many small reasonable choices. With the ranking stated, the choices become mechanical.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner, captured in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L78). Stable. Revisions to this Principle require product-owner approval; they are not implementation details. A scope expansion proposal that implicitly revises this Principle — e.g., "let's add telemetry so we know which Fabro versions are hitting bugs most" — should be surfaced as a Strategy change, not absorbed as a feature.

## HOW: Application

### What Following This Principle Looks Like

- The `fabro report` CLI and the service together make it as frictionless as possible for a self-hosted Fabro user who hit a bug to get their Debug Bundle to the Fabro team. The upload path is short; the URL is easy to share; the team can retrieve the bundle from S3 with their own AWS credentials. Each of these is justified by "team can retrieve the bundle."
- The Report Metadata Page shows only what the Reporter needs to confirm the upload happened and to be able to delete it if they want. It does not try to be useful beyond that because usefulness to the Reporter beyond upload confirmation is not what Strategy is buying.
- Metadata captured at upload (Fabro version, optional reporter message, arbitrary key/value tags) is exactly the metadata the team needs to triage which bundle to look at first. It is not a telemetry feed and it is not an identity system.
- When the team wants a new capability, the first question is "does this help a team member retrieve a bundle?" If the new capability is consumed by the Reporter after upload, it is probably not what Strategy is buying and should be scrutinized.

### What Violating This Principle Looks Like

- Adding analytics that measure how many Reporters complete uploads per week. (This measures product adoption; it does not help the team retrieve a specific bundle.)
- Adding a "status" indicator on the Report Metadata Page showing whether the team has viewed the bundle. (This introduces a support-SLA shape; Strategy is explicit that SLA is not what is being optimized for.)
- Adding a privacy dashboard or audit log for Reporters. (This manufactures compliance posture; Strategy is explicit that compliance is not what is being optimized for.)
- Extending the product toward hosted-Fabro users before the hosted-Fabro question has been reopened as a Strategy question. (The audience is self-hosted, primary and only; extending audience is a Strategy change, not an implementation change.)
- Building a Reporter-facing diagnostic view because "the user might want to see what they sent." (This is the paradigmatic drift — trading team-retrieval investment against Reporter-facing richness, in the exact direction Strategy rejects.)

### The Strategy Test

Before building or proposing any feature, ask two questions in order:

1. **Does this help the Fabro team retrieve a Debug Bundle from a self-hosted Fabro user?** If yes, it may be in scope — check [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Narrow Forever]]. If no, stop.
2. **If this feature exists, is its audience a Reporter *after* upload, or the Fabro team *retrieving* a bundle?** If Reporter-after-upload, this Strategy does not fund it; the feature is trading the wrong direction and should be reconsidered. If Fabro-team-retrieving, it is a candidate for scope (still subject to Narrow Forever and the Product Thesis).

Any feature that fails either test is not a Strategy-funded feature. It may still be a good idea; it belongs in a different product or in Fabro's internal tooling, not here.

## Sources

- `docs/alexandria/sources/product-vision.md` (L78 Strategic WHY; L52–64 Users section; L12–L14 Vision Summary; L115–128 Raven's Inferences and Open Questions — self-hosted-audience scope)
- `docs/alexandria/inventory/card-manifest.md` (F5 entry)

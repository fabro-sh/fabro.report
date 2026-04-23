# Decision - Sentry for Service Errors, Not Bundle Processing

## WHAT: The Choice

The `fabro.report` service sends its **own server-side errors** — panics, upload-coordination failures, unexpected conditions, Feedback-intake-endpoint failures, Discord-webhook failures (beyond the self-reporting retry path), anything that would otherwise be a silent server-side bug — to **Sentry**. This is operational monitoring for the service's own code path, and it spans both intake flows (Debug Bundle and Feedback) because both intake flows run in the same Rust service on the same Fargate task. Sentry is **not** used as a downstream processor for the **contents** of Debug Bundles uploaded by Reporters, nor for the **contents** of Feedback submitted by Feedback Submitters; Debug Bundles are opaque tarballs to this service, Feedback payloads (feedback.txt + feedback.json + attachments) are the Feedback Submitter's voice captured for the Fabro team, and nothing inside either payload is parsed, forwarded, or handed to Sentry in any form. The two uses of Sentry — monitoring *our* code (allowed) and forwarding *user* payloads from either intake type (forbidden) — are structurally distinct; this Decision is the place that distinction is drawn and written down so the distinction does not dissolve under later re-reads.

The load-bearing phrasing is *Sentry-for-service-errors is yes; Sentry-as-bundle-or-feedback-processor is no, permanently*. An agent reading [[Principle - Reject Like bun.report Drift]] cold — which lists "Sentry integration" in its drift rejection — would conclude that Sentry has no place in this product at all. That conclusion is wrong for the operational side and correct for the payload-processing side, for both intake types. This Decision exists to prevent a Factory-mode agent from either deleting the service's operational monitoring (because "F3 says no Sentry") or reintroducing bundle-as-crash-payload forwarding or feedback-as-voice-forwarding (because "A13 allows Sentry"). All three failures are precluded by the two uses living on opposite sides of a named distinction, and the distinction applies equally across both intake types: the service's own errors are Sentry-targets regardless of which flow produced them; neither flow's payload content is ever a Sentry-target. This Decision's scope was extended 2026-04-23 from "bundle processing" to "bundle or feedback processing" when the product expanded to two intake types; the underlying commitment is unchanged in substance.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — Strategy optimizes for the team retrieving Debug Bundles from self-hosted Reporters. Service-side reliability is load-bearing for retrieval: a service that silently loses upload coordinations fails Strategy. Sentry-for-service-errors is the observability mechanism that preserves Strategy's reliability premise. Sentry-as-bundle-processor would serve a Reporter-facing-richness goal that Strategy explicitly rejects, which is why that use is forbidden.
  - [[Principle - Narrow Forever]] — Sentry-for-service-errors does **not** expand the product's public surface. It is invisible to Reporters, invisible on the Report Metadata Page, invisible in CLI output. No new endpoint, no new vocabulary, no new URL shape. It lives entirely in the service's operational monitoring layer. This is the structural test that distinguishes operational monitoring (inside this Principle's allowances) from scope expansion (outside them).
- Paired with:
  - [[Decision - Service Stack]] — the infrastructure-layer Decision that lives at the same operational layer as this one. A12 chooses the stack; this card chooses the service's own-error observability posture. The two Decisions answer complementary questions about the service's operational posture and are intended to be read together as a pair.
  - [[Principle - Reject Like bun.report Drift]] — the paired rejection. F3's "Sentry integration" drift entry rejects Sentry-as-bundle-processor (the `bun.report` pattern where crash payloads from Reporters are parsed and forwarded to Sentry for structured processing). This Decision is the positive choice that names the *other* use of Sentry — service-side operational error tracking — so the F3 rejection does not accidentally generalize beyond its actual target. The two cards together close both escape hatches: F3 forbids bundle-as-crash-payload forwarding; A13 permits server-panic monitoring and forbids anything beyond that.
- Related:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point's operational monitoring is about whether the pickup itself is functioning (did the upload land? did the metadata persist? did the delete succeed?). It is not about analyzing what was dropped off. Sentry-for-service-errors answers the first kind of question; Sentry-as-bundle-processor would answer the second kind, which this Decision refuses.
- Referenced by:
  - [[Agent - Reporter]] — Reporters never see Sentry. No Sentry telemetry captures Reporter identity (Reporters have no identity — [[Decision - Capability URLs over Auth]]); no Sentry event is rendered to the Reporter; the CLI output register ([[Principle - Minimal Public Surface Feels Calm]]) is not affected by this Decision. The Reporter-facing surface is unchanged.
  - [[Standard - Noun Vocabulary]] — "Debug Bundle" remains an opaque tarball under this Decision. Nothing in this Decision parses, inspects, or describes bundle contents. The Noun Vocabulary is unaffected; no new user-facing term is introduced.

## WHY: Rationale

### Options Considered

- **Option A (rejected): No service-side error tracking at all.** Rejected because unnoticed server errors are the exact class of failure Sentry prevents. The service's reliability premise — bundles upload successfully, URLs resolve, deletes take effect — is load-bearing for [[Principle - Strategy Support Effectiveness for Self-Hosted Users]]. An undetected panic in the upload-coordination path breaks the Reporter's handoff without anyone on the Fabro team knowing; a 30-day-retention product gives a narrow window to catch operational bugs before they matter, and a best-effort "check the logs sometimes" posture is not sufficient coverage for that window. The cost of accepting Sentry-for-service-errors is small (one SaaS dependency, one DSN in Secrets Manager, one crate in the Rust service); the cost of not accepting it is invisible bugs on a narrow-window product.
- **Option B (rejected): Sentry-as-downstream-bundle-processor — parse Debug Bundles, extract crash data, forward to Sentry for structured display.** This is the `bun.report` pattern, and it is explicitly forbidden by [[Principle - Reject Like bun.report Drift]]. Rejected here for the record, not because the prohibition was in doubt but because naming the rejection in the same card that permits the operational use is what keeps the two uses distinct. Parsing bundle contents is the single largest drift vector this product has; the drift is paradigmatic and the rejection is categorical.
- **Option C (rejected): CloudWatch-only, no Sentry.** CloudWatch receives Fargate's stdout/stderr by default; the service can log errors there and the team can build dashboards or alarms against the log stream. Rejected as *insufficient*: CloudWatch is a good log store but a poor error-triage surface. Panics with stack traces, error-rate spikes, and per-error aggregation are the features Sentry offers natively and CloudWatch-plus-log-scraping approximates poorly. The team's operational effectiveness benefits from a purpose-built tool for the purpose-built job; CloudWatch remains in the operational stack for logs (per [[Decision - Service Stack]]) and Sentry joins it for errors.
- **Option D (chosen): Sentry for the service's own server-side errors only. Debug Bundle contents are never sent to Sentry, never parsed, never forwarded.** Chosen because it captures the operational-monitoring benefit (Option A rejected) without introducing the bundle-processing surface (Option B rejected) or the CloudWatch-only limitation (Option C rejected). The distinction between the two uses of Sentry is named explicitly so later agents cannot re-collapse them.

### What Drove the Choice

Two forces drove the choice. First, the service has a narrow operational window (30-day retention — [[Decision - 30-Day Retention Default]]) and a small total-traffic footprint; operational bugs must be caught quickly because the service does not accumulate the kind of history that makes "we'll notice eventually" a valid posture. Sentry is the industry-default error-aggregation tool and meets the team where it already is for operational work. Second, the F3 drift rejection for Sentry is specifically about bundle-as-crash-payload forwarding (the pattern `bun.report` embodies); an agent reading F3 without A13 would either delete service-side monitoring that belongs here or reintroduce bundle forwarding that F3 forbids. Writing this Decision resolves the apparent contradiction by naming the distinction; F3's drift rejection and A13's operational choice are complementary, not conflicting.

The product owner confirmed the distinction explicitly in source on 2026-04-23: F3's Sentry rejection is about the `bun.report` downstream-processor use; service-side error tracking with Sentry is correct and in-scope. This Decision is the card form of that confirmation, written so a Factory-mode agent reading the library from scratch arrives at the same understanding without needing to re-derive it from source.

### What It Costs

- **One SaaS dependency.** Sentry is a third-party service and introduces a vendor relationship. Accepted because the operational benefit — catching server-side bugs in a narrow-window product — is worth one vendor, and Sentry's API surface is mature and well-supported. The DSN lives in Secrets Manager or SSM Parameter Store (per [[Decision - Service Stack]]) so rotation is operationally normal.
- **One Rust crate in the service.** The Sentry integration is a crate dependency, which adds a small amount of compile time and a small amount of binary size. Trivially accepted.
- **Operational discipline on what *not* to send.** This Decision binds the service to send its own errors and nothing else. A reviewer (or automated check) needs to verify that Debug Bundle contents never end up in a Sentry event — even accidentally, even in an error message that quotes part of a bundle payload. The cost is named rather than eliminated; [[Principle - Reject Like bun.report Drift]]'s F3 drift list is the escalation path if that discipline slips.
- **Does not expand the public surface, which is also a cost.** Sentry events never surface to Reporters. A Reporter whose upload failed sees the error-output register from [[Standard - Interaction Patterns (CLI + Delete)]] and [[Principle - Minimal Public Surface Feels Calm]] — not a Sentry event ID, not a "please report this crash" link, not any Sentry-shaped artifact. The separation between operational observability and the Reporter-facing surface is a deliberate narrowness, not an oversight.

### Scope of Applicability

This Decision governs what the service sends to Sentry. It does **not** govern what CloudWatch receives from Fargate (stdout/stderr logs per the default Fargate pattern — owned by [[Decision - Service Stack]] and [[System - Service Topology]]). It does **not** govern metrics (upload count, storage size, delete count — deferred per source, decide when/if needed). It does **not** govern tracing. It does **not** govern any Reporter-facing surface — nothing Sentry captures flows to Reporters. It does **not** override [[Principle - Reject Like bun.report Drift]]'s prohibition on bundle-as-crash-payload forwarding; it clarifies that prohibition's scope rather than weakening it.

### Conditions That Would Trigger Revisiting

This Decision is stable, but the specific conditions that would put it back on the product owner's desk:

1. **Sentry's own operational posture changing in a way the team cannot accept** — vendor policy shift, privacy model change, reliability degradation. Would trigger evaluating a replacement (e.g., self-hosted error aggregator) while preserving the "service-side only, never bundle contents" distinction this Decision establishes.
2. **A Reporter-facing demand for richer in-product error context** that would require surfacing Sentry information in CLI or web output. Under this Decision, that demand is Reporter-facing surface expansion and gets routed through [[Principle - Narrow Forever]] as a scope-change conversation, not absorbed here as an observability tweak.
3. **A pattern of operational bugs in the bundle-handling path** that cannot be caught by service-side error tracking alone. Would trigger a conversation about bundle-validation monitoring (still not bundle-content forwarding), which is a different observability question than this Decision answers.

None of those conditions exist today. This Decision is stable.

## WHEN: Timeline

Confirmed 2026-04-23 by the product owner in the Observability section of `docs/alexandria/sources/tech-stack-and-deployment.md` (L24–L29, including the load-bearing distinction against F3 at L27). Written into this card as part of the 2026-04-23 Amplifier-tier follow-on batch. The paired inline note in [[Principle - Reject Like bun.report Drift]] is drafted in the same batch; together they close the apparent-contradiction gap an agent reading F3 cold would otherwise hit. Stable by its own terms.

## HOW: Application

### Following This Decision

- The Rust service initializes Sentry at startup using a DSN supplied from Secrets Manager or SSM Parameter Store (per [[Decision - Service Stack]]'s secret-injection pattern).
- Panics in the service's code path are captured by Sentry automatically (via the Sentry Rust integration's panic handler). Explicit `capture_error`-shaped calls in error paths the service handles without panicking are acceptable; they must not include Debug Bundle contents.
- Sentry events include service metadata (service name, version, deployment environment) and error context (request path, stack trace, error type). They do **not** include Debug Bundle bytes, Debug Bundle contents-derived values, or Reporter-supplied message text beyond what is necessary to diagnose the service-side failure.
- The Sentry dashboard is a Fabro-team operational surface. It is not linked from `fabro.report`'s public surface; it is not referenced in CLI output; it is not mentioned on the Report Metadata Page.
- CloudWatch continues to receive stdout/stderr logs from Fargate (per default Fargate behavior). Sentry and CloudWatch are complementary, not redundant; logs go to CloudWatch, error aggregation to Sentry.

### Violating This Decision

- **Wrong:** Parsing Debug Bundle contents to extract crash data and forwarding that data to Sentry. This is the `bun.report` pattern and is forbidden by [[Principle - Reject Like bun.report Drift]]. A13 clarifies the scope of the operational use; it does not weaken F3's drift rejection.
- **Wrong:** Including Debug Bundle bytes, or content derived from them, in Sentry error events. The bundle is opaque to the service; an error that references bundle content is a service that stopped treating the bundle as opaque.
- **Wrong:** Surfacing Sentry event IDs, links, or "please report this crash" prompts to Reporters. Reporters never see Sentry. The CLI error register and the Report Metadata Page error register are the Reporter-facing surface, and they are governed by [[Principle - Minimal Public Surface Feels Calm]] and [[Standard - Interaction Patterns (CLI + Delete)]] — not by Sentry's UX conventions.
- **Wrong:** Capturing Reporter identity in Sentry context (IP addresses, user agents tied to a specific Reporter, any identifier that links events to Reporters across uploads). Reporters have no identity on this service ([[Decision - Capability URLs over Auth]]); Sentry context must respect that.
- **Wrong:** Using Sentry as a log aggregator ("forward everything, including routine INFO lines"). Sentry is for errors; routine logs go to CloudWatch. Blurring the two overwhelms the Sentry surface and loses the error-aggregation benefit.

### Test

Before adding any Sentry call, event, context field, or integration to the service, ask three questions in order. First: "Is this event about the *service's own code* going wrong — a panic, an unhandled error, a broken assumption in the upload-coordination path?" If yes, it belongs in Sentry. If the event is about *bundle contents* in any form, it is bundle-processing and is forbidden. Second: "Does this event expose anything the Reporter's surface does not already expose?" Sentry must not carry Reporter identity, Reporter-supplied free-text beyond diagnostic necessity, or bundle contents. Third: "Is Sentry replacing or augmenting the Reporter-facing error-output register?" If yes, the answer is no — Reporter-facing errors follow [[Principle - Minimal Public Surface Feels Calm]] and [[Standard - Interaction Patterns (CLI + Delete)]]; Sentry is invisible to Reporters by design.

## Sources

- `docs/alexandria/sources/tech-stack-and-deployment.md` (L24–L29 Observability section — authoritative for Sentry-as-service-error-tracker confirmation and the load-bearing distinction against F3's Sentry-as-bundle-processor rejection; confirmed 2026-04-23)
- `docs/alexandria/library/rationale/principles/Principle - Reject Like bun.report Drift.md` (F3 Drift Examples — the Sentry drift entry whose scope this Decision clarifies, not weakens)
- `docs/alexandria/inventory/card-manifest.md` (A13 entry)

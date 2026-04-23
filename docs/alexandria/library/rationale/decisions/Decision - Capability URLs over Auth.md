# Decision - Capability URLs over Auth

## WHAT: The Choice

The entire public-surface access control for `fabro.report` is **possession of the Report's URL**. Possession grants the ability to view the Report Metadata Page and the ability to delete the Report; there is no second factor, no login, no account, no session, no token exchange beyond the URL itself. The URL *is* the capability. This Decision records that capability-URL security was deliberately chosen over any authentication or authorization model, explains what was considered and rejected, and names what the choice costs.

The load-bearing phrasing is *the URL is the whole access-control mechanism*. An agent reading this card should not come away believing "capability URLs are the v1 access model, and we'll add auth later when it matters." Auth will not be added later; auth is permanently out of scope under [[Principle - Narrow Forever]]. Capability URLs are forever.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — Strategy optimizes for the team retrieving a Debug Bundle from a self-hosted Reporter. Auth on the public surface would add friction to every step of the pickup (account creation, session management, credential storage) without improving the team's ability to retrieve from S3. This Decision is the concrete access-model expression of that Strategy.
  - [[Principle - Narrow Forever]] — authentication, accounts, sessions, and per-user scoping are on the permanently-out-of-scope list F6 enumerates. This Decision is the positive choice that makes that rejection coherent: if we are not adding auth, we need an access model, and this is it.
- Paired with:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point has exactly two handoffs: the Reporter drops the Debug Bundle and receives the URL; the Triager picks up the bundle from S3 using their own AWS credentials. The capability URL is the token of the first handoff. The S3 credentials are the token of the second. Neither handoff involves authentication on the `fabro.report` service, which is what makes the pickup-point thesis structurally coherent.
  - [[Principle - Reject Like bun.report Drift]] — proposals to add per-team auth, user accounts, or "log in to see your uploads" surfaces are named drift shapes in F3. This Decision is the upstream access-model commitment those proposals have to overturn — and they cannot, because auth is out-of-scope by [[Principle - Narrow Forever]].
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the **Reporter** definition explicitly states "no authentication, no login, no account." The **Report Metadata Page** definition explicitly states "capability-URL addressed (possession of URL grants view + delete)." This Decision is the rationale behind both of those vocabulary choices.
- Related:
  - [[Decision - 30-Day Retention Default]] — capability URLs depend on the URL eventually expiring. A URL that grants view + delete forever is a larger security exposure than one that expires after 30 days; the retention default is what bounds the capability-URL surface's lifetime risk. The two Decisions reinforce each other.
- Referenced by:
  - [[Decision - No Content Dedup]] — that Decision cites the absence of Reporter identity (established here) as the reason per-Reporter dedup is not an option; no identity means no per-Reporter scoping for any purpose.
  - [[Agent - Reporter]] — the capability URL returned by `fabro dump --upload` (or `fabro upload`) is the Reporter's entire access surface to the service after upload; there is no login, no "my uploads" view, no recovery flow — the Reporter role as written is the role-layer expression of this Decision.
  - [[Agent - Triager]] — Triagers do not "unlock" the Debug Bundle through the URL; they use AWS IAM for S3 retrieval and URL possession only to optionally delete a Report on the Reporter's request. The Triager role's no-service-auth shape is the team-side expression of this Decision.
  - [[Component - Report]] — the Report's ID and URL are the capability-bearing values; that Component inherits this Decision for its access semantics and carries the URL-is-capability property as a structural field.
  - [[Component - Report Metadata Page]] — the rendered surface addressed by the capability URL; implements view + delete on URL possession alone, with no second factor.
  - [[Standard - Competitive Context]] — enumerates per-user accounts / per-team scoping as features explicitly not borrowed from `bun.report`; this Decision is the upstream access-model commitment that closes that borrowing vector.
  - [[Standard - Information Architecture]] — the IA Standard names URL possession as coextensive with the product's access model and routable surface; this Decision is the access-model half of that IA commitment.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the delete button on the Report Metadata Page authorizes solely on URL possession with no additional credential check; A7's client-side-confirmation flow is the interaction-layer expression of this Decision.
  - [[Standard - Accessibility Policy]] — the delete-confirmation accessibility requirement is load-bearing precisely because the delete action is the one Reporter-initiated capability on the service after upload under this Decision; if the confirmation is inaccessible, the capability is effectively inaccessible.
- Implemented by:
  - [[System - Retention, Storage, and Key Scheme]] — the key scheme produces URLs whose capability is non-guessable (32 hex characters = 128 bits of entropy, drawn from a CSPRNG); the non-guessability property this Decision declares load-bearing is enforced structurally by that System.
  - [[System - Service Topology]] — the service-side half of the access model (Fargate task mints presigned S3 URLs via its IAM role) and the Triager-side half (Triagers retrieve from S3 with their own AWS credentials, a separate IAM principal) both live in that topology. This Decision names the access model; A14 names the operational pieces that implement it.
- Stack dependency:
  - [[Decision - Service Stack]] — the capability-URL model depends on AWS-native IAM for the service-side presigned-URL mint and Triager-side S3 retrieval. A12's AWS-native posture is what makes this Decision's access model operationally coherent without cross-cloud credential management.
- Observability boundary:
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — Sentry context respects the no-identity shape this Decision establishes. Reporter identity is never captured in Sentry events (no IP tied to a Reporter, no user-agent identifier linked across uploads); the operational-monitoring path preserves the access model's identity posture.

## WHY: Rationale

### Options Considered

- **Option A (rejected): Authentication for Reporters — login required to upload, view, or delete a Report.** Rejected because it imposes an account-creation step on a Reporter who has already hit a Fabro bug and now has to create a `fabro.report` account before they can get their Debug Bundle to the Fabro team. That is friction in the exact place [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] is optimizing to remove. Every friction beat added to the Reporter's path is a beat where the pickup might not happen.
- **Option B (rejected): Authentication for Triagers — team-facing login that lets Fabro team members browse and delete Reports through the service.** Rejected because Triagers do not use the service. [[Standard - Noun Vocabulary]] defines the Triager as someone who retrieves Debug Bundles directly from S3 using their own AWS credentials; giving Triagers a second login on `fabro.report` would both duplicate the S3 credential system and manufacture a team-facing surface this product explicitly refuses to have (see [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Reject Like bun.report Drift]] on team dashboards).
- **Option C (rejected): Hybrid — public capability URLs for view, but a deletion password or email-verification flow.** Rejected because it would require capturing a Reporter-side secret at upload time, which re-introduces an identity-shaped concept ("your password," "your email") on a product that deliberately has no Reporter identity. It also creates a new recovery-flow surface area (password-reset, email verification) that is itself a scope expansion. Half-authentication is worse than no authentication for this product's shape; the capability URL is either the whole access story or it is a v1 sketch of something more elaborate, and [[Principle - Narrow Forever]] rules out the second reading.
- **Option D (rejected): Signed URLs with short expirations plus a refresh mechanism.** Rejected because refresh implies a session concept, session implies identity, and identity is what this product does not have. Short-lived signed URLs are the correct choice inside a larger service with an auth system; they are overkill for a product whose entire lifecycle boundary is "30-day retention."
- **Option E (chosen): Capability URLs only. Possession of the URL grants view + delete. No second factor. The URL is the access model.** Chosen because it matches the product shape end-to-end: the Reporter receives the URL at upload time and shares it out-of-band wherever makes sense (GitHub issue, Slack, email); the Fabro team can delete a Report if a Reporter shares the URL with them and asks; the URL becomes unreachable when the Report expires after 30 days. Access control collapses into URL possession, which collapses into "share the URL with whoever should have access." That is the entire model, and it matches the narrow product.

### What Drove the Choice

The Strategic WHY (confirmed 2026-04-22) names *support effectiveness for self-hosted users* as the single optimization target — specifically, making it easy for Reporters to get Debug Bundles to the Fabro team. Authentication anywhere on the Reporter path imposes friction in the direction Strategy is removing friction from; the correct access model is the one that imposes no additional steps on the pickup.

A Triager-retrieves-from-S3-directly model (also confirmed in the source) means `fabro.report` never needs to know which Triager is looking at what; the service has no trust boundary to enforce on the team side because the team side does not use the service. That structural fact — there is no team-facing surface to protect — collapses the access-control problem to "control access to the Reporter-facing view and delete, nothing else," which is exactly what a capability URL does.

The "narrow forever" commitment (F6) makes explicit that auth, accounts, and per-user scoping are permanently out of scope. Any access model that would be a v1 step toward auth fails that commitment; only a model that can stand as the permanent access model passes it. Capability URLs pass. Anything involving Reporter identity does not.

### What It Costs

- **URL leak equals access.** If a Reporter pastes the URL into a public GitHub issue and later wishes they hadn't, anyone who saw the URL has view + delete capability until the Report expires or is deleted. The mitigation is the 30-day retention ([[Decision - 30-Day Retention Default]]) and the delete button on the Report Metadata Page. The cost is accepted because the alternative — preventing URL leaks by adding auth — costs more than URL leaks do in practice for this product's traffic profile.
- **No "list my uploads" view for Reporters.** A Reporter who uploaded a Debug Bundle yesterday and lost the URL has no way to recover it through this service. The service has no identity to attach uploads to, by design. The correct remedy is to upload again. The cost is accepted because building a "my uploads" view would require Reporter identity, which is out of scope by [[Principle - Narrow Forever]].
- **Anyone with the URL can delete.** This is framed as a feature (the Reporter can delete their own Report; a Fabro Triager the Reporter shares the URL with can also delete if asked) and not a bug. Nothing in the service attempts to distinguish "original Reporter" from "anyone in possession of the URL," because there is no way to distinguish them without identity, and there is no identity. The cost is named rather than mitigated.
- **Non-guessable URLs are load-bearing.** If an agent ever generates URLs with insufficient entropy (predictable IDs, short slugs, sequential numbers), the capability model collapses into no access control at all. The key scheme is part of the security model; it must produce URLs that cannot be reasonably brute-forced. That constraint is enforced by [[System - Retention, Storage, and Key Scheme]] (32 hex characters = 128 bits of entropy from a CSPRNG).

### Scope of Applicability

This Decision governs access control on the `fabro.report` service's public surface — CLI upload, Report Metadata Page view, Report Metadata Page delete. It does **not** govern access to the underlying S3 bucket where Debug Bundles are stored; that is an AWS-IAM concern for the Fabro team's infrastructure, and Triagers access S3 with their own AWS credentials by design. It also does **not** govern access to the parent Fabro product; Fabro's own auth model (if any) is out of scope here.

### Conditions That Would Trigger Revisiting

Because auth is on [[Principle - Narrow Forever]]'s permanently-out-of-scope list, revisiting this Decision requires reopening that Principle as a Strategy change, not treating this as an implementation detail. Specifically:

1. The product owner explicitly decides to add authentication to the public surface, in writing, in a source-material update. This is an identity change for the product under the framing of [[Principle - Narrow Forever]] — not an implementation tweak.
2. A pattern of capability-URL compromise surfaces that cannot be addressed by tightening key entropy or shortening retention, and the product owner decides the correct remedy is an identity layer. This would be an invalidation signal for this Decision; today, no such pattern exists.
3. The audience widens to populations for whom capability URLs are insufficient (e.g., if [[Decision - Who This Service Is For]] is reopened to include hosted-Fabro users with an existing identity system). This is upstream-shaped: if the audience changes, the access model may need to change to match.

None of those conditions exist today. This Decision is stable.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L87 "Auth: none on the public surface. Capability URLs only."). Further supported by the Security Model section of the same source (L68–L72). Written into this card on the same date as part of the Amplifier-tier Decisions Log. Stable by its own terms and structurally load-bearing: the capability-URL choice is part of the product's identity, not part of its implementation detail.

## HOW: Application

### Following This Decision

- The `fabro dump --upload` (or `fabro upload`) CLI uploads a Debug Bundle and returns a short URL. The URL contains enough entropy that guessing it is computationally infeasible. No login step; no credential capture; the Reporter shares the URL out-of-band however they choose.
- The Report Metadata Page at that URL renders on any request to the URL, regardless of who is requesting. There is no session cookie, no CSRF token tied to a login, no access-check step beyond "can the request reach this URL." The delete button on that page submits an action that requires no additional authentication — possession of the URL is the whole authorization.
- A Triager who needs to investigate a Debug Bundle copies the Report ID out of the shared URL and retrieves the bundle from S3 using their own AWS credentials. The service plays no role in that path. The service's S3 bucket is configured so that direct S3 access requires the team's AWS identity, which is orthogonal to the capability URL.
- The key scheme (see [[System - Retention, Storage, and Key Scheme]]) produces IDs and URLs with sufficient entropy that brute-forcing valid URLs is not a practical attack vector. That entropy requirement is part of this Decision's enforceability.

### Violating This Decision

- **Wrong:** Adding a login page to the service, even "just for Reporters who want to manage their uploads." This introduces Reporter identity, which is out of scope, and builds the exact account surface this Decision rejects.
- **Wrong:** Adding a password, PIN, or confirmation code to the delete action. The capability URL is the whole access story; a deletion secret implies a Reporter-side identity concept that the product does not have.
- **Wrong:** Using sequential Report IDs, short slugs, or low-entropy identifiers in URLs. This makes URLs guessable, which collapses the capability-URL model into effectively open access. Non-guessable URLs are load-bearing.
- **Wrong:** Building a "list my uploads" endpoint, even without authentication — e.g., using some local client-side cache of URLs the Reporter has uploaded. This introduces a Reporter-side state concept the service should not have, and it creates a new capability surface (listing) the service was designed to avoid.
- **Wrong:** Adding an "email me a copy of the URL" feature after upload. Email capture is identity capture; identity capture is out of scope under [[Principle - Narrow Forever]].

### Test

Before adding any access-check, login flow, permission layer, or Reporter-identifying mechanism to `fabro.report`, ask: "Does this add a second factor beyond URL possession?" If yes, this Decision rejects it — not because access control is unimportant but because URL possession is the access model. Proposals to add any other access-control layer are scope changes and go to the product owner as such.

A second test: "Does this feature assume the service knows *who* is making a request?" If yes, the service does not know, by design. Any feature that assumes a persistent Reporter or Triager identity on the service surface is resting on a foundation this Decision refuses.

## Sources

- `docs/alexandria/sources/product-vision.md` (L68–L72 Security Model section; L87 Confirmed Product Decision "Auth: none on the public surface. Capability URLs only.")
- `docs/alexandria/inventory/card-manifest.md` (A9 entry)

# Decision - 30-Day Retention Default

## WHAT: The Choice

Every Report on `fabro.report` expires **30 days after upload**. At expiration, the Debug Bundle is deleted from S3, the Report is removed from the service's active set, and any future request against the Report's URL resolves as not-found. Thirty days is the **default retention window**, and on this product it is also the **only retention window**: Reporters cannot extend retention, shorten retention, or opt into indefinite retention. The only party who can alter the retention policy — as a policy, not per-Report — is the product owner, and only through an explicit decision recorded in source material.

The load-bearing phrasing is *30 days, same for every Report, Reporter-immutable*. An agent reading this card should not come away believing "30 days is a sensible default and Reporters can adjust it per-upload." Per-Report retention overrides, retention tiers, and indefinite retention are all permanently out of scope under [[Principle - Narrow Forever]]. The product has one retention number, and this card names it.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — Strategy says the product exists so the Fabro team can retrieve Debug Bundles from self-hosted Reporters. Thirty days is a retention window calibrated to that retrieval shape: long enough to cover the realistic triage window for a self-hosted-support workflow (report filed, team sees the issue, team retrieves the bundle, team investigates), short enough that the service is not holding potentially-sensitive bundle data indefinitely.
  - [[Principle - Narrow Forever]] — indefinite retention, per-Report overrides, and retention tiers are on F6's permanently-out-of-scope list. Retention is a single number by commitment, not a parameter surface. This Decision is the positive choice that makes those rejections concrete: not "Reporters can't choose retention in v1," but "Reporters do not choose retention, ever."
- Paired with:
  - [[Decision - Capability URLs over Auth]] — capability URLs grant view + delete to anyone in possession. A URL that grants access forever is a larger lifetime-risk surface than one that becomes unreachable after 30 days. Retention is what bounds the capability-URL exposure window; without retention, the access-model cost profile would be different.
  - [[Decision - No Content Dedup]] — the Rationale for no dedup is *bundles expire anyway*, which means that Decision depends on this one. If retention were, say, 10 years, the dedup math would change and A10 would need to be revisited. The two Decisions are coupled.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the **Report** canonical definition names "retention deadline (default 30 days)" as an authoritative field. This Decision is the rationale behind that field's default value and the reason the default is also, in practice, the only value.
- Related:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point is a temporary handoff location, not a permanent repository. Retention is structural to the thesis: a pickup that never expires becomes an archive, and an archive is a different product category entirely.
  - [[Principle - Reject Like bun.report Drift]] — proposals for retention flexibility often arrive disguised ("Triagers need more time for some bundles," "let Reporters pin important bundles"). F3 names retention-extension as a drift shape; this Decision is the upstream commitment F3 rejects against.
- Referenced by:
  - [[Component - Report]] — the Report carries a retention deadline computed as upload timestamp plus 30 days; the Component card inherits this Decision for the deadline's default value and the Reporter-immutable commitment.
  - [[Component - Report Metadata Page]] — during the retention window the page renders on request; at the deadline the Report and its Debug Bundle are deleted and the URL resolves as not-found, with no rendered expired state.
  - [[Standard - Information Architecture]] — A6 captures the "expired Report returns not-found" shape as an IA-layer commitment; this Decision is the retention policy that produces the not-found state, and together they refuse a rendered expired / deleted tombstone page.
- Implemented by:
  - [[System - Retention, Storage, and Key Scheme]] — this System is the mechanism that enforces the 30-day rule. Terraform-declared S3 lifecycle rule (`expiration.days = 30`) runs server-side inside S3; no cron, no application-side sweeper.

## WHY: Rationale

### Options Considered

- **Option A (rejected): Shorter retention (e.g., 7 days).** Shorter retention would reduce the service's exposure window, but 7 days is not long enough to cover realistic triage latency for a self-hosted-user support workflow. A Reporter files an issue, the team sees it on Monday, the triage priority lands it mid-week, the bundle is retrieved late week — a 7-day window leaves almost no margin for the bundle to still be available by the time a Triager goes to retrieve it. [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] is optimizing for team retrieval; a retention window that routinely expires before the team retrieves is retention that defeats the Strategy.
- **Option B (rejected): Longer retention (e.g., 90 days or 1 year).** Longer retention covers more edge cases (Triager out for extended PTO, bundle needed for a post-mortem weeks after the fix), but every additional day of retention is an additional day of holding potentially sensitive bundle data — and a Debug Bundle produced by a self-hosted Fabro user may contain workflow data, stack traces, or environment context that the Reporter did not intend to share indefinitely. Thirty days is the longest window where "we're holding this briefly to support you" remains the natural framing; past 30 days the framing drifts toward "we're holding this indefinitely for reasons," which is closer to the compliance/archival posture that [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] explicitly names as *not* what Strategy is optimizing for.
- **Option C (rejected): Retention flexibility — per-Report overrides or retention tiers.** Rejected because per-Report retention adds a parameter surface to the public product (how is it set? at upload time? after upload? only by Triagers?), and retention tiers imply a multi-tier product shape that conflicts with [[Principle - Narrow Forever]]. Flexibility in retention is not a neutral implementation choice; it is a scope expansion. F3 names "extend retention policy per Report (custom expiration windows, indefinite retention tiers)" as a drift shape to reject.
- **Option D (rejected): Indefinite retention — Reports never expire unless deleted.** Rejected because indefinite retention makes `fabro.report` an archive, which is a different product category. It also lets capability-URL leaks compound forever ([[Decision - Capability URLs over Auth]] depends on retention bounding the leak window), and it removes the natural privacy posture that comes from "the service forgets after a month."
- **Option E (chosen): 30 days. Same window for every Report. Reporter-immutable; only the product owner can change the policy, and only for future Reports, through an explicit Strategy-adjacent decision recorded in source material.** Chosen because 30 days covers the realistic triage window for a self-hosted-support workflow, keeps the product's data-holding posture natural ("briefly, while we help"), bounds the capability-URL leak surface, and matches the narrow-forever commitment by being a single number rather than a parameter.

### What Drove the Choice

The source states the decision in one line (`product-vision.md` L81: "Retention: 30-day default expiration.") and reinforces it in the end-to-end flow description (`product-vision.md` L29: "Unless deleted earlier, the report expires after 30 days (default retention)."). The initialize configuration also records 30 days as the retention default (`initialize-output.md` L23). The number itself is a product-owner judgment; this card's role is to name the number, explain what it buys, and close the escape hatches that would otherwise erode it.

The two structural pressures on the retention number are in tension: shorter retention reduces the service's data-holding posture and bounds capability-URL leak risk; longer retention reduces the chance that a Triager retrieves a bundle that has already expired. Thirty days is the product-owner-chosen balance between those pressures. It is not a precise optimum — no measurement would produce "exactly 30 days is correct." It is a defensible default in the right neighborhood, stable enough to anchor downstream cards.

The Reporter-immutable aspect is load-bearing. Letting Reporters pick retention per-upload would turn every upload into a retention decision, which is a beat of cognitive load on a Reporter who is already dealing with a Fabro bug. Letting Triagers pick retention per-Report would turn the service into a Triager-facing surface, which [[Product Thesis - Pickup Point, Not Triage Surface]] refuses. Locking retention to a single number — adjustable only by the product owner as a policy change — keeps the service's public surface minimal and the retention decision out of the hot path.

### What It Costs

- **Some bundles will expire before triage completes.** If a Triager is out, or the Report sits in a triage queue longer than 30 days, the Debug Bundle is gone before the Triager gets to it. The mitigation is the Reporter running `fabro dump --upload` again if asked; the bundle can be regenerated from the same buggy environment. The cost is accepted because the alternative — longer retention — costs more in holding posture than re-uploads cost in Reporter effort.
- **No "important bundle" concept.** Every Report is equal in the retention model; there is no way to mark a Report as important-enough-to-keep-longer. Accepted because marking Reports would require either Reporter identity (for Reporter-side marking) or a Triager-facing surface (for Triager-side marking), both of which are out of scope. Triagers who need a bundle longer than 30 days retrieve it before expiration and store it in Triager-side tooling.
- **Capability-URL leaks are bounded but not eliminated.** If a URL leaks on day 1, it grants view + delete for 30 days. Thirty days is a bound on the exposure window, not zero exposure. Accepted because the alternative (auth) is out of scope by [[Principle - Narrow Forever]], and the Reporter can always delete the Report directly to shorten the exposure.
- **Retention enforcement is a real operational concern.** A 30-day retention policy is only the correct policy if bundles actually expire on schedule. A retention bug that lets bundles live beyond 30 days turns this Decision's promises into a lie. The mechanism side of retention lives in [[System - Retention, Storage, and Key Scheme]]; this card names the policy.

### Scope of Applicability

This Decision governs retention policy for `fabro.report` Reports. It does **not** govern retention in Triager-owned tooling after a Debug Bundle has been retrieved from S3; once a bundle is pulled into Triager-side workflows, the retention clock for this service is no longer the authoritative one. It also does **not** govern retention for the parent Fabro product's data; Fabro has its own retention decisions that are out of scope here. A Triager who wants a bundle beyond 30 days retrieves it and stores it where Triager-side retention applies.

### Conditions That Would Trigger Revisiting

This Decision is stable, but stability is not permanence. Specific conditions that would put it back on the product owner's desk:

1. **Systematic retention-expiration-before-retrieval.** If a recurring pattern emerges where Triagers miss the retention window on non-trivial numbers of Reports, the retention number is misaligned with the real triage workflow and should be revisited. Today, no such pattern exists (no workflow data to measure against yet).
2. **New data-handling pressure on `fabro.report`.** If the product owner decides self-hosted Reporter bundle data carries a privacy or compliance posture that 30 days is too long for, the retention window shortens. This is a Strategy-adjacent decision that goes through the product owner.
3. **Upstream change to [[Decision - No Content Dedup]].** If dedup becomes in scope for any reason, the retention math couples to it and both decisions should be reviewed together. The source's one-line rationale for no dedup is explicitly retention-based; coupling runs both directions.
4. **Audience change under [[Decision - Who This Service Is For]].** If hosted-Fabro users are ever brought into scope, their bundle-retention expectations may differ from self-hosted Reporters' and the retention default could need to split or adjust.

None of those conditions exist today. Revisions require an explicit product-owner decision recorded in source material; agents observing retention-related drift ("let's keep this one longer, just this once") should treat those as proposals routed to the product owner, not as implementation calls.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L81 "Retention: 30-day default expiration."). Reinforced in the end-to-end flow description (L29) and in the initialize configuration (`docs/alexandria/initialize-output.md` L23). Written into this card on the same date as part of the Amplifier-tier Decisions Log. Stable. The 30-day number is a product-owner judgment call; the *Reporter-immutable, single-value* commitment around the number is what this Decision locks in durably.

## HOW: Application

### Following This Decision

- Every Report is created with a retention deadline of upload-time + 30 days. The deadline is recorded as an authoritative field on the Report (see [[Standard - Noun Vocabulary]]) and enforced by the retention mechanism defined in [[System - Retention, Storage, and Key Scheme]].
- When a Report reaches its retention deadline, the Debug Bundle is deleted from S3, and subsequent requests to the Report's URL resolve as not-found. The service surface treats expired Reports the same as never-existed Reports — there is no "expired" state the Reporter Metadata Page renders, because the page is not served for expired Reports.
- Reporters who want a Report gone sooner than 30 days can delete it via the delete button on the Report Metadata Page; [[Decision - Capability URLs over Auth]] is what makes that deletion path available without any authentication step. Deletion brings the retention clock to zero; it does not extend or reset any clock.
- Triagers who need a bundle beyond 30 days retrieve it before expiration and store it in their own tooling. The service does not support post-retrieval retention requests; storage past 30 days is a Triager-side concern.

### Violating This Decision

- **Wrong:** Adding a per-Report retention override (e.g., `fabro dump --upload --retain 90d`). This turns retention into a parameter surface, which is exactly the scope expansion F3 and F6 refuse. The correct posture is one retention number, product-owner-adjustable as a policy, not Reporter-adjustable per upload.
- **Wrong:** Adding a "pin this Report" or "extend retention" mechanism on the Report Metadata Page. This would let URL holders keep bundles alive indefinitely, which defeats the retention-bounded-exposure argument that [[Decision - Capability URLs over Auth]] depends on.
- **Wrong:** Letting the retention deadline silently reset when a Reporter revisits the Report Metadata Page. Access should not extend retention; the deadline is set at upload and counts down.
- **Wrong:** Not actually deleting the Debug Bundle at the deadline. A retention policy that holds bundles past the stated deadline turns this Decision's promises into a lie to Reporters; the enforcement is part of the commitment.
- **Wrong:** Surfacing retention-near-expiration notifications to Reporters that ask them to act ("your Report expires in 3 days — extend?"). The Reporter has no extension capability; a notification that implies one misrepresents the product. If any retention-expiration notification ever exists, its copy inherits [[Principle - Minimal Public Surface Feels Calm]] and does not offer agency the Reporter does not have.

### Test

Before shipping any feature that touches a Report's lifecycle — creation, display, deletion, expiration — ask: "Does every Report in this flow expire exactly 30 days after upload, with no per-Report variation?" If yes, the feature is consistent with this Decision. If the feature introduces any Report whose retention differs from 30 days, for any reason, it is violating this Decision and should be rejected or escalated to the product owner as a policy-change question.

A second test: "Does this feature assume the retention window is adjustable?" If yes, it is designing against a product shape that does not exist. Correct features assume 30 days, period, for every Report — and if the retention number ever changes (by product-owner decision), the feature adjusts uniformly, not selectively.

## Sources

- `docs/alexandria/sources/product-vision.md` (L29 end-to-end flow — "Unless deleted earlier, the report expires after 30 days (default retention)"; L81 Confirmed Product Decision "Retention: 30-day default expiration.")
- `docs/alexandria/initialize-output.md` (L23 retention default in initialize features list)
- `docs/alexandria/inventory/card-manifest.md` (A11 entry)

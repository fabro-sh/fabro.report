# Decision - Who This Service Is For

## WHAT: The Choice

`fabro.report` serves **Reporters who are self-hosted Fabro users — primary and only.** A Reporter in scope is someone running Fabro on their own systems who hits a bug, runs `fabro dump --upload` (or `fabro upload` against a previously-dumped tarball), and uploads a Debug Bundle. Bundles produced by users of any future hosted-Fabro offering are **out of scope under this Decision**; if hosted-Fabro ever needs bundle intake, that is a separate product question, not a scope expansion of `fabro.report`. This Decision formalizes the audience commitment that was confirmed by the product owner on 2026-04-22 as part of the Strategic WHY.

The load-bearing phrasing is *self-hosted, primary and only*. "Self-hosted" names the audience the Strategic WHY optimizes for. "Primary and only" closes the escape hatch that would otherwise read this as "self-hosted is the initial target and we'll add others later," which would turn every hosted-Fabro-shaped feature request into an implementation discussion rather than a Strategy change.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — this Decision is the direct audience formalization of that Strategy; F5 says what the product is optimizing for ("team retrieves Debug Bundles from self-hosted Fabro users"), and this Decision writes down exactly who that audience is so downstream cards do not silently widen it.
  - [[Principle - Narrow Forever]] — audience expansion is a scope change under F6; a hosted-Fabro audience cannot arrive here by way of an implementation decision, only by way of an explicit Strategy change. This Decision locks the audience in the same frame that F6 locks the feature scope in.
- Paired with:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the pickup-point thesis assumes the pickup is happening between a self-hosted Reporter and a Fabro-team Triager retrieving from S3 with their own credentials. Changing the audience would change the two endpoints of the pickup, which would invalidate the thesis at the audience layer rather than at the feature layer.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the canonical **Reporter** definition (no identity, no login, identified only by Fabro version metadata) is written for a self-hosted-Fabro Reporter. A hosted-Fabro user would have a different identity model (they already exist in the hosted product), and silently extending "Reporter" to cover both populations would break this Standard.
- Related:
  - [[Principle - Reject Like bun.report Drift]] — an "audience expansion to hosted-Fabro users" proposal that arrives without flagging itself as a Strategy change is the audience-layer version of the `bun.report` drift pattern; the same rejection posture applies.
- Referenced by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — F5's forward-reference to "`Decision - Who This Service Is For` (manifest C4)" resolves to this card.
  - [[Decision - Capability URLs over Auth]] — the capability-URL access model is calibrated to a self-hosted Reporter with no existing identity; if the audience widened to include hosted-Fabro users, the access model could need to change, coupling that Decision to this one.
  - [[Decision - 30-Day Retention Default]] — the 30-day retention window is calibrated to a self-hosted-support workflow; an audience change under this Decision could require revisiting the retention number.
  - [[Agent - Reporter]] — inherits the self-hosted-only audience from this Decision; the Reporter role as written describes a Fabro user running Fabro on their own systems, not a hosted-Fabro user or a generic "anyone with a Debug Bundle" Reporter.
  - [[Agent - Triager]] — the Triager's retrieval model (own AWS credentials, direct S3 access) is also tied to the self-hosted audience shape; widening the Triager definition (e.g., to third-party support partners, or to hosted-Fabro team members) is an audience change under this Decision.
  - [[Standard - Competitive Context]] — the competitive-analysis one-pager scopes comparisons to the self-hosted-Fabro-user audience named here; adjacent products serving different populations (managed offerings, crash-report SaaS, file-sharing services) are structurally unhelpful comparisons because their optimization targets differ.
  - [[Decision - Service Stack]] — the stack is chosen to fit the self-hosted audience's realistic traffic profile (single-task Fargate, single-region default); an audience change under this Decision is one of the conditions A12 names as triggering a revisit.

## WHY: Rationale

### Options Considered

- **Option A (rejected): Audience includes self-hosted Fabro users *and* any future hosted-Fabro offering's users from day one.** This would avoid needing a second intake product later if hosted-Fabro happens. Rejected because the hosted-Fabro product does not exist yet, its identity model is unknown, and designing `fabro.report`'s intake around a hypothetical second population pollutes every Reporter-shaped decision with "will this also work for the hosted case?" second-guessing. The Strategic WHY as confirmed names "users who run fabro on their own systems," which does not cover hosted users, and that phrasing was deliberate.
- **Option B (rejected): Audience is self-hosted users "for v1," with hosted-Fabro bundle intake explicitly on the Roadmap for later.** Rejected because [[Principle - Narrow Forever]] treats "for v1" framings as the decay path that erodes scope commitments one relaxation at a time. Putting hosted-Fabro on the Roadmap as "later" turns every design discussion into "let's make this flexible enough for hosted users too," which is audience-shaped drift into a fuller product.
- **Option C (chosen): Audience is self-hosted Fabro users, primary and only. Hosted-Fabro bundle intake is a separate product question, deferred — not queued, not scheduled, not on a roadmap here. Reopening the audience requires an explicit Strategy change initiated by the product owner.** Chosen because it matches the load-bearing shape of [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] and [[Principle - Narrow Forever]]: the audience commitment is an identity decision at the same level as the feature-scope commitment, not an implementation detail. Scope changes require product-owner approval; so should audience changes.

### What Drove the Choice

The Strategic WHY was confirmed by the product owner verbatim on 2026-04-22: *Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs.* That phrasing names self-hosted users as the target and names the motivation (team retrieval to make support effective). A hosted-Fabro offering's users — assuming one ever exists — live in a different product with a different identity model, different trust boundaries, and potentially different team-retrieval mechanics. Pulling both populations into one intake product would force a design that is optimal for neither.

The shape of the Reporter role, written into [[Standard - Noun Vocabulary]], is also audience-specific. A self-hosted Reporter has no identity on the service by design; a hosted-Fabro user already has an identity in the hosted product, and a bundle intake that ignored that identity would be throwing away useful team-retrieval context while a bundle intake that used it would be a different product shape entirely. The two audiences want different intake shapes; `fabro.report` commits to one of them.

Raven's Inferences section of the source (`product-vision.md` L115–128) specifically flags the hosted-Fabro question as unresolved at Vision-card time. This Decision resolves it in the direction the Confirmed Strategic WHY already leans — self-hosted only — and records the resolution so it stops appearing as an open question in downstream cards.

### What It Costs

- **Foreclosed future.** If hosted-Fabro ever ships, its bundle intake is not this service. A second intake product would have to be built or a Strategy change would have to be negotiated. That cost is accepted because the alternative — carrying a dual-audience design through every downstream card — is a larger tax paid continuously rather than a one-time decision paid later if and only if hosted-Fabro happens.
- **No audience-growth story.** Product discussions that assume "we'll start narrow and widen" lose that frame for this product. This is correct under [[Principle - Narrow Forever]] but worth naming: the audience is also narrow-forever, not just the feature scope.
- **Friction for well-meaning drift proposals.** An agent noticing that the service could "easily" accept hosted-Fabro bundles too will have to route that noticing through the product owner as a Strategy change. That friction is the point: it keeps the audience commitment from dissolving one convenience at a time.

### Scope of Applicability

This Decision governs the audience of `fabro.report` as an intake service and the audience inherited by every downstream card that names Reporters, Triagers, or the pickup relationship between them. It does **not** govern Fabro itself; the parent product at `~/p/fabro-sh/fabro` has its own audience decisions that are out of scope here. It also does **not** govern what the Fabro team may choose to do internally with bundles once retrieved — that is a Triager-side concern performed in triager-owned tooling, not on this service.

### Conditions That Would Trigger Revisiting

This Decision is not locked forever — but the lock is lifted only by a deliberate Strategy change initiated by the product owner, not by an implementation discussion. Specifically:

1. The product owner explicitly reopens the audience as a Strategy question, in writing, in a source-material update to `docs/alexandria/sources/product-vision.md`. (An informal mention in passing does not count.)
2. A hosted-Fabro offering has been announced or shipped, and the product owner decides that its bundle intake should land in `fabro.report` rather than in a purpose-built hosted intake product. That is a Strategy change, not a scope expansion; this Decision is the thing it changes.
3. A systematic pattern of self-hosted Reporters failing to retrieve bundles to the Fabro team surfaces, and the root cause turns out to be an audience-scope mismatch (e.g., self-hosted deployments have fundamentally different bundle-production mechanics than `fabro dump` assumes). This would be an invalidation signal for [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] itself, at which point this Decision is revisited as a downstream consequence.

None of those conditions exist today. This Decision is stable and expected to remain so.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner as part of the Strategic WHY in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L78). Written into this card on the same date as part of the Core-tier build. Stable by its own terms. This Decision is expected to outlive every individual product-layer card in the library and is load-bearing for any downstream card that names the Reporter or Triager role.

## HOW: Application

### Following This Decision

- Every downstream card that names a Reporter inherits the self-hosted-Fabro-user shape. [[Agent - Reporter]] describes a Fabro user running Fabro on their own systems; it does not describe a hosted-Fabro user or a Fabro-agnostic "anyone with a Debug Bundle" shape.
- Every downstream card that names a Triager inherits the Fabro-team-with-own-AWS-credentials shape. [[Agent - Triager]] describes the retrieval path as direct S3 access from the team's own tooling, not as a service-mediated retrieval.
- Design discussions that reference "our users" resolve to self-hosted Fabro users by default, without caveat. Cards that need to distinguish between populations (there are none today) should name the population explicitly; cards that do not distinguish should assume self-hosted.

### Violating This Decision

- **Wrong:** A product-layer card describes a Reporter role that works for "any Fabro user, self-hosted or hosted." This silently widens the audience and invalidates the Reporter definition in [[Standard - Noun Vocabulary]]. The correct fix is to narrow the card back to self-hosted and, if hosted users are genuinely on the product owner's mind, flag that as a Strategy-change question for the product owner rather than resolving it in the card.
- **Wrong:** An implementation decision (key scheme, retention policy, metadata fields) is designed with "we'll want this to also work for hosted-Fabro users" as an implicit constraint. This is audience-shaped drift into a dual-audience product. The correct fix is to design for self-hosted-only; if hosted is ever added, the design gets revisited as a Strategy change.
- **Wrong:** A card or CLI string refers to the Reporter as "the user" or "the customer" in a way that would read naturally for a hosted-Fabro customer. The audience is a self-hosted Reporter; the vocabulary should match.

### Test

Before writing a card, a CLI string, or an implementation-level constraint that involves a Reporter or Triager, ask: "Does this assume the Reporter is running Fabro on their own systems, and the Triager is retrieving from S3 with their own AWS credentials?" If yes, the assumption is in scope under this Decision. If the assumption has drifted toward a more general audience (hosted users, managed customers, generic Fabro consumers), stop and narrow it — or flag it to the product owner as a Strategy-change question.

## Sources

- `docs/alexandria/sources/product-vision.md` (L78 Strategic WHY — "users who run fabro on their own systems"; L53–L64 Users section — Reporter and Triager definitions; L115–128 Raven's Inferences and Open Questions — hosted-Fabro question flagged and this Decision resolves it)
- `docs/alexandria/inventory/card-manifest.md` (C4 entry)

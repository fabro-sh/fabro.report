# Principle - Narrow Forever (Roadmap as Commitment)

## WHAT: The Principle

The intake-only scope of `fabro.report` is a permanent identity decision, not a v1 simplification. The Roadmap for this product is not a plan that lists future features; it is a commitment that names future features that will never be built. "Narrow forever" is the load-bearing framing — confirmed on 2026-04-22 by the product owner — and it means exactly what it says: assume forever. Scope expansion toward a fuller `bun.report`-shaped product (symbolication, in-browser rendering of Debug Bundle contents, markdown rendering, feedback collection, team dashboards, triage UIs, per-user accounts, cross-upload identity tracking) is permanently out-of-scope, not "future work." That is the identity decision. A Roadmap on this product, read correctly, reads as a list of rejections.

This Principle exists because "narrow" is a word that decays. Every software product that has ever been small has been told, by someone acting in good faith, that it should grow. The decay path is consistent: first the scope is "narrow for v1"; then it is "narrow for now"; then it is "narrow, but we'll revisit"; then a feature lands that would have been rejected at "narrow for v1" and the narrowness commitment is already gone. `fabro.report` short-circuits that decay by recording, upstream and in writing, that the scope is not "narrow for v1" — it is *narrow forever*. That phrasing makes scope expansion a Strategy change requiring product-owner approval, not an implementation decision to be argued feature-by-feature.

## WHERE: Ecosystem

- Paired with:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the Product Thesis is the identity statement in product-shape terms; this Principle is the same identity statement expressed as a commitment over time. "Pickup point, not triage surface" is what the product *is*; "narrow forever" is the posture that keeps it that way across the entire life of the product.
  - [[Principle - Reject Like bun.report Drift]] — F3 is the concrete drift list; this Principle is the positive commitment. F3 enumerates the specific `bun.report`-shaped features that must be rejected in any given design discussion; this Principle is the upstream commitment those rejections appeal to. Without F6, F3 would read as "here are features we're choosing not to build right now," which is an argument that loses. With F6, F3 reads as "here are features we have permanently committed not to build," which is not an argument at all — it is a reference to a settled decision.
- Serves:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — Strategy says the product is optimizing for team retrieval of Debug Bundles from self-hosted users, with three named anti-goals (formal support SLA, compliance posture, marketing telemetry). Narrow Forever locks in that scope is the permanent enforcement of that Strategy, not a v1 staging of a fuller product that might later pursue the anti-goals.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the canonical five nouns (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) are the entire entity surface this product will ever have. New nouns added to the product imply scope expansion; under this Principle, that is a Strategy change, not a vocabulary update. The Noun Vocabulary is effectively closed, not growing.
- Referenced by:
  - [[Principle - Minimal Public Surface Feels Calm]] — "minimal public surface" is the structural condition that makes the product's calm tone affordable. Calm is cheap when the surface is a CLI plus one Report Metadata Page; it gets expensive fast if the surface grows. Every feature this Principle keeps permanently out of scope is a feature whose tone does not have to be designed or kept consistent.
  - [[Decision - Who This Service Is For]] — audience expansion (e.g., hosted-Fabro users) is a scope change under this Principle; the Decision locks the audience in the same frame this Principle locks feature scope.
  - [[Decision - Capability URLs over Auth]] — auth, accounts, and per-user scoping are on this Principle's permanently-out-of-scope list; the Decision is the positive access-model choice that makes those rejections coherent.
  - [[Decision - No Content Dedup]] — content-hash dedup extends this Principle's logic to a new drift shape beyond the enumerated nine: an invisible operational mode would grow the public-scope discussion surface even without a visible feature landing. Legitimate under this Principle's "list can grow" clause; the Decision records the rejection and aligns the data model with the narrow-forever commitment.
  - [[Decision - 30-Day Retention Default]] — retention flexibility (per-Report overrides, retention tiers, indefinite retention) extends this Principle's logic to a new drift shape beyond the enumerated nine. Legitimate under the "list can grow" clause; the Decision locks retention as a single product-owner-adjustable number so the drift has no toehold.
  - [[Agent - Reporter]] — Reporter-facing scope expansion (accounts, "my uploads" views, post-upload feedback, in-browser bundle viewing) is on the permanently-out-of-scope list; the Reporter role as written is the Reporter role forever under this Principle.
  - [[Agent - Triager]] — Triager-facing UI (dashboards, per-team scoping, listing endpoints, any in-service triage surface) is the single most load-bearing permanently-out-of-scope item; the Triager role's "never interacts with the intake service itself" shape is the structural commitment this Principle exists to defend.
  - [[Component - Report]] — Report-field expansion (mutable state, investigation status, ACL, owner ID, content hash) is a scope question under this Principle, not an implementation question; the Component card's closed field list is a direct expression of narrow-forever on the data model.
  - [[Component - Report Metadata Page]] — page-content expansion (extra fields, tabs, sections, navigation) is on this Principle's permanently-out-of-scope list; the Component card's four-element structural contract is the page-layer expression of narrow-forever.
  - [[Standard - Competitive Context]] — re-surveying competitors to find "features worth adding" is itself a drift-shaped activity under this Principle; the one-pager closes the competitive-analysis question rather than opening it.
  - [[Standard - Information Architecture]] — IA growth (home pages, listings, per-user views, team dashboards, browse endpoints) is on this Principle's permanently-out-of-scope list; the IA Standard is the page-and-route-layer expression of narrow-forever.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — additional CLI subcommands (`fabro reports list`, `fabro report status`), post-upload feedback prompts, and multi-step delete flows are on this Principle's permanently-out-of-scope list; the three-command surface plus one-click delete is the interaction-layer expression of narrow-forever.
  - [[Standard - Accessibility Policy]] — the accessibility floor is bounded by the two surfaces this Principle keeps permanently small (CLI + one web page). No compliance program grows here because the surface does not grow.
  - [[System - Retention, Storage, and Key Scheme]] — alternate retention tiers, content-addressed layouts, per-team bucket prefixes, and additional storage mechanisms are scope expansions under this Principle; the one-bucket / one-lifecycle-rule / one-key-scheme layout is the storage-layer expression of narrow-forever.

## WHY: Belief

Products drift because "scope for v1" is defaulted to "scope we can defend only for v1." An agent acting in good faith will, correctly, treat v1-shaped constraints as relaxable when v1 ships. `fabro.report` is a greenfield product, which means v1 has not yet shipped, which means every agent touching this library is in a position to treat the narrow scope as v1-shaped. That is the drift risk this Principle addresses.

The commitment is phrased as identity rather than as strategy because identity changes require a different kind of approval than strategy changes. A strategy change is a trade-off re-evaluation: "we used to think X; we now think Y." An identity change is a product re-founding: "this product is not what it was." The scope of `fabro.report` is an identity commitment precisely so that any expansion proposal lands on the product owner's desk as a re-founding question, not as a strategy question, and has to clear a high bar before anything ships.

The cost of this Principle is that it forecloses futures. `fabro.report` will not, under this Principle, ever grow into a triage product, a crash-analysis tool, a team dashboard, or a telemetry feed — even if those would be valuable products. That cost is accepted because the alternative — a product that carries the narrowness like a v1 constraint — costs the commitment entirely. A half-committed narrowness is indistinguishable from gradual expansion, and gradual expansion is how `fabro.report` would become a second-rate `bun.report` clone. Narrow forever is the only shape of this commitment that survives contact with agents.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner, in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L79). Stable and permanent by its own terms — revision requires an explicit product-owner identity-change conversation, not an implementation discussion. This card is Foundation-tier because every Decision card and every product-layer card in this library inherits its scope commitment from this Principle; if this Principle is weakened, every downstream scope commitment weakens with it.

## HOW: Application

### Permanently Out-of-Scope List (The Roadmap-as-Rejection)

The following features exist in adjacent products (notably `bun.report`). Under this Principle, none of them will be built in `fabro.report`, now or ever. This is the complete list of the major drift shapes known at initialize time; the list can grow as new drift shapes are noticed, but items do not leave the list.

- **Symbolication.** No server-side crash parser, stack-frame resolver, or symbolication service. Symbolication belongs to Triagers with their own tooling, retrieving Debug Bundles from S3 directly.
- **Sentry integration (and any crash-reporting SaaS integration).** No third-party integrations that expect structured crash payloads. Debug Bundles are opaque tarballs to this service.
- **Debug Bundle content rendering.** No in-browser view of files inside the tarball. Not on the Report Metadata Page, not at a secondary endpoint, not via a "convenience" render.
- **Markdown rendering of structured crash data.** Related to but distinct from bundle rendering; `bun.report` renders stack frames and logs as formatted markdown. `fabro.report` renders nothing from inside the bundle.
- **Feedback collection.** No post-upload feedback form, follow-up email, or "tell us what happened" flow on the Report Metadata Page. The optional reporter message at CLI upload time is the entire feedback surface.
- **Team dashboards.** No team-facing surface listing recent Reports, filtering by Fabro version, or aggregating any Report data. Triagers retrieve bundles directly from S3 with their own credentials; that is the entire team-side interface.
- **Triage UIs.** No Triager-facing screens of any kind on the intake service. Triagers do not interact with the service itself.
- **Per-user accounts.** No account concept, no login, no user record. Reporters have no identity on the service by design.
- **Cross-upload identity tracking.** No "this looks like the same user as last week." Reporters have no identity across uploads or within one upload.

Each of these items is something an agent could be asked to build "since you're already there." Under this Principle, the answer is no, and the answer does not need to be re-derived each time — this Principle is the derivation.

### Commitment vs Plan Framing

A Roadmap is usually read as "features planned for the future." This product's Roadmap — this Principle — is read as "features named for the future in order to commit to not building them." That inversion matters.

- The commitment framing means a design discussion that starts "eventually we'll need X" where X is on the out-of-scope list above should be answered "no, we permanently will not need X in this product." Not "that's future work." Not "someday." Not "interesting — let's revisit." The agent should stop that branch of the conversation.
- The commitment framing means adding a feature to the "not now, but maybe later" pile is itself a violation. There is no "maybe later" pile for the out-of-scope list. If an item moves off the list, it is because the product owner explicitly reopened scope as a Strategy change, not because an agent quietly relabeled it.
- The commitment framing means this Principle can be cited against proposals at any stage: design, implementation, review, or retrospective. "We said narrow forever" is a complete answer to a scope-expansion proposal.

### Pairing with F3 (Concrete Drift List) and F5 (Strategy)

This Principle pairs deliberately with [[Principle - Reject Like bun.report Drift]] and [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] to form a three-card enforcement frame:

- F5 (Strategy) says *what the product is optimizing for* — team retrieval of Debug Bundles from self-hosted Fabro users, with explicit anti-goals.
- This Principle (F6, Narrow Forever) says *the scope that serves that Strategy is permanent*. Scope is not a v1 constraint subject to relaxation.
- F3 (Reject Like bun.report Drift) says *here are the specific feature shapes that would violate the commitment*, with concrete drift examples.

An agent facing a scope-expansion proposal should be able to cite the full chain: Strategy rules out the direction of the expansion; Narrow Forever rules out the timing argument ("maybe later"); Reject Like bun.report Drift rules out the specific feature shape. All three are required, because each closes a different escape hatch.

### Decision Heuristic

When someone proposes a feature that expands scope toward the out-of-scope list — or toward a new shape not yet on the list but structurally similar — ask three questions in order:

1. **Is the feature on the permanently-out-of-scope list above, or a near-shape of something on that list?** If yes, reject and cite this Principle. Do not implement; do not defer; do not queue for later.
2. **If not on the list, would accepting this feature move the product toward triage-surface shape rather than pickup-point shape?** Check [[Product Thesis - Pickup Point, Not Triage Surface]]. If yes, this is a drift shape that should be named and added to the list, and rejected.
3. **Does the proposal explicitly flag itself as a scope change requiring product-owner approval?** If yes, route to the product owner as a Strategy change, not as an implementation decision. If no, the proposal is a drift and should be reframed or rejected.

The identity decision is named "narrow forever" because that is what it is. Agents reading this card should treat that phrasing as load-bearing — not soften it to "intentionally narrow," "narrow for this release," or "narrow for the foreseeable future." The commitment is forever. That is the shape of the product.

## Sources

- `docs/alexandria/sources/product-vision.md` (L79 "Narrow forever" Confirmed Product Decision; L35–48 "What `fabro.report` Is Not"; L102–107 "Template Relationship: `bun.report`")
- `docs/alexandria/inventory/card-manifest.md` (F6 entry — Conan-promoted from Amplifier to Foundation 2026-04-22, product-owner approved)

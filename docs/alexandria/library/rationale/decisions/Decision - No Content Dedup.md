# Decision - No Content Dedup

## WHAT: The Choice

`fabro.report` **does not dedup Debug Bundles by content hash** — or by any other content-based signature — at upload time, at storage time, or anywhere in the retention lifecycle. Two Reporters who happen to upload byte-identical Debug Bundles produce two separate Reports with separate IDs, separate URLs, separate retention deadlines, and two copies of the Debug Bundle in S3. The service performs no hash computation, maintains no content-identity metadata, and runs no background job that coalesces identical bundles. This Decision records that choice and names what it costs.

The load-bearing phrasing is *no content hash, no content identity, no coalescing, ever*. An agent reading this card should not come away believing "no dedup is the v1 choice and we'll add a hash-based dedup later when storage costs matter." Dedup will not be added later; if storage costs ever genuinely require deduplication, the product owner reopens this Decision as a Strategy-adjacent change, not as an implementation optimization.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Minimal Public Surface Feels Calm]] — dedup adds surface area that is invisible to the Reporter but still part of the product: a hash-computation step on upload, content-identity metadata in storage, a collision-handling policy ("if a new upload matches an existing hash, do we point at the existing Debug Bundle, or copy, or ignore?"). Each of those is a decision this product does not have to make, and the calm register depends on not having to make it.
  - [[Principle - Narrow Forever]] — adding dedup later would be a scope question (new behavior, new edge cases, new operational mode), not a vocabulary update or an implementation tweak. This Decision aligns the data model with the narrow-forever commitment: a Report is a Report, each one is its own thing, even if two happen to contain identical bytes.
- Paired with:
  - [[Decision - 30-Day Retention Default]] — the Rationale is that bundles expire after 30 days, so any storage-cost case for dedup is small by construction. Retention bounds the worst case; without retention, dedup's cost-benefit math would look very different. These two Decisions are a system: retention makes dedup unnecessary, and no dedup keeps the retention implementation simple.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the **Report** is the upload event and public identity. The **Debug Bundle** is the tarball. Two Reports pointing at one Debug Bundle (the hash-dedup shape) would break the one-to-one relationship the Noun Vocabulary implies. This Decision keeps each Report bound to its own Debug Bundle in storage.
- Related:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point does not analyze what was dropped. Hashing a Debug Bundle to compare it to prior uploads is a form of light content analysis; even if the result is used only for dedup and never surfaced, the service would be looking inside the bundle in a way the pickup-point thesis refuses.
  - [[Principle - Reject Like bun.report Drift]] — dedup proposals can arrive disguised as operational improvements ("let's save S3 costs on repeat uploads"); this Decision is the upstream commitment those proposals have to overturn, and the rejection is not a close call.
- Referenced by:
  - [[Component - Report]] — the Report is a per-upload identity, not a per-content identity; the Component card inherits this Decision for its identity semantics and explicitly refuses content-hash or canonical-bundle fields on the Report.
  - [[Component - Report Metadata Page]] — the page explicitly does not show symbolicated stacks, rendered logs, or any content-derived information; this Decision is the upstream commitment that makes "no content inspection" a structural property rather than an implementation choice.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the CLI does not hash the Debug Bundle to check for a prior upload before sending; A7's command set enforces the no-dedup posture by not adding a content-check step. Duplicate uploads from the same Reporter produce two Reports, two URLs, and no service-side signal.
- Implemented by:
  - [[System - Debug Bundle Retention, Storage, and Key Scheme]] — the storage layout assumes one Debug Bundle per Report; A5's Report IDs are cryptographically-random (not content-derived) and the key scheme does not model content-identity or reference counts.

## WHY: Rationale

### Options Considered

- **Option A (rejected): Content-hash dedup at upload time.** Compute a hash of the Debug Bundle during upload; if the hash matches an existing Debug Bundle, point the new Report at the existing storage object and skip the upload-to-S3 step. Rejected because the benefit is a marginal storage-cost saving on a product where bundles expire in 30 days anyway, and the cost is a new operational mode: hash collision handling, reference counting for deletion (if two Reports share one bundle and one is deleted, is the bundle deleted?), and a fairness question (a Reporter whose bundle matched an older one now has a retention deadline tied to… which Report's deadline?). Every one of those is a decision this Decision prevents.
- **Option B (rejected): Content-hash dedup as a background coalescing job.** Let uploads land normally, and periodically scan storage for byte-identical Debug Bundles and coalesce them. Rejected for the same reference-counting and retention-overlap reasons as Option A, plus a new failure mode: a background job that can orphan or double-delete Debug Bundles under race conditions. For a product whose entire public identity is "pickup point," running a content-analysis sweep over stored bundles is a disproportionately invasive operational mode.
- **Option C (rejected): Per-Reporter dedup.** Dedup only against a Reporter's own prior uploads. Rejected because [[Decision - Capability URLs over Auth]] establishes that Reporters have no identity on the service; "same Reporter" is not a concept the system can detect. Per-Reporter dedup would require adding a Reporter-identity layer, which is out of scope by [[Principle - Narrow Forever]].
- **Option D (chosen): No dedup. Every upload produces a fresh Report and a fresh storage object, period.** Chosen because the Rationale in the source is exactly right at this product's scale: bundles expire after 30 days, so the worst-case storage cost of duplication is bounded by the 30-day retention window, which is small. The value of dedup is near-zero against a real cost (hash computation on every upload, content-identity metadata tracking, collision and reference-count logic). This is an uneven trade in the wrong direction, and rejecting it is the correct default.

### What Drove the Choice

The source states the Rationale in one line (`product-vision.md` L86: "Dedup: none. No content-hash dedup. Rationale: bundles expire anyway."). That line is short but load-bearing: it names dedup's value (storage savings) and names the structural fact that eliminates the value (retention bounds the duplication). Beyond the math, dedup is a content-analysis behavior, and a pickup-point product does not analyze the bundle under [[Product Thesis - Pickup Point, Not Triage Surface]]. Even if the analysis is "cheap" and "just for storage savings," the service is now doing something with bundle contents that its thesis forbids.

The calm-register argument (F4) is also present: every invisible operational mode is a place where future agents will have to make decisions the product owner has not made. "If a collision happens, what do we do?" is a question this product should never have to answer, and the way to guarantee it never has to answer it is to not introduce the collision in the first place. No dedup is not the absence of a decision; it is the decision to not maintain a collision-handling stance.

### What It Costs

- **Storage cost of duplicates is not optimized.** If two Reporters happen to upload byte-identical Debug Bundles in the same retention window, two copies live in S3 for up to 30 days. At this product's expected volume (Fabro bug reports from self-hosted Fabro users), the cost is expected to be marginal. The cost is accepted because the alternative — maintaining dedup — costs more in operational complexity than duplicate storage costs in dollars.
- **No "you've already uploaded this" signal for Reporters.** A Reporter who accidentally runs `fabro dump --upload` twice on the same Debug Bundle (or runs `fabro upload` twice against the same tarball) produces two Reports and two URLs. The service does not notice the duplication and does not signal it back. Accepted because signaling would require hash computation and a same-content comparison, which is the dedup surface area this Decision rejects.
- **No "canonical bundle" concept.** Two Reports with identical bundles have no relationship the service can see. Accepted because introducing a canonical-bundle concept would require the content-identity machinery this Decision rejects, and the team-retrieval workflow does not need that concept anyway — Triagers retrieve by Report ID, not by bundle content.

### Scope of Applicability

This Decision governs content-identity behavior inside `fabro.report` — specifically at the service layer, covering upload, storage, and retention. It does **not** govern what the Fabro team may choose to do in Triager-side tooling after retrieving bundles from S3; if Triagers want to deduplicate bundles in their own analysis workflow, that is their tooling decision, outside this service. It also does **not** govern Fabro's internal debugging tooling, which is out of scope for this library.

### Conditions That Would Trigger Revisiting

Dedup-later proposals can arrive convincingly at any storage-cost review; this Decision is what blocks them from landing as implementation decisions. Revisiting requires:

1. The product owner explicitly decides dedup is in scope, in writing, in a source-material update. The decision should include a collision policy, a reference-counting policy, and a retention-deadline-overlap policy — the three operational modes this Decision prevents.
2. Observed S3 storage cost for `fabro.report` becomes large enough that dedup's benefit measurably changes the product's cost profile. Given a 30-day retention window and self-hosted-user bundle volumes, this is unlikely to happen; if it does, it is a Strategy-adjacent signal, not an implementation signal, and goes through the product owner.
3. A retention policy change (see [[Decision - 30-Day Retention Default]]) extends retention long enough that duplication cost starts to dominate. In that case, the retention decision and this Decision are coupled and should be revisited together.

None of those conditions exist today. This Decision is stable.

## WHEN: Timeline

Confirmed 2026-04-22 by the product owner in the Confirmed Product Decisions section of `docs/alexandria/sources/product-vision.md` (L86 "Dedup: none. No content-hash dedup. Rationale: bundles expire anyway."). Written into this card on the same date as part of the Amplifier-tier Decisions Log. Stable. This Decision is coupled with [[Decision - 30-Day Retention Default]] — if retention changes substantially, the dedup math changes, and this Decision is one of the upstream inputs that would be reviewed.

## HOW: Application

### Following This Decision

- The upload path treats every upload as a fresh Report. A new Report ID is minted, a new S3 storage object is written for the Debug Bundle, and a new retention deadline is set. No hash is computed; no lookup against prior uploads is performed; no content-identity metadata is stored.
- Retention applies per-Report. When a Report expires at its 30-day deadline, its Debug Bundle is deleted from S3 regardless of whether any other Report happens to hold identical bytes. Deletion is a simple per-object operation; no reference counting, no "is this bundle still referenced elsewhere" check.
- Storage cost is not optimized for duplicate content. Operational dashboards (if they ever exist, which [[Principle - Narrow Forever]] discourages) measure raw upload volume and storage footprint, not a notional "deduplicated footprint." There is no deduplicated footprint.

### Violating This Decision

- **Wrong:** Adding a SHA-256 hash computation to the upload path "because it's cheap" and storing the hash as Report metadata. This is the first half of dedup — content identity. Even if no coalescing happens, the hash is now a visible content-analysis signal the service maintains, which drifts toward the dedup surface area this Decision rejects.
- **Wrong:** Adding a background job that scans S3 for byte-identical objects. This is the coalescing half, with the added failure mode of race conditions against deletion.
- **Wrong:** Logging "this bundle matches a prior upload" as an analytics or monitoring signal. This requires content-identity tracking and creates a Reporter-facing or Triager-facing surface (however minor) around content identity, which this Decision does not authorize.
- **Wrong:** Designing the key scheme so that Report IDs derive from Debug Bundle content in any way (e.g., content-hash-based paths). This couples the storage layout to content identity and makes removing dedup later structurally hard; the storage layout should treat the Debug Bundle as an opaque blob addressed by a content-independent Report ID.

### Test

Before adding any step that computes, stores, or compares properties of the Debug Bundle itself — hash, size-plus-magic-bytes fingerprint, type detection beyond what the opaque-tarball model allows — ask: "Does this property of the bundle get used for anything the pickup-point thesis authorizes?" Size, for inclusion on the Report Metadata Page, is authorized (it's one of the five canonical metadata fields in [[Standard - Noun Vocabulary]]). Anything else — especially anything that enables cross-Report comparison — is dedup-shaped surface area, and this Decision rejects it.

A second test: "If I retained every upload forever, would this feature become necessary?" If yes, the feature is a workaround for a retention-less world; this product has 30-day retention by [[Decision - 30-Day Retention Default]], and the feature is not needed. Rejecting dedup is cheap precisely because retention makes it unnecessary.

## Sources

- `docs/alexandria/sources/product-vision.md` (L86 "Dedup: none. No content-hash dedup. Rationale: bundles expire anyway.")
- `docs/alexandria/inventory/card-manifest.md` (A10 entry)

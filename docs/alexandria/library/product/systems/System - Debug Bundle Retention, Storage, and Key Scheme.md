# System - Debug Bundle Retention, Storage, and Key Scheme

## WHAT: Definition

The **data-topology system for the Debug Bundle flow** — the card that gives `fabro.report`'s Debug Bundle intake its retention, storage, and identifier mechanics. One S3 bucket, one lifecycle rule, one key scheme, one URL shape — each chosen so that the policy commitments made in [[Decision - 30-Day Retention Default]], [[Decision - Capability URLs over Auth]], and [[Decision - No Content Dedup]] are enforced by infrastructure rather than by application code. Specifically: a dedicated S3 bucket (`fabro-report-bundles` by default, env-overridable) in `us-east-1` (also env-overridable); date-partitioned per-Report keys of the form `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` with a sidecar `<YYYY>/<MM>/<DD>/<id>/metadata.json`; public URLs of the form `fabro.report/r/<32-hex-chars>` (128 bits of entropy); and a Terraform-declared S3 lifecycle rule that expires every object 30 days after creation. This System card is paired with [[System - Service Topology]] — that card owns the **compute/DNS/IAM** topology; this card owns the **data** topology for the Debug Bundle flow. Different change frequencies, different concerns, separate cards.

**Scope:** this card covers the Debug Bundle flow only. The parallel data-topology system for the Feedback flow is `System - Feedback Storage & Notification` (manifest A17, to be drafted in Phase 3 of the 2026-04-23 feedback-scope cascade — plain-text forward reference; no wikilink until the card exists). A17 owns the `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/feedback.json` + `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/feedback.txt` + optional `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/attachments/...` key layout under the same `fabro-report-bundles` bucket, the UUIDv7 (not 32-hex-char) ID shape, the Discord webhook push that is new to the Feedback flow, and the failure-self-reporting behavior that keeps Discord posts reliable. The bucket is shared between the two flows at the bucket level — one bucket, two top-level prefixes — but the key schemes, ID shapes, object layouts, and (for Feedback) notification side are flow-specific and live in separate System cards. This card was renamed 2026-04-23 from the scope-ambiguous `System - Retention, Storage, and Key Scheme` to the explicit `System - Debug Bundle Retention, Storage, and Key Scheme` to make the flow-specificity visible at the filename level.

## WHERE: Ecosystem

- Policy anchors (mechanism-for):
  - [[Decision - 30-Day Retention Default]] — names the 30-day retention commitment. This System card is the mechanism that enforces it: an S3 lifecycle rule with `expiration.days = 30`, declared in Terraform, runs server-side inside S3 with no cron and no application-side sweeper.
  - [[Decision - Capability URLs over Auth]] — names capability URLs as the entire access model. This System card implements the **non-guessability** property that A9 declares load-bearing: 32 hex characters = 128 bits of entropy, generated from a CSPRNG, which is the industry floor for cryptographically-unguessable secret tokens.
  - [[Decision - No Content Dedup]] — names the no-dedup posture. This System card's key scheme is the structural expression of that Decision: Report IDs are cryptographically-random, never derived from content hashes, so two Reports with byte-identical Debug Bundles are always two distinct keys.
- Entities stored:
  - [[Component - Report]] — the first-class entity whose fields this System's storage representation persists. The Report's ID is what appears as `<id>` in the key scheme; the Report's URL is what wraps that ID for public addressing; the Report's authoritative fields (per [[Standard - Noun Vocabulary]]) are what the sidecar `metadata.json` carries.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — "Debug Bundle" is the tarball (stored at `<…>/<id>/bundle.tar.gz`) and "Report" is the upload event (stored as the sidecar `metadata.json` and addressed by the capability URL). This System card uses those exact terms; the `.tar.gz` payload is never called a "payload," "archive," or "dump," and the `metadata.json` is never called a "Report" on its own.
- Scope commitments:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point has exactly two file-shaped things per drop-off (the Debug Bundle and its metadata receipt), one retention deadline, and one URL. The per-Report directory layout is the pickup-point thesis expressed in the S3 key scheme.
  - [[Principle - Narrow Forever]] — the storage surface is deliberately flat: one bucket, one key pattern, one URL shape, one lifecycle rule. Additional bucket prefixes, alternate retention tiers, or content-addressed layouts are scope expansions that go to the product owner, not storage-layer refactors.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped storage (content-addressed stack-frame caches, symbolication artifact storage, per-team sharding) is named drift. This System's storage is opaque-blob storage for Debug Bundles plus a sidecar JSON; nothing downstream of "put the bytes somewhere and expire them" lives here.
- Roles that traverse this System:
  - [[Agent - Triager]] — Triagers retrieve Debug Bundles directly from S3 with their own AWS credentials. The date-partitioned key scheme is a quality-of-life property for Triagers: `aws s3 ls s3://fabro-report-bundles/2026/04/23/` enumerates a day's Reports for a Triager doing workload triage.
  - [[Agent - Reporter]] — Reporters never touch S3 directly. Their only interaction with this System is *indirect*: they receive a capability URL whose `<id>` segment corresponds 1:1 to the Report directory in S3. Non-guessability of that URL is load-bearing for the Reporter's access model (per [[Decision - Capability URLs over Auth]]).
- IA position:
  - [[Standard - Information Architecture]] — A6 defers the exact S3 key layout and URL path shape to this System card. A6 names the endpoints and the one-CLI-one-page-per-Report contract; this card names the path strings and the key strings that satisfy it.
- Referenced by:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — A7's presigned-URL upload architecture coordinates with this System: the service mints a Report ID (32 hex chars, CSPRNG) and a presigned S3 PUT URL targeting the date-partitioned key that this card specifies. A7 names the CLI-visible behavior; this card names the path the bytes land at.
- Paired topology and stack:
  - [[System - Service Topology]] — the paired compute/DNS/IAM topology. A14 cites this card's bucket name and region; this card cites A14's Fargate IAM role as the write principal. The two together describe the full deployed system.
  - [[Decision - Service Stack]] — the choice of Rust on AWS Fargate. The AWS-native IAM access path from Fargate to S3 is one of the load-bearing reasons for that choice; this System is the data-side expression of A12's stack commitment.
- Source provenance:
  - `docs/alexandria/sources/storage-and-keys.md` — the raw material Bryan confirmed on 2026-04-23. Authoritative for every numeric choice in this card (128 bits, 30 days, 32 hex characters, `us-east-1`, `fabro-report-bundles`).

## WHY: Rationale

This System card exists because data topology and compute topology change at different rates and for different reasons, and collapsing them into one card would hide that asymmetry. The data topology here — bucket name, key scheme, URL format, lifecycle rule — is the part of the deployment that the product's **policy commitments** bind most tightly to. Changing the bucket or the region is an environment override; changing the key scheme, the URL entropy, or the retention mechanism is a change that ripples through [[Decision - 30-Day Retention Default]], [[Decision - Capability URLs over Auth]], [[Decision - No Content Dedup]], and possibly [[Standard - Information Architecture]]. The card is scoped to that tightly-coupled slice so that each piece has a single place it is specified.

The load-bearing claim of this System is that **infrastructure enforces the commitments**. The 30-day retention is not a cron job or an application-side sweeper; it is an S3 lifecycle configuration declared in Terraform that S3 executes server-side. The capability URL's non-guessability is not a runtime property of the service; it is a construction-time property of the ID (128 bits from a CSPRNG). The no-dedup posture is not a "we decided not to compute hashes" convention; it is a structural property of using random IDs rather than content-derived keys. Each policy commitment has exactly one mechanism, and that mechanism lives in code Terraform can stamp out identically across environments.

The paired *per-Report directory* layout (`<id>/bundle.tar.gz` + `<id>/metadata.json`) is the second load-bearing choice. It makes the Debug Bundle and its metadata expire on the same day under the same lifecycle rule (no orphan metadata, no orphan bundles); it makes single-Report deletion a prefix delete (atomic at the Report level); and it accepts one extra S3 read per metadata-page render in exchange for those properties. That trade-off is named explicitly in source (`storage-and-keys.md` L50) and accepted here.

## WHEN: Timeline

Drafted 2026-04-23 from source material captured the same day (`docs/alexandria/sources/storage-and-keys.md`). Stable. Revisions to the numeric choices — bucket name default, region default, entropy floor, date-partition granularity, lifecycle duration — are source-level changes that feed back into this card. Revisions to the *mechanisms* — replacing S3 lifecycle with cron, swapping the key scheme for a content-addressed one, raising or lowering the entropy floor — are policy-level changes that revisit the Decisions this System implements, not implementation-level refactors.

Paired with [[System - Service Topology]] (drafted 2026-04-23 in the same follow-on batch). A14 cites this card's bucket as its storage dependency; this card cites A14's Fargate IAM role as its write-side principal.

## HOW: Specification

### S3 Bucket

- **Name:** `fabro-report-bundles` by default. Overridable via environment variable so staging / dev / alternate deployments can use different bucket names without code changes.
- **Region:** `us-east-1` by default. Also environment-variable-overridable.
- **Access model:** private bucket. Public read is **not** enabled. The Fargate task has write access via its IAM role (owned by the paired `System - Service Topology` card); Triagers read directly using their own AWS credentials (per [[Agent - Triager]]). There is no public S3 URL for a Debug Bundle; public addressing is always through the Report URL rendered by the service.
- **Isolation:** dedicated bucket for `fabro.report`, not shared with any other Fabro infrastructure. Isolated blast radius; isolated retention policy; isolated IAM boundary.

### Key Scheme

- **Pattern:** `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` (Debug Bundle) alongside `<YYYY>/<MM>/<DD>/<id>/metadata.json` (sidecar metadata) — one per-Report "directory" (S3 prefix) containing both files.
- **Date granularity:** daily partition. Adjustable if volume demands (unlikely at this product's expected scale).
- **`<id>`:** the Report ID — 32 hex characters (see URL Format below).
- **Properties this layout guarantees:**
  - Debug Bundle and its metadata share one prefix, so the lifecycle rule expires them together (no orphan metadata, no orphan bundles).
  - Single-Report deletion is a prefix delete (atomic at the Report level), not two separate key deletes.
  - Browsing by day with `aws s3 ls s3://fabro-report-bundles/<YYYY>/<MM>/<DD>/` lists a day's Reports — a quality-of-life property for Triagers.
  - Flat listing scans remain cheap because each date-partition is small relative to the whole bucket.

### Report URL Format

- **Shape:** `fabro.report/r/<random_hex>`.
- **Length:** 32 hex characters — 128 bits of entropy.
- **Entropy rationale:** 128 bits is the industry floor for cryptographically-unguessable secret URLs (Dropbox, GitHub Gist, Imgur, and comparable products land at ~128 bits). Below 128 bits, the capability-URL model ([[Decision - Capability URLs over Auth]]) becomes fragile to defend in a security review; 160 bits is overkill for a non-financial, non-regulated-data, 30-day-retention product.
- **Character set:** lowercase hex only (`a-f`, `0-9`). URL-safe by construction, case-insensitive-friendly.
- **Example:** `fabro.report/r/a3f8b29d01e4c76f2a1b8d39e5c07f4b`.
- **Generation:** cryptographically-random, drawn from a CSPRNG. Not sequential. Not timestamp-derived. Not content-hash-derived.
- **Path vs storage layout:** the public URL path is `/r/<id>` with *no* date segment. The date partition is internal to S3. A request to `fabro.report/r/<id>` maps server-side to `<YYYY>/<MM>/<DD>/<id>/` where `<YYYY>/<MM>/<DD>` comes from the Report's upload timestamp (persisted on the metadata record).

### Retention Mechanism

- **Mechanism:** S3 lifecycle rule declared in Terraform (`expiration.days = 30`). S3 enforces the deadline server-side.
- **No cron. No application-side sweeper.** The retention commitment in [[Decision - 30-Day Retention Default]] is enforced by infrastructure. This is the load-bearing operational property: retention survives service downtime, survives code changes, and cannot drift from the declared policy.
- **Scope of expiration:** every object in the bucket. Because both the `bundle.tar.gz` and the `metadata.json` for a Report share the same creation day, both expire on the same day under the same rule.
- **At expiration:** the Debug Bundle and the sidecar metadata are deleted from S3. The Report's URL resolves as not-found (per [[Component - Report]] lifecycle step 4).

### Metadata Sidecar

- **Location:** `<YYYY>/<MM>/<DD>/<id>/metadata.json`, next to the Debug Bundle tarball in the same per-Report prefix.
- **Fields:** the Report's authoritative fields per [[Standard - Noun Vocabulary]] — ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata tags, retention deadline.
- **Trade-off accepted:** rendering the [[Component - Report Metadata Page]] requires a second S3 read (fetch the JSON before rendering the HTML). Acceptable for a minimal one-page receipt that is not on a hot path, and structurally preferable to out-of-band metadata storage (which would require separate retention engineering to keep bundle and metadata lifecycles aligned).

### Anti-Examples (What This System Is Not)

- **Wrong:** Enforcing retention via an application-side cron that scans the bucket and deletes expired objects. Adds operational overhead, invites drift when the cron misses a run, and duplicates a mechanism S3 provides natively.
- **Wrong:** Content-addressed keys (e.g., `sha256(bundle)/<id>/bundle.tar.gz`). Forbidden by [[Decision - No Content Dedup]]; introduces the hash-and-compare surface area that Decision rejects.
- **Wrong:** Sequential or timestamp-derived Report IDs. Breaks capability-URL unguessability ([[Decision - Capability URLs over Auth]]). Every Report ID must come from a CSPRNG.
- **Wrong:** Storing metadata in the S3 object's user-metadata fields only. Object user-metadata has a small size limit and is not independently retrievable as a JSON blob; the sidecar file gives the Report Metadata Page a clean read path and future flexibility without changing the bundle object.
- **Wrong:** Mixed retention — bundle at 30 days, metadata at indefinite (or vice versa). Creates orphan metadata or orphan bundles at the expiration boundary; the sidecar layout exists specifically to keep these in lockstep.
- **Wrong:** Public-read S3 bucket ACL or per-object public URLs. The access model is capability URLs rendered by the service, not direct S3 public URLs. Exposing the bucket publicly breaks the auth model and makes the date-partition layout externally enumerable.
- **Wrong:** Per-Reporter or per-team S3 prefixes. There is no Reporter identity ([[Decision - Capability URLs over Auth]]) and no Triager-facing scoping surface; per-Reporter or per-team prefixes would imply an identity surface the product refuses to add.

### Test

Before changing the key scheme, URL format, entropy floor, bucket layout, or retention mechanism, ask three questions in order. First: "Does the proposed change preserve every commitment in [[Decision - 30-Day Retention Default]], [[Decision - Capability URLs over Auth]], and [[Decision - No Content Dedup]]?" If any commitment is weakened, the change is a policy change, not a mechanism change, and goes to the product owner through the relevant Decision card. Second: "Does the proposed change preserve the property that Terraform declaratively stamps out the infrastructure?" If the change introduces runtime-only state (a cron, a sweeper, a hash index), the mechanism has drifted off this card's "infrastructure enforces commitments" principle. Third: "Does the proposed change preserve Debug-Bundle-and-sidecar-metadata co-expiration?" If not, expired-Report handling becomes ambiguous and the Report's lifecycle contract in [[Component - Report]] is broken.

## Sources

- `docs/alexandria/sources/storage-and-keys.md` (full file — authoritative for bucket name, region, key scheme, URL format, entropy floor, retention mechanism, and sidecar metadata layout; confirmed 2026-04-23)
- `docs/alexandria/sources/product-vision.md` (L87 capability-URL access; L29 30-day retention default; L31 receipt page content)
- `docs/alexandria/inventory/card-manifest.md` (A5 entry)

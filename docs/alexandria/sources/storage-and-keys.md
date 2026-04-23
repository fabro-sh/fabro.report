# Source Material — Storage, Keys, and Retention Mechanism

**Status:** raw source material, not a card. Captures the storage-layer choices Bryan confirmed on 2026-04-23.
**Captured:** 2026-04-23
**Source:** direct confirmation from Bryan Helmkamp in response to unblock-interview Group 2.
**Feeds card:** A5 `System - Retention, Storage, and Key Scheme` (manifest Amplifier tier).

---

## Confirmed Decisions

### S3 Bucket

- **New dedicated S3 bucket** for `fabro.report`.
- **Not shared** with any other Fabro infrastructure. Isolated blast radius; isolated retention policy; isolated IAM boundary.
- **Access model:** private bucket. Public read is NOT enabled. The Fargate task has write access via its IAM role; Triagers read directly using their own AWS credentials (per A2 `Agent - Triager`).
- **Region:** not specified in this source; assume the Fargate task's region (likely `us-east-1` or `us-west-2`).

### Key Scheme

- **Date-partitioned keys.** Structure: `<YYYY>/<MM>/<DD>/<id>.tar.gz` or similar.
- Rationale: natural partition for browsing, easy to correlate with retention lifecycle, no overhead from flat namespace listing.
- Date granularity: daily partition is the working default. Can be adjusted if volume demands (unlikely given the product's internal-support use case).

### Report URL Format

- **Shape:** `fabro.report/r/<random_hex>`.
- The `<random_hex>` segment is URL-safe by construction (hex is a-f, 0-9 only).
- **Length / entropy:** Bryan delegated this to Raven's judgment. **Raven's choice: 32 hex characters = 128 bits of entropy.**
  - Rationale for 128 bits: capability URLs are the *entire* access control mechanism for the product (per A9 `Decision - Capability URLs over Auth`). A9 explicitly states "non-guessable URLs are load-bearing." 128 bits is the industry standard floor for cryptographically-unguessable tokens; Dropbox, GitHub Gist, Imgur, and similar secret-URL products all land at ~128 bits equivalent. Below 128 bits, the capability-URL model starts to feel more fragile to defend in security review.
  - Rationale against shorter: 64 bits (16 hex) gives 18 quintillion values, which is brute-force-infeasible in the abstract, but with internet-scale attacker resources and no rate limiting (Bryan confirmed no rate limits in Group 5), 128 is the safer floor.
  - Rationale against longer: 160 bits (40 hex) is overkill for a non-financial, non-regulated-data, 30-day-retention product. 128 is the Goldilocks.
  - **Example URL shape:** `fabro.report/r/a3f8b29d01e4c76f2a1b8d39e5c07f4b` (32 hex chars).
- **ID generation:** cryptographically-random, not sequential. Implementation likely uses the Rust `rand` crate with a CSPRNG source. Not a timestamp-derived or counter-based ID.

### Retention Mechanism

- **S3 lifecycle rule** enforces the 30-day retention policy per A11 `Decision - 30-Day Retention Default`.
- No cron job. No application-side sweeper. The retention is declared in Terraform as an S3 bucket lifecycle configuration (`expiration.days = 30`) and S3 enforces it directly.
- This is the correct choice: zero operational overhead, survives service downtime, survives code changes, cannot drift from the declared policy.
- **Metadata and bundle expire together.** The retention deadline field on a Report (per F2 and A3) and the actual S3 object lifecycle must land on the same day. If metadata is stored as a sidecar JSON object in S3, it also expires via lifecycle; if metadata is stored out-of-band (e.g., in a small DynamoDB or the S3 object's own user-metadata), the retention story has to be engineered to match.

## Implications Raven Observes (flag for review)

- **Metadata storage location is unresolved.** The Report has fields (ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata tags, retention deadline). Where do those live? Options:
  - **Inside the S3 object's user-metadata** (via `x-amz-meta-*` headers). Simple, lifecycle-synced automatically, but size-limited (~2KB total metadata) and awkward for arbitrary key/value tags.
  - **In a sidecar JSON object** adjacent to the bundle (`<date-path>/<id>.meta.json` next to `<date-path>/<id>.tar.gz`). Flexible, lifecycle-synced via identical prefix expiration, but requires a second S3 read at metadata-page render time.
  - **In a small datastore** (DynamoDB, Postgres). More flexible, richer queries, but introduces a second service to manage and a second lifecycle policy to keep in sync. Probably overkill for this product.
  - **Raven's best-guess:** sidecar JSON object co-located with the bundle. Keeps the single-store model, lifecycle auto-syncs, avoids the size-limit awkwardness of object user-metadata.
- **Hex format is aesthetic as well as technical.** A 32-hex URL like `fabro.report/r/a3f8b29d01e4c76f2a1b8d39e5c07f4b` is long-ish. An alternative — base62 encoding of 128 bits = 22 characters — would be shorter. Bryan chose hex explicitly; assumed rationale: clearer-looking, case-insensitive-friendly, URL-safe without any encoding considerations. Keep hex.
- **Browsing the bucket by date is now easy for Triagers.** `aws s3 ls s3://fabro-reports-bucket/2026/04/23/` lists a day's Reports. This is a small but real quality-of-life property.

## Open Questions (flagged for human judgment)

None of these block A5 drafting:

1. **Metadata storage: sidecar JSON vs object user-metadata vs datastore?** Not explicitly decided. Raven recommends sidecar JSON.
2. **What's the bucket name?** Likely `fabro-report-bundles` or similar. Not specified.
3. **Does the URL redirect to a path including the date, or is `/r/<id>` the whole path?** The public URL format is `/r/<id>` with no date segment; the date partition is internal-only to S3. Confirm.

## What This Source Does Not Cover

- **How bundles are uploaded.** That's CLI-side (Group 3, captured in `cli-interaction-patterns.md`) — specifically: presigned URLs so bytes do not flow through the service's compute.
- **Report Metadata Page rendering details.** That's A4 (structural) + C2 (visual).
- **Access-control mechanism.** That's A9 (capability URLs, no auth).

## Notes for Future A5 Card

When Sam drafts A5:

- **Load-bearing claim:** S3 lifecycle is the retention mechanism. No cron, no sweeper, no drift-from-policy risk.
- **Authoritative key scheme:** `<YYYY>/<MM>/<DD>/<id>.tar.gz` (+ sidecar metadata if chosen).
- **Authoritative URL format:** `fabro.report/r/<32-hex-chars>`.
- **Entropy floor:** 128 bits — not negotiable below this per A9's "non-guessable URLs are load-bearing" claim.
- **Cross-refs:** A9 (capability URLs → entropy requirement comes from there), A10 (no content dedup → key scheme uses random IDs, not content hashes), A11 (30-day retention → S3 lifecycle is the mechanism), F2 (Report fields → where they live in storage), A3 (Component - Report → its persistence representation).
- **Anti-examples to include:** sequential / timestamp-derived IDs (breaks unguessability); cron-based retention (operational overhead, drift risk); bundle and metadata expiring on different schedules (creates orphan metadata or orphan bundles).

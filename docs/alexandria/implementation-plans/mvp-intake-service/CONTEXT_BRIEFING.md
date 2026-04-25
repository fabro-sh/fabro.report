# Context Briefing

## Task Frame

**Task:** Build the full MVP `fabro.report` intake service end-to-end on the chosen stack — Rust + Docker + Fargate + Terraform + S3 + Route53 + Sentry + Discord webhook — across four planning surfaces (Report flow, Feedback flow, Operational topology, CLI surface). The first demoable slice is the full Report flow (upload → receipt page → delete).

**Target type:** System (operational + data) and Components (Report, Feedback, Report Metadata Page) and CLI surface, governed by Decisions and Standards.

**Task type:** new (greenfield service buildout from a closed library; no migration concerns).

**Constraints (non-negotiable, all sourced from the library):**

1. **Stack is Rust on AWS Fargate, Docker-packaged, Route53-fronted, Terraform-provisioned end-to-end, with SSR HTML for the one rendered page.** No TypeScript service code, no SPA, no Fly/Cloudflare/Workers, no ClickOps, no CDK/Pulumi. ([[Decision - Service Stack]])
2. **Capability URLs are the entire access model on the Debug Bundle flow.** No auth, accounts, sessions, or second factor. URL = view + delete authorization. URL must carry 128 bits of CSPRNG entropy (32 lowercase hex chars). ([[Decision - Capability URLs over Auth]], [[System - Debug Bundle Retention, Storage, and Key Scheme]])
3. **Feedback flow is fire-and-forget.** No URL returned, no rendered page, no GET endpoint, no DELETE endpoint, no recall. Exactly one Feedback route exists: `POST /v1/feedback`. ([[Decision - Fire-and-Forget Feedback]])
4. **30-day S3 lifecycle retention applies to both prefixes** (`<YYYY>/<MM>/<DD>/<id>/...` for Reports; `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/...` for Feedback). Server-side, Terraform-declared, no cron, no application sweeper. Retention is product-owner-immutable per submission. ([[Decision - 30-Day Retention Default]])
5. **No content dedup, no content hashing, no content inspection** of either intake type. Each upload/submission is a separate identity. ([[Decision - No Content Dedup]])
6. **Report and Feedback are atomically distinct entities** with asymmetric ID shapes (32-hex-char CSPRNG vs UUIDv7), asymmetric upload paths (presigned-S3-PUT vs direct POST to Fargate), asymmetric storage layouts, asymmetric notification side (none vs Discord push), and asymmetric submitter-side surface (capability URL + delete vs nothing). They MUST NOT be conflated. ([[Component - Report]], [[Component - Feedback]], [[Standard - Noun Vocabulary]] Usage Rules 6–7)
7. **Triagers never touch the service for either flow.** No Triager-facing endpoints, no list endpoints, no GET-by-id endpoints, no team dashboard, ever. ([[Agent - Triager]], [[Standard - Information Architecture]])
8. **IP capture is Feedback-only and strictly scoped to abuse protection.** Stored in `feedback.json` + CloudWatch server log; expires with the Feedback at 30 days; never exported to analytics, geolocation, compliance audit, or cross-submission joins. Debug Bundles capture no IP. ([[Decision - IP Logging for Abuse Protection]])
9. **Sentry captures the service's own errors only**, across both flows. Bundle/Feedback contents must never be forwarded to Sentry. Reporter-facing error output must never expose Sentry IDs/links. ([[Decision - Sentry for Service Errors, Not Bundle Processing]])
10. **One bucket, two prefixes, one Fargate task, one Route53 record, one IAM role, one Secrets Manager / SSM Parameter Store entry set, one Terraform repo.** No second microservice, no queue, no database, no cache, no sidecar, no autoscaling. Single-region `us-east-1` default. Discord is the single non-AWS third-party egress. ([[System - Service Topology]])
11. **Canonical CLI command set is closed at four:** `fabro dump`, `fabro dump --upload`, `fabro upload` (with `--dry-run`), `fabro feedback`. Adding a fifth is a scope change. ([[Standard - Noun Vocabulary]], [[Standard - Interaction Patterns (CLI + Delete)]])
12. **Bundle bytes never traverse Fargate compute.** Debug Bundle uploads use presigned S3 PUT URLs; the service mints the URL and the CLI uploads bytes directly to S3. 50 MB hard cap (CLI pre-flight + S3 `Content-Length` constraint). ([[Standard - Interaction Patterns (CLI + Delete)]])
13. **Feedback bytes DO traverse Fargate compute.** `POST /v1/feedback` is direct (multipart or JSON envelope — wire format not pinned at Standard layer); Fargate writes S3 objects + posts Discord, synchronously, in one request lifecycle. No queue. ([[System - Feedback Storage & Notification]])
14. **Discord webhook is the single notification target for Feedback**, with structured failure self-reporting (primary fails → second webhook call carries error text + `discord-payload.json` + `inputs.json`; if that also fails, emit Sentry event + CloudWatch log; no third retry). The webhook URL lives in `FEEDBACK_DISCORD_WEBHOOK_URL` via Secrets Manager / SSM Parameter Store. ([[System - Feedback Storage & Notification]])
15. **The Report Metadata Page renders exactly four elements:** Report ID, upload timestamp, bundle size, delete button. Anything else is drift. Expired/deleted Reports return not-found (no rendered tombstone). ([[Component - Report Metadata Page]])
16. **CLI tone register is `bun`-calibrated, "efficient and reassured."** Same register across all four canonical commands and on the Report Metadata Page. `NO_COLOR`-aware on every CLI command. WCAG 2.2 AA on the page. ([[Principle - Minimal Public Surface Feels Calm]], [[Standard - Accessibility Policy]])
17. **No third intake type, ever.** Two intake types — Debug Bundles + Feedback — exactly, forever. ([[Principle - Narrow Forever]])

**Acceptance criteria (derived from library + the goal statement):**

- `fabro report` CLI (and/or `fabro dump --upload` / `fabro upload`) successfully uploads a tarball to S3 via presigned PUT and the Reporter receives a `fabro.report/r/<32-hex-chars>` URL on stdout in the calm register.
- Visiting that URL renders the SSR Report Metadata Page showing ID, upload timestamp, bundle size, and a delete button — and nothing else.
- The delete button (URL-possession-authorized; client-side confirmation) removes both the metadata sidecar and the bundle object atomically; the URL then resolves as 404.
- `fabro feedback` (with optional positional text/files, `-e/--email`, stdin) POSTs to `/v1/feedback`, the service writes `feedback.json` + `feedback.txt` + optional `attachments/<filename>` under `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`, posts to Discord with the component-v2 payload, and the CLI prints a confirmation line (no URL).
- Discord webhook failures trigger the self-reporting second call carrying `discord-payload.json` + `inputs.json`; second-call failure emits a Sentry event.
- Both prefixes auto-expire at 30 days via the Terraform-declared S3 lifecycle rule.
- Sentry captures service-side errors across both flows; nothing inside either payload reaches Sentry.
- All AWS resources (Fargate service, ALB, Route53 record, IAM role, Secrets Manager / SSM entries, S3 bucket + lifecycle rules) are declared in Terraform; nothing exists out-of-Terraform.

## Primary Cards (full content)

The four planning surfaces map onto these primary cards. Each is reproduced with the load-bearing extracts the planner needs in order to decompose tickets without re-reading the library.

### [[Decision - Service Stack]]

**Type:** Decision (architecture)
**Relevance:** Pins the entire stack choice for every planning surface. Closes language, runtime, deployment target, IaC tool, and rendering posture.

**WHAT (the choice):** Rust application packaged as a Docker image, deployed on AWS Fargate, fronted by `fabro.report` registered on Route53, provisioned end-to-end by Terraform. SSR HTML (no SPA, no client-side framework); client-side JavaScript only where it carries real UX load (the delete-confirmation affordance).

**Load-bearing phrasing:** *AWS-native by construction, not by coincidence.* The dominant operational path is S3 access, and IAM-role-based S3 access from a Fargate task is what the stack is buying. Cross-cloud alternatives reintroduce credential management this Decision trades away.

**Rejected options (so the planner does not relitigate):**
- Bun/TS on Fly (rejected: imports the `bun.report` "we're like bun.report" framing at the implementation layer; Fly does not simplify S3 access).
- Rust on Fly (rejected: Fly→S3 requires long-lived AWS keys or cross-cloud federation — the security-posture gap is decisive).
- Cloudflare Worker + R2 (rejected: diverges from team's AWS tooling; Terraform-for-Workers is less mature).

**Application rules ("Following This Decision"):**
- New service code is in Rust. No TypeScript service code.
- Rendering is server-side. No SPA, no React/Vue/Svelte/Solid.
- Single Docker image deployed as a Fargate task. Secrets injected from Secrets Manager / SSM Parameter Store as env vars at task-start.
- S3 access via the task's IAM role. No baked-in credentials.
- Every infra resource (S3 bucket, Fargate service, Route53 records, IAM roles, lifecycle rules, Secrets Manager / SSM entries) is declared in Terraform.
- `fabro.report` DNS served from Route53; domain registration in the same AWS account.

**Out of scope for this Decision (left to implementation):** CI/CD pipeline (GitHub Actions assumed but not locked); Rust web framework (Axum assumed per Fabro's `fabro-server`, not pinned); test-against-real-S3 posture (mocked S3 by default).

**Three-question Test before any infra change:** (1) preserves AWS-native IAM path Fargate→S3? (2) preserves Terraform as single source of truth? (3) preserves Narrow Forever's permanent-shape commitment at the infra layer?

---

### [[System - Service Topology]]

**Type:** System (operational topology)
**Relevance:** Owns the operational pieces all four planning surfaces traverse: Fargate, Route53, IAM role, Secrets Manager / SSM, the shared S3 bucket, the Discord webhook egress, and the Terraform repo.

**The Pieces (each justified by a specific product need; nothing optional):**

- **Fargate task (runs the Rust service).** Single-task service by default, small task size. Hosts the Rust application that handles **both intake types**: Debug Bundle upload-coordination, Report Metadata Page render, Report delete, and Feedback intake at `POST /v1/feedback`. **Does NOT** handle Debug Bundle bytes (presigned S3 URLs). **DOES** handle Feedback payload bytes (parses POST body, writes S3 objects, posts Discord — all from inside Fargate).
- **Route53 DNS record.** `fabro.report` registered on AWS Route53, points at the Fargate ALB endpoint. Same endpoint serves both intake flows. Same AWS account; no external DNS provider.
- **IAM role attached to the Fargate task.** Least-privilege grants: presigned-URL generation for the Debug Bundle prefix; metadata-object writes for the Debug Bundle prefix; Feedback object writes (`feedback.json`, `feedback.txt`, optional attachments) under `feedback/`; reads of the Secrets Manager / SSM entries (Sentry DSN, `FEEDBACK_DISCORD_WEBHOOK_URL`, etc.). **Does NOT** grant: read of bundle/Feedback contents (no service-side post-write read), writes outside the bundle bucket, any unused permission. No baked-in keys; the task assumes its role at runtime.
- **Secrets Manager (or SSM Parameter Store) entries.** Injected as environment variables at task-start. Secret set includes Sentry DSN, `FEEDBACK_DISCORD_WEBHOOK_URL`, and any others. Choice between Secrets Manager vs SSM Parameter Store is implementation detail (both support the pattern).
- **S3 bucket (data plane).** Shared between flows: `<YYYY>/<MM>/<DD>/<id>/` for Reports (owned by [[System - Debug Bundle Retention, Storage, and Key Scheme]]); `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/` for Feedback (owned by [[System - Feedback Storage & Notification]]).
- **Discord webhook (third-party egress, new 2026-04-23).** Outbound HTTPS POST to the Discord webhook URL on every Feedback. **First non-AWS third-party dependency.** Webhook URL is the sole authentication. Failure self-reporting is governed by [[System - Feedback Storage & Notification]].
- **Terraform repository.** Declares every AWS piece above. Single source of truth. The Discord webhook URL *value* is created in Discord's UI; the *secret holding it* is a Terraform resource.

**Request flow (covers both intakes plus page render and delete):**

1. CLI/browser resolves `fabro.report` via DNS → Route53 returns Fargate ALB endpoint → request reaches the Rust service.
2. **Debug Bundle upload coordination** (`fabro dump --upload` / `fabro upload`): service mints Report ID + presigned S3 PUT URL; CLI uploads bytes directly to S3.
3. **Report Metadata Page render:** service reads metadata JSON from S3 + renders SSR HTML.
4. **Report delete:** service removes Report from active set + deletes Debug Bundle object from S3.
5. **Feedback submission** (`fabro feedback`): CLI POSTs to `/v1/feedback`. Service generates UUIDv7, writes `feedback.json` + `feedback.txt` + optional attachments under `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`, posts Discord, returns CLI confirmation, logs the event.

**Secret-injection flow:** Terraform declares Secrets Manager / SSM entries → Fargate task definition references them → task-start injects them as env vars → Rust service reads on startup. No secrets in the Docker image, source control, or Terraform state plaintext.

**IAM flow:** Fargate task assumes its role → role allows presigned-URL gen for Debug Bundle prefix, metadata writes there, Feedback writes under `feedback/`, Secrets reads. **Does NOT allow** content reads, writes outside the bucket, or unused permissions. Triagers' S3 retrieval is a separate IAM principal (Fabro team's own AWS credentials), orthogonal to the Fargate role.

**Discord egress flow:** Fargate POSTs the component-v2 payload to the webhook URL. Webhook URL is the sole auth. On success, submission complete. On failure, second webhook call carries error + `discord-payload.json` + `inputs.json` per [[System - Feedback Storage & Notification]].

**Topology guarantees (each is load-bearing):**
- **No database.** State lives entirely in S3 (per-Report metadata sidecars). No RDS, DynamoDB, or managed K/V.
- **No load-balancer subtlety.** Single Fargate-service ALB with a single target group. No multi-target-group routing, no WAF, no custom healthcheck wiring beyond the Fargate default.
- **No autoscaling complexity.** Default single-task / small-task-size config. Autoscaling is implementation detail to revisit only if volume demands.
- **No message queue, no cache, no sidecar container.** Both flows synchronous.
- **Single-region (`us-east-1` default, env-overridable), single-cloud for AWS; Discord is the one non-AWS dependency** (Sentry is also third-party but treated as AWS-native via Secrets Manager DSN).
- **Observability stack:** CloudWatch receives Fargate stdout/stderr; Sentry receives service-side errors. No metrics/tracing in the default topology.
- **Third-party egress is strictly bounded:** Sentry + Discord, nothing else. Adding a third egress is a topology expansion routed to product owner.

**Anti-examples (what this topology IS NOT):** separate microservice for any function; SQS/Kafka between service and S3; RDS/DynamoDB for metadata; ElastiCache or CloudFront in front of dynamic endpoints; baked-in AWS keys in Docker/Fargate; non-Route53 DNS; ClickOps or CDK alongside Terraform; multi-region active-active.

**Three-question Test for any topology change:** (1) preserves AWS-native IAM path service→S3? (2) preserves Narrow Forever's minimal-topology commitment? (3) preserves declarative-stamp-out (no runtime-only state, no ClickOps, no out-of-Terraform resource)?

---

### [[System - Debug Bundle Retention, Storage, and Key Scheme]]

**Type:** System (data topology — Debug Bundle flow)
**Relevance:** Owns the data shape of the Report flow: bucket name, S3 key layout, URL shape, retention mechanism, sidecar metadata. The Report-flow planner consults this card directly.

**S3 Bucket:**
- **Name:** `fabro-report-bundles` by default. Env-overridable for staging/dev/alternate deployments.
- **Region:** `us-east-1` by default. Env-overridable.
- **Access model:** Private. Public read NOT enabled. Fargate task writes via its IAM role; Triagers read directly with their own AWS credentials. No public S3 URL; public addressing is always through the rendered Report URL.
- **Isolation:** Dedicated to `fabro.report`, not shared with any other Fabro infrastructure.

**Key Scheme (Debug Bundle prefix):**
- **Pattern:** `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` (Debug Bundle) AND `<YYYY>/<MM>/<DD>/<id>/metadata.json` (sidecar) — one per-Report S3 prefix containing both files.
- **Date granularity:** Daily.
- **`<id>`:** 32 hex characters (the Report ID).
- **Properties guaranteed:** bundle + metadata expire together under the same lifecycle rule; single-Report deletion is a prefix delete (atomic at the Report level); date-partition browsing for Triagers (`aws s3 ls s3://fabro-report-bundles/<YYYY>/<MM>/<DD>/`); flat listing scans remain cheap.

**Report URL Format:**
- **Shape:** `fabro.report/r/<random_hex>`.
- **Length:** 32 hex characters = **128 bits of entropy.**
- **Entropy rationale:** 128 bits is the industry floor for cryptographically-unguessable secret URLs (Dropbox, GitHub Gist, Imgur land at ~128 bits). Below 128 bits the capability-URL model becomes fragile; 160 is overkill.
- **Character set:** lowercase hex only (`a-f`, `0-9`).
- **Example:** `fabro.report/r/a3f8b29d01e4c76f2a1b8d39e5c07f4b`.
- **Generation:** CSPRNG. Not sequential. Not timestamp-derived. Not content-hash-derived.
- **Path vs storage layout:** public URL path is `/r/<id>` with **no date segment**. The date partition is internal to S3. Server-side the request maps to `<YYYY>/<MM>/<DD>/<id>/` where the date comes from the Report's upload timestamp (persisted on the metadata record).

**Retention Mechanism:**
- **Mechanism:** S3 lifecycle rule declared in Terraform (`expiration.days = 30`). S3 enforces server-side. **No cron. No application-side sweeper.**
- **Scope of expiration:** every object in the bucket. Bundle + metadata share the same creation day, expire same day under the same rule.
- **At expiration:** bundle + sidecar deleted from S3; URL resolves as not-found.

**Metadata Sidecar:**
- **Location:** `<YYYY>/<MM>/<DD>/<id>/metadata.json`, next to the bundle in the same per-Report prefix.
- **Fields:** the Report's authoritative fields per [[Standard - Noun Vocabulary]] — ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata tags, retention deadline.
- **Trade-off accepted:** rendering the Report Metadata Page requires a second S3 read (fetch JSON before rendering HTML). Acceptable for a non-hot-path one-page receipt.

**Anti-examples:** application-side cron retention; content-addressed keys; sequential or timestamp-derived Report IDs; metadata in S3 user-metadata only; mixed retention (bundle 30 days, metadata indefinite); public-read ACL or per-object public URLs; per-Reporter or per-team prefixes.

**Three-question Test for any storage change:** (1) preserves every commitment in [[Decision - 30-Day Retention Default]], [[Decision - Capability URLs over Auth]], and [[Decision - No Content Dedup]]? (2) preserves Terraform declarative-stamp-out? (3) preserves bundle-and-sidecar co-expiration?

---

### [[System - Feedback Storage & Notification]]

**Type:** System (data + notification topology — Feedback flow)
**Relevance:** Owns the data shape and Discord-push mechanics of the Feedback flow. The Feedback-flow planner consults this card directly. Asymmetric to A5 in every specific that matters.

**S3 Bucket and Prefix:**
- **Bucket:** `fabro-report-bundles` by default — same bucket A5 uses. Env-overridable.
- **Prefix:** `feedback/` — all Feedback objects under this top-level prefix. Debug Bundle objects use no `feedback/` segment.
- **Region:** `us-east-1` by default, env-overridable.
- **Access model:** private. Same posture as Reports; Triagers read with own AWS credentials.

**Key Scheme (Feedback prefix):**
- **Pattern:** `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/<object>` where `<object>` ∈ {`feedback.json`, `feedback.txt`, `attachments/<filename>`}.
- **Date partition:** daily, matching A5.
- **`<uuidv7>`:** Feedback's UUIDv7 in canonical lowercase-hex hyphenated form (e.g., `018f2e7c-3a4b-7000-8000-111122223333`). Time-ordered for chronological Triager browsing.
- **Properties guaranteed:** envelope + message + attachments share one prefix → expire together; manual single-Feedback delete (for abuse cleanup) is a prefix delete; date-partition browsing for Triagers; clean separation from Debug Bundle prefix.

**Object Shapes:**
- **`feedback.json` (always present):** structured envelope. At minimum `id` (UUIDv7), `timestamp` (ISO 8601), `ip` (server-captured), `email` (optional), `fabro_version`, `platform`, `arch`, `retention_deadline` (`timestamp + 30 days`), `attachments` (manifest array of `{filename, size, content_type}`; empty when none).
- **`feedback.txt` (when text supplied):** message text as the Feedback Submitter supplied it. Plain UTF-8. Not wrapped, not modified, not parsed server-side.
- **`attachments/<filename>` (when files attached):** file bytes as uploaded; filename preserved with implementation-level sanitization where security requires (reject path traversal sequences). Content-type stored in `feedback.json` manifest.

**Upload Path (Direct POST to Fargate — distinct from Debug Bundles' presigned-URL architecture):**

1. CLI sends HTTPS `POST /v1/feedback` to the Fargate service (same Route53 endpoint). Body carries: message text (if any), file attachments (if any), email (if `-e`/`--email` passed), Fabro version / platform / arch metadata. **Wire format (multipart form vs JSON envelope with base64 files vs other) is implementation detail not pinned at the Standard layer.**
2. Fargate task parses the body, generates a UUIDv7, constructs the S3 key prefix from current date + UUIDv7, writes objects to S3 via its IAM role.
3. Fargate constructs the Discord payload and POSTs to the webhook URL (held in `FEEDBACK_DISCORD_WEBHOOK_URL`).
4. Fargate emits a structured server-log line (UUIDv7, timestamp, IP, submission size, attachment count) to stdout. CloudWatch captures it.
5. Fargate returns a confirmation response to the CLI. **No URL returned.** CLI prints the confirmation line per [[Standard - Interaction Patterns (CLI + Delete)]].

The service is **synchronous end-to-end**: CLI waits for confirmation; service does not emit confirmation until S3 writes + Discord post complete (or fail and self-report). No background worker, no queue, no deferred-processing mode. Synchrony is a deliberate Narrow Forever simplicity choice.

**Discord Webhook (Notification Side):**
- **Webhook URL** held in `FEEDBACK_DISCORD_WEBHOOK_URL` via Secrets Manager / SSM Parameter Store. URL itself is the sole auth (Discord's webhook model). No bot user, no OAuth.
- **Payload shape:** Discord's **component-v2** pattern, modeled on `bun.report`'s backend `sendToDiscord` implementation:
  - **Header:** `## <email>` if Submitter provided one; neutral placeholder (e.g., `## (no email provided)`) otherwise.
  - **Sysinfo one-liner:** `Fabro v<version> <platform> <arch>` — always present.
  - **Message body:** the `feedback.txt` content if present; absent or placeholder for attachment-only submissions.
  - **File attachments tree:** ASCII-list of first 20 attachment filenames with "… and N more" suffix when over 20.
  - **Footer:** Feedback's UUIDv7, optionally the IP (for Triager abuse-pattern awareness), S3 link to the Feedback's prefix (`s3://fabro-report-bundles/feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`) or a presigned URL when implementation chooses.
- **Attached tarball (optional):** if total attachment size fits within Discord's webhook upload limit (~25 MB typical), the service may construct a tarball of attachments and attach it inline. When over the limit, **skipped** (tree still lists; S3 link in footer is the retrieval path). No silent truncation; the service either attaches the full tarball or none.
- **Rate-limit awareness:** Discord webhooks have rate limits (flagged as a concern). **No per-IP soft rate limit implemented as of 2026-04-23.** Abuse protection deferred to A18's logging-for-investigation scope. If abuse patterns emerge, a per-IP rate limit may be added later as a mechanism change to this card (not a policy change to A18).

**Failure Self-Reporting (Discord Webhook Reliability):**

When the primary Discord webhook call fails (any non-2xx response, any network error), the service makes a **second webhook call** carrying:
- **Header:** failure-notice marker (e.g., `## ⚠️ webhook failure`, register-calibrated per F4).
- **Body:** error text (HTTP status, message, truncated as needed).
- **Attached files:** `discord-payload.json` (the payload the first call attempted) + `inputs.json` (the incoming request payload).
- The service POSTs the second payload to the same webhook URL.
- **If the second call also fails:** emit a Sentry event + log to CloudWatch. **No third retry.** S3 object stands as the authoritative record.

Load-bearing property: **visibility without silent failure.** The team sees that something went wrong in the same Discord channel they would have read a successful post in.

**Retention Mechanism:**
- S3 lifecycle rule (`expiration.days = 30`) applied to every object under `feedback/`. Server-side; no cron, no app-side sweeper. Same pattern as A5.
- At expiration: `feedback.json`, `feedback.txt`, all `attachments/...` deleted. Discord post is governed by Discord's own retention.

**Anti-examples:** returning a capability URL on `POST /v1/feedback` success; implementing `DELETE /v1/feedback/:id` ("only for abuse cleanup"); implementing `GET /v1/feedback/:id` for any consumer; using presigned S3 URLs for Feedback uploads; writing IP to a second persistence layer; adding a second notification target (email/Slack/SMS/pager); retrying primary Discord call multiple times before self-reporting; deduping Feedbacks by content hash; storing the webhook URL in source control / Docker image / Terraform plaintext.

**Three-question Test for any change:** (1) preserves A19, A18, A11? (2) preserves Narrow Forever's minimal-topology commitment? (3) preserves asymmetry with A5 (different prefix, different ID, different upload path, different layout, plus Discord notification)?

---

### [[Component - Report]]

**Type:** Component (entity)
**Relevance:** The first-class entity for the Debug Bundle flow. Carries the authoritative field list and the four-event lifecycle.

**Authoritative Fields:**
- **ID** — internal identifier; basis of the S3 key; must have CSPRNG entropy sufficient that URLs are not brute-forceable.
- **URL** — public short URL derived from the Report ID. Possession = capability for view + delete.
- **Upload timestamp** — measured by the service; basis for retention-deadline computation; rendered on the page.
- **Bundle size** — byte size; the **one** property of the Debug Bundle this service observes; rendered on the page.
- **Optional reporter message** — short free-form string from `-m "short message"`. The **entire** feedback surface for the Debug Bundle flow under F3.
- **Fabro version** — captured at upload outside the tarball; the **only** property that identifies a Reporter (the Reporter has no service-side identity).
- **Arbitrary key/value metadata tags** — free-form. The service does not interpret them. Team workflow hints (linked issue, triage bucket) live here when Reporters choose; they earn no first-class status.
- **Retention deadline** — `upload timestamp + 30 days`. Reporter-immutable; product-owner-adjustable as policy only.

**Lifecycle (exactly four events, no other states):**
1. **Created at successful upload.** `fabro dump --upload` (or `fabro upload`) uploads the bundle to S3; service mints Report ID, writes the storage object, records the Report's fields, returns the URL to the CLI. Creation is **atomic with successful upload** — partial uploads do not produce Reports.
2. **Possibly visited (and re-visited) via the Report Metadata Page.** Visits do not modify the Report; no "viewed" state.
3. **Deletable at any time during retention, via URL possession.** Delete button on the page submits a delete action authorized solely by URL possession. Removes Report from active set + deletes bundle from S3. Terminal — no undelete, no archive, no "recently deleted" view.
4. **Auto-deletes at retention deadline** via the S3 lifecycle rule.

No modification after creation. There is no edit operation for any Report field.

**Key couplings (load-bearing):**
- **A Report contains a reference to a Debug Bundle; they are separate entities.** A Report is public (URL, metadata, delete button); a Debug Bundle is never public (S3-only, Triager-credentialed retrieval).
- **The Report's URL is the capability token.** Not the ID in isolation — the URL. (The Report ID *alone* is what a Triager uses as an S3 key; that path is governed by AWS IAM.)
- **The Report's retention deadline is 30 days from creation, not adjustable.**

---

### [[Component - Feedback]]

**Type:** Component (entity)
**Relevance:** The first-class entity for the Feedback flow. Asymmetric to Report in every specific that matters.

**Asymmetries with [[Component - Report]] (load-bearing — must not be conflated):**
- **ID shape:** Report uses 32-hex 128-bit CSPRNG (capability token); Feedback uses **UUIDv7** (time-ordered, server-generated, **NOT a capability token, NOT a secret**).
- **Access model:** Report is capability-URL-addressed (view + delete); Feedback has no URL, no rendered page, no delete from the Submitter's side.
- **Notification surface:** Report is NOT pushed anywhere (Reporter shares URL out-of-band); Feedback IS pushed (Discord webhook to private team channel immediately after S3 writes).
- **Storage layout:** Report = `bundle.tar.gz` + sidecar `metadata.json` per-Report prefix; Feedback = `feedback.json` + `feedback.txt` + optional `attachments/` per-Feedback prefix under `feedback/`.
- **Verb:** Debug Bundles are *uploaded*; Feedbacks are *submitted*.

**Authoritative Fields:**
- **ID (UUIDv7)** — server-generated at the moment `POST /v1/feedback` is received. Time-ordered. Not unguessable; fine because not a capability token.
- **Message text (optional)** — `feedback.txt`. Absent for attachment-only submissions. No size cap at the Component layer; implementation caps governed by A17 + Discord constraints.
- **File attachments manifest (optional)** — list inside `feedback.json` (`{filename, size, content_type, storage-key}`); bytes stored as separate S3 objects under `attachments/`.
- **Email (optional)** — opaque string from `-e/--email`. Not validated, not an account token, not used to authorize anything, not used to group submissions, not a notification target. Rendered as Discord post header if present; neutral placeholder otherwise.
- **Client IP (server-captured)** — observed at `POST /v1/feedback`. Written to `feedback.json`. Retained for the standard 30 days (no separate lifecycle). **Strictly abuse protection** per A18.
- **Fabro version / platform / arch** — captured by CLI in the request payload. Stored in `feedback.json`. Appears as Discord sysinfo one-liner.
- **Timestamp** — submission moment, measured by the service. Embedded in UUIDv7 + stored in `feedback.json`.
- **Retention deadline** — `submission timestamp + 30 days`. Submitter-immutable; product-owner-adjustable as policy only.

**Lifecycle (exactly three events):**
1. **Created at successful submission.** Service mints UUIDv7, writes `feedback.json` + `feedback.txt` (if any) + attachments to S3 under `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`, posts Discord, logs the event, returns CLI confirmation. **Creation is atomic with the S3 write and Discord post; if either fails, A17's failure-self-reporting path applies.**
2. **Read (optionally) by a Triager.** Discord post is primary surface; S3 retrieval (Triager's own AWS credentials) is secondary. Reads do not modify; no "read" state.
3. **Auto-deletes at retention deadline** via S3 lifecycle rule on the `feedback/` prefix. Discord post governed by Discord's own retention.

No modification after creation. No delete from the Submitter's side. No edit operation.

---

### [[Component - Report Metadata Page]]

**Type:** Component (rendered surface)
**Relevance:** The service's ONE rendered surface on the Debug Bundle flow. Owns the structural contract of the page.

**The four elements (and only these four):**
1. **Report ID** — internal identifier, displayed for Reporter recognition + Triager S3 lookup. Not a clickable link.
2. **Upload timestamp** — moment of successful upload (recommended default ISO 8601 UTC per [[Standard - Receipt Page Sketch]]).
3. **Bundle size** — byte size, human-readable (recommended default decimal `2.3 MB` per C3).
4. **Delete button** — triggers the delete action authorized solely by URL possession. Copy + confirmation flow per [[Standard - Interaction Patterns (CLI + Delete)]].

**No other elements.** No navigation, no breadcrumbs, no related-Reports list, no "your other uploads," no footer links beyond C3's symbol-only footer.

**What the page does NOT show:**
- Debug Bundle contents (no file listing, no log preview, no rendered stack frame, no excerpt).
- Reporter identity (there is none).
- Triager-facing data (no team dashboard, no "viewed by team," no assignment, no triage status).
- "Recent uploads" or "my uploads" view.
- Form, feedback prompt, or "tell us more" surface.
- Symbolicated stack, rendered logs, or any content-derived information.
- Navigation or secondary pages (no other pages exist).

**Capability-URL access model:**
- Any request to the URL renders the page (during retention; while not deleted).
- No session cookie, no login, no access-check beyond "can the request reach this URL."
- Delete button submits a delete action requiring no additional auth — possession of the URL is the authorization.
- No "only the original Reporter can delete" concept.
- URL entropy is structural: 128 bits per A5.

**Expired-Report behavior:** URL resolves as not-found. **No "expired" state, no "this Report has been deleted" surface, no tombstone page.** Deliberate; prevents an entire scope-expansion vector.

**Tone inheritance:** Every user-facing string inherits [[Principle - Minimal Public Surface Feels Calm]]. Tone consistency between CLI and the page is non-negotiable.

**Deferred (referenced, not owned by this card):**
- **Visual system** — [[Standard - Design System (Receipt Page + CLI Output)]] (typography Geist + JetBrains Mono, palette, semantic tokens, dark-mode-only, brand-asset rule).
- **Visual layout** — [[Standard - Receipt Page Sketch]] (narrow ~500px centered column, three terminal-style metadata rows, secondary text-link delete affordance, symbol-only footer, generic-404 state pages).
- **Delete confirmation flow** — [[Standard - Interaction Patterns (CLI + Delete)]].
- **Accessibility** — [[Standard - Accessibility Policy]] (WCAG 2.2 AA, keyboard-first, semantic HTML).
- **URL shape** — [[System - Debug Bundle Retention, Storage, and Key Scheme]] (`fabro.report/r/<32-hex-chars>`).

The `/ui` skill is the recommended implementation tool.

---

### [[Standard - Interaction Patterns (CLI + Delete)]]

**Type:** Standard (interaction mechanics — CLI surface + delete flow)
**Relevance:** Owns the interaction-mechanic specifications for the four canonical CLI commands AND the delete-confirmation flow on the Report Metadata Page. The CLI-surface planner consults this card directly.

**CLI Command Set (closed at four):**

- **`fabro dump`** — produces a Debug Bundle tarball locally. Prints the tarball path on success. Does not upload, does not call `fabro.report`.
- **`fabro dump --upload`** — produces + uploads in one step. Returns the Report URL with a confirmation line. **Reporter's primary path.**
- **`fabro upload`** — uploads a previously-dumped Debug Bundle tarball. Returns the Report URL.
- **`fabro upload --dry-run`** — verifies the upload path (size pre-check under 50 MB, auth to coordination endpoint, ability to reach S3 for a presigned PUT) **without transferring bytes.** Useful on constrained or unreliable networks.
- **`fabro feedback`** — Feedback flow CLI. See "Feedback CLI" below.

**Flags available to upload paths:**
- `-m "short message"` — optional Reporter message (becomes a [[Component - Report]] field).
- arbitrary key/value metadata tags — exact flag syntax (e.g., `--tag key=value`) is implementation-level, not pinned.

**Upload Architecture (load-bearing):**

**Presigned S3 URLs.** The CLI does **NOT** POST bundle bytes to the service. Instead:
1. CLI calls the service's coordination endpoint and requests an upload slot.
2. Service mints a Report ID, persists the Report's metadata (ID, timestamp, Fabro version, reporter message, retention deadline, any tags), and returns a presigned S3 PUT URL **plus** the eventual Report URL to the CLI.
3. CLI uploads the Debug Bundle bytes **directly to S3** using the presigned URL.
4. S3 stores the bundle at the date-partitioned key per A5.

**Property:** Bundle bytes never flow through Fargate. Saves compute, bandwidth, memory; scales with S3 bandwidth.

**Network requirement:** the CLI needs reachability to BOTH the service (for coordination) and S3 (for the bytes). On most networks these are the same; `fabro upload --dry-run` surfaces the asymmetry before a long transfer when they differ.

**Progress Indicator:**
- Progress bar during upload (percentage + byte counter or ETA), client-side, reading from the HTTP library's progress callback.
- Bundles can be up to 50 MB; silent upload violates *reassured*.
- Compact in the `bun`-calibrated register. Degrades gracefully when `NO_COLOR` is set and in CI log contexts where the terminal may not repaint lines.

**Error Output:**
- Plain English. Names the problem. Suggests the next action where possible. **No stack traces on the happy-path failure surface.** No status codes as the primary surface (`UPLOAD_COMPLETE` / `ERR_NETWORK_TIMEOUT` are wrong).
- Color-when-available (red errors, yellow warnings, green successes). **`NO_COLOR`-aware** per https://no-color.org/.
- Register calibrated against `bun`.

**Delete Confirmation:**
- **Client-side confirmation.** The delete button on the Report Metadata Page triggers a client-side "are you sure?" before the actual DELETE request to the service.
- A small amount of client-side JavaScript is acceptable for the confirmation affordance (despite the SSR-only default for the page) because it's a UX-safety affordance, not a data-handling step.
- **Why not a two-step server roundtrip:** would add a page load, transient session state tied to the Reporter, complication. Client-side is faster, stateless, prevents accidental clicks.
- **Accessibility:** native browser `confirm()` is accessible by default. Custom modal needs focus trap, ESC-to-cancel, focus return, `role="dialog"`, `aria-modal="true"`. [[Standard - Accessibility Policy]] specifies the constraints; this Standard leaves modality choice open.
- **Authorization:** URL possession is the whole authorization. No additional credential check.

**Max Bundle Size:**
- **50 MB hard cap.** Pre-flight check in the CLI against the local tarball **before** the upload begins.
- **Belt-and-suspenders at S3:** presigned URL issued with a `Content-Length` constraint, so S3 also rejects an oversized upload if the CLI pre-check is bypassed.
- Size-limit rejection follows the error-output register.

**Feedback CLI (`fabro feedback`):**

- **Command syntax:** `fabro feedback [options] [feedback text ... | files ...]`. Positional arguments interpreted as feedback text (if no existing file matches the path) OR file attachments (if the path resolves to a file). Stdin accepted via pipe.
- **Options:**
  - `-e, --email <email>` — sets the email for this submission. Optional. Opaque string; **format validation is implementation detail, not specified at the Standard layer.**
  - `-h, --help` — shows help and exits.
- **Typical invocations:**
  - `fabro feedback "Love the new release!"` — plain text.
  - `fabro feedback report.txt details.log` — files only.
  - `echo "please document X" | fabro feedback --email you@example.com` — stdin + email.
- **Return value:** confirmation message on success. **No URL is returned. No Report ID is displayed. No link to a service-hosted page is shown** (because no such page exists for the Feedback flow).
- **Upload shape:** direct `POST /v1/feedback` to the service. Distinct from the Debug Bundle flow's presigned-S3-URL architecture. **Wire format (multipart form vs JSON envelope) is implementation detail, not pinned.**
- **Size handling:** **No explicit cap is specified for Feedback at the Standard layer** (the 50 MB cap is for Debug Bundle tarballs). Large attachments (esp. exceeding Discord webhook attachment limits) are A17's graceful-fallback concern, not an interaction-pattern detail. A future Feedback payload cap, if fixed, lands here.
- **Error output:** same register as Debug Bundle (plain English, no stack traces, color-when-available, `NO_COLOR`-aware, `bun`-calibrated).
- **Progress indicator:** minimal — spinner or short "sending…" line in the same compact register. Silent transfer violates *reassured*.

**No Delete on the Feedback Flow (explicit divergence):**

The Feedback flow has **no delete capability** — at the CLI, at the service, anywhere on the product's surface. Structural, not a missing feature:
- No URL is returned → no URL to use as authorization for delete.
- No rendered page exists → no surface to put a delete button on.
- **No delete endpoint on the service.** No `DELETE /v1/feedback/:id`, no delete link in the Discord post, no CLI subcommand. Adding any is a scope expansion that re-introduces a post-submission Submitter-facing surface the flow refuses.
- Expiration only via the 30-day S3 lifecycle rule.

**Load-bearing phrasing:** *Feedback is permanent from the user's perspective, expires only via S3 lifecycle.* A Feedback Submitter who wants to recall a submission contacts the Fabro team out-of-band; the team can manually delete the S3 objects with their own AWS credentials if legitimate.

**Anti-Examples:** silent success; status-code prose; two-step server-side delete confirmation; uploading bundle bytes through the service; content-hash pre-upload check; all-caps error codes; post-upload feedback prompts; color-only signaling; **returning a URL on `fabro feedback` success**; **adding any delete action for Feedback (CLI subcommand, Discord post button, service endpoint)**; **reusing the 50 MB cap as a `fabro feedback` attachment cap without confirmation.**

---

## Supporting Cards (summaries)

| Card | Type | Key Insight |
| --- | --- | --- |
| [[Decision - Capability URLs over Auth]] | Decision | URL possession = entire access model on the Debug Bundle flow. No auth, accounts, sessions, second factor, ever. The URL grants view AND delete. URL-leak-equals-access is a named cost (mitigated by 30-day retention + delete button). |
| [[Decision - Fire-and-Forget Feedback]] | Decision | Inverse-shape pair to A9. Feedback returns NOTHING post-submission — no URL, no page, no delete, no email confirmation, no reference ID. The CLI confirmation IS the entire post-submission contract. The route table contains exactly `POST /v1/feedback` and nothing else for Feedback. |
| [[Decision - 30-Day Retention Default]] | Decision | 30 days, same for every Report and every Feedback, submitter-immutable. Reporters can shorten via delete; Feedback Submitters cannot shorten. Enforced by the S3 lifecycle rule, not a cron. Per-Report or per-Feedback overrides are permanently out of scope. |
| [[Decision - No Content Dedup]] | Decision | No content hash, no content identity, no coalescing, ever. Each upload is its own Report; each submission is its own Feedback. The CLI does NOT hash-and-check before upload. Two byte-identical bundles produce two Reports with two URLs and two S3 objects. |
| [[Decision - IP Logging for Abuse Protection]] | Decision | Feedback's `ip` is the library's first identity-data capture. Strictly abuse protection. Written to `feedback.json` + CloudWatch server log; expires with the Feedback. NEVER analytics, geolocation, cross-submission joins, compliance audit, marketing telemetry. Debug Bundles capture no IP. No per-IP rate limit implemented as of 2026-04-23 (deferred). |
| [[Decision - Sentry for Service Errors, Not Bundle Processing]] | Decision | Sentry captures service-side errors across both flows (panics, upload-coordination failures, `POST /v1/feedback` failures, S3 write errors, Discord webhook double-failure). Bundle/Feedback contents are NEVER sent to Sentry. Sentry IDs/links never surface to Reporters or Feedback Submitters. |
| [[Standard - Information Architecture]] | Standard | The IA: 4 CLI commands + 1 rendered page (Report Metadata Page) per Report on the Debug Bundle flow; the Feedback flow renders no page at all. No home, no docs, no about, no help, no listing, no browse, no search, no account pages, no team dashboards, no second rendered page, no Feedback receipt. HTTP endpoints: render endpoint, delete endpoint, upload-coordination endpoint, `POST /v1/feedback`. Nothing else. |
| [[Standard - Noun Vocabulary]] | Standard | Seven canonical entities (Report, Debug Bundle, Reporter, Triager, Report Metadata Page, Feedback, Feedback Submitter). Verb asymmetry: Debug Bundles are *uploaded*; Feedbacks are *submitted*. Usage Rules 6 + 7 explicitly forbid conflating Report ↔ Feedback or Reporter ↔ Feedback Submitter. CLI command set is closed at four. |
| [[Standard - Accessibility Policy]] | Standard | WCAG 2.2 AA on the Report Metadata Page (semantic HTML, keyboard-first, screen-reader support, accessible delete-confirmation). `NO_COLOR` support across all four CLI commands. The Feedback flow has no web surface, so its accessibility floor is the CLI half only. The `/ui` skill is the recommended implementation tool. |
| [[Principle - Minimal Public Surface Feels Calm]] | Principle | Tone register: "efficient and reassured," warm/conversational, calibrated against `bun` CLI. Applies to all four CLI commands, the Report Metadata Page, the `fabro feedback` confirmation, AND the Discord post text. Silent success is wrong. Ceremony creep is wrong. Status-code prose is wrong. Real English over jargon. Symbols sparingly. Canonical names exactly. |
| [[Principle - Narrow Forever]] | Principle | Two intake types — Debug Bundles + Feedback — exactly, forever. No third intake. The "permanently out-of-scope" list (symbolication, Sentry-as-bundle-processor, content rendering, markdown rendering of crash data, post-bundle-followup feedback, team dashboards, triage UIs, accounts, cross-submission identity tracking) applies to BOTH flows equally. |
| [[Principle - Reject Like bun.report Drift]] | Principle | Concrete drift-shape list. The single most named drift is page enrichment on the Report Metadata Page. The Sentry-distinction-pattern (also applied to feedback): F3 forbids Sentry-as-bundle-processor; A13 permits Sentry for service errors. Same pattern: F3 forbids post-upload feedback ON the Debug Bundle flow; the standalone `fabro feedback` CLI is its OWN intake, not a bundle-flow extension. Stack-layer drift (borrowing `bun.report`'s Bun/TS-on-Fly) is also forbidden. |
| [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] | Principle | The product exists so the Fabro team can help self-hosted Fabro users. Three explicit anti-goals: NOT a formal support SLA, NOT compliance posture, NOT marketing telemetry. Every design choice is checked against "does this help team retrieval?" |
| [[Product Thesis - Pickup Point, Not Triage Surface]] | Product Thesis | The product is a pickup point for both intake types: Reporters/Feedback Submitters drop, Triagers pick up out-of-band. Triage happens elsewhere. Validation criteria include the four-element page content. |
| [[Standard - Design System (Receipt Page + CLI Output)]] | Standard | (C2) Visual system for the Report Metadata Page + CLI output: Geist + JetBrains Mono typography, Fabro-lifted palette, semantic tokens (e.g., `--color-coral` for destructive framing, `--color-focus` for the keyboard focus ring), dark-mode-only posture, brand-asset rule. WCAG 2.2 AA contrast floor (4.5:1 body, 3:1 large text and UI components) is the calibration constraint. |
| [[Standard - Receipt Page Sketch]] | Standard | (C3) Concrete layout for the Report Metadata Page: narrow ~500px centered column, three terminal-style metadata rows (ID / Uploaded / Size), secondary text-link delete affordance (NOT a big red button), symbol-only footer, generous vertical padding. Generic-404 for expired/deleted. ISO 8601 UTC timestamp default; decimal `2.3 MB` size default. |
| [[Standard - Competitive Context]] | Standard | (C1) `bun feedback`'s `sendToDiscord` is the pattern reference for the Feedback Discord component-v2 payload. The borrowing is at the same narrow layer as `bun.report`-the-URL-shape borrowing for Debug Bundles. |
| [[Agent - Reporter]] | Agent | Self-hosted Fabro user who ran `fabro dump --upload` (or `fabro upload`). No identity on the service — identified only by Fabro version metadata. Audience of CLI + (optionally) Report Metadata Page. Already frustrated by the bug; calibrated for *efficient and reassured*. |
| [[Agent - Feedback Submitter]] | Agent | Self-hosted Fabro user who ran `fabro feedback`. Optional email + server-logged IP + Fabro version/platform/arch metadata. Audience of `fabro feedback` CLI only. Sharing voice with a trusted team — same "efficient and reassured" register, different source emotion. |
| [[Agent - Triager]] | Agent | Fabro team member. **Never interacts with the service for either flow.** Two parallel workflows: Debug Bundle = direct S3 retrieval with own AWS credentials; Feedback = read in Discord (primary surface) + optional S3 retrieval for large attachments. No team dashboard, no listing endpoint, no GET endpoint, no Triager-facing surface — ever. |
| [[Decision - Who This Service Is For]] | Decision | Audience commitment: self-hosted Fabro users, primary and only. Hosted-Fabro users are out of scope. Audience widening would reopen multiple Decisions, not just this one. |

## Relationship Map

**Stack and topology backbone:**

- [[Decision - Service Stack]] (A12) is implemented by [[System - Service Topology]] (A14) which depends on [[System - Debug Bundle Retention, Storage, and Key Scheme]] (A5) and [[System - Feedback Storage & Notification]] (A17) for its data plane.
- A14 owns the operational topology (compute/DNS/IAM/secrets/Terraform/Discord egress); A5 owns the Debug Bundle data topology; A17 owns the Feedback data + notification topology. A14 cites A5's bucket as its IAM target; A5 cites A14's Fargate task role as the write principal; A17 cites both.
- [[Decision - Sentry for Service Errors, Not Bundle Processing]] (A13) is paired with A12 — both name the operational posture; A13 is implemented inside A14's secret-injection flow (Sentry DSN comes via Secrets Manager / SSM Parameter Store).

**Report flow chain:**

- [[Component - Report]] (A3) is the entity. A5 is its mechanism (key scheme, retention, sidecar). [[Component - Report Metadata Page]] (A4) is its rendered surface (governed by C2 + C3 visually + [[Standard - Accessibility Policy]] (A8) + [[Standard - Interaction Patterns (CLI + Delete)]] (A7)). [[Decision - Capability URLs over Auth]] (A9) is its access model. [[Decision - 30-Day Retention Default]] (A11) is its retention policy. [[Decision - No Content Dedup]] (A10) governs its identity semantics. [[Agent - Reporter]] (A1) creates it via `fabro dump --upload`/`fabro upload`. [[Agent - Triager]] (A2) retrieves the bundle by extracting the ID from the URL and going to S3 directly.

**Feedback flow chain:**

- [[Component - Feedback]] (A16) is the entity. A17 is its mechanism (key scheme, Discord push, failure self-reporting, retention). [[Decision - Fire-and-Forget Feedback]] (A19) is its access model (the inverse of A9). [[Decision - IP Logging for Abuse Protection]] (A18) is its identity-data scope. A11 is its retention policy (extended 2026-04-23). [[Agent - Feedback Submitter]] (A15) creates it via `fabro feedback`. A2 (Triager) reads it via Discord (primary) and optionally S3 (secondary). A7 owns its CLI mechanics; A8 binds CLI accessibility. **Notably, A4 has no Feedback equivalent — Feedback renders no page.**

**CLI surface chain:**

- [[Standard - Noun Vocabulary]] (F2) anchors the four canonical command names. [[Standard - Interaction Patterns (CLI + Delete)]] (A7) owns the mechanics (presigned-URL upload, progress, error register, 50 MB cap, delete confirmation, `fabro feedback` shape). [[Standard - Accessibility Policy]] (A8) binds `NO_COLOR` to all four commands. [[Principle - Minimal Public Surface Feels Calm]] (F4) governs the tone register (also applies to the Report Metadata Page AND the Discord post). [[Standard - Information Architecture]] (A6) names the closed four-command surface and the four-endpoint HTTP surface.

**Inverse-shape Decision pair:**

- A9 (Capability URLs over Auth) and A19 (Fire-and-Forget Feedback) are explicitly inverse-paired. A9 = "capability-URL access OVER auth" on the Debug Bundle flow (because there's a post-upload surface to gate); A19 = "no access model at all" on the Feedback flow (because there's no post-submission surface to gate in the first place). Both honor F6's identity-free commitment via different mechanisms.

**Atomicity-defending pair:**

- F2 Usage Rules 6+7 forbid conflating Report ↔ Feedback or Reporter ↔ Feedback Submitter. A3 and A16 each carry a "[other entity] is not a [this entity]" load-bearing note pointing at the other. A1 and A15 do the same.

**Three-card scope-defense frame:**

- F5 (Strategy) + F6 (Narrow Forever) + F3 (Reject Like bun.report Drift) form a chain: F5 says what's optimized for; F6 says scope is permanent; F3 enumerates the specific drift shapes. All three needed; each closes a different escape hatch.

**Visual triplet (page specification only complete when all three read together):**

- A4 (structural contract — four elements, expired = 404, capability-URL access) + C2 (visual system) + C3 (visual layout). Visual direction is silent on A4 by design.

**Cross-flow shared infrastructure:**

- A14 names the SHARED bucket (one bucket, two prefixes), the SHARED Fargate task (one service, both intakes), the SHARED IAM role (least-privilege grants for both prefixes + secrets), the SHARED Sentry observability (across both flows, per A13's extended scope), the SHARED Route53 endpoint (same domain serves both intakes), and the SHARED `us-east-1` region.

## Gap Manifest

These items are not specified in the library and the implementation plan will need to decide them (or escalate). Each is potential ticket surface.

| Dimension | Topic | Searched | Found | Recommendation |
| --- | --- | --- | --- | --- |
| HOW (URL shape) | Exact paths for upload coordination, render, delete | A6, A7, A5, A14 | Render is `fabro.report/r/<32-hex-chars>`. Coordination + delete paths are NOT pinned (A6 explicitly defers path strings to A5; A5 specifies the render URL only). | Plan must propose concrete paths (e.g., `POST /v1/uploads`, `DELETE /r/<id>`). Calibrate against A6's "endpoint exists / IA role" framing. |
| HOW (wire format) | `POST /v1/feedback` body shape | A17, A7 | A17: "Exact wire format (multipart form vs. JSON envelope with base64-encoded files vs. other) is an implementation detail consistent with Fargate's request-handling stack; this Standard does not pin it." A7 echoes this. | Plan must choose multipart form vs JSON envelope (with implications for max payload, CLI implementation, server-side parsing). Recommend multipart for files; document the choice. |
| HOW (wire format) | Upload coordination request/response shape | A7, A5 | Not specified. The CLI calls a coordination endpoint and receives a presigned PUT URL + the eventual Report URL; the request body shape (Fabro version, optional `-m` message, metadata tags) is not pinned. | Plan must define the JSON shape for the coordination request and response. |
| HOW (wire format) | Metadata-tags flag syntax | A7, F2, A3 | "Exact flag syntax (`--tag key=value` or similar) is an implementation-level decision consistent with `bun`-calibrated tone; this Standard does not pin the spelling." | Plan recommends `--tag key=value` (or repeated `--tag k1=v1 --tag k2=v2`). Document the wire shape too. |
| HOW (CLI binary) | The `fabro` binary itself | A6 (L70), A7, F2 | "The `fabro dump` / `fabro dump --upload` / `fabro upload` / `fabro feedback` surface is **new** to Fabro's CLI — it is not currently in Fabro's documented command set — and building this intake service implies adding those subcommands." Lives in the parent Fabro repo at `~/p/fabro-sh/fabro`, not in this repo. | Plan must scope work in BOTH repos: this repo for the Rust service + Terraform; parent Fabro repo for the four new subcommands. Decide ticket-decomposition boundary explicitly. |
| HOW (error responses) | HTTP error response format | All cards | Not specified. Tone register (plain English, no status-code prose for users) is specified at A7, but the wire format for service errors is open. | Plan must choose JSON error envelope shape (e.g., `{error: {message, code?}}`) vs plain text vs problem+json (RFC 7807). Keep human-readable; CLI must render it in the F4 register. |
| HOW (env vars) | Complete env var inventory | A14 | A14 names `FEEDBACK_DISCORD_WEBHOOK_URL`, the Sentry DSN env var (unnamed), and notes secrets are injected via Secrets Manager / SSM. Region overrides are env-overridable per A12/A14. Bucket name is env-overridable per A5. | Plan must enumerate all env vars and their sources (Secrets Manager / SSM / static): bucket name, region, Sentry DSN, `FEEDBACK_DISCORD_WEBHOOK_URL`, log level, deployment env tag, etc. |
| HOW (IAM) | Concrete IAM policy shape | A14 | Least-privilege intent named (presigned-URL gen for Debug Bundle prefix, metadata writes there, Feedback writes under `feedback/`, Secrets reads). Specific Action strings + Resource ARNs are NOT enumerated. | Plan must produce a concrete IAM policy document (e.g., `s3:PutObject` on `arn:aws:s3:::fabro-report-bundles/feedback/*`, `s3:PutObject` on `arn:aws:s3:::fabro-report-bundles/*` excluding `feedback/`, etc.). Decide whether one policy or multiple. |
| HOW (Terraform) | Module boundaries | A12, A14, A5, A17 | Terraform-as-IaC is locked. Module structure (single root vs sub-modules per AWS resource group) is NOT specified. | Plan must propose module structure (e.g., `modules/network`, `modules/compute`, `modules/storage`, `modules/secrets`, `modules/dns`, `modules/observability`) AND repo location (this repo vs separate). |
| HOW (CI/CD) | CI/CD pipeline | A12 | A12: "Does NOT govern the CI/CD pipeline choice (GitHub Actions is assumed, per Fabro's existing pattern, but not locked on this card)." | Plan must scope CI/CD: build Docker image, run tests, push to ECR, run `terraform plan` + `apply`, run smoke tests post-deploy. GitHub Actions is the assumed default. |
| HOW (local dev) | Local development story | All cards | NOT specified. A12: "Does NOT govern the testing-against-real-S3 posture (tests run against a mocked S3; real-S3 testing if added goes behind a gate — implementation detail)." | Plan must define local dev: docker-compose with localstack/MinIO? cargo run with env vars + mocked S3? How to test Discord webhook flow locally? |
| HOW (observability) | Beyond Sentry + CloudWatch | A13, A14 | A13 explicitly does NOT govern metrics or tracing. A14: "Metrics and tracing are not in the default topology; they are added if and when a measurable need appears." | Plan should NOT add metrics/tracing in MVP per Narrow Forever, but document the deliberate absence so future agents do not "fill the gap." Decide whether structured logging library is in scope. |
| HOW (presigned URLs) | Presigned URL lifetime | A5, A7 | NOT specified. | Plan must choose presigned-PUT URL TTL. Recommend short (e.g., 5–15 min) to bound the upload window but long enough to absorb slow uploads near the 50 MB cap. |
| HOW (request size limits) | Service-level body size limits | A14, A17, A7 | Debug Bundles: 50 MB hard cap (A7) — but bundles never reach Fargate (presigned URL flow). Feedback: "no explicit 50 MB cap is specified for Feedback at the Standard layer" (A7). | Plan must choose a Fargate request body size limit for `POST /v1/feedback` (sized to accommodate text + several attachments without enabling abuse). Document it. Coordinate with Discord's webhook attachment limit (~25 MB typical). |
| HOW (rate limiting) | Per-IP rate limit on `/v1/feedback` | A18, A17 | A17: "This System does not implement a per-IP soft rate limit as of 2026-04-23; abuse protection is deferred to A18's logging-for-investigation scope. If abuse patterns emerge that overwhelm the Discord webhook despite IP logging, a per-IP rate limit may be added later as a mechanism change to this card." | Plan should NOT implement rate limiting in MVP (deferred per A17/A18). Document the deliberate absence. ALB-level basic protections (e.g., AWS WAF) are explicitly outside the topology per A14. |
| HOW (abuse protection) | Concrete abuse mechanism beyond IP logging | A18, A17 | "If a Feedback submission fails inside the Fargate task — S3 write error, UUIDv7 generation error, payload parse error, Discord webhook call failure that also fails its self-reporting retry — Sentry is the operational error tracker." Out-of-band cleanup via AWS tooling for abuse Feedbacks. | MVP: IP logging only (per A18). No per-IP rate limit. No content filtering. Document this is the deliberate posture and what triggers reopening it. |
| HOW (Discord payload) | Exact component-v2 JSON shape | A17 | Header / sysinfo / body / attachments tree / footer named. Exact JSON keys, embed structure, color, and Discord-specific component-v2 schema NOT enumerated. | Plan must produce a concrete sample payload, modeled on `bun feedback`'s `sendToDiscord` per C1. Must respect F4 tone register. |
| HOW (UUIDv7) | Generation library | A16, A17 | UUIDv7 ID shape locked. Specific Rust crate not named. | Implementation choice. Pick a maintained UUIDv7 crate. |
| HOW (CSPRNG) | Generation source for 32-hex-char Report IDs | A5 | "Cryptographically-random, drawn from a CSPRNG. Not sequential. Not timestamp-derived. Not content-hash-derived." | Use `rand::rngs::OsRng` or `getrandom`. Implementation choice. |
| HOW (delete confirmation) | Native `confirm()` vs custom modal | A4, A7, A8 | Both options allowed; A8 specifies the constraints if custom modal is chosen (focus trap, ESC, focus return, `role="dialog"`, `aria-modal="true"`). | Recommend native `confirm()` for MVP (accessible by default; trivial). Custom modal can be revisited if visual direction (C2/C3) demands it. |
| HOW (timestamp format) | Page render timestamp format | A4, C3 | C3 recommends ISO 8601 UTC default. | Plan documents the format chosen. |
| HOW (size format) | Page render size format | A4, C3 | C3 recommends decimal `2.3 MB` default. | Plan documents the format chosen. |
| HOW (Sentry) | Sentry init + crate choice | A13 | "The Rust service initializes Sentry at startup using a DSN supplied from Secrets Manager or SSM Parameter Store... Panics in the service's code path are captured by Sentry automatically (via the Sentry Rust integration's panic handler). Explicit `capture_error`-shaped calls in error paths the service handles without panicking are acceptable; they must not include Debug Bundle contents." | Use `sentry` crate with the Rust integration. Plan documents what context is included in events (service name, version, deployment env, request path, error type, stack trace). NEVER include payload bytes. |
| HOW (web framework) | Rust framework | A12 | A12: "Does NOT govern the Rust web framework (Axum is assumed, per Fabro's `fabro-server`, but the specific crate is implementation detail)." | Use Axum (consistent with parent `fabro-server`). Document the choice. |
| HOW (S3 SDK) | AWS SDK | A12 | NOT specified. | Use `aws-sdk-s3` (official AWS Rust SDK). Document. |
| HOW (HTTP server) | Healthcheck endpoint | A14 | A14: "no custom healthcheck wiring beyond the Fargate service default." | Implement minimal `/health` (returns 200) for the Fargate ALB target group default healthcheck. NOT publicly meaningful. |
| HOW (HTTP server) | Robots.txt / discoverability | F4, A4, A6 | NOT specified. The page contents are public-by-URL-possession but not intended for crawler indexing. | Recommend `robots.txt` returning `Disallow: /` since URLs are capability tokens that should not be indexed. Document this consciously; the service has no public listing surface anyway. |
| HOW (HTTP server) | TLS termination | A14 | NOT explicitly specified. ALB is named. | Standard ALB pattern: terminate TLS at the ALB; HTTP between ALB and Fargate task. ACM certificate for `fabro.report`. Document. |
| HOW (data) | Atomicity of upload-coordination write vs presigned PUT | A3 lifecycle, A5, A7 | A3 lifecycle event 1: "Creation is atomic with successful upload — partial uploads do not produce Reports." A5 says metadata sidecar is written separately. The atomicity property is *aspirational* but the mechanism is not pinned. | Plan must decide: write the metadata sidecar BEFORE issuing the presigned PUT (then the bundle may never arrive — orphan metadata until lifecycle expires), or AFTER successful PUT confirmation (more complex; how does the service know the PUT succeeded?). Common pattern: write sidecar after a `HEAD` confirms the bundle landed, OR write sidecar atomically and accept that some Reports may be metadata-only orphans cleaned up by the 30-day rule. Recommend the latter for MVP simplicity. |
| HOW (data) | Atomicity of `POST /v1/feedback` (S3 writes + Discord post) | A16 lifecycle, A17 | A16 lifecycle event 1: "Creation is atomic with the S3 write and the Discord post; if either fails, the failure-self-reporting path in A17 applies, and the Feedback either completes fully or self-reports its partial state to the team." | Plan must define order: write S3 first, then post Discord (so a Discord failure leaves an S3 record for the Triager to find via S3 link in the failure-self-report). What if S3 write itself fails partway through (e.g., `feedback.json` succeeds but an attachment fails)? Recommend: write all S3 objects with retries; if any fail, return error to CLI without posting Discord. |
| HOW (delete) | Atomicity of delete (sidecar + bundle) | A3 lifecycle, A5 | "Single-Report deletion is a prefix delete (atomic at the Report level), not two separate key deletes." | Plan must implement as `DeleteObjects` with both keys, OR an S3 prefix delete via `ListObjectsV2` + `DeleteObjects`. Latter is more general (handles future per-Report extensions). |
| HOW (CLI) | Where the `fabro report` CLI command lives | Goal statement, A6, F2 | The goal statement says "`fabro report` CLI uploads tarball." The library specifies `fabro dump --upload` and `fabro upload` (NOT `fabro report`). | **NEEDS_CLARIFICATION:** the goal statement uses `fabro report` but the library's canonical CLI commands per F2 are `fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro feedback`. The pre-2026-04-23 name `fabro report` is **superseded**. Plan should use the canonical names; flag the discrepancy with the product owner. |
| HOW (visual) | Receipt page assets (fonts, palette tokens) | C2, C3, A4 | C2 specifies typography (Geist + JetBrains Mono), Fabro-lifted palette, semantic tokens, dark-mode-only, brand-asset rule. Concrete CSS / SCSS tokens not in the briefing. | Plan must implement C2's visual system; consult C2 + C3 directly during ticket execution. The `/ui` skill is the recommended implementation tool per A8/A4. |
| WHEN (deployment) | Staging vs production environments | A14, A12 | A14: "the bucket name and region are env-overridable so staging / dev / alternate deployments can use different bucket names without code changes." Number of environments NOT specified. | Plan should propose at least staging + production (separate AWS accounts ideal, separate buckets minimum). Document the choice. |
| WHY (security) | Security review checkpoint | A9, A5 | "If an agent ever generates URLs with insufficient entropy (predictable IDs, short slugs, sequential numbers), the capability model collapses into no access control at all." A14 Test #1. | Plan should include a security-review ticket validating: 32-hex CSPRNG IDs; presigned URL TTL; IAM least-privilege; secrets never in source / image / Terraform plaintext; TLS termination; bucket public-read NOT enabled. |

## Anti-Patterns to Flag for Implementers

These are the failure modes the library names most loudly. Implementers should treat each as a tripwire.

**`bun.report` drift (the top-level anti-pattern shape):**
1. Adding a "Contents" tab, "Inspect bundle" button, file listing, log preview, or symbolicated stack render to the Report Metadata Page. ([[Principle - Reject Like bun.report Drift]], [[Component - Report Metadata Page]])
2. Symbolicating crash data server-side (parser, stack-frame resolver, symbolicator). ([[Principle - Reject Like bun.report Drift]])
3. Forwarding Debug Bundle contents to Sentry (Sentry-as-bundle-processor). The `bun.report` pattern, explicitly forbidden. ([[Decision - Sentry for Service Errors, Not Bundle Processing]], [[Principle - Reject Like bun.report Drift]])
4. Rendering Markdown / logs / any content from inside the bundle. ([[Component - Report Metadata Page]], [[Principle - Reject Like bun.report Drift]])
5. Adding a post-upload feedback prompt on the Report Metadata Page or via a CLI follow-up ("How was your reporting experience?"). ([[Agent - Reporter]], [[Principle - Reject Like bun.report Drift]])
6. Building any team dashboard, listing, or filter-by-Fabro-version surface. ([[Agent - Triager]], [[Standard - Information Architecture]], [[Principle - Narrow Forever]])
7. Adding per-team scoping, accounts, login, OAuth, or any auth concept. ([[Decision - Capability URLs over Auth]], [[Principle - Narrow Forever]])
8. Tracking Reporter or Feedback Submitter identity across uploads/submissions. ([[Decision - Capability URLs over Auth]], [[Decision - IP Logging for Abuse Protection]], [[Principle - Narrow Forever]])
9. Borrowing `bun.report`'s vocabulary ("Crash Report," "Stack Trace," "Symbolicator," "Feedback Form") in code, CLI strings, or docs. ([[Standard - Noun Vocabulary]])
10. Borrowing `bun.report`'s stack template (Bun/TypeScript on Fly). The stack is permanently AWS-native Rust. ([[Decision - Service Stack]])

**Conflating Report and Feedback (atomicity violations):**
11. Returning a URL on `fabro feedback` success — even as "you can view your feedback here." Violates fire-and-forget. ([[Decision - Fire-and-Forget Feedback]], [[Standard - Interaction Patterns (CLI + Delete)]])
12. Implementing `GET /v1/feedback/:id` for any consumer (Submitter or Triager). Triagers read Discord + S3-direct; Submitters have no read affordance. ([[Decision - Fire-and-Forget Feedback]], [[System - Feedback Storage & Notification]])
13. Implementing `DELETE /v1/feedback/:id` even "only for abuse cleanup." Out-of-band Fabro-team deletion uses AWS tooling against S3 directly. ([[System - Feedback Storage & Notification]])
14. Treating UUIDv7 as a secret. UUIDv7 is time-ordered and guessable; that's fine because it's NOT a capability token. ([[Component - Feedback]])
15. Using presigned S3 URLs for Feedback uploads. Calibrated for Debug Bundles' larger payloads only; Feedback uses direct POST. Switching is a mechanism-level refactor to A17, not a neutral choice. ([[System - Feedback Storage & Notification]])
16. Using direct POST for Debug Bundle bytes. Defeats the "bundle bytes never traverse Fargate" property. ([[Standard - Interaction Patterns (CLI + Delete)]])
17. Calling a Feedback a "Report" or vice versa, in code, CLI, Discord post, or docs. They are distinct entities with distinct identity models. ([[Standard - Noun Vocabulary]] Usage Rule 6)
18. Calling a Feedback Submitter a "Reporter" or vice versa. The roles are asymmetric in identity model and post-pickup surface. ([[Standard - Noun Vocabulary]] Usage Rule 7)
19. Using "upload" as the verb for Feedback submission, or "submit" as the verb for Debug Bundle uploads. The verb asymmetry mirrors the entity asymmetry. ([[Standard - Noun Vocabulary]])

**Atomicity / state violations:**
20. Adding a content-hash pre-upload check ("you've already uploaded this; return existing URL?"). Forbidden by [[Decision - No Content Dedup]]. The CLI does NOT hash bundles.
21. Adding a SHA-256 hash field to Report metadata "because it's cheap." First half of the dedup surface area; rejected. ([[Decision - No Content Dedup]])
22. Implementing retention via an application-side cron that scans the bucket. Use the S3 lifecycle rule. ([[System - Debug Bundle Retention, Storage, and Key Scheme]], [[System - Feedback Storage & Notification]])
23. Sequential or timestamp-derived Report IDs. Breaks capability-URL unguessability. Use CSPRNG. ([[Decision - Capability URLs over Auth]], [[System - Debug Bundle Retention, Storage, and Key Scheme]])
24. Public-read S3 bucket ACL or per-object public URLs. ([[System - Debug Bundle Retention, Storage, and Key Scheme]])
25. Per-Reporter or per-team S3 prefixes. There is no Reporter identity. ([[System - Debug Bundle Retention, Storage, and Key Scheme]])
26. Rendering a "this Report has expired" or "this was deleted" tombstone page. Expired = 404, full stop. ([[Component - Report Metadata Page]], [[Decision - 30-Day Retention Default]])
27. Adding "view my submission" or "my feedback" view for Feedback Submitters. ([[Agent - Feedback Submitter]], [[Decision - Fire-and-Forget Feedback]])
28. Using IP for analytics, geolocation, cross-submission joins, compliance audit, or marketing telemetry. Strictly abuse protection. ([[Decision - IP Logging for Abuse Protection]])
29. Hashing the IP at rest "to anonymize." False-privacy shape; preserves joinability while defeating investigation. ([[Decision - IP Logging for Abuse Protection]])
30. Storing IP outside `feedback.json` + CloudWatch (e.g., DynamoDB table of "recent submission IPs," Redis cache that persists IPs). ([[Decision - IP Logging for Abuse Protection]])
31. Implementing a per-IP rate limit on `POST /v1/feedback` in MVP. Deferred per A17/A18 unless abuse patterns demand it.
32. Adding a second notification target (email digest, Slack, SMS, pager). One target per intake. ([[System - Feedback Storage & Notification]])
33. Retrying the primary Discord webhook call multiple times before self-reporting. The pattern is: primary fails → self-report fires once → if self-report fails, emit Sentry. No exponential backoff, no queueing. ([[System - Feedback Storage & Notification]])

**Tone / accessibility / vocabulary drift:**
34. Silent success (just printing the URL with no confirmation line). Violates *reassured*. ([[Principle - Minimal Public Surface Feels Calm]], [[Standard - Interaction Patterns (CLI + Delete)]])
35. All-caps error codes (`UPLOAD_COMPLETE`, `ERR_NETWORK_TIMEOUT`) on the user-facing surface. ([[Principle - Minimal Public Surface Feels Calm]])
36. Stack traces shown to Reporters/Feedback Submitters on the happy-path failure surface. ([[Standard - Interaction Patterns (CLI + Delete)]])
37. Color-only signaling for errors. Must be readable as text. ([[Standard - Accessibility Policy]])
38. Hard-coded ANSI escape codes without a `NO_COLOR` guard. ([[Standard - Accessibility Policy]])
39. Mouse-only delete button on the page; styled `<div>` masquerading as a button; missing `aria-label` on icon-only buttons. ([[Standard - Accessibility Policy]])
40. Surfacing Sentry event IDs, links, or "please report this crash" prompts to Reporters or Feedback Submitters. ([[Decision - Sentry for Service Errors, Not Bundle Processing]], [[Decision - Fire-and-Forget Feedback]])
41. Shortened canonical names ("receipt page," "your bundle," "Feedbacker," "Submitter") in code, CLI, Discord post, or docs. ([[Standard - Noun Vocabulary]])
42. Discord post with emoji cascade ("A new feedback has arrived! 🎉"), URGENT framing, or any non-`bun`-calibrated voice. ([[Principle - Minimal Public Surface Feels Calm]] extended to Discord post, [[System - Feedback Storage & Notification]])

**Stack / topology violations:**
43. ClickOps any AWS resource. Terraform is the single source of truth. ([[Decision - Service Stack]], [[System - Service Topology]])
44. Baking AWS keys into the Docker image, env files, or source control. Use the Fargate task IAM role. ([[Decision - Service Stack]], [[System - Service Topology]])
45. Adding RDS, DynamoDB, ElastiCache, SQS, Kafka, a sidecar container, or any second service. ([[System - Service Topology]])
46. Multi-region active-active deployment by default. Single-region. ([[System - Service Topology]])
47. Adding a third non-AWS egress (anything beyond Sentry + Discord). ([[System - Service Topology]])
48. Adding a Triager-facing endpoint, even read-only. The team has no service-side surface. ([[Agent - Triager]], [[Standard - Information Architecture]], [[Principle - Narrow Forever]])

## What's Already Decided

The planner does NOT need to relitigate any of these — they are settled in Decisions cards. Reference the citation when a stakeholder questions any of them.

| Decision | Settled in | Don't reopen |
| --- | --- | --- |
| Stack: Rust + Docker + Fargate + Route53 + Terraform + SSR HTML | [[Decision - Service Stack]] | Bun/TS-on-Fly, Rust-on-Fly, Cloudflare Worker, SPA, ClickOps, CDK, Pulumi, multi-region |
| Access model (Debug Bundle): capability URLs only | [[Decision - Capability URLs over Auth]] | Auth, accounts, login, sessions, hybrid (URL+password), signed URLs with refresh |
| Access model (Feedback): fire-and-forget | [[Decision - Fire-and-Forget Feedback]] | Capability URL on `fabro feedback` success, reference-only ID, silent success, email confirmation, GET endpoint, DELETE endpoint, rendered page |
| Retention: 30 days, both flows, submitter-immutable | [[Decision - 30-Day Retention Default]] | 7 days, 90 days, 1 year, indefinite, per-Report overrides, retention tiers, "pin this Report," extension on revisit |
| Dedup: none | [[Decision - No Content Dedup]] | Content-hash dedup at upload, background coalescing job, per-Reporter dedup, content-hash field on metadata, "you've already uploaded this" signal |
| IP logging: Feedback only, abuse protection only | [[Decision - IP Logging for Abuse Protection]] | No IP at all (rejected), hashed IP, IP retained beyond 30 days, IP analytics, IP for cross-submission joins, IP for geolocation, IP on Debug Bundle uploads |
| Sentry: service errors only, both flows | [[Decision - Sentry for Service Errors, Not Bundle Processing]] | No Sentry at all, Sentry-as-bundle-processor, Sentry-event-IDs surfaced to Reporters, Sentry as a log aggregator |
| Audience: self-hosted Fabro users only | [[Decision - Who This Service Is For]] | Hosted-Fabro users, anyone-with-a-Debug-Bundle generic Reporter, third-party support partners |
| Bucket: `fabro-report-bundles` (env-overridable), `us-east-1` default, private | [[System - Debug Bundle Retention, Storage, and Key Scheme]], [[System - Feedback Storage & Notification]] | Public-read bucket, multiple buckets, per-team prefixes |
| Key scheme (Debug Bundle): `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` + `<YYYY>/<MM>/<DD>/<id>/metadata.json` | [[System - Debug Bundle Retention, Storage, and Key Scheme]] | Content-addressed keys, sequential IDs, single-file storage |
| Key scheme (Feedback): `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/feedback.json` + `feedback.txt` + `attachments/<filename>` | [[System - Feedback Storage & Notification]] | Single-file storage, different prefix, derived-from-content keys |
| URL format: `fabro.report/r/<32-hex-chars>` (128 bits CSPRNG) | [[System - Debug Bundle Retention, Storage, and Key Scheme]] | Sequential IDs, short slugs, timestamp-derived IDs, content-derived IDs |
| Report ID shape vs Feedback ID shape | [[Component - Report]] vs [[Component - Feedback]] | Treating UUIDv7 as a capability token; treating Report ID as time-ordered |
| Bundle size cap: 50 MB | [[Standard - Interaction Patterns (CLI + Delete)]] | Higher cap, lower cap, no cap |
| Feedback payload size cap: not pinned at Standard layer | [[Standard - Interaction Patterns (CLI + Delete)]] | Reusing 50 MB without confirmation; setting a cap without product-owner input |
| Report Metadata Page content: 4 elements | [[Component - Report Metadata Page]] | Adding a 5th element, removing one of the 4 |
| Expired/deleted page: 404, no tombstone | [[Component - Report Metadata Page]], [[Decision - 30-Day Retention Default]] | "This Report expired" page, "Recently deleted" view |
| Notification target (Feedback): Discord webhook only | [[System - Feedback Storage & Notification]] | Email digest, Slack webhook, SMS, pager, second Discord channel |
| Discord webhook failure handling: primary fails → self-report once → if that fails, Sentry | [[System - Feedback Storage & Notification]] | Multiple retries, exponential backoff, queueing |
| Synchronous request flow (both intakes) | [[System - Service Topology]], [[System - Feedback Storage & Notification]] | Background workers, queues, deferred processing |
| Tone register: "efficient and reassured," `bun`-calibrated, applies to CLI + Page + Discord post | [[Principle - Minimal Public Surface Feels Calm]] | Silent success, ceremony creep, status-code prose, emoji cascade, corporate apology prose |
| Accessibility: WCAG 2.2 AA + `NO_COLOR` across 4 commands | [[Standard - Accessibility Policy]] | AAA, no accessibility floor, color-only signaling, mouse-only delete |
| CLI command set: closed at 4 (`fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro feedback`) | [[Standard - Noun Vocabulary]], [[Standard - Information Architecture]] | A 5th command (`fabro reports list`, `fabro feedback delete`, etc.) |
| HTTP endpoint set: closed at 4 (render, delete, upload-coordination, `POST /v1/feedback`) | [[Standard - Information Architecture]] | List endpoints, search endpoints, GET-by-id for Feedback, account endpoints |
| No third intake type, ever | [[Principle - Narrow Forever]] | Log uploads, screenshot uploads, telemetry events, support tickets, session recordings |

## Completion Status

**Status:** DONE_WITH_CONCERNS

**Concerns (non-blocking but worth surfacing to the planner):**

1. **CLI naming discrepancy in the goal statement.** The goal says "`fabro report` CLI uploads tarball." The library's canonical commands per [[Standard - Noun Vocabulary]] are `fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro feedback`. The pre-2026-04-23 name `fabro report` is **superseded**. The plan should use the canonical names and confirm this with the product owner if any ambiguity remains. (Logged in Gap Manifest.)
2. **The four canonical CLI commands live in the parent Fabro repo (`~/p/fabro-sh/fabro`), not in this repo.** [[Standard - Information Architecture]] (L70) explicitly notes "the `fabro dump` / `fabro dump --upload` / `fabro upload` / `fabro feedback` surface is **new** to Fabro's CLI — it is not currently in Fabro's documented command set — and building this intake service implies adding those subcommands." Plan must scope work in BOTH repos.
3. **Several mechanism details are deliberately not pinned at the Standard/System layer** — wire format for `POST /v1/feedback`, exact path strings for upload coordination + delete + render, IAM policy specifics, env var enumeration, presigned URL TTL, request body size limit on Fargate, error-response wire format, Terraform module boundaries, CI/CD pipeline, local dev story. The planner must propose concrete answers for each. The Gap Manifest enumerates them all.
4. **Some implementation choices are explicit "not yet" deferrals** — per-IP rate limit, metrics/tracing, second region. The plan should NOT add these in MVP and should document the deliberate absence so a future agent does not "fill the gap" without escalation.

The library is closed at 24 cards and consistent. No card is missing for any of the four planning surfaces. The asymmetries between Report/Feedback are extensively documented across multiple cards (A3 ↔ A16, A1 ↔ A15, A5 ↔ A17, A9 ↔ A19) and the planner has redundant guard-rails against conflation drift.

**Briefing assembled. `CONTEXT_BRIEFING.md` includes Task Frame, Primary Cards, Supporting Cards, Relationship Map, Gap Manifest, Anti-Patterns, What's Already Decided, and Completion Status. 8 primary cards, 17 supporting cards, 33 gaps surfaced, 48 anti-patterns flagged.**

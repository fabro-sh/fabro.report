# Feedback Queue — MVP Intake Service Briefing

**Date:** 2026-04-24
**Briefer:** Bridget
**Output assembled:** `docs/alexandria/implementation-plans/mvp-intake-service/CONTEXT_BRIEFING.md`

The library is closed at 24 cards and assembled cleanly for this briefing. The items below are demand signals for Sam (Scribe) to consider as candidate cards or card revisions, surfaced from the gaps the briefing exposed. None of these are blocking — the MVP plan can proceed with the existing library — but each represents a place where downstream tickets will need to make implementation choices the library does not currently anchor.

## Card Candidates (New Cards That Could Be Built)

### 1. Standard - HTTP Surface Spec (path strings + wire formats)

- **Why surfaced:** The Gap Manifest has 4 entries about wire formats and path strings: upload-coordination request/response shape, `POST /v1/feedback` body shape (multipart vs JSON envelope), exact paths for upload-coordination + render + delete + Feedback intake, HTTP error response format. A6 explicitly defers path strings to A5; A5 specifies the render URL only; A17 explicitly does not pin the Feedback wire format.
- **Demand signal:** every implementation ticket touching the HTTP layer must propose these. Without an anchor card, drift is possible across PRs.
- **Suggested scope:** path strings, wire formats for both intakes, error envelope shape, healthcheck endpoint, robots.txt. Defer to product owner where the library is intentionally silent.
- **Priority:** medium. The MVP can land without it (the plan will choose), but a follow-up Standard captures the choices for reference.

### 2. Standard - CLI Copy

- **Why surfaced:** Multiple cards (A7, A8, F4) reference an unbuilt `Standard - CLI Copy` as the future home for exact strings. The MVP will produce those strings; without a card, the strings live in code only and re-derive from F4's register-only spec each time.
- **Demand signal:** every CLI string written for the four canonical commands needs to land in the F4 register. A canonical-strings card removes interpretive drift.
- **Suggested scope:** exact success / error / progress strings for each of the four canonical commands. Inherits register from F4. Co-references A7, A8.
- **Priority:** low for MVP (F4 + A7 are sufficient guard-rails), but valuable as soon as the strings stabilize.

### 3. Standard - Terraform Module Layout

- **Why surfaced:** The Gap Manifest entry for "Module boundaries" — A12 locks Terraform-as-IaC but does not prescribe module structure. The plan will choose; no card captures the choice.
- **Demand signal:** every infrastructure PR will reference module boundaries. Without an anchor, a reviewer cannot check "is this in the right module?"
- **Suggested scope:** module breakdown (network, compute, storage, secrets, dns, observability), variable conventions, environment overlay strategy.
- **Priority:** medium. The MVP plan will produce a layout; capturing it as a card prevents future PRs from re-deciding.

### 4. Standard - Local Development Story

- **Why surfaced:** Gap Manifest entry. A12 explicitly does not govern local dev; no card describes how a developer runs the service locally, mocks S3, tests the Discord webhook flow, or runs integration tests.
- **Demand signal:** new contributors will need this as soon as the second developer joins.
- **Suggested scope:** docker-compose / localstack / MinIO recipe, env var sources for local, mock Discord receiver pattern, test data fixtures.
- **Priority:** medium for MVP (the plan will produce one), high once team grows.

### 5. Decision - Atomic Write Strategy for Reports

- **Why surfaced:** A3 lifecycle event 1 says "Creation is atomic with successful upload — partial uploads do not produce Reports." A5's per-Report directory layout makes single-Report deletion atomic, but the *write* atomicity (mint Report ID + persist sidecar metadata + presigned PUT) is not specified. The Gap Manifest entry recommends one of two patterns.
- **Demand signal:** the planner has to choose, and the choice is visible in error scenarios (orphan metadata vs orphan bundles). Future PRs may second-guess.
- **Suggested scope:** decide and document the strategy (write sidecar atomically before/after PUT confirmation; rely on 30-day lifecycle to clean up orphans; or use a HEAD check after PUT).
- **Priority:** high. The plan must decide; capturing the decision prevents litigation in code review.

## Card Revisions (Existing Cards That Could Be Tightened)

### 6. A6 (Information Architecture) — could anchor concrete path strings

- **Current state:** A6 says "the exact path format (host, slug layout, route prefixes) is deferred to [[System - Debug Bundle Retention, Storage, and Key Scheme]] for the Debug Bundle flow and to [[System - Feedback Storage & Notification]] for the Feedback flow; this Standard names the *endpoints* and their *IA roles*, not their path strings."
- **Gap:** A5 only specifies the render URL (`/r/<32-hex-chars>`); no card specifies the upload-coordination, delete, or `POST /v1/feedback` paths concretely. The IA is fully closed except for the path strings themselves.
- **Suggestion:** either add concrete path strings to A6, or add them to A5 + A17, or build the new Standard - HTTP Surface Spec proposed above. Pick one.
- **Priority:** medium. Triggered by the same demand signal as candidate card #1.

### 7. A14 (Service Topology) — could enumerate env vars completely

- **Current state:** A14 names `FEEDBACK_DISCORD_WEBHOOK_URL` and the Sentry DSN env var (unnamed). Bucket name and region are env-overridable per A12/A14.
- **Gap:** the complete set of env vars the service reads at startup is not enumerated. The planner must propose them; future contributors will not know what is exhaustive.
- **Suggestion:** add a "Required Environment Variables" subsection enumerating: bucket name, region, Sentry DSN, `FEEDBACK_DISCORD_WEBHOOK_URL`, log level, deployment env tag, presigned URL TTL, etc.
- **Priority:** medium. Naturally follows MVP implementation; capture once choices are made.

### 8. A14 + A5 + A17 — could specify presigned URL TTL

- **Current state:** Not specified anywhere.
- **Gap:** the planner must choose; no anchor.
- **Suggestion:** add to A14 (operational topology) or A5 (it's a Debug Bundle flow concern) or A7 (it's an interaction-pattern concern affecting the CLI's behavior on slow networks).
- **Priority:** low. Recommend short TTL (5–15 min); document in implementation. Card revision can wait for a deliberation moment.

### 9. A17 — could specify a request body size limit for `POST /v1/feedback`

- **Current state:** A7 says "no explicit 50 MB cap is specified for Feedback at the Standard layer (that cap was calibrated for Debug Bundle tarballs)... If Bryan eventually fixes a Feedback payload cap, the cap lands here as an explicit number in a revision pass."
- **Gap:** the Fargate task needs *some* body size limit; the planner must choose.
- **Suggestion:** when Bryan fixes a number, update A7 + A17 with it. Alternatively, the MVP plan can land an implementation-level limit and signal product-owner review.
- **Priority:** medium. Required for MVP (Fargate must have a request size limit set), but the choice is reversible.

### 10. F2 (Noun Vocabulary) — could explicitly note that "fabro report" is NOT a CLI command

- **Current state:** F2 names the four canonical CLI commands and notes the pre-2026-04-23 names `fabro debug` (bundle producer) and `fabro report` (uploader) are superseded.
- **Gap:** the goal statement for THIS plan still uses "`fabro report` CLI." This is a real-world drift signal indicating the supersession isn't sticking outside the library.
- **Suggestion:** F2 already covers it adequately (the supersession note is explicit). The drift is downstream — in goal-statement language and possibly elsewhere. Consider whether this is a card-revision signal or a process signal (do plans need a vocabulary lint?).
- **Priority:** low for the card; medium for the process. Not blocking the plan.

## Process / Tooling Signals

### 11. The MVP plan involves cross-repo work

- The four canonical CLI commands live in the parent Fabro repo, not in this repo. A6 L70 names this explicitly.
- **Signal:** the plan structure should accommodate two-repo ticket decomposition. This is a planning-process concern, not a library concern, but Bridget noticed it during assembly.

### 12. Library coverage of "deliberately deferred" items

- Several items are deliberately deferred (per-IP rate limit, metrics, tracing, second region). The library is good at naming them as deferred, but a builder agent reading only one card may not realize the deferral is deliberate. The Anti-Patterns section in the briefing surfaces this; the question is whether the *library* should also have a "Deliberate Absences" or "Deferred" Standard.
- **Priority:** low. The current dispersed-across-cards approach works; a single index card might add value but isn't necessary.

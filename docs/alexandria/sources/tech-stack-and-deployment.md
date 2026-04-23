# Source Material — Tech Stack, Deployment, and Ops

**Status:** raw source material, not a card. Captures the technical-implementation choices Bryan confirmed on 2026-04-23 to unblock Factory-mode implementation.
**Captured:** 2026-04-23
**Source:** direct confirmation from Bryan Helmkamp in response to unblock-interview Group 1 + Group 5.
**Feeds:** potential new System/Decision cards; also informs A5, A7, A8 drafting. Not yet mapped to manifest entries.

---

## Language, Packaging, Deployment

- **Language / runtime:** **Rust only.** No TypeScript. The service is a Rust application end-to-end. This diverges from `bun.report`'s Bun/TypeScript template but aligns with the main Fabro codebase's primary language (`~/p/fabro-sh/fabro` is Rust + TS; this service picks the Rust side only).
- **Frontend:** **Server-side rendered HTML.** No SPA, no client-side framework, no React/Vue/Svelte. The Report Metadata Page is generated server-side by the Rust service and delivered as HTML. Client-side JavaScript is allowed only where it adds real value (e.g., the delete-confirmation interaction).
- **Packaging:** **Docker image.** The service builds to a single image for deployment.
- **Deployment target:** **AWS Fargate.** The Docker image runs as a Fargate task. This is a different target from Fabro itself (which deploys on Fly.io) — chosen presumably because the service is already S3-adjacent and an AWS-native deployment simplifies IAM and networking.
- **Domain:** `fabro.report` is **registered on AWS Route53.** No blocker on domain setup.

## Secrets, IAM, Infrastructure-as-Code

- **Secrets management:** **Environment variables injected into the Fargate task.** Standard Fargate pattern; secrets sourced from AWS Secrets Manager or SSM Parameter Store at task-start time.
- **AWS resource access:** **IAM role attached to the Fargate task.** The service accesses S3 (and any other AWS resources) via the task's IAM role, not via baked-in AWS credentials. This is the correct choice and matches least-privilege best practice.
- **Provisioning:** **Terraform.** All infrastructure (S3 bucket, Fargate service, Route53 records, IAM roles, lifecycle rules) lives in Terraform. Not ClickOps, not CDK, not Pulumi.

## Observability

- **Error reporting: Sentry.** The service's own server-side errors (panics, upload failures, unexpected conditions) flow to Sentry.
- **⚠ Load-bearing distinction against F3.** F3 `Principle - Reject Like bun.report Drift` explicitly rejects "Sentry integration" in its drift list. That rejection is about **integrating Sentry as a downstream processor for Debug Bundle contents** — i.e., treating the Debug Bundles that Reporters upload as crash payloads to forward into Sentry. That pattern is what `bun.report` does and what `fabro.report` emphatically does not do. **This is different.** The Sentry usage here is for the `fabro.report` service's own operational errors; the Debug Bundle contents are never forwarded anywhere, never parsed, and never handed to Sentry. Both of these things can be true simultaneously and they do not conflict. The F4/F5/F6 cards should not be changed; F3's "Sentry integration" drift rejection stands exactly as written because it refers to the bundle-processor use case.
- **Other observability:** not explicitly decided yet. Fargate provides CloudWatch logs by default; Sentry covers errors. Metrics (upload count, storage size, delete count) and tracing are not captured; decide when / if needed.

## Rate Limiting

- **No rate limits for now.** The service is intended for low-volume internal use (self-hosted Fabro users submitting occasional bug reports). Rate-limit later if abuse patterns emerge. Not a day-one need.

## Admin / Ops Surface

- **No admin UI.** The Fabro team's operational surface is the AWS console (S3 browser, Fargate logs, CloudWatch, Sentry dashboard) plus whatever Terraform state visibility the team has. No custom admin interface on `fabro.report` itself.

## Testing Strategy

- **Unit tests** for per-function logic.
- **Integration tests against a mocked S3.** The tests do not require a real AWS account to run; they exercise the full upload / metadata / delete paths against an S3 mock (likely `aws-sdk-rust`'s testing facilities or a local S3-compatible server).
- **E2E against real S3 is not the default testing strategy.** If real-S3 testing is added later, it goes behind a gate (env var or feature flag) to avoid cost and CI complexity.

## Implications Raven Observes (flag for review)

These are direct implications of the confirmed answers above. Sam should treat them as defaults unless Bryan corrects them.

- **The Rust choice is consistent with Fabro itself** and provides dogfooding value — the same team maintains both in the same language.
- **Fargate over Fly is a deliberate AWS-native posture.** The service has a strong reason to be near S3 (IAM role-based S3 access is dramatically simpler when both live in AWS). A Fly.io deployment would require either long-lived AWS access keys in Fly secrets (worse security posture) or cross-cloud federation (more complex). Fargate is the right call.
- **Docker as the packaging format is implicit.** Fargate runs containers; any Fargate service is a Dockerized service. No alternative considered.
- **Terraform for IaC is a team-preference decision.** It is the industry default for AWS and matches the "we control our infrastructure explicitly" posture implied by AWS-native + Fargate + S3.
- **Single-region deployment.** Default region is `us-east-1` (confirmed 2026-04-23), overridable via environment variable for alternate deployments.

## Open Questions (flagged for human judgment)

None of these block Factory-mode implementation but should be resolved in a follow-up:

1. **What's the CI/CD pipeline?** GitHub Actions is likely (Fabro uses it per `~/p/fabro-sh/fabro/.github/workflows/`), but not confirmed for this service.
3. **What Rust web framework?** Axum matches Fabro's `fabro-server`; could also be Actix or Rocket. Axum is the default assumption.
4. **Is there a staging environment** or is production the only environment? Greenfield projects often skip staging; flag if that changes.
5. **Database?** No database has been mentioned for this service. S3 is the only persistent store. Metadata per Report (ID, timestamp, size, message, Fabro version, tags, retention deadline) is presumably stored either in the S3 object's metadata or in a sidecar JSON object in S3. No relational database is implied. Confirm at implementation time.

## What This Source Does Not Cover

- **Specific Rust crate choices** (web framework, HTML templating, S3 SDK, Sentry crate) — implementation detail, not product knowledge.
- **CI/CD pipeline details** — not yet captured.
- **Load and scaling targets** — not specified; assume Fargate's default single-task / small-task-size config is fine for internal use.

## Notes for Future Cards

When this source becomes cards, likely shape:

- **Decision - Rust on AWS Fargate over Bun on Fly** — captures the stack + deployment choice with rejected-alternative rationale (Bun/Fly matches bun.report template; Rust/Fly matches Fabro; Rust/Fargate chosen for AWS-native posture with IAM-role-based S3 access). Rationale the library actually benefits from.
- **Decision - Sentry for Service Errors, Not Bundle Processing** — distinguishes the two uses of Sentry explicitly. Prevents future drift where an agent thinks the F3 Sentry rejection contradicts this choice. Could also be a note in F3 itself.
- **System - Service Topology** — Fargate task, Route53, S3 bucket, IAM role, Terraform repo; how the pieces fit.
- **Standard - Observability** — Sentry for errors; CloudWatch for logs; metrics/tracing deferred.

None of these are in the current manifest. Conan will need to Inventory this source material to decide which cards to add.

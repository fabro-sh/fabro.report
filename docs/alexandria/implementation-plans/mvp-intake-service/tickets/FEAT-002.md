---
id: FEAT-002
title: "Terraform module: complete production infrastructure for fabro.report"
outcome: O-3
tier: must
enabler: false
blocked-by: [FEAT-001]
blocks: [FEAT-003]
cards: [Decision - Service Stack, System - Service Topology, System - Debug Bundle Retention, Storage, and Key Scheme, System - Feedback Storage & Notification]
---

## Motivation

[[System - Service Topology]] commits to declarative-stamp-out: every AWS resource is declared in Terraform with no ClickOps. Building this all in one cohesive ticket avoids the trap of multiple horizontal foundation tickets stacked before the first demoable slice — one Terraform apply puts the entire production infrastructure in place, ready for the application tickets to use.

## Description

Author a Terraform module under `terraform/` and a thin root configuration under `terraform/live/production/` that consumes it. The module declares the complete production stack:

- **Networking:** VPC, public + private subnets across two AZs, internet gateway, NAT gateway (or VPC endpoints if cheaper for `us-east-1` S3-only egress), security groups for the ALB and Fargate task.
- **Compute:** ECS cluster, Fargate task definition (referencing the ECR image tag via variable), ECS service with one desired task, task execution role, task IAM role.
- **Load balancing + DNS + TLS:** Application Load Balancer with HTTPS listener, target group on port 8080, ACM certificate for `fabro.report`, Route53 A-record alias to the ALB.
- **Container registry:** ECR repository for the service image with lifecycle policy retaining the last N images.
- **Storage:** Single private S3 bucket (default name `fabro-report-bundles`, env-overridable) with public-access block, versioning disabled, two prefix-scoped lifecycle rules (30-day expiration on the bare-prefix Reports tree and on `feedback/`).
- **Identity:** IAM task role with policies granting the minimum needed surface — `s3:PutObject` / `s3:DeleteObject` / `s3:GetObject` / `s3:HeadObject` on the Reports prefix subtree; `s3:PutObject` on the `feedback/` prefix subtree; `s3:ListBucket` (scoped to the bucket); `secretsmanager:GetSecretValue` (or `ssm:GetParameter`) on the two named secrets.
- **Secrets:** Secrets Manager (or SSM Parameter Store) entries for `SENTRY_DSN` and `FEEDBACK_DISCORD_WEBHOOK_URL`. Values are populated out-of-band (via the AWS console or `aws secretsmanager update-secret`) — Terraform manages only the resource, not the value.

`us-east-1` is the default region; bucket name and region are variables so they can be overridden.

## Context

- [[Decision - Service Stack]] pins the entire stack and explicitly forbids ClickOps, CDK, Pulumi, multi-region, SPA, and non-Route53 DNS.
- [[System - Service Topology]] enumerates every operational piece this module declares; cross-reference it during review to ensure nothing is added that violates Narrow Forever (no RDS, no SQS, no autoscaling complexity, no second microservice, no WAF).
- [[System - Debug Bundle Retention, Storage, and Key Scheme]] owns the bucket layout: `<YYYY>/<MM>/<DD>/<id>/{bundle.tar.gz,metadata.json}` for Reports.
- [[System - Feedback Storage & Notification]] owns the `feedback/` prefix layout: `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/{feedback.json,feedback.txt,attachments/...}`.
- The lifecycle rules are the entire retention mechanism — there is no application-side cron, no sweeper job. Per [[Decision - 30-Day Retention Default]], 30 days is fixed for both prefixes.
- The Discord webhook URL is created in Discord's UI, not by Terraform; Terraform manages only the secret holding it.
- Anti-patterns to avoid: public-read bucket ACL; per-team or per-Reporter S3 prefixes; AWS keys baked into the task definition; multi-region active-active; adding a WAF or a second non-AWS egress beyond Sentry + Discord.

## Acceptance Criteria

- [ ] `terraform/` directory contains a reusable module with `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`.
- [ ] `terraform/live/production/` contains a thin root configuration that calls the module with production inputs.
- [ ] `terraform plan` against `terraform/live/production/` shows the complete resource set with no diff after `apply`.
- [ ] S3 bucket has `block_public_acls`, `block_public_policy`, `ignore_public_acls`, `restrict_public_buckets` all set to `true`.
- [ ] Two `aws_s3_bucket_lifecycle_configuration` rules: one for the Reports prefix (no prefix or empty prefix scoped against the bucket root, exclusive of `feedback/`), one for `feedback/`. Both with `expiration.days = 30`.
- [ ] IAM task role policies enumerate explicit `Action` + `Resource` ARNs; no `*` actions; no `*` resources except where unavoidable (e.g., `arn:aws:logs:*:*:*` for log group autocreation if used).
- [ ] ACM certificate is validated via DNS in Route53; ALB HTTPS listener uses it.
- [ ] No resource in the module uses `lifecycle { ignore_changes = ... }` to mask drift; if drift is acceptable, document why in code comments.
- [ ] Module accepts variables for `bucket_name`, `region`, `domain_name`, `image_tag`, and the Sentry DSN / Discord webhook secret ARNs (passed through, not created with values).

## Implementation Notes

- Use AWS provider ≥ 5.0. Pin the version in `versions.tf`.
- Prefer `aws_iam_policy_document` data sources over inline JSON for IAM policies — easier to review.
- For the lifecycle rule covering "Reports but not `feedback/`," use the rule's `filter.prefix = ""` (or omit `prefix`) and rely on a separate higher-priority rule for `feedback/` — verify with a `terraform plan`-only test that both prefixes get distinct rules.
- The Fargate task definition references the image as `<ecr-uri>:<image_tag>` where `image_tag` is a variable; CI/CD (FEAT-003) will supply the tag from the GitHub Actions workflow.
- Defer remote state backend setup (S3 + DynamoDB lock) to the implementer's preference; document the choice in `terraform/README.md`.
- Document in `terraform/README.md`: how to populate the secret values after `apply`, how to register the `fabro.report` domain (assumed already registered or do it manually), and how to bootstrap the remote state.

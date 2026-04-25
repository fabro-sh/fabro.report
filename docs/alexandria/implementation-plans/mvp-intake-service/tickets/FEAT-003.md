---
id: FEAT-003
title: "GitHub Actions: build → ECR → terraform apply pipeline with PR plan gate"
outcome: O-3
tier: must
enabler: false
blocked-by: [FEAT-002]
blocks: [FEAT-004, FEAT-009, FEAT-011, FEAT-012]
cards: [Decision - Service Stack, System - Service Topology]
---

## Motivation

Once the Terraform module exists, deployments must be automated to honor the declarative-stamp-out commitment. Without CI/CD, the first manual `terraform apply` sets a precedent for human-in-the-loop deploys that would erode the no-ClickOps posture. This ticket is the bridge from "infrastructure is declared" to "infrastructure ships continuously." After this ticket completes, an empty service is reachable at `https://fabro.report` — the first deployable milestone.

## Description

Set up two GitHub Actions workflows:

1. **PR plan workflow** — on PRs touching `terraform/**` or the Rust service, run `terraform fmt -check`, `terraform validate`, and `terraform plan` against `terraform/live/production/`. Post the plan output as a PR comment. Do NOT apply.
2. **Main deploy workflow** — on merges to `main`, build the Docker image with the commit SHA as the tag, push to ECR, then `terraform apply` with the new image tag, then run a smoke test (`curl https://fabro.report/health` expecting 200).

Use OIDC federation (`aws-actions/configure-aws-credentials`) to obtain temporary AWS credentials for the workflows — no long-lived `AWS_ACCESS_KEY_ID` in repo secrets.

## Context

- [[Decision - Service Stack]] notes "GitHub Actions is assumed, per Fabro's existing pattern, but not locked on this card." This ticket commits to it.
- [[System - Service Topology]] requires Terraform to be the single source of truth for AWS state; the workflow must apply the module on every change rather than allowing console edits.
- The `terraform plan` output on PRs gives a manual gate before production change — it doubles as the security review checkpoint per [[Decision - Service Stack]]'s three-question test.
- Anti-pattern: do NOT skip the plan step for "trivial" changes; every infra change goes through the same review path.
- Anti-pattern: do NOT cache the Docker layer to `:latest` and apply by tag mutation — image tags must be commit-SHA-pinned for rollback safety.

## Acceptance Criteria

- [ ] `.github/workflows/terraform-plan.yml` runs on PRs and posts plan output as a comment.
- [ ] `.github/workflows/deploy.yml` runs on `push` to `main`, builds + pushes the image with the commit SHA as the tag, applies Terraform with the new tag, and runs a smoke test.
- [ ] AWS credentials are obtained via OIDC; no static keys in repo secrets.
- [ ] First successful run of `deploy.yml` results in `curl https://fabro.report/health` returning 200 OK from a public network.
- [ ] The smoke test failure aborts the workflow with a non-zero exit; partial deploys do not silently succeed.
- [ ] A README section under `terraform/` or `.github/` documents how to bootstrap the OIDC trust policy in AWS the first time.

## Implementation Notes

- Use `aws-actions/configure-aws-credentials@v4` with `role-to-assume`. Document the IAM role + trust policy that GitHub assumes (this is a one-time bootstrap; can be applied via Terraform from a separate `terraform/live/bootstrap/` workspace if desired).
- Use `hashicorp/setup-terraform@v3` with the same Terraform version pinned in `terraform/versions.tf`.
- For the smoke test, retry up to ~60s with backoff — Fargate task replacement can take a moment after `terraform apply`.
- Build the Docker image with `docker buildx build --platform linux/amd64` to ensure compatibility with Fargate's default x86_64 architecture (unless the task definition is set to ARM64).
- Defer staging to a follow-up plan — this MVP uses production-only per the planning decision.

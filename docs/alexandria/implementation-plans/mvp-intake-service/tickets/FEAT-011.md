---
id: FEAT-011
title: "Sentry init at startup with redaction guard against payload bytes"
outcome: O-4
tier: should
enabler: false
blocked-by: [FEAT-003]
blocks: [FEAT-013]
cards: [Decision - Sentry for Service Errors, Not Bundle Processing, System - Service Topology]
---

## Motivation

[[Decision - Sentry for Service Errors, Not Bundle Processing]] is unusual in the library: it both forbids a use (Sentry-as-bundle-processor) and names a positive use (service errors only). Implementing the affirmative case correctly — including the redaction guard so payload bytes never escape — keeps the service operable in production without violating the library's strict no-content-leak posture.

## Description

Initialize Sentry at service startup using a DSN injected from Secrets Manager (or SSM Parameter Store) via the `SENTRY_DSN` env var. The Rust integration installs a panic handler automatically. Add explicit `sentry::capture_error`-style calls at the four request handlers:

- `POST /v1/uploads` — capture S3 write failures, presigned-URL generation failures, body-parse failures.
- `GET /r/<id>` — capture S3 read failures, template render failures.
- `DELETE /r/<id>` — capture S3 delete failures.
- `POST /v1/feedback` — capture multipart parse failures, S3 write failures (Discord failures land in FEAT-013 with their own self-report path).

Implement a redaction guard layer (a `BeforeSend` hook in the Sentry SDK) that scrubs:

- Any field named `body`, `payload`, `bytes`, `content`, `data`, `attachment` from event metadata.
- Any string longer than ~1024 chars (heuristic for "this might be a payload byte string").
- The `feedback.txt` content under any name.

Document explicit deny-list of fields in code so the redaction is reviewable.

The Sentry event metadata SHOULD include: service name (`fabro-report`), version (commit SHA), deployment env (`production`), request path, request method, error type, stack trace.

## Context

- [[Decision - Sentry for Service Errors, Not Bundle Processing]] — captures: panics, upload-coordination failures, page-render failures, delete failures, feedback-intake failures, Discord webhook double-failure (handled by FEAT-013). Forbids: forwarding Debug Bundle bytes; forwarding Feedback bytes; forwarding `feedback.txt`; forwarding attachment bytes; surfacing Sentry IDs to Reporters or Feedback Submitters.
- [[System - Service Topology]] — Sentry DSN comes from Secrets Manager (or SSM) via `SENTRY_DSN` env var, injected at task-start. NEVER baked into Docker image, NEVER in source.
- [[Decision - IP Logging for Abuse Protection]] — IP MUST NOT be in Sentry events. Add it to the redaction deny-list explicitly.
- Anti-patterns to avoid: forwarding any payload bytes to Sentry; surfacing Sentry event IDs to users in CLI output, page content, or Discord posts ("please report this crash with ID xyz" is the wrong shape); including IP in Sentry events; using Sentry as a log aggregator (it's an error tracker; logs go to CloudWatch).

## Acceptance Criteria

- [ ] Sentry initializes at startup using the `SENTRY_DSN` env var; if the env var is unset, the service still starts but logs a warning (don't hard-fail in dev environments without a DSN).
- [ ] Panics in any request handler are captured automatically by the Sentry panic handler.
- [ ] Each of the four handlers has explicit error-capture calls in its error paths.
- [ ] The `BeforeSend` redaction hook scrubs the named fields and any string > 1024 chars from event metadata.
- [ ] No Sentry event IDs are surfaced to the user in CLI output, page content, or Discord posts.
- [ ] No payload bytes (Debug Bundle bytes, Feedback message text, attachment bytes) appear in any Sentry event in a deliberate test scenario.
- [ ] No client IP appears in any Sentry event.
- [ ] Sentry events are tagged with `environment:production` and `release:<commit-sha>`.

## Implementation Notes

- Crate: `sentry` with the `panic` + `tracing` features for automatic panic capture and `tracing` integration.
- Init Sentry in `main.rs` BEFORE the Axum router builds, so any startup panic is also captured.
- The `BeforeSend` hook lives in the `sentry::ClientOptions::before_send` callback. It receives an `Event` and returns `Option<Event>` — return `None` to drop, or modify-and-return.
- Define the redaction field deny-list as a `const &[&str]` in code so it's grep-able.
- Test the redaction with a deliberate test scenario: trigger an error path that includes a long string, verify the Sentry event (in the Sentry UI's debug mode or a local `sentry-cli` test) does not include it.
- Defer Sentry tracing/performance monitoring to a follow-up plan — error tracking only in MVP per [[System - Service Topology]] and Narrow Forever.

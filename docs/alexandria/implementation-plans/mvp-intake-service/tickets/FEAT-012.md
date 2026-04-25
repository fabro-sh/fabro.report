---
id: FEAT-012
title: "Structured JSON logging, healthcheck refinement, robots.txt"
outcome: O-4
tier: should
enabler: false
blocked-by: [FEAT-003]
blocks: [FEAT-014]
cards: [System - Service Topology, Decision - Sentry for Service Errors, Not Bundle Processing, Component - Report Metadata Page]
---

## Motivation

CloudWatch is the entire log-aggregation surface per [[System - Service Topology]]. Structured JSON logs make CloudWatch Logs Insights queries useful (filter by request ID, status, error type) without adding a metrics or tracing layer. The `robots.txt` and healthcheck refinement are small but load-bearing — `robots.txt` prevents capability URLs from being crawler-indexed, and the healthcheck shape determines whether the ALB target group ever marks the task healthy.

## Description

Replace the FEAT-001 default `tracing-subscriber` formatter with a JSON formatter that emits one log line per request with: timestamp, level, request ID (generated per-request via `uuid` or a tower middleware), method, path, status, duration in ms, error type (when applicable). All four request handlers contribute to this; add a tower middleware that wraps the router.

Refine `/health`:

- Returns 200 OK with a minimal JSON body `{"status":"ok"}` (already in FEAT-001).
- Does NOT include service version, deployment env, or any debug info — keep it boring per [[System - Service Topology]] ("no custom healthcheck wiring beyond the Fargate service default").
- Excluded from request logging (don't emit a log line per healthcheck — too noisy).

Add `/robots.txt`:

- Returns `User-agent: *\nDisallow: /\n` with `Content-Type: text/plain`.
- The receipt page (FEAT-005) already includes `<meta name="robots" content="noindex,nofollow">` as belt-and-suspenders.

## Context

- [[System - Service Topology]] — CloudWatch receives Fargate stdout/stderr; Sentry receives errors. No metrics/tracing in the default topology. Healthcheck is "no custom wiring beyond Fargate default."
- [[Decision - Sentry for Service Errors, Not Bundle Processing]] — log lines must NOT include payload bytes (Debug Bundle bytes, Feedback message text, attachment bytes). Same redaction discipline as FEAT-011's Sentry redaction guard.
- [[Component - Report Metadata Page]] — capability URLs must not be indexed; `robots.txt` plus the page-level `<meta>` cover both robot-respecting crawlers and (best-effort) ones that ignore robots.txt.
- Anti-patterns to avoid: logging request bodies; logging the Feedback message text; logging Debug Bundle metadata fields beyond IDs; logging the Discord webhook URL or Sentry DSN; making `/health` informative (a public debug endpoint is a leak); allowing `/robots.txt` to allow `/r/` (capability URLs leaking to crawlers is a security concern).

## Acceptance Criteria

- [ ] One JSON log line per request appears in CloudWatch with: `timestamp`, `level`, `request_id`, `method`, `path`, `status`, `duration_ms`. Errors include `error_type`.
- [ ] No log line contains payload bytes: a deliberate test that submits a Feedback with the message text "DO_NOT_LOG_THIS_TEXT" must NOT produce a log line containing that string.
- [ ] `/health` returns 200 OK and is excluded from request logging.
- [ ] `/robots.txt` returns `User-agent: *\nDisallow: /\n` with `Content-Type: text/plain`.
- [ ] The receipt page still includes the `<meta name="robots" content="noindex,nofollow">` tag.
- [ ] Log level is configurable via env var (`RUST_LOG`); default is `info` in production.

## Implementation Notes

- Use `tracing` + `tracing-subscriber` with the `json` formatter. The default formatter from FEAT-001 was likely `pretty` or `compact` — swap it.
- Request ID middleware: use `tower-http`'s `RequestIdLayer` or write a small middleware that generates a UUID per request and adds it to the tracing span context.
- For `/health` log exclusion: configure the request-logging middleware with a path filter, or skip logging when `path == "/health"`.
- `/robots.txt` can be served as a static `axum::response::Response` with the body inline — no need to put it on disk.
- Document the log structure in `README.md` so a future ops engineer knows what to query.

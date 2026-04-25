---
id: FEAT-009
title: "POST /v1/feedback: multipart parse, UUIDv7, S3 writes, IP capture"
outcome: O-2
tier: must
enabler: false
blocked-by: [FEAT-003]
blocks: [FEAT-010]
cards: [Component - Feedback, System - Feedback Storage & Notification, Decision - Fire-and-Forget Feedback, Decision - IP Logging for Abuse Protection, Standard - Interaction Patterns (CLI + Delete)]
---

## Motivation

This is the entire HTTP surface for the Feedback flow — exactly one route. After this ticket, the service can persist Feedback to S3 with proper IP capture; FEAT-010 adds the Discord notification side. The fire-and-forget posture, IP-logging-for-abuse-protection-only scope, and asymmetry vs. the Report flow are all load-bearing — most of the implementation work here is *not adding* things (no GET endpoint, no DELETE endpoint, no URL response).

## Description

Implement `POST /v1/feedback` accepting `multipart/form-data` with the following parts:

- `text` (optional, string) — the message text. Goes into `feedback.txt`.
- `email` (optional, string) — opaque, no format validation at this layer.
- `fabro_version`, `platform`, `arch` (required strings) — captured into `feedback.json`.
- `attachments[]` (optional, multiple files) — written under `attachments/` in the S3 prefix.

The handler:

1. Parses the multipart body. Reject with 400 if malformed or if the total body size exceeds 25 MiB.
2. Generates a UUIDv7 ID via the `uuid` crate's `v7` feature.
3. Captures the client IP from the `X-Forwarded-For` header (last entry, since the ALB sets it) or the request remote address as fallback.
4. Constructs `feedback.json` with: `id`, `ip`, `email`, `fabro_version`, `platform`, `arch`, `timestamp`, `retention_deadline` (timestamp + 30 days), `attachments` manifest (array of `{ filename, size }` objects). Per [[Component - Feedback]], no other fields.
5. Writes three (or more) S3 objects under `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/`: `feedback.json` (the envelope), `feedback.txt` (just the message text — empty file if no text was provided), and `attachments/<safe-filename>` for each attachment.
6. Returns 200 with a CLI-shaped confirmation body — plain text or minimal JSON, in the calm tone register. NO URL, NO ID, NO link.

Discord notification is OUT of scope for this ticket — that lands in FEAT-010. Errors return JSON `{ "error": { "message", "code" } }`.

## Context

- [[Component - Feedback]] specifies the field set and the immutability of the Feedback after creation.
- [[System - Feedback Storage & Notification]] specifies the key layout `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/{feedback.json,feedback.txt,attachments/...}` and the failure-self-reporting semantics (FEAT-013 implements those).
- [[Decision - Fire-and-Forget Feedback]] — no URL returned, no ID surfaced, no rendered page, no GET, no DELETE. The CLI confirmation IS the entire post-submission contract.
- [[Decision - IP Logging for Abuse Protection]] — IP goes to `feedback.json` and CloudWatch only; never to Sentry (per [[Decision - Sentry for Service Errors, Not Bundle Processing]]); never analytics; never hashed at rest; never to a separate IP table.
- [[Standard - Interaction Patterns (CLI + Delete)]] — the wire format is multipart per the planning decision; size cap is 25 MiB at the Fargate layer per the planning decision (sized to Discord's webhook attachment ceiling).
- [[Standard - Noun Vocabulary]] Usage Rule 6 — Feedback ≠ Report. Use UUIDv7 (time-ordered, NOT a capability token); do not treat it as a secret.
- Anti-patterns to avoid: returning a URL ("here's where you can see your feedback" is forbidden); returning the UUIDv7 ID; implementing `GET /v1/feedback/:id` for any consumer; implementing `DELETE /v1/feedback/:id`; capturing IP for any purpose beyond abuse protection; putting IP in Sentry events; writing IP to a DynamoDB table for "recent submissions"; using presigned S3 URLs for Feedback (Feedback uses direct POST — bytes traverse Fargate by design); calling a Feedback a "Report" or vice versa anywhere in code or copy.

## Acceptance Criteria

- [ ] `POST /v1/feedback` with valid multipart body returns 200 with a CLI-shaped confirmation that contains NO URL, NO ID, NO link.
- [ ] `feedback.json` is written to S3 at `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/feedback.json` with all required fields.
- [ ] `feedback.txt` is written (empty if no message).
- [ ] Attachments are written to `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/attachments/<safe-filename>` (filename sanitized — strip directory traversal, normalize whitespace).
- [ ] Client IP is captured to `feedback.json` from `X-Forwarded-For` (last hop, the original client) — verify against an ALB-fronted test request.
- [ ] No `GET /v1/feedback/:id` route exists (verify with router introspection).
- [ ] No `DELETE /v1/feedback/:id` route exists.
- [ ] Body size > 25 MiB returns 400 with the JSON error envelope; smaller bodies parse normally.
- [ ] Two consecutive identical Feedback submissions produce two distinct UUIDv7s and two distinct S3 prefixes (no dedup).
- [ ] CloudWatch logs include a structured line with the UUIDv7 + IP + timestamp for each submission.

## Implementation Notes

- Crates: `axum::extract::Multipart`, `uuid` (with `v7` + `serde` features), `serde`, `serde_json`, `chrono` or `time` (for timestamps).
- Set Axum's `DefaultBodyLimit` to 25 MiB on the `/v1/feedback` route specifically (not globally — other routes have different limits).
- Filename sanitization: strip path separators, NUL bytes, and leading dots; normalize whitespace; cap at ~255 chars; if the result is empty, use a fallback like `attachment-<index>`.
- The IP-extraction logic should NOT trust `X-Forwarded-For` blindly in dev — but in production behind the ALB, the ALB sets it and overwrites client-supplied values. Document this in code.
- All S3 writes happen sequentially in this ticket — the failure-handling refinement (FEAT-013) will refactor this if needed.
- If any S3 write fails, return 500 to the CLI without posting Discord (Discord work happens in FEAT-010); roll-forward cleanup is acceptable since the 30-day lifecycle catches partials.
- Defer per-IP rate limiting per [[System - Feedback Storage & Notification]] and [[Decision - IP Logging for Abuse Protection]] — explicitly NOT in MVP.

---
id: FEAT-013
title: "Discord failure self-reporting: second call + Sentry escape on second failure"
outcome: O-5
tier: should
enabler: false
blocked-by: [FEAT-010, FEAT-011]
blocks: [FEAT-014]
cards: [System - Feedback Storage & Notification, Decision - Sentry for Service Errors, Not Bundle Processing]
---

## Motivation

[[System - Feedback Storage & Notification]] specifies an unusual but deliberate failure-handling pattern: ONE retry, but the retry payload self-describes (carries the original payload + inputs + error text), and if THAT also fails, Sentry is the escape. This avoids both silent loss (Feedback on S3 but invisible) and unbounded retry/backoff/queueing (which would force a topology change). Implementing it now while the Discord code is fresh prevents a future bolted-on retry layer that would erode the no-queue posture.

## Description

Replace the FEAT-010 happy-path-only Discord failure path with the full self-report mechanism:

1. **Primary call:** POST the component-v2 payload to `FEEDBACK_DISCORD_WEBHOOK_URL`.
2. **On primary failure** (non-2xx, network error, timeout > 10s):
   - Construct a self-report payload containing: a recognizable failure-self-report header (distinct embed style and title prefix, e.g., "⚠ Feedback notification failed"), an error-text summary of the primary failure (status code, body excerpt, error string), the original `discord-payload.json`, and a sanitized `inputs.json` describing the Feedback metadata (UUIDv7, IP, email, sysinfo, attachments manifest — NOT the message text or attachment bytes).
   - POST the self-report to the same `FEEDBACK_DISCORD_WEBHOOK_URL`.
3. **On self-report failure** (also non-2xx / network / timeout):
   - Emit a Sentry event tagged with the failure context (Feedback UUIDv7, primary error, self-report error).
   - Write a structured CloudWatch log line at `ERROR` level with the same context.
   - Do NOT attempt a third retry. Do NOT queue.
4. The CLI confirmation returned to the Feedback Submitter is identical regardless of which path executed — Submitter has no Discord-failure-aware affordance, consistent with [[Decision - Fire-and-Forget Feedback]].
5. The S3 writes are NOT rolled back on Discord failure — `feedback.json` + `feedback.txt` + attachments remain on S3 as the source of truth; Triagers can locate them via the self-report's S3 prefix path or via the Sentry event.

## Context

- [[System - Feedback Storage & Notification]] — full failure-handling spec. Read this card directly during implementation; the structure of the self-report payload is named there.
- [[Decision - Sentry for Service Errors, Not Bundle Processing]] — Sentry is the escape for the double-failure case but MUST NOT include payload bytes (apply the FEAT-011 redaction guard; the Sentry event for Discord double-failure carries IDs, error text, and the inputs.json — no message text, no attachment bytes).
- [[Decision - IP Logging for Abuse Protection]] — IP appears in `inputs.json` (it's already in `feedback.json`); the self-report payload reaches Discord via the same webhook, so the same Triager-reading audience that sees normal Feedback notifications sees the self-report. This is intentional: the Triager needs the IP to investigate if abuse is the cause of the failure.
- [[Decision - Fire-and-Forget Feedback]] — the CLI confirmation is identical; do not surface Discord failure to the Submitter.
- Anti-patterns to avoid: more than ONE retry; exponential backoff; queueing; using a sidecar process; rolling back S3 writes on notification failure (S3 is the source of truth, Discord is the notification channel); surfacing Sentry event IDs to the Submitter; including the Feedback message text or attachment bytes in the self-report payload OR the Sentry event.

## Acceptance Criteria

- [ ] On primary Discord failure, a single second webhook call is made to the same URL with the self-report payload structure (header, error-text summary, `discord-payload.json`, `inputs.json`).
- [ ] On second Discord failure, a Sentry event is emitted (tagged with the failure context) AND a CloudWatch ERROR log line is written. No third retry.
- [ ] No exponential backoff between primary and secondary call (single fixed timeout, e.g., both calls have a 10s timeout each).
- [ ] The self-report payload's `inputs.json` includes the Feedback's UUIDv7, IP, email, sysinfo, attachments manifest — NOT the message text, NOT attachment bytes.
- [ ] The Sentry event for double-failure does NOT include the message text or attachment bytes (apply the FEAT-011 redaction guard).
- [ ] The CLI confirmation is identical regardless of which failure path fired.
- [ ] S3 objects are NOT deleted on Discord failure — verified by checking the bucket after a deliberately-broken-webhook test.
- [ ] The self-report's visual treatment (Discord embed color, title prefix) makes it visually distinct from a normal Feedback notification when a Triager scans the Discord channel.

## Implementation Notes

- Refactor the FEAT-010 helper that constructs the primary payload so it can be reused inside the self-report's `discord-payload.json` field.
- The `inputs.json` is a separate JSON object embedded inside the self-report payload — could be sent as an attachment to the Discord webhook (Discord supports webhook attachments) OR inlined as a code-block field. Recommend inline code-block for visibility; no need for an actual attachment.
- The 10-second timeout per call is the configurable bound; use `reqwest::ClientBuilder::timeout`.
- Test scenario: temporarily set `FEEDBACK_DISCORD_WEBHOOK_URL` to a `https://httpstat.us/500` endpoint to force primary failure; verify the self-report fires; then point both at it to verify the Sentry escape fires.

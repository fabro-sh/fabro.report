---
id: FEAT-010
title: "Discord webhook component-v2 payload + happy-path POST after S3 writes"
outcome: O-2
tier: must
enabler: false
blocked-by: [FEAT-009]
blocks: [FEAT-013]
cards: [System - Feedback Storage & Notification, Standard - Competitive Context, Principle - Minimal Public Surface Feels Calm]
---

## Motivation

Discord is the entire pickup surface for the Triager on the Feedback flow — without the webhook post, a Feedback that lands on S3 is invisible until someone manually browses the bucket. This ticket implements the happy-path notification only; the failure-self-reporting layer (second call + Sentry escape) lands in FEAT-013 to keep this ticket focused.

## Description

After the S3 writes in FEAT-009 succeed, construct a Discord component-v2 payload and POST it to `FEEDBACK_DISCORD_WEBHOOK_URL`. The payload structure is calibrated against `bun feedback`'s `sendToDiscord` per [[Standard - Competitive Context]] and includes:

- **Header:** a brief title ("New Fabro feedback") and the submission timestamp.
- **Sysinfo block:** Fabro version, platform, arch, optional email (rendered as `<no email>` if absent).
- **Body:** the message text from `feedback.txt`, rendered as a code block or quote so multiline content displays cleanly.
- **Attachments tree:** a list of attachment filenames + sizes; NOT the attachment bytes themselves (Discord webhooks support file attachments but the asymmetry between the S3 source-of-truth and the Discord copy is undesirable — Triagers retrieve attachments from S3 directly).
- **Footer:** a small marker indicating the submission UUIDv7 + the S3 prefix path so the Triager can locate the source-of-truth in S3 with their own AWS credentials.

The webhook URL is read from `FEEDBACK_DISCORD_WEBHOOK_URL` env var (injected by Fargate from Secrets Manager). The POST is synchronous within the request lifecycle — the CLI confirmation does not return until the Discord call either succeeds or the failure-handling path fires.

For this ticket, on Discord failure: log the failure to stderr/CloudWatch and return success to the CLI anyway (FEAT-013 will replace this with the proper self-reporting path). This keeps the happy path testable without the failure-self-reporting complexity.

## Context

- [[System - Feedback Storage & Notification]] — the entire mechanism: write S3 first, then post Discord, in one synchronous request lifecycle. No queue. The component-v2 payload tree is named (header / sysinfo / body / attachments / footer) but the exact JSON keys are implementation detail.
- [[Standard - Competitive Context]] (C1) — `bun feedback`'s `sendToDiscord` is the pattern reference. The borrowing is at the same narrow layer as `bun.report`-the-URL-shape borrowing.
- [[Principle - Minimal Public Surface Feels Calm]] — applies to the Discord post text just as much as to the CLI: no emoji cascade, no URGENT framing, no "🎉 NEW FEEDBACK!", calm `bun`-calibrated voice.
- [[Decision - IP Logging for Abuse Protection]] — IP MUST NOT be in the Discord post (it's strictly a server-log + `feedback.json` field). The Triager retrieves IP from S3 if needed for abuse investigation.
- Anti-patterns to avoid: emoji cascade in the post text; URGENT/CRITICAL framing; including the attachment bytes in the Discord payload (use the attachments tree summary instead); including IP in the Discord post; including the `feedback.txt` raw text without code-block formatting; adding multiple webhook URLs (single notification target per intake — [[System - Service Topology]]).

## Acceptance Criteria

- [ ] After a successful `POST /v1/feedback`, a Discord component-v2 payload is POSTed to `FEEDBACK_DISCORD_WEBHOOK_URL`.
- [ ] The payload includes header / sysinfo / body / attachments tree / footer per the structure named in [[System - Feedback Storage & Notification]].
- [ ] The post text register reads as calm and `bun`-calibrated; no emoji cascade, no URGENT framing.
- [ ] Attachment bytes are NOT in the payload; only filename + size in the attachments tree.
- [ ] IP is NOT in the Discord post.
- [ ] On Discord failure (this ticket only): log to stderr/CloudWatch and return success to CLI; FEAT-013 will replace this with the self-reporting path.
- [ ] The Discord call timeout is bounded (e.g., 10 seconds) so a slow webhook does not block the CLI confirmation indefinitely.
- [ ] A test submission produces exactly one Discord post per Feedback.

## Implementation Notes

- Use `reqwest` (with `rustls-tls` to avoid OpenSSL build complexity in the Docker image) for the HTTPS POST.
- Discord component-v2 schema reference: `https://discord.com/developers/docs/resources/message#message-object` — embed-style components are sufficient; do not need full message-component v2 (the payload is a webhook message).
- Construct the payload as a `serde_json::Value` or a typed struct; either is fine. Pin the JSON shape in a single helper function so FEAT-013 can reuse it for the self-report.
- Truncate the message body if it exceeds Discord's per-embed character limit (~6000 chars total across embeds). Add an ellipsis + footer note "(truncated; full text in S3)".
- The S3 prefix path in the footer is for the Triager's manual S3 lookup; format it as a plain string the Triager can paste into `aws s3 ls s3://<bucket>/<prefix>/`.
- Defer rate-limiting / Discord-API-throttling backoff to FEAT-013.
- The `bun feedback` reference: see `~/p/oven-sh/bun.report` if it's accessible — that's the visual / structural model.

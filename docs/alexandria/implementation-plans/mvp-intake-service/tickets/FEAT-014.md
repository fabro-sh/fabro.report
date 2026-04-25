---
id: FEAT-014
title: "Security review checklist: validate the deployed system's security properties"
outcome: O-3
tier: must
enabler: false
blocked-by: [FEAT-008, FEAT-012, FEAT-013]
blocks: []
cards: [Decision - Capability URLs over Auth, System - Debug Bundle Retention, Storage, and Key Scheme, System - Service Topology, Decision - Service Stack, Decision - IP Logging for Abuse Protection]
---

## Motivation

The library specifies several security properties that are load-bearing — capability-URL entropy, IAM least-privilege, secrets handling, bucket privacy, TLS — but each has a failure mode where the property is asserted in the cards yet not actually held in the running system. Without an explicit validation gate, a regression goes unnoticed until it's exploited. This ticket is the gate that confirms the deployed system actually holds the properties it claims to.

## Description

Run a security review checklist against the deployed production system and the Terraform module. Document findings in `docs/agent/reviews/security-review-mvp-<date>.md`. For each item, capture: the property being verified, the verification command/method, the result (pass/fail/concern), and the remediation if the result is fail or concern.

Checklist items:

1. **Report ID entropy** — generate 100 Report IDs via `POST /v1/uploads`; verify they are 32 lowercase hex characters; spot-check entropy is uniform (no pattern, no sequential).
2. **Presigned PUT URL TTL** — issue a presigned URL via `POST /v1/uploads`; wait 16 minutes; verify the URL rejects the upload (403 SignatureExpired or equivalent).
3. **Presigned PUT `Content-Length` constraint** — attempt an upload with `Content-Length: 60000000` (60 MB > 50 MiB cap); verify S3 rejects it.
4. **IAM least-privilege** — read the IAM task role policy from Terraform; verify it grants only the actions enumerated in O-3's validation criteria; no `s3:*`, no `*` resources except where unavoidable; no policies attached beyond what the module declares.
5. **Bucket privacy** — `aws s3api get-public-access-block --bucket <name>` returns all four flags `true`; `aws s3api get-bucket-acl --bucket <name>` shows no `AllUsers` or `AuthenticatedUsers` grants; a direct `aws s3api get-object` call with no auth returns AccessDenied.
6. **TLS termination** — `curl -v https://fabro.report/health` shows a valid TLS handshake with the ACM-issued certificate; `curl http://fabro.report/health` returns 301 to https or rejects.
7. **No AWS keys in Docker image** — `docker run --entrypoint /bin/sh <image-tag> -c 'env | grep -i aws'` shows no `AWS_ACCESS_KEY_ID` or `AWS_SECRET_ACCESS_KEY`; `docker run --entrypoint /bin/sh <image-tag> -c 'find / -name "*.aws*" 2>/dev/null'` returns nothing.
8. **No secrets in source** — grep the repo for `sk_live_`, `xoxb-`, `discord.com/api/webhooks/`, the Sentry DSN domain — all must return zero matches.
9. **No secrets in Terraform state plaintext** — verify the remote state backend uses encryption-at-rest; verify the secret values are not stored as Terraform-managed values (they are populated out-of-band per FEAT-002).
10. **IP-in-Sentry-or-Discord-post check** — trigger a Feedback submission with a known test IP; verify the IP appears in `feedback.json` + CloudWatch but NOT in any Sentry event metadata and NOT in the Discord post.
11. **Payload-bytes-in-logs/Sentry check** — trigger a Feedback with the message text "SENSITIVE_TEST_MARKER_<random>"; verify the marker appears in `feedback.txt` on S3 but NOT in CloudWatch logs and NOT in any Sentry event.
12. **Capability URL not indexed** — `curl https://fabro.report/robots.txt` returns `Disallow: /`; the receipt page returns `<meta name="robots" content="noindex,nofollow">`.
13. **Webhook URL rotation feasibility** — confirm the Discord webhook URL can be rotated by updating the secret value (manual or `aws secretsmanager update-secret`) without redeploying; document the rotation procedure.
14. **Generic 404 for invalid IDs** — confirm `GET /r/<31-chars>`, `GET /r/<33-chars>`, `GET /r/UPPER_HEX`, `GET /r/contains-g` all return the same generic 404 page (no error message that distinguishes "malformed" from "not found").

## Context

- [[Decision - Capability URLs over Auth]] — URL entropy is structural (128 bits CSPRNG); insufficient entropy collapses the access model. Item 1 + 14 verify this.
- [[System - Debug Bundle Retention, Storage, and Key Scheme]] — bucket privacy is non-negotiable. Items 5 + 12 verify this.
- [[System - Service Topology]] — IAM least-privilege; secrets via Secrets Manager / SSM, never baked. Items 4, 7, 8, 9 verify this.
- [[Decision - Service Stack]] — TLS termination at ALB. Item 6 verifies this.
- [[Decision - IP Logging for Abuse Protection]] — IP is feedback.json + CloudWatch only. Item 10 verifies this.
- [[Decision - Sentry for Service Errors, Not Bundle Processing]] — payload bytes never reach Sentry or logs. Item 11 verifies this.
- The result of this ticket is a checked-in review document, not a code change. If any item fails, file a follow-up ticket and block O-3 closure until resolved.

## Acceptance Criteria

- [ ] All 14 checklist items have a documented result in `docs/agent/reviews/security-review-mvp-<YYYY-MM-DD>.md`.
- [ ] Items that fail produce a follow-up ticket with a clear remediation; O-3 cannot be marked complete until follow-ups close.
- [ ] Item 11 (payload-bytes-in-logs/Sentry) uses the FEAT-011 redaction guard's behavior as the validation; if the marker leaks, the redaction guard is the bug.
- [ ] The review document is committed to the repo as a durable artifact (so future audits start from the prior review's state, not from scratch).

## Implementation Notes

- Many items can be automated as `bash` or `cargo test` smoke tests; consider checking in a `scripts/security-smoke.sh` that runs items 1–6, 10–14 against the deployed system, leaving items 7–9 as human-review steps.
- Item 11 (payload-in-logs) is the most subtle — it depends on FEAT-011's redaction guard catching the marker. If FEAT-011 was implemented with a deny-list of fields rather than a content scanner, the marker text would only be redacted from named fields, not arbitrary error messages — make sure the redaction is also content-aware (string > 1024 chars heuristic from FEAT-011, plus an explicit check for "SENSITIVE_TEST_MARKER" if added as a recurring smoke test).
- Documentation pointers: `docs/agent/reviews/` already exists per the user's CLAUDE.md convention.
- Schedule a re-run of this checklist on a recurring cadence (quarterly?) — defer that scheduling to a follow-up plan; this ticket only establishes the baseline.

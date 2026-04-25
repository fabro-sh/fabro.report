---
id: FEAT-006
title: "DELETE /r/<id>: remove all Report S3 objects, capability URL becomes 404"
outcome: O-1
tier: must
enabler: false
blocked-by: [FEAT-005]
blocks: [FEAT-007]
cards: [Component - Report Metadata Page, System - Debug Bundle Retention, Storage, and Key Scheme, Standard - Interaction Patterns (CLI + Delete), Decision - Capability URLs over Auth]
---

## Motivation

The delete affordance is the Reporter's only way to shorten retention before the 30-day lifecycle fires. Per [[Decision - Capability URLs over Auth]], URL possession authorizes both view and delete — there is no second factor. This ticket wires up the backend; the visual delete button and the client-side native `confirm()` flow land in FEAT-007 / FEAT-008.

## Description

Implement `DELETE /r/<32-hex-chars>` (or `POST /r/<32-hex-chars>/delete` if HTML form posting is preferred — recommend `DELETE` and use `fetch` from the page in FEAT-007). The handler:

1. Validates the path parameter matches `^[0-9a-f]{32}$`. If not, return 404.
2. Reads `r/<id>.json` to discover the date-partitioned bundle key. If the sidecar is missing, return 404 (already deleted or never existed).
3. Performs an atomic prefix delete via `ListObjectsV2` (keys: the flat sidecar plus all `<YYYY>/<MM>/<DD>/<id>/*`) followed by `DeleteObjects` — both calls in one handler invocation, with the `DeleteObjects` call covering all keys at once.
4. Returns 204 No Content on success. The page in FEAT-007 will redirect (client-side) to the now-404 URL.

Authorization is solely possession of the URL — no session check, no token validation beyond the path-match.

## Context

- [[Component - Report Metadata Page]] — "Delete button submits a delete action requiring no additional auth — possession of the URL is the authorization."
- [[System - Debug Bundle Retention, Storage, and Key Scheme]] — single-Report deletion is a prefix delete (atomic at the Report level), not two separate key deletes.
- [[Standard - Interaction Patterns (CLI + Delete)]] — the delete confirmation is client-side (FEAT-007's responsibility); this ticket implements only the backend endpoint.
- [[Decision - 30-Day Retention Default]] — delete shortens retention but the 30-day lifecycle is the ceiling; user-initiated delete is the only floor mechanism.
- Anti-patterns to avoid: requiring a second-step server confirmation (e.g., `POST /r/<id>/confirm-delete`); requiring the Reporter to "sign in" or provide an additional token; rendering a "this Report has been deleted" page after the delete (the URL must resolve as 404).

## Acceptance Criteria

- [ ] `DELETE /r/<valid-id>` for an existing Report returns 204 and removes both the flat sidecar `r/<id>.json` and the date-partitioned `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` from S3.
- [ ] After the delete, `GET /r/<id>` returns the generic 404 page (no tombstone).
- [ ] `DELETE /r/<valid-id>` for an already-deleted or never-existed Report returns 404 (idempotent at the user-visible level).
- [ ] `DELETE /r/<malformed-id>` returns 404.
- [ ] The handler does NOT require any auth header, cookie, or session — URL possession is the only authorization.
- [ ] If `DeleteObjects` fails partway through, the handler returns 500 with the JSON error envelope; partial-delete state is acceptable since the 30-day lifecycle will eventually clean up the rest.

## Implementation Notes

- Use `aws-sdk-s3`'s `delete_objects` (it accepts a batch of keys in one API call — atomic at the call level for up to 1000 keys; we have at most a handful per Report).
- For idempotency, do not error if the prefix is already empty — return 404 instead.
- The IAM role needs `s3:DeleteObject` on both the flat-sidecar prefix and the date-partitioned prefix (granted in FEAT-002).
- Defer rate-limiting on this endpoint to the same posture as [[System - Feedback Storage & Notification]]'s `POST /v1/feedback` — no per-IP rate limit in MVP.
- A single `DELETE /r/<id>` per Report is the entire delete surface — no `DELETE /v1/reports/<id>`, no admin endpoint, no batch delete.

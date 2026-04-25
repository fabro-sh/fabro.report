---
id: FEAT-004
title: "POST /v1/uploads: mint Report ID, write metadata sidecar, issue presigned PUT"
outcome: O-1
tier: must
enabler: false
blocked-by: [FEAT-003]
blocks: [FEAT-005]
cards: [Component - Report, System - Debug Bundle Retention, Storage, and Key Scheme, Decision - Capability URLs over Auth, Standard - Interaction Patterns (CLI + Delete)]
---

## Motivation

This is the entry point for the Report flow. Without it there is no way to get a Debug Bundle into S3 or to mint a capability URL. The sidecar-before-PUT atomicity strategy was chosen during planning over a more complex completion-callback design — this ticket implements the simpler half of that decision.

## Description

Implement `POST /v1/uploads`. The handler:

1. Generates a Report ID via CSPRNG: 16 bytes from `OsRng`, hex-encoded to 32 lowercase characters.
2. Constructs `metadata.json` with: ID, upload timestamp (server-side, ISO 8601 UTC), retention deadline (timestamp + 30 days), Fabro version (from request body), optional reporter message (from request body), arbitrary key/value tags (from request body).
3. Writes `metadata.json` to S3 at `<YYYY>/<MM>/<DD>/<id>/metadata.json` using the IAM role.
4. Generates a presigned S3 PUT URL for `<YYYY>/<MM>/<DD>/<id>/bundle.tar.gz` with: 15-minute TTL, `Content-Length` constraint (≤ 50 MiB = 52428800 bytes belt-and-suspenders cap).
5. Returns a JSON response: `{ "report_url": "https://fabro.report/r/<id>", "upload_url": "<presigned-put-url>", "expires_at": "<iso-8601>" }`.

Request body (JSON): `{ "fabro_version": string, "platform": string, "arch": string, "message": string|null, "tags": { string: string }|null }`. All fields except `fabro_version`, `platform`, `arch` are optional.

Errors return JSON `{ "error": { "message": "<plain English>", "code": "<short_snake_case>" } }` with status 400 (validation) or 500 (S3 failure).

## Context

- [[System - Debug Bundle Retention, Storage, and Key Scheme]] specifies the key layout `<YYYY>/<MM>/<DD>/<id>/{bundle.tar.gz,metadata.json}` and the 32-hex-char ID shape with 128 bits of CSPRNG entropy.
- [[Decision - Capability URLs over Auth]] makes URL possession the entire access model; ID entropy is therefore load-bearing and not negotiable.
- [[Standard - Interaction Patterns (CLI + Delete)]] documents the upload-architecture: bundle bytes flow CLI → S3 directly via presigned PUT, never through Fargate. The 50 MB cap belongs both at the CLI pre-flight and at the presigned URL's `Content-Length` constraint.
- [[Component - Report]] specifies the metadata fields and immutability of the Report after creation.
- Anti-patterns to avoid: returning the bundle URL itself (we return a `presigned PUT URL` separate from the `report_url`); minting sequential or timestamp-derived IDs; allowing `Content-Length` larger than 50 MiB on the presigned URL; including a content hash in `metadata.json` (forbidden by [[Decision - No Content Dedup]]).

## Acceptance Criteria

- [ ] `POST /v1/uploads` with valid body returns 200 with `report_url`, `upload_url`, `expires_at`.
- [ ] The `report_url` matches the regex `^https://fabro\.report/r/[0-9a-f]{32}$`.
- [ ] The presigned PUT URL succeeds when used with `curl -X PUT --data-binary @bundle.tar.gz -H 'Content-Length: <size>'` for sizes ≤ 50 MiB.
- [ ] The presigned PUT URL rejects requests with `Content-Length > 52428800`.
- [ ] After a successful POST, `aws s3api head-object --bucket <name> --key <YYYY>/<MM>/<DD>/<id>/metadata.json` returns 200.
- [ ] Two consecutive identical POSTs produce two distinct Report IDs and two distinct sidecars (no dedup).
- [ ] Invalid request bodies (missing required fields, malformed JSON) return 400 with the JSON error envelope.
- [ ] `metadata.json` does NOT contain a content hash, a Reporter identity, or any field beyond those in [[Component - Report]].

## Implementation Notes

- Use `aws-sdk-s3` (the official AWS Rust SDK). Use `presigning::PresigningConfig::expires_in` for the TTL.
- For the `Content-Length` constraint, set it as a presigned-request header rather than relying on bucket policy.
- Generate the date partition from the server's current UTC time at the moment of ID minting. Document this in code: the date is the *upload coordination* time, not the bundle PUT time (the two could span midnight UTC, leaving the bundle in `<YYYY>/<MM>/<DD>/<id>/` while the actual PUT happens after midnight — that's fine; the key is fixed at coordination time).
- The IAM role's `s3:PutObject` permission on the metadata key is required. The presigned PUT URL is signed by the IAM role, so the role also needs `s3:PutObject` on the bundle key — already granted by FEAT-002.
- Tone register for error messages: plain English, no stack traces, no status-code prose ("UPLOAD_COMPLETE" is wrong). Per [[Principle - Minimal Public Surface Feels Calm]].
- Defer `Content-MD5` header on the presigned URL — adding it requires the CLI to compute MD5 client-side, which the library does not specify. Trade-off: silent data corruption at the network layer is undetected; acceptable risk for MVP since S3's own checksums catch most cases.

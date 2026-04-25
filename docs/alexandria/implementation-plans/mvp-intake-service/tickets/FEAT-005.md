---
id: FEAT-005
title: "GET /r/<id>: render minimal SSR Report Metadata Page with the four elements"
outcome: O-1
tier: must
enabler: false
blocked-by: [FEAT-004]
blocks: [FEAT-006]
cards: [Component - Report Metadata Page, System - Debug Bundle Retention, Storage, and Key Scheme, Decision - Capability URLs over Auth, Decision - 30-Day Retention Default]
---

## Motivation

The Report Metadata Page is the single rendered surface in the entire product. This ticket implements its structural contract — the four elements, the capability-URL access model, the expired-resolves-as-404 behavior — without yet applying the C2/C3 visual treatment (those land in FEAT-007). After this ticket, a Reporter who pasted their URL into a browser sees a working (if plain) page; FEAT-007 makes it beautiful.

## Description

Implement `GET /r/<32-hex-chars>`. The handler:

1. Validates the path parameter matches `^[0-9a-f]{32}$`. If not, return generic 404.
2. Determines the date partition from `metadata.json` lookup — but since the date is encoded in the key path, the handler must either: (a) store a small index from ID → date partition (rejected — adds storage), (b) try recent date partitions in reverse chronological order (rejected — slow as time passes), or (c) include the date partition in the URL itself (rejected — changes the URL shape from the library's `<32-hex-chars>` spec).
3. **Chosen approach:** the URL contains only the ID, but `metadata.json` is also written to a flat keyspace `r/<id>.json` at upload time so the page handler can do a single `GetObject` lookup. Update FEAT-002 / FEAT-004 to write the sidecar to this flat path in addition to (or instead of) the date-partitioned path. Document the choice in code and in the Library Updates as a refinement to [[System - Debug Bundle Retention, Storage, and Key Scheme]].
4. On metadata `GetObject` 404 → generic 404 page (expired, deleted, or never minted — same response per [[Component - Report Metadata Page]]).
5. On metadata `GetObject` success, do `HeadObject` on the bundle key. If the bundle is missing (orphan from incomplete upload), return generic 404. If present, extract `Content-Length` for the bundle size.
6. Render SSR HTML with the four elements: Report ID, upload timestamp (formatted ISO 8601 UTC), bundle size (decimal, e.g., `2.3 MB`), delete affordance (a button or link — visual treatment lands in FEAT-007).

The page in this ticket can be plain semantic HTML — no CSS framework, no design tokens. The structural skeleton is what matters.

## Context

- [[Component - Report Metadata Page]] specifies the four elements and forbids any fifth. Strict.
- [[System - Debug Bundle Retention, Storage, and Key Scheme]] specifies the URL shape `fabro.report/r/<32-hex-chars>` and the date-partitioned key layout. The flat-keyspace sidecar is a planning decision to enable single-lookup page rendering without changing the URL shape — this requires a library update.
- [[Decision - Capability URLs over Auth]] — possession of the URL grants read; no session, no cookie, no auth check beyond "request reached this URL."
- [[Decision - 30-Day Retention Default]] — expired Reports return 404, no tombstone. Same handler covers expired, deleted, never-existed.
- [[Component - Report Metadata Page]]: "What the page does NOT show" is a long list — file listings, content excerpts, Reporter identity, navigation, "your other uploads" — none of these may appear.
- Anti-patterns to avoid: rendering ANY content from inside the bundle (forbidden — `bun.report` drift); adding navigation; showing an "expired" state distinct from 404; treating the URL as anything other than the access token.

## Acceptance Criteria

- [ ] `GET /r/<valid-id>` for an existing Report returns 200 with HTML containing the Report ID, upload timestamp, bundle size, and delete affordance.
- [ ] `GET /r/<valid-id>` for an expired/deleted/never-existed Report returns 404 with a generic 404 body — no tombstone, no "this Report has been deleted" message.
- [ ] `GET /r/<malformed-id>` (e.g., 31 chars, uppercase, contains `g`) returns 404 — same generic page.
- [ ] The HTML contains exactly four content elements (ID, timestamp, size, delete affordance) plus minimal page chrome (`<title>`, `<meta charset>`); no fifth element, no navigation.
- [ ] The page has no client-side JavaScript yet (the delete confirmation arrives in FEAT-006).
- [ ] The page includes `<meta name="robots" content="noindex,nofollow">` to belt-and-suspender the `robots.txt` directive (added in FEAT-012).

## Implementation Notes

- Use `axum::response::Html` for the response type. A small templating library (`maud`, `askama`, or just `format!`) is fine — pick the simplest. No template engine needs to be introduced as infrastructure; one render path is enough for now.
- The flat-keyspace sidecar (`r/<id>.json`) introduced in this ticket is a refinement to [[System - Debug Bundle Retention, Storage, and Key Scheme]]. Update FEAT-002 (Terraform IAM grants) and FEAT-004 (handler writes both sidecars OR only the flat one) accordingly. Recommend: write only the flat sidecar `r/<id>.json` and continue to write `bundle.tar.gz` to the date-partitioned path. This minimizes S3 objects per Report and keeps the date partitioning useful (Triagers browse by date in S3).
- Format the timestamp with `chrono` or `time` crate. Default ISO 8601 UTC per [[Standard - Receipt Page Sketch]].
- Format the size as decimal (`1.5 MB`, not `1.43 MiB`) per [[Standard - Receipt Page Sketch]].
- Tone register: any user-facing copy on the page (the 404 body text, the delete affordance label) inherits [[Principle - Minimal Public Surface Feels Calm]].
- Use `tower-http`'s `ServeFile` for `robots.txt` and any static asset, OR define them as `axum` routes — implementer's choice.

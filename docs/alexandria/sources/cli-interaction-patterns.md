# Source Material — CLI Interaction Patterns

**Status:** raw source material, not a card. Captures the CLI UX choices Bryan confirmed on 2026-04-23.
**Captured:** 2026-04-23
**Source:** direct confirmation from Bryan Helmkamp in response to unblock-interview Group 3.
**Feeds card:** A7 `Standard - Interaction Patterns (CLI + Delete)` (manifest Amplifier tier).

---

## CLI Command Names (Confirmed 2026-04-23 — Cascade Across Library)

**Confirmed CLI structure** (Bryan, 2026-04-23):

- **`fabro dump`** — the bundle-producing command. Produces the Debug Bundle tarball locally. Replaces what the library through commit `3b4fd06` called `fabro debug`.
- **`fabro dump --upload`** — dumps a Debug Bundle AND uploads it in one step. Returns the Report URL. This is the Reporter's primary path.
- **`fabro upload`** — uploads a *previously* dumped Debug Bundle tarball (run `fabro dump` separately first). Supports `--dry-run` to verify the upload path without actually transferring bytes.

**What stays:** the noun "Report" is unchanged. A Report is still the upload event and its public identity (per F2). What changed is the verb a Reporter types.

**What changes:**
- `fabro debug` → `fabro dump` everywhere in the library's prose.
- `fabro report` (as a CLI command) → `fabro dump --upload` or `fabro upload` everywhere in the library's prose. The product domain `fabro.report` and the noun "Report" are unaffected.

**Cascade scope:** F2 `Standard - Noun Vocabulary`, A1 `Agent - Reporter`, and prose references in F1 / F3 / F4 / F5 / F6 / A2 / A3 / A4 / A6 / A9 / A10 / A11 / C1 / C4. To be applied in a coordinated Sam pass alongside A5 / A7 / A8 drafting, not in a pre-pass.

---

## Confirmed Decisions

### CLI Commands

- **`fabro dump --upload`** — dumps a Debug Bundle and uploads it in one step. Returns the Report URL on success. This is the Reporter's primary path.
- **`fabro dump`** (without `--upload`) — dumps the Debug Bundle locally, does not upload. Produces the tarball that `fabro upload` can later act on.
- **`fabro upload`** — uploads a previously-dumped Debug Bundle tarball. Supports `--dry-run` to verify the upload would succeed (size, auth, connectivity) without actually uploading.
- **`fabro upload --dry-run`** — confirms the upload path works without actually uploading. Useful for pre-flight checks before an actual upload.

### Upload Path (Architecture, Not Just UX)

- **Presigned S3 URLs** — the `fabro upload` / `fabro dump --upload` CLI does NOT POST the bundle bytes to the `fabro.report` service. Instead:
  1. CLI calls the service and requests an upload slot.
  2. Service generates a presigned S3 PUT URL with a short expiration, persists Report metadata (ID, timestamp, Fabro version, reporter message, retention deadline), and returns the presigned URL + the eventual Report URL to the CLI.
  3. CLI uploads the Debug Bundle **directly to S3** using the presigned URL.
  4. S3 stores the bundle at the date-partitioned key (per `storage-and-keys.md`).
- Rationale: bundle bytes never flow through `fabro.report`'s Fargate compute. Saves compute cost, saves bandwidth, saves memory pressure, and scales with S3's bandwidth rather than Fargate's.
- **Cost of presigned URLs:** a small amount of service-side coordination, plus the CLI needs network reachability to both the `fabro.report` service and S3 (typically the same, but occasionally different if a corporate network blocks one but not the other).

### Progress Indicator

- **Progress bar during upload.** Shows upload progress (likely percentage + bytes or percentage + ETA) while the CLI streams the bundle to S3 via the presigned URL.
- Rationale: bundles can be up to 50MB; an upload can take seconds to minutes on slow connections; silent upload is anxiety-inducing and violates F4's "efficient and reassured" tone anchor.
- **Implementation note:** the progress bar is rendered client-side in the CLI, reading the upload byte count from the HTTP library's progress callback. No server-side progress tracking is needed.

### Error Output

- **Console printing best practices, including color when available.**
- "Best practices" means: plain English error messages, not status codes or stack traces as the primary surface. Errors name the problem and (where possible) suggest the next action.
- "Color when available" means: errors in red, warnings in yellow, successes in green, neutral output in default terminal color — when the terminal supports color. Honor `NO_COLOR` environment variable (per A8 accessibility source).
- Calibrate the register against `bun`'s error output per F4 `Principle - Minimal Public Surface Feels Calm`.

### Delete Confirmation

- **Client-side confirmation.** The delete button on the Report Metadata Page triggers a client-side "are you sure?" before the actual DELETE request to the service.
- Implementation detail: a small amount of client-side JavaScript is acceptable here (despite the server-side-rendered HTML stance from `tech-stack-and-deployment.md`) because the confirmation is a UX affordance, not a data-handling concern.
- **Why client-side and not a two-step server roundtrip:** a two-step server roundtrip would add a page load and complicate the flow. A client-side confirmation is faster, cheaper, and still prevents accidental clicks (the primary concern).

### Max Bundle Size

- **50 MB hard cap.**
- If a Debug Bundle exceeds 50 MB, the upload is rejected.
- Rationale: bundles this large are rare for debug data; a cap prevents accidental large uploads (misconfigured debug data capture, environment dumps that sweep up too much) and sets a predictable ceiling on storage cost.
- **Error shape when exceeded:** the CLI should report this clearly before attempting upload (size-check on the local tarball), not after the upload fails at S3. The presigned URL generation can include the `Content-Length` constraint so S3 will also reject an oversized upload if the pre-check is bypassed.

## Implications Raven Observes (flag for review)

- **`fabro dump` owns the tarball format.** The bundle-producing command is in the main Fabro repo, not in `fabro.report`. Changes to what goes into a Debug Bundle are owned there, not here. `fabro.report` treats the tarball as an opaque blob with a size.
- **`fabro upload --dry-run` is a nice UX touch.** It lets a Reporter verify (network, auth, file existence) before a potentially-slow upload. Matches the "efficient and reassured" register from F4.
- **Presigned URLs mean the service never sees bundle contents.** This aligns perfectly with F3's "Debug Bundle contents are opaque tarballs to this service" claim. The architecture structurally enforces the vocabulary.
- **The 50 MB cap should appear in user-facing documentation.** Not surfacing it means Reporters discover it by hitting the error.

## Open Questions (flagged for human judgment)

- **What's the exact error message shape when upload fails?** "Console printing best practices" is directional; concrete error strings will live in a downstream `Standard - CLI Copy` or similar card, not in A7. A7 should specify the *register* (succinct, named failure mode, suggested next action, color-when-available, NO_COLOR-aware) and leave exact strings to implementation.
- **Does `fabro upload` take a path argument** (the path to the previously-dumped tarball) or look for it in a conventional location? Likely both — explicit path wins, with a conventional default. Not captured.
- **Does the CLI print the Report URL and nothing else on success, or does it also print a one-line confirmation?** F4 resolves this ("silent success is not an option per `bun`-style reference") — so a confirmation line plus the URL.

## What This Source Does Not Cover

- **Exact error strings / progress-bar character set / success-message wording.** Implementation detail for a downstream card (`Standard - CLI Copy`).
- **How presigned URLs are generated server-side.** That's an A5 / implementation detail, not a UX spec.
- **The `fabro.report` service's own error handling.** Sentry coverage (per `tech-stack-and-deployment.md`) — separate source.

## Notes for Future A7 Card

When Sam drafts A7:

- **Load-bearing:** the Reporter's interaction shape is "run a command, get a URL, back to work." A7 specifies the mechanics that make that shape feel efficient-and-reassured.
- **Key specs:** progress bar, NO_COLOR-honoring colored errors, 50MB cap with pre-upload check, client-side delete confirmation, `--dry-run` on `fabro upload`.
- **Cross-refs:** F4 (tone register), F2 (canonical command names — but see cascade note above), A1 (Reporter experiences this), A4 (delete button lives on Report Metadata Page), A9 (delete via capability URL, no auth), A10 (if asked to verify dedup pre-upload, refuse — no content hash check).
- **Anti-examples to include:** silent success (`bun`-calibrated "at minimum a confirmation mark" per F4); all-caps error codes; stack traces shown to Reporters; a two-step server-side delete flow; uploading bundle bytes through the service instead of via presigned URL.

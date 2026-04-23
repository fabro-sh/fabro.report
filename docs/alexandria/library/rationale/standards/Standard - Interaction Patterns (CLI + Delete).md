# Standard - Interaction Patterns (CLI + Delete)

## WHAT: Definition

The **interaction-pattern spec** for the surfaces a Reporter and a Feedback Submitter actually touch: the Fabro CLI's dump-and-upload flow on the Debug Bundle side (`fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro upload --dry-run`), the Fabro CLI's fire-and-forget feedback flow on the Feedback side (`fabro feedback`, added 2026-04-23), and the delete-confirmation flow on the Report Metadata Page. This Standard names the UX mechanics — command set, upload architecture, progress indicator, error-output shape, delete confirmation, bundle-size cap, Feedback CLI shape, no-delete-on-Feedback commitment — that collectively render the "efficient and reassured" register from [[Principle - Minimal Public Surface Feels Calm]] as concrete, testable interaction choices across both intake types. It does **not** specify exact strings (register is here; wording belongs to a downstream `Standard - CLI Copy` or implementation-level document) and does **not** specify storage mechanism (the presigned-URL flow for Debug Bundles is summarized here as load-bearing *architecture* — the detail lives in [[System - Debug Bundle Retention, Storage, and Key Scheme]]; the Feedback-flow storage and notification mechanism lives in [[System - Feedback Storage & Notification]]). The spec's shape is deliberately narrow because both submitter interactions are narrow: the Reporter runs a command, gets a URL, optionally deletes, back to work; the Feedback Submitter runs a command, gets a confirmation, back to work.

## WHERE: Ecosystem

- Canonical command names:
  - [[Standard - Noun Vocabulary]] — the "Canonical CLI Commands" subsection of F2 is authoritative for `fabro dump`, `fabro dump --upload`, and `fabro upload`. This Standard expands that set with flags, progress, error, and delete specifics but inherits the command names exactly; any drift between the two is a defect in one of them.
- Tone register (load-bearing):
  - [[Principle - Minimal Public Surface Feels Calm]] — "efficient and reassured," calibrated against `bun`. Every interaction mechanic in this Standard is a concrete expression of that register: progress bar exists because silent upload violates *reassured*; error output is compact because verbose prose violates *efficient*; delete confirmation is one click on the server side because a two-step roundtrip violates both.
- Audience:
  - [[Agent - Reporter]] — the Reporter is the audience of every Debug Bundle interaction specified here. The Reporter's emotional state (interrupted by a bug, already frustrated, looking to get back to work) is the calibration target; the specifics in this Standard are what make "efficient and reassured" land on that audience.
  - [[Agent - Feedback Submitter]] — added 2026-04-23. The Feedback Submitter is the audience of the `fabro feedback` CLI interaction specified in the Feedback CLI subsection below. Their emotional state is sharing voice with a trusted team (not crisis-state), and the CLI specifics in this Standard are what make "efficient and reassured" land on that audience through a fire-and-forget confirmation rather than a URL-returning success.
- Paired surface:
  - [[Component - Report Metadata Page]] — the delete-confirmation flow specified here runs on that page. This Standard owns the *interaction mechanics* of the delete button (client-side confirmation, keyboard-first operation); A4 owns the page's structural elements (ID, timestamp, size, button).
  - [[Component - Feedback]] — the entity created by the `fabro feedback` interaction specified in the Feedback CLI subsection. This Standard owns the CLI mechanics (positional text/files, `-e, --email` flag, stdin, confirmation shape, no-delete commitment); A16 owns the entity's fields, storage layout, and lifecycle.
- Access-model coupling:
  - [[Decision - Capability URLs over Auth]] — the delete button authorizes solely by URL possession; there is no additional credential check, no CSRF token tied to a login, no "are you the owner" step. The client-side confirmation prevents accidental clicks; it does not implement authorization.
- Observability boundary:
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — the Reporter-facing error register specified in this Standard is never overlaid with Sentry-shaped artifacts (event IDs, "please report this crash" links, Sentry dashboard URLs). A13 keeps operational monitoring invisible to Reporters; this Standard's error output is the only surface the Reporter sees.
- Scope commitments:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the interaction surface is a CLI upload and a one-page delete affordance. Anything beyond that (dashboards, lists, post-upload feedback surfaces) is thesis-level drift, not an interaction-pattern decision this Standard can make.
  - [[Principle - Narrow Forever]] — the three-command surface is the permanent command surface. Additional subcommands (`fabro reports list`, `fabro report status`, `fabro upload watch`) are scope expansions, not interaction-pattern additions.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped interaction patterns (post-upload feedback prompt, symbolication-status view, "your team is reviewing" line) are named drift. The interaction surface stops at confirmation + URL + optional delete.
- Storage mechanism (referenced, not owned):
  - [[System - Debug Bundle Retention, Storage, and Key Scheme]] — the presigned-URL upload flow (bundle bytes never traverse the service's Fargate compute) is summarized here as a load-bearing architectural property that affects the CLI's behavior (the CLI talks to two endpoints: the service for coordination, and S3 directly for bytes). The key scheme, URL entropy, bucket layout, and retention mechanism live in that System card.
- Service topology (referenced, not owned):
  - [[Decision - Service Stack]] — the presigned-URL architecture is coherent because the compute and the storage live in the same cloud under a shared IAM boundary. A12's AWS-native posture is what makes A7's "bundle bytes never traverse Fargate" property operationally cheap.
  - [[System - Service Topology]] — the operational topology A7's interaction paths traverse (DNS → Fargate for coordination; CLI → S3 for bytes). A14 names the topology pieces; this Standard names the Reporter-facing interaction on top of them.
- Dedup posture (referenced, not owned):
  - [[Decision - No Content Dedup]] — the CLI does **not** hash the Debug Bundle and check for a prior upload before sending. A Reporter who uploads the same tarball twice produces two Reports with two URLs; the service does not notice. This Standard enforces that posture by not adding a pre-upload content-check step.
- Paired accessibility floor:
  - [[Standard - Accessibility Policy]] — `NO_COLOR` support on the CLI and WCAG 2.2 AA + keyboard-first operation on the Report Metadata Page. This Standard names `NO_COLOR`-awareness as a mechanic; A8 binds it to the accessibility floor and specifies the full keyboard-and-screen-reader shape of the delete confirmation.
- Visual direction for the delete affordance (paired Standards):
  - [[Standard - Design System (Receipt Page + CLI Output)]] — C2 specifies the visual system (fonts, palette, semantic tokens, `--color-coral` for destructive framing in the confirmation dialog, `--color-focus` for the keyboard focus ring) this Standard's delete interaction is rendered in. C2 also owns the delete-confirmation accessibility requirements that pair with the mechanics here.
  - [[Standard - Receipt Page Sketch]] — C3 specifies the delete-affordance *shape* (secondary text-link, not a big red button) placed below the metadata rows. This Standard owns the delete *mechanics* (client-side confirmation, URL-possession authorization); C3 owns the *affordance shape* the mechanics attach to.
- Forward references (not yet drafted):
  - `Standard - CLI Copy` (not in manifest; referenced as the future home for exact strings) — exact success lines, exact error wording, exact progress-bar character set. This Standard specifies the *register*; wording is downstream.
- Source provenance:
  - `docs/alexandria/sources/cli-interaction-patterns.md` — the raw material Bryan confirmed on 2026-04-23. Authoritative for the three-command set, presigned-URL architecture, progress bar, error-register, delete confirmation shape, and 50 MB cap.

## WHY: Rationale

Interaction patterns are where a Principle becomes a product. [[Principle - Minimal Public Surface Feels Calm]] says "efficient and reassured"; without a Standard that specifies *how* progress is shown, *how* errors print, *how* delete confirms, every downstream implementation agent is free to re-land "efficient and reassured" differently, and the register fractures across surfaces one commit at a time. This Standard exists to collapse that interpretive space into a small, concrete, reviewable list of mechanics so the Principle is *verifiable* on any surface the product ships.

The load-bearing architectural claim is that **the CLI uploads to S3, not to the service**. Bundle bytes flow through a presigned S3 URL obtained from the service's coordination endpoint; they never traverse the service's Fargate compute. That architecture is named here (rather than deferred entirely to [[System - Debug Bundle Retention, Storage, and Key Scheme]]) because it shapes Reporter-facing interaction directly: the CLI must be reachable to two network destinations (service + S3), the progress bar reads its byte count from the S3 upload stream, and `fabro upload --dry-run` exists as a meaningful UX affordance precisely because there are two coordination moments (size pre-check and presigned-URL issuance) worth verifying before a long transfer. If the architecture were a single-POST-through-service flow, half of this Standard's mechanics would be different. Naming it here makes the coupling explicit.

The second load-bearing claim is that **client-side delete confirmation is correct for this product**. A two-step server-side confirmation would add a page load and complicate the flow. A client-side confirmation is faster and still prevents accidental clicks (the primary concern). The trade-off is accepted explicitly because the alternative (a stateful server-side "pending delete" record) would introduce a kind of state — transient, session-bound, Reporter-associated — that the product otherwise rigorously avoids. Client-side confirmation preserves the zero-session property while still giving the Reporter an exit ramp.

## WHEN: Timeline

Drafted 2026-04-23 from source material captured the same day (`docs/alexandria/sources/cli-interaction-patterns.md`). Replaces the pre-2026-04-23 assumption that the command was a single `fabro report` subcommand. Stable. Revisions to the command set — adding a command, removing `--dry-run`, splitting `fabro dump --upload` into separate commands — are scope changes that revisit [[Standard - Noun Vocabulary]] and [[Principle - Narrow Forever]] before they become interaction-pattern updates. Revisions to the mechanics (progress-bar shape, error-register details, delete-confirmation modality) are implementation-level updates to this card, appropriate to land as Standard updates.

## HOW: Specification

### CLI Command Set

Canonical names, inherited from [[Standard - Noun Vocabulary]] "Canonical CLI Commands." Reproduced here with interaction-pattern detail:

- **`fabro dump`** — produces a Debug Bundle tarball locally. Prints the path to the produced tarball on success. Does not upload. Does not make network calls to `fabro.report`.
- **`fabro dump --upload`** — produces a Debug Bundle and uploads it in one step. Returns the Report URL on success, with a confirmation line (per [[Principle - Minimal Public Surface Feels Calm]] on silent success). Reporter's **primary path**; the CLI-copy register in downstream drafts should treat this as the default success surface.
- **`fabro upload`** — uploads a previously-dumped Debug Bundle tarball. Returns the Report URL on success.
- **`fabro upload --dry-run`** — verifies the upload path (size pre-check under 50 MB, authentication to the service's coordination endpoint, ability to reach S3 for a presigned PUT) without transferring bundle bytes. Useful for pre-flight checks on constrained or unreliable networks.
- **Flags available to upload paths:**
  - `-m "short message"` — optional Reporter message attached as a [[Component - Report]] field. Source: `docs/alexandria/sources/product-vision.md` L58.
  - arbitrary key/value metadata tags — attached as Report fields. Source: `docs/alexandria/sources/product-vision.md` L85. Exact flag syntax (`--tag key=value` or similar) is an implementation-level decision consistent with `bun`-calibrated tone; this Standard does not pin the spelling.

There are no other subcommands on `fabro.report`'s surface. See [[Standard - Information Architecture]] "CLI Surface."

### Upload Architecture (Load-Bearing)

- **Presigned S3 URLs.** The CLI does **not** POST bundle bytes to the `fabro.report` service. Instead:
  1. The CLI calls the service's coordination endpoint and requests an upload slot.
  2. The service mints a Report ID, persists the Report's metadata (ID, timestamp, Fabro version, reporter message, retention deadline, any tags), and returns a presigned S3 PUT URL plus the eventual Report URL to the CLI.
  3. The CLI uploads the Debug Bundle bytes **directly to S3** using the presigned URL.
  4. S3 stores the bundle at the date-partitioned key (per [[System - Debug Bundle Retention, Storage, and Key Scheme]]).
- **Property:** bundle bytes never flow through `fabro.report`'s Fargate compute. Saves compute cost, saves bandwidth, saves memory pressure on the service, and scales with S3's bandwidth rather than the service's.
- **Network requirement:** the CLI needs reachability to *both* the `fabro.report` service (for coordination) and S3 (for the bytes). On most networks these are the same reachability; on some corporate networks they differ, and `fabro upload --dry-run` is the mechanic that surfaces that asymmetry before a long transfer.

### Progress Indicator

- **Progress bar during upload.** Shows percentage and either a byte counter or an ETA while the CLI streams the bundle to S3. Rendered client-side, reading from the HTTP library's progress callback.
- **Rationale:** bundles can be up to 50 MB; an upload can take seconds to minutes on slow networks. Silent upload is anxiety-inducing and violates the *reassured* half of [[Principle - Minimal Public Surface Feels Calm]].
- **Restraint:** a compact progress bar in the `bun`-calibrated register — not a decorative animation, not ASCII art. Degrades gracefully when `NO_COLOR` is set (see [[Standard - Noun Vocabulary]] cross-ref to accessibility above) and in CI log contexts where the terminal may not repaint lines.

### Error Output

- **Console printing best practices.** Errors name the problem in plain English and, where possible, suggest the next action. No stack traces surfaced on the happy-path failure surface. No status codes as the primary surface (`UPLOAD_COMPLETE`, `ERR_NETWORK_TIMEOUT` are wrong per [[Principle - Minimal Public Surface Feels Calm]]).
- **Color-when-available.** Errors render in red, warnings in yellow, successes in green, neutral output in the default terminal color — when the terminal supports color.
- **`NO_COLOR`-aware.** When the `NO_COLOR` environment variable is set (per https://no-color.org/), the CLI emits no color escape codes; the register survives without color because it was built on word choice, not color choice. See [[Standard - Accessibility Policy]] for the accessibility floor this mechanic satisfies.
- **Register calibration.** Against `bun`'s error output per [[Principle - Minimal Public Surface Feels Calm]].

### Delete Confirmation

- **Client-side confirmation.** The delete button on the [[Component - Report Metadata Page]] triggers a client-side "are you sure?" before the actual DELETE request to the service.
- **Implementation note:** a small amount of client-side JavaScript is acceptable for the confirmation affordance (despite the server-side-rendered-HTML default posture for the page), because the confirmation is a UX-safety affordance, not a data-handling step.
- **Why not a two-step server roundtrip:** a server-side "confirm delete" intermediate step would add a page load, introduce transient server-side state tied to the Reporter's session, and complicate the flow. Client-side is faster, stateless, and prevents accidental clicks (the primary concern).
- **Accessibility note:** a native browser `confirm()` dialog is accessible by default. A custom modal needs focus trap, ESC-to-cancel, focus return, `role="dialog"`, and `aria-modal="true"`. [[Standard - Accessibility Policy]] specifies the constraints on either path; this Standard leaves the modality choice open.
- **Authorization:** possession of the URL is the whole authorization. There is no additional credential check on the delete action (per [[Decision - Capability URLs over Auth]]).

### Max Bundle Size

- **50 MB hard cap.** If a Debug Bundle exceeds 50 MB, the upload is rejected.
- **Pre-flight check in the CLI.** Size is checked against the local tarball *before* the upload begins. The Reporter sees the rejection before bytes start moving, not after a long transfer fails at S3.
- **Belt-and-suspenders at S3.** The presigned URL is issued with a `Content-Length` constraint, so S3 also rejects an oversized upload if the CLI pre-check is bypassed (e.g., a non-standard client).
- **Error register:** the size-limit rejection follows the error-output register above. Plain English, names the problem, suggests the action (inspect the bundle or trim its inputs).

### Feedback CLI (`fabro feedback`)

The Feedback flow has its own CLI interaction shape — parallel to `fabro dump --upload` / `fabro upload` on the Debug Bundle side, but structurally different because the Feedback flow is fire-and-forget. Confirmed 2026-04-23 with the product owner; modeled on `bun feedback`.

- **Command syntax:** `fabro feedback [options] [feedback text ... | files ...]`. Positional arguments are interpreted as either feedback text (if no existing file matches the path) or file attachments (if the path resolves to a file). Stdin is accepted via pipe: `echo "..." | fabro feedback --email you@example.com`.
- **Options:**
  - `-e, --email <email>` — sets the email address used for this submission. Optional. Treated as an opaque string; format validation is an implementation detail and is not specified at the Standard layer.
  - `-h, --help` — shows help and exits.
- **Typical invocations:**
  - `fabro feedback "Love the new release!"` — plain text submission.
  - `fabro feedback report.txt details.log` — file attachments, no text.
  - `echo "please document X" | fabro feedback --email you@example.com` — stdin text + email.
- **Return value:** a confirmation message on success. **No URL is returned.** No Report ID is displayed. No link to a service-hosted page is shown — because no such page exists for the Feedback flow. The Feedback Submitter's involvement ends at the confirmation line.
- **Upload shape:** a direct `POST /v1/feedback` to the service. This differs from the Debug Bundle flow's presigned-S3-URL architecture: Debug Bundle tarballs are large (up to 50 MB, kept out of Fargate compute via presigned URLs); Feedback payloads are smaller (text + small file attachments) and the presigned-URL indirection is overkill for the payload size. Exact request shape (multipart form, JSON envelope) is an implementation detail consistent with the service's existing request handling; this Standard does not pin the wire format. If payload-size pressure surfaces later, switching to a presigned-URL architecture for Feedback is a mechanism change that revisits [[System - Feedback Storage & Notification]] rather than this Standard.
- **Size handling:** no explicit 50 MB cap is specified for Feedback at the Standard layer (that cap was calibrated for Debug Bundle tarballs, which can grow large with build output, stack traces, and environment capture). Large Feedback attachments — especially when they exceed Discord's webhook attachment size limits — are a graceful-fallback mechanism question owned by [[System - Feedback Storage & Notification]], not an interaction-pattern detail. If Bryan eventually fixes a Feedback payload cap, the cap lands here as an explicit number in a revision pass.
- **Error output:** follows the same error-output register specified above (plain English, no stack traces, color-when-available, `NO_COLOR`-aware, register calibrated against `bun`). The `fabro feedback` CLI is part of the NO_COLOR-aware set; see [[Standard - Accessibility Policy]] for the accessibility floor that binds this mechanic across all four canonical CLI commands (`fabro dump`, `fabro dump --upload`, `fabro upload`, `fabro feedback`).
- **Progress indicator:** minimal. Feedback payloads are small enough that a full progress bar is unnecessary; a spinner or a short "sending…" line is sufficient, in the same compact register as the Debug Bundle progress bar. Silent transfer violates *reassured* for the same reason silent upload does on the Debug Bundle flow.

### No Delete on the Feedback Flow (Explicit Divergence from the Debug Bundle Flow)

The Feedback flow **has no delete capability** — at the CLI, at the service, anywhere on the product's surface. This is an explicit, load-bearing divergence from the Debug Bundle flow's delete-via-URL-possession shape specified in the Delete Confirmation section above. The divergence is structural, not a missing feature:

- **No URL is returned.** The Feedback Submitter never receives a capability URL for their submission, so there is no URL to use as authorization for a delete action. The capability-URL access model from [[Decision - Capability URLs over Auth]] applies to the Debug Bundle flow only — see A9's scope-note and [[Decision - Fire-and-Forget Feedback]] for why the Feedback flow diverges.
- **No rendered page exists.** There is no service-hosted page a Feedback Submitter could visit to click a "delete" button. The Feedback flow renders nothing publicly; there is no surface to put a delete button on.
- **No delete endpoint on the service.** There is no `DELETE /v1/feedback/:id` endpoint, no delete link in the Discord post, no CLI subcommand for deleting a submitted Feedback. Attempting to add any of these is a scope expansion that re-introduces a post-submission submitter-facing surface the Feedback flow is designed not to have.
- **Expiration happens only via the 30-day S3 lifecycle rule.** Feedback, like Debug Bundles, expires after 30 days (per [[Decision - 30-Day Retention Default]]'s extended scope across both flows). The S3 lifecycle rule in [[System - Feedback Storage & Notification]] handles expiration server-side; the Feedback Submitter cannot accelerate it, cannot extend it, and cannot observe it.

**The load-bearing phrasing is *Feedback is permanent from the user's perspective, expires only via S3 lifecycle*.** A Feedback Submitter should understand — implicitly, from the CLI's fire-and-forget confirmation; or explicitly, if they ask — that the service will hold their submission for up to 30 days, that there is no recall mechanism, and that the submission has already landed in both S3 and Discord by the time the CLI prints the confirmation line. The trade-off is accepted because the alternatives (a capability URL with a delete action, an email-verification flow, an account-based delete) all re-introduce a post-submission surface the Feedback flow is designed not to have — and because Feedback Submitters who want to recall a submission can contact the Fabro team directly (out-of-band, via the same Discord channel or via email), who can manually delete the S3 objects if the request is legitimate.

An agent reading this Standard cold should not propose "let's add a delete action to the Discord post" as a convenience feature; that proposal is a violation of the fire-and-forget commitment and a re-introduction of a submitter-facing post-submission surface. It is rejected at the Standard layer and at the forward-referenced A19 Decision layer.

### Anti-Examples (Interaction Patterns This Standard Refuses)

- **Wrong:** Silent success — a command that prints only the URL with no confirmation line. Violates *reassured*; the Reporter cannot tell from the output whether the URL is a success or an accident. At minimum a confirmation line plus the URL.
- **Wrong:** Status-code prose — `UPLOAD_COMPLETE`, `ERR_SIZE_EXCEEDED`, `RETURN=0`. Violates the conversational register per [[Principle - Minimal Public Surface Feels Calm]].
- **Wrong:** A two-step server-side delete confirmation flow. Introduces transient session state the product otherwise avoids; adds a page load; violates the zero-session posture.
- **Wrong:** Uploading bundle bytes through the `fabro.report` service (a single POST to the service rather than a presigned S3 URL). Breaks the "bundle bytes never flow through Fargate" architectural property and defeats the cost/memory rationale.
- **Wrong:** A content-hash pre-upload check ("you've already uploaded this; return existing URL?"). Forbidden by [[Decision - No Content Dedup]]; adds the exact surface area that Decision rejects.
- **Wrong:** All-caps error codes (`ERR_NETWORK_TIMEOUT`) or stack traces shown to Reporters on the happy-path failure surface. The CLI-copy register is conversational, not log-line.
- **Wrong:** Post-upload feedback prompts ("how was your reporting experience?") rendered after a successful upload. Forbidden by [[Principle - Reject Like bun.report Drift]].
- **Wrong:** Color-only signaling for errors. Red-without-text fails colorblind users and the `NO_COLOR` opt-out; every error must be readable as text without color.
- **Wrong:** Returning a URL on `fabro feedback` success (as a "you can view your feedback here" affordance). The Feedback flow is fire-and-forget; a URL would imply a service-hosted page (which does not exist) or a delete capability (which does not exist). Violates the fire-and-forget commitment and [[Decision - Fire-and-Forget Feedback]].
- **Wrong:** Adding a delete action (CLI subcommand, Discord post button, service endpoint) for submitted Feedback. Feedback has no delete path — see the "No Delete on the Feedback Flow" subsection above. The correct recourse for a Feedback Submitter who wants a submission removed is an out-of-band request to the Fabro team, not a service feature.
- **Wrong:** Reusing the 50 MB Debug Bundle cap as a `fabro feedback` attachment cap at the interaction-pattern layer without confirmation. The 50 MB number is calibrated for Debug Bundle tarballs; Feedback payload limits (especially around Discord's attachment-size constraint) are a mechanism question owned by [[System - Feedback Storage & Notification]], not an interaction-pattern decision.

### Test

Before shipping any CLI string, error path, progress shape, or delete-flow detail, ask three questions in order. First: "Does this mechanic render the 'efficient and reassured' register from [[Principle - Minimal Public Surface Feels Calm]]?" If not, it's off-register — recalibrate against `bun`. Second: "Does this mechanic respect the architectural property that bundle bytes never flow through the service?" If the CLI path starts POSTing bundle bytes to the service, the architecture has drifted and the answer is not an interaction-pattern tweak — it's a regression of the presigned-URL flow. Third: "Does this mechanic preserve [[Decision - Capability URLs over Auth]]'s zero-session, URL-is-authorization model?" If a new step introduces a session cookie, a CSRF token tied to a login, or a server-side pending-delete record, the delete flow has grown state it should not have.

## Sources

- `docs/alexandria/sources/cli-interaction-patterns.md` (full file — authoritative for Debug Bundle command set, presigned-URL architecture, progress bar, error register, delete confirmation, 50 MB cap, `--dry-run`; confirmed 2026-04-23)
- `docs/alexandria/sources/product-vision.md` (L58 `-m` flag; L85 metadata tags; L22–L32 end-to-end flow)
- `docs/alexandria/sources/feedback-flow.md` (2026-04-23 — `fabro feedback` CLI syntax, `-e`/`--email` flag, stdin support, positional text-or-files arguments, fire-and-forget confirmation shape, direct-POST upload to `/v1/feedback`, no-delete commitment)
- `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` (Part 3 A7 medium update — parallel Feedback CLI section; no-delete as explicit divergence from the Debug Bundle flow)
- `docs/alexandria/inventory/card-manifest.md` (A7 entry)

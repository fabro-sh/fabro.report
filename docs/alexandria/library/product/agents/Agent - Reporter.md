# Agent - Reporter

## WHAT: Definition

A **Reporter** is a self-hosted Fabro user who hit a bug and ran `fabro dump --upload` (or `fabro upload` against a previously-dumped tarball) to upload a Debug Bundle to `fabro.report` so the Fabro team can retrieve it. The Reporter has no identity on this service — no account, no login, no user record — and is identified only by the Fabro version captured as metadata at upload time. Their entire interaction with the service is the Fabro CLI's dump-and-upload flow: one command (or two, if they chose to dump and upload separately), one Debug Bundle upload, one short URL returned on stdout, and whatever CLI flags they chose to pass. They may optionally visit the Report Metadata Page at that URL, but nothing obliges them to. The Reporter's goal is simple and time-bounded: get the Debug Bundle to the Fabro team, get the URL into the right place (GitHub issue, Slack, email), and get back to the work the bug interrupted.

## WHERE: Ecosystem

- Identity and vocabulary:
  - [[Standard - Noun Vocabulary]] — the canonical **Reporter** definition (no identity, no login, identified only by Fabro version metadata; may optionally attach a short message via CLI flag; expected to share URL out-of-band) is the authoritative definition this card expands; any drift between the two is a defect in one of them.
- Audience commitment:
  - [[Decision - Who This Service Is For]] — the Reporter described here is a **self-hosted Fabro user, primary and only**. A hosted-Fabro offering's users are out of scope under that Decision; this card inherits that audience shape and does not describe a generic "anyone with a Debug Bundle" Reporter.
- Counterpart role:
  - [[Agent - Triager]] — the Fabro team member on the other end of the pickup. The Reporter and Triager are the two endpoints of the handoff, and the asymmetry between them — no-identity Reporter on the service, credentialed Triager retrieving from S3 — is load-bearing for the product shape. A Reporter never becomes a Triager within this service; the two roles are structurally separate.
- Access model:
  - [[Decision - Capability URLs over Auth]] — the short URL returned by `fabro dump --upload` (or `fabro upload`) is the Reporter's **entire access surface** to the service after upload. Possession of the URL grants view of the Report Metadata Page and the ability to delete the Report. There is nothing else the Reporter can do on this service; there is no login, no "my uploads" view, no recovery flow.
- Tone of the interaction:
  - [[Principle - Minimal Public Surface Feels Calm]] — the emotional goal for the Reporter is **efficient and reassured**. The Reporter is already frustrated by the bug that sent them to `fabro dump --upload`; the CLI and the Report Metadata Page must confirm the upload cleanly and step out of the way. Warmth that slows the handoff is worse than no warmth at all.
- Scope coherence:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — the Reporter drops a Debug Bundle at the pickup point and leaves. They are not an audience for in-browser rendering, symbolication, or bundle inspection; designing Reporter-facing features beyond upload + URL + optional delete is the single most common drift shape this product rejects.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`'s Reporter-equivalent is a richer role with a post-upload web experience (markdown-rendered crash data, feedback prompts). `fabro.report`'s Reporter is deliberately thinner; any proposal to enrich the Reporter's post-upload surface is the paradigmatic drift this Principle enumerates.
- Strategic why:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the Reporter's narrow interaction shape is calibrated to the Strategy. Every design choice on the Reporter-facing surface is a trade between "team can retrieve the Debug Bundle more easily" and "Reporter sees a richer surface," and the Strategy ranks the first strictly above the second.
- Scope commitment:
  - [[Principle - Narrow Forever]] — Reporter-facing scope expansion (accounts, "my uploads" views, post-upload feedback, in-browser bundle viewing) is permanently out-of-scope; the Reporter role described here is the Reporter role forever.
- Referenced by:
  - [[Component - Report]] — the upload event the Reporter creates when they run `fabro dump --upload` (or `fabro upload`); carries the URL they share and the authoritative field list the Reporter contributes to (Fabro version, optional message, metadata tags).
  - [[Component - Report Metadata Page]] — the one-page surface at the returned URL that the Reporter may optionally visit; carries the delete button they may optionally use.
  - [[Standard - Information Architecture]] — A6 describes the Reporter's IA traversal as "run `fabro dump --upload`, optionally visit the Report's URL once or twice." Reporters do not browse and do not navigate; the IA is the commitment that preserves that.
- UX and mechanism the Reporter experiences:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the `fabro dump --upload` / `fabro upload` CLI UX specifics (progress indicator, error surface, success output, `--dry-run`) the Reporter actually experiences.
  - [[Standard - Accessibility Policy]] — the accessibility floor for the two surfaces the Reporter touches: CLI `NO_COLOR` support and WCAG 2.2 AA on the Report Metadata Page. A Reporter using a screen reader, operating the terminal without color, or navigating with the keyboard alone must still hit the pickup-point flow cleanly.
  - [[System - Retention, Storage, and Key Scheme]] — the mechanism behind the 30-day retention that bounds how long the URL the Reporter shares remains valid.
  - [[System - Service Topology]] — the operational topology the Reporter's request flow traverses: DNS → Fargate for upload coordination and page render; CLI → S3 directly for bundle bytes.
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — Reporters never see Sentry. The operational monitoring posture A13 names is invisible to the Reporter's CLI output, the Report Metadata Page, and every other Reporter-facing surface.

## WHY: Rationale

- Strategy: [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the Reporter's minimal interaction shape is the concrete expression of "make it easy for self-hosted Fabro users to get Debug Bundles to the team." A login step, an account, or a post-upload engagement flow would each add friction in the exact place Strategy is optimizing to remove it.
- Thesis: [[Product Thesis - Pickup Point, Not Triage Surface]] — the Reporter is one endpoint of the pickup. Their job is to drop the Debug Bundle and share the URL; the service's job ends at "URL exists." A Reporter who stays on the service after upload to read their own bundle in-browser is evidence the product has drifted toward a triage surface.
- Driver: Without this card, downstream product-layer cards (Component - Report, Component - Report Metadata Page, Standard - Interaction Patterns) each risk inventing their own implicit Reporter — one that has a user ID, one that has an account, one that "wants to see" what they uploaded. Naming the Reporter once, with the no-identity shape load-bearing, prevents each downstream card from silently widening the role.

## WHEN: Timeline

New role, no past. Confirmed 2026-04-22 by the product owner in the Users section of `docs/alexandria/sources/product-vision.md` (L52–L59). Stable. The no-identity, no-account, Fabro-version-only shape is calibrated to the `fabro.report` service's identity decisions ([[Principle - Narrow Forever]], [[Decision - Capability URLs over Auth]], [[Decision - Who This Service Is For]]); revision of this role requires revising at least one of those upstream cards, not an implementation-level change.

## HOW: Behavior

### Context: When a Reporter Arrives

The Reporter is not a product-evangelist persona; they arrive because something failed. A self-hosted Fabro user was using Fabro to run a workflow, Fabro broke, and the user ran `fabro dump --upload` (or, having already produced a tarball with `fabro dump`, ran `fabro upload`) to ship it to the Fabro team. By the time the Reporter sees the CLI output, they have already been interrupted by a failure, already lost context on whatever they were actually trying to do, and are looking to spend the minimum possible time on the reporting path so they can either (a) resume their original work, (b) share the URL with the Fabro team and wait, or (c) try a workaround. The CLI's output shape is calibrated to that emotional state — this is why [[Principle - Minimal Public Surface Feels Calm]] specifies *efficient and reassured* and not, say, *welcoming* or *thorough*.

### What the Reporter Does

- Runs `fabro dump --upload` (optionally with `-m "short message"` or arbitrary key/value metadata tags) to produce and upload a Debug Bundle in one step, or runs `fabro dump` followed by `fabro upload` when they want the two phases separate (e.g., to use `fabro upload --dry-run` first).
- Reads the CLI's success output, which confirms the upload and includes the URL.
- Copies the URL into wherever they are going to share it (GitHub issue, Slack, email). This step is out-of-band — the service plays no role in it.
- Optionally visits the Report Metadata Page at that URL to confirm upload details (Report ID, upload timestamp, bundle size) or to delete the Report if they change their mind about sharing it.
- Returns to their original work.

### What the Reporter Does Not Do

- Does not log in. There is no account concept on this service.
- Does not interact with S3. Debug Bundle contents are never exposed to the Reporter through the service; the bundle path is CLI-upload-only.
- Does not see a list of their past uploads. The service has no Reporter identity to tie past uploads to, by design. A Reporter who has lost the URL has no way to recover it through this service; the correct remedy is to upload again. See [[Decision - Capability URLs over Auth]].
- Does not receive follow-up messages from the service. No "your Report expired," no "we've received your crash report," no "tell us how it went." The optional CLI-flag message at upload time is the entire feedback channel. See [[Principle - Reject Like bun.report Drift]].
- Does not interact with Triagers through the service. Communication between a Reporter and the Fabro team, if it happens, happens wherever the URL was shared (GitHub issue, Slack, email), not through `fabro.report`.
- Does not use a "dashboard." There is no Reporter-facing dashboard, nor will there be. See [[Principle - Narrow Forever]].

### Examples

- **Example 1 — the happy path.** A self-hosted Fabro user hits a bug while running `fabro run`. They run `fabro dump --upload -m "cargo install hangs on CI"`. The CLI produces the tarball, uploads the Debug Bundle, prints a confirmation line and a short URL, and exits. The Reporter pastes the URL into a GitHub issue with a one-line description of the bug. They close the terminal and resume their original work. They never visit the Report Metadata Page. The Fabro team picks up the URL in the issue, extracts the Report ID, and retrieves the Debug Bundle from S3 using their own AWS credentials. The pickup is complete; the Reporter's involvement is over.
- **Example 2 — the Reporter changes their mind.** A Reporter runs `fabro dump --upload`, gets a URL, pastes it into a Slack channel, and then realizes the Debug Bundle contains a configuration fragment with a path they do not want shared. They visit the Report Metadata Page at the URL, click the delete button, confirm the deletion, and the Report is gone. No login was required; possession of the URL was the entire authorization. See [[Decision - Capability URLs over Auth]].

### Anti-Examples

- **Wrong:** Designing any feature that presumes the Reporter will return to the Report Metadata Page to *read* bundle contents (a file listing, a logs view, a symbolicated stack trace, a markdown-rendered log). The Reporter's post-upload surface is view-metadata + delete, full stop; anything else is a drift into triage-surface shape. See [[Product Thesis - Pickup Point, Not Triage Surface]] and [[Principle - Reject Like bun.report Drift]].
- **Wrong:** Treating the Reporter as "the user" in the generic sense implied by most web products. This product has no "user" concept — only Reporters (who have no identity) and Triagers (who use S3 directly). See [[Standard - Noun Vocabulary]]. A card or CLI string that calls the Reporter "the user" has already started to drift toward the account-shaped identity model this product rejects.
- **Wrong:** Adding a "recover my URL" flow for Reporters who lost the URL. This requires Reporter identity (email, account, cookie), which the product does not have and will not have. See [[Decision - Capability URLs over Auth]]. The correct answer is: upload again.
- **Wrong:** Adding a post-upload feedback prompt ("How was your reporting experience? Tell us what went wrong.") on the Report Metadata Page or via a CLI follow-up. This turns pickup into conversation and is on the permanently-out-of-scope list. See [[Principle - Narrow Forever]].

## Sources

- `docs/alexandria/sources/product-vision.md` (L52–L59 Users section → Reporter definition; L22–32 end-to-end flow; L74–88 Confirmed Product Decisions — identity, auth, dedup; L92–L98 Confirmed Entities — Reporter at L98)
- `docs/alexandria/inventory/card-manifest.md` (A1 entry)

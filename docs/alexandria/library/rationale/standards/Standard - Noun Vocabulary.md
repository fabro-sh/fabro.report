# Standard - Noun Vocabulary

## WHAT: Definition

The canonical name, definition, and common-mistake list for every first-class entity in the `fabro.report` product. Seven entities — Report, Debug Bundle, Reporter, Triager, Report Metadata Page, Feedback, Feedback Submitter — each with exactly one canonical spelling and one authoritative definition. Every other card, every CLI string, every log line, every doc, and every agent-generated surface must use these exact terms in these exact senses. New nouns may not be introduced without first being added here.

The vocabulary grew from five entities to seven on 2026-04-23 when the product expanded from one intake type (Debug Bundles) to two (Debug Bundles + Feedback). The two new nouns — **Feedback** (the submission entity) and **Feedback Submitter** (the persona) — are parallel in shape to Report and Reporter but distinct in identity model and post-pickup mechanics; they are not synonyms for their Debug-Bundle-side counterparts and must not be used interchangeably. A third intake type would require new canonical nouns and a product-owner identity-change conversation (see [[Principle - Narrow Forever]]'s "two intake types, forever" ceiling).

## WHERE: Ecosystem

- Implements:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — makes the thesis's entity vocabulary explicit and testable; the thesis uses these five nouns and no others.
- Related:
  - [[Principle - Reject Like bun.report Drift]] — `bun.report` uses nouns like "Crash Report" and "Stack Trace" that belong to a different product category; drifting into that vocabulary signals a scope drift.
- Referenced by:
  - [[Principle - Minimal Public Surface Feels Calm]] — the tone Principle depends on this Standard. Warm / conversational register does not override canonical names; user-facing strings must still use "Report," "Debug Bundle," "Reporter," "Triager," and "Report Metadata Page" as written here.
  - [[Decision - Who This Service Is For]] — the Reporter definition (no identity, Fabro-version-only) is written for a self-hosted-Fabro Reporter; the Decision is the audience commitment that makes that definition coherent.
  - [[Decision - Capability URLs over Auth]] — the Reporter-has-no-identity and Report-Metadata-Page-is-capability-URL-addressed entries in this Standard are the direct vocabulary consequence of that Decision.
  - [[Decision - No Content Dedup]] — the one-Report-to-one-Debug-Bundle relationship in this Standard depends on not coalescing by content hash; the Decision preserves that vocabulary shape.
  - [[Decision - 30-Day Retention Default]] — the Report's "retention deadline (default 30 days)" authoritative field takes its default value from the Decision.
  - [[Agent - Reporter]] — must use "Reporter"; not "User," "Uploader," "Submitter." The Agent card expands the canonical definition into role context, goals, and anti-patterns, but inherits the exact wording here.
  - [[Agent - Triager]] — must use "Triager"; not "Team Member," "Maintainer," "Responder." The Agent card expands the canonical definition into role context including the load-bearing "never interacts with the intake service itself" property recorded here.
  - [[Component - Report]] — uses "Report" as its authoritative name, inherits the full authoritative field list from this Standard, and extends it with lifecycle and key couplings. No field may appear on the Component card that is not in this Standard's entry.
  - [[Component - Report Metadata Page]] — uses this exact page name, never shortened to "Metadata Page," "Receipt Page," "Landing Page," "Status Page," or "Dashboard." The Component card's visible-field set is drawn from this Standard's Report entry.
  - [[Standard - Competitive Context]] — the not-borrowed list uses the canonical nouns (Report, Debug Bundle, Reporter, Triager, Report Metadata Page) to name what `fabro.report` will not import from `bun.report` or adjacent categories; vocabulary drift into `bun.report`'s nouns is one of the earliest drift symptoms this Standard calls out.
  - [[Standard - Information Architecture]] — the IA Standard uses these canonical names exactly — the page is the Report Metadata Page; the URL addresses a Report; the CLI uploads a Debug Bundle. The IA's one-CLI-one-page vocabulary is coextensive with this Standard.
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — "Debug Bundle" remains an opaque tarball under A13; the Noun Vocabulary is unaffected by the operational monitoring posture, and no new user-facing term is introduced by that Decision.
- Forward-referenced, to be drafted in Phase 3 of the 2026-04-23 feedback-scope cascade (plain-text references; no wikilinks until the cards exist):
  - `Agent - Feedback Submitter` (manifest A15, to be drafted in Phase 3) — must use "Feedback Submitter"; not "User," "Reporter," "Submitter" alone, "Feedbacker," or any shortened form. The Agent card will expand the canonical definition here into role context, identity-model specifics (optional email + server-logged IP + Fabro version metadata), and anti-patterns, inheriting the exact wording.
  - `Component - Feedback` (manifest A16, to be drafted in Phase 3) — uses "Feedback" as its authoritative name. No field may appear on the Component card that is not listed in this Standard's Feedback entry, and the Component card will extend the entry with lifecycle and storage couplings.
  - `System - Feedback Storage & Notification` (manifest A17, to be drafted in Phase 3) — uses "Feedback" for the submission (not "Message," "Submission," "Note") and "Discord channel" as a downstream notification target, not a first-class noun on the service. Discord channel membership / governance is out of scope for this library.
- Conforming references (each uses these exact terms):
  - [[System - Retention, Storage, and Key Scheme]] — uses "Debug Bundle" for the tarball (not "Payload," "Archive," "Dump") and "Report" for the upload event with its sidecar metadata; the S3 key scheme and URL shape preserve the canonical vocabulary.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the card that owns the CLI UX; inherits the canonical command names from the "Canonical CLI Commands" subsection below and expands them with flags, progress, error, and delete mechanics.
  - [[Standard - Accessibility Policy]] — uses "Report Metadata Page" in heading and `<title>` copy, "Delete Report" on the button, and the canonical nouns in any ARIA labels.
- Provenance:
  - `docs/alexandria/alexandria-config.json` → `discovery.confirmed_entities` — authoritative mirror. This card and the config must remain identical by design. Any drift between the two is a defect in one of them; treat this card as authoritative for card-authoring and the config as authoritative for tooling, and reconcile.

## WHY: Rationale

- Strategy anchor: [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — a narrow service has no room for synonym drift; every noun earns its place by naming a thing a Reporter, Triager, or Fabro engineer actually interacts with in the service of team retrieval of Debug Bundles.
- Scope anchor: [[Principle - Narrow Forever]] — the closed five-noun set is itself an expression of narrow-forever; new nouns imply scope expansion and require Strategy-change approval, not a vocabulary update.
- Driver: Two Factory-mode builder agents working adjacent features will invent different words for the same thing (Report vs Upload, Reporter vs User, Report Metadata Page vs Receipt Page) unless the vocabulary is fixed in one authoritative place. Once synonym drift starts, CLI strings contradict docs, docs contradict code, and the library's link graph starts pointing at ghost nouns. The cost of preventing that is a single card. The cost of recovering from it, after 20 cards and several PRs have accumulated drift, is much higher.

## WHEN: Timeline

Established 2026-04-22, alongside [[Product Thesis - Pickup Point, Not Triage Surface]]. Expanded 2026-04-23 from five entities to seven when the product scope broadened from one intake type to two; **Feedback** and **Feedback Submitter** were added as first-class canonical nouns, and the CLI Commands table grew to include `fabro feedback`. Stable at seven entities. All entities were confirmed through direct product-owner input (initialize conversation 2026-04-22 for the first five; the 2026-04-23 `docs/alexandria/sources/feedback-flow.md` capture for the two added nouns). Add new nouns by updating both this card and `alexandria-config.json`; do not introduce a new noun via a single downstream card. A new noun that corresponds to a new intake type (a third kind of submission beyond Debug Bundle and Feedback) is not a vocabulary update — it is an identity change under [[Principle - Narrow Forever]] and requires a product-owner identity-change conversation first.

## HOW: Specification

### Canonical Entities

| Canonical Name        | Short Definition                                                                                                                                 | Authoritative Fields / Distinctions                                                                                                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Report                | The upload event and its public identity. Created when `fabro dump --upload` (or `fabro upload` against a previously-dumped tarball) successfully uploads a Debug Bundle to S3. | Fields: ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata tags, retention deadline (default 30 days). Delete capability is tied to URL possession.           |
| Debug Bundle          | The compressed tarball produced by `fabro dump` and uploaded by `fabro dump --upload` or `fabro upload`. Stored in S3, never served publicly.     | Internal structure is owned by the `fabro dump` command in the main Fabro repo and is out of scope for this library. The service treats a Debug Bundle as an opaque blob with a size.                                    |
| Reporter              | Whoever ran `fabro dump --upload` (or `fabro upload`). Has no identity on the service — no account, no login, no user record.                     | Identified only by Fabro version metadata captured at upload time. May optionally attach a short message via a CLI flag. Expected to share the URL out-of-band (GitHub, Slack, email).                                   |
| Triager               | A member of the Fabro team who retrieves Debug Bundles for bug investigation. Currently Bryan + team.                                             | **Never interacts with the intake service itself.** Retrieves bundles directly from S3 using their own AWS credentials. The service has no Triager-facing surface.                                                        |
| Report Metadata Page  | The minimal one-page landing page at a Report's URL. Shows Report ID, upload timestamp, bundle size, and a delete button. Does not expose contents.| The service's only rendered surface for the Debug Bundle flow. Capability-URL addressed (possession of URL grants view + delete). Never displays Debug Bundle contents; never exposes Reporter identity (there isn't one). Has no Feedback-flow equivalent — Feedback is fire-and-forget with no page. |
| Feedback              | The submission event and its storage identity for the Feedback flow. Created when `fabro feedback` successfully uploads a Feedback Submitter's message text (optional), file attachments (optional), and optional email to the service. | Fields: UUIDv7 ID (server-generated), message text (optional), file attachments manifest (optional), email (optional, from `-e`/`--email`), server-logged client IP (abuse protection), Fabro version / platform / arch (CLI-captured), timestamp, retention deadline (default 30 days). **No capability URL, no delete action, no rendered page** — the Feedback Submitter's involvement ends at CLI confirmation. Stored in S3 as two objects (`feedback.json` + `feedback.txt`) and pushed to Discord via webhook. |
| Feedback Submitter    | Whoever ran `fabro feedback`. **Not the same role as Reporter** — they have different identity models and different post-pickup mechanics. A Feedback Submitter is any `fabro` CLI user who chose to send voice (bug report, feature request, praise, complaint, general feedback) to the Fabro team. | Identity model: **optional email** (if `-e`/`--email` was passed), server-side IP logging (for abuse protection; see forward-referenced `Decision - IP Logging for Abuse Protection`, manifest A18, to be drafted in Phase 3), and Fabro version / platform / arch metadata. No account, no login, no cross-submission tracking. Expected involvement ends at CLI confirmation — **no URL is returned, no page is rendered, no delete action is offered**. |

### Canonical CLI Commands

The CLI surface is owned by [[Standard - Interaction Patterns (CLI + Delete)]]. This Standard does not specify flags, error shapes, or UX detail. It does, however, anchor the **canonical command names** so no downstream card invents or regresses them. Confirmed 2026-04-23:

| Command                  | Purpose                                                                                                   |
| ------------------------ | --------------------------------------------------------------------------------------------------------- |
| `fabro dump`             | Produces a Debug Bundle tarball locally. Does not upload.                                                 |
| `fabro dump --upload`    | Produces a Debug Bundle AND uploads it in one step. Returns the Report URL on success. Reporter's primary path. |
| `fabro upload`           | Uploads a previously-dumped Debug Bundle tarball. Supports `--dry-run` to verify the upload path without transferring bytes. |
| `fabro feedback`         | Submits Feedback (optional text, optional files, optional `-e`/`--email`) to the service. Fire-and-forget — returns a confirmation, not a URL. Feedback Submitter's only path. Added 2026-04-23. |

The pre-2026-04-23 names `fabro debug` (bundle producer) and `fabro report` (uploader) are **superseded**. The product domain `fabro.report` and the noun "Report" are unchanged — only the verbs a Reporter types changed.

`fabro feedback` was added 2026-04-23 when the product expanded to two intake types. Its shape is modeled on `bun feedback` (see [[Standard - Competitive Context]]) — a second CLI-intake template, parallel to how `bun.report`-the-URL-shape was the template for the Debug Bundle intake. The canonical flag shape is `-e, --email <email>`; feedback text and file arguments are positional; stdin is accepted via pipe.

### Usage Rules

1. Use the exact canonical name as written above — same capitalization, same wording. "Report" is the noun (the upload event and its public identity); it is distinct from the general English word "report" and is not a CLI subcommand name in this product. The CLI commands are `fabro dump`, `fabro dump --upload`, `fabro upload`, and `fabro feedback` (see HOW below for canonical definitions).
2. Use "Report" for the upload event / public identity of a Debug Bundle upload. Use "Debug Bundle" for the tarball itself. A Report contains a reference to a Debug Bundle; they are not synonyms.
3. Use "Reporter" only for someone who has run `fabro dump --upload` or `fabro upload`. Do not use "Reporter" to mean a Fabro user who has not yet encountered a bug or has not yet uploaded — there is no pre-Reporter state in this product.
4. Use "Triager" only for Fabro team members retrieving Debug Bundles or reading Feedback. The Triager role is **one role with two workflows** after the 2026-04-23 scope expansion — S3-direct retrieval for Debug Bundles, Discord-first reading (with optional S3 retrieval) for Feedback. Do not split "Triager" into two separate roles, and do not use "Triager" to mean a Reporter or a Feedback Submitter who is also debugging their own issue.
5. Use "Report Metadata Page" for the service's rendered surface for the Debug Bundle flow. Do not shorten to "Metadata Page," "Receipt," or "Landing Page" in cards, docs, or log lines; shortening invites synonym drift.
6. Use "Feedback" for the submission entity of the Feedback flow. Do not use "Report" or "Debug Bundle" to describe a Feedback, and do not use "Feedback" to describe a Report or a Debug Bundle — they are three distinct entities with three distinct identity models and storage shapes. A Feedback is UUIDv7-addressed and fire-and-forget; a Report is capability-URL-addressed with a delete action; a Debug Bundle is the opaque tarball referenced by a Report.
7. Use "Feedback Submitter" only for someone who has run `fabro feedback`. **Reporter and Feedback Submitter are not interchangeable**: the Reporter's identity model is no-identity-beyond-Fabro-version-metadata, while the Feedback Submitter's identity model is optional-email-plus-server-logged-IP. A given human being might, at different moments, be both a Reporter (when running `fabro dump --upload`) and a Feedback Submitter (when running `fabro feedback`), but the *roles* are distinct and should not be conflated in prose, CLI copy, or Discord post wording. Do not use "User," "Submitter" alone, "Feedbacker," or any other shortened or generic form.

### Common Mistakes (Anti-Examples)

- **Wrong:** Referring to a Debug Bundle as a "Report" because the CLI returns a URL called a "Report URL." The Report is the upload event and public identity; the Debug Bundle is the payload inside it. They are separate nouns.
- **Wrong:** Calling a Reporter a "User" in CLI copy, error messages, or card prose. The product deliberately has no user concept — only Reporters (who have no identity) and Triagers (who use S3 directly).
- **Wrong:** Using "Triager" in any service-facing context (e.g., "this endpoint is for Triagers"). Triagers never interact with the service. If a card describes a Triager-facing surface, the card is wrong about either the Triager role or about the service's scope.
- **Wrong:** Describing the receipt page as a "dashboard" or "triage UI." It is a Report Metadata Page — one URL, one Report, one page, minimal metadata + delete.
- **Wrong:** Importing `bun.report`'s vocabulary ("Crash Report," "Stack Trace," "Symbolicator," "Feedback"). These nouns name features that `fabro.report` does not have; using them signals a scope drift. See [[Principle - Reject Like bun.report Drift]].
- **Wrong:** Inventing a new noun in a downstream card (e.g., "Submission," "Upload Record," "Crash Dump") without first proposing an addition to this card. Downstream drift starts here.
- **Wrong:** Calling a Feedback Submitter a "Reporter" — e.g., "the Reporter ran `fabro feedback` and sent a message." The Reporter role is tied to the Debug Bundle flow; the Feedback Submitter role is tied to the Feedback flow. Their identity models differ (no identity vs. optional email + logged IP), and conflating them misrepresents the product's deliberate asymmetric-identity-per-intake design.
- **Wrong:** Calling a Feedback a "Report" — e.g., "the Feedback Submitter's Report was posted to Discord." Report is the upload event / public identity for a Debug Bundle upload; Feedback is the submission entity for the Feedback flow. Using "Report" as a general term for "any intake" collapses two canonical entities into one, which is the exact atomicity violation this Standard prevents.
- **Wrong:** Using "Upload" as a verb for a Feedback submission ("the Feedback Submitter uploaded their message"). Debug Bundles are uploaded (the tarball is transferred to S3); Feedbacks are **submitted** (the CLI sends the payload to the server, which writes to S3 and posts to Discord). The verb asymmetry mirrors the entity asymmetry.
- **Wrong:** Conflating the two intake flows in prose — e.g., "the intake flow captures bundles and feedback interchangeably." There is **one service with two intake types**, and those types are parallel but not interchangeable. Cards, CLI copy, and doc prose should name the flow explicitly when flow-specific behavior is at stake. When genuinely talking about both flows together (e.g., "both intake types retain data for 30 days"), "intake type" is the correct umbrella noun, not "Report" or "Feedback."

## Sources

- `docs/alexandria/sources/product-vision.md` (L92–100 "Confirmed Entities"; L53–64 Users section)
- `docs/alexandria/sources/feedback-flow.md` (2026-04-23 — Feedback, Feedback Submitter entity definitions; `fabro feedback` CLI shape; identity-model asymmetry vs Reporter)
- `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` (Part 3 — two new canonical nouns; Part 4 — usage rules specification)
- `docs/alexandria/alexandria-config.json` (`discovery.confirmed_entities`)
- `docs/alexandria/inventory/card-manifest.md` (F2 entry)

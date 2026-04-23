# Standard - Noun Vocabulary

## WHAT: Definition

The canonical name, definition, and common-mistake list for every first-class entity in the `fabro.report` product. Five entities — Report, Debug Bundle, Reporter, Triager, Report Metadata Page — each with exactly one canonical spelling and one authoritative definition. Every other card, every CLI string, every log line, every doc, and every agent-generated surface must use these exact terms in these exact senses. New nouns may not be introduced without first being added here.

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
- Conforming forward references (planned in `docs/alexandria/inventory/card-manifest.md`, not yet drafted — each must use these exact terms when drafted):
  - `System - Retention, Storage, and Key Scheme` (manifest A5) — must use "Debug Bundle" for the tarball; not "Payload," "Archive," "Dump."
- Provenance:
  - `docs/alexandria/alexandria-config.json` → `discovery.confirmed_entities` — authoritative mirror. This card and the config must remain identical by design. Any drift between the two is a defect in one of them; treat this card as authoritative for card-authoring and the config as authoritative for tooling, and reconcile.

## WHY: Rationale

- Strategy anchor: [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — a narrow service has no room for synonym drift; every noun earns its place by naming a thing a Reporter, Triager, or Fabro engineer actually interacts with in the service of team retrieval of Debug Bundles.
- Scope anchor: [[Principle - Narrow Forever]] — the closed five-noun set is itself an expression of narrow-forever; new nouns imply scope expansion and require Strategy-change approval, not a vocabulary update.
- Driver: Two Factory-mode builder agents working adjacent features will invent different words for the same thing (Report vs Upload, Reporter vs User, Report Metadata Page vs Receipt Page) unless the vocabulary is fixed in one authoritative place. Once synonym drift starts, CLI strings contradict docs, docs contradict code, and the library's link graph starts pointing at ghost nouns. The cost of preventing that is a single card. The cost of recovering from it, after 20 cards and several PRs have accumulated drift, is much higher.

## WHEN: Timeline

Established 2026-04-22, alongside [[Product Thesis - Pickup Point, Not Triage Surface]]. Stable. All five entities were confirmed through the initialize conversation with the product owner and are cross-referenced against `alexandria-config.json`. Add new nouns by updating both this card and the config; do not introduce a new noun via a single downstream card.

## HOW: Specification

### Canonical Entities

| Canonical Name        | Short Definition                                                                                                                                 | Authoritative Fields / Distinctions                                                                                                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Report                | The upload event and its public identity. Created when `fabro report` successfully uploads a Debug Bundle to S3.                                  | Fields: ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata tags, retention deadline (default 30 days). Delete capability is tied to URL possession.           |
| Debug Bundle          | The compressed tarball produced by `fabro debug` and uploaded by `fabro report`. Stored in S3, never served publicly.                             | Internal structure is owned by the `fabro debug` command in the main Fabro repo and is out of scope for this library. The service treats a Debug Bundle as an opaque blob with a size.                                    |
| Reporter              | Whoever ran `fabro report`. Has no identity on the service — no account, no login, no user record.                                                | Identified only by Fabro version metadata captured at upload time. May optionally attach a short message via a CLI flag. Expected to share the URL out-of-band (GitHub, Slack, email).                                   |
| Triager               | A member of the Fabro team who retrieves Debug Bundles for bug investigation. Currently Bryan + team.                                             | **Never interacts with the intake service itself.** Retrieves bundles directly from S3 using their own AWS credentials. The service has no Triager-facing surface.                                                        |
| Report Metadata Page  | The minimal one-page landing page at a Report's URL. Shows Report ID, upload timestamp, bundle size, and a delete button. Does not expose contents.| The service's only rendered surface. Capability-URL addressed (possession of URL grants view + delete). Never displays Debug Bundle contents; never exposes Reporter identity (there isn't one).                          |

### Usage Rules

1. Use the exact canonical name as written above — same capitalization, same wording. "Report" is distinct from "report" (the latter is a CLI subcommand, a verb, or the general noun).
2. Use "Report" for the upload event / public identity. Use "Debug Bundle" for the tarball itself. A Report contains a reference to a Debug Bundle; they are not synonyms.
3. Use "Reporter" only for someone who has run `fabro report`. Do not use "Reporter" to mean a Fabro user who has not yet encountered a bug or has not yet uploaded — there is no pre-Reporter state in this product.
4. Use "Triager" only for Fabro team members retrieving bundles. Do not use "Triager" to mean a Reporter who is also debugging their own issue — the Triager role is about S3 retrieval, not about who is investigating.
5. Use "Report Metadata Page" for the service's rendered surface. Do not shorten to "Metadata Page," "Receipt," or "Landing Page" in cards, docs, or log lines; shortening invites synonym drift.

### Common Mistakes (Anti-Examples)

- **Wrong:** Referring to a Debug Bundle as a "Report" because the CLI returns a URL called a "Report URL." The Report is the upload event and public identity; the Debug Bundle is the payload inside it. They are separate nouns.
- **Wrong:** Calling a Reporter a "User" in CLI copy, error messages, or card prose. The product deliberately has no user concept — only Reporters (who have no identity) and Triagers (who use S3 directly).
- **Wrong:** Using "Triager" in any service-facing context (e.g., "this endpoint is for Triagers"). Triagers never interact with the service. If a card describes a Triager-facing surface, the card is wrong about either the Triager role or about the service's scope.
- **Wrong:** Describing the receipt page as a "dashboard" or "triage UI." It is a Report Metadata Page — one URL, one Report, one page, minimal metadata + delete.
- **Wrong:** Importing `bun.report`'s vocabulary ("Crash Report," "Stack Trace," "Symbolicator," "Feedback"). These nouns name features that `fabro.report` does not have; using them signals a scope drift. See [[Principle - Reject Like bun.report Drift]].
- **Wrong:** Inventing a new noun in a downstream card (e.g., "Submission," "Upload Record," "Crash Dump") without first proposing an addition to this card. Downstream drift starts here.

## Sources

- `docs/alexandria/sources/product-vision.md` (L92–100 "Confirmed Entities"; L53–64 Users section)
- `docs/alexandria/alexandria-config.json` (`discovery.confirmed_entities`)
- `docs/alexandria/inventory/card-manifest.md` (F2 entry)

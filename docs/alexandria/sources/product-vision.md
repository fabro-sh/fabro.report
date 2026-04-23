# Source Material — Product Vision (1.1)

**Status:** raw source material, not a card. Captured for Conan to inventory.
**Area:** 1.1 Product Vision (Foundation tier, Vision & Strategy domain)
**Captured:** 2026-04-22
**Source:** first-session conversation between Raven and Bryan Helmkamp (project owner).
**Product:** `fabro.report`
**Repo:** `~/p/fabro-sh/fabro.report` (greenfield — no code yet)

---

## Vision Summary (one sentence)

`fabro.report` is a deliberately narrow debug-bundle intake service: a Fabro user hits a bug, runs `fabro report`, and gets back a short URL that the Fabro team can use to retrieve the bundle from S3 — nothing more.

---

## What `fabro.report` Is

A standalone intake service for debug bundles produced by users of the Fabro product (an AI-powered workflow orchestration platform living at `~/p/fabro-sh/fabro`; workflows are defined as Graphviz graphs, each node a stage executed by the workflow engine).

The flow, end-to-end:

1. A Fabro user encounters a bug.
2. They run `fabro report` — a CLI subcommand of the main Fabro binary.
3. That command takes the tarball produced by `fabro debug`, uploads it to S3, and returns a short URL.
4. The user shares that URL (in a GitHub issue, Slack message, email — wherever).
5. A Fabro team member ("Triager") uses their own AWS credentials to retrieve the bundle directly from S3 for investigation.
6. Unless deleted earlier, the report expires after 30 days (default retention).

The URL itself resolves to a minimal one-page metadata receipt showing the Report ID, upload timestamp, and bundle size, plus a delete button. It does **not** expose the bundle contents.

## What `fabro.report` Is Not (load-bearing)

The user described this product as "similar to `bun.report` but for Fabro." That comparison was clarified in the conversation and the clarification is the most important thing in this document:

> **"Similar to `bun.report`"** means only the **intake URL shape** — upload a tarball, receive a secret URL. Nothing downstream of that is borrowed.

Specifically out of scope, even though `bun.report` (at `~/p/oven-sh/bun.report`) does all of them:

- No symbolication of crash reports
- No Sentry integration
- No in-browser rendering of the bundle contents
- No markdown rendering of structured crash data
- No feedback collection workflows
- No team dashboards or triage UI on the service itself

This distinction is load-bearing because Factory-mode builder agents, told this product is "like `bun.report`," will drift toward the fuller `bun.report` shape unless the narrower scope is explicit in the library. The Anti-Patterns card (area 3.5) is expected to name this drift risk directly; the Vision needs to state the narrow scope positively.

## Users

Two roles, with asymmetric identity:

### Reporter
- A Fabro user who hit a bug and ran `fabro report`.
- **No authentication. No login. No account.**
- Identified only by the **Fabro version** captured as metadata at upload time (outside the bundle, as structured metadata).
- May optionally attach a short message via a CLI flag, e.g. `fabro report -m "cargo install hangs on CI"`.
- Expected to share the resulting URL out-of-band (GitHub, Slack, email).

### Triager
- A member of the Fabro team. Currently: Bryan + team.
- **Does not interact with the intake service itself.** They retrieve bundles directly from S3 using their own AWS credentials.
- The service's public surface is effectively write-only-from-CLI plus a metadata receipt; triage happens entirely out-of-band.

## Security Model

Capability URLs. Possession of the URL grants:
- The ability to view the metadata receipt.
- The ability to delete the report.

There is no authentication on the public surface. The bundle contents are never exposed through the service — only through S3, which requires triager credentials. The "secret URL" is the entire access-control mechanism for the public metadata + delete capability.

## Confirmed Product Decisions (from the conversation)

These are decisions already made by Bryan during initialize; they are not up for re-litigation by a Factory-mode builder agent without being flagged as a decision change.

- **Strategic WHY (confirmed 2026-04-22).** Fabro users run Fabro on their own systems. When it fails for them, the Fabro team wants to help. To help, the team needs the data from those Fabro runs. `fabro.report` exists to make it easy for users to get that data to the team. The motivation is *support effectiveness for self-hosted users*, not support-SLA formalization, not marketing telemetry, not compliance.
- **"Narrow forever" (confirmed 2026-04-22).** The intake-only scope is a permanent identity decision, not a v1 simplification. Assume forever. This makes `fabro.report` structurally different from `bun.report` as a product, not just a stripped-down first pass at it. Derived cards — especially Anti-Patterns (3.5) and Roadmap (5.3) — should treat scope expansion (symbolication, rendering, team dashboards, triage UI) as permanently out-of-scope, not as "future work."
- **CLI command names (confirmed 2026-04-23).** The bundle-producing command is `fabro dump` (not `fabro debug` — rename confirmed). The upload paths are `fabro dump --upload` (one-step dump + upload) and `fabro upload` (upload a previously-dumped tarball; supports `--dry-run`). Supersedes earlier references in this source to `fabro report` as the CLI command and `fabro debug` as the bundle-producing command. The noun "Report" is unchanged; only the verbs (CLI command names) moved. Cascade applies across the library — see `docs/alexandria/sources/cli-interaction-patterns.md`.
- **Scope:** upload → S3 → URL. Nothing more on the service's public surface.
- **Retention:** 30-day default expiration.
- **Metadata captured at upload (outside the bundle):**
  - Fabro version
  - Optional reporter message (CLI flag)
  - Arbitrary key/value metadata tags (no first-class "issue link" concept — that's just one possible tag)
- **Dedup:** none. No content-hash dedup. Rationale: bundles expire anyway.
- **Auth:** none on the public surface. Capability URLs only.
- **Bundle content rendering:** never. Contents are not exposed via the service.
- **Triager access path:** direct S3, using triager's own credentials. Not through this service.
- **Bundle structure:** owned by the `fabro debug` command in the main Fabro repo. Out of scope for this library.

## Confirmed Entities

(Mirrored from `alexandria-config.json` → `discovery.confirmed_entities`. Listed here so the Vision source has the vocabulary in one place; the authoritative Noun Vocabulary work is area 2.2.)

- **Report** — the upload event and its public identity. Fields: ID, URL, upload timestamp, bundle size, optional reporter message, Fabro version, arbitrary key/value metadata, retention deadline.
- **Debug Bundle** — the compressed tarball produced by `fabro debug`. Stored in S3, never served publicly. Internal structure is out of scope for this repo.
- **Reporter** — whoever ran `fabro report`. No identity, no login.
- **Triager** — a Fabro team member who retrieves bundles via direct S3 access.
- **Report Metadata Page** — the minimal one-page landing page at the returned URL. Shows ID / timestamp / size plus a delete button. Does not expose bundle contents.

## Template Relationship: `bun.report`

- **Location:** `~/p/oven-sh/bun.report`.
- **Borrowed:** the intake URL shape. Upload a tarball, get a short secret URL, share it.
- **Not borrowed:** symbolication, Sentry integration, in-browser crash viewing, markdown rendering, feedback collection, anything that turns the service into a triage product rather than a pickup point.
- **Why this matters for the Vision:** without this scope distinction recorded, an autonomous agent implementing any given feature will assume "like `bun.report`" means "shaped like `bun.report`'s full product" and will drift. The Vision must state the narrow shape positively so derived cards inherit it.

## Parent Product Context

- **Fabro** (`~/p/fabro-sh/fabro`) — Rust monorepo. AI-powered workflow orchestration. Workflows are Graphviz graphs; each node is a stage run by the workflow engine.
- The `fabro debug` command already exists in Fabro's CLI surface and produces the tarball that `fabro report` uploads.
- The `fabro report` CLI subcommand is **new** — it is not yet present in Fabro's documented command set (`run, exec, serve, validate, parse, cp, model, doctor, install, ps, system prune` per Fabro's AGENTS.md). Building this intake service implies adding the `report` subcommand to Fabro.

## Raven's Inferences (NOT confirmed by Bryan — flag for review)

These are framings Raven introduced during initialize that Bryan did not explicitly confirm or reject. They are plausible but should be treated as inference until confirmed:

- That the service is explicitly aimed at the **current Fabro user population**, not at external users of a future managed Fabro offering. (The confirmed Strategic WHY mentions "users who run fabro on their own systems," which leans strongly toward self-hosted users — but doesn't explicitly exclude a future hosted offering's users.)
- That "headless" is a meaningful adjective for the product's character. Bryan used "headless" but then confirmed a one-page metadata receipt does exist — so the product is **near-headless**, not literally headless. Future cards should say "minimal public surface" rather than "headless" to avoid overclaiming.

## Open Questions (flagged for human judgment)

None of these block this source material, but Conan or a human should resolve them before the Vision card is finalized:

1. **Success criteria.** What does "this product is working" look like? No measurable success criterion was stated. Candidates: bundle volume, time-to-retrieval for triagers, URL-sharing prevalence in GitHub issues. Unresolved.
2. **Who owns the domain?** `fabro.report` implies a dedicated domain registration; that's not discussed in the conversation. Probably orthogonal to Vision but worth logging.
3. **Relationship to any managed-Fabro offering.** If Fabro ever has hosted users, do their bundles land here too, or somewhere else with different auth? Confirmed WHY cites "users who run fabro on their own systems" — self-hosted is the clear primary audience — but a hosted Fabro's users' bundles remain unaddressed.

*Resolved 2026-04-22:*
- Strategic WHY — moved to Confirmed Product Decisions.
- "Narrow forever" — moved to Confirmed Product Decisions. Anti-Patterns (3.5) and Roadmap (5.3) should treat scope expansion as permanently out-of-scope, not as "future work."

## Notes for the Future Vision Card

When this is turned into a `Product Thesis - ...` or `Principle - ...` card in `/rationale/`:

- The **narrow-scope vs `bun.report`** framing belongs in the WHY dimension — it's the core rationale, not just a scope note.
- The **capability-URL security model** is a decision, not a vision element. It should link out to a Key Decisions Log entry (5.1), not be absorbed into the Vision.
- The **30-day retention** and **no dedup** choices are likewise decisions, not vision.
- The **load-bearing distinction** — the thing the Vision card must carry that nothing else will — is: *this product is intentionally a pickup point, not a triage surface.*

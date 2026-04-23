# Source Material — Feedback Flow

**Status:** raw source material, not a card. Captures the feedback-intake flow confirmed 2026-04-23.
**Captured:** 2026-04-23
**Source:** direct confirmation from Bryan Helmkamp.
**Feeds:** new library cards (to be decided by Conan Inventory pass); also updates to F1, F2, F3, F5, F6 for the scope-expansion framing.

---

## 🚩 Scope Expansion — Product Identity Update

`fabro.report` is now a two-intake service: **Debug Bundles** (existing, via `fabro dump --upload` / `fabro upload`) and **Feedback** (new, via `fabro feedback`). This changes the library's load-bearing "narrow forever" framing:

- **Absolute ceiling (confirmed 2026-04-23):** exactly these two intake types, forever. No third intake surface is allowed without an identity-change conversation with the product owner. "Narrow forever" applies *at each of the two intake types* — and at the product level, "two-intake-types-forever" is itself a narrow-forever commitment.
- **F6 Narrow Forever** needs an explicit update: the out-of-scope list's "feedback collection" entry was correct in its original context (no post-upload feedback on the Debug Bundle flow); it does NOT reject this standalone `fabro feedback` command, which is a separate intake shape, not a bundle-flow extension.
- **F3 Reject Like bun.report Drift** needs the Sentry-shape note pattern applied to "feedback": the drift rejection targets "post-upload feedback forms on the Debug Bundle flow," not the standalone `fabro feedback` CLI.

---

## Confirmed Shape

### The CLI

`fabro feedback` — a new subcommand of the main `fabro` CLI. Requires `fabro` CLI to be installed (not a standalone binary). Modeled after `bun feedback`:

```
fabro feedback [options] [feedback text ... | files ...]

Options
  -e, --email <email>   Set the email address used for this submission
  -h, --help            Show this help message and exit

Examples
  fabro feedback "Love the new release!"
  fabro feedback report.txt details.log
  echo "please document X" | fabro feedback --email you@example.com
```

### Scope

**Anything counts as feedback.** Bug reports, feature requests, praise, complaints, general voice — the content is not validated, classified, or filtered by the service. Whatever the user types or pipes in is forwarded intact.

### Server Endpoint

`POST /v1/feedback` — accepts the feedback payload, logs to server logs, assigns a **UUIDv7** as the feedback ID, saves to S3, posts to Discord, returns a success confirmation to the CLI.

### Data Captured

- UUIDv7 (server-generated)
- Client IP (server-captured at request time, for abuse protection / basic attribution)
- Submitted text (if provided)
- Attached files (if provided)
- Email (if provided via `-e` / `--email`)
- Fabro version / platform / arch (captured by the CLI as part of the request)
- Timestamp

### S3 Storage

The feedback is stored in S3 as **two objects** per submission:

- `feedback.json` — structured metadata (ID, timestamp, IP, email, Fabro version / platform / arch, file manifest, any other fields)
- `feedback.txt` — the message text only (not the file attachments; those are stored separately if present)

**Retention:** 30 days (matching Debug Bundle retention per A11).
**Bucket:** same bucket as Debug Bundles (`fabro-report-bundles`), under a distinct prefix.
**Key scheme:** date-partitioned, similar to Debug Bundles. Exact layout TBD — likely `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/feedback.json` + `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/feedback.txt` + optional `feedback/<YYYY>/<MM>/<DD>/<uuidv7>/attachments/...`.

### Discord Webhook (New Team Surface)

- Posts to a **new private Discord channel** (to be created) via a webhook URL stored as an env var (e.g., `FEEDBACK_DISCORD_WEBHOOK_URL`).
- Uses Discord's component-v2 payload pattern (adapt from `bun feedback`'s `sendToDiscord` implementation).
- Message content modeled on `bun feedback`'s shape:
  - **Header:** `## <email>` if email provided, else a neutral placeholder
  - **Sysinfo one-liner:** `Fabro v<version> <platform> <arch>`
  - **Message body:** the feedback text
  - **File attachments tree** (if any): ASCII list of files, first 20 + "… and N more"
  - **Footer:** feedback ID, optional IP, link to the S3 object (presigned URL if useful)
- If a tarball of attachments is produced, it's attached to the Discord message itself.
- If the Discord webhook call fails, a second webhook call should fire with the error plus `discord-payload.json` and `inputs.json` as self-reporting files.

### User Return Value (CLI)

**Fire-and-forget from the user's perspective.** The CLI prints a confirmation that the feedback was received. No URL is returned (unlike the Debug Bundle flow). No metadata page is rendered. The Feedback Submitter's involvement ends at "it was sent."

---

## Key Differences From the Debug Bundle Flow

| Dimension | Debug Bundle Flow | Feedback Flow |
|---|---|---|
| CLI command | `fabro dump --upload` / `fabro upload` | `fabro feedback` |
| Primary entity | Report (upload event + public identity) | Feedback (submission event) |
| User role | Reporter | Feedback Submitter *(new role — not the same as Reporter)* |
| Returns a URL? | Yes — capability URL | No — fire-and-forget |
| Has a public page? | Yes — Report Metadata Page | No — nothing rendered publicly |
| Identity of submitter? | None (Fabro version only) | Optional email; IP server-logged |
| Team notification? | No (Triagers poll S3) | Yes (Discord webhook posts) |
| Access model | Capability URL grants view + delete | No user-side access after submission |
| Retention | 30 days (A11) | 30 days (matches) |

The flows are parallel but not identical. Feedback is strictly fire-and-forget; Debug Bundles give the Reporter back a URL and a delete button.

---

## New Team-Facing Surface: Discord

This is the product's **first team-facing surface**. F1/F6 previously held the line that "the service has no team-facing surface"; Discord is a team-facing surface, just hosted externally. The honest framing:

- The `fabro.report` service itself still has no team-facing UI (no dashboard, no listing endpoint, no login). This survives.
- The service's *behavior* now includes posting team notifications to an external channel. This is new.
- Triagers' workflow for feedback is: read Discord → click through to S3 link if they want to see the full submission. Triagers' workflow for Debug Bundles remains: see URL in GitHub issue / Slack / email → retrieve from S3 directly. **Different surfaces per intake type.**

The Discord webhook is also this library's first integration with a third-party SaaS for notification purposes. F3's drift list rejects third-party integrations that act as downstream bundle processors (Sentry as a crash parser); Discord is a notification target, not a processor. Distinction is similar in shape to the Sentry distinction (A13).

---

## Implications Raven Observes (flag for review)

These are direct implications of the confirmed scope expansion. Treat as defaults unless corrected.

- **New Agent: Feedback Submitter.** A new persona card, parallel to A1 Reporter. Identity model: optional email, server-side IP, Fabro version metadata. Does not interact with the service post-submission (no URL, no delete).
- **New Component: Feedback.** A new first-class entity, parallel to A3 Report. Fields: UUIDv7 ID, message text, file attachments manifest, email (optional), IP, Fabro version / platform / arch, timestamp, retention deadline.
- **Triager role broadens.** A2 Triager's primary Debug Bundle retrieval path (direct S3 with own credentials) is unchanged. For feedback, Triagers read the Discord channel; they can still retrieve S3 objects for the full submission. A2 card likely needs a second behavioral section.
- **F1 Product Thesis identity update.** "Pickup point, not triage surface" remains; scope expands from bundles-only to bundles-and-feedback. The two pickup shapes are parallel but not identical.
- **F6 Narrow Forever update.** "Exactly two intake types, forever" is the new identity commitment. The existing out-of-scope list (symbolication, rendering, team dashboards, triage UIs, per-user accounts, cross-upload identity tracking) stays — these items apply equally to both flows.
- **F2 Noun Vocabulary expansion.** The canonical noun set grows from 5 to at least 7: add Feedback, Feedback Submitter. Possibly an 8th if the Discord channel or webhook is named as a first-class entity — probably not needed.
- **F3 drift list clarification.** The "feedback collection" drift item should be rewritten (similar to the Sentry distinction pattern) to reject *post-upload feedback on the Debug Bundle flow* while allowing the standalone `fabro feedback` intake. Sam's new F3 note pattern is well-tested.
- **New Decisions likely needed:**
  - Decision - Discord Webhook for Feedback Notification (why Discord, what gets posted, how failures self-report)
  - Decision - IP Logging for Abuse Protection (why IPs, what's the retention, scope of use)
  - Decision - Fire-and-Forget Feedback (no URL, no delete — why this diverges from Debug Bundle pattern)
- **New System likely needed:**
  - System - Feedback Storage & Notification (parallel to A5 for Debug Bundles — feedback.json + feedback.txt, Discord webhook posting, failure self-reporting)

---

## Open Questions (flagged for human judgment)

1. **File attachments to Discord:** if Feedback Submitter attaches a 40MB log file, does that still get attached to the Discord message? Discord has a file size limit (usually 8MB without Nitro, 25MB with Nitro Basic). Bryan's answer: graceful fallback? Truncation? Exclude large attachments?
2. **Feedback size cap:** is there a cap on the message text or total attachment size, similar to the 50MB Debug Bundle cap?
3. **Email validation:** the `--email` flag — do we validate format, or accept any string? Treat as opaque string, similar to the optional reporter message on Debug Bundles?
4. **Feedback ID format vs Debug Bundle ID format:** Debug Bundles use 32-hex-char random IDs (128 bits entropy for capability URLs). Feedback uses UUIDv7 (time-ordered, different shape). Worth capturing as an explicit decision — they serve different purposes (capability URL vs database-style ID), so different shapes are defensible.
5. **Server log shape for feedback:** what does "log to server log" look like? Structured JSON? Plain text? Level (info)? Worth pinning down when A7-style CLI/server-interaction cards get drafted.
6. **Discord rate limits:** Discord webhooks have rate limits. If abuse floods feedback, the webhook calls could get rate-limited or blocked. Need a minimal mitigation even though Bryan said "no rate limits" for the Debug Bundle flow — feedback flow may need a separate IP-based soft rate limit specifically to protect the Discord webhook.

---

## What This Source Does Not Cover

- **The Discord channel's governance** — who has access, who moderates, etc. Presumably the Fabro team; not a library concern.
- **Feedback-to-GitHub-issue workflow** — if a Triager wants to turn a piece of feedback into a GitHub issue, that's out-of-band triage, not a service feature.
- **Feedback analytics** — aggregating feedback by theme, etc. Would be a scope expansion beyond the ceiling. Not in scope.

---

## Notes for Future Cards

When Conan inventories this source:

- Consider whether a new Foundation-adjacent card is needed to carry the "two-intake-types-forever" identity commitment explicitly, or whether updates to F1 + F6 suffice.
- The parallelism between the two flows is worth making visible in the library structure (e.g., paired Agent cards, paired Component cards, paired System cards).
- The Discord integration is the first non-AWS third-party dependency in the library. A Decision card naming the webhook pattern and the failure-self-reporting behavior is probably worth a first-class slot.
- The IP logging Decision is small but worth capturing — it's the first capture of Reporter-adjacent identity data (IP) the library has introduced; the rationale ("abuse protection + basic attribution") needs to be pinned.

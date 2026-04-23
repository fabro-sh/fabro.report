# Decision - IP Logging for Abuse Protection

## WHAT: The Choice

`fabro.report` **logs the client IP on every `POST /v1/feedback` request** and stores it as a field on the Feedback (in `feedback.json`). The IP is retained per the standard 30-day S3 lifecycle rule applied to every object under the `feedback/` prefix — no separate retention policy, no indefinite archive, no cold-storage cascade. The IP is written once, at submission time, and expires with the Feedback it accompanies.

**Scope of use is strictly abuse protection and basic submission-source attribution.** The IP exists so that a Triager investigating a pattern of abuse — submission floods, repeated low-quality automated content, targeted Discord-webhook exhaustion attempts, coordinated spam — can correlate Feedbacks by source and take out-of-band action (manual S3 cleanup, IP-level block at the infrastructure layer if it ever becomes necessary, etc.). Beyond that, the IP is not used.

**The IP is explicitly NOT used for:**

- **Marketing telemetry** — counted against [[Principle - Strategy Support Effectiveness for Self-Hosted Users]]'s three anti-goals; the Strategy explicitly refuses adoption-measurement instrumentation, and a field that could be used to measure reach is a field that will be used to measure reach unless the scope is ruled out at the Decision layer.
- **Adoption or usage metrics** — same anti-goal. Counts of distinct submission-IPs, submission-rate-by-IP dashboards, "who's using us" analytics — all out of scope, regardless of how non-identifying the aggregation is.
- **Compliance audit trail** — counted against the same Strategy anti-goal. The service is not a compliance artifact; the IP log is not an audit log; retention matches the Feedback itself precisely because a longer-lived IP record would feel like audit storage rather than abuse-protection signal.
- **Cross-submission user identification** — the Feedback Submitter has no identity across submissions by design (per [[Principle - Narrow Forever]]'s "no cross-submission identity tracking" commitment). Joining Feedbacks by IP to say "this is probably the same Feedback Submitter as last week" is forbidden, even in internal tooling that never surfaces the join publicly.
- **Cross-intake-type identification** — no joining a Feedback's IP against a Debug Bundle's (nonexistent) IP record; Debug Bundles do not capture submitter IP and cannot be used this way, and even if they did, the commitment would hold.
- **Geolocation surfacing** — no IP → country/region/city lookup on the service. The service does not call geolocation APIs, does not enrich the Discord post with geo inference, and does not surface location context anywhere.

The load-bearing phrasing is *IP is an abuse-protection signal, nothing else, ever*. An agent reading this card should not come away believing "IP is logged for v1 abuse protection and we'll use it for analytics later when it matters." The anti-uses above are permanent, not phased. Using IP for analytics would be a Strategy change, not an implementation expansion.

This is the **library's first capture of identity-adjacent data**. Before 2026-04-23, the library had zero identity capture — Reporters had no identity, only Fabro version metadata. The 2026-04-23 scope expansion introduced the Feedback flow, which introduced two identity-adjacent fields: the optional Feedback Submitter email (opaque attribution) and the server-logged IP (abuse protection). This Decision exists to pin the scope of the second of those fields so future agents can find "why do we log IPs?" without having to dig it out of a System card's implementation section, and to make the scope commitment visible at the Decision layer where scope changes are adjudicated.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — Strategy has three explicit anti-goals: not a formal support SLA, not compliance posture, not marketing telemetry. This Decision's scope commitment ("IP for abuse protection only") directly defends the **no marketing telemetry** and **no compliance posture** anti-goals from drift. IP logging is structurally seductive for both: it is the kind of field a well-meaning agent would convert into an analytics signal or an audit trail if the scope were not pinned explicitly.
  - [[Principle - Narrow Forever]] — the F6 out-of-scope list includes "cross-submission identity tracking (across either flow)" and names IP specifically: *"IP logging on the Feedback flow is scoped strictly to abuse protection — not cross-submission tracking, not attribution analytics, not cross-intake-type joining."* This Decision is the positive choice that makes that rejection concrete at the Decision layer. Per-IP analytics, per-IP user modeling, cross-submission IP joins are all on F6's permanently-out-of-scope list.
- Paired with:
  - [[Decision - Fire-and-Forget Feedback]] — A19 names the no-post-submission-surface commitment; this Decision names the identity-data-scope commitment. Together they constrain what the Feedback flow captures and what it does with what it captures: the flow captures optional email + server-logged IP + Fabro version / platform / arch, and uses each strictly for the scope the respective card defines (Discord post header for email, abuse-protection investigation for IP, Discord sysinfo one-liner for version/platform/arch). No field's use widens past its stated scope.
  - [[Decision - Who This Service Is For]] — the audience commitment. Logging an IP is calibrated to self-hosted Fabro users submitting Feedback from their own networks; this Decision inherits the audience shape and does not extend to any future population (hosted-Fabro users, third-party support partners). An audience change under C4 would be one of the conditions that could require revisiting this Decision.
- Vocabulary:
  - [[Standard - Noun Vocabulary]] — the **Feedback Submitter** definition explicitly names "server-logged client IP (abuse protection)" as part of the identity model and links to this card. The **Feedback** entry names `ip` as an authoritative field. This Decision is the rationale behind both of those vocabulary choices.
- Related:
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — A13's "Sentry is for service errors, not bundle processing / not feedback processing" boundary and this Decision's "IP is for abuse protection, not analytics / not compliance" boundary are shape-parallel. Both Decisions carve a narrow, defensible use for a field or tool that would otherwise naturally expand; both pair with [[Principle - Strategy Support Effectiveness for Self-Hosted Users]]'s anti-goals.
  - [[Decision - 30-Day Retention Default]] — A11's 30-day extension to both intake types implicitly covers the IP: the IP is written into `feedback.json` and retained for exactly as long as the Feedback itself. This Decision explicitly inherits that retention; there is no "IP archive" that outlives the Feedback.
- Referenced by:
  - [[Agent - Feedback Submitter]] — the Feedback Submitter's identity model includes server-logged IP; this Decision is the scope guardrail that keeps the IP field from widening into an account-shaped identity concept.
  - [[Component - Feedback]] — the Feedback entity carries `ip` as an authoritative field; the Component card inherits this Decision for that field's scope semantics.
  - [[System - Feedback Storage & Notification]] — A17 is the mechanism that writes the IP into `feedback.json` at submission time and the server log, and that optionally includes the IP in the Discord post footer for Triagers' abuse-pattern awareness; this Decision is the scope commitment A17's write path honors by not adding analytics emission, geolocation lookup, or cross-submission index maintenance.
  - [[Principle - Reject Like bun.report Drift]] — F3 cites this Decision as the identity-data-scope defense against drift. Using IP for analytics, compliance, or cross-submission tracking would be drift in the same shape F3 rejects team dashboards and symbolication — scope expansions disguised as incremental features.
  - [[Standard - Information Architecture]] — A6 cites this Decision as the scope-of-use boundary that explains why the Feedback Submitter's optional email + server-logged IP do *not* create account pages, login endpoints, or per-user IA surfaces.
- Implemented by:
  - [[System - Feedback Storage & Notification]] — the write path for the IP (into `feedback.json` and the CloudWatch server log) and the 30-day S3 lifecycle rule (which expires the IP alongside the rest of the Feedback) are the operational mechanisms this Decision names. A17 is the only place in the library's mechanism layer that handles IPs; no other System, Standard, or Component writes, reads, or surfaces an IP beyond what A17 specifies.
- Observability coupling:
  - [[System - Service Topology]] — A14's CloudWatch observability stack captures the server-log line that includes the IP. CloudWatch's own retention posture applies to the log; this Decision does not extend a second retention story. If CloudWatch's default retention is shorter or longer than 30 days, the operational-visibility surface of the IP may differ slightly from the S3 surface — acceptable because both are scoped to abuse-protection investigation, not long-term record keeping.

## WHY: Rationale

### Options Considered

- **Option A (rejected): Do not log the client IP on the Feedback flow at all.** The simplest option — no IP, no identity-data capture, library stays identity-free. Rejected because the Feedback flow has a real abuse vector the Debug Bundle flow does not: submission floods that overwhelm the Discord webhook or fill the S3 prefix with low-quality content. A Discord webhook rate-limit outage or a flood of garbage Feedbacks does not just consume service capacity; it degrades the team's ability to see real voice. Without an IP signal, the service has no way to recognize that two floods are from the same source, no way to correlate abuse across a day or a week, and no way to target out-of-band cleanup at the source. IP is the minimum identity signal that serves abuse protection; without it, the flow would eventually need a larger identity-data capture (email as required, session cookies, per-user rate buckets) to address abuse — and each of those is a larger scope expansion than IP is.
- **Option B (rejected): Log the IP but hash it at rest.** Store only `sha256(ip || salt)` in `feedback.json`, not the raw IP. Rejected because abuse investigation requires reading the raw IP at triage time — if two submissions come from the same IP, the team needs to *see* the IP, not just notice that two hashed values match. Hashing would defeat the abuse-investigation use while preserving the scope-creep risks (a hashed IP is still a per-submission identifier that could be cross-joined against other hashed IPs). It would also create a false-privacy shape: the hash looks more privacy-respecting than raw IP while actually preserving most of the same concerns (stable per-IP identifier, joinable across submissions within a salt generation). The correct trade-off for abuse protection is raw IP with a tight scope commitment and a short retention, not hashed IP with a permissive scope.
- **Option C (rejected): Log the IP and retain it beyond 30 days.** Store the IP in a separate long-lived record — 90 days, one year, indefinitely — for "pattern analysis" or "historical abuse investigation." Rejected because a longer retention window makes the IP log feel like an audit database rather than an abuse signal; it also breaks parity with the 30-day standard and violates [[Principle - Strategy Support Effectiveness for Self-Hosted Users]]'s "not compliance posture" anti-goal. An IP record that outlives the Feedback it accompanies is a compliance-shaped record, regardless of whether it is used for compliance purposes. The correct retention is precisely the Feedback's retention; they expire together.
- **Option D (rejected): Log the IP for abuse protection, AND allow aggregate analytics on IPs (e.g., "unique submission-IPs per week" metric).** The "use it for what it's logged for, plus a little harmless aggregation." Rejected because "a little harmless aggregation" is the first step on the drift path this Decision exists to prevent. A weekly-unique-IPs metric is a data product; a data product needs a dashboard; a dashboard creates a quality bar for the metric; a quality bar justifies lengthening retention to "smooth out weekly variance"; longer retention justifies a separate IP-retention policy; a separate policy is the audit-shaped retention Option C already rejected. Each step is small and locally defensible; the cumulative path is exactly the anti-goal [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] names. The scope must be tight at the Decision layer.
- **Option E (chosen): Log the IP into `feedback.json` and the server log, retain per standard 30-day rule, use strictly for abuse-protection investigation and basic submission-source attribution.** No analytics, no cross-submission tracking, no compliance audit trail, no geolocation, no long-term record. Chosen because it gives the team the minimum signal needed to protect the Feedback flow from abuse without widening identity capture beyond what [[Principle - Narrow Forever]]'s "narrow within each intake" commitment tolerates.

### What Drove the Choice

Three structural pressures converged on this Decision.

First, the Feedback flow introduces a **real abuse vector** the Debug Bundle flow did not have. Discord webhook rate limits, S3 object proliferation, and team-attention consumption in the Discord channel are all more tractable abuse targets than Debug Bundle uploads (which are larger, harder to automate, and lead to per-Reporter capability URLs the team can individually delete). The service needs *some* signal to distinguish normal voice from abuse patterns. IP is the smallest signal that works.

Second, [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] has three load-bearing anti-goals: not a formal support SLA, not compliance posture, not marketing telemetry. IP logging is uniquely structurally seductive for each of the latter two. *Compliance* reasoning says "we should keep this longer and enrich it in case we ever need to prove something." *Marketing* reasoning says "we should count distinct values, aggregate over time, understand our reach." Both arguments land with a straight face on an IP log; both are Strategy violations. The scope commitment in this Decision closes both arguments by ruling them out in writing at the Decision layer, where scope changes are adjudicated — not in a System card's implementation section, where scope is easier to widen silently.

Third, [[Principle - Narrow Forever]] explicitly names cross-submission identity tracking and any purpose beyond abuse protection as permanently out of scope for IP logging. This Decision is the positive choice that makes F6's enumeration coherent at the Decision layer: F6 says "IP is for abuse protection only"; A18 says *why*, *what that means operationally*, and *what the rejected alternatives were*.

### What It Costs

- **Abuse investigation visibility is bounded to 30 days.** If an abuse pattern spans longer than 30 days — a campaign that submits one piece of garbage per week over six weeks, for example — the IP record for the earliest submissions has already expired by the time the pattern is recognized. Accepted because the alternative (longer retention) is the audit-shaped posture the Strategy anti-goals rule out. The correct mitigation is to recognize patterns within 30 days, which is adequate for the abuse-vector profile this product is likely to see; long-tail sophisticated abuse is outside the threat model.
- **No "who submitted this" inference across submissions.** When two Feedbacks arrive from the same IP a week apart, the team can only see the match while both are still within retention. Once the older one expires, the link is gone. Accepted because longer-lived identity inference is out of scope by [[Principle - Narrow Forever]].
- **No geolocation context for Feedback.** A Triager reading a Feedback cannot see at a glance "this came from Germany" or "this came from a datacenter IP"; the raw IP is in the Discord footer but the team must look it up externally if geographic context matters. Accepted because geolocation is a scope expansion, and because the team can perform lookups out-of-band if they ever need to.
- **Known-but-unmitigated Discord rate-limit exposure.** As of 2026-04-23, the service does not implement a per-IP soft rate limit on `POST /v1/feedback` to protect Discord from flooding abuse. IP logging makes abuse patterns investigable; it does not prevent a single burst from exhausting Discord's webhook rate limits. If abuse patterns emerge that require active mitigation, a per-IP soft rate limit may be added later as a mechanism change to [[System - Feedback Storage & Notification]] (not a scope change to this Decision). Accepted because active mitigation is premature before observed abuse; the IP log is the investigation substrate that would drive the decision to add a rate limit.
- **IP is a form of identity-adjacent data the library previously had zero of.** Before 2026-04-23 the library was identity-free; adding IP logging is a meaningful shift, even though the scope is tight. Accepted because the abuse vector is real and the scope commitment in this Decision is what makes the shift defensible.

### Scope of Applicability

This Decision governs IP logging on the `fabro.report` Feedback flow — specifically at the `POST /v1/feedback` endpoint and the storage of the IP in `feedback.json` and the CloudWatch server log.

It does **not** govern:

- **Debug Bundle uploads** — the Debug Bundle flow does not capture submitter IP; Reporters have no identity capture at all per [[Standard - Noun Vocabulary]] and [[Agent - Reporter]]. This Decision does not introduce IP logging on the Debug Bundle flow; extending IP capture to Debug Bundles would be a scope change that revisits this Decision and [[Decision - Capability URLs over Auth]] together.
- **Triager retrieval paths** — Triagers retrieve objects from S3 using their own AWS credentials, and AWS IAM logging captures their access as AWS access logs (a separate system, owned by the Fabro team's own AWS infrastructure, not by this service). This Decision does not extend to AWS IAM logs.
- **Fabro's internal tooling** — the parent product at `~/p/fabro-sh/fabro` has its own identity and data-handling decisions that are out of scope here.
- **The Discord channel's own retention** — Discord retains posts per its own policy, which is outside this service's control. A Triager who wants to trace a historical abuse pattern beyond 30 days may find partial signal in Discord's post history; that is Discord's concern.

### Conditions That Would Trigger Revisiting

Because the scope commitment is as load-bearing as the logging decision itself, revisiting this Decision requires an explicit product-owner conversation, not an implementation discussion. Specifically:

1. **Observed abuse patterns that IP-within-30-days cannot address.** If a recurring abuse pattern emerges that spans longer than 30 days or that the team cannot investigate with the current IP record, the scope (retention, use) may need to be reopened. The correct routing is to the product owner as a Strategy-adjacent conversation, not a unilateral retention extension by an implementation agent.
2. **Audience change under [[Decision - Who This Service Is For]].** If the audience widens to populations whose abuse vectors differ materially from self-hosted Fabro users (e.g., an entirely new user base with different network patterns), the IP signal's usefulness may shift and this Decision may need to be revisited as a downstream consequence.
3. **A hosted-Fabro offering is brought into scope.** Hosted-Fabro users would have a different identity context (they already exist in the hosted product); the correct identity signal for them may not be IP, and the scope of abuse protection may shift to hosted-product identity. This would be a significant Strategy change and would reopen this Decision along with [[Decision - Who This Service Is For]] and [[Principle - Narrow Forever]].
4. **A proposal to use the IP for any purpose beyond abuse protection.** Analytics, compliance audit, cross-submission tracking, geolocation surfacing — each is explicitly out of scope by this Decision. A proposal to add any of these must be routed to the product owner as a Strategy change, because each touches a named [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] anti-goal.

None of those conditions exist today. This Decision is stable.

## WHEN: Timeline

Confirmed 2026-04-23 by the product owner via `docs/alexandria/sources/feedback-flow.md`'s "Data Captured" section (L50: "Client IP (server-captured at request time, for abuse protection / basic attribution)") and the "Implications Raven Observes" section (L130 names `Decision - IP Logging for Abuse Protection` as a needed Decision with the rationale "why IPs, what's the retention, scope of use"). Written into this card on the same date as part of the 2026-04-23 feedback scope expansion (Inventory Pass 3, Phase 3). Stable.

This Decision is coupled with [[Decision - Fire-and-Forget Feedback]], [[Decision - 30-Day Retention Default]], and [[Principle - Narrow Forever]]'s F6 out-of-scope list. Revisions that touch IP logging scope feed back through all four.

The library had no IP logging and no identity-adjacent data capture before 2026-04-23. The Feedback flow's introduction made an identity-adjacent signal necessary for the first time; this Decision is the scope guardrail that accompanied it.

## HOW: Application

### Following This Decision

- `POST /v1/feedback` captures the client IP observed by the Fargate task and writes it into the Feedback's `feedback.json` at the `ip` field, and to the CloudWatch server log line for the submission. Both are the only places the IP is written.
- The IP is included in the Discord post's footer, visible to Triagers reading the channel; this is the operational visibility that lets the team notice abuse patterns in the flow of normal voice without leaving Discord.
- Abuse investigation consists of: a Triager notices a pattern in the Discord channel (e.g., multiple low-quality Feedbacks in quick succession); the Triager cross-references IPs across recent Feedbacks using their own S3 credentials and AWS tooling (`aws s3 cp` followed by `jq '.ip'` or equivalent) or by reading the CloudWatch logs; if the pattern confirms abuse, the Triager takes out-of-band action (S3-side cleanup, infrastructure-layer block, or escalation to the product owner).
- The 30-day S3 lifecycle rule deletes every object under the `feedback/` prefix at the retention deadline, including `feedback.json`. The IP is gone once the Feedback expires; no separate IP archive persists.

### Violating This Decision

- **Wrong:** Adding an analytics emission that counts unique IPs per week (or any aggregation). This is the marketing-telemetry anti-goal reopening through a side door.
- **Wrong:** Adding a separate long-lived IP record (a database, a CSV export, a second S3 object outside the `feedback/` prefix) that outlives the Feedback itself. This is the compliance-audit-trail anti-goal reopening through a side door.
- **Wrong:** Joining Feedbacks by IP to create a "likely same submitter" inference that persists beyond each individual Feedback's lifetime. This is the cross-submission-identity-tracking F6 anti-goal.
- **Wrong:** Calling a geolocation service with the IP and surfacing country/region/city context anywhere — in the Discord post, in a service-side analytics surface, in a Triager-facing tooling surface. Geolocation is a scope expansion that this Decision rules out.
- **Wrong:** Extending IP capture to the Debug Bundle flow. Debug Bundles have no submitter identity by design; adding IP logging to Debug Bundles would reopen [[Decision - Capability URLs over Auth]] and [[Standard - Noun Vocabulary]]'s Reporter definition, not just this Decision. The correct answer to "should we log IPs on Debug Bundle uploads for abuse protection?" is: the Debug Bundle flow does not have the same abuse vector (capability URLs give per-upload control; presigned S3 URLs limit compute-side abuse), and extending IP logging requires a separate conversation, not an inference from this Decision.
- **Wrong:** Hashing the IP at rest to "anonymize" it while preserving per-IP joinability. False-privacy shape; preserves joinability while defeating investigation. The correct posture is raw IP with a tight scope commitment and a short retention, not hashed IP with a permissive scope.
- **Wrong:** Storing the IP outside `feedback.json` and the CloudWatch server log — e.g., a DynamoDB table of "recent submission IPs," a Redis cache for rate-limit bucketing that persists IPs beyond immediate use, a Parquet export to an analytics data lake. This Decision's scope commitment is about what uses of IP are allowed; storing IP outside the scoped sites widens the use surface by widening the storage surface.

### Test

Before adding any feature that reads, writes, aggregates, exports, or transforms the client IP, ask three questions in order.

First: "Is this feature necessary for abuse-protection investigation by a Triager, within the 30-day Feedback retention window?" If yes, the feature is within this Decision's scope. If no, the feature is out of scope and must be rejected or escalated to the product owner as a Strategy change.

Second: "Does this feature treat the IP as a durable identifier beyond the individual Feedback's lifetime?" If yes — if the feature stores, aggregates, or cross-references the IP beyond the Feedback's 30-day window, or if it joins IPs across submissions to infer identity — the feature violates this Decision's scope commitment.

Third: "Does this feature surface, export, or transform the IP in a way that serves a purpose on [[Principle - Strategy Support Effectiveness for Self-Hosted Users]]'s anti-goal list — marketing telemetry, adoption metrics, compliance audit?" If yes, the feature is a Strategy violation.

A second-order test: "Would removing the IP field from `feedback.json` tomorrow break this feature?" If yes and the feature is not directly about abuse-protection investigation, the feature has widened the IP's use beyond what this Decision authorizes.

## Sources

- `docs/alexandria/sources/feedback-flow.md` (2026-04-23 — L50 client IP "server-captured at request time, for abuse protection / basic attribution"; L130 "Decision - IP Logging for Abuse Protection" as needed Decision with scope-of-use rationale)
- `docs/alexandria/reviews/inventory-pass-3-feedback-scope.md` (Part 2 — A18 verdict BUILD (thin), parallel in shape to A10; Part 5 — Phase 3 scope)
- `docs/alexandria/inventory/card-manifest.md` (A18 entry — build order 28, added 2026-04-23 in Inventory Pass 3)

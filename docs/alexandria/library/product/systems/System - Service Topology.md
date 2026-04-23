# System - Service Topology

## WHAT: Definition

The **operational-topology system** that names the running pieces of `fabro.report` — the Fargate task, the Route53 DNS record, the IAM role attached to the task, Secrets Manager (or SSM Parameter Store) for env-var secret injection, the S3 bucket that backs the product's storage (owned by [[System - Retention, Storage, and Key Scheme]]), and the Terraform repository that declares all of them — and how they fit together at runtime: request flow, secret-injection flow, IAM flow. This System is paired with [[System - Retention, Storage, and Key Scheme]]; A5 owns the **data** topology (bucket layout, keys, URLs, lifecycle), A14 owns the **operational** topology (compute, DNS, IAM, secrets, IaC). The two change at different rates for different reasons, which is why they are separate cards. A14 deliberately does **not** describe a database (there is none), a load balancer with nuanced configuration (a single Fargate-service ALB or equivalent is the entire load-balancer story), or autoscaling complexity (single-task / small-task-size defaults are fine for the product's traffic profile).

## WHERE: Ecosystem

- Stack commitment:
  - [[Decision - Service Stack]] — A12 is the Decision this System implements. A12 chooses Rust on AWS Fargate, Docker-packaged, SSR HTML, Route53-fronted, Terraform-provisioned; this System card is the operational expression of that choice at the piece-by-piece topology layer. Every piece named here is justified upstream by A12.
- Strategy alignment:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — the operational topology stays minimal because minimal topology serves reliable team retrieval. Every piece named here is justified against the retrieval path F5 optimizes for; topology expansion that does not serve team retrieval is out of scope under the Strategy.
- Paired System:
  - [[System - Retention, Storage, and Key Scheme]] — A5 owns the data topology (S3 bucket name, key scheme, URL format, lifecycle rule, metadata sidecar); this card owns the operational topology (how the Fargate task reaches that bucket, what IAM principal holds the write grant, how secrets land in the task). A5's bucket is *the* bucket in A14's IAM flow; A14's Fargate task role is *the* write principal A5's access model depends on. The two cards cross-reference once each; neither restates the other.
- Observability posture:
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — Sentry lives inside this topology. The Fargate task initializes Sentry with a DSN injected via Secrets Manager / SSM Parameter Store (the same secret-injection flow this System names); Sentry events flow outbound to Sentry's SaaS. This card names the topology-level mechanics (where the DSN comes from, what emits events); A13 names the observability policy (what is and is not sent).
- Access-model coupling:
  - [[Decision - Capability URLs over Auth]] — the service's public surface is the Route53-mapped Fargate endpoint; the capability URL shape (`fabro.report/r/<id>`) resolves through this topology's DNS and Fargate. The IAM role that grants S3 write access to the Fargate task is the service-side half of the access model; Triagers retrieve bundles from the same S3 bucket using their own AWS credentials (per [[Agent - Triager]]), which is a separate IAM path orthogonal to this System.
- Scope commitments:
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point's topology should be as minimal as the product is narrow. One Fargate service, one Route53 record, one IAM role, one Secrets Manager / SSM entry set, one S3 bucket, one Terraform repository. Anything that proposes a second compute service, a second DNS zone, a second IAM-role-owner, or a parallel IaC tool is topology drift and is out of scope under this thesis.
  - [[Principle - Narrow Forever]] — topology complexity is a scope vector. Additional microservices, message queues, databases, caches, sidecar containers, and multi-region deployments are scope expansions, not topology refactors. This System stays this minimal; growing it is a scope-change conversation, not an implementation detail.
  - [[Principle - Reject Like bun.report Drift]] — `bun.report`-shaped topology (separate symbolication service, team-database, auth provider, analytics pipeline) is named drift. This topology has none of those components, and will not acquire them under this Principle.
- Roles that traverse this System:
  - [[Agent - Reporter]] — Reporters reach this topology via DNS (`fabro.report` resolves to the Fargate service's endpoint), talk to the Rust service for upload coordination and Report Metadata Page rendering, and upload bundles via presigned S3 URLs (bundle bytes never flow through Fargate — per [[Standard - Interaction Patterns (CLI + Delete)]]). The Reporter's network path touches the service and S3 directly; only the coordination-endpoint path traverses the Fargate compute this System hosts.
  - [[Agent - Triager]] — Triagers never reach this topology's compute. Their retrieval path goes directly to S3 using their own AWS credentials (a separate IAM principal from the Fargate task role this System names). This System owns the service-side access; Triager access is orthogonal.
- Surfaces depending on this topology:
  - [[Component - Report Metadata Page]] — the page is rendered by the Rust service running on the Fargate task this System names. SSR HTML delivery is the compute path this topology supports end-to-end.
  - [[Standard - Interaction Patterns (CLI + Delete)]] — the presigned-URL upload architecture depends on this topology's Rust service (minting IDs and presigned URLs) and the S3 bucket (accepting direct PUTs from the CLI). The architecture's load-bearing property — bundle bytes never traverse Fargate compute — is an operational property of this topology.
  - [[Standard - Accessibility Policy]] — A8's web-accessibility floor is delivered by the SSR-HTML pipeline this topology runs. Topology changes that introduce a client-rendered front-end would move the accessibility surface; this System stays SSR.
- Source provenance:
  - `docs/alexandria/sources/tech-stack-and-deployment.md` — the raw material Bryan confirmed on 2026-04-23. Authoritative for every piece named in this topology (Fargate task, Route53 domain registration, Secrets Manager / SSM Parameter Store secret injection, IAM role attached to the Fargate task, Terraform as the IaC tool, `us-east-1` default region).

## WHY: Rationale

This System card exists because the product's operational topology is a distinct concern from its data topology, and collapsing them into one card would hide the asymmetry. The operational pieces — Fargate, Route53, IAM roles, Secrets Manager / SSM Parameter Store, the Terraform repository — change at different rates and for different reasons than the data pieces. A stack-version bump, a Rust-dependency update, a task-size adjustment, a DNS TTL change, a region migration — these are operational changes whose blast radius lives on this card. A bucket-name change, a key-scheme adjustment, a lifecycle-duration change, a URL-entropy update — these are data changes whose blast radius lives on [[System - Retention, Storage, and Key Scheme]]. Splitting the cards lets each piece be reviewed for its own load without either half accidentally constraining the other.

The load-bearing claim of this System is that **the topology is minimal by design, and minimality is the operational expression of [[Principle - Narrow Forever]]**. One Fargate task, one DNS record, one IAM role, one Secrets source, one S3 bucket, one Terraform repository. Each piece exists because a specific part of the product requires it; none exists for "flexibility" or "future-proofing." Topology-expansion proposals — a separate microservice for a specific function, a message queue to decouple a path, a cache tier to speed a response — are all scope vectors, and each one would need to clear [[Principle - Narrow Forever]] as a scope-change conversation rather than landing as an implementation detail here.

A secondary load-bearing property is that **infrastructure is Terraform-declared end-to-end**. Every piece named in this card — Fargate service, Route53 record, IAM role, Secrets Manager / SSM Parameter Store entries, bucket-access policies — is a Terraform resource. No ClickOps, no CDK, no Pulumi. Declarative-stamp-out reproducibility is what makes staging, dev, and alternate deployments coherent; it is also what makes infrastructure review-able (the Terraform diff is the change). That property is named in [[System - Retention, Storage, and Key Scheme]] Test and applies uniformly to this card's pieces as well.

## WHEN: Timeline

Drafted 2026-04-23 from source material captured the same day (`docs/alexandria/sources/tech-stack-and-deployment.md`). Pairs with [[Decision - Service Stack]] (A12) and [[Decision - Sentry for Service Errors, Not Bundle Processing]] (A13) in the same batch. Stable. Revisions to the operational pieces — the secret-injection source (Secrets Manager vs SSM), the region default, the task-size default, the DNS TTL — are source-level changes that feed back into this card. Revisions to the *topology shape* — introducing a second service, adding a message queue, switching to a different compute backend — are scope-level changes that revisit [[Decision - Service Stack]] and [[Principle - Narrow Forever]] before they become operational updates here.

## HOW: Specification

### The Pieces

Each piece named below exists because a specific part of the product requires it. Nothing here is optional; nothing here is redundant.

- **Fargate task (runs the Rust service).** A single-task Fargate service by default, small-task-size. Hosts the Rust application that handles upload-coordination requests, Report Metadata Page rendering, and delete requests. Does **not** handle bundle-byte uploads directly — those go to S3 via presigned URLs (per [[Standard - Interaction Patterns (CLI + Delete)]]).
- **Route53 DNS record.** `fabro.report` is registered on AWS Route53 and points at the Fargate service's load-balancer endpoint. The domain registration lives in the same AWS account as the rest of the infrastructure; no external DNS provider is involved.
- **IAM role attached to the Fargate task.** Grants the service S3 access (write to the bundle bucket for metadata; presign-URL generation for bundle PUTs) and any other AWS resources the service needs. The role's permissions are least-privilege: only the S3 bucket owned by [[System - Retention, Storage, and Key Scheme]], only the actions required to mint presigned URLs and persist metadata, only the Secrets Manager / SSM Parameter Store entries the service reads at startup. No baked-in credentials; the task assumes its role from Fargate at runtime.
- **Secrets Manager (or SSM Parameter Store) entries.** Secrets (Sentry DSN, any other service secrets) live in AWS Secrets Manager or SSM Parameter Store and are injected into the Fargate task as environment variables at task-start time. Standard Fargate pattern. The choice between Secrets Manager and SSM Parameter Store is an implementation detail not locked on this card; both support the required pattern.
- **S3 bucket (data plane).** Owned by [[System - Retention, Storage, and Key Scheme]]; referenced here because the topology's IAM flow culminates in bucket access. Not restated.
- **Terraform repository.** Declares every piece above: Fargate service definition, Route53 record, IAM role and policy, Secrets Manager / SSM Parameter Store entries, S3 bucket and lifecycle rule (the last two owned by A5). One repository; one source of truth; one `terraform plan` that shows every infrastructure change.

### How They Fit Together

- **Request flow (CLI upload coordination or Report Metadata Page render):**
  1. CLI (for upload) or browser (for page render) resolves `fabro.report` via DNS.
  2. DNS returns the Fargate service's load-balancer endpoint (from the Route53 record this System declares).
  3. The request reaches the Fargate task running the Rust service.
  4. For upload coordination: the service mints a Report ID and presigned S3 PUT URL; the CLI then uploads bundle bytes directly to S3 (bytes do not traverse Fargate).
  5. For metadata-page render: the service reads the metadata JSON from S3 (per [[System - Retention, Storage, and Key Scheme]]) and renders the page as SSR HTML.
- **Secret-injection flow:**
  1. Terraform declares Secrets Manager (or SSM Parameter Store) entries for secrets the service needs (Sentry DSN, etc).
  2. The Fargate task definition references those entries; at task-start time, Fargate injects them into the task as environment variables.
  3. The Rust service reads the env vars on startup and configures its dependencies (Sentry client, etc).
  4. No secrets live in the Docker image. No secrets live in source control. No secrets live in the Terraform state in plaintext (Terraform references the secret ARNs, not their values).
- **IAM flow:**
  1. The Fargate task assumes the IAM role this System declares.
  2. Role permissions allow: presigned-URL generation for the bundle bucket; metadata object writes to the bundle bucket; reads of the configured Secrets Manager / SSM Parameter Store entries.
  3. Role permissions do **not** allow: read access to bundle contents (the service never reads uploaded bundles); writes outside the bundle bucket; any permission the service does not actually exercise.
  4. Triagers' S3 retrieval is a separate IAM principal (the Fabro team member's own AWS credentials), orthogonal to this role.

### Properties This Topology Guarantees

- **No database.** The service's state lives entirely in S3 (metadata sidecars per Report, per [[System - Retention, Storage, and Key Scheme]]). No RDS, no DynamoDB, no managed key-value store. A database would be a topology expansion requiring a product-owner Decision.
- **No load-balancer subtlety.** A single Fargate-service ALB with a single target group is the entire load-balancer story. No multi-target-group path routing, no WAF rules, no custom healthcheck wiring beyond the Fargate service default.
- **No autoscaling complexity.** Default single-task / small-task-size config is fine for the product's traffic profile (low-volume internal use). Autoscaling is an implementation detail to revisit if volume patterns demand, not a baseline topology feature.
- **No message queue, no cache, no sidecar container.** The upload flow is synchronous through the presigned-URL path; the metadata-page render is a two-read path (metadata JSON + template render). No intermediary components.
- **Single-region, single-cloud.** Default region is `us-east-1` (overridable via environment variable per source). Multi-region deployment would require explicit Terraform configuration and is not a baseline topology property.
- **Observability stack.** CloudWatch receives Fargate's stdout/stderr by default; Sentry receives the service's own server-side errors (per [[Decision - Sentry for Service Errors, Not Bundle Processing]]). Metrics and tracing are not in the default topology; they are added if and when a measurable need appears.

### Anti-Examples (What This Topology Is Not)

- **Wrong:** A separate microservice for upload coordination, bundle processing, or metadata management. The Rust service is monolithic by design; decomposition into microservices is a topology-complexity choice that conflicts with [[Principle - Narrow Forever]]'s narrowness commitment at the infrastructure layer.
- **Wrong:** A message queue (SQS, Kafka) between the service and S3. Uploads are synchronous through presigned URLs; there is no work-queue shape to introduce.
- **Wrong:** A relational database (RDS) or a managed key-value store (DynamoDB) for metadata. Metadata lives as an S3 sidecar JSON per [[System - Retention, Storage, and Key Scheme]]; a database would be a parallel storage surface with its own retention, backup, and access-control engineering.
- **Wrong:** A cache tier (ElastiCache, CloudFront in front of the dynamic endpoints) for bundle downloads or metadata-page renders. Bundle downloads are out-of-service-path by design; metadata-page renders are cheap enough that caching is not a baseline requirement.
- **Wrong:** Baked-in AWS access keys in the Docker image or Fargate task definition. The task's IAM role is the only access mechanism; any keyed-credentials path is a security regression that [[Decision - Service Stack]] explicitly rejects.
- **Wrong:** DNS hosted outside Route53 (Cloudflare DNS, a registrar's default NS). The domain is registered on Route53 and served from Route53; splitting registration and DNS is an unnecessary piece of topology to maintain.
- **Wrong:** Infrastructure managed through the AWS console (ClickOps) or through a second IaC tool (CDK alongside Terraform). Terraform is the single source of truth; any out-of-Terraform changes are drift.
- **Wrong:** Multi-region active-active deployment by default. Single-region is the baseline; multi-region is a Decision-level scope change.

### Test

Before changing the topology — adding a piece, removing a piece, changing the secret-injection source, changing the region, changing the IAM role's scope — ask three questions in order. First: "Does the change preserve [[Decision - Service Stack]]'s AWS-native IAM path from the service to S3?" If the proposed topology reintroduces cross-cloud credentials, long-lived access keys, or any parallel access mechanism, the change is undoing A12's load-bearing property. Second: "Does the change preserve [[Principle - Narrow Forever]]'s minimal-topology commitment at the infrastructure layer?" If the change adds a component that did not previously exist (microservice, queue, database, cache, sidecar), it is a scope expansion — not an operational refactor — and goes to the product owner through [[Decision - Service Stack]] and [[Principle - Narrow Forever]]. Third: "Does the change preserve the property that Terraform declaratively stamps out every piece of the infrastructure?" If the change introduces runtime-only state, ClickOps configuration, or an out-of-Terraform resource, the declarative-stamp-out property has drifted and the change is a regression on [[Decision - Service Stack]]'s IaC commitment.

## Sources

- `docs/alexandria/sources/tech-stack-and-deployment.md` (full file — authoritative for Fargate as the compute backend, Route53 for DNS, IAM role-based S3 access, Secrets Manager / SSM Parameter Store for secret injection, Terraform as IaC, single-region `us-east-1` default; confirmed 2026-04-23)
- `docs/alexandria/inventory/card-manifest.md` (A14 entry)

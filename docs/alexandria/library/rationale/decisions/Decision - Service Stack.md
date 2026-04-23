# Decision - Service Stack

## WHAT: The Choice

The `fabro.report` service is a **Rust application** packaged as a **Docker image**, deployed on **AWS Fargate**, fronted by the **`fabro.report` domain registered on Route53**, and provisioned end-to-end by **Terraform**. The rendered output is **server-side-rendered HTML** (no SPA, no client-side framework); client-side JavaScript appears only where it carries real UX load (the delete-confirmation affordance). This Decision records that stack-and-deployment choice as a single coherent commitment: Rust + Docker + Fargate + Route53 + Terraform is *the* stack, chosen together, for reasons that depend on all five pieces fitting.

The load-bearing phrasing is *AWS-native by construction, not by coincidence*. The reason to pick this stack over Bun-on-Fly (the `bun.report` template) or Rust-on-Fly (the Fabro parent-product deployment) is that the service is S3-adjacent and the S3 access path is the load-bearing piece of the operational architecture ([[Decision - Capability URLs over Auth]], [[System - Retention, Storage, and Key Scheme]]). IAM-role-based S3 access from a Fargate task is dramatically simpler than any cross-cloud alternative; that simplification is what the stack is buying.

## WHERE: Ecosystem

- Governed by:
  - [[Principle - Strategy Support Effectiveness for Self-Hosted Users]] — Strategy optimizes for the team retrieving Debug Bundles from self-hosted Reporters. The stack's payoff is an operational architecture (IAM role → S3) that makes team retrieval reliable; Fargate adjacency to S3 is not a technology preference but a direct expression of that Strategy at the infrastructure layer.
  - [[Principle - Narrow Forever]] — the stack is chosen as a permanent shape, not a v1 simplification. "We'll port it later to X" is not on the table; the narrow-forever commitment applies to infrastructure choices the same way it applies to feature scope. Stack expansion (additional services, additional compute backends, multi-region) is a scope vector to resist for the same reason a team dashboard is.
  - [[Principle - Reject Like bun.report Drift]] — borrowing `bun.report`'s Bun/TypeScript-on-Fly template is a drift vector. This Decision forecloses it explicitly: the stack choice borrows nothing structural from `bun.report` beyond the intake-URL shape (per [[Product Thesis - Pickup Point, Not Triage Surface]]). The template is not a reference for the service's implementation stack.
- Paired with:
  - [[Decision - Sentry for Service Errors, Not Bundle Processing]] — the observability-layer Decision that lives at the same infrastructure layer as this one. A13 names how operational errors are caught; this card names what the service is and where it runs. The two Decisions answer complementary questions about the service's operational posture and are intended to be read together.
  - [[Product Thesis - Pickup Point, Not Triage Surface]] — a pickup point's infrastructure should be as narrow as its feature surface. One compute backend, one bucket, one domain, one Terraform repository is the stack-layer expression of the pickup-point thesis.
- Related:
  - [[Decision - Capability URLs over Auth]] — capability URLs depend on S3 being reachable with the correct IAM principal for Triager retrieval and a service-side presigned-URL mint for Reporter upload. The IAM-role-based access path this Decision chooses is what makes that access model's Triager side work without the service brokering bytes.
  - [[Decision - 30-Day Retention Default]] — retention is enforced by an S3 lifecycle rule declared in Terraform ([[System - Retention, Storage, and Key Scheme]]). Terraform-as-the-IaC-tool and S3-as-the-storage-backend are stack-level choices this Decision locks in; A11's policy depends on both.
- Implemented by:
  - [[System - Service Topology]] — the operational-topology System card that names the concrete pieces (Fargate task, Route53 record, IAM role, Secrets Manager / Parameter Store, S3 bucket, Terraform repository) and how they fit together. A14 is the "how this Decision shows up as infrastructure" card; this Decision is the "why these pieces, not others" card.
  - [[System - Retention, Storage, and Key Scheme]] — the data-topology System card that depends on this Decision's AWS-native posture. A5's IAM model is the Fargate task role granting write access to the bucket; that model is only load-bearing if the compute lives in AWS, which this Decision commits to.
- Referenced by:
  - [[Standard - Interaction Patterns (CLI + Delete)]] — A7's presigned-S3-URL upload architecture is coherent only because the compute and the storage live in the same cloud under a shared IAM boundary. A Fly-deployed service minting presigned URLs for AWS S3 would work, but the IAM posture would be worse; this Decision is the operational-architecture commitment A7's upload flow rests on.
  - [[Standard - Accessibility Policy]] — A8's web accessibility floor is for the server-rendered HTML that this stack produces. The "SSR, no SPA" piece of this Decision is what makes the accessibility floor structurally cheap: there is no client-side rendering pipeline to audit beyond the delete-confirmation JavaScript.
  - [[Component - Report Metadata Page]] — the page is delivered as server-side-rendered HTML. The rendering posture lives on this card; the page content lives on A4.

## WHY: Rationale

### Options Considered

- **Option A (rejected): Bun / TypeScript on Fly.io — matches the `bun.report` template.** Rejected on two independent grounds. First, `bun.report`'s vocabulary and product shape are structurally different from `fabro.report`'s; borrowing the implementation stack from a product whose feature scope has already been rejected as drift ([[Principle - Reject Like bun.report Drift]]) imports the "we're like bun.report" framing at the implementation layer, which the product has already refused at the feature layer. Second, Fly.io does not simplify the service's dominant operational path (S3 access), which the AWS-native alternative does — so the borrowed-stack approach costs the narrowness argument without recovering any infrastructure benefit.
- **Option B (rejected): Rust on Fly.io — matches Fabro's own deployment shape.** Rust-on-Fly would share a deployment template with the parent product and would reuse the team's existing Fly operational experience. Rejected because `fabro.report`'s dominant operational path is not "run a Rust service behind a load balancer" (Fabro's shape) but "run a Rust service that reads and writes S3 on behalf of Reporters and Triagers." IAM role-based access from a Fargate task to an S3 bucket in the same account is the industry-default least-privilege posture; from a Fly.io task it requires either long-lived AWS access keys stored in Fly secrets (worse security posture, rotation overhead) or cross-cloud federation (more moving parts, more failure modes, more bespoke infrastructure). The security-posture gap is the decisive factor; matching Fabro's deployment template is not worth re-introducing long-lived credentials.
- **Option C (rejected): Cloudflare Worker + R2.** Worker + R2 fits the product shape well — narrow, low-traffic, edge-delivered, cheap. Rejected on two grounds. First, it diverges from the team's existing AWS tooling (IAM, CloudWatch, Secrets Manager, S3 console) and would add a second set of operational surfaces the team would need to run in parallel. Second, Terraform support for Cloudflare Workers is less mature than for Fargate; `terraform-provider-aws` is canonical, while Workers-via-Terraform involves a smaller community provider with a narrower feature set. Mature IaC support is part of what the stack is buying ("Terraform declaratively stamps out the infrastructure" — [[System - Retention, Storage, and Key Scheme]] Test), and weakening that property for a cost saving the product does not need is a poor trade.
- **Option D (chosen): Rust on AWS Fargate, Docker-packaged, server-side-rendered HTML, Route53-fronted, Terraform-provisioned.** Chosen because the combination makes the service's dominant operational path — read and write S3 with least-privilege credentials — structurally simple. The Rust choice aligns with the parent Fabro codebase (dogfooding value, same-team language competence). Fargate aligns the compute with the storage under a single IAM boundary. Docker is implicit in Fargate. Terraform is the team's existing IaC choice for AWS and has mature support for every resource in the service's blast radius. SSR HTML fits the product's one-page-per-Report shape without an SPA's build-pipeline complexity. Each piece of the stack is chosen *together with* the others; the coherence across pieces is the load-bearing property.

### What Drove the Choice

Two forces drove the choice. First, the service's dominant operational flow is S3 access, and IAM-role-based access from AWS compute to AWS storage is the narrowest, most-supported, least-credential-exposing path. Every alternative reintroduces credential management the AWS-native path removes. Second, the stack choice is a permanent commitment under [[Principle - Narrow Forever]]; a stack chosen for "v1 speed" becomes a migration story the narrow-forever commitment would have to absorb later, which would itself be a scope expansion on operational work. Picking a stack that matches the product's permanent shape eliminates that drift.

The Rust choice specifically is consistent with the parent Fabro codebase's primary language. This is not a strategic requirement (Fabro itself mixes Rust and TypeScript in `~/p/fabro-sh/fabro`); it is a secondary benefit — the same team maintains both services in the same language, reducing context-switch cost. Rust-on-Fargate combines that language benefit with the AWS-native storage posture; neither alternative carries both benefits.

### What It Costs

- **Fargate cold-start and baseline cost.** Fargate tasks are not free; a single small task running continuously costs more than a Cloudflare Worker that bills per request. The cost is accepted because the service's traffic profile is low-volume but steady, and Fargate's predictable runtime simplifies operational reasoning (no cold starts on occasional requests).
- **AWS-account coupling.** The service is coupled to one AWS account's IAM surface, one AWS region by default (`us-east-1`), and one team's AWS console access. Migrating off AWS would be non-trivial. Accepted because [[Principle - Narrow Forever]] commits the stack to permanence; optionality for cloud migration is not a property the product is buying.
- **Terraform skill requirement.** Operating this service requires a team member who can read and write Terraform. Accepted because the team already operates Fabro's infrastructure through similar tooling; Terraform is not a new skill introduced for this service.
- **Single-region default.** The default deployment is single-region (`us-east-1`, overridable via environment variable). Multi-region deployment would require a bucket-replication strategy and additional Terraform configuration. Accepted because the service's audience (self-hosted Fabro users) does not require multi-region latency, and bundle durability is bounded by the 30-day retention anyway ([[Decision - 30-Day Retention Default]]).

### Scope of Applicability

This Decision governs the service's implementation and deployment stack. It does **not** govern the CI/CD pipeline choice (GitHub Actions is assumed, per Fabro's existing pattern, but not locked on this card). It does **not** govern the Rust web framework (Axum is assumed, per Fabro's `fabro-server`, but the specific crate is implementation detail). It does **not** govern the testing-against-real-S3 posture (tests run against a mocked S3; real-S3 testing if added goes behind a gate — implementation detail). It also does **not** govern retention mechanism, URL entropy, or key scheme — those are separate policy choices that live in [[Decision - 30-Day Retention Default]], [[Decision - Capability URLs over Auth]], and [[System - Retention, Storage, and Key Scheme]].

### Conditions That Would Trigger Revisiting

This Decision is stable but not eternal. Specific conditions that would put it back on the product owner's desk:

1. **AWS no longer being the right cloud for Fabro-team operations.** If Fabro itself migrates off AWS, the S3-adjacency argument loses force, and the stack choice becomes a lone AWS deployment the team no longer otherwise operates. This is a team-shaped reason to revisit, not a `fabro.report`-internal one.
2. **A measured, repeatable operational failure mode inherent to Fargate** (rather than to the team's use of it). Fargate-the-platform failing in a way that cannot be mitigated would be invalidation. No such pattern exists today.
3. **An identity-shaped scope change to the product** — for example, opening [[Decision - Who This Service Is For]] to hosted-Fabro users at a scale where single-region Fargate no longer fits. This is upstream-shaped; if the audience changes, the stack may need to change to match.

None of those conditions exist today. This Decision is stable.

## WHEN: Timeline

Confirmed 2026-04-23 by the product owner in the Confirmed Product Decisions captured in `docs/alexandria/sources/tech-stack-and-deployment.md`. Written into this card as part of the 2026-04-23 Amplifier-tier follow-on batch. Pairs with [[Decision - Sentry for Service Errors, Not Bundle Processing]] (same source, same date) and [[System - Service Topology]] (same source, drafted in the same batch). Stable by its own terms; revisions require a product-owner decision recorded in source material, not an implementation-level refactor.

## HOW: Application

### Following This Decision

- New service code is written in Rust. No TypeScript service code. Client-side JavaScript is limited to surfaces where it carries real UX load (the delete-confirmation affordance on the [[Component - Report Metadata Page]], per [[Standard - Interaction Patterns (CLI + Delete)]]).
- Rendering is server-side. The Report Metadata Page is generated in the Rust service and delivered as HTML. No SPA build step, no client-side routing, no React / Vue / Svelte / Solid.
- The service packages to a single Docker image and deploys as a Fargate task. Secrets flow from AWS Secrets Manager or SSM Parameter Store into the task as environment variables. S3 access uses the task's IAM role, not baked-in AWS credentials.
- All infrastructure (S3 bucket, Fargate service, Route53 records, IAM roles, lifecycle rules, Secrets Manager / Parameter Store entries) is declared in Terraform. No ClickOps, no CDK, no Pulumi. Stacks are reproducible from source.
- `fabro.report` DNS is served from Route53; the domain registration lives in the same AWS account as the infrastructure.

### Violating This Decision

- **Wrong:** Adding a TypeScript / Node.js service component alongside the Rust service. The stack is Rust end-to-end; introducing a second runtime language on the server side is a stack expansion, not an implementation detail.
- **Wrong:** Building a SPA front-end. The Report Metadata Page is server-side-rendered HTML. A React / Vue / Svelte front-end is a different product shape.
- **Wrong:** Deploying to Fly.io, Cloudflare Workers, or any non-AWS backend. Cross-cloud deployment reintroduces the credential-management surface this Decision explicitly trades away.
- **Wrong:** Managing any piece of the infrastructure through the AWS console (ClickOps). Terraform is the authoritative source of truth; console-only changes are drift.
- **Wrong:** Baking long-lived AWS credentials into container images, environment files, or source control. The task's IAM role is the only S3-access mechanism; any other credential path is a security regression.
- **Wrong:** Splitting compute across multiple regions or multiple cloud providers without a product-owner Decision reopening this one. Single-region, single-cloud is the default by design.

### Test

Before adding any implementation-language, runtime, deployment-target, or IaC-tool decision to `fabro.report`, ask three questions in order. First: "Does this change preserve the AWS-native IAM path from the service to S3?" If it introduces cross-cloud credential federation, long-lived access keys, or any additional credential surface, the change is undoing the Decision's load-bearing property. Second: "Does this change preserve Terraform as the single source of truth for infrastructure?" If it moves any resource out of Terraform (ClickOps fallback, Pulumi adjunct, Helm-chart side-channel), the declarative-stamp-out property is weakening. Third: "Does this change respect [[Principle - Narrow Forever]]'s permanent-shape commitment at the infrastructure layer?" If it's a v1-shaped optimization that implies "we can port later," the narrow-forever commitment is being renegotiated at the stack layer, which this Decision refuses.

## Sources

- `docs/alexandria/sources/tech-stack-and-deployment.md` (full file — authoritative for Rust choice, Docker packaging, Fargate deployment, Route53 domain, Secrets Manager / SSM Parameter Store secret-injection, IAM-role S3 access, Terraform as IaC, SSR HTML posture, single-region `us-east-1` default; confirmed 2026-04-23)
- `docs/alexandria/sources/product-vision.md` (L87 capability-URL access model; L102–L107 bun.report template-relationship framing that this Decision answers at the stack layer)
- `docs/alexandria/inventory/card-manifest.md` (A12 entry)

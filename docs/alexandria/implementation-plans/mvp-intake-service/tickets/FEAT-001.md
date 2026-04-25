---
id: FEAT-001
title: "Bootstrap Rust + Axum service with /health endpoint and Dockerfile"
outcome: O-3
tier: must
enabler: false
blocked-by: []
blocks: [FEAT-002]
cards: [Decision - Service Stack, System - Service Topology]
---

## Motivation

The plan needs a runnable Rust service binary before there is anything for Terraform to deploy or for CI/CD to package. This ticket creates the skeleton so the next tickets have something concrete to wire up. Kept deliberately tiny so the foundation phase reaches a deployed-but-empty service in three tickets.

## Description

Initialize a Rust workspace at the repo root with Axum as the web framework. Implement exactly one route — `GET /health` returning `200 OK` with a minimal JSON body (e.g., `{"status":"ok"}`). Write a multi-stage Dockerfile that produces a small runtime image (`distroless` or `debian-slim` base). The service must read its listen port from `PORT` env var (default 8080) so Fargate can wire it up.

## Context

- [[Decision - Service Stack]] pins Rust + Docker + Fargate + Route53 + Terraform. Axum is the assumed web framework per Fabro's `fabro-server`; this ticket commits to it explicitly.
- [[System - Service Topology]] specifies a single Fargate task hosting the Rust service; `/health` is required for the ALB target group's default healthcheck.
- The service must use the IAM role assumed at runtime — no AWS keys or other secrets are needed at this stage (the `/health` endpoint touches no AWS resources).
- Anti-pattern: do not introduce SPA build tooling, a frontend framework, or any client-side dependency. SSR HTML only (none required by this ticket).

## Acceptance Criteria

- [ ] `cargo build --release` produces a binary that starts and serves `GET /health` on the port given by `PORT`.
- [ ] `docker build .` produces a runnable image; `docker run -p 8080:8080 <image>` returns 200 OK on `curl localhost:8080/health`.
- [ ] The Dockerfile uses a multi-stage build; the runtime image does not include the Rust toolchain.
- [ ] Cargo workspace structure follows Fabro's `fabro-server` conventions where applicable (`Cargo.toml` at repo root, `src/main.rs`, `src/lib.rs` as needed).
- [ ] No additional routes beyond `/health` exist yet; later tickets add them.

## Implementation Notes

- Crates: `axum`, `tokio` with `rt-multi-thread`, `serde`, `serde_json`, `tracing` + `tracing-subscriber` (init JSON formatter early so subsequent tickets get logs for free).
- The `/health` body is intentionally minimal — it's an ALB liveness probe, not a public status surface. Future observability work (FEAT-012) will not extend it into a debug endpoint.
- Place the Dockerfile at the repo root so the GitHub Actions build context is straightforward.
- Defer test infrastructure to per-ticket scope; a single smoke test that hits `/health` is sufficient here.

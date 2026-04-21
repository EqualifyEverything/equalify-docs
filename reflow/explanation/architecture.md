---
title: Architecture
date: 2026-04-17
author: Equalify Tech Team
description: A system overview of Equalify Reflow — what it's made of, how data moves through it, and how it's deployed.
---

# Architecture

Equalify Reflow is three services working together: a conversion engine, a WordPress plugin, and a feedback service. This page is a system overview for people evaluating or integrating Reflow. For implementation detail — service classes, middleware order, Lua scripts, circuit-breaker thresholds — see [`docs/explanation/architecture.md` in the contributor repo](https://github.com/EqualifyEverything/equalify-reflow/blob/main/docs/explanation/architecture.md).

## System Components

A WordPress site (or any API client — including the hosted web app) talks to the Reflow API, which exposes three endpoint groups: **Document**, **Pipeline Viewer**, and **Approval**. The API is a FastAPI + Uvicorn service backed by three pieces of infrastructure:

- **Docling** — PDF extraction (layout analysis, table structure recognition, OCR)
- **S3** — document storage (uploads, generated markdown, extracted figures)
- **Redis** — job state, rate-limit counters, and the pub/sub channel used for progress streaming

### Conversion engine — `equalify-reflow`

The core service. A FastAPI application that accepts PDF uploads, runs the five-phase pipeline, and returns accessible markdown. Key responsibilities:

- REST API for document submission, status, streaming, and PII approval
- Microsoft Presidio PII scan before any AI processing
- Five-phase conversion pipeline with Claude-based agents
- Change ledger recording every AI edit with reasoning
- Job state and progress streaming via Redis

### WordPress plugin — `equalify-reflow-wp`

Integrates Reflow into a WordPress Media Library workflow. Admins process PDFs; readers see an accessible viewer with a table of contents, full-text search, and downloads. See [use the WordPress plugin](../how-to/use-the-wordpress-plugin.md).

### Feedback loop

Issue reports and text corrections submitted from the viewer are collected by the Equalify team and used to prioritise which pipeline phases need the most work. This is how real-world usage feeds back into the conversion quality over time.

## How a document moves through the system

1. **PDF uploaded** — the client sends the file to the API, which stages it in the S3 temp bucket.
2. **PII scan (Presidio)** — every document is scanned before any AI sees it:
   - *No PII detected* — the job is queued for processing.
   - *PII detected* — the job is held until a human approves or cancels it.
3. **Pipeline (5 phases)** — each phase runs an AI agent over the current document state; every edit is recorded in the change ledger with its reasoning.
4. **Results stored** — final markdown and extracted figures land in the S3 results bucket.
5. **Job marked completed** — an SSE event notifies every connected client.
6. **Client downloads** — the API hands back pre-signed S3 URLs for the markdown and figures.

## Real-time progress via Server-Sent Events

The pipeline runs independently of any connected client — fault-tolerant by design:

1. Client submits and gets a `job_id`
2. Client requests a short-lived **stream token** (browsers can't send custom headers on `EventSource`)
3. Client opens SSE with the token as a query parameter
4. Pipeline publishes progress events; the SSE endpoint relays them to subscribed clients
5. If a client disconnects, the pipeline keeps running. The client can reconnect and replay missed events

Both the built-in viewer and the WordPress plugin consume this same stream.

## Technology choices

| Concern | Stack |
|---|---|
| API framework | Python 3.11+, FastAPI, Uvicorn — async throughout |
| PDF extraction | [IBM Docling](https://github.com/docling-project/docling) — layout analysis and table structure recognition |
| AI | Claude 4.5 (Haiku as the default tier; Sonnet reserved for heavier analysis); swappable between AWS Bedrock and Anthropic direct |
| Agent framework | [PydanticAI](https://ai.pydantic.dev/) — schema-validated structured outputs, tool calling |
| PII detection | [Microsoft Presidio](https://microsoft.github.io/presidio/) |
| Object storage | S3 (real in production, [Floci](https://github.com/floci-io/floci) emulator locally) |
| Job state + pub/sub | Redis |
| Containerisation | Docker / Docker Compose (local); ECS Fargate (production) |
| Observability | Prometheus + Grafana (metrics), Jaeger (traces) |

## Privacy and security posture

- **PII scan up front.** Every document passes through Presidio before any AI sees it. Matches trigger a human-in-the-loop approval token — nothing proceeds without an explicit decision.
- **Authenticated API surface.** `/api/v1/*` endpoints require `X-API-Key`. Browser-friendly streaming uses short-lived, single-use tokens exchanged from the API key, never the API key itself.
- **Intentionally public.** The viewer SPA, `/docs` (OpenAPI), `/health`, and `/metrics` are public by design — documentation and operational probes should always be reachable.
- **Data lifecycle.** Temp uploads live in a dedicated S3 bucket with a short retention; results live in a separate bucket. Old jobs and their artefacts are cleaned up automatically.
- **No stored credentials in the pipeline path.** Production uses AWS IAM roles for Bedrock; API keys never leave the server side. Redaction happens at the middleware boundary before any log line.

## Deployment shapes

- **Local development** — `make dev` brings up the full stack in Docker Compose with hot reload. Bedrock and Anthropic direct are both supported locally via `AI_PROVIDER`.
- **Production (UIC deployment)** — ECS Fargate behind an ALB, ElastiCache Redis, S3 for storage, Bedrock for AI, CloudWatch for logs and alarms. Zero-downtime deploys via CodeDeploy blue/green.
- **Other deployments** — the stack is containerised end-to-end, so alternative hosts (self-hosted Docker, other cloud providers with compatible services) are viable. Bring your own S3-compatible storage and Redis; point `AI_PROVIDER` at whichever backend you have credentials for.

## Resilience

Reflow is designed so that a flaky dependency doesn't cascade:

- S3 operations sit behind retries with exponential backoff and circuit breakers — sustained S3 trouble fails fast instead of piling up timeouts.
- `/health` verifies Redis, S3, and queue connectivity; `/health/ready` is a lighter readiness probe for orchestration.
- Pipeline steps degrade gracefully — a non-fatal failure emits an error event and the pipeline continues with the best output it has.

For the thresholds, retry counts, specific circuit-breaker states, and the resilience test surface, see the [contributor architecture doc](https://github.com/EqualifyEverything/equalify-reflow/blob/main/docs/explanation/architecture.md) and [`docs/explanation/s3-resilience.md`](https://github.com/EqualifyEverything/equalify-reflow/blob/main/docs/explanation/s3-resilience.md) in the reflow repo.

---
title: Architecture
date: 2026-03-23
author: Equalify Tech Team
description: System architecture, components, data flow, and deployment topology of Equalify Reflow.
---

# Architecture

Equalify Reflow is composed of three services that work together: a conversion engine, a WordPress plugin, and a feedback service. This page covers the conversion engine architecture in detail.

## System Components

```
                    ┌─────────────────┐
                    │  WordPress Site  │
                    │  (reflow-wp)     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
┌──────────────────────────────────────────────────┐
│              Equalify Reflow API                  │
│              (FastAPI + Uvicorn)                   │
│                                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐│
│  │ Document  │  │ Pipeline │  │ Approval         ││
│  │ Endpoints │  │ Viewer   │  │ Endpoints        ││
│  └─────┬────┘  └─────┬────┘  └────────┬─────────┘│
│        │             │                │           │
│  ┌─────┴─────────────┴────────────────┴─────────┐│
│  │           Service Layer                       ││
│  │  ┌────────────┐  ┌──────────┐  ┌───────────┐ ││
│  │  │ Processing │  │ Storage  │  │ Queue     │ ││
│  │  │ Service    │  │ Service  │  │ Service   │ ││
│  │  └──────┬─────┘  └─────┬───┘  └─────┬─────┘ ││
│  └─────────┼──────────────┼─────────────┼───────┘│
└────────────┼──────────────┼─────────────┼────────┘
             │              │             │
    ┌────────┼──────────────┼─────────────┼────────┐
    │        ▼              ▼             ▼        │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
    │  │ Docling  │  │    S3    │  │  Redis   │   │
    │  │ Serve    │  │          │  │          │   │
    │  └──────────┘  └──────────┘  └──────────┘   │
    │              Infrastructure                   │
    └──────────────────────────────────────────────┘
```

*System component diagram showing the three-layer architecture: a WordPress site connects to the Equalify Reflow API (FastAPI + Uvicorn), which contains Document, Pipeline Viewer, and Approval endpoint groups. These feed into a shared Service Layer (Processing, Storage, and Queue services), which communicates with the infrastructure layer containing Docling Serve (PDF extraction), S3 (document storage), and Redis (job state and queuing).*

### Conversion Engine (equalify-reflow)

The core service. A FastAPI application that accepts PDF uploads, runs the seven-stage pipeline, and returns accessible markdown.

**Key responsibilities:**
- REST API for document submission, status tracking, and SSE streaming
- PII scanning via Microsoft Presidio before any AI processing
- Seven-stage conversion pipeline with AI agents (Claude via AWS Bedrock)
- Change ledger recording every edit with reasoning
- Job state management and event streaming via Redis
- Document storage and retrieval via S3

### WordPress Plugin (equalify-reflow-wp)

Integrates the conversion engine with WordPress. Administrators process PDFs from the Media Library; results are stored as WordPress posts and served through a built-in viewer.

See the [WordPress Plugin Guide](wordpress-plugin.md).

### Feedback Service (equalify-reflow-feedback)

A separate FastAPI + SQLite service that collects issue reports and text corrections from viewers. Provides filtering, aggregation, and a Metabase dashboard for analyzing feedback patterns.

## Data Flow

### Standard Processing Flow

```
1. PDF uploaded → S3 temp bucket
2. PII scan (Microsoft Presidio)
   ├─ Pass → queue for processing
   └─ Fail → hold for human approval
3. Pipeline processing (7 stages)
   └─ Each stage: AI agent processes → edits recorded in change ledger
4. Results stored in S3 results bucket
5. Job marked completed → SSE event emitted
6. Client downloads markdown + figures via pre-signed S3 URLs
```

*Data flow diagram showing the six steps of document processing: PDF upload to S3, PII scanning with pass/fail branching, seven-stage pipeline processing with change ledger recording, results storage in S3, job completion notification via SSE, and client download of markdown and figures via pre-signed URLs.*

### SSE Streaming Architecture

Real-time progress is delivered via Server-Sent Events. The architecture is designed so the pipeline runs independently of client connections:

1. Client submits a document and receives a `job_id`
2. Client requests a single-use **stream token** (because `EventSource` can't send headers)
3. Client opens SSE connection with the token as a query parameter
4. Pipeline emits events to Redis pub/sub → SSE endpoint relays to connected clients
5. If the client disconnects, the pipeline continues. The client can reconnect and replay missed events

This pattern is used by both the built-in viewer and the WordPress plugin.

## Service Layer

### ProcessingService

Orchestrates the conversion pipeline. Manages the dossier (document context that accumulates through pipeline stages), coordinates AI agents, and records the change ledger.

### StorageService

Wraps S3 operations with circuit breakers and retry logic. Handles upload, download, and pre-signed URL generation for both temp and results buckets.

### QueueService

Redis-based job queuing. Documents are enqueued after PII scanning and dequeued by background workers.

### JobService

Manages job state in Redis using Lua scripts for atomic operations. Tracks status transitions, stores metadata, and publishes state-change events.

### PIIDetectionService

Scans document text using Microsoft Presidio. Detects email addresses, phone numbers, SSNs, and other PII entity types. Configurable confidence threshold.

## AI Agent Architecture

The pipeline uses [PydanticAI](https://ai.pydantic.dev/) to define agents with tool-call interfaces. Each agent:

- Receives the page as both an **image** and **markdown text**
- Has access to the **dossier** — accumulated document context from prior stages
- Makes edits through **tool calls**, each requiring a reasoning explanation
- Can spawn **sub-agents** for specialized tasks (alt text, tables, lists)

Tool registration is **conditional** — vision tools are only provided when the task involves images, reducing prompt token waste for text-only work.

### Model Tiers

The pipeline uses different model tiers for different tasks:

- **High-capability** (Claude Sonnet) — structure analysis, page content correction, cross-page assembly
- **Efficient** (Claude Haiku) — heading reconciliation, code detection, cleanup

Model selection is managed centrally in `src/agents/model_tiers.py`.

## Infrastructure

### Local Development

```bash
make dev  # Starts everything via Docker Compose
```

| Service | Port | Purpose |
|---------|------|---------|
| API Gateway | 8080 | FastAPI application |
| Redis | 6379 | Job state, queues, pub/sub |
| LocalStack | 4566 | S3 emulation |
| Docling Serve | 5001 | PDF extraction sidecar |
| Prometheus | 9090 | Metrics collection |
| Grafana | 3001 | Monitoring dashboards |
| Jaeger | 16686 | Distributed tracing |

Code is mounted into the container with hot reload enabled — edit files on your host and changes take effect immediately.

### Production (AWS ECS)

- **ECS Fargate** — containerized API with auto-scaling
- **S3** — temp and results buckets with lifecycle policies
- **Redis (ElastiCache)** — managed Redis for job state
- **AWS Bedrock** — Claude model access (no API keys needed, uses IAM roles)
- **CloudWatch** — logs and metrics forwarding
- **Terraform** — infrastructure as code in `terraform/`

### Resilience

- **Circuit breakers** on S3 operations — prevent cascading failures when S3 is degraded
- **Retry logic** with exponential backoff on transient failures
- **Health checks** — `/health` verifies Redis, S3, and queue connectivity; `/health/ready` for orchestration probes
- **Graceful degradation** — pipeline steps that fail non-fatally emit an error event and continue

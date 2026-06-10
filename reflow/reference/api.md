---
title: API Reference
date: 2026-04-16
author: Equalify Tech Team
description: REST endpoints, authentication, request/response shapes for the Equalify Reflow API.
---

# API Reference

All endpoints are prefixed with `/api/v1/`.

**Interactive docs are the source of truth.** The OpenAPI schema is auto-generated from the code and stays in lockstep with the server. For the authoritative, always-current reference, see the running Swagger at:

- **Public instance:** [https://reflow.equalify.uic.edu/docs](https://reflow.equalify.uic.edu/docs)
- **Your own instance:** `http://<your-host>:8080/docs` (public — no auth required)

This page summarises the endpoints an API integrator is most likely to use. For recipe-style walkthroughs, see [how to integrate via the API](../how-to/integrate-via-api).

## Authentication

All `/api/v1/*` endpoints require an API key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: YOUR_API_KEY" https://your-instance/api/v1/documents/submit
```

The following are **publicly accessible** (no API key required):

- `/` — viewer SPA
- `/docs`, `/redoc`, `/openapi.json` — API documentation
- `/health`, `/health/ready` — health checks
- `/metrics` — Prometheus scrape target

Browser `EventSource` connections cannot send custom headers. For SSE, exchange an API key for a short-lived [stream token](#streaming-events).

## Document processing

### Submit a document

```
POST /api/v1/documents/submit
```

Upload a PDF. Scanned for PII, then queued for the conversion pipeline.

**Multipart request body:**

| Field | Type | Default | Description |
|---|---|---|---|
| `file` | file | required | PDF file (max 100 MB, max 50 pages) |
| `skip_pii_scan` | boolean | `false` | Bypass PII scanning (requires `skip_reason`) |
| `skip_reason` | string | — | Justification; recorded in the audit trail |
| `review_mode` | string | `"auto"` | `"auto"` completes immediately; `"human"` holds for ledger review |
| `generate_debug_bundle` | boolean | `false` | Save all agent prompts and responses |

**Response** (`201 Created`):

```json
{
  "job_id": "abc123-def456",
  "status": "pii_scanning",
  "message": "Document submitted successfully",
  "estimated_completion_minutes": 5
}
```

### Get job status

```
GET /api/v1/documents/{job_id}
```

Response shape changes with status. The most interesting fields on completion are `markdown_url`, `figures[]`, `llm_cost`, and `warnings[]`. Pre-signed S3 URLs are short-lived — download promptly.

**Possible statuses:**

| Status | Meaning |
|---|---|
| `pii_scanning` | Being scanned for PII |
| `awaiting_approval` | PII detected — waiting for human review |
| `processing` | Converting |
| `completed` | Done |
| `failed` | Error during processing |
| `denied` | PII review denied |

**Completed response (elided):**

```json
{
  "job_id": "abc123-def456",
  "status": "completed",
  "markdown_url": "https://s3.../result.md",
  "figures": [{"figure_id": "figure_1_1", "url": "https://s3.../figure_1_1.png", "page": 1, "alt_text": "", "caption": "..."}],
  "llm_cost": {"input_tokens": 45000, "output_tokens": 12000, "total_tokens": 57000, "estimated_cost_dollars": 0.34},
  "warnings": []
}
```

### Get change ledger

```
GET /api/v1/documents/{job_id}/ledger
```

Every edit the pipeline made, grouped by page — action, target, before/after text, and reasoning. Only available after processing completes.

## Streaming events

For live progress, use Server-Sent Events. This is how the built-in viewer shows real-time progress.

### Get a stream token

```
POST /api/v1/documents/{job_id}/stream/token
```

Returns a short-lived (5 min), single-use token for the SSE endpoint.

```json
{
  "token": "st_abc123...",
  "expires_in_seconds": 300,
  "stream_url": "/api/v1/documents/abc123-def456/stream?token=st_abc123..."
}
```

### Connect to the stream

```
GET /api/v1/documents/{job_id}/stream?token={stream_token}
```

**Event types:**

| Event | Data | Description |
|---|---|---|
| `pipeline:phase` | `{user_phase, display_name, step_name, step_number, total_steps}` | A pipeline stage started |
| `processing:complete` | `{}` | Finished successfully |
| `processing:error` | `{error}` | Failed |
| `done` | `{}` | Stream is closing |

> **Use `user_phase` for progress UI.** Every `pipeline:phase` event carries a `user_phase` field — one of `extraction`, `analysis`, `headings`, `translation`, `assembly`, or `review`. That's the stable public contract, matching the five phases the viewer displays. `display_name` (human-readable, e.g. "Heading Reconciliation") and `step_name` (internal identifier) are also provided for richer progress detail, but their values are not a stable contract — drive any UI state off `user_phase`.

## PII approval

When PII is detected, the job enters `awaiting_approval`. The status response includes an `approval_token` and `approval_url`.

### Get review details

```
GET /api/v1/approval/{token}/review
```

Returns job details and PII findings for the review interface.

### Submit a decision

```
POST /api/v1/approval/{token}/decision
```

**Request:**

```json
{
  "decision": "approved",
  "justification": "Course material, instructor contact info is public",
  "reviewed_by": "admin@your-institution.edu"
}
```

Approved → the job moves to `processing`. Denied → the job moves to `denied` and the document is deleted from temporary storage.

## Pipeline viewer endpoints

These endpoints power the built-in pipeline viewer served at the root path `/` — a visualizer for the AI pipeline. The viewer shows real-time progress, versioned markdown diffs, and the change ledger as a document moves through each stage. Not intended for bulk document processing.

### Process with streaming

```
POST /api/v1/pipeline/process/stream
```

Synchronous: upload a PDF and receive the full pipeline execution as an SSE stream, with versioned markdown snapshots at each stage.

**Multipart request body:**

| Field | Type | Default | Description |
|---|---|---|---|
| `file` | file | required | PDF to process |
| `images_scale` | float | `2.0` | Page image DPI scale (1.0–3.0) |
| `do_table_structure` | boolean | `true` | Enable table structure extraction |
| `ocr_languages` | string | — | OCR language codes (e.g., `"en,es"`) |

**SSE event types** emitted during the stream:

| Event | Description |
|---|---|
| `session` | Session ID for reconnection |
| `init` | Docling extraction complete — metadata + initial markdown |
| `page_image` | Individual page image (base64 PNG) |
| `figure_image` | Individual extracted figure (base64 PNG) |
| `processing` | A pipeline step is starting |
| `step` | A pipeline step completed — includes version diff |
| `error` | A step failed (non-fatal, pipeline continues) |
| `done` | Processing complete |

### Reconnect to a session

```
GET /api/v1/pipeline/sessions/{session_id}/stream?last_event_id={id}
```

Replays all events after the given ID. The pipeline continues running whether or not a client is connected.

## Health

```
GET /health        # Full check (Redis, S3, Docling, queues)
GET /health/ready  # Readiness probe
```

## Rate limiting

Per-IP limits apply to submission (`POST /api/v1/documents/submit`) and status polling. Violations return `429 Too Many Requests` with a `Retry-After` header. Adjust via env vars — see the contributor-side rate-limiting docs in the [equalify-reflow](https://github.com/EqualifyEverything/equalify-reflow) repo.

## Error format

```json
{"detail": "Job not found"}
```

| Status | Meaning |
|---|---|
| `400` | Bad request |
| `401` | Missing or invalid API key |
| `404` | Job or resource not found |
| `413` | File too large (max 100 MB) |
| `422` | Validation error |
| `429` | Rate limited |
| `500` | Internal server error |

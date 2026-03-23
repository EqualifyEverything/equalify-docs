---
title: API Reference
date: 2026-03-23
author: Equalify Tech Team
description: Complete reference for the Equalify Reflow REST API — document submission, job tracking, SSE streaming, and review workflows.
---

# API Reference

The Equalify Reflow API accepts PDF documents, processes them through the conversion pipeline, and returns accessible markdown with extracted images. All endpoints are prefixed with `/api/v1/`.

Interactive documentation is available at `/docs` on any running instance (requires HTTP Basic auth in production).

## Authentication

All API requests require an API key passed in the `X-API-Key` header:

```bash
curl -H "X-API-Key: YOUR_API_KEY" https://your-instance/api/v1/documents/submit
```

For browser-based SSE connections (where custom headers aren't possible), use a **stream token** instead — see [Streaming Events](#streaming-events).

## Document Processing

### Submit a Document

```
POST /api/v1/documents/submit
```

Upload a PDF for processing. The document is scanned for PII, then queued for the conversion pipeline.

**Request** (multipart/form-data):

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `file` | file | required | PDF file to process |
| `skip_pii_scan` | boolean | `false` | Bypass PII scanning (requires `skip_reason`) |
| `skip_reason` | string | — | Justification for skipping PII scan (recorded in audit trail) |
| `review_mode` | string | `"auto"` | `"auto"` completes immediately; `"human"` holds for ledger review |
| `generate_debug_bundle` | boolean | `false` | Save all agent prompts and responses for debugging |
| `max_rounds` | integer | `1` | Iterative refinement rounds (1–5) |

**Response** (`201 Created`):

```json
{
  "job_id": "abc123-def456",
  "status": "pii_scanning",
  "message": "Document submitted successfully",
  "estimated_completion_minutes": 3
}
```

**Example — submit with curl:**

```bash
curl -X POST https://your-instance/api/v1/documents/submit \
  -H "X-API-Key: YOUR_API_KEY" \
  -F "file=@syllabus.pdf" \
  -F "review_mode=human"
```

**Example — submit from WordPress (PHP):**

```php
$response = wp_remote_post($api_url . '/api/v1/documents/submit', [
    'headers' => ['X-API-Key' => $api_key],
    'body'    => ['file' => new CURLFile($pdf_path, 'application/pdf')],
]);
$job_id = json_decode(wp_remote_retrieve_body($response))->job_id;
```

### Get Job Status

```
GET /api/v1/documents/{job_id}
```

Returns the current state of a processing job. The response shape changes based on the job's status.

**Possible statuses:**

| Status | Description |
|--------|-------------|
| `pii_scanning` | Document is being scanned for PII |
| `awaiting_approval` | PII detected — waiting for human review |
| `processing` | Document is being converted |
| `completed` | Processing finished successfully |
| `needs_review` | Low-confidence results require human review |
| `failed` | Processing encountered an error |
| `denied` | PII review was denied |

**Completed response example:**

```json
{
  "job_id": "abc123-def456",
  "status": "completed",
  "review_mode": "auto",
  "markdown_url": "https://s3.../result.md",
  "figures": [
    {
      "figure_id": "figure_1_1",
      "url": "https://s3.../figure_1_1.png",
      "page": 1,
      "alt_text": "Bar chart showing enrollment by department",
      "caption": "Figure 1: Fall 2025 Enrollment"
    }
  ],
  "confidence_score": 0.92,
  "llm_cost": {
    "input_tokens": 45000,
    "output_tokens": 12000,
    "total_tokens": 57000,
    "estimated_cost_dollars": 0.34
  },
  "warnings": []
}
```

The `markdown_url` and figure `url` fields are pre-signed S3 URLs valid for a limited time. Download them promptly after job completion.

**Example — poll until complete (JavaScript):**

```javascript
async function waitForCompletion(jobId, apiKey) {
  while (true) {
    const res = await fetch(`/api/v1/documents/${jobId}`, {
      headers: { 'X-API-Key': apiKey }
    });
    const data = await res.json();

    if (data.status === 'completed') return data;
    if (data.status === 'failed') throw new Error(data.error);

    await new Promise(r => setTimeout(r, 5000)); // poll every 5s
  }
}
```

### Get Change Ledger

```
GET /api/v1/documents/{job_id}/ledger
```

Returns the complete change ledger — every edit the pipeline made, grouped by page. Only available after processing completes.

**Response example:**

```json
{
  "job_id": "abc123-def456",
  "document_title": "BIOS 343 Syllabus",
  "total_pages": 13,
  "pages_with_changes": 11,
  "total_edits": 69,
  "pages": [
    {
      "page": 1,
      "edit_count": 8,
      "entries": [
        {
          "entry_id": "e1",
          "action": "modify",
          "target": "heading",
          "before": "## BIOS 343",
          "after": "# BIOS 343: Animal Physiology",
          "reasoning": "Promoted to H1 — this is the document title based on font size and position",
          "confidence": 0.95,
          "needs_review": false
        }
      ]
    }
  ],
  "final_markdown_url": "https://s3.../result.md"
}
```

## Streaming Events

For real-time progress tracking, connect to the SSE (Server-Sent Events) stream. This is how both the built-in viewer and the WordPress plugin display live progress.

### Get a Stream Token

```
POST /api/v1/documents/{job_id}/stream/token
```

Browser `EventSource` cannot send custom headers. This endpoint generates a single-use, short-lived token that authenticates the SSE connection via query parameter.

**Response:**

```json
{
  "token": "st_abc123...",
  "expires_in_seconds": 300,
  "stream_url": "/api/v1/documents/abc123-def456/stream?token=st_abc123..."
}
```

### Connect to the Stream

```
GET /api/v1/documents/{job_id}/stream?token={stream_token}
```

Opens an SSE connection. The server sends events as the pipeline progresses.

**Event types:**

| Event | Data | Description |
|-------|------|-------------|
| `pipeline:phase` | `{display_name, step_number, total_steps}` | A pipeline stage started |
| `processing:complete` | `{}` | Processing finished successfully |
| `processing:error` | `{error}` | Processing failed |
| `done` | `{}` | Stream is closing |

**Example — connect from JavaScript:**

```javascript
// 1. Get stream token (requires API key)
const tokenRes = await fetch(`/api/v1/documents/${jobId}/stream/token`, {
  method: 'POST',
  headers: { 'X-API-Key': apiKey }
});
const { token, stream_url } = await tokenRes.json();

// 2. Connect to SSE stream (token in URL, no headers needed)
const source = new EventSource(stream_url);

source.addEventListener('pipeline:phase', (e) => {
  const { display_name, step_number, total_steps } = JSON.parse(e.data);
  console.log(`Step ${step_number}/${total_steps}: ${display_name}`);
});

source.addEventListener('processing:complete', () => {
  source.close();
  // Fetch final results via GET /documents/{job_id}
});

source.addEventListener('processing:error', (e) => {
  const { error } = JSON.parse(e.data);
  console.error('Processing failed:', error);
  source.close();
});
```

**Example — connect from WordPress (PHP + JavaScript):**

```javascript
// In admin-media.js — after submitting the PDF:

// Get token through WordPress REST proxy (avoids exposing API key to browser)
const tokenRes = await fetch(`/wp-json/equalify-reflow/v1/stream-token/${attachmentId}`, {
  headers: { 'X-WP-Nonce': wpNonce }
});
const { token, stream_url, job_id } = await tokenRes.json();

// Connect using the client-facing API URL
const source = new EventSource(`${clientApiUrl}${stream_url}`);

source.addEventListener('pipeline:phase', (e) => {
  const data = JSON.parse(e.data);
  updateProgressBar(data.step_number, data.total_steps, data.display_name);
});

source.addEventListener('processing:complete', () => {
  source.close();
  completeProcessing(attachmentId); // triggers result download
});
```

The WordPress plugin proxies the stream token through its own REST endpoint so the browser never sees the API key. The SSE connection itself uses the single-use token.

## PII Approval

When PII is detected, the job enters `awaiting_approval` status. The status response includes an `approval_token` and `approval_url`.

### Get Review Details

```
GET /api/v1/approval/{token}/review
```

Returns job details and PII findings for the review interface.

**Response:**

```json
{
  "job_id": "abc123",
  "status": "awaiting_approval",
  "pii_findings": [
    {"entity_type": "EMAIL_ADDRESS", "text": "john@example.com", "score": 0.95},
    {"entity_type": "PHONE_NUMBER", "text": "312-555-0100", "score": 0.88}
  ],
  "created_at": "2026-03-23T10:00:00Z",
  "expires_at": "2026-03-23T11:00:00Z"
}
```

### Submit Approval Decision

```
POST /api/v1/approval/{token}/decision
```

**Request:**

```json
{
  "decision": "approved",
  "justification": "Course material, no student PII — instructor contact info is public",
  "reviewed_by": "admin@uic.edu"
}
```

If approved, the document is queued for processing. If denied, the job moves to `denied` status.

## Pipeline Viewer (Development)

These endpoints power the built-in pipeline viewer at `/viewer`. They're useful for development and debugging but are not part of the standard document processing flow.

### Process with Streaming

```
POST /api/v1/pipeline/process/stream
```

Upload a PDF and receive an SSE stream of the full pipeline execution with versioned markdown snapshots at each stage. Unlike the document submission flow, this is synchronous — you upload and stream in one connection.

**Request** (multipart/form-data):

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `file` | file | required | PDF to process |
| `images_scale` | float | `2.0` | Page image DPI scale (1.0–3.0) |
| `do_table_structure` | boolean | `true` | Enable table structure extraction |
| `ocr_languages` | string | — | OCR language codes (e.g., `"en,es"`) |

**SSE event types:**

| Event | Description |
|-------|-------------|
| `session` | Session ID for reconnection |
| `init` | Docling extraction complete — metadata + initial markdown |
| `page_image` | Individual page image (base64 PNG) |
| `figure_image` | Individual extracted figure (base64 PNG) |
| `processing` | A pipeline step is starting |
| `step` | A pipeline step completed — includes version diff |
| `error` | A step failed (non-fatal, pipeline continues) |
| `done` | Processing complete |

### Reconnect to Session

```
GET /api/v1/pipeline/sessions/{session_id}/stream?last_event_id={id}
```

If disconnected, reconnect to a running session and replay all events after the given ID. The pipeline continues running whether or not a client is connected.

### Feedback and Review

The pipeline viewer supports multi-round feedback sessions for iterative refinement:

```
POST /api/v1/pipeline/sessions/{session_id}/feedback
```

Submit edits or comments anchored to specific text spans. The system generates candidate changes that can be individually accepted or rejected:

```
POST /api/v1/pipeline/sessions/{session_id}/review
```

When satisfied, finalize the session:

```
POST /api/v1/pipeline/sessions/{session_id}/approve
```

## Health Checks

```
GET /health        # Full health check (Redis, S3, queues)
GET /health/ready  # Readiness probe for orchestration
```

## Rate Limiting

The API enforces per-IP rate limits on document submission. Default limits can be configured via environment variables. When rate-limited, the API returns `429 Too Many Requests` with a `Retry-After` header.

## Error Responses

All errors follow a consistent format:

```json
{
  "detail": "Job not found"
}
```

| Status | Meaning |
|--------|---------|
| `400` | Bad request (invalid parameters, job not ready) |
| `401` | Missing or invalid API key |
| `404` | Job or resource not found |
| `413` | File too large (max 50 MB) |
| `422` | Validation error (details in response body) |
| `429` | Rate limited |
| `500` | Internal server error |

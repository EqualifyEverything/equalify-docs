---
title: Integrate via the API
date: 2026-04-16
author: Equalify Tech Team
description: Task-focused recipes for the common API workflows — submit a PDF, stream progress, handle PII approval, and download results.
---

# Integrate via the API

Recipes for the workflows most API integrations need. For the full endpoint surface, see the [API reference](../reference/api).

All examples assume you have an API key and the Reflow API host (`$API_URL`) set up. For a deployed UIC instance, use `https://reflow.equalify.uic.edu`.

## Submit a PDF and wait for completion

The simplest integration — submit, poll, download. Good for batch jobs and server-side code that doesn't need real-time progress.

```javascript
async function convertPdf(apiUrl, apiKey, pdfBuffer, filename) {
  // 1. Submit
  const formData = new FormData();
  formData.append('file', new Blob([pdfBuffer], { type: 'application/pdf' }), filename);

  const submitRes = await fetch(`${apiUrl}/api/v1/documents/submit`, {
    method: 'POST',
    headers: { 'X-API-Key': apiKey },
    body: formData,
  });
  const { job_id } = await submitRes.json();

  // 2. Poll every 5 seconds
  while (true) {
    const statusRes = await fetch(`${apiUrl}/api/v1/documents/${job_id}`, {
      headers: { 'X-API-Key': apiKey },
    });
    const data = await statusRes.json();

    if (data.status === 'completed') {
      const markdown = await (await fetch(data.markdown_url)).text();
      return { markdown, figures: data.figures, ledger_url: `${apiUrl}/api/v1/documents/${job_id}/ledger` };
    }
    if (data.status === 'failed') throw new Error(data.error);
    if (data.status === 'denied') throw new Error('PII review denied');
    if (data.status === 'awaiting_approval') {
      throw new Error(`PII approval required — see approval_url: ${data.approval_url}`);
    }

    await new Promise(r => setTimeout(r, 5000));
  }
}
```

## Stream progress via SSE

For user-facing integrations (like a progress bar), use Server-Sent Events. Browsers can't send `X-API-Key` from `EventSource`, so you exchange your API key for a short-lived stream token first.

```javascript
async function submitAndStream(apiUrl, apiKey, pdfFile, onPhase, onComplete, onError) {
  // 1. Submit normally (server-side or through a proxy)
  const formData = new FormData();
  formData.append('file', pdfFile);
  const submitRes = await fetch(`${apiUrl}/api/v1/documents/submit`, {
    method: 'POST',
    headers: { 'X-API-Key': apiKey },
    body: formData,
  });
  const { job_id } = await submitRes.json();

  // 2. Exchange API key for a stream token
  const tokenRes = await fetch(`${apiUrl}/api/v1/documents/${job_id}/stream/token`, {
    method: 'POST',
    headers: { 'X-API-Key': apiKey },
  });
  const { stream_url } = await tokenRes.json();

  // 3. Connect to SSE — token is in the URL, no headers
  const source = new EventSource(`${apiUrl}${stream_url}`);

  source.addEventListener('pipeline:phase', (e) => {
    // `user_phase` is the stable public contract: "extraction" | "analysis" |
    // "headings" | "translation" | "assembly" | "review". Drive progress UI
    // off that. display_name / step_name are for richer human-readable output.
    const { user_phase, display_name, step_number, total_steps } = JSON.parse(e.data);
    onPhase({ user_phase, display_name, step_number, total_steps });
  });

  source.addEventListener('processing:complete', async () => {
    source.close();
    const statusRes = await fetch(`${apiUrl}/api/v1/documents/${job_id}`, {
      headers: { 'X-API-Key': apiKey },
    });
    onComplete(await statusRes.json());
  });

  source.addEventListener('processing:error', (e) => {
    source.close();
    onError(JSON.parse(e.data).error);
  });

  return job_id;
}
```

In browser integrations where you should **never** expose the API key to the client (the WordPress plugin works this way), proxy the submit + token-exchange through your own server, and only hand the browser the stream URL.

## Handle a PII-flagged document

When Presidio flags PII in a submission, the job lands in `awaiting_approval` instead of `processing`. Your integration needs a decision step before the pipeline runs.

```javascript
async function approveAndContinue(apiUrl, apiKey, jobId, justification, reviewedBy) {
  // Fetch the approval token + findings
  const statusRes = await fetch(`${apiUrl}/api/v1/documents/${jobId}`, {
    headers: { 'X-API-Key': apiKey },
  });
  const { status, approval_token, pii_findings } = await statusRes.json();

  if (status !== 'awaiting_approval') {
    throw new Error(`Job is in status ${status}, not awaiting_approval`);
  }

  // Show the findings to your reviewer (UI varies by app)
  console.log('PII findings:', pii_findings);
  //   e.g. [{entity_type: "EMAIL_ADDRESS", text: "...", score: 1.0}]

  // Submit the decision
  const decisionRes = await fetch(`${apiUrl}/api/v1/approval/${approval_token}/decision`, {
    method: 'POST',
    headers: { 'X-API-Key': apiKey, 'Content-Type': 'application/json' },
    body: JSON.stringify({
      decision: 'approved',  // or 'denied'
      justification,
      reviewed_by: reviewedBy,
    }),
  });

  return decisionRes.json();
}
```

The approval token is single-use, has a 4-hour TTL, and is scoped to this specific job. After approval, continue with the normal poll/SSE flow above.

## Retrieve the change ledger

Every edit the pipeline made, with reasoning:

```bash
curl -H "X-API-Key: $API_KEY" \
  "$API_URL/api/v1/documents/$JOB_ID/ledger" \
  | jq '.pages[0].entries[0]'
```

Sample output:

```json
{
  "entry_id": "e1",
  "action": "modify",
  "target": "heading",
  "before": "## BIOS 343",
  "after": "# BIOS 343: Animal Physiology",
  "reasoning": "Promoted to H1 — this is the document title based on font size and position"
}
```

The ledger is the audit trail for every AI-driven change. Good for quality review, regression testing against specific documents, and showing users *why* an edit was made.

## Download figures

Figures are extracted as separate files and linked from the markdown with standard `![alt](path)` syntax. The status response lists them with pre-signed URLs:

```javascript
for (const fig of data.figures) {
  const res = await fetch(fig.url);
  const blob = await res.blob();
  // Save alongside the markdown. Use fig.figure_id as the filename for the
  // links in result.md to resolve.
}
```

Pre-signed URLs are short-lived — download promptly after the job completes.

## PHP submission from WordPress

```php
$response = wp_remote_post($api_url . '/api/v1/documents/submit', [
    'headers' => ['X-API-Key' => $api_key],
    'body'    => ['file' => new CURLFile($pdf_path, 'application/pdf')],
]);
$job_id = json_decode(wp_remote_retrieve_body($response))->job_id;
```

Keep the API key on the server side. Never inject it into page HTML or browser-executed JS — use the stream-token exchange pattern above instead.

## When to use the pipeline viewer endpoints instead

The endpoints above (`/documents/*`, `/approval/*`) are the right choice for production integrations. They are async, durable, and rate-limited.

The pipeline viewer endpoints (`/pipeline/process/stream`, `/pipeline/sessions/*`) are for interactive, one-off processing — they run synchronously, stream intermediate markdown versions, and don't produce durable jobs. Use them when building a viewer-like UI, not for bulk conversion.

---
title: WordPress Plugin Guide
date: 2026-03-23
author: Equalify Tech Team
description: Install and use the Equalify Reflow for WordPress plugin to convert PDFs to accessible HTML from the Media Library.
---

# WordPress Plugin Guide

The Equalify Reflow for WordPress plugin lets administrators process PDF attachments directly from the WordPress Media Library. Converted documents are served through an accessible viewer that supports table of contents navigation, full-text search, and downloadable markdown.

## Installation

1. Download or clone the plugin into your WordPress plugins directory:

```bash
cd wp-content/plugins/
git clone https://github.com/EqualifyEverything/equalify-reflow-wp.git
```

2. Activate the plugin from **Plugins > Installed Plugins** (or **Network Activate** for multisite).

3. Navigate to **Settings > Equalify Reflow** and configure:
   - **API URL** — the address of your Reflow instance (e.g., `https://pdf.equalify.uic.edu`)
   - **API Key** — your `X-API-Key` credential
   - **Client API URL** (optional) — if the browser needs a different address than the server to reach the API (common in local development with Docker)

## Processing a PDF

1. Go to **Media Library** and click on any PDF attachment.
2. In the attachment detail panel, find the **Equalify Reflow** section.
3. Click **Run Equalify Reflow**.

The plugin submits the PDF to the Reflow API and displays real-time progress as the document moves through each pipeline stage:

- **Extracting** — Docling parses the PDF structure
- **Analyzing Structure** — document classification and outline
- **Reconciling Headings** — heading hierarchy normalization
- **Page Corrections** — per-page content and formatting fixes
- **Assembling** — cross-page boundary fixes and cleanup

When processing completes, the plugin automatically:

- Downloads the accessible markdown and saves it to `wp-content/uploads/equalify-reflow/`
- Sideloads extracted figures into the WordPress media library with alt text
- Sets the attachment status to **Ready**

## The Viewer

Each processed document gets a public URL at:

```
/equalify-reflow/{attachment-id}/{slug}/
```

The viewer renders the markdown as accessible HTML with:

- **Table of contents** — auto-generated from document headings
- **Full-text search** — search within the document with result highlighting and keyboard navigation
- **Downloads** — download the original PDF or the accessible markdown
- **Responsive layout** — adapts to any screen size

### Browsing All Documents

A document index is available at `/equalify-reflow/` listing all processed and enabled documents.

### Downloading a Bundle

Each document also has a download endpoint at:

```
/equalify-reflow/{attachment-id}/{slug}/download/
```

This serves a ZIP file containing the markdown and all extracted figures.

## PDF Link Annotation

When the plugin detects PDF links in your post content, it automatically adds an accessibility icon next to each link. Clicking the icon opens the accessible viewer instead of downloading the PDF.

This works automatically for any PDF in the media library that has been processed and enabled. No shortcodes or manual markup required.

## Managing Documents

From the Media Library attachment panel:

- **Enable/Disable** — toggle public visibility of the accessible version without deleting it
- **Delete Reflow Data** — remove the markdown, figures, and all metadata. The original PDF is untouched
- **Re-process** — run the pipeline again to pick up improvements (deletes existing data first)

## Feedback Collection

If feedback is enabled in settings, the viewer includes a feedback interface where users can:

- **Report issues** — describe a problem with category tagging (content, formatting, accessibility, structure)
- **Suggest corrections** — select text and propose edits with before/after tracking

Feedback is sent to the [Equalify Reflow Feedback Service](https://github.com/EqualifyEverything/equalify-reflow-feedback), a separate service that collects and aggregates reports across all connected clients.

### Configuring Feedback

In **Settings > Equalify Reflow**:

- **Enable Feedback** — turn feedback collection on or off
- **Feedback Service URL** — address of the feedback service
- **Feedback API Key** — authentication key for the feedback service

## How It Works Under the Hood

The plugin communicates with the Reflow API through a series of REST calls:

1. **Submit** — `POST /api/v1/documents/submit` uploads the PDF with PII scanning skipped (WordPress content is assumed pre-vetted)
2. **Stream token** — `POST /api/v1/documents/{job_id}/stream/token` generates a single-use token for the browser
3. **SSE stream** — browser connects to `GET /api/v1/documents/{job_id}/stream?token=...` for real-time progress
4. **Status check** — `GET /api/v1/documents/{job_id}` retrieves the completed result with pre-signed S3 URLs
5. **Download** — the plugin fetches the markdown and figure images from S3 and stores them locally in WordPress

If the SSE stream disconnects, the plugin falls back to polling the status endpoint every 5 seconds until the job completes.

All API communication happens server-side (PHP) except for the SSE stream, which connects directly from the browser. The stream token mechanism ensures the API key is never exposed to the client.

## Multisite Support

The plugin supports WordPress multisite installations. Each site in the network can have its own API configuration (URL, key, feedback settings). Activate the plugin at the network level and configure per-site in each site's **Settings > Equalify Reflow**.

## Troubleshooting

**"Connection failed" when processing**

- Verify the API URL is reachable from your server (not just your browser)
- Check that the API key is correct in settings
- If using Docker locally, the Client API URL may need to differ from the server-side API URL

**Processing seems stuck**

- The plugin falls back to polling if SSE disconnects. Check the browser console for connection errors
- Large documents (20+ pages) can take 5–8 minutes — this is normal

**Figures not displaying**

- Figures are sideloaded as WordPress attachments. Check that `wp-content/uploads/` is writable
- Verify the original PDF attachment still exists (figures are attached as children)

**Viewer returns 404**

- Flush rewrite rules: **Settings > Permalinks > Save Changes** (no changes needed, just save)
- Verify the attachment status is "Ready" and "Enabled" in the Media Library panel

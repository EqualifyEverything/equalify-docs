---
title: Use the WordPress plugin
date: 2026-04-16
author: Equalify Tech Team
description: Install, configure, and use the Equalify Reflow for WordPress plugin to convert PDFs to accessible HTML from the Media Library.
---

# Use the WordPress plugin

The Equalify Reflow for WordPress plugin lets administrators process PDF attachments directly from the Media Library. Converted documents are served through an accessible viewer that supports a table of contents, full-text search, and downloadable markdown.

For a narrative first-time walkthrough of installing the plugin and converting your first PDF, see [process your first PDF with WordPress](../tutorials/process-your-first-pdf-with-wordpress). This page is the task reference for day-to-day use.

## Install and configure

1. Download the latest release ZIP from the [equalify-reflow-wp releases page](https://github.com/EqualifyEverything/equalify-reflow-wp/releases). Save it somewhere you can find it — you don't need to unzip it.

2. In the WordPress admin, go to **Plugins > Add New > Upload Plugin**. Choose the ZIP, click **Install Now**, then **Activate Plugin**. On a multisite, use **Network Activate** instead so each site can point at its own Reflow instance.

3. Navigate to **Settings > Equalify Reflow** and fill in these fields:

   - **API URL** — the address of the Reflow service the plugin should talk to. If your organisation is using the hosted UIC instance, this is `https://reflow.equalify.uic.edu`. If you're running Reflow yourself, this is wherever you've deployed it. You'll only change this if you move to a different Reflow instance.
   - **API Key** — the access credential for that Reflow instance. It's issued to you during partner onboarding. Treat it like a password: paste it here, don't share it, don't commit it to Git.
   - **Client API URL** (optional) — leave this blank in normal use. It only matters if the server and the visitor's browser need to reach the API at different addresses — which really only happens in local development with Docker.

> **Power users:** you can also install the plugin by uploading the ZIP's contents directly to `wp-content/plugins/` over SFTP, or via WP-CLI (`wp plugin install <path-to-zip> --activate`). The admin UI flow above is the supported path for everyone else.

## Process a PDF

1. Go to **Media Library** and click on any PDF attachment.
2. In the attachment detail panel, find the **Equalify Reflow** section.
3. Click **Run Equalify Reflow**.

The plugin sends the PDF to the Reflow service and shows live progress through the five stages of the conversion:

- **Extraction** — pulling the text, tables, and images out of the PDF
- **Analysis** — working out what kind of document this is and building its outline
- **Headings** — sorting out the heading levels so the outline is consistent
- **Translation** — going through page by page and correcting the text and layout
- **Assembly** — stitching the pages together and cleaning up the seams

When processing finishes, the plugin automatically:

- Saves the accessible version into WordPress
- Adds each extracted figure to the Media Library with alt text
- Marks the attachment as **Ready**

## The viewer

Each processed document gets a public URL at:

```
/equalify-reflow/{attachment-id}/{slug}/
```

The viewer renders the markdown as accessible HTML with:

- **Table of contents** — built automatically from the document's headings
- **Full-text search** — search inside the document with highlighted results and keyboard shortcuts
- **Downloads** — the original PDF or the accessible text version
- **Responsive layout** — adapts to any screen size, from phone to desktop

### Browsing all documents

A document index is available at `/equalify-reflow/` listing all processed and enabled documents.

### Downloading a bundle

Each document has a download endpoint at:

```
/equalify-reflow/{attachment-id}/{slug}/download/
```

This serves a ZIP containing the accessible text version of the document and all extracted figures.

## PDF link annotation

When the plugin detects PDF links in your post content, it automatically adds an accessibility icon next to each link. Clicking the icon opens the accessible viewer instead of downloading the PDF.

This works automatically for any processed, enabled PDF in the Media Library — no shortcodes or manual markup required.

## Manage documents

From the Media Library attachment panel:

- **Enable / Disable** — show or hide the accessible version on your public site without deleting it
- **Delete Reflow Data** — remove the accessible text, figures, and all related metadata. The original PDF is untouched
- **Re-process** — run the conversion again to pick up pipeline improvements (existing data is replaced)

## Collect feedback

If feedback is enabled in settings, the viewer includes an interface where users can:

- **Report issues** — describe a problem and tag it with a category (content, formatting, accessibility, structure)
- **Suggest corrections** — highlight text and propose an edit; the original and your suggested version are captured together

Reports are reviewed by the Equalify team and feed directly into pipeline improvements.

### Configuring feedback

In **Settings > Equalify Reflow**:

- **Enable Feedback** — the on/off switch for the feedback widgets inside the viewer. Turn this off if you don't want readers to submit reports at all.
- **Feedback Service URL** — where the reports are sent. Your partner onboarding will tell you which URL to use.
- **Feedback API Key** — the access credential for sending feedback, issued with your onboarding. Like the main API key, treat it as a secret.

## Under the hood

For developers and integrators: the plugin talks to the Reflow API through a short sequence of REST calls — submit the PDF, open a live progress stream using a short-lived token so the API key never reaches the browser, then fetch the finished text and figures and save them into WordPress. If the live stream drops, the plugin quietly falls back to checking progress every few seconds until the job finishes.

The full API walkthrough is in [integrate via the API](integrate-via-api).

## Multisite support

The plugin supports WordPress multisite. Each site can have its own API configuration (URL, key, feedback settings). Activate at the network level and configure per-site in each site's **Settings > Equalify Reflow**.

## Troubleshooting

**"Connection failed" when processing**

- Check that the API URL in settings is reachable from the WordPress server, not just from your laptop
- Check that the API key is correct — a stray space when pasting is a common cause
- If you're running everything locally with Docker, you may need to fill in the Client API URL

**Processing seems stuck**

- If the live progress stream drops, the plugin falls back to checking progress every few seconds — give it 30 seconds
- Large documents (20+ pages) can take 5–8 minutes; this is normal

**Figures not displaying**

- Figures are added to the Media Library as attachments. Make sure `wp-content/uploads/` is writable by the web server
- Check the original PDF attachment still exists — the figures are stored as children of it

**Viewer returns 404**

- Go to **Settings > Permalinks > Save Changes** (no changes needed, just save — this refreshes WordPress's URL rules)
- Check the attachment is marked as "Ready" and "Enabled" in the Media Library panel

**Job finishes in seconds with no AI improvements visible**

- The Reflow service's AI credentials most likely expired. The person who runs your Reflow instance needs to refresh them and restart the service.

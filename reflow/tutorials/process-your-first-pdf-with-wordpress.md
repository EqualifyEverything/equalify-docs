---
title: Tutorial — Process your first PDF with WordPress
date: 2026-04-16
author: Equalify Tech Team
description: Install the Equalify Reflow for WordPress plugin, configure it against a running Reflow instance, and convert your first PDF end-to-end.
---

# Tutorial: Process your first PDF with WordPress

By the end of this walkthrough you will have installed the Equalify Reflow for WordPress plugin, pointed it at a running Reflow API, and converted a real PDF from the Media Library into an accessible viewer page. Plan for **~20 minutes** on a site that already has WordPress running.

For day-to-day reference on the plugin's features, see [use the WordPress plugin](../how-to/use-the-wordpress-plugin.md). This tutorial walks you through the path end-to-end; that page is the task reference after you're set up.

## What you need

- **A WordPress site** (single or multisite) where you have administrator access. WordPress 6.0+ recommended.
- **A Reflow API instance** you can reach over HTTPS and its `X-API-Key` credential. If your organisation uses the UIC-hosted instance, that's `https://reflow.equalify.uic.edu`; partners receive the API key through onboarding.
- **A PDF** already uploaded to your Media Library, or one you'll upload during the tutorial.

If you don't yet have partner access, see [getting started § partners](../getting-started.md#for-partners).

## 1. Install the plugin

Download the latest release ZIP from the [equalify-reflow-wp releases page](https://github.com/EqualifyEverything/equalify-reflow-wp/releases). Keep the ZIP as-is — you don't need to unzip it.

In the WordPress admin:

1. Go to **Plugins > Add New**.
2. Click **Upload Plugin** at the top of the page.
3. Choose the ZIP you just downloaded and click **Install Now**.
4. When the upload finishes, click **Activate Plugin**. On a multisite, choose **Network Activate** so individual sites can point at their own Reflow instance later.

> **Power users:** if you prefer the command line, you can also install via WP-CLI (`wp plugin install <path-to-zip> --activate`) or by uploading the ZIP's contents to `wp-content/plugins/` over SFTP. The admin UI path above is the one we document and support for everyone else.

## 2. Configure the API connection

Navigate to **Settings > Equalify Reflow** and fill in:

- **API URL** — the address of the Reflow service the plugin should talk to. For the UIC-hosted instance, paste `https://reflow.equalify.uic.edu`. You'll only change this later if you move to a different Reflow instance.
- **API Key** — the access credential you were issued during partner onboarding. Treat it like a password.
- **Client API URL** — leave blank. It's only needed if the WordPress server and the visitor's browser need to reach the Reflow service at different addresses, which usually only happens in local Docker development.

Save. The plugin runs a quick connectivity check; if the green "Connected" indicator doesn't appear, see [Troubleshooting](#troubleshooting) below.

## 3. Upload or find a PDF

In the WordPress admin, go to **Media > Library**. Either upload a new PDF or click an existing one. The attachment detail panel opens.

Scroll to the **Equalify Reflow** section. If the plugin is configured correctly, you'll see a **Run Equalify Reflow** button.

## 4. Start the conversion

Click **Run Equalify Reflow**.

The plugin sends the PDF to the Reflow service and shows a progress bar as it moves through the five stages of the conversion:

1. **Extraction** — pulling the text, tables, and images out of the PDF
2. **Analysis** — working out what kind of document this is and building its outline
3. **Headings** — sorting out the heading levels so the outline is consistent
4. **Translation** — going through page by page and correcting the text and layout
5. **Assembly** — stitching the pages together and cleaning up the seams

Expected duration for a 6–10 page document: **~2–5 minutes**. Larger or more complex documents take proportionally longer. If the live progress connection drops, the plugin falls back to checking progress every few seconds; you'll see the bar keep moving.

## 5. See the accessible viewer

When processing completes, the attachment panel shows:

- **Status: Ready**
- A link to the public viewer at `/equalify-reflow/{attachment-id}/{slug}/`

Open the viewer link. You should see:

- The document rendered as an accessible web page
- A **table of contents** down the side, built automatically from the document's headings
- A **search** box that highlights matches across the document
- Download links for the original PDF and the accessible text version

Click through the table of contents and check that the outline matches what you'd expect for this document. This is the single most important quality check — see [interpret the output](../how-to/interpret-the-output.md) for the full 4-minute review.

## 6. Check the media library updated

Back in **Media > Library**, notice:

- The figures from the PDF are now separate Media Library entries, attached to the original PDF
- Each figure has alt text written by the pipeline's image-description specialist

## 7. Explore what else the plugin does

Now that the document is processed, the plugin has automatically activated a few features:

- **PDF link annotation** — anywhere you link to this PDF in a post or page, an accessibility icon appears next to the link. Clicking it opens the accessible viewer instead of downloading the PDF.
- **Document index** — `/equalify-reflow/` lists every processed and enabled document on the site.
- **Download bundle** — append `/download/` to the viewer URL to get a ZIP containing the accessible text and figures.

## 8. Enable feedback (optional)

In **Settings > Equalify Reflow**, turn on **Enable Feedback** and point it at a feedback service (e.g. `https://feedback.equalify.example/api`). Readers of the accessible viewer can now:

- Report issues and tag them with a category (content, formatting, accessibility, structure)
- Highlight specific text and propose a correction (select text → suggest edit)

Feedback flows to the central [Equalify Reflow Feedback Service](https://github.com/EqualifyEverything/equalify-reflow-feedback), which collects reports from every connected WordPress site and feeds them back into pipeline improvements.

## Troubleshooting

| Problem | First thing to try |
|---|---|
| "Connection failed" when saving settings | Confirm the API URL is reachable from the WordPress server itself, not just from your laptop. If you have command-line access to the server, `curl $API_URL/health` is a quick check (no API key needed). Otherwise ask whoever maintains the server. |
| Progress bar stuck at 0% | The live progress stream may have dropped. The plugin falls back to checking progress every few seconds; give it 30 seconds. |
| Processing completes but figures don't appear | Make sure `wp-content/uploads/` is writable by the web server. |
| Viewer URL returns 404 | Go to **Settings > Permalinks > Save Changes** (no changes needed, just save — this refreshes WordPress's URL rules). |
| Processing finishes in seconds with poor-quality output | The Reflow service's AI credentials most likely expired. Ask whoever runs your Reflow instance to refresh them and restart the service. |

## Where to go next

- [Use the WordPress plugin](../how-to/use-the-wordpress-plugin.md) — day-to-day reference (managing documents, re-processing, multisite setup)
- [Interpret the output](../how-to/interpret-the-output.md) — the reviewer's quality checklist
- [Provide feedback](../how-to/provide-feedback.md) — how to submit corrections and issue reports
- [How it works](../explanation/how-it-works.md) — what's actually happening during those five stages

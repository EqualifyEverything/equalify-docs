---
title: Tutorial — Use the pipeline viewer
date: 2026-04-17
author: Equalify Tech Team
description: Convert a PDF in the browser using the Reflow pipeline viewer at reflow.equalify.uic.edu — drag-and-drop, live progress, and every tool the viewer offers.
---

# Tutorial: Use the pipeline viewer

By the end of this walkthrough you'll convert a PDF using the built-in pipeline viewer in a browser — no WordPress, no API integration, no code. You'll watch the pipeline progress live, inspect each phase's output, look at the change ledger, and download the final accessible markdown. Plan for **~10 minutes**.

The viewer is the fastest path for accessibility reviewers, partners previewing Reflow before a pilot, and anyone who wants to see what the pipeline actually does to a document. For the WordPress-admin flow, see [process your first PDF with WordPress](process-your-first-pdf-with-wordpress); for programmatic integration, see [integrate via the API](../how-to/integrate-via-api).

## What you need

- **A running Reflow instance.** If you have access to the UIC deployment, use `https://reflow.equalify.uic.edu`. If you're running it yourself, `make dev` in the [`equalify-reflow`](https://github.com/EqualifyEverything/equalify-reflow) repo brings up `http://localhost:8080/`.
- **A PDF to convert.** Anything under 100 MB and 50 pages. Short documents (under ten pages) finish in 2–4 minutes.
- **A browser.** Tested in current Chrome, Firefox, and Safari.

No API key is needed — the viewer SPA is served same-origin and talks to the API through short-lived browser tokens under the hood.

## 1. Open the viewer

Point your browser at the Reflow instance root — for example `https://reflow.equalify.uic.edu/` or `http://localhost:8080/` for local dev. You'll land on the upload page: a header with the Equalify Reflow logo, the five pipeline phases as tabs (greyed out until a document is loaded), and a dashed drop zone in the centre.

## 2. Upload a PDF

Three ways to start:

- **Drag and drop** a PDF file anywhere on the drop zone.
- **Click the drop zone** to open a file picker.
- **Keyboard**: tab to the drop zone, then press **Enter** or **Space** to open the file picker.

The viewer immediately starts processing. You'll see the stage tabs activate one by one as the pipeline advances.

## 3. Watch the pipeline run

The top tab row shows the five public phases: **Extraction → Analysis → Headings → Translation → Assembly**. Each tab lights up as its stage begins and shows its elapsed time when the stage completes.

- An **active spinner** on a tab means the stage is running right now.
- A **green checkmark** means the stage completed successfully.
- An **amber skip icon** means the stage was skipped (for example, OCR when the PDF didn't need it).
- A **red alert icon** means the stage errored non-fatally; the pipeline continues with the best output it had.

By default the viewer **auto-advances** — as each stage finishes, the selected tab jumps to the latest one. The auto-advance toggle in the stats bar lets you pause it if you want to dwell on an earlier stage while the pipeline continues in the background.

Typical end-to-end time for a 6-page PDF on Claude Haiku 4.5: **~3 minutes**, about $0.50 of AI cost. Longer documents scale roughly linearly.

## 4. Navigate the layout

Once the first stage completes, the viewer opens up into its working layout:

![Completed Equalify Reflow run. All five pipeline stage tabs across the top show green checkmarks. A stats bar lists run metrics — page count, average characters per page, figure count, total time, cost — alongside Feedback, Auto-advance, and New PDF buttons. Below, three panels: page thumbnails on the far left, the original PDF page image in the centre-left, and the converted accessible HTML in the centre-right. A Changes panel on the far right summarises how many edits the AI made in the current stage.](https://raw.githubusercontent.com/EqualifyEverything/equalify-docs/main/reflow/assets/reflow-complete.png)

*Header and stage tabs at the top, stats bar below, page thumbnails on the far left, page image and rendered markdown in the middle, changes panel on the right.*

### The page sidebar (left)

Lists every page in the document. Click a page number to jump to it. The active page is highlighted with a blue left border. Only appears for multi-page documents.

### The page image + markdown split (centre)

- **Left panel:** the original PDF page as a rendered image. A **Copy Image** button in the top-right copies the PNG to your clipboard — useful for pasting into review tickets or accessibility audits.
- **Right panel:** the current stage's markdown for this page, rendered as HTML.
- **Drag the vertical divider** between the two panels to resize. Useful when a page is image-heavy or text-heavy.

### The changes panel (right)

Shows how many edits the AI made in the currently selected stage. Click **View Details** to open a full modal with each edit's before, after, and the AI's reasoning.

When you're on the **Analysis** stage, this panel is replaced by a structure metadata view — see [step 6](#6-analysis-stage-the-structure-metadata-panel).

### The stats bar

Top-of-content strip showing the document's key facts:

- **Pages** — total page count
- **Chars/page** — average characters per page (low numbers on a page-heavy document can signal scanned content)
- **"Likely scanned"** — appears when the classifier flags image-only pages
- **Figures** — how many figures Docling extracted
- **Time** — total elapsed pipeline time
- **Cost / tokens** — running LLM cost and token count (only when AI stages have run)

The **Auto-advance** toggle and **New PDF** button live at the right end of the stats bar.

## 5. Inspect the output of each phase

Click any stage tab to see what that phase produced. The markdown panel re-renders with the document as it looked after that stage, and the change count updates to show how many edits happened there.

This is the core value of the viewer: you can see *exactly what each phase changed*. The Extraction stage has no AI edits (it's Docling), but Translation might have dozens. Open the **View Details** modal on a stage to read every edit and its reasoning.

### 6. Analysis stage: the structure metadata panel

When you click the **Analysis** tab, the right-hand changes panel is replaced by a **structure metadata** panel. This shows the document dossier the AI built during analysis:

- **Page Attributes** — layout type (single-column, double-column, presentation, poster), flags for images / tables / lists / equations / scanned pages
- **Outline** — every heading found, with recommended level and page number
- **Code Blocks** — detected code blocks with language + first/last line
- **Footnotes** — every footnote with its marker and body

Click the expand icon to see the full metadata in a modal. This is the context every downstream phase uses to make decisions — reading it tells you why the AI made the calls it did.

## 7. Download what you need

From any stage:

- **Download markdown** (right panel, top-right icon) — the current stage's full-document markdown
- **Download stage version** (stage tab download icon) — the version produced by that specific stage. `v0` is Docling output; later versions are after AI edits
- **Copy markdown** (right panel) — copy the current page's markdown to your clipboard
- **Copy page image** (left panel) — PNG of the current page

For the final accessible output, use the download icon on the **Assembly** tab — that's the fully-processed `v3` markdown.

## 8. Keyboard shortcuts

The viewer is fully keyboard-navigable. Every major region has a direct-jump shortcut so you don't have to tab through a long list of elements:

| Shortcut | Does |
|---|---|
| **⌘ / Ctrl + 0** | Open the skip-navigation menu |
| **⌘ / Ctrl + 1** | Jump to the page sidebar |
| **⌘ / Ctrl + 2** | Jump to the stage tabs |
| **⌘ / Ctrl + 3** | Jump to the rendered preview |
| **⌘ / Ctrl + 4** | Jump to the changes panel |
| **⌘ / Ctrl + /** | Open this shortcut help inside the viewer |

All interactive controls (tabs, page buttons, drop zone, panel buttons) respond to **Enter** and **Space** when focused. The drop zone accepts a drop while focused — you can drag a PDF from Finder/Explorer onto a keyboard-focused drop zone without touching the mouse.

## 9. Warnings banner

If the PDF classifier detects something the pipeline handles less well — a scanned document, very long text blocks, unusual structure — a warnings banner appears above the stats bar describing what it found. The pipeline still runs; the banner is a heads-up for reviewers, not a failure.

## 10. Start over with a new document

**New PDF** in the stats bar (or **⌘ / Ctrl + U** via the skip menu) resets the viewer to the upload screen. Previous processing isn't discarded on the server — if you need to come back to it, the pipeline viewer sessions remain queryable via the API for a retention period. For the viewer UI itself, think of each upload as a one-shot session.

## Tips

- **Scanned documents.** The viewer triggers OCR as a conditional step within Extraction. Expect the Extraction phase to take 1–2 minutes longer on scanned PDFs. Check the stats bar for the "Likely scanned" indicator.
- **Reviewing accessibility.** Always check the Analysis stage's outline against your mental model of the document. If the outline looks wrong, the downstream Headings and Assembly stages are working from a bad map — flag it via [provide feedback](../how-to/provide-feedback).
- **Cost awareness.** The stats bar's cost counter updates live. If the cost is climbing faster than you expected (over roughly $0.10 per page is a heads-up), the document probably triggered extra subagent calls — an image-heavy or table-heavy layout.
- **Skip navigation keyboard.** ⌘/Ctrl+0 is the accessibility entry point. Screen reader users should start there.

## What the viewer doesn't do

The pipeline viewer is a one-shot interactive tool — it's not a document management system. Specifically:

- No persistent library of past conversions (use the WordPress plugin or the API for that)
- No collaboration or review assignments (feedback goes through the [feedback mechanism](../how-to/provide-feedback))
- No batch processing (upload one PDF at a time; for bulk jobs, use [the API](../how-to/integrate-via-api))

For any of those, use the WordPress plugin or integrate against the API directly.

## Where to go next

- [Interpret the output](../how-to/interpret-the-output) — the reviewer's 4-minute quality scan for a converted document
- [Process your first PDF with WordPress](process-your-first-pdf-with-wordpress) — the WP-admin flow that builds on the same pipeline
- [Provide feedback](../how-to/provide-feedback) — submit corrections and issue reports from the built-in viewer
- [How it works](../explanation/how-it-works) — what the pipeline is actually doing during those five phases

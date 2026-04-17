---
title: Interpret the output
date: 2026-04-16
author: Equalify Tech Team
description: A reviewer's checklist for a converted document — what to look at, in what order, and when to ask for a correction.
---

# Interpret the output

You've got a converted document back from Reflow. Before you ship it, walk through this checklist to confirm the pipeline got the important things right.

For the list of what the output contains and which document types convert well, see [supported document types](../reference/supported-document-types.md).

## The 4-minute quality scan

Most problems surface in the first few minutes of review. Work the list in this order — problems higher on the list are more consequential and harder to fix later.

### 1. Structure (30 seconds)

Read the heading outline — most markdown viewers show it as a table of contents, or you can scan the `#`, `##`, `###` lines in the raw markdown.

- **Does the H1 match the document title?** There should be exactly one H1, and it should be the document's actual title — not a section header.
- **Do section H2s match what you'd expect?** Anything that looks like "Introduction", "Methods", "Conclusion" (or the equivalent for your document type) should be H2.
- **Are sub-sections nested sensibly?** No level-skipping (H2 → H4 without an H3 in between).

If the structure is wrong, everything downstream is built on a bad skeleton. File a correction.

### 2. Content accuracy (1–2 minutes)

Sample the text. You don't need to read every word; spot-check:

- **First paragraph of each major section** — OCR errors tend to cluster at page boundaries and unusual layouts.
- **Numbers, dates, proper nouns** — these are the highest-stakes words in most documents. A date off by a day or a misspelled name is worse than a paragraph of awkward phrasing.
- **Footnotes and citations** — check that numbering matches and footnotes appear in the right places.

### 3. Tables (30 seconds per table)

For every table:

- **Header row identified?** The top row should be formatted differently (in markdown tables, the `|---|---|` row signals headers).
- **Cells aligned with the right headers?** Read one data row and confirm each value matches its column header.
- **For complex tables** (merged cells, multi-level headers) — verify manually. The table subagent does its best, but complex layouts can confuse it.

### 4. Images (1 minute for most documents)

Open the image list from the response (the `figures` array) and scan the generated alt text:

- **Is the alt text accurate?** It should describe what the image conveys — not just "an image" or "figure 3".
- **Does it match the caption?** If the original caption said "Figure 2: Enrollment trends 2020–2024", the alt text should describe those trends, not restate the caption.
- **Are decorative images appropriately blank?** Logos, borders, and spacers should have empty alt text.

### 5. Links and emphasis (30 seconds)

- **Hyperlinks are clickable** (not just plain text URLs).
- **Bold/italic** is preserved where it carries meaning (defined terms, emphasis in the source).
- **Code blocks** are formatted as code blocks, not inline text.

## Warnings on the job response

The API response includes a `warnings` array. Treat any warning there as a starting point for review — the pipeline is telling you where it noticed something unusual (scanned pages, failed table reconstructions, confidence below threshold on specific pages).

## When to ask for a correction

Not every issue needs a pipeline change. A rough decision tree:

- **One-off, content-specific issues** (a specific heading got the wrong level, alt text on one image is weak): file a correction via the feedback mechanism. This goes into the aggregated feedback service and helps train future prompt improvements; it doesn't need to go back through the pipeline today.
- **Systematic issues affecting many documents** (every syllabus of a certain format has the same heading mistake): worth filing a formal issue against the pipeline — the fix is usually a prompt tweak, which is easier to validate once and ship broadly.
- **Missing content** (a whole paragraph is gone): rerun the document. This is rare and usually indicates a transient failure rather than a pipeline quality issue.

See [provide feedback](provide-feedback.md) for how to submit a correction through the WordPress plugin.

## If the quality is consistently bad for your document type

Check [supported document types](../reference/supported-document-types.md) — some document types are outside the current scope and are expected to produce lower-quality output. If your document type *is* in scope and quality is still poor, that's a pipeline issue worth flagging so it gets prioritised.

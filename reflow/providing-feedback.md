---
title: Providing Feedback
date: 2026-03-23
author: Equalify Tech Team
description: How to report issues and suggest corrections in converted documents.
---

# Providing Feedback

When you encounter an issue in a converted document, you can report it directly from the viewer. Feedback helps us identify the most impactful problems and prioritize pipeline improvements.

## Reporting an Issue

In the document viewer, click the **Feedback** button to open the feedback panel. You can submit two types of feedback:

### Issue Reports

Describe a problem you've found. Each report includes:

- **Description** — what's wrong (minimum 10 characters)
- **Category** — select the best fit:
  - **Content** — incorrect text, missing content, OCR errors
  - **Formatting** — layout issues, broken tables, misplaced images
  - **Accessibility** — missing alt text, incorrect heading levels, unlabeled elements
  - **Structure** — wrong reading order, misplaced sections, broken lists
- **Page** (optional) — which page the issue appears on
- **Section** (optional) — which section of the document

### Text Corrections

If you find incorrect text, you can propose a specific fix:

1. Select the incorrect text in the viewer
2. Click **Suggest Correction**
3. Enter the corrected text
4. Add a brief explanation of what was wrong

The system records both the original and corrected text, along with the page and section context.

## Where Feedback Goes

Feedback is sent to the [Equalify Reflow Feedback Service](https://github.com/EqualifyEverything/equalify-reflow-feedback), a centralized service that collects reports from all connected viewers — both the WordPress plugin and the standalone viewer.

The feedback service provides:

- **Filtering** — search feedback by document, category, type, source, and date range
- **Aggregated statistics** — totals by category, type, and time window
- **Analytics dashboard** — a Metabase instance for exploring feedback patterns

## How Feedback Improves the System

Feedback directly informs pipeline development:

- **Issue patterns** reveal which pipeline stages need the most work. If "table formatting" is the most-reported category, table handling gets prioritized
- **Text corrections** provide concrete before/after examples that can be used as test cases for regression testing
- **Document-specific reports** identify which document types are most challenging, guiding the selection of pilot documents for testing
- **Volume trends** show whether pipeline updates are reducing the rate of issues over time

## Rate Limits

To prevent abuse, feedback submissions are rate-limited to 5 per IP address per hour. A honeypot field is included to filter automated spam.

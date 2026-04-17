---
title: Providing Feedback
date: 2026-03-23
author: Equalify Tech Team
description: How to report issues and suggest corrections for converted documents.
---

# Providing Feedback

When you encounter an issue in a converted document, feedback helps us identify the most impactful problems and prioritize pipeline improvements.

## How to Report

Feedback is collected through the [Equalify Reflow for WordPress](use-the-wordpress-plugin.md) plugin. When feedback is enabled in the plugin settings, the document viewer includes a feedback interface where users can submit two types of reports:

### Issue Reports

Describe a problem you've found. Each report includes:

- **Description** — what's wrong
- **Category** — select the best fit:
  - **Content** — incorrect text, missing content, OCR errors
  - **Formatting** — layout issues, broken tables, misplaced images
  - **Accessibility** — missing alt text, incorrect heading levels, unlabeled elements
  - **Structure** — wrong reading order, misplaced sections, broken lists
- **Page** (optional) — which page the issue appears on
- **Section** (optional) — which section of the document

### Text Corrections

If you find incorrect text, you can propose a specific fix by providing the original text and your corrected version along with a brief explanation.

## Where Feedback Goes

Feedback is sent to the [Equalify Reflow Feedback Service](https://github.com/EqualifyEverything/equalify-reflow-feedback), a centralized service that collects reports from all connected WordPress sites.

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

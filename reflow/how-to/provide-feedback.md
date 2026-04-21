---
title: Providing Feedback
date: 2026-04-21
author: Equalify Tech Team
description: How to report a problem or suggest a correction from inside the accessible viewer.
---

# Providing Feedback

You're reading a converted document and something looks wrong. Maybe a heading is missing, a table has its columns shuffled, or a paragraph has a small typo. Reflow's feedback tools let you flag it in a few seconds without leaving the page — and what you send goes directly to the people improving the pipeline.

There are two ways to tell us something is off:

- **Report an issue** — describe a problem in your own words and tag it with a category.
- **Suggest a correction** — highlight a specific piece of text and propose a replacement.

Pick the one that matches what you found. If you're not sure, use "Report an issue" — it's the right choice whenever the problem isn't a simple word-for-word fix.

Both options are available from both surfaces where Reflow documents are viewed: the [web app](use-the-web-app.md) at `reflow.equalify.uic.edu` (via the **Feedback** button that appears once a conversion finishes) and the [WordPress plugin](use-the-wordpress-plugin.md)'s document viewer.

> Feedback is only available when the site has enabled it. If you don't see the buttons described below, the document's publisher hasn't turned feedback on, and you'll need to contact them another way.

## Report an issue

Use this when something structural or general is wrong — a missing section, a broken table, unhelpful alt text, the wrong reading order.

1. In the viewer, open the feedback panel (look for a **"Report an issue"** or feedback button near the top of the document).
2. **Describe what's wrong** in plain language. Be specific: "The grading table on page 3 has the columns 'Assignment' and 'Weight' swapped" is much more useful than "Table is broken".
3. **Pick a category** that best matches the problem:
   - **Content** — the text itself is wrong, or something is missing (typos, OCR mistakes, missing paragraphs)
   - **Formatting** — the layout is off (broken tables, images in the wrong place)
   - **Accessibility** — something a screen reader needs is missing or wrong (no alt text, wrong heading levels)
   - **Structure** — the document's organisation is off (wrong reading order, misplaced sections, broken lists)
4. Optionally add the **page** and **section** where you found the problem. This helps the team reproduce it quickly.
5. Click **Submit**.

After you submit, you'll see a short confirmation. You don't need to do anything else — the report is already with the team.

## Suggest a correction

Use this when you can point to specific text that should say something different. Short factual errors, typos, and tightly-scoped phrasing changes are all good candidates.

1. **Highlight the text** in the viewer that's wrong — just select it with your mouse or keyboard the way you would copy text.
2. A **"Suggest a correction"** prompt appears near your selection. Click it.
3. You'll see the **original text** you highlighted. Type your **suggested replacement** underneath.
4. Add a **short reason** if it isn't obvious — "typo", "date should be March 3, not March 13", and so on.
5. Click **Submit**.

The team will see the original and your suggestion side by side, along with where in the document the selection came from.

## What happens after you submit

Every submission goes to a central feedback service where the team can:

- Filter and search feedback by document, category, or date
- Track which categories are most reported — if "Tables" is the top category this month, table handling is the next thing to improve
- Use your text corrections as real-world examples to test pipeline improvements against

You won't usually get a personal reply, but your report is part of how Reflow gets better: systematic patterns in feedback are what drive the next round of pipeline work.

## Under the hood

For developers and integrators: the viewer sends each report to the [Equalify Reflow Feedback Service](https://github.com/EqualifyEverything/equalify-reflow-feedback), a small companion service that collects reports from every site running Reflow, stores them, and exposes a Metabase dashboard for the team to explore patterns. The feedback service is configured per-site in **Settings > Equalify Reflow** in the WordPress plugin, or in the equivalent configuration for the hosted web app.

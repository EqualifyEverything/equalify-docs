---
title: Understanding the Output
date: 2026-03-23
author: Equalify Tech Team
description: What the accessible version of your document includes, how to evaluate quality, and what to look for in review.
---

# Understanding the Output

When Equalify Reflow processes a PDF, it produces an accessible markdown document and a set of extracted images. This guide explains what the output includes, how to evaluate its quality, and what the system's current limitations are.

## What You Get

### Accessible Markdown

The primary output is a single markdown file containing the full document content with:

- **Semantic headings** — a properly nested heading hierarchy (H1 through H6) that reflects the document's logical structure, not just its visual appearance
- **Alt text for images** — descriptive text for informational images, charts, and diagrams. Decorative images (logos, background graphics) are marked as decorative
- **Accessible tables** — tables with header rows identified, preserving the relationship between headers and data cells
- **Reconstructed lists** — bulleted and numbered lists with proper nesting, even when the original PDF layout fragmented them across columns or pages
- **Hyperlinks** — URLs detected in the text are converted to clickable links
- **Clean reading order** — content flows in a logical sequence, free from the column-jumping and page-break artifacts of the PDF layout

### Extracted Figures

Images, charts, diagrams, and photos are extracted from the PDF and saved as separate files. Each figure includes:

- A unique identifier linking it to its position in the markdown
- The page number where it appeared
- The original caption, if one existed in the PDF

Alt text generation for extracted figures is planned but not yet implemented — the `alt_text` field exists in the API response but is currently empty. Alt text generated during page content correction (Stage 4) applies to figures referenced inline in the markdown, not to the extracted image files themselves.

### The Change Ledger

Every edit the pipeline makes is recorded in a change ledger. Each entry includes:

- **Action** — what type of change was made (add, modify, delete)
- **Target** — what was changed (heading, paragraph, table, figure, etc.)
- **Before / After** — the exact text before and after the edit
- **Reasoning** — why the AI made this change

The ledger is available through the API (`GET /api/v1/documents/{job_id}/ledger`) and in the pipeline viewer's Changes panel.

## Evaluating Quality

### What to Check

When reviewing a converted document, focus on these areas:

**Structure**
- Does the heading hierarchy make sense? H1 should be the document title, H2s should be major sections, and so on
- Are sections in the correct order?
- Do lists have the right nesting?

**Content Accuracy**
- Is the text faithful to the original? Look for OCR errors (character substitutions, missing words)
- Are numbers, dates, and proper nouns correct?
- Are footnotes in the right place and correctly numbered?

**Tables**
- Do tables have header rows identified?
- Are cells aligned with the correct headers?
- For complex tables (merged cells, multi-level headers), verify the structure manually

**Images**
- Do informational images have descriptive alt text?
- Is the alt text accurate — does it convey the same information as the image?
- Are decorative images (logos, dividers) appropriately left without alt text?

**Formatting**
- Are hyperlinks clickable (not just plain text URLs)?
- Is emphasis (bold, italic) preserved where it carries meaning?
- Are code blocks properly identified and formatted?

### Quality by Document Type

Some document types convert better than others:

| Document Type | Typical Quality | Common Issues |
|---------------|----------------|---------------|
| Syllabi and course materials | High | Occasional heading level disagreements |
| Policy documents | High | Complex nested numbering schemes |
| Letters and memos | High | Letterhead content may be over-described |
| Academic chapters | Medium | Footnote ordering, reading order in multi-column layouts |
| Presentations (slides) | Medium | Slide boundaries, text embedded in images |
| Infographics and posters | Lower | Spatial relationships lost when linearized |
| Brochures with complex layouts | Lower | Multi-column reading order confusion |

### Performance Benchmarks

From the 30-document pilot (174 total pages):

| Metric | Value |
|--------|-------|
| Average cost per document | $0.45 |
| Average cost per page | $0.08 |
| Average processing time | ~2 min 44 sec |
| Fastest (1-page document) | 15 seconds |
| Slowest (11-page scanned chapter) | 8 min 13 sec |

## Known Limitations

The system is designed for **course materials** — syllabi, academic papers, policy documents, presentations, and similar content. The following document types are outside the current scope and may produce lower-quality results:

- **Scanned multi-column academic chapters** — reading order detection across columns is unreliable for scanned documents
- **Heavy infographics** — spatial relationships (flow diagrams, organizational charts) are lost when linearized to text
- **Mathematical equations** — complex LaTeX formulas are not fully supported
- **Bilingual scanned documents** — OCR quality degrades significantly with mixed-language scanned content
- **Documents over 40 pages** — the system processes them, but quality and cost scale with complexity

When the pipeline detects a document type it handles poorly, it emits **warnings** in the response. These warnings appear in both the API response and the viewer interface.

## Providing Feedback

If you find an issue in a converted document, the viewer includes a feedback interface:

1. **Report an issue** — describe the problem and categorize it (content, formatting, accessibility, structure)
2. **Suggest a correction** — select the incorrect text and propose a replacement

Feedback is collected by the [Equalify Reflow Feedback Service](https://github.com/EqualifyEverything/equalify-reflow-feedback) and used to identify patterns that guide pipeline improvements. The most common feedback categories directly inform which pipeline stages get refined next.

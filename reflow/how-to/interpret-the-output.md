---
title: Interpret the output
date: 2026-04-16
author: Equalify Tech Team
description: A reviewer's checklist for a converted document — what to look at, in what order, and when to ask for a correction.
---

# Interpret the output

You've got a converted document back from Reflow. Before you publish it, take a few minutes to check the pipeline got the important things right.

The converted document is viewable from the [web app](use-the-web-app) at `reflow.equalify.uic.edu`. The questions below apply to all converted output.

For the list of what the output contains and which document types convert well, see [supported document types](../reference/supported-document-types).

## What a good conversion looks like

Imagine you've just opened the accessible viewer for a ten-page syllabus. Here's what you'd see if the conversion went well:

- The course title sits at the top of the page as the clear main heading — not as a line of bold text that merely *looks* like a title. Below it, section names like "Course description", "Schedule", "Grading", and "Policies" form a tidy outline that you can jump between from the table of contents on the side.
- The weekly schedule that was a bulleted list in the PDF is still a bulleted list in the viewer — not a run of paragraphs where each line starts with a dash character.
- A table of assignment weights keeps its rows and columns, with the top row clearly marked as headers. Reading across a row, each percentage sits under the right column.
- The instructor's photo has a short, factual description next to it (something like "Portrait of Professor Ramirez") — a real alt text, not "image" or a filename. A purely decorative border graphic has no description at all, which is correct: a screen reader will skip past it rather than interrupting the reader.
- Links to the course policy page are real, clickable links. Italics on a Latin term or bold on a defined word are preserved where they carry meaning.

A **poor** conversion, by contrast, looks flat: big runs of paragraph text with no outline, lists collapsed into prose, tables whose rows no longer line up with their headers, images labelled "image1.png" or missing descriptions entirely, and sections that end mid-sentence because a page break wasn't stitched back together.

Keep that picture in mind as you work through the questions below.

## The 4-minute quality scan

Most problems surface in the first few minutes of review. Work the list in this order — problems higher on the list are more consequential and harder to fix later.

### 1. Structure (30 seconds)

Look at the table of contents on the side of the viewer (it's built from the document's headings).

- **Does the outline read like a natural summary of the document?** If you read only the headings, do they make sense as a table of contents for this document?
- **Is there exactly one main title at the top, and is it the document's real title** — not a section name?
- **Do the top-level section names match what you'd expect?** For a syllabus you'd expect "Course description", "Schedule", and so on — not the first sentence of the introduction.
- **Do sub-sections nest sensibly under their parents**, without any surprising jumps in level?

If the outline is wrong, everything below it is built on a shaky skeleton. File a correction.

### 2. Content accuracy (1–2 minutes)

Spot-check the text rather than reading every word.

- **Read the first paragraph of each major section.** Does it make sense? Text errors tend to cluster at page boundaries and unusual layouts.
- **Check the numbers, dates, and proper nouns.** These are the highest-stakes words in most documents — a date off by a day or a misspelled name matters more than a paragraph of awkward phrasing.
- **Look at any footnotes or citations.** Do the numbers line up, and do the footnotes appear near where they're referenced?

### 3. Tables (30 seconds per table)

For every table in the document:

- **Is the top row clearly marked as headers** — either visually bold or set apart from the data rows?
- **Pick one data row and read across it.** Does each cell sit under the correct column header?
- **If the table has merged cells or multi-level headers**, check it manually — complex table layouts are where automated conversion is most likely to stumble.

### 4. Images (1 minute for most documents)

Go through the images in the viewer (or open the accompanying figure list).

- **Does the alt text actually describe what the image conveys?** "Bar chart showing enrolment doubled between 2020 and 2024" is useful. "An image" or "figure 3" is not.
- **Does the alt text add to the caption, rather than just repeating it?** If the caption already says "Figure 2: Enrolment trends 2020–2024", the alt text should describe what those trends look like.
- **Are decorative images (logos, borders, spacers) appropriately left without alt text** so screen readers skip past them?

### 5. Links and emphasis (30 seconds)

- **Are hyperlinks clickable**, rather than showing as plain text URLs?
- **Is bold or italic formatting kept** where it carries meaning — defined terms, emphasis the author used deliberately?
- **If the document contained code examples**, are they set apart from the body text rather than flowing inline?

## Warnings from the pipeline

The viewer (or the API response, if you're integrating directly) includes a list of warnings the pipeline produced — places where it noticed something unusual such as a scanned page, a table it wasn't confident about, or an image it couldn't classify. Treat each warning as a starting point for review; it's the pipeline telling you where to look first.

## When to ask for a correction

Not every issue needs a pipeline change. A rough decision tree:

- **One-off issues specific to this document** (a single heading got the wrong level, alt text on one image is weak): submit feedback from the viewer. It feeds into the central feedback service and helps improve future conversions; the document you're looking at today doesn't need to be re-run for this.
- **Systematic issues affecting many documents** (every syllabus in this format has the same heading mistake): worth filing a formal issue against the pipeline — the fix is usually a tweak to the AI's instructions, which can be validated once and rolled out to everyone.
- **Missing content** (a whole paragraph or page is gone): re-run the document. This is rare and usually points to a transient glitch rather than a quality problem.

See [provide feedback](provide-feedback) for how to submit a correction from the viewer.

## If the quality is consistently bad for your document type

Check [supported document types](../reference/supported-document-types) — some document types are outside the current scope and are expected to produce lower-quality output. If your document type *is* in scope and quality is still poor, that's a pipeline issue worth flagging so it gets prioritised.

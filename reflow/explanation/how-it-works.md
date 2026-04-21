---
title: How Equalify Reflow Works
date: 2026-03-23
author: Equalify Tech Team
description: The five-stage pipeline that converts PDFs into accessible, semantic markdown.
---

# How Equalify Reflow Works

## The Thesis

Documents are written in two languages at once. There's the text — the words on the page. And there's the **visual language** — the cues of size, weight, position, proximity, and spacing that tell you what those words *mean* structurally. Biggest text, centred, top of page? Title. Small italic text under an image? Caption. Indented block with a bullet? List item. Sighted readers understand this language fluently without much thought — but a screen reader, or any program that only sees the text, doesn't.

Modern AI models can now read both images and text at the same time. That means they can look at a page the way a sighted reader does and also produce a structured, machine-readable version of it — one where headings are marked as headings, lists as lists, images with proper descriptions. That is exactly the translation accessible documents need.

But having a model that *can* translate isn't the same as having a system that *does* translate reliably. A bilingual dictionary contains real knowledge, but it doesn't make you a translator. Reliable translation requires a process: knowing what you're translating from, what you're translating to, what counts as correct, and how to verify the result. That's what Equalify Reflow is.

## Why Markdown

Instead of trying to "fix" PDFs — a format designed for print, where accessibility is bolted on after the fact — we extract the content and rebuild it in a format that is accessible by design.

That format is **Markdown** — a simple plain-text way of writing documents where a line starting with `#` is a heading, a line starting with `-` is a list item, and so on. It was invented to make writing for the web easier, and it has a few useful properties:

- **Owned by no one** — plain text, no proprietary tools, no vendor lock-in
- **Readable without software** — open it in any text editor and you can still follow the structure
- **Structure is explicit** — headings, lists, tables, links all carry clear structural meaning rather than being faked with formatting
- **Converts cleanly to accessible HTML** — with proper headings, table headers, alt text, and page landmarks
- **Universally understood** — readable by people, by AI models, and by other programs

## The Pipeline

Equalify Reflow is reachable through two user-facing surfaces — the hosted [web app](../how-to/use-the-web-app.md) at `reflow.equalify.uic.edu` for browser-based use, and the [WordPress plugin](../how-to/use-the-wordpress-plugin.md) for sites that manage PDFs from the Media Library — both of which run documents through the same pipeline described below.

Equalify Reflow converts PDFs through a five-stage pipeline:

### Stage 1: Extraction

[IBM Docling](https://github.com/docling-project/docling) handles the first pass. It pulls the text, tables, images, and reading order straight out of the PDF using the structural information already inside the file. This gets us roughly 70% of the way there before any AI is involved, which keeps conversions fast and inexpensive.

If the document is a scan — an image of a page rather than a PDF that already contains selectable text — Docling runs optical character recognition (OCR) to read the text off the image first.

### Stage 2: Analysis

Before the AI processes a page, we need to understand what we're looking at. This stage looks at both the visual presentation and the text pulled from Docling to classify the document — is this a poster, an academic paper, a syllabus, a flyer?

The document type matters because it changes how Reflow talks to the AI in later stages. A two-column academic paper needs different handling than a single-page event poster, so the instructions we send the AI are tailored to match.

This stage also builds a working summary of the document: an outline of its headings and sections, a note on each page's layout and what it contains (images, tables, equations), where footnotes live, and anything else that needs special attention. That summary is carried forward so every later stage has the full context of the document it is working on — not just the single page in front of it.

### Stage 3: Headings

Headings come first because a correct heading outline is the backbone of an accessible document. Screen reader users navigate almost entirely by jumping between headings, so the difference between a real heading and text that merely *looks* big is enormous. Get the outline right and everything else has a skeleton to hang on.

At this stage the AI studies the visual cues — font size, weight, position, spacing — and decides which lines are top-level headings, which are sub-headings, and so on, working across the whole document so the outline stays consistent from start to finish.

### Stage 4: Translation

This is where the core translation happens. For each page, the AI is shown two things side by side: a picture of the page, and the current draft of its text. Its job is to edit the draft so that it faithfully matches what a sighted reader sees on the page.

Every edit the AI makes is recorded along with a short explanation of *why* it made that edit. That gives reviewers a trail they can audit later — see the change ledger below.

For particular tasks that benefit from focused expertise, the main AI hands off to specialists:

- **Image descriptions** — writing alt text, summarising charts, and deciding which images are decorative (and can be skipped by screen readers) versus meaningful (which need a description)
- **Tables** — getting header rows right, matching cells to their column headers, and handling complex layouts like merged cells
- **Lists** — rebuilding nested lists and reconnecting lists that were broken across columns or pages

### Stage 5: Assembly

The final pass joins the individual pages into one continuous document and smooths over the seams. Pages are a print idea — on a screen they get in the way. The AI looks at every page boundary and fixes the usual paper-to-screen problems: a word split across pages, a table or list chopped in half by a page break, a footnote stranded far from the sentence it belongs to.

The result is a single document that flows naturally on any screen size and any device, with its accessibility built in rather than bolted on.

## PII Protection

Before any AI processing happens, every document is scanned for personal information — names, email addresses, phone numbers, Social Security numbers, and so on — using [Microsoft Presidio](https://microsoft.github.io/presidio/). If anything sensitive is detected, the document is held for a human to review before it continues through the pipeline. Reflow is designed for course materials, not student records.

## The Change Ledger

Every edit made by the pipeline is recorded with:

- **What changed** — the text before and after
- **Why** — the AI's own explanation for the edit
- **Where** — which page and which element

The result is a transparent audit trail. In human-review mode, an administrator can inspect every change before the document is finalised.

## Tech Stack

- **[FastAPI](https://fastapi.tiangolo.com/)** — Python async web framework
- **[IBM Docling](https://github.com/docling-project/docling)** — PDF extraction and OCR
- **[Claude](https://www.anthropic.com/claude) via [AWS Bedrock](https://aws.amazon.com/bedrock/)** — multimodal AI processing
- **[PydanticAI](https://ai.pydantic.dev/)** — agent framework with tool-call architecture
- **[Microsoft Presidio](https://microsoft.github.io/presidio/)** — PII detection
- **[Redis](https://redis.io/)** — job queuing, state management, and event streaming
- **[S3](https://aws.amazon.com/s3/)** — document storage with circuit breakers
- **[Docker](https://www.docker.com/)** — containerized development and deployment
- **[Terraform](https://www.terraform.io/)** — AWS infrastructure as code

## Learn More

Reflow is open-source at [EqualifyEverything/equalify-reflow](https://github.com/EqualifyEverything/equalify-reflow) under AGPL-3.0-or-later — clone it to run the pipeline locally with Docker, or read the contributor docs in that repo for implementation detail.

To follow releases and roadmap discussions, visit the [Getting Started](../getting-started.md) guide or [sign up as a partner](https://equalify.uic.edu/signup/reflow).

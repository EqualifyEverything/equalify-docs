---
title: How Equalify Reflow Works
date: 2026-03-23
author: Equalify Tech Team
description: The five-stage pipeline that converts PDFs into accessible, semantic markdown.
---

# How Equalify Reflow Works

## The Thesis

Documents are primarily written in two languages at once. There's the text — the words on the page. And there's the **visual language** — the conventions of size, weight, position, proximity, and spacing that tell you what those words *mean* structurally. Biggest text, centered, top of page? Title. Small italic text under an image? Caption. Indented block with a bullet? List item. This is a language that sighted people understand fluently without much thought.

Multimodal AI models — models that process both images and text — have an understanding of **visual language** and the coding knowledge to express it as **semantic structure**. This makes translating from a visual layout to accessible HTML a natural fit for what these models already do.

But having a model that *can* translate isn't the same as having a system that *does* translate reliably. A bilingual dictionary contains real knowledge, but it doesn't make you a translator. Translation requires architecture: knowing what you're translating from, what you're translating to, what counts as correct, and how to verify the output. That's what Equalify Reflow is.

## Why Markdown

Instead of trying to "fix" PDFs — a format designed for print fidelity where accessibility is bolted on after the fact — we extract the content and rebuild it in a format that is natively accessible.

That format is **Markdown**:

- **Democratic by design** — plain text, no proprietary tooling, owned by no one
- **Human-readable without rendering** — open it in any text editor and understand the structure
- **Semantically rich** — headings, lists, tables, links all have explicit structural meaning
- **Maps directly to HTML** — renders losslessly into accessible HTML with proper heading hierarchy, table headers, alt text, and landmark regions
- **Lingua franca** — readable by humans, AI models, and computer programs alike. LLMs already output markdown by default because it's efficient, structured, and carries meaning

## The Pipeline

Equalify Reflow converts PDFs through a five-stage pipeline:

### Stage 1: Extraction

[IBM Docling](https://github.com/docling-project/docling) handles the first pass. It uses smaller, efficient models and whatever structural data already exists inside the PDF to produce a first-pass markdown version. This handles mechanical parsing — text blocks, tables, images, reading order — without burning expensive LLM calls on mechanical work. Gets you roughly 70% of the way there.

If the document is scanned (image-only), Docling applies OCR to extract the text before proceeding.

### Stage 2: Analysis

Before the AI processes a page, we need to understand what we're looking at. This stage analyzes the PDF's visual presentation alongside the semantic data pulled from Docling to classify the document type — is this a poster, an academic paper, a syllabus, a flyer?

The document type matters because it **dynamically adjusts the prompt** given to the model. A two-column academic paper needs different handling than a single-page event poster.

This stage also produces a structural map of the document: an outline of headings and sections, page-level attributes (layout type, content flags like images, tables, equations), footnote locations, and any elements that need special attention. All of this context is carried forward through the pipeline as a **dossier** that informs every downstream decision.

### Stage 3: Headings

Headings come first because a valid heading hierarchy is the backbone of document accessibility. Get that right and everything else has a skeleton to hang on. The agent infers heading levels from visual signals: font size, weight, position, spacing — and reconciles them into a consistent hierarchy across the entire document.

### Stage 4: Translation

This is where the core translation happens. Each page is given to a multimodal LLM as both an **image** and its **current markdown interpretation**. The model's job is to edit the markdown to make it match what the visual page communicates.

The model works through **tool calls** — each edit includes a reasoning explanation, giving us insight into how the model is interpreting the document. This reasoning trail is recorded in a **change ledger** for auditability.

Some of those tool calls spawn **specialist sub-agents** for tasks that need focused expertise:

- **Alt Text Agent** — image description, chart summarization, decorative vs. meaningful labeling
- **Table Agent** — cell relationships, header associations, complex table structures
- **List Agent** — nested list reconstruction, continuation across visual breaks

### Stage 5: Assembly

The final pass brings all the individual markdown pages together into a single document and removes page boundaries — pages are a print metaphor, and on screens they're an obstacle. An AI examines the boundaries between pages and fixes artifacts from the paged presentation: words split across pages, tables or lists that were broken by a page break, footnotes relocated to their logical position, and other seams left over from the original layout.

The result is a reflowable, responsive document that adapts to any viewport, any device, any rendering context — accessible by construction.

## PII Protection

Before any AI processing occurs, every document is scanned for personally identifiable information using [Microsoft Presidio](https://microsoft.github.io/presidio/). If PII is detected — names, emails, phone numbers, SSNs — the document is held for human review before proceeding. The system is designed for course materials only, not student records.

## The Change Ledger

Every edit made by the pipeline is recorded with:

- **What changed** — the before and after text
- **Why** — the model's reasoning for the edit
- **Where** — the page and target element

This ledger is available for human review, creating a transparent audit trail. In `human` review mode, an administrator can inspect every change before the document is finalized.

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

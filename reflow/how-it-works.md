---
title: How Equalify Reflow Works
date: 2026-03-04
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

### Stage 1: Efficient Initial Extraction

[IBM Docling](https://github.com/docling-project/docling) handles the first pass. It uses smaller, efficient models and whatever structural data already exists inside the PDF to produce a first-pass markdown version. This handles mechanical parsing — text blocks, tables, images, reading order — without burning expensive LLM calls on mechanical work. Gets you roughly 70% of the way there.

### Stage 2: Structure Analysis

Before the AI processes a page, we need to understand what we're looking at. This stage analyzes the PDF's visual presentation alongside the semantic data pulled from Docling to classify the document type — is this a poster, an academic paper, a syllabus, a flyer?

The document type matters because it **dynamically adjusts the prompt** given to the model. A two-column academic paper needs different handling than a single-page event poster.

This stage also flags elements that need special attention: double-column layouts, images, tables, complex list structures. All of this context is carried forward through the pipeline to steer the model toward more optimal outcomes at each subsequent stage.

### Stage 3: Headings First

Headings come first because a valid heading hierarchy is the backbone of document accessibility. Get that right and everything else has a skeleton to hang on. The agent infers heading levels from visual signals: font size, weight, position, spacing.

### Stage 4: The Translator

This is where the core translation happens. Each page is given to a multimodal LLM as both an **image** and its **current markdown interpretation**. The model's job is to edit the markdown to make it match what the visual page communicates.

The model works through **tool calls** — each edit includes a reasoning explanation, giving us insight into how the model is interpreting the document. This reasoning trail is valuable for identifying patterns and improving the pipeline over time.

Some of those tool calls spawn **specialist sub-agents** for tasks that need focused expertise:

- **Alt Text Agent** — image description, chart summarization, decorative vs. meaningful labeling
- **Table Agent** — cell relationships, header associations, complex table structures

### Stage 5: Escape the Page

The final pass brings all the individual markdown pages together into a single document and removes page boundaries — pages are a print metaphor, and on screens they're an obstacle. An AI examines the boundaries between pages and fixes artifacts from the paged presentation: words split across pages, tables or lists that were broken by a page break, and other seams left over from the original layout.

The result is a reflowable, responsive document that adapts to any viewport, any device, any rendering context — accessible by construction.

## Tech Stack

- **[FastAPI](https://fastapi.tiangolo.com/)** — Python async web framework
- **[IBM Docling](https://github.com/docling-project/docling)** — PDF extraction
- **[Claude](https://www.anthropic.com/claude) via [AWS Bedrock](https://aws.amazon.com/bedrock/)** — multimodal AI processing
- **[PydanticAI](https://ai.pydantic.dev/)** — agent framework
- **[Microsoft Presidio](https://microsoft.github.io/presidio/)** — PII detection
- **[Redis](https://redis.io/)** — job queuing and state management
- **[S3](https://aws.amazon.com/s3/)** — document storage with circuit breakers
- **[Docker](https://www.docker.com/)** — local development and deployment
- **[Terraform](https://www.terraform.io/)** — AWS infrastructure as code

## Learn More

The source code is not yet publicly available. We are currently in the UIC pilot phase and plan to open-source the full project under the AGPL license.

To learn more or request early access, visit the [Getting Started](#/docs/getting-started) guide or [sign up as a partner](#/partner).
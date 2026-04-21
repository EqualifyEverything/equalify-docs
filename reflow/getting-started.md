---
title: Getting Started
date: 2026-04-21
author: Equalify Tech Team
description: Choose how to use Equalify Reflow and how to request partner access.
---

# Getting Started

Equalify Reflow turns PDFs into accessible web documents. This page helps you pick the right way to use it and, if your organisation hasn't onboarded yet, how to request access.

## Choose how to use Reflow

There are three ways to use Reflow. Pick the one that matches how you work — you can always start with one and add another later.

### Web app — for one-off conversions in the browser

The hosted web app at **[reflow.equalify.uic.edu](https://reflow.equalify.uic.edu)** is the quickest path. Open the URL, drag a PDF into the browser, and watch the conversion happen live. When it's done, read it in the built-in viewer and download the accessible markdown. No sign-in, no API key, no installation — the instance is open and protected by per-IP rate limits.

Best for: individual users who need to convert a document now, accessibility reviewers evaluating Reflow's output, or anyone who doesn't run a WordPress site.

Start here: [process your first PDF with the web app](tutorials/process-your-first-pdf-with-the-web-app.md) · [use the web app](how-to/use-the-web-app.md)

### WordPress plugin — for publishing accessible PDFs on a WordPress site

The Equalify Reflow plugin connects your WordPress site to a Reflow instance. Administrators process PDFs directly from the Media Library; visitors read them through an accessible viewer with a table of contents, full-text search, and a download link for the original PDF.

Best for: organisations that already publish PDFs through a WordPress site and want those PDFs to have an accessible counterpart without changing their publishing workflow.

Start here: [process your first PDF with WordPress](tutorials/process-your-first-pdf-with-wordpress.md) · [use the WordPress plugin](how-to/use-the-wordpress-plugin.md)

### API integration — for your own application or workflow

Reflow has an HTTP API that accepts a PDF and returns the accessible version, with live progress updates along the way. If you have your own content system, a batch pipeline, or a custom viewer, you can build directly against it.

Best for: developers integrating Reflow into a CMS other than WordPress, building automation around document intake, or embedding the output in a bespoke viewer.

Start here: [integrate via the API](how-to/integrate-via-api.md) · [API reference](reference/api.md)

## For partners

The hosted web app is open to anyone. Becoming a partner is for organisations that want a dedicated API key (to integrate via the API or the WordPress plugin), early access to pipeline improvements, and a voice in what gets built next. Partners get:

- **A dedicated API URL and key** for the WordPress plugin or a custom API integration
- **Early access** to new features and pipeline improvements
- **Roadmap input** — influence what gets prioritised
- **Direct support** during pilot deployments

You don't need to be an engineer to contribute. We need accessibility experts who can evaluate outputs, institutions willing to test the pipeline on real document collections, and practitioners who understand day-to-day remediation work and can help prioritise what matters most.

### How to request access

1. [Sign up on the partner form](https://equalify.uic.edu/signup/reflow), or email [Blake Bertuccelli-Booth](https://it.uic.edu/profiles/blake-bertuccelli-booth/) (b3b@uic.edu) directly.
2. Tell us a little about your organisation, the kinds of documents you'd like to convert, and which of the three paths above matches how you want to use Reflow.
3. The Equalify team will be in touch to discuss next steps and, if things move forward, issue your API credentials.

Between signing up and hearing back, you can use the public web app at [reflow.equalify.uic.edu](https://reflow.equalify.uic.edu) for evaluation — it runs through the same pipeline partners use.

## For developers curious about the internals

Reflow is open source at [EqualifyEverything/equalify-reflow](https://github.com/EqualifyEverything/equalify-reflow) under AGPL-3.0-or-later. The README has setup instructions for running the stack locally with Docker; the `docs/` directory has contributor-focused tutorials, how-to guides, reference pages, and explanation docs.

## Want to follow along?

To follow releases and roadmap discussions, [sign up as a partner](https://equalify.uic.edu/signup/reflow).

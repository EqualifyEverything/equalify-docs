---
title: Getting Started
date: 2026-03-23
author: Equalify Tech Team
description: How to get involved with Equalify Reflow as a partner, WordPress user, or developer.
---

# Getting Started

## For Partners

We're looking for institutional partners who want early access and a voice in what gets built next. Partners get:

- **Early access** to new features and pipeline improvements
- **Roadmap commenting** — influence what gets prioritized
- **Direct support** during pilot deployments

You don't need to be an engineer to contribute. We need accessibility experts who can evaluate outputs, institutions willing to test the pipeline on real document collections, and practitioners who understand day-to-day remediation to help prioritize what matters most.

[Sign up for partner access](https://equalify.uic.edu/signup/reflow) or contact [Blake Bertuccelli-Booth](https://it.uic.edu/profiles/blake-bertuccelli-booth/) (b3b@uic.edu) directly.

## For WordPress Users

The [Equalify Reflow for WordPress](wordpress-plugin.md) plugin connects your WordPress site to the Reflow API. From the Media Library, select any PDF and click **Run Equalify Reflow** — the plugin handles submission, tracks progress in real time, and stores the accessible version alongside the original.

Processed documents are served through a built-in viewer with:

- Auto-generated table of contents
- Full-text search with result highlighting
- Download options for the original PDF and accessible markdown
- Optional feedback collection for reporting issues

See the [WordPress Plugin Guide](wordpress-plugin.md) for installation and configuration.

## For Developers

Equalify Reflow is open source under the [AGPL-3.0 license](https://www.gnu.org/licenses/agpl-3.0.en.html).

### Quick Start

```bash
# Clone the repository
git clone https://github.com/EqualifyEverything/equalify-reflow.git
cd equalify-reflow

# Start all services (Docker required)
make dev

# API is now available at http://localhost:8080
# Interactive docs at http://localhost:8080/docs
# Pipeline viewer at http://localhost:8080/viewer
```

The `make dev` command starts the full stack: API server, Redis, LocalStack (S3 emulation), Docling extraction service, and monitoring (Prometheus, Grafana, Jaeger).

### Submit Your First Document

```bash
curl -X POST http://localhost:8080/api/v1/documents/submit \
  -H "X-API-Key: YOUR_API_KEY" \
  -F "file=@document.pdf"
```

The API returns a `job_id` you can use to check status or stream progress. See the [API Reference](api-reference.md) for the full endpoint documentation.

### What You'll Need

- **Docker** and **Docker Compose** — all services run in containers
- **AWS credentials** (optional) — for AI processing via Bedrock. Without them, LocalStack provides S3 emulation but AI features require real AWS access
- **make** — the Makefile wraps all common operations

See the [Contributing Guide](contributing.md) for the full development workflow, testing strategy, and how to add pipeline stages.

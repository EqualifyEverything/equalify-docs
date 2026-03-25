---
title: Contributing
date: 2026-03-23
author: Equalify Tech Team
description: Development setup, testing strategy, project structure, and how to contribute to Equalify Reflow.
---

# Contributing

Equalify Reflow is open source under the [AGPL-3.0 license](https://www.gnu.org/licenses/agpl-3.0.en.html). We welcome contributions — bug fixes, pipeline improvements, documentation, and new integrations.

## Prerequisites

- **Docker** and **Docker Compose** — all services run in containers
- **make** — wraps all common operations
- **AWS credentials** (optional) — needed for AI processing via Bedrock. Without them, you can still run the infrastructure and tests that mock LLM calls

## Development Setup

```bash
# Clone the repo
git clone https://github.com/EqualifyEverything/equalify-reflow.git
cd equalify-reflow

# Copy the example environment file
cp .env.example .env

# Start all services
make dev
```

This starts:

| Service | Port | Description |
|---------|------|-------------|
| API Gateway | `localhost:8080` | FastAPI app with hot reload |
| Redis | `localhost:6379` | Job state, queues, pub/sub |
| LocalStack | `localhost:4566` | S3 emulation |
| Docling Serve | `localhost:5001` | PDF extraction sidecar |
| Prometheus | `localhost:9090` | Metrics |
| Grafana | `localhost:3001` | Dashboards |
| Jaeger | `localhost:16686` | Tracing |

Code is volume-mounted from `src/` into the container. Edits on your host trigger automatic reload — no rebuild needed.

### Useful Commands

```bash
make dev              # Start all services
make down             # Stop all services
make logs             # View all logs
make logs-api         # View API logs only
make shell            # Shell into the API container
make redis-cli        # Connect to Redis CLI
make health           # Verify infrastructure health
```

**Important:** Do not run `python`, `pytest`, or `uv` directly on your host. Everything runs inside Docker:

```bash
make test-fast        # Run unit tests (~30s)
make test-integration # Run integration tests (~2min)
make test-e2e         # Run end-to-end tests (~5min)
make coverage         # Tests with coverage report
```

## Project Structure

```
src/
├── main.py                    # FastAPI app entry point + worker startup
├── config.py                  # Settings from environment variables
├── dependencies.py            # Dependency injection (S3, Redis, services)
├── api/                       # REST endpoints
│   ├── documents.py           # Document submission and status
│   ├── pipeline_viewer.py     # Pipeline viewer
│   └── approval.py            # PII approval workflow
├── workers/                   # Background task processors
│   ├── pii_worker.py          # PII detection queue consumer
│   └── timeout_worker.py      # Approval timeout checks
├── services/                  # Business logic
│   ├── pipeline_viewer.py     # 5-stage pipeline orchestration
│   ├── document_processing.py # Job lifecycle management
│   ├── storage.py             # S3 with circuit breakers
│   ├── queue.py               # Redis queue operations
│   ├── job.py                 # Job state (Lua scripts)
│   └── pii_detection.py       # Presidio integration
├── agents/                    # AI pipeline
│   ├── orchestrator.py        # Pipeline orchestration + dossier
│   ├── dossier.py             # Document context model
│   ├── shared_prompts.py      # Reusable prompt fragments
│   ├── model_tiers.py         # Model selection (Sonnet/Haiku)
│   ├── worker.py              # Per-page content correction agent
│   ├── paragraph_agent.py     # Sub-agent orchestration
│   ├── recovery.py            # Error recovery agent
│   ├── critic.py              # Verification agent
│   ├── document_worker.py     # Cross-page assembly agent
│   └── prompts/               # Stage-specific prompt modules
│       ├── structure_analysis.py
│       ├── heading_reconciliation.py
│       ├── boundary_fix.py
│       ├── footnote_relocation.py
│       └── revision.py
├── middleware/                 # HTTP middleware
│   ├── auth.py                # API key + docs auth
│   ├── rate_limit.py          # Per-IP rate limiting
│   └── metrics.py             # Prometheus instrumentation
├── shared/                    # Constants and shared models
└── utils/                     # Helpers (retry, circuit breaker, tokens)
```

## Testing

The project uses a three-tier testing strategy:

### Unit Tests (`make test-fast`)

Fast, isolated tests that mock external dependencies (S3, Redis, LLM calls). Run these before every commit.

```bash
make test-fast
```

Key patterns:
- **PydanticAI agents** — mock `_get_*_agent` or `_get_*_subagent` factories to avoid real LLM calls
- **Circuit breakers** — use `reset_llm_circuit_breaker()` in `autouse=True` fixtures to prevent state leakage between tests
- **Conditional tools** — test `prepare` functions return `ToolDefinition` or `None` based on task type

### Integration Tests (`make test-integration`)

Tests that exercise real service interactions (Redis, S3 via LocalStack) but still mock LLM calls. Run before PRs.

```bash
make test-integration
```

### End-to-End Tests (`make test-e2e`)

Full pipeline tests with real documents. Requires AWS credentials for Bedrock. Run before merges.

```bash
make test-e2e
```

### Test Markers

Tests are tagged with pytest markers:

```python
@pytest.mark.unit          # Unit test (mocked dependencies)
@pytest.mark.integration   # Needs Redis + S3
@pytest.mark.e2e           # Full pipeline, real LLM calls
@pytest.mark.slow          # Takes >10 seconds
```

## Development Workflow

1. **Create a feature branch** from `main`
2. **Start services** with `make dev`
3. **Edit code** in `src/` — changes auto-reload in the container
4. **Run unit tests** with `make test-fast` for quick feedback
5. **Test manually** via the viewer at `http://localhost:8080/viewer` or the API at `http://localhost:8080/docs`
6. **Run integration tests** with `make test-integration` before opening a PR
7. **Open a pull request** against `main`

## Adding a Pipeline Stage

Pipeline stages are defined in `src/services/pipeline_viewer.py`. Each stage:

1. Receives the current markdown and document context (the dossier)
2. Makes modifications
3. Returns an updated version

To add a new stage:

1. Create a prompt module in `src/agents/prompts/` defining the agent's instructions
2. Add the step method to `PipelineViewerService` in `src/services/pipeline_viewer.py`
3. Wire it into the pipeline sequence in the `process()` method
4. Add a version bump if the stage produces a new document version
5. Write unit tests mocking the LLM calls
6. Update the viewer stage groupings in `clients/viewer/src/components/pipeline-viewer/StageTabs.tsx`

## Code Style

- **Python 3.11+** with type hints
- **Pydantic** models for all data structures
- **Async/await** throughout — the FastAPI app is fully asynchronous
- **No direct host execution** — all code runs in Docker containers

## Getting Help

- Open an issue on [GitHub](https://github.com/EqualifyEverything/equalify-reflow/issues)
- For partner support, contact [Blake Bertuccelli-Booth](mailto:b3b@uic.edu)

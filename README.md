# Startup Hunter System

A multi-agent research system that turns an investment thesis into a prioritised list of startups, with analyses grounded in public sources and versioned evaluation criteria.

## How it works

1. Discovery turns the thesis into searches across public sources, then normalises, deduplicates and filters the results.
2. Each startup is researched and passed through three parallel analyses.
3. A synthesiser brings the analyses together against the evaluation frameworks in `knowledge/`.
4. A reporting agent ranks the results and produces a final report.

## Runtime, harness and tools

- **Runtime:** Python and Google ADK run the agent pipeline, with Gemini models served through Vertex AI. The app can run locally through ADK's web interface or as an authenticated Cloud Run service. The deployment enables Cloud Trace and can persist sessions in Agent Engine.
- **Harness:** an ADK `SequentialAgent` coordinates discovery, per-company analysis and reporting. For each company, research feeds a `ParallelAgent` with business model, metrics and market analysts; a synthesiser combines their outputs. Typed state and per-candidate checkpoints connect the stages. Failed research produces an insufficient-data result instead of a diagnosis, and failures are isolated per company.
- **Tools:** the discovery agent calls `find_candidates`, which queries Google Search grounding via Vertex AI, Spanish startup sources via Firecrawl, the Y Combinator directory and GitHub concurrently, then deduplicates by domain. The research agent uses `fetch_url` and Firecrawl to extract website content. Evaluation frameworks are loaded from `knowledge/` into the synthesiser's context, without a vector database.
- **Cost controls:** candidate limits, timeouts and quotas bound the work; retrieved page text is capped before it reaches the model.

## Run locally

Requires Python 3.11 to 3.13, [uv](https://docs.astral.sh/uv/) and Google Cloud Application Default Credentials for live model calls. Copy `.env.example` to `.env` and configure your Google Cloud project and search credentials.

```bash
uv sync --all-groups
uv run python -m pytest -q
uv run python -m google.adk.cli web agents
```

Unit tests do not require credentials or external calls. Separate evaluations are available in `evals/`. Live research and evaluations use Vertex AI and configured search providers, so they can incur costs and produce different results over time.

## Deployment

`scripts/deploy.ps1` runs preflight checks and tests, then deploys to Cloud Run with authenticated access. It stores the Firecrawl API key in Secret Manager and supports optional session persistence through Agent Engine. See the script for the required environment variables.

The deployment is a single-user demo, not a multi-tenant application. Do not grant access to users who should not share application state.

More details are available in the [project documentation](docs/README.md), currently in Spanish.

## Related project

[Startup Hunter for Tales Venture](https://github.com/brunodpl/startup-hunter-talesventure) addresses evidence-based company research with a different architecture: a deterministic Python pipeline and a separately installed Hermes agent, focused on finding clients for a growth service rather than investment targets.

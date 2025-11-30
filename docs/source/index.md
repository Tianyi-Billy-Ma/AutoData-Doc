# Welcome to AutoData

**AutoData** is a multi-agent system that automates how you study a target domain, plan a crawler, generate Python code, and validate the collected dataset. The public release lives at [AutoData](https://github.com/Tianyi-Billy-Ma/AutoData) while these docs mirror the development branch.

```{note}
Need the CLI switches or configuration schema? Jump to {doc}`getting_started` and {doc}`configuration` for the most common workflows.
```

## What AutoData Solves

Modern websites require research, tooling, scripted browsing, and a validation loop before data is trustworthy. AutoData turns that process into a repeatable workflow:

- **Automated research** – the Research Squad (Plan, Tool, Browser, Blueprint agents) explores APIs and webpages, accumulating facts in OHCache.
- **Code generation** – the Development Squad (Engineer, Test, Validation agents) turns blueprints into runnable crawlers and executes them inside the sandboxed work directory.
- **Supervisor loop** – the Supervisor agent arbitrates hand-offs, injects HumanAgent confirmations when `disable_human` is false, and stops when your success criteria are satisfied.
- **Provenance artifacts** – every run writes configs, summaries, browser captures, and checkpoints under `outputs/<run_name>/` so you can re-run or audit later.

## Architecture in Brief

| Layer | Responsibilities |
| --- | --- |
| **Supervisor Agent** | Owns the AutoData graph, routes work, enforces task deadlines, and decides when to finish. |
| **Research Squad** | `PlanAgent` drafts strategies, `ToolAgent` calls external APIs, `BrowserAgent` performs browser-use sessions, `BlueprintAgent` distills findings into implementation-ready blueprints, and `HumanAgent` can be left enabled for manual approvals. |
| **Development Squad** | `EngineerAgent` writes Python crawlers, `TestAgent` executes them (uv/pytest-style) and inspects logs, and `ValidationAgent` checks data quality plus schema expectations. |
| **Shared substrate** | OHCache hypergraph for context routing, LangGraph for execution, CheckpointManager for persistence, and PluginSpec for optional domain tuning. |

See {doc}`configuration` for all tunable knobs exposed by `AutoDataConfig`.

## Execution Lifecycle

1. **Initialize** – `uv run python -m autodata.main --config configs/default.yaml` loads the YAML/TOML/JSON config, merges CLI overrides (model, task, log level, etc.), and materializes `outputs/<run_name>/`.
2. **Build graph** – `AutoData.build()` constructs the LangGraph with every agent plus optional plugins. If `checkpoint_config.resume_from` is set, state is hydrated before execution.
3. **Research loop** – Supervisor dispatches to Research Squad members. OHCache routes only the relevant conversations via hyperedges you can predefine in `ohcache_config.hyperedges`.
4. **Development loop** – Blueprint instructions trigger Engineer/Test/Validation. Tool execution happens in `work/` while artifacts land in `results/`.
5. **Finalize** – On success AutoData writes `summary.json`, optional checkpoints, browser recordings, cached artifacts, and log files. Clean exits keep directories intact; enabling `auto_checkpoint` persists snapshots during the run.

## Key Innovations

- **OHCache (Oriented Hypergraph Cache)** keeps token budgets predictable by caching artifacts and routing messages by type, not history length.
- **Checkpoint CLI** (`python -m autodata.checkpoint ...`) lets you list, clean, or resume runs without touching the full pipeline.
- **Plugin surface** allows prompt injections and additional LangChain tools per domain (financial, sport, academic, etc.).
- **uv-first toolchain** guarantees reproducible environments (see {doc}`getting_started`).

## When to Use AutoData

Use AutoData whenever you must collect structured data from the open web with auditability:
- Need autonomous research before writing a crawler.
- Want generated code you can inspect, test, or adapt.
- Require reproducible outputs, cached context, and resumable checkpoints.
- Prefer orchestration that plays nicely with OpenAI, Anthropic, Google, or an OpenRouter-compatible provider without code changes.

```{toctree}
:maxdepth: 2
:caption: GETTING STARTED

getting_started
```

```{toctree}
:maxdepth: 2
:caption: CONFIGURATION

configuration
```

```{toctree}
:maxdepth: 2
:caption: REFERENCES

references
```

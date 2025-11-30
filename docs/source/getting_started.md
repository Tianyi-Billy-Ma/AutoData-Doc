Follow this guide when you first pull the repository or whenever you spin up a fresh environment.

# 1. Prerequisites

- macOS / Linux shell with **Python 3.11+** (AutoData ships type hints that assume 3.11).
- [uv](https://github.com/astral-sh/uv) for dependency and virtualenv management.
- Playwright system dependencies (Chromium download + lib dependencies).
- API keys for at least one LLM provider (OpenAI, Anthropic, Google, or OpenRouter).

```{tip}
`uv` installs project dependencies into `.venv/`. Always run CLI commands through `uv run ...` so the lockfile is honored.
```

# 2. Clone & Install

```bash
# Clone either the public repo or the dev fork
git clone https://github.com/Tianyi-Billy-Ma/AutoData.git
cd AutoData

# Sync dependencies for development, testing, and docs
uv sync --group dev,test,docs

# Download browser binaries and system deps for browser-use
playwright install
playwright install-deps  # Linux containers only
```

# 3. Provide Credentials

1. Copy the example environment file and edit it with your keys.
   ```bash
   cp .env.example .env
   # fill in OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.
   ```
2. Export any additional API keys required by plugins or tools (e.g., `PPLX_API_KEY` for the Perplexity search tool, `TIINGO_API_KEY` for the financial plugin).
3. When using OpenRouter (or any OpenAI-compatible proxy), set:
   ```bash
   export OPENROUTER_API_KEY="..."
   export OPENROUTER_BASE_URL="https://openrouter.ai/api/v1"
   ```
   AutoData automatically picks up these variables if `llm_config.api_key`/`base_url` are left `null`.

# 4. Run Your First Task

```bash
# Validate configuration without running agents
uv run python -m autodata.main --config configs/default.yaml --dry-run

# Kick off a real task (overrides run name at the CLI)
uv run python -m autodata.main \
  --config configs/default.yaml \
  --run-name aapl-daily-2024 \
  --task "Collect daily AAPL candles for 2024"
```

Common CLI flags:
- `--visualize-graph` saves a Mermaid rendering of the LangGraph graph.
- `--execution-strategy` chooses between synchronous (`stream`, `run`) and async (`astream`, `arun`) execution APIs.
- `--disable-human` skips interactive HumanAgent confirmations.
- `--overwrite` / `--force-overwrite` let you reuse the same `run_name` safely.

# 5. Inspect the Outputs

Every run writes to `outputs/<run_name>/` (auto-generated if you omit `--run-name`). Key folders:

| Path | Contents |
| --- | --- |
| `config.yaml` | Frozen configuration with merged CLI overrides so you can reproduce the run. |
| `summary.json` | High-level metadata, tool outputs, and dataset pointers. |
| `artifacts/` under `results/` | Generated Python files, research documents, CSV/JSON dumps, or ZIP archives returned by agents. |
| `work/` | Temporary working directory for the Python REPL tool and engineer/test agents. Safe to clean between runs. |
| `logs/` | Structured logs at the level defined by `log_config.log_level`. |
| `cache/` | OHCache artifacts + metadata files. Useful for debugging context routing. |
| `browser/` | Playwright/browser-use state, screenshots, and optional video recordings. |
| `checkpoint/` | Serialized checkpoints when `checkpoint_config.enabled` and/or `auto_checkpoint` are true. |

# 6. Resume or Clean Runs

- List checkpoints: `uv run python -m autodata.checkpoint --run-name aapl-daily-2024 list --json`
- Save ad-hoc checkpoint: `uv run python -m autodata.checkpoint save --name after-blueprint --stage=research`
- Resume a run: add `checkpoint_config.resume_from: "checkpoint/<file>.bin"` to your config or pass `--checkpoint.resume_from=<file>` on the CLI.
- Clean up old checkpoints: `uv run python -m autodata.checkpoint clean --max-keep=5 --older-than-days=7`

# 7. Helpful Developer Commands

| Goal | Command |
| --- | --- |
| Format & lint | `uv run ruff format && uv run ruff check .` |
| Run tests | `uv run pytest` or target files such as `uv run pytest tests/workflows/test_ci_lockfile.py` |
| Debug a single agent | `uv run python -m dev.testing.debug --agent=EngineerAgent --prompt="Generate pytest for requests client"` |
| Inspect parity harness | `uv run python -m tests.workflows.agent_parity_harness` |

```{warning}
Runs can make network calls through browser-use and tool APIs. Ensure you comply with each site's terms of use when providing tasks.
```

You are ready to tailor the configuration. Continue to {doc}`configuration` for the full schema and environment variable reference.

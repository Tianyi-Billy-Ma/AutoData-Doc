AutoData is driven by a single `AutoDataConfig` object that can be loaded from YAML, TOML, or JSON. This page describes the important sections, how CLI overrides behave, and which environment variables you should set beforehand.

```{tip}
All configuration files live under `configs/`. Copy `configs/default.yaml`, rename it, and commit the copy for task-specific presets.
```

# LLM Provider Setup

AutoData supports multiple LLM backends through LangChain's `init_chat_model`. Keep `llm_config.api_key` and `llm_config.base_url` as `null` to let AutoData infer settings from environment variables, or hardcode them per run.

## OpenAI

```bash
export OPENAI_API_KEY="your-openai-key"
```

## Anthropic

```bash
export ANTHROPIC_API_KEY="your-anthropic-key"
```

## Google

```bash
export GOOGLE_API_KEY="your-google-key"
```

## OpenRouter (or any OpenAI-compatible proxy)

```bash
export OPENROUTER_API_KEY="your-openrouter-key"
export OPENROUTER_BASE_URL="https://openrouter.ai/api/v1"
```

When these variables are present, AutoData auto-populates `llm_config.api_key`/`base_url`. This applies to other OpenAI-compatible services as well.

# File Formats & Loading

```bash
# Pick the config you want to run
uv run python -m autodata.main --config configs/finance.yaml

# Override detection if the extension is missing
uv run python -m autodata.main --config configs/generated --config-format yaml
```

Internally `AutoDataConfig.from_file()` parses the file, validates every field via Pydantic, and expands relative paths to absolute ones. CLI arguments then override the loaded structure via `apply_cli_overrides`. The priority order is: **CLI > config file > defaults**.

# Core Fields

| Field | Description |
| --- | --- |
| `run_name` | Logical name for the run. Used for folder names and checkpoint manifests. Required when reusing directories with `--overwrite`. |
| `task` | Free-form instruction that the Supervisor reads when executing the graph. CLI flag `--task` temporarily overrides it. |
| `task_timeout` | Max run duration in seconds (default `3600`). |
| `disable_human` | When `true`, AutoData auto-confirms HumanAgent prompts and remains non-interactive. |
| `execution_strategy` | Choose `stream`, `run`, `astream`, or `arun` to control synchronous vs async LangGraph execution. |
| `enabled_plugins` / `plugin_config` | List of plugin modules under `autodata.plugins` the graph should import (`financial`, `sport`, `academic`, …). Each plugin can inject prompts or LangChain tools. |

# Storage & Logging

```yaml
storage_config:
  type: "file"
  output_dir: "./outputs"
  run_name: null
  overwrite: false
  force_overwrite: false
  compression: null
```

- `output_dir` defines where AutoData writes runs. Paths are resolved relative to the repo unless you specify an absolute path.
- Enable `overwrite` if you want to reuse the same `run_name`. Adding `force_overwrite` skips the safety prompt entirely.
- `run_dir`, `cache_dir`, `work_dir`, and `video_dir` are derived from `run_name`; you rarely need to set them manually.

Logging is configured through `log_config`:

```yaml
log_config:
  log_level: "INFO"   # override with --log-level
  log_file: null       # optional path relative to run_dir/logs
  metrics_port: 9090   # reserved for Prometheus exporters
```

# Language Model Settings

The `llm_config` object mirrors LangChain's `init_chat_model` arguments. Example:

```yaml
llm_config:
  model: "gpt-4o-mini"
  model_provider: null      # auto-inferred when omitted
  temperature: 0.0
  base_url: null            # point to OpenRouter or self-hosted proxy
  api_key: null             # auto-detect OPENAI_/ANTHROPIC_/OPENROUTER_... variables
  configurable_fields: null # e.g. "any" or ["temperature", "model"]
```

If you want to override the base URL or key inside the configuration file, set `llm_config.api_key`/`base_url` explicitly. Otherwise leave them `null` and rely on exported environment variables.

# Tooling & Plugins

```yaml
tool_config:
  work_dir: null                # falls back to outputs/<run_name>/work
  cache_dir: null               # inherits from config.cache_dir
  PerplexitySearchToolModel: "sonar"
```

- Set `PPLX_API_KEY` to let `ToolAgent` call the Perplexity search tool.
- When plugins are listed in `enabled_plugins`, AutoData loads their `PluginSpec` definitions, applies prompt injections per agent, and binds additional LangChain tools.

# OHCache Hypergraph

```yaml
ohcache_config:
  enable_ohcache: true
  cache_dir: null        # defaults to outputs/<run_name>/cache
  auto_cleanup: false
  hyperedges:
    - id: research
      source: [PlanAgent]
      target: [SupervisorAgent, HumanAgent, BrowserAgent]
      message_type: "plan"
```

- Hyperedges define which agents share messages. Sources are singleton sets (an agent emits a message) and targets can include any number of recipients.
- Message types (`message_type`) act like channels—Blueprint updates can be isolated from Browser observations.
- When `enable_ohcache` is false the system reverts to naïve context passing. Leave it enabled for best token usage.
- Cached artifacts persist on disk per entry (`cache/meta/*.json` + `cache/artifacts/*`). Set `auto_cleanup: true` to prune expired entries on startup.

# Checkpoints

```yaml
checkpoint_config:
  enabled: true
  auto_checkpoint: true      # save snapshots automatically after agents run
  checkpoint_dir: null       # defaults to outputs/<run_name>/checkpoint
  export_json: false         # when true, writes human-readable JSON next to binaries
  resume_from: null
  max_checkpoints: null
```

You can override any of these via CLI using dot notation, e.g. `--checkpoint.resume_from=checkpoint/manual.bin`. Use the dedicated CLI for inspection:

```bash
uv run python -m autodata.checkpoint list --run-name aapl-run
uv run python -m autodata.checkpoint load checkpoint/manual.bin --json
```

# Browser & Agent Controls

AutoData splits browser-use settings into two models. You can provide a legacy flat `browser_config` block (as shown in `configs/default.yaml`) or the explicit nested structure:

```yaml
browser_use_browser_config:
  headless: true
  disable_security: false
  user_agent: null
  args: []               # Chromium arguments
  record_video_dir: null # leave null to write under outputs/<run>/browser

browser_use_agent_config:
  max_steps: 100
  max_actions_per_step: 10
  llm_timeout: 120
  generate_gif: false
  file_system_path: null
```

`setup_output_directory()` ensures `browser_use_agent_config.file_system_path` points to `outputs/<run_name>/browser` so browser-use can persist histories between agent calls.

# CLI Override Reference

| Flag | Effect |
| --- | --- |
| `--config-path PATH` | Load an alternative config file (default `configs/default.yaml`). |
| `--config-format {yaml,json,toml}` | Force parsing format if the extension is missing. |
| `--run-name NAME` | Override `run_name`/`storage_config.run_name`. Validated for alphanumeric/`-_`. |
| `--task "..."` | Override the task description for this invocation only. |
| `--model`, `--temperature` | Override LLM settings. |
| `--output-dir PATH` | Redirect the entire run folder (useful for shared storage). |
| `--log-level LEVEL` | Set logging verbosity without editing the config file. |
| `--visualize-graph` | Persist graph diagrams under the run directory. |
| `--dry-run` | Stop after validation and directory creation. |
| `--disable-human` | Mirrors the config flag for a non-interactive run. |
| `--overwrite`, `--force-overwrite` | Control existing run directory handling. |
| `--checkpoint.*` | Use dot-notation to override any checkpoint setting from the CLI. |

Always keep your configuration files in version control when possible so the generated `summary.json` plus `config.yaml` in `outputs/<run_name>/` allow you to reconstruct a run precisely.

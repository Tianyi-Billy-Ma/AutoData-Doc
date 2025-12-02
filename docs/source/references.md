# Citation

If you use AutoData in research, please cite:

```bibtex
@inproceedings{autodata2025,
  title={AutoData: A Multi-Agent System for Open Web Data Collection},
  author={Ma, Tianyi and Qian, Yiyue and Zhang, Zheyuan and Wang, Zehong and Qian, Xiaoye and Bai, Feifan and Ding, Yifan and Luo, Xuwei and Zhang, Shinan and Murugesan, Keerthiram and others},
  booktitle={NeurIPS},
  year={2025}
}
```

# Agent Roster

| Agent | Responsibility |
| --- | --- |
| `Supervisor` | Owns the LangGraph, routes between squads, and halts when the task is satisfied. |
| `PlanAgent` | Drafts collection strategies, defines subtasks, and seeds OHCache with plan messages. |
| `ToolAgent` | Executes LangChain tools (e.g., Perplexity search) plus plugin-specified tools. |
| `BrowserAgent` | Operates the `browser-use` automation stack to browse, click, and scrape pages. |
| `BlueprintAgent` | Consolidates research output into executable Python + testing guidance. |
| `EngineerAgent` | Writes crawler code, typically in the run-scoped `work/` directory. |
| `TestAgent` | Runs the generated code (pytest, uv run, etc.) and reports errors/logs. |
| `ValidationAgent` | Validates dataset schema, does QA, and reports pass/fail signals. |
| `HumanAgent` | Optional manual approval gate; disabled automatically with `--disable-human`. |

# Directory Layout

```
AutoData/
├── autodata/           # source package containing agents, core, tools, plugins
├── configs/            # YAML/TOML/JSON presets for AutoDataConfig
├── docs/, openspec/, evaluation/, tests/
└── outputs/<run_name>/ # generated per run (config, summary, artifacts, cache, checkpoints)
```

File/folder highlights under each run:

- `summary.json` – metadata about the plan, code artifacts, validation verdicts.
- `results/` – packaged datasets, scripts, markdown reports, zipped deliverables.
- `browser/` – browser-use recordings/screenshots (`record_video_dir` when enabled).
- `logs/` – run-specific logs, useful for debugging agent loops.
- `cache/` – OHCache artifact store (`meta/*.json` + `artifacts/*`).
- `checkpoint/` – serialized state snapshots, loadable via `python -m autodata.checkpoint`.

# Command Quick Reference

| Command | Purpose |
| --- | --- |
| `uv run python -m autodata.main --config <file>` | Primary entry point; respects CLI overrides and writes outputs. |
| `uv run python -m autodata.checkpoint list|save|load|clean` | Inspect or manage checkpoints without kicking off a full task. |
| `uv run python -m dev.testing.debug --agent=PlanAgent --prompt="..."` | Exercise an individual agent with a custom prompt. |
| `uv run ruff format && uv run ruff check .` | Apply formatting and lint rules required by CI. |
| `uv run pytest` | Execute the unit/integration test suite (see `README.md` for targeted workflows). |
| `playwright install && playwright install-deps` | Ensure browser-use has the Chromium binaries it needs. |

# Environment Variables

| Variable | Usage |
| --- | --- |
| `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY` | Consumed automatically by LangChain's `init_chat_model`. |
| `OPENROUTER_API_KEY`, `OPENROUTER_BASE_URL` | Enables third-party OpenAI-compatible endpoints. |
| `PPLX_API_KEY` | Unlocks the Perplexity search tool (used by `ToolAgent`). |
| Domain-specific API keys | e.g., `TIINGO_API_KEY`, `SPORTSDATA_API_KEY` when enabling plugins such as `financial` or `sport`. |

(references-supported-arguments)=
# Supported Arguments

AutoData groups configuration into dataclasses (such as `task_config` or `storage_config`). Unless otherwise noted, each field below can be supplied either in your config file or on the CLI with the auto-generated flag `--<field-name>` (underscores become hyphens). When a field is marked *config only*, prefer defining it in YAML/TOML/JSON for readability.

## Task Configuration (`task_config`)

| Field | Default | CLI Flag(s) | Description |
| --- | --- | --- | --- |
| `config` | `configs/default.yaml` | `--config`, `--config-path`, `-c` | Location of the configuration file to load. |
| `config_format` | `null` | `--config-format` | Explicit config format (`yaml`, `json`, `toml`) when AutoData cannot infer it. |
| `task` | `""` | `--task` | Natural-language instruction executed by the Supervisor. |
| `run_name` | `null` | `--run-name` | Logical name used to derive output folders and checkpoints. |
| `disable_human` | `False` | `--disable-human` | Auto-approve HumanAgent prompts for unattended runs. |
| `task_timeout` | `3600` | `--task-timeout` | Maximum runtime in seconds before the graph aborts. |
| `execution_strategy` | `"stream"` | `--execution-strategy` | Execution API: `stream`, `run`, `astream`, or `arun`. |
| `dry_run` | `False` | `--dry-run` | Validate configuration and exit without building the graph. |
| `verbose` | `False` | `--verbose` | Emit additional initialization logs. |
| `visualize_graph` | `False` | `--visualize-graph` | Persist the LangGraph diagram to disk. |

## Storage Configuration (`storage_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `type` | `"file"` | `--type` | Storage backend (`file`, future database adapters). |
| `output_dir` | `"./outputs"` | `--output-dir` | Root directory that holds every run. |
| `file_format` | `"json"` | `--file-format` | Serialization format for summary files. |
| `compression` | `null` | `--compression` | Compression codec (`gzip`, `bz2`, `lzma`). |
| `database_url` | `null` | `--database-url` | Connection string if writing to a database backend. |
| `overwrite` | `True` | `--overwrite / --no-overwrite` | Allow reusing an existing run directory. |
| `force_overwrite` | `True` | `--force-overwrite / --no-force-overwrite` | Skip the confirmation prompt when overwrite is enabled. |

## Logging (`log_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `metrics_enabled` | `True` | `--metrics-enabled / --no-metrics-enabled` | Enable Prometheus metrics server. |
| `metrics_port` | `9090` | `--metrics-port` | Port exposed by the metrics endpoint. |
| `log_level` | `"INFO"` | `--log-level` | Logging verbosity. |
| `log_file` | `null` | `--log-file` | Optional log file path relative to the run directory. |

## Language Model (`llm_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `model` | `"gpt-4o"` | `--model` | Chat model identifier. |
| `model_provider` | `null` | `--model-provider` | Explicit provider name when inference cannot deduce it. |
| `temperature` | `0.0` | `--temperature` | Sampling temperature. |
| `base_url` | `null` | `--base-url` | Custom OpenAI-compatible endpoint (e.g., OpenRouter). |
| `api_key` | `null` | `--api-key` | Override API key instead of relying on environment variables. |
| `configurable_fields` | `null` | `--configurable-fields` | Runtime-editable LLM fields (`"any"` or comma-separated list). |

## Tool Configuration (`tool_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `run_dir` | `null` | `--run-dir` | Override the directory exposed to tool processes. |
| `work_dir` | `null` | `--work-dir` | Scratch directory for engineers/tests (defaults to `outputs/<run>/work`). |
| `tools_cache_dir` | `null` | `--tools-cache-dir` | Persistent cache for tool downloads. |
| `PerplexitySearchToolModel` | `"sonar"` | `--perplexity-search-tool-model` | Model slug passed to the Perplexity API. |

## OHCache (`ohcache_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `enable_ohcache` | `False` | `--enable-ohcache / --no-enable-ohcache` | Toggle the OHCache hypergraph + caching layer. |
| `cache_dir` | `null` | `--cache-dir` | Directory where cache metadata and artifacts live. |
| `auto_cleanup` | `False` | `--auto-cleanup / --no-auto-cleanup` | Delete expired cache entries on startup. |
| `hyperedges` | `[]` | *config only* | Define template hyperedges (YAML/TOML keeps the structure readable). |

## Checkpoints (`checkpoint_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `checkpoint_enabled` | `False` | `--checkpoint-enabled / --no-checkpoint-enabled` | Master switch for checkpoint support. |
| `auto_checkpoint` | `False` | `--auto-checkpoint / --no-auto-checkpoint` | Save checkpoints automatically between agents. |
| `checkpoint_dir` | `null` | `--checkpoint-dir` | Custom directory for checkpoint binaries. |
| `export_json` | `False` | `--export-json / --no-export-json` | Emit human-readable JSON next to binaries. |
| `resume_from` | `null` | `--resume-from` | Path to the checkpoint to restore before execution. |
| `max_checkpoints` | `null` | `--max-checkpoints` | Retention limit for automatic checkpoint pruning. |

## Plugins (`plugin_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `enabled_plugins` | `[]` | `--enabled-plugins` | List of plugin identifiers (e.g., `financial`, `sport`). |

## Browser Settings (`browser_use_browser_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `headless` | `True` | `--headless / --no-headless` | Run browser automation without a visible window. |
| `disable_security` | `False` | `--disable-security / --no-disable-security` | Relax browser security features (use cautiously). |
| `user_agent` | `null` | `--user-agent` | Custom user agent string. |
| `args` | `null` | `--args` | Extra Chromium launch flags (comma-separated). |
| `record_video_dir` | `null` | `--record-video-dir` | Directory for browser-use session recordings. |

## Browser Agent (`browser_use_agent_config`)

| Field | Default | CLI Flag | Description |
| --- | --- | --- | --- |
| `max_steps` | `20` | `--max-steps` | Maximum browser agent steps. |
| `max_actions_per_step` | `50` | `--max-actions-per-step` | Cap on actions executed within a single step. |
| `llm_timeout` | `null` | `--llm-timeout` | Timeout in seconds for LLM calls during browser control. |
| `generate_gif` | `null` | `--generate-gif` | Enable GIF generation (path or `true`). |
| `file_system_path` | `null` | `--file-system-path` | Custom filesystem root for browser-use session artifacts. |

# Special Thanks

- [Browser-use](https://github.com/browser-use/browser-use) – browser automation foundation.
- [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules.git), [spec-kit](https://github.com/github/spec-kit.git), [Cursor](https://cursor.com), [Codex](https://openai.com/codex/), [Claude Code](https://www.claude.com/product/claude-code) – tooling inspiration.

# Citation

If you use AutoData in research, please cite:

```bibtex
@inproceedings{autodata2025,
  title        = {AutoData: A Multi-Agent System for Open Web Data Collection},
  author       = {Tianyi-Billy-Ma and Contributors},
  booktitle    = {NeurIPS},
  year         = {2025},
  url          = {https://arxiv.org/abs/2505.15859}
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

# Special Thanks

- [Browser-use](https://github.com/browser-use/browser-use) – browser automation foundation.
- [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules.git), [spec-kit](https://github.com/github/spec-kit.git), [Cursor](https://cursor.com), [Codex](https://openai.com/codex/), [Claude Code](https://www.claude.com/product/claude-code) – tooling inspiration.

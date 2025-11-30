# Project Setup and Configuration

AutoData uses a flexible configuration system. You can set up your environment variables and YAML configs as follows.

## LLM Provider Setup

Set your API keys in your environment:

```bash
# Standard Providers
export OPENAI_API_KEY="your-openai-key"
export ANTHROPIC_API_KEY="your-anthropic-key"
export GOOGLE_API_KEY="your-google-key"

# OR for OpenRouter
export OPENROUTER_API_KEY="your-openrouter-key"
export OPENROUTER_BASE_URL="https://openrouter.ai/api/v1"
```

## Configuration

Configure your model in `configs/default.yaml`:

```yaml
llm_config:
  model: "gpt-4o-mini"
  temperature: 0.0
```

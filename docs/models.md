# Model Selection and Custom Providers

Codex supports a wide range of AI models, from OpenAI's latest models to local open-source models running on your own machine. This guide explains how to select models and configure custom model providers.

## Quick Start

### Using OpenAI Models (Default)

By default, Codex uses OpenAI's models. Simply run:

```bash
codex
```

And sign in with your ChatGPT account or API key.

### Using Local Models with LMStudio

If you have [LMStudio](https://lmstudio.ai/) running with models loaded:

```bash
codex --oss
```

This will automatically connect to your local LMStudio instance (default port: 1234).

### Using Ollama

If you prefer [Ollama](https://ollama.ai/):

```bash
codex --oss --local-provider ollama
```

This connects to your local Ollama instance (default port: 11434).

### Selecting a Specific Model

You can specify which model to use with the `-m` or `--model` flag:

```bash
codex -m gpt-5.1
codex --oss -m llama3.3:70b
```

## Built-in Model Providers

Codex comes with three built-in model providers:

1. **OpenAI** (`openai`) - Default provider using OpenAI's API
2. **LMStudio** (`lmstudio`) - Local models via LMStudio
3. **Ollama** (`ollama`) - Local models via Ollama

## Configuring Custom Model Providers

You can add your own model providers by editing `~/.codex/config.toml`. This allows you to use any OpenAI-compatible API endpoint.

### Basic Configuration

Add a `[model_providers.<name>]` section to your config:

```toml
[model_providers.my-provider]
name = "My Custom Provider"
base_url = "https://api.example.com/v1"
env_key = "MY_API_KEY"
```

Then use it:

```bash
export MY_API_KEY="your-api-key-here"
codex -c model_provider=my-provider
```

### Configuration Options

Each model provider can be configured with these options:

#### Required Fields

- **`name`** - Friendly display name for the provider
- **`base_url`** - Base URL for the OpenAI-compatible API endpoint

#### Authentication

- **`env_key`** - Environment variable containing the API key
  ```toml
  env_key = "MY_API_KEY"
  ```

- **`env_key_instructions`** - Help text shown to users if the API key is missing
  ```toml
  env_key_instructions = "Get your API key from https://example.com/api-keys"
  ```

- **`experimental_bearer_token`** - Hardcoded API key (not recommended for security)
  ```toml
  experimental_bearer_token = "sk-..."
  ```

- **`requires_openai_auth`** - Set to `true` if this provider requires OpenAI authentication
  ```toml
  requires_openai_auth = false
  ```

#### Wire Protocol

- **`wire_api`** - API protocol to use (default: `"chat"`)
  - `"responses"` - Modern OpenAI Responses API (used by OpenAI, LMStudio)
  - `"chat"` - Classic Chat Completions API (used by most providers)
  
  ```toml
  wire_api = "chat"  # or "responses"
  ```

#### HTTP Configuration

- **`query_params`** - Additional query parameters for requests
  ```toml
  [model_providers.azure.query_params]
  api-version = "2025-04-01-preview"
  ```

- **`http_headers`** - Static HTTP headers to include
  ```toml
  [model_providers.my-provider.http_headers]
  X-Custom-Header = "custom-value"
  ```

- **`env_http_headers`** - HTTP headers from environment variables
  ```toml
  [model_providers.my-provider.env_http_headers]
  X-Custom-Header = "MY_HEADER_ENV_VAR"
  ```

#### Retry and Timeout Settings

- **`request_max_retries`** - Max HTTP request retries (default: 4)
- **`stream_max_retries`** - Max SSE stream reconnection attempts (default: 5)
- **`stream_idle_timeout_ms`** - Stream idle timeout in milliseconds (default: 300000 = 5 minutes)

```toml
request_max_retries = 4
stream_max_retries = 10
stream_idle_timeout_ms = 300000
```

## Common Provider Examples

### Azure OpenAI

```toml
[model_providers.azure]
name = "Azure OpenAI"
base_url = "https://your-resource.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"
wire_api = "chat"

[model_providers.azure.query_params]
api-version = "2025-04-01-preview"
```

Usage:
```bash
export AZURE_OPENAI_API_KEY="your-key"
codex -c model_provider=azure -m gpt-4
```

### Custom LMStudio Port

If your LMStudio runs on a different port:

```toml
[model_providers.lmstudio]
name = "LMStudio"
base_url = "http://localhost:8080/v1"
wire_api = "responses"
requires_openai_auth = false
```

Or use environment variables:
```bash
export CODEX_OSS_PORT=8080
codex --oss
```

### Custom Ollama Configuration

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
wire_api = "chat"
requires_openai_auth = false
```

### OpenRouter

```toml
[model_providers.openrouter]
name = "OpenRouter"
base_url = "https://openrouter.ai/api/v1"
env_key = "OPENROUTER_API_KEY"
wire_api = "chat"

[model_providers.openrouter.http_headers]
HTTP-Referer = "https://codex.local"
X-Title = "Codex CLI"
```

Usage:
```bash
export OPENROUTER_API_KEY="sk-or-..."
codex -c model_provider=openrouter -m anthropic/claude-3.5-sonnet
```

### Together.ai

```toml
[model_providers.together]
name = "Together AI"
base_url = "https://api.together.xyz/v1"
env_key = "TOGETHER_API_KEY"
wire_api = "chat"
```

Usage:
```bash
export TOGETHER_API_KEY="your-key"
codex -c model_provider=together -m meta-llama/Llama-3.3-70B-Instruct
```

### Groq

```toml
[model_providers.groq]
name = "Groq"
base_url = "https://api.groq.com/openai/v1"
env_key = "GROQ_API_KEY"
wire_api = "chat"
```

Usage:
```bash
export GROQ_API_KEY="gsk_..."
codex -c model_provider=groq -m llama-3.3-70b-versatile
```

## Using Configuration Profiles

Instead of passing `-c model_provider=<name>` every time, you can create configuration profiles:

```toml
[profiles.local]
model_provider = "lmstudio"
model = "llama-3.3-70b-instruct"

[profiles.groq]
model_provider = "groq"
model = "llama-3.3-70b-versatile"
approval_policy = "never"  # Auto-approve for faster iteration
```

Then use:
```bash
codex -p local    # Use LMStudio
codex -p groq     # Use Groq
```

## Local Model Best Practices

### LMStudio Setup

1. Install LMStudio from [lmstudio.ai](https://lmstudio.ai/)
2. Download your preferred models (e.g., Llama 3.3 70B, DeepSeek R1)
3. Start the local server: `lms server start`
4. Run Codex: `codex --oss`

Codex will automatically:
- Verify LMStudio is running
- Check if the model is downloaded
- Download the model if needed
- Load the model in the background

### Model Selection with LMStudio

When using `--oss`, Codex defaults to `openai/gpt-oss-20b`. You can override this:

```bash
codex --oss -m deepseek/r1-distill-qwen-32b
codex --oss -m meta-llama/llama-3.3-70b-instruct
```

### Ollama Setup

1. Install Ollama from [ollama.ai](https://ollama.ai/)
2. Pull models: `ollama pull llama3.3:70b`
3. Run Codex: `codex --oss --local-provider ollama -m llama3.3:70b`

## Environment Variables

You can customize provider behavior with environment variables:

### OpenAI
- `OPENAI_BASE_URL` - Override OpenAI API endpoint
- `OPENAI_ORGANIZATION` - OpenAI organization ID
- `OPENAI_PROJECT` - OpenAI project ID

### Local (OSS) Providers
- `CODEX_OSS_BASE_URL` - Override local provider URL
- `CODEX_OSS_PORT` - Override local provider port (default: 1234 for LMStudio, 11434 for Ollama)

Examples:
```bash
# Use OpenAI proxy
export OPENAI_BASE_URL="https://my-proxy.example.com/v1"
codex

# Use LMStudio on custom port
export CODEX_OSS_PORT=8080
codex --oss

# Use completely custom local endpoint
export CODEX_OSS_BASE_URL="http://my-server:9000/v1"
codex --oss
```

## Troubleshooting

### "LM Studio is not responding"

Make sure LMStudio is running:
```bash
lms server start
```

Or check if it's accessible:
```bash
curl http://localhost:1234/v1/models
```

### "Model provider not found"

Check your config:
```bash
cat ~/.codex/config.toml
```

Make sure the provider name matches what you specified with `-c model_provider=<name>`.

### API Key Issues

If you get authentication errors, verify your environment variable is set:
```bash
echo $MY_API_KEY
```

And that the `env_key` in your config matches the environment variable name.

### Wire API Mismatch

If you get errors about request format, try changing the `wire_api`:
- Most third-party providers use `wire_api = "chat"`
- OpenAI and LMStudio support `wire_api = "responses"`

## See Also

- [Configuration Reference](https://developers.openai.com/codex/config-reference)
- [Configuration Basics](./config.md)
- [Getting Started](./getting-started.md)

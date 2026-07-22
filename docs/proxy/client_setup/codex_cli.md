---
title: Codex (CLI)
sidebar_label: Codex (CLI)
---

import Image from '@theme/IdealImage';

# Connect Codex CLI to LiteLLM

[Codex](https://github.com/openai/codex) reads all of its configuration from `~/.codex/config.toml`. You define LiteLLM as a custom model provider there, and register the [MCP gateway](../../mcp.md) in the same file. This is the same config the [Codex surface inside ChatGPT Desktop](./codex_chatgpt_desktop.md) uses, so setting it up once covers both.

## Quick reference

| Setting | Value |
|---|---|
| Config file | `~/.codex/config.toml` |
| `base_url` | `<LITELLM_PROXY_BASE_URL>/v1` (e.g. `http://localhost:4000/v1`) |
| Provider key | Your LiteLLM [virtual key](../virtual_keys.md), via an env var |
| MCP endpoint | `<LITELLM_PROXY_BASE_URL>/<server_name>/mcp` |

## LLM setup

### 1. Install Codex

```bash
npm i -g @openai/codex
```

### 2. Define LiteLLM as a model provider

Codex uses the OpenAI Responses API, which LiteLLM serves at `/v1/responses`. Add a provider block to `~/.codex/config.toml` and select it as the default. The `env_key` names the environment variable Codex reads your virtual key from, so no secret is stored in the file:

```toml title="~/.codex/config.toml"
model = "claude-sonnet-5"
model_provider = "litellm"

[model_providers.litellm]
name = "LiteLLM"
base_url = "http://localhost:4000/v1"
env_key = "LITELLM_API_KEY"
wire_api = "responses"
```

Export the key, then run Codex:

```bash
export LITELLM_API_KEY="sk-1234"

codex
```

`model` can be any public model name from your LiteLLM config. Override it per run with `codex --model gemini-2.5-pro`.

Codex reports `provider: litellm` on startup and routes the task through the gateway:

<Image img={require('../../../img/client_setup/codex_cli_llm.png')} />

### 3. Verify

Ask Codex to make a small change, then check the Admin UI under **Logs** or **Usage**; the request should be attributed to your virtual key and the model you selected.

## MCP setup

Register the LiteLLM MCP gateway as a server in the same `~/.codex/config.toml`. Recent Codex versions support remote streamable-HTTP MCP servers natively:

```toml title="~/.codex/config.toml"
[mcp_servers.litellm]
url = "http://localhost:4000/my_mcp_server/mcp"
bearer_token = "sk-1234"
```

`my_mcp_server` must match a key under `mcp_servers:` in your gateway config. The `bearer_token` is your virtual key; Codex sends it as `Authorization: Bearer`, which the gateway accepts for authentication.

If the server doesn't show up, you're likely on an older Codex build that ignores remote MCP servers unless the experimental flag is set. Add it above the server entry, or upgrade Codex:

```toml title="~/.codex/config.toml"
[features]
experimental_use_rmcp_client = true
```

For a LiteLLM server that fronts an upstream OAuth provider, run `codex mcp login litellm` to complete the flow instead of setting a static token; see [MCP OAuth](../../mcp_oauth.md).

## Next steps

- [Codex in ChatGPT Desktop](./codex_chatgpt_desktop.md) uses this same config file
- [LiteLLM virtual keys](../virtual_keys.md)
- [MCP gateway reference](../../mcp.md)

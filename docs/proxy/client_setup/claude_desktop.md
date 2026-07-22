---
title: Claude Desktop (GUI)
sidebar_label: Claude Desktop (GUI)
---

# Connect Claude Desktop to LiteLLM

[Claude Desktop](https://claude.ai/download) can route its model traffic through LiteLLM using third-party inference, and reach your [MCP gateway](../../mcp.md) tools through a local bridge. Everything is configured from the app.

## Quick reference

| Setting | Value |
|---|---|
| Gateway URL | `<LITELLM_PROXY_BASE_URL>` (e.g. `http://localhost:4000`) |
| API Key | Your LiteLLM [virtual key](../virtual_keys.md) |
| MCP endpoint | `<LITELLM_PROXY_BASE_URL>/<server_name>/mcp` |
| MCP auth header | `x-litellm-api-key: Bearer <virtual key>` |

## LLM setup

### 1. Enable Developer Mode

In Claude Desktop, go to **Help -> Claude -> Help** and click **Enable Developer Mode**.

![](https://colony-recorder.s3.amazonaws.com/files/2026-04-22/64274593-33e6-4a7b-a7f3-a08f8aea8209/ascreenshot_8a9c909a978544888dafb6e0c7e3f468_text_export.jpeg)

### 2. Open Configure Third-Party Inference

Open the Claude menu from the menu bar icon, click **Developer**, then **Configure Third-Party Inference...**

![](https://colony-recorder.s3.amazonaws.com/files/2026-04-22/2fcad657-4f8c-4dc2-b9ff-597de4e98030/ascreenshot_241063b192ae4c75996aaefdab991f13_text_export.jpeg)

![](https://colony-recorder.s3.amazonaws.com/files/2026-04-22/dbb36dff-bbbe-4ddd-b30e-25b2c41bff47/ascreenshot_a7516b203052432f9a1d08cbe92cd214_text_export.jpeg)

### 3. Enter your gateway URL and virtual key

In the inference settings dialog, put your LiteLLM proxy URL in **Gateway URL** and your virtual key in **API Key**, then save.

![](https://colony-recorder.s3.amazonaws.com/files/2026-04-22/2d0daa12-d874-42ca-bc3e-f38c27c701e4/ascreenshot_8c8be28828974c10ab53124fa13e67c3_text_export.jpeg)

Create the virtual key from the Admin UI under **Virtual Keys -> + Create New Key** if you don't have one.

![](https://colony-recorder.s3.amazonaws.com/files/2026-04-22/6a5b1233-de81-48be-8a17-e026d3dd9b49/ascreenshot_23dbd432db6d4f90ab5b0d598edd5a40_text_export.jpeg)

### 4. Verify

Restart Claude Desktop, open a new conversation, and send a message. Confirm the request appears in the Admin UI under **Usage**, attributed to your virtual key.

![](https://colony-recorder.s3.amazonaws.com/files/2026-04-22/9e72faf1-0b5e-49d5-8ac4-b64dcd2b2f94/ascreenshot_813a1b584a1f4523ab7f7702f5985be0_text_export.jpeg)

## MCP setup

Claude Desktop's native **Custom Connectors** (Settings -> Connectors) only support OAuth or authless remote servers, not custom headers. Because LiteLLM authenticates with the `x-litellm-api-key` header, connect through the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge, which Claude Desktop runs locally and which forwards your header to the gateway.

Open **Settings -> Developer -> Edit Config** (this opens `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS) and add:

```json title="claude_desktop_config.json"
{
  "mcpServers": {
    "litellm-tools": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote@latest",
        "https://your-litellm-proxy.com/my_mcp_server/mcp",
        "--header",
        "x-litellm-api-key:Bearer sk-1234"
      ]
    }
  }
}
```

`my_mcp_server` must match a key under `mcp_servers:` in your gateway config. Save the file and restart Claude Desktop; the server's tools appear in the tools menu.

:::info

Claude Desktop can block `localhost` MCP URLs. For anything beyond a quick local test, front your gateway with an HTTPS URL (a tunnel such as ngrok or Cloudflare Tunnel works). When LiteLLM fronts an upstream server that uses OAuth, add it through the native Custom Connectors UI instead and let LiteLLM handle the flow; see [MCP OAuth](../../mcp_oauth.md).

:::

## Next steps

- [LiteLLM virtual keys](../virtual_keys.md)
- [MCP gateway reference](../../mcp.md)

# Prompt Studio MCP

Connect [Prompt Studio](https://promptstudio.app) to Cursor, Grok Bot, ChatGPT, Claude, and Grok (xAI). After you sign in, two phrases are enough:

- **“Save this to Prompt Studio”** or **“save it to Prompt Studio”** stores the phrase and attaches outputs in your library.
- **“Use prompt X from Prompt Studio”** or **“fetch my … prompt from Prompt Studio”** loads that saved phrase so the chat you are in can follow it.

The server is already hosted. This repository is the public install package: a plugin manifest, remote MCP config, and skills. It does not contain a local stdio server, API keys, or a service role key.

| Field | Value |
| --- | --- |
| MCP URL | `https://mcp.promptstudio.app/mcp` |
| Product docs | [promptstudio.app/mcp](https://promptstudio.app/mcp) |
| Publisher | Quick One B.V. (Prompt Studio) |
| Auth | OAuth. The client opens a browser sign-in. |
| Listing | **Not listed yet.** Cursor Marketplace and Grok Bot plugin search show Prompt Studio only after a person submits this repo and review finishes. |

Image generation is a queue. The Worker does not run models. `generate_image` is available on Pro when the account has image credits.

## Install

Sign in with the Prompt Studio account that owns the library. There is nothing to paste from a dashboard: no API key, no client secret, no service role.

### Cursor

**After this plugin is listed** (it is not listed today): open **Customize** in the sidebar, search for **Prompt Studio**, choose **Install**, and finish the browser sign-in.

**Until then**, add the hosted server yourself. Project config is `.cursor/mcp.json`. Global config is `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "prompt-studio": {
      "url": "https://mcp.promptstudio.app/mcp"
    }
  }
}
```

Reload Cursor, then start the OAuth sign-in from the MCP panel. You can also open this install link:

[Add Prompt Studio to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=prompt-studio&config=eyJwcm9tcHQtc3R1ZGlvIjp7InVybCI6Imh0dHBzOi8vbWNwLnByb21wdHN0dWRpby5hcHAvbWNwIn19)

To try the plugin package locally before it is listed, copy this repo into `~/.cursor/plugins/local/prompt-studio` (the folder needs either the root `plugin.json` or `.cursor-plugin/plugin.json`), then run **Developer: Reload Window** and check **Customize**. A marketplace install with the same name wins over the local copy.

Publishers submit the repo at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish). See [docs/PUBLISH.md](docs/PUBLISH.md).

### Grok Bot

Grok Bot has no settings form for a server URL. In the chat, say:

> Add this MCP server: https://mcp.promptstudio.app/mcp

Confirm the card, then finish **Authorize** in the browser. If it stays on “Waiting for authorization”, use **Reopen**. The tools are there on the next message. The bot runs in the cloud, so use this public HTTPS endpoint.

**Once the plugin is listed** (it is not listed today), install the same package from **Plugins** in the sidebar (in the mobile app: avatar, then **Plugins**), then authorize. On a Cursor team, an admin can hide marketplace plugins from Grok Bot. Searching before submit will not find this package.

### ChatGPT

ChatGPT talks to remote MCP servers through a custom connector. Developer mode has to be on, and a workspace admin may need to allow custom connectors first (**Workspace settings → Permissions & roles**).

1. Open **Settings → Connectors**.
2. Open **Advanced settings** and turn on **Developer mode**.
3. Create a connector. Name it **Prompt Studio**.
4. MCP server URL: `https://mcp.promptstudio.app/mcp`
5. Authentication: **OAuth**.
6. Confirm that you trust the connector and create it.
7. Complete the Prompt Studio sign-in when ChatGPT asks. Wait until the tool scan finishes.
8. In a chat, enable the Prompt Studio connector and approve tool calls when ChatGPT asks.

ChatGPT reaches the server from OpenAI’s network. Leave client id and client secret empty so it can use dynamic client registration. Do not paste an API key.

### Claude

Claude on the web, desktop, and Cowork can add this as a custom connector. On Team and Enterprise, an Owner adds it.

**Pro and Max**

1. Go to **Customize → Connectors**.
2. Choose **Add custom connector**.
3. Name it **Prompt Studio**.
4. MCP server URL: `https://mcp.promptstudio.app/mcp`
5. Leave OAuth client id and secret empty. Claude discovers the authorization server and registers a client.
6. Choose **Add**, then sign in when asked.

**Team and Enterprise**

1. Go to **Organization settings → Connectors**.
2. Choose **Add**, then **Custom**. If it asks for a type, choose **Web**.
3. Enter `https://mcp.promptstudio.app/mcp`.
4. Leave advanced OAuth fields empty and add the connector.

**Claude Desktop config (`mcp-remote`)**

Desktop builds that only launch a local command can reach the same URL through [`mcp-remote`](https://www.npmjs.com/package/mcp-remote). **Settings → Developer → Edit Config** opens `claude_desktop_config.json` (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows). Add:

```json
{
  "mcpServers": {
    "prompt-studio": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.promptstudio.app/mcp"]
    }
  }
}
```

Restart Claude Desktop. `mcp-remote` opens a browser for OAuth and keeps the refresh token on the machine. Node.js has to be installed so `npx` can run.

### Other MCP clients

Any client that speaks Streamable HTTP can use `https://mcp.promptstudio.app/mcp` and the OAuth discovery documents:

- `https://mcp.promptstudio.app/.well-known/oauth-protected-resource`
- `https://mcp.promptstudio.app/.well-known/oauth-authorization-server`

Scopes are `read` and `write`. The authorization server advertises dynamic client registration and PKCE (`S256`).

Agent Plugins 1.0.0 names this transport `streamable-http`. Cursor and Grok Bot name it `http`. Both point at the same URL. The root [`mcp.json`](mcp.json) is the Agent Plugins file. The Cursor manifest inlines the `http` entry so marketplace install does not depend on the other name.

## What the skills do

| You say | Skill | Tools |
| --- | --- | --- |
| “Save this to Prompt Studio” / “save it to Prompt Studio” | `save-to-prompt-studio` | `save_generation`, or `create_prompt` / `update_prompt` plus attach |
| “Use prompt X from Prompt Studio” / “fetch my … prompt from Prompt Studio” | `use-prompt-from-prompt-studio` | `search_prompts`, then `get_prompt`, then use the phrase |

`search_prompts` returns a short preview. The skill loads the full phrase with `get_prompt` before anyone treats it as the prompt.

Saving keeps the prompt even if one attachment hits a free-plan limit. The reply should say which output failed.

Large files go through `create_asset_upload`, an HTTP PUT of the raw bytes, and `finalize_asset_upload`. They do not go through the tool arguments as base64.

## Plugin files

```text
plugin.json                 Agent Plugins manifest
.cursor-plugin/plugin.json  Cursor and Grok Bot marketplace manifest
mcp.json                    Remote MCP URL (Streamable HTTP, OAuth)
skills/                     Save and fetch workflows
assets/logo.svg             Brand mark, referenced by a relative path
```

Paths in the manifests stay inside this repo. Nothing in git is a secret.

## License

[MIT](LICENSE). Copyright Quick One B.V.

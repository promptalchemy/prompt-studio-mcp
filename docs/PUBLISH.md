# Publishing Prompt Studio

**Not listed yet.** This package is ready to submit. Prompt Studio is not in the [Cursor Marketplace](https://cursor.com/marketplace) and not in the Grok Bot plugin list. Do not write product copy, changelog notes, or support replies that say it is already listed. Update this file after a listing is actually live.

Publisher: **Quick One B.V.** (Prompt Studio).  
Repository: https://github.com/promptalchemy/prompt-studio-mcp  
MCP URL: `https://mcp.promptstudio.app/mcp`  
Docs: https://promptstudio.app/mcp

## Cursor Marketplace

Review is manual. Submitting the repo does not publish the plugin.

1. Keep this repository public.
2. Open https://cursor.com/marketplace/publish
3. Submit `https://github.com/promptalchemy/prompt-studio-mcp`
4. Wait for the Cursor team. Every plugin and later update is reviewed before it is listed.

The marketplace manifest is [`.cursor-plugin/plugin.json`](../.cursor-plugin/plugin.json):

- `name` is `prompt-studio` (lowercase kebab-case)
- `homepage` is `https://promptstudio.app/mcp`
- `logo` is the relative path `assets/logo.svg`
- `mcpServers` is an inline Streamable HTTP server at `https://mcp.promptstudio.app/mcp` with `"type": "http"` (the name Cursor’s plugin loader uses)
- Skills are `./skills/`
- There is no `variables` block, because there is no API key to configure

The repo also has a root [`plugin.json`](../plugin.json) for the Agent Plugins standard, and [`mcp.json`](../mcp.json) with `"type": "streamable-http"` for clients that load that standard. Both entries are the same hosted URL. There is no multi-plugin marketplace manifest, because this repo is one plugin.

### Checklist

- Public git repo, MIT license, README with install steps
- Kebab-case name `prompt-studio`
- Logo committed and referenced with a relative path
- Skill frontmatter has `name` and `description`
- No absolute filesystem paths and no `..` in manifest paths
- No API keys, client secrets, bearer tokens, or service role keys
- OAuth is client-managed. This package does not embed credentials

## Grok Bot

Grok Bot uses this same plugin package once a marketplace listing exists. Until that listing exists, users add the server in chat:

> Add this MCP server: https://mcp.promptstudio.app/mcp

Do not tell users to search Plugins for Prompt Studio before the listing is live. After it is live, Plugins → Prompt Studio is the install path, and the chat URL still works.

## Auth that review will see

The server is OAuth 2.0 authorization-code with PKCE. Discovery:

- https://mcp.promptstudio.app/.well-known/oauth-protected-resource
- https://mcp.promptstudio.app/.well-known/oauth-authorization-server

Dynamic client registration is `https://mcp.promptstudio.app/register`. Scopes are `read` and `write`. Clients that support registration leave client id and secret empty. Do not add a client secret to this repo to “make review easier.”

## What the hosted server does

`https://mcp.promptstudio.app/mcp` is the only server this package connects to. The Worker stores and returns prompts and assets. It does not run image models. `generate_image` queues a job for a Pro account that has image credits, then `poll_generation` reads the result. Free accounts and empty credit balances get a 402 from those tools. That behavior belongs in the skills, not in a second local server.

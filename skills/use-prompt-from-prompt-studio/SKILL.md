---
name: use-prompt-from-prompt-studio
description: >-
  Fetches a saved prompt from Prompt Studio and uses its full phrase. Use when
  someone says "Use prompt X from Prompt Studio", "fetch my … prompt from
  Prompt Studio", "get my prompt from Prompt Studio", or asks to run a prompt
  they saved, from Cursor, Grok Bot, ChatGPT, Claude, or Grok (xAI).
---

# Use a prompt from Prompt Studio

Prompt Studio stores the phrase. This client is the model that follows it. Loading a prompt does not run anything inside Prompt Studio.

The MCP server is `https://mcp.promptstudio.app/mcp`. Sign-in is OAuth. Do not ask for an API key.

## When this applies

Use this skill when the user wants a prompt that already lives in their library. Typical wording:

- "Use prompt X from Prompt Studio"
- "fetch my brand voice prompt from Prompt Studio"
- "get my onboarding prompt from Prompt Studio"
- "use the Prompt Studio prompt called hero banner"

## Steps

1. If the Prompt Studio tools are not available, say the server is not connected and point them at install: in Grok Bot, "Add this MCP server: https://mcp.promptstudio.app/mcp". In Cursor, add that URL under MCP. In ChatGPT or Claude, add it as a custom connector. Then stop.
2. Call `search_prompts` with their words as `query`. Search matches the prompt name and the phrase body. Shared prompts are included unless they ask for their own only (`include_shared` false). Add `folder_id` or `tool_id` only after you have resolved those with `list_folders` or `list_tools`.
3. Read the matches. `phrase_preview` is about 240 characters, not the prompt. When several rows fit, show the names and ask which one. When one row is clearly the one they named, continue with that id.
4. Call `get_prompt` with that id before you follow the prompt or paste it into the chat. Use the full `phrase` from `get_prompt`. Signed asset URLs on that response expire; fetch again if a later step needs the file.
5. Apply the phrase in this conversation. Follow it as the instruction they asked you to use. Prompt Studio does not call ChatGPT, Claude, Grok, or Cursor for you.
6. When they did not name a prompt and want to browse, call `list_prompts` (newest first). Pass `folder_id` to browse a folder. The default list is thread heads and standalone prompts.

## After you load it

Say which prompt you used (name, and folder when it has one). Then do the work the phrase describes. If they only asked to see it, show the phrase and stop.

If `get_prompt` shows `access_role` viewer, you can read and use the phrase. Do not call `update_prompt`, `save_generation` with that `prompt_id`, or delete tools unless they can edit.

To save a new result back onto that prompt, switch to the save skill and pass this `prompt_id` to `save_generation`.

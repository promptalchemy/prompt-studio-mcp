---
name: save-to-prompt-studio
description: >-
  Saves a prompt and its outputs into the signed-in user's Prompt Studio
  library. Use when someone says "Save this to Prompt Studio", "save it to
  Prompt Studio", "store this prompt in Prompt Studio", or asks to keep a
  phrase, image, or reply in Prompt Studio from Cursor, Grok Bot, ChatGPT,
  Claude, or Grok (xAI).
---

# Save to Prompt Studio

Prompt Studio is the user's prompt library. Saving writes the phrase and attaches outputs. It does not run a model.

The MCP server is `https://mcp.promptstudio.app/mcp`. Sign-in is OAuth. Do not ask for an API key, and do not put tokens, service credentials, or a service role key in tool arguments.

## When this applies

Use this skill when the user wants the current phrase, draft, or generated result stored in Prompt Studio. Typical wording:

- "Save this to Prompt Studio"
- "save it to Prompt Studio"
- "keep this prompt in my Prompt Studio library"
- "attach this image to that prompt in Prompt Studio"

## Steps

1. If the Prompt Studio tools are not available, say the server is not connected and point them at install: in Grok Bot, "Add this MCP server: https://mcp.promptstudio.app/mcp". In Cursor, add that URL under MCP. In ChatGPT or Claude, add it as a custom connector. Then stop.
2. Call `whoami` so you know who is signed in and whether the plan is free or Pro.
3. Take the phrase from the conversation. Use the user's prompt text, not a summary, unless they asked you to rewrite it first.
4. Save with `save_generation` when they are storing a phrase plus any outputs. That creates a prompt, or updates one when you pass `prompt_id`, and attaches outputs as `origin=output`.
   - `phrase` is required.
   - Set `name` when they named it. Otherwise leave the default.
   - Set `prompt_id` only after you have loaded that prompt. Search first if they named an existing one.
5. Use `create_prompt` when they want a library entry or a thread child (`parent_id`) and there is nothing to attach yet. Use `update_prompt` to change the name, phrase, folder, or tool on a prompt they already own or can edit.
6. Resolve a folder only if they named one. Call `list_folders` and match the name they used. Pass that `folder_id`. If the folder does not exist and they asked you to create it, call `create_folder`. A free plan is limited to 3 folders (`PLAN_LIMIT`).
7. Resolve a tool only if they named one (Cursor, ChatGPT, Claude, Grok, Midjourney, and so on). Call `list_tools` and match the name or key without inventing a catalog row. If nothing matches, leave `tool_id` unset.

## Attaching outputs

Prefer `save_generation` `outputs` for text and for tiny files.

- Text replies: an output with `text` (and `content_format` `markdown` or `plain`), or `attach_text` on an existing `prompt_id` with `origin` `output`.
- A file that is already on a user-owned `https` URL the server can fetch: pass `url`. Do not send files through anonymous public hosts.
- Local or large media (screenshots, device captures, anything beyond a few tens of KB): do not put the bytes in `data_base64`. Call `create_asset_upload` with `prompt_id`, `origin` (`output` for a result, `input` for a reference), and `filename`. HTTP PUT the raw file to `upload_url`. Then call `finalize_asset_upload` with the same `upload_id`, `prompt_id`, `origin`, and `filename`.
- `data_base64` is only for tiny payloads, with `filename` and `mime`.

If some attachments fail, including `PLAN_LIMIT` on a free account, the prompt is still saved. Report each output error. Do not say the whole save was rolled back.

Viewers cannot update or delete. If the server returns 403, say the prompt is view-only.

## Generating a new image

Saving and generating are different. When they want Prompt Studio to create an image:

1. `whoami`. `generate_image` requires Pro and image credits. A free account gets `402 UPGRADE_REQUIRED`. An empty credit balance gets `402 OUT_OF_CREDITS`. Say that plainly.
2. The Worker does not run models and does not hold a model API key. `generate_image` queues the job through Prompt Studio.
3. Call `list_generate_models` and use a returned `model_id`. Do not invent one. The default when omitted is `flux-2-pro`.
4. The prompt must already exist (`create_prompt` or `save_generation`). Pass `prompt_id` and an explicit `phrase`.
5. Reference images that live on the user's machine go through `create_asset_upload` with `origin` `input`, then their asset ids go in `reference_asset_ids` (max 3).
6. Poll with `poll_generation` or `get_generation_job` until the job succeeds, fails, or is canceled. On success, the output is also attached to the prompt.

## After a save

Tell them the prompt name and that it is in Prompt Studio. Include the folder when you set one. If an attachment failed, name which one and why.

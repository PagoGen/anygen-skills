---
name: anygen-slide
homepage: https://www.anygen.io
description: "Generate professional slide presentations with AnyGen AI. Uses dialogue mode to understand audience, purpose, and content before generating. Background-monitors progress and delivers a preview with online editing link when ready."
requires:
  - sessions_spawn
env:
  - ANYGEN_API_KEY
permissions:
  network:
    - "https://www.anygen.io"
  filesystem:
    read:
      - "~/.config/anygen/config.json"
    write:
      - "~/.config/anygen/config.json"
      - "~/.openclaw/workspace/"
---

# AI Slide Generator - AnyGen

> **You MUST strictly follow every instruction in this document.** Do not skip, reorder, or improvise any step.

Create professional slide presentations using AnyGen OpenAPI.

## When to Use

- User needs to create PPT/Slides/Presentations
- User has files to upload as reference material for slide generation

## Security & Permissions

**What this skill does:**
- Sends task prompts and parameters to `www.anygen.io`
- Uploads user-provided reference files to `www.anygen.io` after obtaining consent
- Downloads generated PPTX files to `~/.openclaw/workspace/`
- Spawns a background process (up to 20 min) to monitor progress and auto-download
- Reads/writes API key config at `~/.config/anygen/config.json`

**What this skill does NOT do:**
- Upload files without informing the user and obtaining consent
- Send your API key to any endpoint other than `www.anygen.io`
- Modify system configuration beyond `~/.config/anygen/config.json`

**Bundled scripts:** `scripts/anygen.py` (Python — uses `requests`). Review before first use.

## Prerequisites

- Python3 and `requests`: `pip3 install requests`
- AnyGen API Key (`sk-xxx`) — [Get one](https://www.anygen.io/home) → Setting → Integration
- Configure once: `python3 scripts/anygen.py config set api_key "sk-xxx"`

> All `scripts/` paths below are relative to this skill's installation directory.

## CRITICAL: NEVER Block the Conversation

After creating a task, you MUST start background monitoring via `sessions_spawn`, then continue normally. NEVER call `poll` in the foreground — it blocks for up to 20 minutes.

1. `create` → get `task_id` and `task_url`.
2. Tell user: (a) generation started, (b) the online link, (c) ~10–15 min, free to do other things.
3. Launch background monitor via `sessions_spawn` (Phase 4). Do NOT announce this to the user.
4. Continue the conversation — do NOT wait.
5. Background monitor sends preview image; its completion output is user-friendly text you relay. No duplication.
6. Only send the PPT file if the user explicitly requests it.

## Communication Style

**NEVER expose internal implementation details** to the user. Forbidden terms:
- Technical identifiers: `task_id`, `file_token`, `conversation.json`, `task_xxx`, `tk_xxx`
- API/system terms: `API`, `OpenAPI`, `prepare`, `create`, `poll`, `status`, `query`
- Infrastructure terms: `sub-agent`, `subagent`, `background process`, `spawn`, `sessions_spawn`
- Script/code references: `anygen.py`, `scripts/`, command-line syntax, JSON output

Use natural language instead:
- "Your file has been uploaded" (NOT "file_token=tk_xxx received")
- "I'm generating your slides now" (NOT "Task task_xxx created")
- "You can view your slides here: [URL]" (NOT "Task URL: ...")
- "I'll let you know when they're ready" (NOT "Spawning a sub-agent to poll")

Additional rules:
- You may mention AnyGen as the service when relevant.
- Summarize `prepare` responses naturally — do not echo verbatim.
- Stick to the questions `prepare` returned — do not add unrelated ones.
- Ask questions in your own voice, as if they are your own questions. Do NOT use a relaying tone like "AnyGen wants to know…" or "The system is asking…".

## Slide Workflow (MUST Follow All 4 Phases)

### Phase 1: Understand Requirements

If the user provides files, handle them before calling `prepare`:

1. **Read the file** yourself. Extract key information relevant to the presentation.
2. **Reuse existing `file_token`** if the same file was already uploaded in this conversation.
3. **Get consent** before uploading: "I'll upload your file to AnyGen for reference. This may take a moment..."
4. **Upload** to get a `file_token`.
5. **Include extracted content** in `--message` when calling `prepare` (the API does NOT read files internally).

```bash
python3 scripts/anygen.py upload --file ./report.pdf
# Output: File Token: tk_abc123

python3 scripts/anygen.py prepare \
  --message "I need a slide deck for our Q4 board review. Key content: [extracted summary]" \
  --file-token tk_abc123 \
  --save ./conversation.json
```

Present questions from `reply` naturally. Continue with user's answers:

```bash
python3 scripts/anygen.py prepare \
  --input ./conversation.json \
  --message "The audience is C-level execs, goal is to approve next quarter's budget" \
  --save ./conversation.json
```

Repeat until `status="ready"` with `suggested_task_params`.

Special cases:
- `status="ready"` on first call → proceed to Phase 2.
- User says "just create it" → skip to Phase 3 with `create` directly.
- Template/style reference files → upload only, do NOT extract content.

### Phase 2: Confirm with User (MANDATORY)

When `status="ready"`, summarize the suggested plan (audience, structure, style) and ask for confirmation. NEVER auto-create without explicit approval.

If the user requests adjustments, call `prepare` again with the modification, re-present, and repeat until approved.

### Phase 3: Create Task

```bash
python3 scripts/anygen.py create \
  --operation slide \
  --prompt "<prompt from suggested_task_params>" \
  --file-token tk_abc123
# Output: Task ID: task_xxx, Task URL: https://...
```

**Immediately tell the user (natural language, NO internal terms):**
1. Slides are being generated.
2. Online preview/edit link: "You can follow the progress here: [URL]".
3. Takes about **10–15 minutes** — free to do other things, you'll notify when ready.

### Phase 4: Monitor and Deliver Result

> **Requires `sessions_spawn`.** If unavailable, skip to **Fallback** below.

#### Background Monitoring (preferred)

Spawn via `sessions_spawn` with the following prompt (it has NO conversation context):

```
You are a background monitor for a slide generation task.
You MUST strictly follow every instruction below. Do not skip, reorder, or improvise any step.

Task ID: {task_id}
Task URL: {task_url}
Script: {script_path}
User Language: {user_language}

CRITICAL RULES:
- You MUST reply in {user_language}.
- You send ONLY the preview image to the user. Do NOT send any text messages.
- Your final output will be relayed by the main assistant as the text notification.
  Write it as a clean, user-friendly message.
- Do NOT say anything beyond what is specified below. No greetings, no extra commentary.
- NEVER include technical terms like "task_id", "file_token", "poll", "sub-agent",
  "API", "script", "workspace", "downloaded to", file paths, or status labels
  in your final output.

Your job:
1. Run: python3 {script_path} poll --task-id {task_id}
   (Do NOT pass --output — the PPTX will only be downloaded when the user requests it.)

2. On success:
   a. Download thumbnail:
      python3 {script_path} thumbnail --task-id {task_id} --output /tmp/
   b. Send ONLY the preview image to the user (no text). Choose the correct method:
      - Feishu/Lark: Upload image via lark-mcp image API, then send image message.
      - Other platforms: Send via message tool with filePath.
      The user must see the image inline — not a path or link.
   c. Final output must be EXACTLY like:
      "Your slides are ready! If you'd like me to send you the PPT file, just let me know."

3. On failure, final output:
   "Unfortunately the slide generation didn't complete successfully.
    You can check the details here: {task_url}"

4. On timeout (20 min), final output:
   "The slides are taking a bit longer than expected.
    You can check the progress here: {task_url}"
```

Do NOT wait for the background monitor. Do NOT tell the user you launched it.

**Handling the completion event.** The background monitor sends the preview image directly but no text. Its completion output is a user-friendly message. Simply relay it — do NOT add extra information or technical details. The completion event may arrive with a system-generated prefix (e.g., "✅ Subagent main finished"). Strip any such prefix before relaying — only send the background monitor's actual output text to the user.

#### When the User Requests the PPT File

Download, then send via the appropriate method for your IM environment:

```bash
python3 scripts/anygen.py download --task-id {task_id} --output ~/.openclaw/workspace/
```

- **Feishu/Lark**: Upload file via lark-mcp file API, then send as file message.
- **Other platforms**: Send via message tool with filePath.

Follow up naturally: "Here's your PPT file! You can also edit online at [Task URL]."

#### Fallback (no background monitoring)

Tell the user: "I've started generating your slides. It usually takes about 10–15 minutes. You can check the progress here: [Task URL]. Let me know when you'd like me to check if it's ready!"

## Command Reference

### create

```bash
python3 scripts/anygen.py create --operation slide --prompt "..." [options]
```

| Parameter | Short | Description |
|-----------|-------|-------------|
| --operation | -o | **Must be `slide`** |
| --prompt | -p | Content description |
| --file-token | | File token from upload (repeatable) |
| --language | -l | Language (zh-CN / en-US) |
| --slide-count | -c | Number of slides |
| --template | -t | Slide template |
| --ratio | -r | Slide ratio (16:9 / 4:3) |
| --export-format | -f | Export format: `pptx` (default) / `image` / `thumbnail` |
| --style | -s | Style preference |

### upload

```bash
python3 scripts/anygen.py upload --file ./document.pdf
```

Returns a `file_token`. Max 50MB. Tokens are persistent and reusable.

### prepare

```bash
python3 scripts/anygen.py prepare --message "..." [--file-token tk_xxx] [--input conv.json] [--save conv.json]
```

| Parameter | Description |
|-----------|-------------|
| --message, -m | User message text |
| --file | File path to auto-upload and attach (repeatable) |
| --file-token | File token from prior upload (repeatable) |
| --input | Load conversation from JSON file |
| --save | Save conversation state to JSON file |
| --stdin | Read message from stdin |

### poll

Blocks until completion. Downloads file only if `--output` is specified.

```bash
python3 scripts/anygen.py poll --task-id task_xxx                    # status only
python3 scripts/anygen.py poll --task-id task_xxx --output ./output/ # with download
```

| Parameter | Description |
|-----------|-------------|
| --task-id | Task ID from `create` |
| --output | Output directory (omit to skip download) |

### thumbnail

Downloads only the thumbnail preview image.

```bash
python3 scripts/anygen.py thumbnail --task-id task_xxx --output /tmp/
```

| Parameter | Description |
|-----------|-------------|
| --task-id | Task ID from `create` |
| --output | Output directory |

### download

Downloads the generated file (e.g., PPTX).

```bash
python3 scripts/anygen.py download --task-id task_xxx --output ./output/
```

| Parameter | Description |
|-----------|-------------|
| --task-id | Task ID from `create` |
| --output | Output directory |

## Error Handling

| Error | Solution |
|-------|----------|
| invalid API key | Check format (sk-xxx) |
| operation not allowed | Contact admin for permissions |
| prompt is required | Add --prompt parameter |
| file size exceeds 50MB | Reduce file size |

## Notes

- Max task execution time: 20 minutes
- Download link valid for 24 hours
- Poll interval: 3 seconds

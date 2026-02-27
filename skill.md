---
name: anygen
description: "Generate AnyGen AI tasks supporting multiple modes: chat (general), slide (PPT), doc (document), storybook, data_analysis, website. Returns task link after submitting prompt."
---

# AnyGen Content Generator

Create AI generation tasks using AnyGen OpenAPI, supporting multiple content generation modes.

## When to use

- User needs to create PPT/Slides (slide)
- User needs to generate documents (doc)
- User needs to create storyboards (storybook)
- User needs data analysis (data_analysis)
- User needs to generate websites (website)
- User needs general AI conversation/generation (chat)
- User provides content description and needs AI to automatically generate various types of content

## Prerequisites

- Python3
- requests library: `pip3 install requests`
- AnyGen API Key (format: `sk-xxx`)

### Getting API Key

If you don't have an API Key, you need to obtain one from the AnyGen website:

1. Visit [AnyGen](https://www.anygen.io)
2. Go to **Setting**
3. Switch to **Integration** Tab
4. Click to generate API Key

### Configuring API Key (Recommended)

After obtaining the API Key, it's recommended to save it to a config file to avoid entering it every time:

```bash
# Save API Key
python3 ~/.claude/my_skills/anygen/scripts/anygen.py config set api_key "sk-xxx"

# View current config
python3 ~/.claude/my_skills/anygen/scripts/anygen.py config get

# View config file path
python3 ~/.claude/my_skills/anygen/scripts/anygen.py config path
```

Config file location: `~/.config/anygen/config.json`

**API Key Priority**: Command line argument > Environment variable `ANYGEN_API_KEY` > Config file

## Required User Inputs

| Field | Description | Required |
|-------|-------------|----------|
| API Key | AnyGen API Key, format `sk-xxx` | Yes |
| Operation | Operation type (see table below) | Yes |
| Prompt | Content description/prompt | Yes |

### Supported Operation Types

| Operation | Description |
|-----------|-------------|
| `chat` | General mode (SuperAgent) |
| `slide` | Slides mode (SuperAgent Slides) |
| `doc` | Doc mode (SuperAgent Doc) |
| `storybook` | Storybook mode |
| `data_analysis` | Data analysis mode |
| `website` | Website development mode |

**Note**: Only `slide` and `doc` support file download. Other operations (`chat`, `storybook`, `data_analysis`, `website`) only return a task URL for online viewing.

## Skill Invocation Flow

### Step 1: Collect Required Information

Before execution, confirm the following information with the user:

```
Please provide the following information:

【Required】
1. API Key: Your AnyGen API Key (format: sk-xxx)
2. Generation type: slide (PPT) or doc (document)
3. Content description: Describe what you want to generate

【Recommended - Can significantly improve generation quality】
4. Style preference: What style do you prefer? For example:
   - PPT: Business formal, minimalist modern, tech-style, academic, creative and lively, etc.
   - Document: Formal report, technical documentation, marketing copy, academic paper, etc.
5. Reference files: Do you have any reference materials? For example:
   - Existing PPT/documents as style reference
   - Related PDFs, images, text materials
   - Company logo or brand assets
   (Supported formats: PDF, PNG, JPG, DOCX, PPTX, TXT)

【Other optional parameters】
- Language: zh-CN (default) or en-US
- PPT page count: Default determined by AI
- PPT template: business, education, etc.
- PPT ratio: 16:9 (default) or 4:3
- Document format: docx (default) or pdf
```

> **Tip**: Style description and reference files are not mandatory, but providing this information can help AnyGen generate content that better matches your expectations.

### Step 2: Create Task

```bash
python3 ~/.claude/my_skills/anygen/scripts/anygen.py create \
  --api-key "sk-xxx" \
  --operation slide \
  --prompt "A presentation about the history of artificial intelligence" \
  --language zh-CN \
  --slide-count 10 \
  --ratio "16:9" \
  --style "tech-style, minimalist modern" \
  --file ./reference.pdf
```

#### Create Task Parameters

| Parameter | Short | Description | Required |
|-----------|-------|-------------|----------|
| --api-key | -k | API Key | Yes |
| --operation | -o | slide or doc | Yes |
| --prompt | -p | Content description | Yes |
| --language | -l | Language (zh-CN/en-US) | No |
| --slide-count | -c | Number of PPT pages | No |
| --template | -t | PPT template | No |
| --ratio | -r | PPT ratio (16:9/4:3) | No |
| --doc-format | -f | Document format (docx/pdf) | No |
| --file | | Attachment file path (can be used multiple times) | No |
| --style | -s | Style preference (e.g., 'business formal', 'minimalist modern') | No |

### Step 3: Poll Task Status

After successful creation, a task_id will be returned. Use the following command to poll:

```bash
python3 ~/.claude/my_skills/anygen/scripts/anygen.py poll \
  --api-key "sk-xxx" \
  --task-id "task_abc123xyz"
```

The script will automatically poll until the task completes or fails, querying every 3 seconds.

### Step 4: Download File (Optional)

After the task completes, you can download the file:

```bash
python3 ~/.claude/my_skills/anygen/scripts/anygen.py download \
  --api-key "sk-xxx" \
  --task-id "task_abc123xyz" \
  --output ./output/
```

## One-Click Execution (Create + Poll + Download)

```bash
# If API Key is already configured, --api-key parameter can be omitted
python3 ~/.claude/my_skills/anygen/scripts/anygen.py run \
  --operation slide \
  --prompt "A presentation about the history of artificial intelligence" \
  --style "business formal" \
  --file ./company_logo.png \
  --output ./output/
```

The `run` command will automatically:
1. Create the task
2. Poll and wait for completion
3. Download the generated file

## Output Examples

### Task Created Successfully

```
[INFO] Creating task...
[SUCCESS] Task created successfully!
Task ID: task_abc123xyz
```

### Polling Progress

```
[INFO] Querying task status: task_abc123xyz
[PROGRESS] Status: processing, Progress: 30%
[PROGRESS] Status: processing, Progress: 60%
[PROGRESS] Status: processing, Progress: 90%
[SUCCESS] Task completed!
File name: AI_History.pptx
Download link: https://xxx.feishu.cn/file/xxx
Link expires at: 2024-02-13 12:00:00
```

### Task Failed

```
[ERROR] Task failed!
Error message: Generation timeout
```

### Download Complete

```
[INFO] Downloading file...
[SUCCESS] File saved: ./output/AI_History.pptx
```

## Error Handling

| Error Message | Description | Solution |
|---------------|-------------|----------|
| invalid API key | Invalid API Key | Check if API Key is correct |
| operation not allowed | No permission for this operation | Contact admin for permissions |
| prompt is required | Missing prompt | Add --prompt parameter |
| task not found | Task does not exist | Check if task_id is correct |
| Generation timeout | Generation timed out | Recreate the task |

## Notes

- Maximum execution time per task is 10 minutes
- Download link is valid for 24 hours
- Single attachment file should not exceed 10MB (after Base64 encoding)
- Polling interval is 3 seconds

## Files

```
anygen/
├── skill.md              # This document
└── scripts/
    └── anygen.py         # Main script
```

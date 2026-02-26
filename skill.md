---
name: anygen
description: "生成 AnyGen AI 任务，支持多种模式：chat (通用)、slide (PPT)、doc (文档)、storybook (故事板)、data_analysis (数据分析)、website (网站开发)。提交 prompt 后返回任务链接。"
---

# AnyGen Content Generator

使用 AnyGen OpenAPI 创建 AI 生成任务，支持多种内容生成模式。

## When to use

- 用户需要创建 PPT/Slide (slide)
- 用户需要生成文档 (doc)
- 用户需要创建故事板 (storybook)
- 用户需要数据分析 (data_analysis)
- 用户需要生成网站 (website)
- 用户需要通用 AI 对话/生成 (chat)
- 用户提供内容描述，需要 AI 自动生成各类内容

## Prerequisites

- Python3
- requests 库: `pip3 install requests`
- AnyGen API Key (格式: `sk-xxx`)

### 获取 API Key

如果没有 API Key，需要在 AnyGen 网站获取：

1. 访问 [AnyGen](https://www.anygen.io)
2. 进入 **Setting** (设置)
3. 切换到 **集成** Tab
4. 点击生成 API Key

### 配置 API Key (推荐)

获取 API Key 后，建议保存到配置文件，避免每次输入：

```bash
# 保存 API Key
python3 ~/.claude/my_skills/anygen/scripts/anygen.py config set api_key "sk-xxx"

# 查看当前配置
python3 ~/.claude/my_skills/anygen/scripts/anygen.py config get

# 查看配置文件路径
python3 ~/.claude/my_skills/anygen/scripts/anygen.py config path
```

配置文件位置: `~/.config/anygen/config.json`

**API Key 优先级**: 命令行参数 > 环境变量 `ANYGEN_API_KEY` > 配置文件

## Required User Inputs

| Field | Description | Required |
|-------|-------------|----------|
| API Key | AnyGen API Key，格式 `sk-xxx` | Yes |
| Operation | 操作类型 (见下表) | Yes |
| Prompt | 内容描述/提示词 | Yes |

### 支持的操作类型

| Operation | 说明 |
|-----------|------|
| `chat` | 通用模式 (SuperAgent) |
| `slide` | Slides 模式 (SuperAgent Slides) |
| `doc` | Doc 模式 (SuperAgent Doc) |
| `storybook` | Storybook 模式 |
| `data_analysis` | 数据分析模式 |
| `website` | Website 开发模式 |

## Skill Invocation Flow

### Step 1: 收集必要信息

执行前需要向用户确认以下信息：

```
请提供以下信息：

【必填】
1. API Key: 你的 AnyGen API Key (格式: sk-xxx)
2. 生成类型: slide (PPT) 或 doc (文档)
3. 内容描述: 描述你想生成的内容

【推荐提供 - 可显著提升生成效果】
4. 风格偏好: 你希望的风格是什么？例如：
   - PPT: 商务正式、简约现代、科技感、学术风格、创意活泼等
   - 文档: 正式报告、技术文档、营销文案、学术论文等
5. 参考文件: 是否有参考资料？例如：
   - 已有的 PPT/文档作为风格参考
   - 相关的 PDF、图片、文本资料
   - 公司 Logo 或品牌素材
   (支持格式: PDF, PNG, JPG, DOCX, PPTX, TXT)

【其他可选参数】
- 语言: zh-CN (默认) 或 en-US
- PPT 页数: 默认由 AI 决定
- PPT 模板: business, education, etc.
- PPT 比例: 16:9 (默认) 或 4:3
- 文档格式: docx (默认) 或 pdf
```

> **提示**: 风格描述和参考文件不是必须的，但提供这些信息可以让 AnyGen 生成更符合你期望的内容。

### Step 2: 创建任务

```bash
python3 ~/.claude/my_skills/anygen/scripts/anygen.py create \
  --api-key "sk-xxx" \
  --operation slide \
  --prompt "关于人工智能发展历史的演示文稿" \
  --language zh-CN \
  --slide-count 10 \
  --ratio "16:9" \
  --style "科技感、简约现代" \
  --file ./reference.pdf
```

#### 创建任务参数

| 参数 | 简写 | 说明 | 必填 |
|------|------|------|------|
| --api-key | -k | API Key | Yes |
| --operation | -o | slide 或 doc | Yes |
| --prompt | -p | 内容描述 | Yes |
| --language | -l | 语言 (zh-CN/en-US) | No |
| --slide-count | -c | PPT 页数 | No |
| --template | -t | PPT 模板 | No |
| --ratio | -r | PPT 比例 (16:9/4:3) | No |
| --doc-format | -f | 文档格式 (docx/pdf) | No |
| --file | | 附件文件路径 (可多次使用) | No |
| --style | -s | 风格偏好 (如 '商务正式', '简约现代') | No |

### Step 3: 轮询任务状态

创建成功后会返回 task_id，使用以下命令轮询：

```bash
python3 ~/.claude/my_skills/anygen/scripts/anygen.py poll \
  --api-key "sk-xxx" \
  --task-id "task_abc123xyz"
```

脚本会自动轮询直到任务完成或失败，每 3 秒查询一次。

### Step 4: 下载文件（可选）

任务完成后可以下载文件：

```bash
python3 ~/.claude/my_skills/anygen/scripts/anygen.py download \
  --api-key "sk-xxx" \
  --task-id "task_abc123xyz" \
  --output ./output/
```

## 一键执行（创建 + 轮询 + 下载）

```bash
# 如果已配置 API Key，可省略 --api-key 参数
python3 ~/.claude/my_skills/anygen/scripts/anygen.py run \
  --operation slide \
  --prompt "关于人工智能发展历史的演示文稿" \
  --style "商务正式" \
  --file ./company_logo.png \
  --output ./output/
```

`run` 命令会自动：
1. 创建任务
2. 轮询等待完成
3. 下载生成的文件

## Output Examples

### 创建任务成功

```
[INFO] 创建任务中...
[SUCCESS] 任务创建成功!
Task ID: task_abc123xyz
```

### 轮询进度

```
[INFO] 查询任务状态: task_abc123xyz
[PROGRESS] 状态: processing, 进度: 30%
[PROGRESS] 状态: processing, 进度: 60%
[PROGRESS] 状态: processing, 进度: 90%
[SUCCESS] 任务完成!
文件名: 人工智能发展历史.pptx
下载链接: https://xxx.feishu.cn/file/xxx
链接有效期至: 2024-02-13 12:00:00
```

### 任务失败

```
[ERROR] 任务失败!
错误信息: Generation timeout
```

### 下载完成

```
[INFO] 下载文件中...
[SUCCESS] 文件已保存: ./output/人工智能发展历史.pptx
```

## Error Handling

| 错误信息 | 说明 | 解决方案 |
|----------|------|----------|
| invalid API key | API Key 无效 | 检查 API Key 是否正确 |
| operation not allowed | 无权限执行该操作 | 联系管理员获取权限 |
| prompt is required | 缺少提示词 | 添加 --prompt 参数 |
| task not found | 任务不存在 | 检查 task_id 是否正确 |
| Generation timeout | 生成超时 | 重新创建任务 |

## Notes

- 单个任务最长执行时间为 10 分钟
- 下载链接有效期为 24 小时
- 附件文件单个不超过 10MB (Base64 编码后)
- 轮询间隔为 3 秒

## Files

```
anygen/
├── skill.md              # 本文档
└── scripts/
    └── anygen.py         # 主脚本
```

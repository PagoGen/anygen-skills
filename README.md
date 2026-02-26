# AnyGen Skills

[AnyGen](https://www.anygen.io) AI 内容生成技能，用于 Claude Code。

## 功能

支持多种 AI 内容生成模式：

| 模式 | 说明 |
|------|------|
| `slide` | PPT/演示文稿生成 |
| `doc` | 文档生成 |
| `chat` | 通用 AI 对话 |
| `storybook` | 故事板生成 |
| `data_analysis` | 数据分析 |
| `website` | 网站开发 |

## 安装

### 作为 Claude Code Skill 使用

```bash
# 克隆到 Claude Code skills 目录
git clone https://github.com/PagoGen/anygen-skills.git ~/.claude/my_skills/anygen
```

### 依赖

```bash
pip3 install requests
```

## 配置

获取 API Key 后保存到配置文件：

```bash
python3 scripts/anygen.py config set api_key "sk-xxx"
```

API Key 获取方式：访问 [AnyGen](https://www.anygen.io) → Setting → 集成 → 生成 API Key

## 使用示例

### 生成 PPT

```bash
python3 scripts/anygen.py run \
  --operation slide \
  --prompt "关于人工智能发展历史的演示文稿" \
  --style "商务正式" \
  --output ./output/
```

### 生成文档

```bash
python3 scripts/anygen.py run \
  --operation doc \
  --prompt "项目技术方案文档" \
  --output ./output/
```

### 附加参考文件

```bash
python3 scripts/anygen.py run \
  --operation slide \
  --prompt "公司季度汇报" \
  --file ./data.xlsx \
  --file ./logo.png \
  --output ./output/
```

## 详细文档

查看 [skill.md](./skill.md) 获取完整使用说明。

## License

MIT

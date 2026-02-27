# AnyGen Skills

[English](./README.md)

一系列用于 [AnyGen](https://www.anygen.io) 的 Claude Code 技能集合 - AI 驱动的内容生成平台。

## 可用技能

| 技能 | 说明 | 状态 |
|------|------|------|
| [task-creator](./task-creator/) | 创建和下载 AnyGen AI 任务（幻灯片、文档、对话等） | ✅ 可用 |

## 安装

将所需的技能文件夹复制到 Claude Code 技能目录：

```bash
# 复制 task-creator 技能
cp -r task-creator ~/.claude/my_skills/anygen-task-creator
```

或克隆整个仓库：

```bash
git clone https://github.com/PagoGen/anygen-skills.git ~/.claude/my_skills/anygen-skills
```

## 前置要求

- Python 3
- requests 库：`pip3 install requests`
- AnyGen API Key（[点此获取](https://www.anygen.io) → 设置 → 集成）

## 仓库结构

```
anygen-skills/
├── README.md              # 英文说明
├── README_zh.md           # 本文件
├── task-creator/          # 任务创建技能
│   ├── README.md
│   ├── README_zh.md
│   ├── skill.md
│   └── scripts/
│       └── anygen.py
└── [future-skill]/        # 更多技能即将推出
```

## 贡献

欢迎贡献！你可以：
- 报告问题
- 提交 Pull Request
- 建议新技能

## 许可证

MIT

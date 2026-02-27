# AnyGen Skills

[中文](./README_zh.md)

A collection of Claude Code skills for [AnyGen](https://www.anygen.io) - AI-powered content generation platform.

## Available Skills

| Skill | Description | Status |
|-------|-------------|--------|
| [task-creator](./task-creator/) | Create and download AnyGen AI tasks (slide, doc, chat, etc.) | ✅ Ready |

## Installation

Copy the desired skill folder to your Claude Code skills directory:

```bash
# Copy task-creator skill
cp -r task-creator ~/.claude/my_skills/anygen-task-creator
```

Or clone the entire repository:

```bash
git clone https://github.com/PagoGen/anygen-skills.git ~/.claude/my_skills/anygen-skills
```

## Prerequisites

- Python 3
- requests library: `pip3 install requests`
- AnyGen API Key ([Get one here](https://www.anygen.io) → Setting → Integration)

## Repository Structure

```
anygen-skills/
├── README.md              # This file
├── README_zh.md           # Chinese version
├── task-creator/          # Task creation skill
│   ├── README.md
│   ├── README_zh.md
│   ├── skill.md
│   └── scripts/
│       └── anygen.py
└── [future-skill]/        # More skills coming soon
```

## Contributing

Contributions are welcome! Feel free to:
- Report issues
- Submit pull requests
- Suggest new skills

## License

MIT

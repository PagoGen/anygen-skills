# Finance Report — 投研PDF报告生成器

## ⛔ 强制规则（违反 = 失败）

> **生成PDF后必须立即用独立短消息发送 MEDIA: 指令。**
> 
> 这不是可选步骤。这是整个skill的最终交付物。
> 没发MEDIA: = 报告没交付 = 任务失败。
>
> ```
> 消息1: 分析结论（纯文字，简短）
> 消息2: MEDIA:/home/xxx/.openclaw/workspace/reports/anygen-XXX.pdf
> ```
>
> ❌ 不要把 MEDIA: 混在长文字中（解析失败）
> ❌ 不要忘记发 MEDIA:（等于白做）
> ❌ 不要输出到 ~/reports/（LocalMediaAccessError）
> ✅ 输出到 ~/.openclaw/workspace/reports/
> ✅ MEDIA: 单独一条极短消息

## 何时使用

用户要求深度分析、财报分析、赛道扫描、出报告时激活。

## 场景路由

| 用户说 | Workflow |
|--------|----------|
| "分析XX" / "研究XX" | `references/initiating-coverage.md` |
| "XX财报分析" | `workflows/earnings-analysis.md` |
| "XX财报预演" | `workflows/earnings-preview.md` |
| "XX赛道扫描" / "XX vs YY" | `workflows/sector-scan.md` |
| "算XX估值" | `references/valuation-methodologies.md` |

## 执行流程（强制顺序）

1. 按对应workflow收集数据（用 `scripts/cli.py`）
2. 按方法论分析（反偏见、证据分级、变异视角）
3. **读取 `templates/components-ref.md`** — HTML组件速查表
4. 按速查表结构生成HTML → 注入CSS → Chromium渲染PDF
5. **输出PDF到 `~/.openclaw/workspace/reports/`**
6. **消息1: 回复分析结论（纯文字，简短摘要）**
7. **消息2: 只含 `MEDIA:/absolute/path/to/file.pdf`（极短消息，必须单独发！）**

⚠️ 第6步和第7步必须是两条独立消息。第7步不能省略。

## 数据收集

使用 `scripts/cli.py` 获取所有数据：
```bash
cd <skill_dir>/scripts
python3 cli.py quote '{"symbol":"NVDA"}'
python3 cli.py fundamentals '{"symbol":"NVDA"}'
python3 cli.py historical '{"symbol":"NVDA","years":3}'
# 完整命令列表: python3 cli.py
```

## PDF生成

1. **必读**: `templates/components-ref.md`（HTML结构速查）
2. **快速指南**: `workflows/pdf-output-quickstart.md`
3. **完整规范**: `workflows/pdf-output.md`（排版、图表、验收）
4. CSS样式: `templates/report-style.css`
5. 品牌配置: `config/output.yaml`

## 方法论

| 框架 | 参考文件 |
|------|---------|
| 反偏见清单 | `references/anti-bias.md` |
| 证据分级T1/T2/T3 | `references/evidence-hierarchy.md` |
| 变异视角 | `references/variant-view.md` |

## 底线

1. 每个数字来自 `cli.py` 工具，不编造
2. 关键判断标注[T1][T2][T3]
3. HTML结构照抄 `components-ref.md`，不发明新class
4. 每份报告Headline不同，反映当次核心变化
5. **PDF必须输出到 `~/.openclaw/workspace/reports/`**
6. **PDF生成后必须发 MEDIA: 指令（独立短消息）— 不发 = 任务失败**

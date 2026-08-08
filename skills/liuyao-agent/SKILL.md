---
name: liuyao-agent
description: 六爻占卜智能体。用户提到六爻、起卦、卦象、卜卦、占卜、应期、旺衰、爻、用神、世应等词时使用。可完成：起卦（手工/随机）、卦象解读、旺衰计算、应期判断、转六亲视角。
compatibility: 需要 Node 18+；脚本零外部依赖（node scripts/liuyao-calc.mjs 直接运行）
---

# 六爻占卜智能体（liuyao-agent）

## 这是什么

六爻起卦与解读的完整智能体：**确定性计算由脚本完成**（起卦、干支四柱、旺衰计算、转六亲），**判断与解读由你完成**（按本 skill 的方法论与参考文档）。所有卦象与旺衰事实必须来自脚本输出，不得自行推算。

## 何时使用

- 用户提出要占卜/起卦（问事：感情、事业、财运、健康、应期等）
- 用户已有卦象（手工摇卦/报数）需要解读
- 用户要求看未来某段时间的旺衰应期

## 能力概览

1. **起卦**（`cast`）：六爻 + 干支四柱 + 本卦/变卦 + 旬空
2. **旺衰计算**（`strength`）：annual/monthly/five_day/intra_day 各时间单位
3. **转六亲视角**（`perspective`）：某爻为参照核心的局部关系
4. **解读**：取用神 → 看旺衰 → 看动变 → 断吉凶与应期

## 项目文件说明（先读这里，按需打开）

| 文件 | 是什么 | 什么时候读 |
|---|---|---|
| `references/01-cast.md` | 起卦：问题确认、起卦方式、六爻/四柱/卦象结构 | 起卦时 |
| `references/02-reading.md` | 解读流程：取用神、世应、动变生克、吉凶判断 | 每次解读时（必读） |
| `references/03-strength.md` | 旺衰纪律：何时必须计算、各时间单位规则、样本语义 | 涉及未来/应期时（必读） |
| `references/04-tool-protocol.md` | liuyao-calc.mjs 命令参数/输出 + 状态协议（含 `retrieve` 检索约定） | 需要计算时（对照参数） |
| `references/05-basic-knowledge.md` | 简易知识：六亲、用神取用、旺衰原则 | 简易模式解读兜底（完整知识库检索见「快速路径」） |
| `references/06-knowledge-injection.md` | 基础知识注入说明：常驻基础 + 确定占问/正式推演两环节提示词（skill 形式） | 进入确定占问或正式推演环节前（必读） |

## 快速路径

- **第一件事**：知识库模式检测 + 声明。用 `node <技能绝对路径>/scripts/liuyao-calc.mjs retrieve --query <检索词>`，输出 `mode: 'complete'|'basic'`：
  - `complete` → 完整知识库可用，解读取义一律走 `retrieve` 检索，**不得直接读取 assets-private 知识库文件**
  - `basic` → 简易模式，用 `references/05-basic-knowledge.md` 兜底，不得宣称使用了完整知识库
  - 知识库存放/变更/检索协议详见仓库 `docs/knowledge-retrieval.md`
- **用户要起卦**：先按 `references/06-knowledge-injection.md`「环节一：确定占问」确认所问之事，再按三种方式之一起卦（见 `references/01-cast.md`）：随机算法（`cast` 默认）、手动摇卦（`cast --lines <6位背数>` 逐爻录入）、用户指明（`cast --name <本卦名> [--changed-name <变卦名>]`，**必须带 `--timestamp`**）→ 得到卦象
- **解读**：先读 `06-knowledge-injection.md`「环节二：正式推演」与 `02-reading.md` 流程；先 `retrieve --query` 检索相关取用/断卦规则，命中的正文作为取义参考，未命中时用 `05-basic-knowledge.md` 或如实说明，不编造规则；涉及未来/应期先读 `03-strength.md` 并调 `strength`
- **需要计算时**：对照 `04-tool-protocol.md` 调脚本

## 脚本调用说明

- 脚本路径用**绝对路径**或先 `cd` 到技能目录：`node /绝对/路径/skills/liuyao-agent/scripts/liuyao-calc.mjs <子命令> ...`。脚本内部通过自身位置解析状态目录（`<技能根>/.stars-state/`），与当前工作目录无关；但 shell 里写相对路径 `scripts/...` 时要求 cwd 是技能根目录
- **多卦可追溯**：每次 `cast` 覆盖旧卦前，旧卦会自动归档到 `<技能根>/.stars-state/archive/<session>.json`（不丢失），输出带 `previousHexagram` + `archiveFile`。**默认只解读最新卦**；需要查看历史卦用 `history`（列出）或 `history --show <序号>`（详情）。多套独立占问用 `--session <名称>` 隔离
- **会话残留**：同一 `--session` 反复使用会累积状态；开始新占问时先 `reset` 或换 `--session`
- **卦象可视化**：`visualize [--no-open]` 生成卦象 HTML（本卦/变卦爻线图、六亲/世应/六神/动爻/旬空标注）并**用本地浏览器打开**；页面与源数据存 `<技能根>/.stars-state/visuals/<session>-<时间戳>.html/.json`，**可追溯**。用户要求「看卦象图」时使用

## 重要边界

- 所有卦象、四柱、旺衰事实必须来自 `liuyao-calc.mjs` 输出，**不得自行推算**
- 工具失败或未返回时，明确说明缺少时点依据，**不得自行外推**旺衰
- 范围样本是附加事实，**不改卦、不改动爻、不生成吉凶结论**——模型必须在脚本返回后再写对应时间判断

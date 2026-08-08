# 06 脚本协议：ziwei-calc.mjs

> 何时读：需要计算时（对照参数）。脚本零依赖，仅需 Node 18+。脚本路径用**绝对路径**或先 `cd` 到技能根目录（状态目录由脚本按自身位置解析，与 cwd 无关）。

## 状态协议

- 会话状态存 `<skill 根>/.stars-state/<session>.json`（默认 session 名 `default`，可用 `--session <id>` 区分多人）
- 脚本自动读写状态文件，**你不需要维护候选/账本状态**——每次调用都是对同一会话的增量操作
- `--session` 只影响状态文件，不影响参数
- **会话残留**：同一 session 重复 `init` 会覆盖旧反推状态（输出带 `previousState` 提示）；多套独立反推用 `--session` 隔离或先 `reset`

## 子命令总览

| 子命令 | 用途 | 关键参数 |
|---|---|---|
| `init` | 生成会话与 15 候选 | `--gender male\|female` `--date YYYY-MM-DD` `[--time HH:MM]` `[--no-true-solar]` `[--place]` `[--timezone]` `[--longitude]` `[--session]` |
| `candidates` | 候选状态列表 | （无，读会话） |
| `chart` | 取某候选完整命盘 | `--candidate-key` |
| `year-facts` | 某候选某年大限/流年/小限事实 | `--candidate-key` `--year` `[--solar-month]` `[--lunar-month]` `[--summary]` |
| `year-comparison` | 全部候选某年对比 | `--year` `[--solar-month]` |
| `dimension-compare` | 全部候选指定宫位星曜对比 | `--palaces 夫妻宫,官禄宫`（最多 6 个） |
| `record-finding` | 记录核对结论 | `[--candidate-key]` `--scope-type year\|month\|dimension\|...` `--scope-value` `--judgment` `--summary` |
| `exclude` | 排除候选 | `--candidate-keys k1,k2` `--reason` |
| `lock` | 提出锁定建议（等待用户确认） | `--candidate-key` `--reason` |
| `retrieve` | 知识库检索（RAG） | `--query <检索词>` `[--category 逗号分隔]` `[--limit 默认6]` `[--knowledge-dir]` |
| `visualize` | 生成盘面 HTML 并用本地浏览器打开 | `[--no-open 只生成不打开]` `[--session]` |
| `reset` | 清空当前会话 | （无） |

## 知识检索（retrieve）

- 用途：解读/核对取义时**必须**用 `retrieve` 检索完整知识库，**禁止直接翻阅 assets-private 知识库 JSON 文件**
- 输出：`mode: complete|basic`（知识库模式声明）、`status: ok|no_match|no_assets`、`hits[{text, section, subject, categories}]`、`discipline`（反编造纪律）
- `mode: complete` → 完整知识库可用；`basic`/`no_assets` → 简易模式，用 `references/07-basic-knowledge.md` 兜底
- 未命中时不得编造规则内容：调整查询词（加星曜+宫位+四化词）重试，或引用简易知识并如实说明
- 完整协议见仓库 `docs/knowledge-retrieval.md`

## 已知时辰直达

- `init --time HH:MM`（如 `--time 23:30`）时输出带 **`recommendedCandidateKey`**（与钟表时间匹配的候选 key，含 `hint` 提示）——直接 `chart --candidate-key <该key>` 一步取盘，**无需在 15 个候选中手工挑**
- 真太阳时默认启用（输出带 `trueSolar: { applied, correctionMinutes }` 标注校准量）；明确知道钟表时间无误时加 `--no-true-solar`（按钟表标准时排盘，`trueSolar.applied=false`）

## 输出

- 全部输出 JSON 到 stdout；错误输出到 stderr（exit 1）
- 关键字段：`candidateStatuses`（候选含 `clockWindow`）、`daXian`/`liunian`/`xiaoxian`/`lunarMonths`（时间层事实）、`eliminatedKeys`/`activeCandidateCount`（排除结果）
- `lunarMonths` 每段带 `lunarMonthLabel` + `continuesFromPreviousMonth`/`continuesIntoNextMonth` 跨月标记：同一流月跨两个公历月时两次展开各出现一段，属同一流月，**不得重复计数**
- `--summary` 时 `liunian`/`xiaoxian` 只保留 `targetPalaceName`/`overlayMutagens`/`highlights`（省略十二宫星曜明细），流月不受影响

## 示例

```bash
# 起盘（女，1995-03-08，时辰未知）
node scripts/ziwei-calc.mjs init --gender female --date 1995-03-08

# 起盘（已知 23:30 出生 → 输出 recommendedCandidateKey，chart 一步取盘）
node scripts/ziwei-calc.mjs init --gender female --date 2004-11-05 --time 23:30

# 某候选 2024 年事实（含 12 月流月；只核对流月时加 --summary）
node scripts/ziwei-calc.mjs year-facts --candidate-key "date:1995-03-07:hour:11:source:1995-03-08" --year 2024 --solar-month 12 --summary

# 对比全部候选感情/事业宫位
node scripts/ziwei-calc.mjs dimension-compare --palaces 夫妻宫,官禄宫

# 排除（按纪律先确认/先核对）
node scripts/ziwei-calc.mjs exclude --candidate-keys "date:1995-03-07:hour:11:source:1995-03-08" --reason "用户明确晚上出生，该候选为清晨"

# 记录结论
node scripts/ziwei-calc.mjs record-finding --candidate-key "..." --scope-type year --scope-value 2024 --judgment 吻合 --summary "2024 年官禄宫四化引动与入职事件吻合"

# 检索知识（解读取义用；输出 mode 声明完整/简易）
node scripts/ziwei-calc.mjs retrieve --query "官禄宫破军化权 事业" --category pattern
```

## 注意事项

- 子命令与工具一一对应（对照表见 SKILL.md「命名对照」）；参数缺失时脚本报「缺少参数 --xxx」
- `year-facts`/`year-comparison` 单轮有 72 次调用上限，聚焦最关键的候选与年份，不逐盘逐年穷举
- 重复同参数调用会返回缓存结果（`cached: true`），直接引用即可，不要重复发起

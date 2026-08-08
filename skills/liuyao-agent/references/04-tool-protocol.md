# 04 脚本协议：liuyao-calc.mjs

> 何时读：需要计算时（对照参数）。脚本零依赖，仅需 Node 18+。脚本路径用**绝对路径**或先 `cd` 到技能根目录（状态目录由脚本按自身位置解析，与 cwd 无关）。

## 状态协议

- 卦象状态存 `<skill 根>/.stars-state/<session>.json`（默认 session 名 `liuyao-default`，可用 `--session <id>` 区分多人/多套占问）
- `cast` 写入卦象；`strength`/`perspective` 读取当前卦并计算
- **每次 `cast` 覆盖旧卦前，旧卦自动归档到 `<skill 根>/.stars-state/archive/<session>.json`**（追加，可追溯不丢失）；输出带 `previousHexagram`（旧卦摘要）与 `archiveFile`。默认只用最新卦，历史卦用 `history` 显式查看

## 子命令

| 子命令 | 用途 | 关键参数 |
|---|---|---|
| `cast` | 起卦（写状态，覆盖前自动归档旧卦；三种方式见 01-cast.md） | `--question` `[--timestamp ISO]` `[--seed 整数]` `[--name 本卦名]` `[--changed-name 变卦名]` `[--lines 6位背数]` `[--timezone]` `[--session]` |
| `strength` | 旺衰计算 | `--target`（见 03）`--scope annual\|monthly\|five_day\|intra_day` `[--timezone]` |
| `perspective` | 转六亲视角 | `--anchor-position 1-6` |
| `history` | 查看本会话历史卦 | `[--show 序号]`（默认列出摘要，`--show N` 查看第 N 条详情） |
| `retrieve` | 知识库检索（RAG） | `--query` `[--category 逗号分隔]` `[--limit 默认6]` `[--knowledge-dir]` |
| `visualize` | 生成卦象 HTML 并用本地浏览器打开 | `[--no-open 只生成不打开]` `[--session]` |
| `reset` | 清空当前会话 | （无） |

## 知识检索（retrieve）

- 用途：解读取用/规则查询时**必须**用 `retrieve` 检索完整知识库，**禁止直接翻阅 assets-private 知识库 JSON 文件**
- 输出：`mode: complete|basic`（知识库模式声明）、`status: ok|no_match|no_assets`、`hits[{text, section, subject, categories}]`、`discipline`（反编造纪律）
- `mode: complete` → 完整知识库可用；`basic`/`no_assets` → 简易模式，用 `references/05-basic-knowledge.md` 兜底
- 未命中时不得编造规则内容：调整查询词重试，或引用简易知识并如实说明
- 完整协议见仓库 `docs/knowledge-retrieval.md`

## 示例

```bash
# 起卦（若已有卦会自动归档）
node scripts/liuyao-calc.mjs cast --question "当前工作是否稳定" --timestamp "2026-08-06T12:00:00+08:00" --seed 12345

# 查看历史卦（列出 / 查看第 0 条）
node scripts/liuyao-calc.mjs history
node scripts/liuyao-calc.mjs history --show 0

# 看今年旺衰
node scripts/liuyao-calc.mjs strength --target "2027年" --scope annual

# 看本月（从 8 月 6 日起覆盖到当月干支月末；注意起卦日临近月末时只是残月，见 03）
node scripts/liuyao-calc.mjs strength --target "2026-08-06" --scope monthly

# 看今天
node scripts/liuyao-calc.mjs strength --target "今天" --scope intra_day

# 转六亲视角（以第 3 爻为参照）
node scripts/liuyao-calc.mjs perspective --anchor-position 3

# 检索知识（解读取义用；输出 mode 声明完整/简易）
node scripts/liuyao-calc.mjs retrieve --query "官鬼爻 占婚姻 应期"
```

## 注意事项

- 单轮最多 12 个不同 target；重复 target 直接复用结果
- `strength` 输出包含每爻旺衰等级（过旺/旺/平/衰/死）、四柱系数、动变影响与计算步骤——这是解读的事实依据
- `monthly` 输出带 `partialMonth`/`coverageNote`：起卦日处于月内时明确标注覆盖区间不是完整月（语义详见 `03-strength.md`）

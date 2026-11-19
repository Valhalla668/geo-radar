# geo-radar

GEO 热点雷达的 **Python 实现**：把公开热榜接成选题卡、合规预警、竞品声量与可校验的草稿。

纯 Python，**零运行时依赖**（只有连 PostgreSQL 时才需要 `psycopg`）。同一套规则、同一套数据格式，与 Node 插件版（[`geo-trend-radar`](https://github.com/Valhalla668/geo-trend-radar)）共用数据目录，两个实现可以交替跑、结果逐行对比。

- **CLI**：`geo-radar scan / topics / alerts / sov / report / calendar / baseline / draft / evidence / db / status`
- **库**：`from geo_radar import api`，每个动作一个函数，返回 `{ok, message, ...}`，`message` 是给人看的 Markdown

## 安装

```bash
git clone https://github.com/Valhalla668/geo-radar.git
cd geo-radar

python -m pip install -e .            # 可选：装成命令行
python -m pip install -e ".[postgres]"  # 需要连数据库时
```

不安装也能直接用：在仓库根目录 `python -m geo_radar ...`。

## 快速开始

```bash
python -m geo_radar status                     # 数据目录、快照、关键词分组、基线进度
python -m geo_radar scan --label daily         # 采集热榜 → 选题卡 + 预警 + 声量 + 日历
python -m geo_radar scan --candidate "方里粉饼适合干皮吗" --candidate "最好用的粉底液"
python -m geo_radar topics --days 3 --limit 10
python -m geo_radar alerts --days 3 --level high
python -m geo_radar sov --days 7
python -m geo_radar report --period weekly
python -m geo_radar baseline init              # 建立审计网格
python -m geo_radar baseline next --size 10
python -m geo_radar draft skeleton --topic-id GT-20260101-001
python -m geo_radar db env                     # 凭据放哪、变量名是什么
```

`--json` 输出结构化结果（脚本用），默认输出 Markdown（人读）。
配置用 `--config config.json`，键与插件版的 `config` 一致（`platforms`、`apiUrl`、`dataDir`、`keywordGroups`、`databases`、`promptsPath` …）。

> Windows 控制台若不是 UTF-8，中文会显示为乱码：先 `chcp 65001`，或设 `PYTHONIOENCODING=utf-8`。

## 数据存放

默认数据目录 `~/.dsh/geo-trend-radar`，与插件版完全一致：

```
snapshots/<id>.json       扫描快照（matchedItems + mentions）
latest.json               最近一次快照指针
topics/<id>.md            选题卡
alerts/<id>.json          预警
baseline/audit.json       审计网格
reports/*.md              日报 / 周报
drafts/*.{json,md}        草稿与骨架
```

所以两个实现能读同一批数据。已验证：本包能读插件版写的快照与 1,440 行审计；插件版也能读本包写的快照，并产出同样的选题卡（同 id、同分数、同关联提示词）。

数值语义也对齐了 JavaScript：`Math.round` 半值向上取整，整数值打印 `50` 而不是 `50.0`。

## 数据库口令

连接信息写配置，**口令不写配置**：

```bash
export GEO_EVIDENCE_PASSWORD=...      # 或写进 .env
```

优先级：真实环境变量 > `.env` > 配置里的明文 `password`（仅兼容，会告警）。
`.env` 查找顺序：`envFile` 配置 → `<数据目录>/.env` → 当前目录 `.env`。
`db env` 显示当前解析到哪、变量名是什么；`db status` 只显示来源，不显示口令。

数据库完全可选：没装 `psycopg` 或没配库时，`db` / `evidence` 相关命令会给出明确提示并退出，扫描、选题、预警、声量、报告、审计、草稿校验照常可用。

## 与插件版的对应关系

| 能力 | 插件版（Node） | 本仓库（Python） |
| --- | --- | --- |
| 采集与打分 | `geo_scan` | `scan` |
| 选题卡 | `geo_topics` | `topics` |
| 预警 | `geo_alerts` | `alerts` |
| 声量 | `geo_sov` | `sov` |
| 报告 | `geo_report` | `report` |
| 日历 | `geo_calendar` | `calendar` |
| 基线审计 | `geo_baseline` | `baseline init/next/record/metrics/status` |
| 草稿骨架与校验 | `geo_draft` | `draft skeleton/validate/render` |
| 证据准入 | `geo_evidence` | `evidence lookup/sku` |
| 数据库通道 | `geo_db` | `db status/template/env/test/schema/sample` |
| Web 看板 | 有（DSH 对话内） | 无，只出 Markdown |

规则、阈值、渲染文案、JSON 结构都与插件版逐条对应。

## 测试

```bash
python -m pytest tests -q
# 临时目录不可写时（受限沙箱）：
python -m pytest tests -q --basetemp=.pytest-tmp
```

46 个用例，覆盖打分、选题卡、预警、声量与日历、快照读写（含插件版快照互通）、审计指标、草稿三闸门、`.env` 解析与优先级、SQL 生成与错误映射。

## 已知限制

- 打分是**规则**而非语义理解：反讽、多义词会误判，选题卡优先级需人工确认。
- 热榜是各平台公开榜单，不含付费投放；「声量份额」是 AI 份额的公开信号代理。
- `draft` 只判定 G1 Schema / G2 证据绑定 / G4 合规；G3 事实蕴含、G5 品牌语气与三签需人审。
- 数据库准入的「有效期」依赖证据表 `valid_until`，为空视为长期有效——这是配置约定，不是法律判断。
- 真实的 PostgreSQL 连接受本机环境限制，目前只验证到连接失败路径与 SQL 生成。

## 许可

MIT，见 [`LICENSE`](LICENSE)。

# 贾静文 · C10H 提交包 · 索引

挑战：C10H 安装并驾驭 Hermes Agent（`ch-20260717031609-rnur5a`）
交付方式：**本地交付 + 平台提交**（challengeId `ch-20260717031609-rnur5a`，`get-challenge` 核对状态 `published`，交付物要求 `*安装记录*`、`*配置说明*`、`*demo*`、`*AI日志*`，截止 2026-12-31）

## 文件清单

| 文件 | 内容 | 对应交付物模式 |
|---|---|---|
| `贾静文_C10H_安装记录.md` | 环境基线、7 步安装过程、4 条真实排错、耗时实测、可复现性 | `*安装记录*` |
| `贾静文_C10H_配置说明.md` | config.yaml 字段与理由、SOUL.md 改写、自定义技能、定时任务、运行态与限制、回滚 | `*配置说明*` |
| `贾静文_C10H_demo记录.md` | 两次实跑的命令/耗时/原始输出、模型自称 vs 实际配置核对表、运行态快照、复现命令 | `*demo*` |
| `贾静文_C10H_AI日志.md` | 与 AI 协作全过程的真实记录（含卡点与决策） | `*AI日志*` |
| `贾静文_C10H_拿来说明.md` | 挑战要求 → 证据文件的对照说明 | — |
| `贾静文_C10H_自定义技能\eduseed-precheck\SKILL.md` | 自写技能原文（2107 B） | 加分项证据 |
| `配置\config.yaml` / `SOUL.md` / `SOUL.原始备份.md` | 生效配置与人设原文、改写前备份 | 配置证据 |
| `原始输出\` | `c10h_demo1.txt`、`c10h_demo2.txt`、`c10h_usage1.json`、`c10h_usage2.json`、`c10h_skills.txt`、`c10h_cron.txt`、`c10h_status.txt` | 原始证据 |

## 三条硬证据

1. **本地技能计数 0 → 1**：`hermes skills` 报 `0 hub-installed, 38 builtin, 1 local — 39 enabled`。
2. **定时任务真实存在**：job `f2a519ad7af8`，`0 9 * * *`，Asia/Shanghai，last run `2026-11-09T22:00:01 ok`。
3. **两次实跑可复现**：78.1 s（人设自述）、210.8 s（`-s eduseed-precheck` 加载技能后复述检查项）。

## 明确标注的未验证项（不当作完成度）

- cron **自然调度**（`source=scheduled`）：要到 2026-11-10 09:00 才首次发生，打包时无证据。
- 多平台接入：`platforms: {}`，本机未配置任何 IM 平台。
- `hermes skills publish` 原创技能发布：未走通，+10% 加分不计入。
- 评分维度：驾驭 25 / 集成 20 / 产物 15 / AI 20 / 复盘 20；红线 `missing_artifacts` / `no_ai_log` / `one_shot_ai` 会把对应维度封顶 5 分。

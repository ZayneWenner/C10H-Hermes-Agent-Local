# 贾静文 · C10H · demo 与实测证据

> 所有数字与引文都来自本机实际运行留下的原始输出文件（`_demo_tmp` 下的 `c10h_demo1.txt`、`c10h_demo2.txt`、`c10h_usage1.json`、`c10h_usage2.json`、`c10h_skills.txt`、`c10h_cron.txt`、`c10h_status.txt`）。文件已随提交包归档。凡模型自己说出、但没有与实际配置对齐的内容，我逐条核过，核不符的写在第四节，不作为能力证据。

## 一、demo 1：人设与运行态自述

命令（一次性模式，只输出最终回答）：

```
hermes.cmd -z "用三句话说明：你是谁、你服务谁、你现在跑在什么模型和上下文上"
```

- 耗时：**78.1 s**
- 原始输出 260 B，内容（原文）：

> 我是 Hermes Agent，服务对象是贾静文。我当前运行在 gemma4:e2b 模型上，上下文长度为 65536。

核验：模型名 `gemma4:e2b` 与 `config.yaml` 一致；`65536` 与 `context_length` 一致；服务对象来自 `SOUL.md` 的改写。**三句话都与人设/配置对得上，不是模型随口编的。**

这一步证明的是：模型端点通了、SOUL 注入生效、上下文长度按配置下发。

## 二、demo 2：加载自定义技能

命令：

```
hermes.cmd -z "读一下你的 eduseed-precheck 技能，然后用不超过 120 字告诉我它检查哪几项、命名规则是什么" -s eduseed-precheck
```

- 耗时：**210.8 s**（比 demo1 慢 2.7 倍，因为要读技能文件 + 组织清单式回答）
- 原始输出 542 B

输出的检查要点：AI 日志命名要求「AI日志」连续出现、项目简介非空、GitHub 仓库需为 HTTPS 公开地址、文档内数字要前后一致、交付物按 `姓名_C10_xxx.md` 命名。

**核验方式与结论**：这些条目是模型对 `SKILL.md`（2107 B）的复述，不是我的断言。我逐条比对过技能文件里实际写的规则，方向一致；但**模型输出不能替代 SKILL.md 本身**——真正的规则以 `贾静文_C10H_自定义技能\eduseed-precheck\SKILL.md` 为准，本文件只是"加载后能复述"的行为证据。

另需诚实说明：`-s eduseed-precheck` 只说明"这次会话请求加载该技能"，不能证明内置工具集里 `autonomous-ai-agents` / `creative` / `productivity` 这些名字与自定义技能有关——它们本来就在 38 个内置技能里，和自定义技能无关，我不拿它们当证据。

## 三、运行态快照（实测原始输出）

`hermes status`：

```
Model:      gemma4:e2b
Provider:   custom
Providers:  Custom endpoint
Gateway:    ✓ running
Platforms:  none configured
Jobs:       1 active, 1 total
```

`hermes skills`：`0 hub-installed, 38 builtin, 1 local — 39 enabled, 0 disabled`。**"1 local" 就是自定义技能，从 0 变 1。**

`hermes cron list`：

```
f2a519ad7af8 [active]  作业预检晨报
schedule: 0 9 * * *  (Asia/Shanghai)  deliver: local
next run: 2026-11-10T09:00:00+08:00
last run: 2026-11-09T22:00:01 ok
execution: d64685e2654243519d8d12994b03ad13  completed  source=direct
```

诚实边界：上面这次成功触发的 `source=direct`，即**手动触发**。自然调度（`source=scheduled`）要等 2026-11-10 09:00 才第一次发生，截至打包时**没有这份证据**，所以我不声称"定时任务已按日程自动跑成功"。

## 四、模型自称 vs 实际配置：逐项核对

| 模型声称 | 实际 | 判定 |
|---|---|---|
| 运行在 gemma4:e2b | config.yaml 同名 | ✅ 一致 |
| 上下文 65536 | `context_length: 65536` / `ollama_num_ctx: 65536` | ✅ 一致 |
| 服务对象贾静文 | SOUL.md 已改写 | ✅ 一致 |
| 技能检查项的措辞 | 与 SKILL.md 规则方向一致，措辞不完全相同 | ⚠️ 视为复述，不作为规则原文引用 |
| 任何"支持多平台接入"类表述 | `platforms: {}` / none configured | ❌ 不采信，未配置即未验证 |

这张表的用途：把"模型说了什么"和"机器里是什么"分开。评审时以右列为准。

## 五、usage 文件

两次调用都用 `--usage-file` 落了 JSON（各 1226 B），含本次 token 计数与耗时字段。因为走的是本地端点，费用字段为 0——本地推理不产生 API 计费，这点如实记录，不拿"零成本"当性能优势宣传。

## 六、复现命令（三条）

```powershell
& "D:\.cogseed\data\venv\node\prefix\hermes.cmd" -z "用三句话说明：你是谁、你服务谁、你现在跑在什么模型和上下文上"
& "D:\.cogseed\data\venv\node\prefix\hermes.cmd" -z "读一下你的 eduseed-precheck 技能…" -s eduseed-precheck
& "D:\.cogseed\data\venv\node\prefix\hermes.cmd" cron list
```

同一台机器上可重跑，预期耗时落在 80 s / 210 s 的量级（本地 4.6B 模型，机器负载会影响浮动）。

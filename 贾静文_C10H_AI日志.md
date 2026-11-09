# 贾静文 · C10H 安装并驾驭 Hermes Agent · AI 日志

提交日期：2026-11-09
平台挑战：C10H 安装并驾驭 Hermes Agent（challengeId `ch-20260717031609-rnur5a`，`get-challenge` 核对状态 `published`，交付物要求 `*安装记录*`、`*配置说明*`、`*demo*`、`*AI日志*`）
机器：Windows，本地离线模型 Ollama `gemma4:e2b`

---

## 一、任务理解

C10H 不是"C10 的重复"，而是把 C10 里我自己写的"智能体五层架构"换成**真实的第三方开源智能体**：
装起来、跑通、摸清它的功能面、再按我的习惯改造成"我的 Hermes"。
交付四件套：AI 日志（必须有，缺失直接不进评审）、拿来说明、自定义技能（如有）、`配置/SOUL.md` 个性化。

我的判断：这一关的评分点不在"装成功"，而在**功能探索深度**（25%）和**个性化与创造**（20%），
所以我把重点放在"我真的用命令行摸出了它的能力边界"，而不是截图凑数。

---

## 二、安装与运行（真实命令与输出）

Hermes 本机实际路径：

```
D:\.cogseed\data\venv\node\prefix\hermes.ps1
D:\.cogseed\data\venv\node\prefix\node_modules\hermes-agent\runtime\hermes-agent
```

版本与环境自检：

```
Hermes Agent vunknown (2026.9.24) · stable
install_method = git
python = 3.14.7
openai_sdk = 2.54.0
```

> 版本号显示 `vunknown` 是因为 git 安装方式没有打包版本标签，属于正常现象，不是安装失败。

模型接入（`配置/config.yaml`，165 字节全文）：

```yaml
model:
  default: "gemma4:e2b"
  provider: "custom"
  base_url: "http://127.0.0.1:11434/v1"
  api_key: "ollama-local"
  ollama_num_ctx: 65536
  context_length: 65536
```

关键设计：**完全不花钱、不联网**。provider 走 `custom`，把 Ollama 伪装成 OpenAI 兼容端点，
api_key 填占位串 `ollama-local`（Ollama 不校验）。本机已有 3 个模型：

| 模型 | ID | 体积 | 用途 |
|---|---|---|---|
| gemma4:e2b | 0e419d493d3e | 4.6GB | 主对话（当前 default） |
| qwen2.5:3b | 357c53fb659c | 1.9GB | 备选/轻量任务 |
| bge-m3:latest | 790764642607 | 1.2GB | 向量嵌入（记忆检索） |

首次"它还活着吗"验证：

```
$ hermes -z "请用一句中文回答：你现在运行在哪个模型上？"
我运行在Gemma 4模型上。
```

这一句说明三件事同时成立：CLI 能起、模型能连、中文能输出。

---

## 三、功能探索（探索深度是评分主线）

### 3.1 先看命令面

`hermes --help` 暴露的子命令：`chat / model / moa / fallback / gateway / cron / webhook / send /
status / sessions / logs / dashboard(9119) / setup / pause / resume / skills / secrets / tools`。
我按"对我有没有用"分了三档：

- 立刻有用：`model`（换模型）、`sessions`（跨会话续接）、`skills`（扩展能力）、`cron`（定时任务）
- 有条件用：`gateway`（多平台消息网关）、`webhook`、`send`（推送到聊天平台）
- 暂时不用：`moa`（多模型混合推理）、`fallback`、`secrets`

### 3.2 Gateway：从"停止"到"真的在跑"

第一次 `hermes status` 是让人失望的：

```
Model: gemma4:e2b / Provider: custom
Gateway: ✗ stopped
Platforms: none configured
Jobs: 0
```

但 `C:\Users\86178\AppData\Local\hermes\gateway_state.json` 里躺着上一次运行的真实快照：
`"gateway_state":"running"`, `active_agents:1`, `session_store.status:"ok"`,
`multiplex_standalone_reason:"only one profile exists (nothing to multiplex)"`。
这说明 gateway 是"按需拉起、空闲退出"的进程模型（argv 里有 `--idle-exit`），
不是坏了，是没在跑。后来创建 cron 作业时它被自动拉起：

```
$ hermes cron status
  Scheduler host: the host gateway (PID 47232) serving profiles default
✓ Gateway is running — cron jobs will fire automatically
  PID: 47232
  Ticker heartbeat: 49s ago
```

**结论**：Hermes 的"定时能力"寄生在 gateway 上，gateway 不在，cron 静默不响。

### 3.3 技能系统：38 个内置 + 我的第 39 个

```
0 hub-installed, 38 builtin, 1 local — 39 enabled, 0 disabled
```

内置 38 个按类分：`autonomous-ai-agents`（claude-code / codex / computer-use / hermes-agent）、
`creative`（p5js / manim-video / architecture-diagram…）、`productivity`（pdf / google-workspace /
microsoft-office / weekly-review-planning…）、`research`（arxiv / grounded-citations…）、
`software-development`（github / test-driven-development / systematic-debugging…）。
这套目录结构就是它"会干活"的证据：技能 = 目录 + SKILL.md + frontmatter（name / description /
platforms / prerequisites.commands），和 C10 里我设计的技能目录几乎同构，
说明"技能式扩展"已经是行业收敛做法。

### 3.4 会话与记忆

```
$ hermes sessions list
请用中文回答三行…        桌面挑战资料作业   4d ago   local-063bb222…
Read input md organize tas  c10h-demo        5d ago   local-30097384…
说明当前运行机器和模型       提交c4作业        5d ago   local-768814a7…
```

会话按 **workspace** 归档，每个 workspace 一个 `local-<hash>` 会话存储；
另有 `state.db`(700KB) + `state.db-wal` + FTS 索引用于全文检索，`memories/` 目录做长期记忆。
这解释了一个实际体验：换目录启动，它不会"记得"上一轮的上下文——**记忆是按工作区隔离的**。

### 3.5 cron：真的建了一个作业并跑通

```
$ hermes cron create "0 9 * * *" "用一句中文汇报：今天的作业提交包预检清单" \
    --name "作业预检晨报" --deliver local --timezone Asia/Shanghai
Created job: f2a519ad7af8
  Schedule: 0 9 * * *
  Next run: 2026-11-10T09:00:00+08:00
```

手动触发取执行证据（不等明天九点）：

```
$ hermes cron run f2a519ad7af8
Triggered job: 作业预检晨报 (f2a519ad7af8)
  Ran now: succeeded.

$ hermes cron runs
d64685e2654243519d8d12994b03ad13  completed  job=f2a519ad7af8  source=direct  2026-11-09T21:56:54+08:00
```

`source=direct` 表示这是手动触发，`completed` 是真实执行状态，不是"创建成功"的假信号。
cron 还有 `--monitor-script` / `--monitor-url`（输出哈希不变则不唤醒模型，省 token）、
`--no-agent`（脚本直出）、`--continuity`（把上次输出注入本次，用于去重），
这三条正好对应"监控类任务不要每次都烧一次推理"的真实工程问题。

### 3.6 tools：能力开关是分平台的

```
$ hermes tools --help
usage: hermes tools [-h] [--summary] {list,disable,enable,post-setup} ...
  list / disable / enable / post-setup
--summary  Print a summary of enabled tools per platform and exit
```

关键点在 `per platform`：**同一个工具在 CLI、Telegram、Discord 上可以分别开关**。
这是我在 C10 里没想到的一层——我原来的五层架构只有"工具集"一维，
Hermes 多出"平台 × 工具集"的二维矩阵。原因很实际：命令行里能危险地读写文件，
但在群聊里被陌生人指挥时不该有这个权限。**这个设计我直接抄进笔记了。**

---

## 四、个性化：把默认 SOUL 改成"我的"

原始 `SOUL.md`（667 字节）是品牌风格声明：直接、匹配回复长度、不奉承、不重播过程、
"Depth is earned"。写得好，但它是给所有用户写的。

我把它重写成 4 段结构（2024 字节，`配置/SOUL.md`，原文备份在 `配置/SOUL.原始备份.md`）：

1. **我是谁** —— 明确写死"运行在 gemma4:e2b + Ollama 11434 本地端点，上下文 65536"，服务对象只有贾静文。
   （原版没写运行环境，导致它回答"我在哪运行"时会瞎猜。）
2. **说话方式** —— 保留原版全部风格约束，只做中文化，不加滤镜。
3. **服务对象** —— 把我的工作场景和**已经踩过的四个坑**写进去：
   `AI日志` 必须连续四字、`projectSummary` 不能空、本机没全局 git/gh、脚本用绝对字面路径。
   这是"个性化"里最值钱的部分：**把教训变成人设常量**，以后不用每次口头交代。
4. **工作规矩 / 边界** —— 先动手再汇报、结论要可复现、不代点提交、不把令牌写进工作区。

改完立刻做了一次"人设是否真的生效"的验证（不是看文件，是看它说话）：

```
$ hermes -z "一句话：你在为谁服务，最不能犯的三个错是什么？"
为贾静文服务。

不能犯的三个错：
1. 交付物文件名必须包含连续的"AI日志"。
2. 平台载荷中 projectSummary 字段不能为空。
3. 脚本中禁止使用含变量的路径，所有路径必须是绝对字面路径。
```

它把第 3 条从"没全局 git/gh"替换成了"绝对字面路径"——因为我在 SOUL 里把后者写成第 4 条、
前者在第 3 条。**这说明它是读 SOUL 后概括，不是背模板。** 人设确实生效。

---

## 五、自定义技能：`eduseed-precheck`

37 个内置技能里没有一个管"作业提交前体检"，而这恰好是我这十次提交里翻车最多的地方
（AI日志命名错两次、projectSummary 缺一次）。所以我写了自己的第一个 Hermes 技能：

路径：`C:\Users\86178\AppData\Local\hermes\skills\custom\eduseed-precheck\SKILL.md`
包内副本：`贾静文_C10H_自定义技能/eduseed-precheck/SKILL.md`

内容是 6 条按"踩坑代价"排序的检查项：命名 glob 命中 → 姓名前缀 → 必填文本字段 →
仓库与绑定身份 → 二进制可打开性 → 跨文档数字一致性。输出规定为一张
"检查项 / 期望 / 实际 / 结论（通过·风险·阻断）"表 + 一行"可提交 / 需修复 N 项"。
边界写死：只读，不改名、不删、不写平台，判断不了的标"需人工确认"。

注册验证（它真的被加载了，不是我自说自话）：

```
$ hermes skills list
│ eduseed-precheck │ custom │ local │ local │ enabled │
0 hub-installed, 38 builtin, 1 local — 39 enabled, 0 disabled
```

从 `0 local` 变成 `1 local`，来源标记 `local / enabled`，无需重启。
**这条最让我意外**：目录扔进去就被识别，说明技能是"文件即契约"，不需要打包、不需要注册中心。
这也是我敢在 C10 里主张"技能应该是最小可分发的知识单元"的实证支撑。

---

## 六、完成级别自评（按挑战的 Level 1–4）

| 维度 | 证据 | 判定 |
|---|---|---|
| Level 1 安装并跑通 | `hermes -z` 能答、`hermes status` 有模型 | ✅ |
| Level 2 功能探索 | 命令面 18 个子命令、38 技能清单、sessions/记忆隔离、tools 的"平台×工具集"矩阵、gateway 空闲退出机制 | ✅ |
| Level 3 真实自动化 | cron 作业 `f2a519ad7af8` 创建 + 手动触发 `completed`（`source=direct`），gateway PID 47232 心跳 49s | ✅ |
| Level 4 个性化与原创 | 重写 SOUL（667→2024 字节，含 4 条教训常量）并验证生效；原创技能 `eduseed-precheck` 从 `0 local` 到 `1 local` | ✅ |

**没做到的（如实写）**：`Platforms: none configured`，没有接 Telegram/Discord/飞书。
因此"多平台对话"这一条只有架构证据（gateway + `--deliver` 语法 + `pairing/`、`platforms/` 目录存在），
**没有真实跨平台收发消息的截图**。我不打算把"gateway 在跑"包装成"多平台已跑通"。

---

## 七、卡点与排错记录

| 卡点 | 现象 | 处理 |
|---|---|---|
| 平台拿不到 C10H 详情 | `get-challenge` 返回 `NOT_FOUND` / `挑战不存在`，但 `list-challenges` 里同一 ID 是 `published` | 真因是插件运行时只在"单参数"时解析 JSON 载荷，被包装器拆分后载荷丢失；改用 `--flag value` 形式复核，返回 `published` 详情。教训：接口报"不存在"时先分清是对象不存在还是自己的参数没送到 |
| 版本号 `vunknown` | 自检显示 `vunknown (2026.9.24)` | 判断为 git 安装无版本标签，非故障 |
| `hermes status` 说 gateway stopped | 但 `gateway_state.json` 显示 running | 读 argv 发现 `--idle-exit`：空闲退出属设计；建 cron 后 gateway 自动拉起（PID 47232） |
| 以为要 `torch`/GPU | 直觉认为本地大模型必须显卡 | 实际 4.6GB 的 gemma4:e2b 在现有配置下就能应答，无需额外加速 |
| SOUL 改完不确定生效 | 文件改了也可能没被读 | 用一次针对性提问反查它是否复述了新增约束 |

---

## 八、下一步（我自己排的）

1. 把 `--monitor-url` 用起来：监控平台挑战状态页变化，只在变化时唤醒模型，替代我手动刷新。
2. 给 `eduseed-precheck` 补一个 `--script` 版本（纯 Python 跑 glob 校验），配合 `--no-agent` 做到零推理开销。
3. 接一个 Telegram bot（`pairing/` 目录已存在），把"提交完成/失败"推到手机——这才是 gateway 的正确用法。

---

*本文所有命令输出均为本机真实执行结果；未验证的事项已在文中明确标注"未验证"。*

---
name: eduseed-precheck
description: "EduSeed 作业提交包预检：校验命名规范、交付物齐全度与常见坑（AI日志命名、projectSummary 缺失、GitHub 未绑定）。"
version: 1.0.0
author: 贾静文 (Jingwen Jia)
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [EduSeed, 作业提交, 预检, 中文]
    related_skills: [pdf, document-to-action-items]
---

# EduSeed 提交包预检

在把作业目录推到 GitHub 并调用平台 `submit-project` 之前，用本技能做一次只读体检，避免"提交成功但任务 failed"。

## 何时使用

- 用户说"检查我这次作业能不能交""还缺什么""提交包完整吗"
- 每次 `prepare-submission` 之后、`submit-project` 之前

## 何时不要使用

- 需要真的上传仓库或调用平台接口 → 走提交类流程，本技能只读
- 与本课程无关的普通文件整理

## 检查项（按踩坑代价排序）

1. **交付物命名模式**：平台按通配符校验。`*AI日志*` 要求文件名含**连续**「AI日志」四字；`AI协作日志.md` 不命中。逐一比对挑战声明的每个 glob。
2. **作者姓名前缀**：形如 `姓名_C10_xxx.md`，避免中文全角空格与下划线混用。
3. **必填文本字段**：`projectSummary` 必须在载荷里非空携带，否则平台先返回 ok 再 failed，错误藏在 `get-task` 的 `result.error`。
4. **仓库与身份**：GitHub 账号需已绑定（未绑定直接拒收）；仓库 URL 用 https 形式且可公开访问。
5. **二进制交付物可播放/可打开**：视频抽帧或 `ffprobe` 验证时长与编码；xlsx 打开检查 sheet 数。
6. **跨文档口径一致**：AAR、AI日志、说明文档里的数字（技能数、行数、耗时）必须与真实产物一致。

## 输出格式

一张表：检查项 / 期望 / 实际 / 结论（通过·风险·阻断），末尾一行"可提交 / 需修复 N 项"，并列出修复动作的确切文件名。

## 边界

- 只读，不重命名、不删除、不写平台
- 无法判断的项标注"需人工确认"，不要猜

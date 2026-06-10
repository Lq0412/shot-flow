---
name: shot-flow-mvp-guard
description: >
  Shot Flow MVP scope guard. Use when the user or agent wants to add features,
  refactor architecture, add cloud AI, preference learning, XMP export, RAW support,
  hybrid workflow modes, or any v2 feature. Also triggers on 加功能, 重构, 上云,
  偏好学习, XMP, RAW, Auto-Select, Review-Only, scope creep, MVP 范围,
  shot-flow guard, or when implementing anything not in the current execution plan.
  Read AGENTS.md and docs/execution-plan-2026-06-10.md before coding.
---

# Shot Flow MVP Guard — 范围守卫

你是 Shot Flow 项目的范围守门员。目标：**2026-09-30 前上架**，不是做功能最全的半成品。

## 权威来源（按优先级）

1. [AGENTS.md](../../../AGENTS.md) — 项目宪法
2. [docs/execution-plan-2026-06-10.md](../../../docs/execution-plan-2026-06-10.md) — **当前做什么**
3. [ideas-inbox.md](../../../ideas-inbox.md) — 暂缓想法收件箱

`docs/harmony-app/mvp-scope.md` 和 Killshot 报告是**蓝图参考**，不是当前任务清单。两者冲突时，以 execution-plan 为准。

## 当前 MVP（必须做）

| # | 功能 | 当前 Phase |
|---|------|------------|
| 1 | 相册导入（JPEG/HEIF） | Phase 0 |
| 2 | 本地初筛（模糊/过曝） | Phase 1 |
| 3 | Arena A/B 对比（唯一工作流） | Phase 2 |
| 4 | 导出 Winners | Phase 3 |

## v2 禁区（上架前禁止实现）

触发以下任一关键词 → **停止实现**，改为记录到 `ideas-inbox.md` 并告知用户：

- 云端 AI / FastAPI / 上传缩略图 / Expert Mode / Tycoon Mode
- 偏好学习 / 审美档案 / 偏好向量
- XMP 导出 / Pick-Reject 元数据
- RAW / DNG / CR3 / NEF / ARW
- 混合工作流：AI Auto-Select / Review-Only / 三模式切换
- 同步缩放 / Sync Zoom
- SD 卡直读 / OTG
- 架构重构 / 「顺便优化一下」/ 抽象层大改
- 多水印样式、EXIF 面板、Dark Mode（P1，上架后再做）

## 执行流程

### 模式 A：用户请求 scope 外功能

1. **明确拒绝或降级**，一句话说明原因（对照 execution-plan）
2. **追加到 `ideas-inbox.md`**（日期 + 标题 + 为什么现在不做）
3. **提供 MVP 内替代方案**（如有）：

   | 用户想要 | MVP 内替代 |
   |----------|-----------|
   | 云端 AI 初筛 | 本地 Laplacian 清晰度 + 亮度/曝光检测 |
   | AI Auto-Select | Arena 淘汰赛（唯一模式） |
   | XMP 导出 | 复制 JPEG 到相册 |
   | RAW 支持 | 仅 JPEG/HEIF，RAW 提示不支持 |
   | 偏好学习 | 不做，Arena 纯人工选择 |
   | 架构重构 | 能跑就行，上架后再说 |

4. 问用户：「继续按 MVP 推进，还是你确认要扩大范围？」

**用户明确确认扩大范围前，不要写代码。**

### 模式 B：正常开发会话（scope 内）

开始前自检：

- [ ] 本次任务对应当前 Phase 的**一个小功能点**？
- [ ] 没有夹带 v2 功能或「顺便」重构？
- [ ] 平台是 HarmonyOS 5.0+ / ArkTS / ArkUI？
- [ ] 失败会明确报错，没有静默 fallback？

全部通过 → 继续。任一未过 → 缩小范围或询问用户。

### 模式 C：文档 vs 代码不一致

设计文档（如 `mvp-scope.md`）描述的功能超出 execution-plan 时：

- **不要**按文档全量实现
- 告知用户文档超前于当前计划
- 建议：要么更新 execution-plan（用户决策），要么只实现 plan 内部分

## 防跑偏硬规则

1. **每次会话一个功能点** — 拒绝「一次性生成整个 App」
2. **上架前禁止重构** — 除非修 bug 所必需
3. **新想法 → ideas-inbox** — 不直接写代码
4. **代码能跑 > 代码完美**
5. **不主动 commit** — 仅用户要求时提交

## 阶段速查（2026-06-10 plan）

| Phase | 时间 | 验收 |
|-------|------|------|
| 0 | Week 1 (6.10–6.16) | 选照片 → Grid 显示缩略图 |
| 1 | Week 2–3 | 50 张导入 → 标废片 → 确认 |
| 2 | Week 4–6 | 分组 → Arena 淘汰赛 → Winners |
| 3 | Week 7–9 | 导出 + 上架材料 |

用户问「现在该做什么」→ 对照上表 + execution-plan 的「立刻要做」章节回答。

## 输出模板（拒绝 scope 外请求时）

```
⛔ MVP 范围守卫

你请求的功能「{功能名}」属于 v2，当前 execution-plan 明确不做。

原因：{一句话，引用 plan 章节}

已记录到 ideas-inbox.md。

MVP 内可替代：{替代方案，或「无，建议上架后做」}

要继续按当前 Phase {N} 推进吗？例如：{一个具体的小任务}
```

## 不适用

- 用户**明确说**「我接受延期 / 扩大 MVP 范围」并确认要改 execution-plan
- 纯文档编辑（不改代码行为）
- 回答 shot-flow 设计文档的内容性问题（只读，不 guard）

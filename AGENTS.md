# Shot Flow — Agent 工作约定

> 本文件是 AI Agent 在本仓库开发时的项目宪法。与全局 skill 冲突时，**以本文件为准**。

## 项目是什么

Shot Flow 是鸿蒙 5.0+ 原生 AI 照片选片 App（ArkTS + ArkUI）。本仓库当前以设计文档为主，逐步加入鸿蒙工程代码。

- **目标**：2026-09-30 前上架华为应用市场
- **开发方式**：AI Coding（Cursor / Claude Code），用户做产品决策与验收
- **时间约束**：每天 1–2 小时

## 当前阶段权威文档

| 优先级 | 文档 | 用途 |
|--------|------|------|
| **P0** | [docs/execution-plan-2026-06-10.md](docs/execution-plan-2026-06-10.md) | **当前做什么、不做什么** — 以这份极简计划为准 |
| P1 | [docs/harmony-app/mvp-scope.md](docs/harmony-app/mvp-scope.md) | 完整功能蓝图（含 v2 设想，勿全部当作当前任务） |
| P1 | [docs/harmony-app/harmony-tech-stack.md](docs/harmony-app/harmony-tech-stack.md) | ArkTS API 映射与技术选型 |
| P2 | `docs/interaction/`、`docs/ai-engine/`、`docs/architecture/` | 设计参考，实现时按需读取 |
| P2 | [docs/reference/](docs/reference/) | 官方政策整理（激励计划报名入口等） |

**重要**：`mvp-scope.md` 与 Killshot 报告中的扩展功能（云端 AI、混合工作流、XMP、偏好学习等）**不属于当前 MVP**。用户明确要求前，不要主动实现。

## 平台与技术约束

- **目标平台**：HarmonyOS 5.0+，ArkTS，ArkUI
- **照片格式（MVP）**：JPEG / HEIF only
- **处理位置（MVP）**：全部设备端本地，不上云
- **无静默降级**：依赖/API 失败时明确报错，禁止悄悄换假数据或空结果糊弄过去
- **代码质量**：能跑优先，上架前不做架构重构

## 当前 MVP 范围（2026-06-10 执行计划）

### 必须做

1. 相册导入照片
2. 本地初筛（模糊 / 过曝）
3. Arena A/B 对比选最佳（唯一工作流模式）
4. 导出 Winners 到相册

### 明确不做（v2，上架后再说）

| 功能 | 处理方式 |
|------|----------|
| 云端 AI / FastAPI 服务 | 拒绝实现，记到 [ideas-inbox.md](ideas-inbox.md) |
| 偏好学习 | 同上 |
| XMP 导出 | 同上 |
| RAW / DNG 支持 | 同上 |
| 三种工作流模式（Auto-Select / Review-Only） | 同上，MVP 只做 Arena |
| 同步缩放、SD 卡直读、团队协作 | 同上 |

用户说「加功能」「重构」「上云」时，触发 **shot-flow-mvp-guard** skill 做范围检查。

## AI Coding 会话规则

每次会话只做 **一个小功能点**，流程：

```
1. 确认今天要做什么（对照 execution-plan 当前 Phase）
2. 读取相关现有代码 + 设计文档片段
3. 生成/修改代码 → 模拟器验证 → 修到能跑
4. 用户明确要求时才 commit
```

### 给 AI 下指令时

- 附上华为官方 Codelab / API 文档链接（如有）
- 明确平台：HarmonyOS 5.0+、ArkTS、ArkUI
- 不要一次生成整个 App 或整个模块

## Git 约定

对齐全局 `pr-commit` skill，本项目 scope 如下：

| 目录/区域 | scope | 说明 |
|-----------|-------|------|
| `harmony-app/`、`entry/`、`AppScope/` | `app` / `harmony` | 鸿蒙客户端代码 |
| `cloud/`、`backend/`、`api/` | `cloud` / `api` | 云端服务（v2，当前不应有） |
| `docs/` | `docs` | 文档 |
| `.cursor/`、`.github/`、CI | `chore` / `ci` | 配置与工具 |

Commit 格式：`type(scope): subject`（subject 默认中文，与 `git log` 历史一致）

### Agent 行为

- **不要**主动 `git commit` / `git push`，除非用户明确要求
- **不要**在 commit message 中添加 `Co-authored-by: Cursor` 等 Agent 行
- **不要**提交 `.env`、密钥、构建产物

## 文档产出路径

| 类型 | 路径 |
|------|------|
| 产品分析报告 | `docs/product-killshot-YYYY-MM-DD.md` |
| 执行计划 | `docs/execution-plan-YYYY-MM-DD.md` |
| 设计修正计划 | `docs/superpowers/plans/` |
| 新想法（不立即做） | [ideas-inbox.md](ideas-inbox.md) |

## 相关全局 Skills（建议安装）

| Skill | 用途 |
|-------|------|
| `code-discipline` | 禁兜底/猜测/冗余代码 |
| `pr-commit` | Commit / PR 规范 |
| `ai-design-smell` | UI 避坑（Arena / 初筛页定稿前） |
| `shot-flow-mvp-guard` | 本项目范围守卫（`.cursor/skills/`） |

## 最低上架标准（底线）

- [ ] 相册导入（JPEG/HEIF）
- [ ] 自动筛除模糊/过曝废片
- [ ] A/B 对比选择
- [ ] 导出 Winners
- [ ] 隐私政策 + 用户协议
- [ ] 不崩溃

水印、设置页、最近记录 — 加分项，非必须。

# Shot Flow — AI 照片选片工具 鸿蒙 App 参考

> 本仓库是鸿蒙 App 开发的参考文档库，整理了从 "片刻" 桌面版项目中提取的有价值设计思路、架构模式和交互方案。

## 文档结构

```
docs/
├── architecture/          # 架构设计思路
│   ├── cloud-ai-service.md        # 云端 AI 服务设计（FastAPI + 部署方案）
│   ├── runtime-device-mgmt.md     # GPU/CPU 运行时管理思路
│   └── state-persistence.md       # 状态持久化与恢复模式
│
├── harmony-app/           # 鸿蒙 App 专项
│   ├── harmonyos-strategy.md      # 鸿蒙激励计划战略方案
│   ├── harmony-tech-stack.md      # 鸿蒙端技术栈与 API 映射
│   └── mvp-scope.md               # MVP 功能范围与时间线
│
├── interaction/           # 交互设计思路
│   ├── arena-pk-design.md         # 擂台 A/B 对决交互设计
│   ├── prescreen-ux.md            # 智能初筛与复核交互
│   └── watermark-styles.md        # 水印样式设计思路
│
├── ai-engine/             # AI 算法思路（不包含代码）
│   ├── multi-signal-grouping.md   # 多信号融合分组思路
│   ├── quality-scoring.md         # 多维画质评分思路
│   ├── preference-learning.md     # 用户偏好学习机制
│   └── llm-judge-prompt.md        # LLM 视觉判断 prompt 工程
│
└── market/                # 市场与竞品
    └── competitive-analysis.md    # 竞品分析与差异化定位
```

## 核心思路速查

| 思路 | 一句话说明 | 详见 |
|------|-----------|------|
| 擂台 PK 交互 | 侧边 A/B 对决替代列表/网格浏览，用户左右选择 | [arena-pk-design.md](docs/interaction/arena-pk-design.md) |
| 三层 AI 能力阶梯 | 极速(本地CV) / 专家(本地AI) / 土豪(云端LLM)，按需升级 | [mvp-scope.md](docs/harmony-app/mvp-scope.md) |
| 多信号融合分组 | DINOv2 + 人脸 + EXIF时间 + GPS + 文件名，权重自适应 | [multi-signal-grouping.md](docs/ai-engine/multi-signal-grouping.md) |
| 偏好学习 | 记录用户擂台决策，累积审美偏好，越用越懂你 | [preference-learning.md](docs/ai-engine/preference-learning.md) |
| 运行时分层传播 | UI→API→环境变量→模型加载，统一控制 GPU/CPU | [runtime-device-mgmt.md](docs/architecture/runtime-device-mgmt.md) |
| 加速器感知并发 | CUDA 多线程喂 GPU，MPS/CPU 单线程，按硬件调整 | [runtime-device-mgmt.md](docs/architecture/runtime-device-mgmt.md) |
| 无静默降级 | 依赖缺失时明确报错，不返回空结果 | [cloud-ai-service.md](docs/architecture/cloud-ai-service.md) |

## 目标

- 鸿蒙 5.0+ 原生 App（ArkTS + ArkUI）
- 云端 AI 服务（Python FastAPI）
- 2026-09-30 前上架华为应用市场（激励计划 ¥3,000）

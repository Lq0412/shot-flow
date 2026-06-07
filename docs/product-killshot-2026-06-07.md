# Product Killshot Report — Shot Flow
> Generated: 2026-06-07 | Top Competitor Analyzed: Aftershoot ($45/mo, market leader)

## Executive Summary

Shot Flow 是一款定位鸿蒙生态的 AI 选片工具，以 Arena A/B 对决为核心交互创新，配合偏好学习系统和三级 AI 能力阶梯。竞品分析发现：**AI 选片市场全部桌面端优先，移动端几乎空白，鸿蒙生态零竞争。** 但双视角分析揭示了一个根本性矛盾——文档设计的 AI 引擎过度精巧，而用户体验层严重欠设计。最大的战略风险不是技术，而是平台锁定：鸿蒙生态缺少目标专业用户。**建议：(1) Arena 作为精修工具而非主流程，增加 AI 自动选片模式；(2) 将 Speed Mode 完全本地化实现离线可用；(3) 偏好学习系统可视化、可分享，打造「不可逆」锁定效应。** 立即可做的 6 项 Quick Win 改动每项工作量 ≤ 2 天，但能显著改变产品体感。

## Product Snapshot

| 维度 | 内容 |
|------|------|
| **产品名称** | Shot Flow（照片流） |
| **一句话定位** | AI 驱动的智能选片工具——用 Arena 对决帮你从连拍中选出最佳照片 |
| **目标用户** | 鸿蒙设备上的摄影爱好者 + 轻度专业用户（婚礼/人像/活动摄影） |
| **核心特性** | Arena A/B 对决淘汰赛、三级 AI（Speed/Expert/Tycoon）、AI 预筛选、多信号聚类分组、偏好学习、EXIF 水印渲染 |
| **技术栈** | 前端：ArkTS + ArkUI（鸿蒙 5.0+ 原生）；后端：Python FastAPI（DINOv2 + NIMA + InsightFace） |
| **当前状态** | 预开发阶段（14 份设计文档，零代码） |
| **目标上线** | 2026-09-30 鸿蒙应用市场（激励机制 3000 元） |

## Competitive Landscape

### Top Competitor: Aftershoot

| 维度 | 详情 |
|------|------|
| **定位** | AI Cull + Edit + Retouch + Gallery 全链路平台 |
| **定价** | $10-60/月（Culling Only $10/mo，Complete $45/mo，Max $60/mo） |
| **核心优势** | ① 完全离线本地处理，隐私优先 ② AI 编辑学习个人风格（需 5000+ 张训练）③ 自动选片 4000 张 < 10 分钟 ④ 客户画廊交付（2026 新增）|
| **核心劣势** | ① AI 选片准确率受诟病（Reddit: "waste of money"，用户仍需人工复核）② 只看锐度不看情感/创意（体育巅峰时刻、艺术表达被误杀）③ 纯桌面端，无移动端 ④ 风格偏好不透明，2 年使用后仍不可控 |
| **锁定因素** | 个人 AI Profile 训练积累、全链路依赖（选片→修图→精修→交付）、年付订阅惯性 |
| **用户口碑** | 94% 推荐率（官方数据）；但 Reddit/论坛高频抱怨「AI 不靠谱」「创意片被杀」「不信任 AI 选择」|

### Secondary Competitors

| 竞品 | 定位 | 定价 | 关键差异 |
|------|------|------|---------|
| **Narrative Select** | AI 辅助（人在环路） | $10-60/月 | 最快 RAW 预览、Scenes View、Close-ups Panel；偏保守，适合控制型用户 |
| **FilterPixel** | 场景特化 AI + 云端 | $10-28/月 | 唯一支持体育/活动/会议等场景的特化模型；94% 准确率；云端处理设备无关 |
| **Photo Mechanic 6** | 纯手动极速浏览 | $139 买断 | 无 AI，纯速度；体育摄影师/新闻摄影师标准工具 |
| **Photoscope (iOS)** | 消费级照片对比 | 免费 | 基础对比工具，无 AI 评分，非专业定位 |
| **AI Photo Culling - PhotoPicker (iOS)** | 移动端 AI 选片 | $6/周或$15/年 | 最接近 Shot Flow 的移动端竞品；支持从 SD 卡直接读取、XMP 导出 |

### 鸿蒙市场

**零竞争。** 华为应用市场「摄影与视频」分类中无任何 AI 选片工具。鸿蒙 6 自带 AI 构图辅助和智能相册分类，但无连拍选片或专业选片功能。Shot Flow 具备先发优势。

---

## Part I: Competitive Killshot（竞品杀手锏）

> 来自 Agent A（Aftershoot 资深用户视角）的核心洞察

### 🎯 Quick Wins（低投入，高影响）

| # | 特性 | 为什么能杀死竞品 | 工作量 | 影响 |
|---|------|-----------------|--------|------|
| 1 | **"Quick Win"AI 接受按钮** | Arena 分组中 AI 置信度差距大的组（如最高分 8.5 vs 次高分 5.0），一键接受 AI 推荐，跳过淘汰赛。省 40%+ 滑动次数 | Low（1-2天） | High |
| 2 | **同步缩放对比模式 (Sync Zoom)** | 竞品没有。双照片同步放大到同一区域对比锐度，是专业选片的核心需求。解决「左看右看记不住」的痛点 | Med（3-5天） | High |
| 3 | **XMP 侧边栏元数据导出** | 选片结果写入 XMP（星级、色标、Pick/Reject 标记），可导入 Lightroom/Capture One。这是专业工作流的「语言」| Med（3-5天） | High |
| 4 | **偏好可视化仪表盘** | "你的审美：偏爱明亮色调、浅景深、自然表情"。让看不见的 AI 偏好变成看得见的「我的审美档案」。竞品没有 | Med（3-5天） | High |
| 5 | **离线 Speed Mode（完全本地化）** | Aftershoot 的核心卖点是离线可用。Shot Flow 的 Speed Mode 也能离线——只需把 pHash/ORB/Laplacian 在设备端实现。免服务器成本，免网络依赖 | High（1-2周） | High |
| 6 | **从 Aftershoot 迁移向导** | 引导用户选择常用模式（保守/标准/激进），映射为偏好向量初始值。降低冷启动焦虑 | Low（1-2天） | Med |

### 🔥 Switching Triggers（降低切换成本的特性）

**ST-1: 混合模式——Arena 不是唯一选择**

Aftershoot 用户的核心习惯是「AI 自动选 + 人工审核」。对 3000 张婚礼照片强制 Arena 对决 = 1500 次滑动 × 3 秒 = 75 分钟，比 Aftershoot 的 40 分钟更慢。

→ 增加三种工作流模式：
- **AI Auto-Select**：AI 自动选出每组最佳（类 Aftershoot），仅边界案例进入 Arena
- **Arena Mode**：完整淘汰赛（当前设计）
- **Review-Only**：跳过 AI，纯手动网格浏览+标记

→ **优先级：P0**。Arena 应是精修工具而非主流程。

**ST-2: SD 卡直读 + 现场选片流程**

专业摄影师的移动端刚需是：SD 卡读卡器 → 手机 → 现场开始选片 → 回工作室后同步到桌面。设计「插入读卡器」引导流程，支持 USB-C/OTG。

→ **优先级：P1**（鸿蒙 USB-OTG API 支持）

**ST-3: 跨平台同步策略**

纯鸿蒙 = 自我设限。专业摄影师的工作流是 MacBook/iPad + iPhone。建议：利用已有 FastAPI 后端，同期开发一个 Web App（PWA），至少覆盖桌面端浏览和结果查看。

→ **优先级：P1**。不是「未来计划」，是「同期开发」。

### 💀 Deal-Breaker Gaps（不解决就不可能被采用的致命缺陷）

| # | 致命缺陷 | 影响 | 修复方案 |
|---|---------|------|---------|
| DB-1 | **无 RAW 文件支持** | 专业摄影师 100% 拍 RAW。不能处理 CR3/NEF/ARW = 玩具不是工具 | P0：至少支持 JPEG + HEIF + DNG；P1：完整 RAW 缩略图提取 |
| DB-2 | **无下游工作流衔接** | "复制到 winners/ 文件夹" 不是工作流。结果死了在文件夹里 | P0：XMP 导出；P1：Lightroom 集成 / 分享到编辑 App |
| DB-3 | **高量级处理能力未验证** | 100 张免费上限，无 2000+ 张测试数据 | P0：取消免费版数量限制，改为功能限制（仅 Speed Mode） |
| DB-4 | **全量云端依赖** | Expert/Tycoon 必须上传。婚礼现场 WiFi 不可靠 | P0：Speed Mode 离线化；P1：上传队列 + 自动重试 |
| DB-5 | **Arena 强制模式** | 对高量级拍摄（婚礼 3000+ 张），Arena 比竞品更慢 | P0：增加 AI Auto-Select 模式 |

### 🔒 Lock-In Opportunities（锁定机会——用了就回不去）

**LOCK-1: 偏好学习系统 → 个人审美档案（最强锁定）**

Aftershoot 2 年不学习你的选择。Shot Flow 的偏好学习是竞品没有的唯一特性。但要生效必须：
- 扩展偏好向量从 3 维 → 7+ 维（增加色彩温度、构图平衡、景深、表情）
- 可视化展示（"你的审美图谱"）
- 可导出/分享（团队同步、师徒传承）
- 累积统计（"3000 次决策后，你的偏好置信度 87%"）

**LOCK-2: 客户/场景专属配置**

"这是张家的婚礼——张太太偏好明亮自然，张先生喜欢戏剧感。" 每客户偏好档案 + 人脸记忆，跨会话激活。离开 Shot Flow = 丢失所有客户的审美档案。

**LOCK-3: Arena 淘汰赛作为社交内容**

"500 张照片如何选出 1 张最佳——看我的淘汰赛晋级图！" 可分享的赛程图/视频。摄影师自发传播 = 免费获客。竞品无此机制。

**LOCK-4: 累积分析仪表盘**

"今年你选了 47,000 张照片。淘汰率 62%。最常拒绝原因：闭眼（23%）。偏爱 f/2.8 拍摄。" 个人摄影数据日志，1 年积累后不可丢弃。

**LOCK-5: 团队协同选片**

影楼 5 位摄影师共享审美标准 → 离开 = 5 人同时失去共享校准。B2B 锁定。

---

## Part II: User Experience Evolution（体验进化）

> 来自 Agent B（Shot Flow 资深用户视角）的核心洞察

### 🎯 Quick Wins

| # | 改进 | 维度 | 工作量 | 影响 |
|---|------|------|--------|------|
| 1 | **Arena 动画从 500ms 降到 150ms**，高级动画放设置项 | 速度 | Low（改 1 个常量） | High |
| 2 | **Arena 比较后显示可见的 Undo 按钮**（底部 3 秒浮现） | UX | Low（1天） | Med |
| 3 | **首页显示"继续上次"卡片 + 最近会话列表** | UX | Low（1-2天） | High |
| 4 | **云处理三阶段进度条**（上传 N/200 → 分析 N/200 → 分组中） | UX | Med（2-3天） | High |
| 5 | **预筛选缩略图放大至 2 列 + 置信度徽章**（红/黄色标） | UX | Low（1天） | Med |
| 6 | **水印设置增加"摄影师名称/工作室名"自定义文本** | 粘性 | Low（半天） | Med |

### ⚡ Speed Optimizations（速度优化）

**SPD-1: 后台处理 + 推送通知** `High Effort | High Impact`
当前设计：上传 → 等待评分 → 等待分组 → 开始 Arena。200 张照片在 Expert Mode 下等待 2-4 分钟，期间 UI 冻结。

→ 用户点「分析」后进入后台，系统通知「预筛选结果已就绪」。状态机已有（pending → scanning → hashing → grouping → done），只需非阻塞化 UI。

**SPD-2: 精细化 Arena 进度持久化** `Med Effort | High Impact`
状态文件缺少 `currentPairIndex`、`currentRound` 字段。App 被杀后重新打开无法恢复到具体比较对。

→ 扩展状态 Schema：`{ groupId, roundNumber, pairIndex, leftPhoto, rightPhoto }`。冷启动时显示"恢复：第 5 组，第 2 轮"横幅。

**SPD-3: 预筛选"胶片扫描"模式** `Low Effort | Med Impact`
分组展开 → 逐张点击 → 恢复，47 张被拒照片需要 47 次点击。

→ 横向全幅滑动条：上划恢复、下划确认拒绝。一动作一照片，无需分类导航。

**SPD-4: 水印批量渲染 + 进度条** `Med Effort | Med Impact`
30 张冠军照片 Canvas 渲染无进度指示，UI 冻结 45 秒像崩溃。

→ Worker 线程后台渲染 + "正在添加水印 12/30..." 进度条 + 可取消。

### ✨ Quality Enhancements（质量提升）

**Q-1: 自适应时间阈值分组** `Med Effort | High Impact`
固定时间阈值（如 2 秒）在 3fps 慢速连拍和 30fps 高速连拍之间表现差异巨大。

→ 计算会话中位间隔时间，阈值 = `median × 3` 或最大 5 秒。文件名连续 + 间隔均匀 = 同一连拍。

**Q-2: 正面检测前置——解决背影误杀** `Med Effort | High Impact`
背影/侧面照片被 EAR（眼部纵横比）误判为闭眼。婚礼 20-30% 的情感瞬间是从背后拍摄。

→ 先做人脸姿态分类器，yaw > 45° 跳过闭眼检测。标记为"非正面——跳过眼部检查"。

**Q-3: 偏好向量扩展至 7+ 维** `Med Effort | High Impact`
当前 3 维（aesthetic, sharpness, brightness）在 300+ 次决策后饱和，无法捕获新维度。

→ 增加色彩温度（HSV H 通道中位数）、构图平衡（三分法偏差）、景深（背景模糊比）、表情（微笑检测）。信号大多已在评分管线中存在。

**Q-4: 场景类型自动检测 + 权重预设** `Med Effort | Med Impact`
固定权重（aesthetic=0.50, sharpness=0.20, face=0.15, tech=0.15）对演唱会/产品/风光等场景不适用。

→ 利用已有 DINOv2 embedding 做零样本分类（"婚礼仪式"/"人像"/"演唱会"/"风光"/"产品"），各场景映射权重预设。

**Q-5: 合影主次人脸分级惩罚** `Med Effort | Med Impact`
10 人合影 1 人闭眼 → 全图 -1.5 惩罚可能误杀 9 人状态良好的照片。

→ 按人脸面积和位置排序主次人脸，闭眼惩罚仅应用于前 2-3 张主要人脸，次要人脸降为 -0.3。

**Q-6: LLM 评判一致性校验** `Med Effort | Med Impact`
Tycoon Mode 同一照片两次评判可能结果不同（LLM 不确定性 3-5%）。

→ 首次置信度 < 0.8 的照片自动二次评判。不一致则标记"不确定——需人工复核"。

### 🎨 UI/UX Upgrades（界面交互升级）

**UX-1: Arena 触觉反馈** `Low Effort | Med Impact`
设计仅有视觉反馈（绿色边框/灰色淘汰），无触觉。移动端缺少触觉确认导致误操作。

→ 使用鸿蒙 `@ohos.vibrator` API：轻触=选择已注册，双击=撤销触发，中等震动=分组完成。

**UX-2: 水印全分辨率预览** `Low Effort | Low Impact`
选完水印样式直接批量渲染，品牌 Logo 匹配失败只有在导出后才能发现。

→ 批量渲染前显示首张照片的全分辨率水印预览，可平移/缩放检查文字可读性。

### 💪 Stickiness Multipliers（粘性倍增器）

**STK-1: 周报摘要 + 个人基准线** `Med Effort | High Impact`
6 个月后用户不知道自己是否进步。"本周选片 347 张，比上月快 23%，AI 一致率 78%。"

→ 周一推送通知 + 应用内仪表盘。连续使用天数计数器（Streak）。

**STK-2: 冠军合集可分享拼图** `Med Effort | High Impact`
选片完成 = 导出到文件夹，无视觉总结。错失病毒传播闭环。

→ 自动生成 3×3 或 4×4 冠军照片拼图（带 Shot Flow 品牌水印）。一键分享到微信/小红书。

**STK-3: 偏好档案导出 + 团队同步** `Med Effort | High Impact`
影楼多摄影师需统一审美标准。导出偏好为二维码/链接，团队成员导入对齐。

→ "团队模式"锁定共享偏好向量，防止单人使用漂移。

**STK-4: 客户人脸记忆** `High Effort | High Impact`
重复拍摄同一家庭/客户时，自动识别"张家的上次拍摄"，优先展示客户状态最佳的照片。

→ 会话完成后提示「保存客户档案」，存储 InsightFace 向量 + 昵称。下次检测到相同人脸自动激活。

**STK-5: 主动通知"有新照片待选片"** `Low Effort | Med Impact`
鸿蒙支持相册变更监听。检测到新照片 → 推送"你有 47 张昨天的照片，准备选片吗？"

→ 从被动工具变为主动习惯。

**STK-6: 推荐系统 + 偏好档案共享** `Med Effort | Med Impact`
新用户前 30 次决策是流失高风险期（偏好学习未生效）。

→ 推荐人分享偏好档案作为新用户"起点"，双方获 50 次 Tycoon Mode 免费额度。偏好学习从个人苦练变社交优势。

---

## Part III: Unified Priority Matrix

### Priority Classification

| 优先级 | 数量 | 描述 |
|--------|------|------|
| **P0 (Do Now)** | 9 | 阻塞性缺陷修复 + 高 ROI Quick Win。不解决 = 产品不可用或不可卖 |
| **P1 (Next Sprint)** | 10 | 竞争力关键特性。解决后能与竞品正面竞争 |
| **P2 (Roadmap)** | 8 | 重要但非紧急的差异化特性 |
| **P3 (Nice-to-have)** | 5 | 打磨和锦上添花 |

### Implementation Roadmap

#### Phase 0 — 架构修正（Week 1, 与 MVP 开发并行）

> 这些是设计文档级别的修正，应在编码开始前/同时调整

- [ ] **P0** [设计修正] 增加三种工作流模式：AI Auto-Select / Arena Mode / Review-Only（当前只有 Arena）`[docs/interaction/arena-pk-design.md]`
- [ ] **P0** [设计修正] Speed Mode 完全本地化路径——定义设备端算法实现方案，不经过云 API `[docs/harmony-app/harmony-tech-stack.md]`
- [ ] **P0** [设计修正] 免费版取消 100 张数量限制，改为功能限制（仅 Speed Mode 免费） `[docs/market/competitive-analysis.md]`
- [ ] **P0** [设计修正] 增加 RAW/DNG 缩略图支持方案（至少 DNG + JPEG + HEIF） `[docs/harmony-app/mvp-scope.md]`
- [ ] **P0** [设计修正] XMP 侧边栏导出方案（星级、Pick/Reject、可选色标） `[docs/harmony-app/mvp-scope.md]`

#### Phase 1 — Quick Wins（Week 2-3, MVP 开发期嵌入）

- [ ] **P0** Arena 动画默认 150ms，500ms 放设置项「沉浸动画」 `[docs/interaction/arena-pk-design.md]`
- [ ] **P0** 首页「继续上次」恢复卡片 + 最近会话列表 `[docs/harmony-app/mvp-scope.md]`
- [ ] **P0** Arena "Quick Win"AI 接受按钮（AI 置信度差距 > 阈值时显示） `[docs/interaction/arena-pk-design.md]`
- [ ] **P0** 云处理三阶段进度条（上传 → 分析 → 分组） `[docs/architecture/cloud-ai-service.md]`
- [ ] **P0** 精细化 Arena 进度持久化 Schema（groupId, roundNumber, pairIndex） `[docs/architecture/state-persistence.md]`
- [ ] **P0** 水印设置增加"摄影师名称"自定义文本字段 `[docs/interaction/watermark-styles.md]`

#### Phase 2 — Competitive Parity（Weeks 4-8）

- [ ] **P1** 同步缩放对比模式（双照片同步放大同一区域） `[docs/interaction/arena-pk-design.md]`
- [ ] **P1** 离线 Speed Mode 完整实现（设备端 pHash + ORB + Laplacian） `[docs/harmony-app/harmony-tech-stack.md]`
- [ ] **P1** 偏好向量扩展至 7 维（增加色彩/构图/景深/表情） `[docs/ai-engine/preference-learning.md]`
- [ ] **P1** 偏好可视化仪表盘「我的审美档案」 `[docs/ai-engine/preference-learning.md]`
- [ ] **P1** 正面检测前置——解决背影/侧面闭眼误杀 `[docs/ai-engine/quality-scoring.md]`
- [ ] **P1** 自适应时间阈值分组（基于连拍频率自动调整） `[docs/ai-engine/multi-signal-grouping.md]`
- [ ] **P1** 后台处理 + 推送通知（非阻塞化 UI） `[docs/architecture/cloud-ai-service.md]`
- [ ] **P1** 预筛选"胶片扫描"快捷模式（横向滑动一动作一照片） `[docs/interaction/prescreen-ux.md]`
- [ ] **P1** Arena 触觉反馈（鸿蒙 vibrator API） `[docs/interaction/arena-pk-design.md]`
- [ ] **P1** 可见的 Undo 按钮（决策后底部浮现 3 秒） `[docs/interaction/arena-pk-design.md]`

#### Phase 3 — Market Differentiation（Month 2-3）

- [ ] **P2** 场景类型自动检测 + 权重预设（DINOv2 零样本分类） `[docs/ai-engine/quality-scoring.md]`
- [ ] **P2** 合影主次人脸分级惩罚机制 `[docs/ai-engine/quality-scoring.md]`
- [ ] **P2** LLM 评判一致性二次校验（低置信度照片自动重判） `[docs/ai-engine/llm-judge-prompt.md]`
- [ ] **P2** 冠军合集可分享拼图（3×3/4×4 + 品牌 + 一键社交分享） `[docs/interaction/watermark-styles.md]`
- [ ] **P2** 偏好档案导出 + 团队同步（二维码/链接分享） `[docs/ai-engine/preference-learning.md]`
- [ ] **P2** 从竞品迁移向导（Aftershoot 模式映射为偏好初始值） `[docs/interaction/prescreen-ux.md]`
- [ ] **P2** 周报摘要 + 个人基准线 + 连续使用 Streak `[docs/ai-engine/preference-learning.md]`
- [ ] **P2** 跨平台 Web App（PWA，复用 FastAPI 后端） `[docs/harmony-app/harmonyos-strategy.md]`

#### Phase 4 — Lock-In & Growth（Month 3+）

- [ ] **P3** 客户人脸记忆跨会话（InsightFace 向量 + 昵称） `[docs/ai-engine/preference-learning.md]`
- [ ] **P3** 推荐系统 + 偏好档案共享获客（双方 Tycoon 免费额度） `[docs/market/competitive-analysis.md]`
- [ ] **P3** 水印全分辨率预览 + Before/After 滑块 `[docs/interaction/watermark-styles.md]`
- [ ] **P3** 主动通知"新照片待选片"（相册变更监听） `[docs/harmony-app/harmony-tech-stack.md]`
- [ ] **P3** SD 卡直读 + 现场选片流程（USB-C OTG） `[docs/harmony-app/harmony-tech-stack.md]`

---

## Strategic Insights（战略级洞察）

### 洞察 1: Arena 是精修工具，不是主流程

Arena A/B 对决是 Shot Flow 最独特的创新，也是最大的 UX 陷阱。对 20-50 张人像拍摄，Arena 很有趣且高效。对 3000 张婚礼拍摄，强制 Arena = 比竞品更慢。

**建议**：Arena 定位为「AI 自动选片后的精修环节」——先让 AI 自动筛选，再把 AI 不确定的边界案例放进 Arena 让人做决定。这既保留了创新体验，又解决了高量级场景的效率问题。

### 洞察 2: 偏好学习是真正的护城河

Aftershoot 用户最大的抱怨是「2 年了 AI 还是乱选」。Shot Flow 的偏好学习是竞品完全缺失的特性。但要发挥锁定效应，必须：
1. **可视化**——让用户「看见」自己的审美在成长
2. **可分享**——团队/师徒间共享偏好档案
3. **可累积**——个人摄影数据日志，越用越离不开

### 洞察 3: 鸿蒙先发 ≠ 成功保障

鸿蒙应用市场确实没有 AI 选片工具，但「蓝海里没有鱼」的问题不能忽视。专业摄影师的主流工作流是 MacBook/iPad + Lightroom/Capture One。

**务实建议**：
- 鸿蒙版作为首发入口（拿激励奖金）
- **同期开发 Web App**（PWA，复用同一 FastAPI 后端），覆盖桌面/iPad 用户
- 核心竞争力（偏好学习、Arena 精修）是平台无关的

### 洞察 4: AI 引擎过度设计，UX 严重欠设计

14 份设计文档中，AI 引擎相关（多信号融合、质量评分、LLM 提示工程）设计精细到算法参数级别。但用户体验层（Arena 动画时长、进度指示、恢复机制、离线容错）描述停留在功能层面——"做了什么"而非"用了 200 次后什么感受"。

**建议**：每条 UX 规范补充「高频使用场景测试」——假设用户每周用 3 次，每次处理 200 张照片，200 次决策后哪里会烦？哪里会困惑？

---

## Appendix

### Raw Analysis: Competitor Power User Perspective（Agent A 完整输出）

> "Bottom line up front: 当前形式下，没有什么能让我从 Aftershoot 切换过来。Shot Flow 在为错误的平台解决错误的问题。但确实存在真正有吸引力的路径——只是需要做出艰难的战略选择。"
>
> **5 条核心建议（按优先级）：**
> 1. 为你的用户所在平台而构建。纯鸿蒙 = 爱好项目，不是产品。立即增加 Web App 或桌面伴侣。
> 2. 支持 RAW 文件。专业摄影师不拍 JPEG。
> 3. 在 Arena 旁边增加 AI Auto-Select 模式。Arena 是好精修工具，却是糟糕的主流程。
> 4. 建立 XMP/Lightroom 桥梁。选片结果必须流入编辑工作流。
> 5. All-in 偏好学习作为你的身份。这是唯一竞品没有的东西。让它可见、可验证、可分享。
>
> **锁定机会**：偏好可视化仪表盘、客户专属配置（人脸+审美偏好）、团队协同、Arena 赛程图社交分享、摄影分析数据日志。
>
> **致命缺陷**：鸿蒙独占、无 RAW、全量云端依赖、Arena 强制模式、无下游工作流。

### Raw Analysis: Your Product Power User Perspective（Agent B 完整输出）

> "6 个月每日使用后最大的感受：AI 引擎精巧但 UX 层欠设计。最大的改进不是更聪明的算法，而是更流畅的工作流。"
>
> **5 个"立即发布"建议（Low Effort, High Impact）：**
> 1. Quick Win AI 接受按钮（省 40%+ 滑动）
> 2. Arena 动画 500ms→150ms（改一个常量）
> 3. 首页恢复卡片（防止最大挫折——丢失进度）
> 4. 云处理进度条（把"冻结 App"变"透明过程"）
> 5. 水印自定义文本（一个设置字段，让水印从相机焦点变摄影师焦点）
>
> **速度瓶颈**：后台处理+推送、批量水印+进度、离线 Speed Mode、精细化 Arena 状态持久化。
>
> **AI 质量问题**：固定时间阈值分组不适配不同连拍速率、背影闭眼误杀、偏好向量 3 维饱和、合影人脸惩罚过重、LLM 评判一致性。
>
> **粘性建议**：周报摘要+基准线、冠军合集可分享拼图、偏好档案团队同步、客户人脸记忆、主动通知。

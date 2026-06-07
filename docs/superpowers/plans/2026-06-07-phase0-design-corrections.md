# Phase 0: 设计修正 — 编码前文档级修正

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在编码开始前修正 5 份设计文档中的结构性缺陷，使 MVP 开发不会建立在错误假设上。

**Architecture:** 纯文档修改任务，不涉及代码。每个 Task 对应一份设计文档的修改，修改内容来自 Product Killshot 分析报告的 P0 优先级发现。所有修改以「补充」为主（新增章节），不删除现有设计。

**Tech Stack:** Markdown 文档编辑

---

## File Structure

| 文件 | 操作 | 职责 |
|------|------|------|
| `docs/interaction/arena-pk-design.md` | **修改** | 新增混合工作流模式章节（AI Auto-Select / Arena / Review-Only） |
| `docs/harmony-app/harmony-tech-stack.md` | **修改** | 新增 Speed Mode 本地化实现路径 + RAW/DNG 缩略图支持方案 |
| `docs/market/competitive-analysis.md` | **修改** | 修改免费版策略（取消 100 张限制 → 功能限制）+ 新增跨平台路线 |
| `docs/harmony-app/mvp-scope.md` | **修改** | 更新 P0 功能表（新增混合模式、RAW 支持、XMP 导出）+ 调整验收标准 |
| `docs/architecture/state-persistence.md` | **修改** | 新增精细化 Arena 进度持久化 Schema + XMP 侧边栏导出方案 |
| `docs/product-killshot-2026-06-07.md` | **只读参考** | Killshot 报告，提供修改理由 |

---

### Task 1: arena-pk-design.md — 新增混合工作流模式

**Files:**
- Modify: `docs/interaction/arena-pk-design.md`

**背景:** 当前设计只有 Arena 淘汰赛一种模式。对 3000 张婚礼，强制 Arena = 1500+ 次滑动 ≈ 75 分钟，比 Aftershoot 的 40 分钟更慢。Arena 是精修工具，不应是唯一工作流。

- [ ] **Step 1: 在"为什么擂台模式更好"章节之后，新增"工作流模式"章节**

在 `## 为什么擂台模式更好` 结尾（第 14 行）之后，`## 交互设计` 之前，插入：

```markdown
## 工作流模式（三种）

> **核心洞察**：擂台 A/B 对决是独创交互，但不适合作为唯一工作流。
> 高量级场景（婚礼 2000+ 张）下，强制 Arena 比竞品更慢。
> 解决方案：三种模式按需选择，Arena 作为精修工具而非主流程。

### 模式一：AI 自动选片 (Auto-Select)

**适用场景**：高量级拍摄（婚礼、活动、体育），用户信任 AI 做首轮筛选。

**流程**：
1. AI 预筛选（淘汰技术问题照片）→ 用户复核
2. AI 自动从每组中选出综合评分最高的 1 张（含偏好加权）
3. 用户以网格模式浏览 AI 选出的冠军照片
4. **可选**：对 AI 不确定的边界案例（top-2 分差 < 0.3）进入 Arena 精修

**交互**：
```
┌──────────────────────────────────┐
│  AI 自动选片结果                   │
│  共 35 组，AI 已选出 35 张          │
│                                   │
│  ✓ 28 张高置信选择（分差 > 0.3）   │
│  ? 7 张边界案例（需确认）          │
│                                   │
│  [ 一键接受全部 ]  [ 逐张审核 ]     │
│  [ 进入 Arena 精修边界案例 → ]      │
└──────────────────────────────────┘
```

**关键参数**：
- AI 置信度阈值：`confidence_gap > 0.3` 视为高置信，自动通过
- 边界案例：`0.1 < confidence_gap < 0.3`，标记为"需确认"
- 分差极小：`confidence_gap < 0.1`，强制进入 Arena

### 模式二：Arena 擂台淘汰（现有设计）

**适用场景**：中小量级（50-300 张），用户喜欢逐对比较的参与感。

（保持现有 `## 交互设计` 以下全部内容不变）

### 模式三：纯网格浏览 (Review-Only)

**适用场景**：用户想快速浏览全部照片，自行标记保留/淘汰。

**流程**：
1. AI 预筛选 + 分组（同其他模式）
2. 以网格模式展示分组后的照片
3. 用户手动标记 Pick / Reject
4. 支持组内星级评分

**交互**：
```
┌──────────────────────────────────┐
│  第 1 组（8 张）         [全部保留]│
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐       │
│  │ ✓ │ │ ✓ │ │ ✗ │ │ ✓ │       │
│  └───┘ └───┘ └───┘ └───┘       │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐       │
│  │ ✓ │ │ ✓ │ │ ✗ │ │ ✗ │       │
│  └───┘ └───┘ └───┘ └───┘       │
│  [ ✓ ] Pick  [ ✗ ] Reject  [★] Rating │
└──────────────────────────────────┘
```

### 模式选择 UI

在"开始分析"之后、"进入选片"之前，显示模式选择：

```
┌──────────────────────────────────┐
│       选择选片方式                 │
│                                   │
│  ┌──────────────────────────┐    │
│  │ ⚡ AI 自动选片             │    │
│  │ AI 选最佳，你审核          │    │
│  │ 适合 200+ 张              │    │
│  └──────────────────────────┘    │
│                                   │
│  ┌──────────────────────────┐    │
│  │ 🏟️ Arena 对决 (推荐)      │    │
│  │ 逐对 PK，选出冠军          │    │
│  │ 适合 50-200 张            │    │
│  └──────────────────────────┘    │
│                                   │
│  ┌──────────────────────────┐    │
│  │ 📋 手动浏览               │    │
│  │ 网格模式，自己决定         │    │
│  │ 适合少量精选              │    │
│  └──────────────────────────┘    │
│                                   │
│  [ 记住我的选择 ]                  │
└──────────────────────────────────┘
```

- 记住上次选择到 Preferences
- 根据照片数量智能推荐：> 200 张推荐 Auto-Select，50-200 推荐 Arena，< 50 推荐 Review-Only
```

- [ ] **Step 2: 在手势系统表格中补充"Quick Win"手势**

在 `## 手势系统` 表格中（第 34-44 行），在 `| 滑动 ≥ 50dp 松手 | 确认选择 |` 之后追加一行：

```markdown
| 底部「AI 推荐」按钮 | 当组内 AI 置信度差距大时，一键接受 AI 推荐，跳过本组 Arena |
```

- [ ] **Step 3: 在竞技场配对算法章节末尾补充"智能跳过"逻辑**

在 `## 竞技场配对算法` 章节（第 102-108 行）末尾追加：

```markdown
### 智能跳过（Quick Win）

当某组满足以下条件时，Arena 页面顶部显示「AI 推荐」按钮：
- 组内照片数 ≤ 3 张
- 最高分与次高分差距 > 1.5（10 分制）
- 最低分照片 fatal flag 数量 ≥ 1

用户点击「AI 推荐」→ 直接采用 AI 最高分照片作为冠军，跳过该组全部对决。
此决策记入撤销栈，可撤销后重新进入 Arena。
```

- [ ] **Step 4: 验证修改完整性**

Run: `grep -n "工作流模式\|三种\|Quick Win\|AI 自动选片\|Review-Only\|Auto-Select" docs/interaction/arena-pk-design.md`
Expected: 至少 8 行匹配，覆盖新增的三个模式名称和 Quick Win 功能

- [ ] **Step 5: Commit**

```bash
git add docs/interaction/arena-pk-design.md
git commit -m "docs: 新增三种工作流模式（Auto-Select/Arena/Review-Only）和 Quick Win 机制

- AI Auto-Select: AI 自动选最佳，用户审核，适合高量级
- Arena Mode: 现有淘汰赛，适合中等量级
- Review-Only: 纯网格浏览，适合少量精选
- Quick Win: AI 置信度差距大时一键接受推荐
- 智能模式推荐（根据照片数量）"
```

---

### Task 2: harmony-tech-stack.md — Speed Mode 本地化 + RAW 支持

**Files:**
- Modify: `docs/harmony-app/harmony-tech-stack.md`

**背景:** 当前 Speed Mode（免费层）描述为"鸿蒙端本地实现"但未给实现方案，且所有 AI 处理实际走云端 API。RAW 文件标记为"首版不支持"。这两项在竞品分析中均为致命缺陷。

- [ ] **Step 1: 修改功能映射表中的 RAW 支持策略**

在 `## Web 版 → 鸿蒙端功能映射` 表格中（第 7-18 行），将 `| RAW 支持 | **首版不支持** | 后续用 \`image.createImageSource()\` 扩展 |` 替换为：

```markdown
| RAW 支持 | **MVP 支持 DNG + 嵌入式预览** | DNG 直读 + CR3/NEF/ARW 嵌入 JPEG 预览提取 |
```

- [ ] **Step 2: 在功能映射表之后，新增 Speed Mode 本地化实现章节**

在功能映射表（第 18 行）之后、`## 鸿蒙端关键 API 速查` 之前，插入：

```markdown
## Speed Mode 本地化实现方案

> **核心原则**：Speed Mode 是唯一免费的、完全离线的处理模式。
> 它不能是"最差的模式"——它是用户信任建立的第一步。

### 算法清单（全部设备端运行）

| 算法 | 用途 | ArkTS 实现复杂度 | 参考实现 |
|------|------|-----------------|---------|
| **感知哈希 (pHash)** | 相似照片检测 + 分组 | 中（DCT 变换） | 纯 ArkTS 实现，无需原生模块 |
| **ORB 特征匹配** | 精确相似度验证 | 高（建议 NAPI） | OpenCV 算法移植为 C NAPI |
| **HSV 直方图** | 色彩相似度 | 低（像素遍历） | 纯 ArkTS |
| **Laplacian 方差** | 模糊检测 | 低（卷积运算） | 纯 ArkTS |
| **时间戳分组** | 连拍序列识别 | 极低（排序 + 差值） | EXIF 读取 + 排序 |

### 实现优先级

```
Phase 1 (MVP)：
  ✓ 时间戳分组  — 必须有，最简单，最可靠
  ✓ HSV 直方图  — 必须有，色彩分组基础
  ✓ Laplacian 方差 — 必须有，模糊检测基础
  ✓ pHash       — 必须有，相似检测核心

Phase 2 (优化)：
  ○ ORB 特征匹配 — 精确验证，可替代为 pHash 足够场景
```

### 性能预估（鸿蒙手机）

| 操作 | 200 张照片预计耗时 |
|------|-------------------|
| EXIF 读取 + 时间戳分组 | < 1 秒 |
| HSV 直方图计算 | ~5 秒 |
| pHash 计算 | ~10 秒 |
| Laplacian 模糊检测 | ~5 秒 |
| **总计（Speed Mode 全流程）** | **~20 秒** |

### 与 Expert Mode 的能力对比

| 能力 | Speed Mode | Expert Mode |
|------|-----------|-------------|
| 模糊检测 | Laplacian 方差（基础） | NIMA + MUSIQ + CLIP-IQA（精准） |
| 人脸检测 | ❌ | InsightFace（精准） |
| 闭眼检测 | ❌ | InsightFace EAR（精准） |
| 美学评分 | HSV 直方图粗估 | NIMA 深度模型（精准） |
| 相似分组 | pHash + 时间戳 | DINOv2 语义 + 8 信号融合 |
| 分组准确率目标 | ~75% | ~90%+ |
| 预筛选准确率目标 | ~70%（仅模糊/曝光） | ~85%+ |
| 离线可用 | ✅ | ❌ |
| 速度 | ~20 秒/200 张 | ~40 秒/200 张（含网络） |
| 费用 | 免费 | Pro 订阅 |

### Worker 线程使用

```typescript
import worker from '@ohos.worker';

// Speed Mode 计算在 Worker 线程执行，不阻塞 UI
const speedWorker = new worker.ThreadWorker('workers/SpeedModeWorker.ts');

speedWorker.onmessage = (event) => {
  const { type, progress, result } = event.data;
  if (type === 'progress') {
    this.progressPercent = progress; // 更新 UI 进度条
  }
  if (type === 'complete') {
    this.handleSpeedModeResult(result);
  }
};

// 发送任务
speedWorker.postMessage({
  photos: this.selectedPhotos,
  algorithms: ['timestamp', 'hsv', 'phash', 'laplacian']
});
```

## RAW 文件支持方案

### MVP 策略：DNG 直读 + 嵌入式预览

专业相机 RAW 文件（CR3/NEF/ARW/RAF 等）内部通常嵌入一张全尺寸 JPEG 预览图。
MVP 阶段不需要解码 RAW 数据——提取这张嵌入式预览即可用于选片。

### 支持矩阵

| 格式 | MVP 策略 | 实现方式 |
|------|---------|---------|
| **JPEG** | ✅ 完整支持 | PhotoViewPicker 直接支持 |
| **HEIF** | ✅ 完整支持 | PhotoViewPicker 直接支持 |
| **PNG** | ✅ 完整支持 | PhotoViewPicker 直接支持 |
| **DNG** | ✅ 预览提取 | `image.createImageSource()` 直读 |
| **CR3 (Canon)** | 🔶 嵌入预览 | 解析 TIFF/CR3 结构提取 PreviewImage |
| **NEF (Nikon)** | 🔶 嵌入预览 | 解析 TIFF/NEF 结构提取嵌入 JPEG |
| **ARW (Sony)** | 🔶 嵌入预览 | 解析 TIFF 结构提取嵌入 JPEG |
| **RAF (Fujifilm)** | 🔶 嵌入预览 | 解析 RAF 结构提取嵌入 JPEG |

> ✅ = MVP 支持  🔶 = P1 支持（需要原生模块解析 RAW 结构）

### 嵌入式预览提取思路

```typescript
// 方法 1：鸿蒙 image API 直接读取（DNG 有效）
const imageSource = image.createImageSource(rawFilePath);
const pixelMap = await imageSource.createPixelMap();

// 方法 2：对 CR3/NEF/ARW，需要解析文件结构提取嵌入 JPEG
// RAW 文件是 TIFF 结构，内含多张 IFD（Image File Directory）
// 最后一个 IFD 通常是全尺寸 JPEG 预览
// 实现：NAPI C 模块读取 TIFF IFD 定位嵌入 JPEG 的偏移和长度
```

### 导入流程调整

```typescript
// PhotoPicker 选择时同时接受 RAW 格式
const picker = new photoAccessHelper.PhotoViewPicker();
const result = await picker.select({
  MIMETypes: ['image/jpeg', 'image/png', 'image/heif', 'image/dng'],
  maxSelectNumber: 500  // 免费版也取消数量限制（见 Task 3）
});

// 对 RAW 文件：自动提取预览用于选片，但记住原始路径用于最终导出
for (const uri of result.photoUris) {
  const mimeType = await getMimeType(uri);
  if (isRawFormat(mimeType)) {
    const preview = await extractEmbeddedPreview(uri);
    this.photos.push({ originalUri: uri, preview, isRaw: true });
  } else {
    this.photos.push({ originalUri: uri, preview: uri, isRaw: false });
  }
}
```
```

- [ ] **Step 2: 修改照片访问代码示例，支持更多格式**

将 `## 鸿蒙端关键 API 速查` 下的照片访问代码示例（第 29-32 行）中的 `MIMETypes` 修改：

```typescript
// 选择照片（无需权限）
import { photoAccessHelper } from '@kit.MediaLibraryKit';

const picker = new photoAccessHelper.PhotoViewPicker();
const result = await picker.select({
  MIMETypes: ['image/jpeg', 'image/png', 'image/heif', 'image/dng'],
  maxSelectNumber: 500
});
```

- [ ] **Step 3: 验证修改完整性**

Run: `grep -n "Speed Mode\|RAW\|pHash\|DNG\|Worker\|嵌入式预览\|嵌入 JPEG" docs/harmony-app/harmony-tech-stack.md`
Expected: 至少 15 行匹配

- [ ] **Step 4: Commit**

```bash
git add docs/harmony-app/harmony-tech-stack.md
git commit -m "docs: Speed Mode 本地化实现方案 + RAW 文件支持策略

- Speed Mode 5 个算法全部设备端运行，目标 20 秒/200 张
- Worker 线程方案避免阻塞 UI
- RAW 支持：MVP 支持 DNG 直读 + CR3/NEF/ARW 嵌入预览提取
- PhotoPicker 扩展支持 DNG MIME 类型
- 免费版数量上限提升至 500 张"
```

---

### Task 3: competitive-analysis.md — 免费版策略 + 跨平台路线

**Files:**
- Modify: `docs/market/competitive-analysis.md`

**背景:** 当前免费版限制 100 张，但在竞品分析中发现这个限制阻碍用户评估。Aftershoot 30 天无限制试用。100 张不够一场婚礼。同时，纯鸿蒙策略在专业摄影市场缺少目标用户。

- [ ] **Step 1: 修改定价策略表格中的 Free 层描述**

将 `| **Free** | ¥0 | 极速模式、基础水印、单次 ≤ 100 张 |` 替换为：

```markdown
| **Free** | ¥0 | 极速模式（本地 AI）、基础水印、无数量限制 |
```

- [ ] **Step 2: 在定价策略表格之后，新增免费版策略说明**

在 `**注意**：桌面端免费...` 段落（第 69-72 行）之后，追加：

```markdown
### 免费版策略：功能限制而非数量限制

**旧策略（已废弃）**：Free 层 ≤ 100 张/次
**新策略**：Free 层无数量限制，仅 Speed Mode 可用

**理由**：
1. **评估友好**：100 张不够一场婚礼的评估。Aftershoot 提 30 天无限制试用，用户需要体验完整流程才能判断价值
2. **Speed Mode 有价值**：本地 pHash + Laplacian + HSV 已经能提供 75% 分组准确率和基础模糊检测。这不是"残废版"，是"入门版"
3. **降低服务器成本**：Speed Mode 全部本地计算，无限量不增加云端成本
4. **自然升级路径**：用户从 Speed Mode 获得价值 → 需要人脸检测/闭眼检测/精准美学评分 → 升级 Expert Mode

**对比竞品试用策略**：

| 竞品 | 免费策略 |
|------|---------|
| Aftershoot | 30 天无限制试用，之后付费 |
| Narrative Select | 免费试用，无需信用卡 |
| FilterPixel | 4 个免费项目，无需信用卡 |
| **Shot Flow** | **永久免费 Speed Mode + 付费 Expert/Tycoon** |

永久免费 + 功能限制 比 限时试用 更适合移动端市场：
- 移动用户对"试用期过了"更敏感
- Speed Mode 不断产生价值 → Expert Mode 是"锦上添花"而非"解锁必需"
- 偏好学习在 Speed Mode 中也会累积 → 用户不会因免费而失去锁定效应
```

- [ ] **Step 3: 在 SWOT 分析表格之后，新增"跨平台路线"章节**

在 SWOT 分析（第 55-58 行）之后，`## 定价策略建议` 之前，插入：

```markdown
## 跨平台路线

> **战略洞察**：鸿蒙先发 ≠ 成功保障。专业摄影师主流工作流是 MacBook/iPad + Lightroom。
> 鸿蒙生态是首发入口（拿激励奖金 + 先发优势），但核心能力（偏好学习、Arena 精修）
> 应设计为平台无关。

### 路线图

```
2026.06-09 ─ 鸿蒙原生 App（首发）
              │
              │ FastAPI 后端已天然跨平台
              │
2026.10-12 ─ Web App (PWA)
              │ 复用同一后端 API
              │ 覆盖桌面端/iPad 用户
              │ Arena 交互适配鼠标/触控板
              │
2027.Q1   ─ 数据同步
              │ 偏好档案云端同步
              │ 鸿蒙 App ↔ Web App 数据打通
              │
远期       ─ iOS App（如市场验证成功）
```

### Web App (PWA) 关键决策

| 维度 | 决策 |
|------|------|
| 框架 | Vue 3 + Vite（轻量、构建快、生态成熟）|
| UI | Tailwind CSS（快速迭代，适配桌面宽屏）|
| API | 复用现有 FastAPI 后端，零修改 |
| Arena 交互 | 鼠标拖拽 + 键盘 ←/→ 适配 |
| 离线 | Speed Mode 通过 WASM 实现（pHash/Laplacian 可编译为 WASM）|
| 预估工期 | MVP 4-6 周（复用后端 + 设计文档）|

### 为跨平台设计的架构原则

1. **后端 API 不感知客户端类型**：所有端（鸿蒙/Web/iOS）调用相同 REST API
2. **偏好数据存云端**：按 user_id 索引，跨端同步
3. **Arena 决策数据格式统一**：JSON schema 跨平台通用
4. **XMP 导出为标准格式**：任何端都能生成 Lightroom 可读的元数据
```

- [ ] **Step 4: 验证修改完整性**

Run: `grep -n "无数量限制\|功能限制\|跨平台\|PWA\|Web App\|WASM" docs/market/competitive-analysis.md`
Expected: 至少 10 行匹配

- [ ] **Step 5: Commit**

```bash
git add docs/market/competitive-analysis.md
git commit -m "docs: 免费版取消数量限制改为功能限制 + 新增跨平台路线

- Free 层：永久免费 Speed Mode，无数量限制
- 理由：100 张不够评估；Speed Mode 本地运行无服务器成本
- 新增跨平台路线：鸿蒙首发 → Web App (PWA) → 数据同步
- Web App 复用 FastAPI 后端，预估 4-6 周 MVP"
```

---

### Task 4: mvp-scope.md — 更新 P0 功能表和验收标准

**Files:**
- Modify: `docs/harmony-app/mvp-scope.md`

**背景:** 之前三个 Task 的设计修正需要反映到 MVP 功能表和验收标准中。混合模式、RAW 支持、XMP 导出、Speed Mode 离线化都是 P0 级别，需要体现在功能列表和时间线中。

- [ ] **Step 1: 修改 P0 功能表，新增和更新条目**

将 P0 功能表（第 8-19 行）替换为：

```markdown
| # | 功能 | 鸿蒙实现 | 工期估算 |
|---|------|---------|---------|
| 1 | **相册导入**（含 DNG/HEIF） | PhotoPicker + photoAccessHelper + RAW 预览提取 | 1.5 周 |
| 2 | **Speed Mode 本地 AI 初筛** | 设备端 pHash + Laplacian + HSV（Worker 线程） | 1.5 周 |
| 3 | **云端 AI 初筛（Expert Mode）** | 上传缩略图 → 云端 quality.py → 返回拒绝列表 | 1 周 |
| 4 | **初筛复核页** | Grid 展示被拒照片 + 按原因分组 + 点击/批量恢复 | 1 周 |
| 5 | **Speed Mode 本地分组** | 设备端 pHash + 时间戳 + HSV 聚类 | 1 周 |
| 6 | **云端 AI 分组（Expert Mode）** | 上传特征 → 云端 clustering.py → 返回分组结果 | 1 周 |
| 7 | **混合工作流模式选择** | AI Auto-Select / Arena PK / Review-Only 三模式 UI | 0.5 周 |
| 8 | **A/B 擂台 PK** | Swiper/Stack 左右对比 + 手势 ←/→ + Quick Win | 2 周 |
| 9 | **Winners/Losers 导出 + XMP** | fs 复制 + XMP 侧边栏写入（星级、Pick/Reject） | 1 周 |
| 10 | **水印渲染（2-3 种样式）** | Canvas 绘制 + 自定义摄影师名称 + ImagePacker 导出 | 1.5 周 |
| 11 | **进度保存/恢复** | Preferences + JSON（含精细化 Arena 进度） | 0.5 周 |
| 12 | **设置页**（模式选择/初筛力度/摄影师名称） | 基础表单 | 0.5 周 |
| 13 | **云端服务搭建** | FastAPI + 部署 | 1 周 |
|    | **小计** | | **~14 周（3.5 个月）** |
```

- [ ] **Step 2: 更新 P1 功能表**

将 P1 功能表（第 24-29 行）替换为：

```markdown
| # | 功能 | 说明 | 工期估算 |
|---|------|------|---------|
| 14 | **星级评分**（Arena 中） | 擂台时 1-5 星评级，写入 XMP | 0.5 周 |
| 15 | **EXIF 信息面板** | 擂台页显示镜头/焦距/光圈/ISO | 0.5 周 |
| 16 | **Dark Mode** | ArkUI 系统级主题切换 | 0.5 周 |
| 17 | **最近使用记录 + 快速恢复** | Preferences 存历史 + 首页恢复卡片 | 0.5 周 |
| 18 | **偏好学习（本地累积）** | 云端返回偏好权重 + 本地持久化 | 1 周 |
| 19 | **RAW 完整支持（CR3/NEF/ARW）** | NAPI 模块解析 RAW 提取嵌入预览 | 1.5 周 |
|    | **小计** | | **~4.5 周** |
```

- [ ] **Step 3: 更新时间线**

将时间线总览（第 38-48 行）替换为：

```markdown
## 时间线总览

```
6月7日 ────────────────────────────────────────────────────── 10月中
  │←── MVP Core (14 周) ──→│← P1 打磨 (4.5 周) →│← 缓冲/审核 →│
  │                          │                      │              │
  W1-W14                    W15-W19                W20           提交
```

> ⚠️ **注意**：MVP 工期从 11 周调整为 14 周。新增内容：Speed Mode 本地化（+1.5 周）、
> 混合工作流模式（+0.5 周）、XMP 导出（+0.5 周）、RAW 预览提取（+0.5 周）。
> 若需维持 9 月底提交，建议 P0 功能按批次交付：先交付 Speed Mode 全流程 → 再叠加 Expert Mode。
```

- [ ] **Step 4: 更新逐周行动清单**

将逐周行动清单（第 50-87 行）替换为：

```markdown
## 逐周行动清单

### Week 1（6月7日-6月13日）
- [ ] 注册华为开发者联盟账号 + 实名认证
- [ ] 报名鸿蒙应用开发者激励计划 2026
- [ ] 安装 DevEco Studio + 创建项目骨架
- [ ] 搭建云端 FastAPI 服务（Hello World + 图片上传）
- [ ] 实现照片选择功能（PhotoPicker + DNG/HEIF 支持）

### Week 2-3（6月14日-6月27日）
- [ ] **Speed Mode 本地算法**：实现 pHash + Laplacian + HSV（Worker 线程）
- [ ] Speed Mode 本地初筛（仅模糊/曝光检测，无需云端）
- [ ] Speed Mode 本地分组（pHash + 时间戳聚类）
- [ ] 初筛复核页 UI（按原因分组 + 批量恢复）

### Week 4-5（6月28日-7月11日）
- [ ] 云端 AI 初筛 API（移植 `quality.py` 核心逻辑）
- [ ] 云端 AI 分组 API（移植 `clustering.py` 核心逻辑）
- [ ] Speed Mode ↔ Expert Mode 切换逻辑

### Week 6-7（7月12日-7月25日）
- [ ] **工作流模式选择 UI**（Auto-Select / Arena / Review-Only）
- [ ] AI Auto-Select 模式：AI 选最佳 + 用户审核边界案例
- [ ] Review-Only 模式：网格浏览 + Pick/Reject

### Week 8-10（7月26日-8月15日）
- [ ] **A/B 擂台 PK 核心 UI**（最关键的功能，最需要打磨）
- [ ] 手势系统（←/→/↑/↓ + PinchGesture 缩放 + Quick Win）
- [ ] 状态管理（分组数据、擂台进度、撤销栈 + 精细化持久化）
- [ ] Arena 动画默认 150ms（非 500ms）

### Week 11-12（8月16日-8月29日）
- [ ] Winners/Losers 文件导出
- [ ] XMP 侧边栏写入（星级 + Pick/Reject 标记）
- [ ] 水印渲染（2-3 种样式 + 自定义摄影师名称）

### Week 13-14（8月30日-9月12日）
- [ ] 进度保存/恢复（含精细化 Arena 进度）
- [ ] 设置页（模式选择/初筛力度/摄影师名称）
- [ ] 整体集成测试

### Week 15-19（9月13日-10月17日）— P1 打磨
- [ ] EXIF 信息面板 + 星级评分
- [ ] 偏好学习（本地 + 云端）
- [ ] Dark Mode
- [ ] 最近使用记录 + 首页恢复卡片
- [ ] RAW 完整支持（CR3/NEF/ARW NAPI 模块）

### Week 20（10月18日-10月24日）
- [ ] 提交华为应用市场审核
- [ ] 根据审核反馈调整
```

- [ ] **Step 5: 更新验收标准**

将验收标准（第 89-105 行）替换为：

```markdown
## MVP 验收标准

### 功能验收
- [ ] 能从相册选择照片（支持 JPEG/HEIF/PNG/DNG，≥500 张）
- [ ] Speed Mode 全流程离线可用（本地初筛 + 本地分组）
- [ ] Speed Mode 分组准确率 ≥ 75%（同一连拍序列）
- [ ] 云端 AI 初筛准确率 ≥ 85%（模糊/曝光/闭眼）
- [ ] 云端 AI 分组正确率 ≥ 90%
- [ ] 三种工作流模式可用（Auto-Select / Arena / Review-Only）
- [ ] Arena 擂台 PK 流畅度：左右切换 < 200ms，动画 ≤ 150ms
- [ ] Quick Win：AI 置信度差距大时一键接受
- [ ] 导出包含 XMP 侧边栏（星级 + Pick/Reject，Lightroom 可读）
- [ ] 水印渲染正确：至少 2 种样式 + 自定义摄影师名称
- [ ] 进度保存/恢复正常（含 Arena 精细化进度恢复）
- [ ] 无崩溃（Monkey 测试 1000 次操作）

### 上架验收
- [ ] HarmonyOS 5.0+ 兼容
- [ ] 华为应用市场审核通过
- [ ] 隐私政策 + 用户协议页面
- [ ] App 备案证明（中国大陆要求）
- [ ] 月活≥400（目标 3 个月内达到）
```

- [ ] **Step 6: 验证修改完整性**

Run: `grep -n "混合工作流\|Speed Mode\|Quick Win\|XMP\|DNG\|三种工作流\|Auto-Select\|Review-Only" docs/harmony-app/mvp-scope.md`
Expected: 至少 12 行匹配

- [ ] **Step 7: Commit**

```bash
git add docs/harmony-app/mvp-scope.md
git commit -m "docs: 更新 MVP 功能表反映 Phase 0 设计修正

P0 新增：Speed Mode 本地化、混合工作流模式、XMP 导出、RAW 预览提取
P1 新增：RAW 完整支持（CR3/NEF/ARW）
工期调整：MVP 11→14 周，总工期 15→20 周
验收标准更新：新增 Speed Mode 离线、三种模式、XMP、DNG 支持"
```

---

### Task 5: state-persistence.md — 精细化 Arena 进度 + XMP 导出

**Files:**
- Modify: `docs/architecture/state-persistence.md`

**背景:** 当前 Arena 进度只存 `arenaProgress` 字段但未定义具体 Schema，App 被杀后无法恢复到具体比较对。XMP 导出是专业工作流的"语言"，没有它选片结果就死在文件夹里。

- [ ] **Step 1: 在鸿蒙端本地状态 Schema 中补充精细化 Arena 进度定义**

在 `interface TaskState` 定义中（第 63-70 行），将 `arenaProgress: ArenaProgress` 保留，并在其下方新增精细化定义：

```markdown
### 精细化 Arena 进度 Schema

```typescript
interface ArenaProgress {
  // 当前活跃的工作流模式
  workflowMode: 'auto_select' | 'arena' | 'review_only'

  // Arena 模式专用
  arena?: {
    // 全局进度
    currentGroupIndex: number     // 当前处理到第几组（0-based）
    totalGroups: number           // 总组数

    // 当前组的锦标赛进度
    currentRound: number          // 当前第几轮（1-based，1=首轮）
    totalRounds: number           // 本组总轮数
    currentPairIndex: number      // 当前比较对在本轮中的索引

    // 当前比较对的照片
    currentPair: {
      leftPhotoUri: string        // 左侧照片 URI
      rightPhotoUri: string       // 右侧照片 URI
      leftScore: number           // AI 综合分（含偏好加权）
      rightScore: number
    }

    // 各组状态
    groups: ArenaGroupState[]
  }

  // Auto-Select 模式专用
  autoSelect?: {
    highConfidenceCount: number   // 高置信选择数
    borderlineCount: number       // 边界案例数
    userReviewedCount: number     // 用户已审核数
    aiAcceptedCount: number       // 一键接受数
  }

  // Review-Only 模式专用
  reviewOnly?: {
    pickedUris: string[]          // 用户标记 Pick 的照片
    rejectedUris: string[]        // 用户标记 Reject 的照片
    ratings: Record<string, number> // URI → 星级评分
  }
}

interface ArenaGroupState {
  groupId: string
  status: 'pending' | 'in_progress' | 'completed' | 'skipped'
  winner: string | null          // 冠军照片 URI
  skipReason?: 'quick_win' | 'user_skip'  // 跳过原因
  roundResults: {
    round: number
    pairs: {
      leftUri: string
      rightUri: string
      winnerUri: string
      loserUri: string
    }[]
  }[]
}
```

### 冷启动恢复逻辑

```typescript
// App 冷启动时检查是否有未完成任务
async restoreOnColdStart(): Promise<void> {
  const pref = await preferences.getPreferences(context, 'shot_flow');
  const saved = await pref.get('current_task', '');

  if (!saved) {
    // 无任务，显示首页
    this.showHomePage();
    return;
  }

  const task: TaskState = JSON.parse(saved);

  if (task.status === 'done') {
    // 任务已完成，显示结果页
    this.showResults(task);
    return;
  }

  // 有未完成任务，显示恢复横幅
  this.showResumeBanner({
    mode: task.arenaProgress.workflowMode,
    groupInfo: task.arenaProgress.arena
      ? `第 ${task.arenaProgress.arena.currentGroupIndex + 1}/${task.arenaProgress.arena.totalGroups} 组`
      : '',
    photoCount: task.photos.length,
    lastModified: task.lastModified
  });
}

// 恢复横幅 UI
@Builder
ResumeBanner(info: ResumeInfo) {
  Row() {
    Text(`继续上次：${info.mode} ${info.groupInfo}`)
      .fontSize(14)
    Button('恢复')
      .onClick(() => this.resumeTask())
    Button('放弃')
      .fontColor(Color.Red)
      .onClick(() => this.discardTask())
  }
  .width('100%')
  .padding(12)
  .backgroundColor('#FFF3E0')
  .borderRadius(8)
}
```
```

- [ ] **Step 2: 在偏好学习持久化章节之后，新增 XMP 导出章节**

在 `## 偏好学习的持久化` 章节（第 112 行）之后，文件末尾，追加：

```markdown
## XMP 侧边栏导出方案

> **背景**：选片结果必须能流入专业编辑工作流（Lightroom/Capture One）。
> XMP 侧边栏是行业标准的元数据交换格式。没有 XMP 导出，"winners/" 文件夹对专业用户毫无意义。

### XMP 侧边栏格式

```xml
<?xpacket begin="" id="W5M0MpCehiHzreSzNTczkc9d"?>
<x:xmpmeta xmlns:x="adobe:ns:meta/">
  <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
    <rdf:Description rdf:about=""
      xmlns:xmp="http://ns.adobe.com/xap/1.0/"
      xmlns:dc="http://purl.org/dc/elements/1.1/"
      xmlns:lr="http://ns.adobe.com/lightroom/1.0/"
      xmlns:stEvt="http://ns.adobe.com/xap/1.0/sType/ResourceEvent#"
      xmp:CreatorTool="Shot Flow/1.0"
      lr:pickLabel="{pickValue}"
      lr:colorLabel="{colorValue}">

      <!-- 星级评分 -->
      <xmp:Rating>{1-5}</xmp:Rating>

      <!-- Pick/Reject 标记：1=Pick, -1=Rejected, 0=Unflagged -->
      <lr:pickLabel>{1|-1|0}</lr:pickLabel>

      <!-- 色标（可选）：0=无, 1=红, 2=黄, 3=绿, 4=蓝, 5=紫, 6=品红 -->
      <lr:colorLabel>{0-6}</lr:colorLabel>

      <!-- 标签（Shot Flow 自定义） -->
      <dc:subject>
        <rdf:Bag>
          <rdf:li>ShotFlow</rdf:li>
          <rdf:li>Arena:{groupId}</rdf:li>
          <rdf:li>{champion|runner_up| eliminated}</rdf:li>
        </rdf:Bag>
      </dc:subject>
    </rdf:Description>
  </rdf:RDF>
</x:xmpmeta>
<?xpacket end="w"?>
```

### 写入规则

| 场景 | Rating | Pick | Color | Tags |
|------|--------|------|-------|------|
| 冠军照片 | 用户评分（默认 4） | 1 (Pick) | 无 | ShotFlow, Arena:{groupId}, champion |
| Arena 中被淘汰 | 无 | -1 (Rejected) | 无 | ShotFlow, Arena:{groupId}, eliminated |
| 预筛选被拒（用户未恢复） | 无 | -1 (Rejected) | 无 | ShotFlow, prescreen_rejected |
| 预筛选被拒（用户恢复后晋级） | 用户评分 | 1 (Pick) | 无 | ShotFlow, restored |
| Auto-Select 边界案例（用户确认保留） | 用户评分 | 1 (Pick) | 2 (黄) | ShotFlow, borderline_kept |

### 鸿蒙端实现

```typescript
// XMP 侧边栏文件路径：原文件名 + .xmp
function getXmpPath(photoUri: string): string {
  const basePath = photoUri.replace(/\.[^.]+$/, '');
  return `${basePath}.xmp`;
}

// 生成 XMP 内容
function generateXmp(options: XmpOptions): string {
  const pickValue = options.isPick ? '1' : options.isRejected ? '-1' : '0';
  const colorValue = String(options.colorLabel ?? 0);

  return `<?xpacket begin="" id="W5M0MpCehiHzreSzNTczkc9d"?>
<x:xmpmeta xmlns:x="adobe:ns:meta/">
  <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
    <rdf:Description rdf:about=""
      xmlns:xmp="http://ns.adobe.com/xap/1.0/"
      xmlns:lr="http://ns.adobe.com/lightroom/1.0/"
      xmp:CreatorTool="Shot Flow/1.0"
      lr:pickLabel="${pickValue}"
      lr:colorLabel="${colorValue}">
      <xmp:Rating>${options.rating ?? 0}</xmp:Rating>
      <dc:subject xmlns:dc="http://purl.org/dc/elements/1.1/">
        <rdf:Bag>
          <rdf:li>ShotFlow</rdf:li>
          ${options.groupId ? `<rdf:li>Arena:${options.groupId}</rdf:li>` : ''}
          ${options.tag ? `<rdf:li>${options.tag}</rdf:li>` : ''}
        </rdf:Bag>
      </dc:subject>
    </rdf:Description>
  </rdf:RDF>
</x:xmpmeta>
<?xpacket end="w"?>`;
}

// 批量导出
async exportXmpFiles(photos: ExportPhoto[]): Promise<void> {
  for (const photo of photos) {
    const xmpContent = generateXmp({
      isPick: photo.isWinner,
      isRejected: !photo.isWinner,
      rating: photo.rating,
      groupId: photo.groupId,
      tag: photo.isWinner ? 'champion' : 'eliminated'
    });

    const xmpPath = getXmpPath(photo.uri);
    // 原子写入：temp + rename
    const tempPath = `${xmpPath}.tmp`;
    await writeFile(tempPath, xmpContent);
    await renameFile(tempPath, xmpPath);
  }
}
```

### 兼容性验证清单

- [ ] Adobe Lightroom Classic：导入含 .xmp 侧边栏的照片 → 自动读取星级和 Pick 状态
- [ ] Adobe Lightroom CC (Cloud)：同上
- [ ] Capture One：读取 .xmp 侧边栏的星级和色标
- [ ] Adobe Bridge：读取全部 XMP 字段
- [ ] ExifTool：`exiftool -xmp:all photo.xmp` 验证字段完整性
```

- [ ] **Step 3: 验证修改完整性**

Run: `grep -n "ArenaProgress\|workflowMode\|currentPairIndex\|XMP\|xmpmeta\|pickLabel\|generateXmp\|冷启动" docs/architecture/state-persistence.md`
Expected: 至少 15 行匹配

- [ ] **Step 4: Commit**

```bash
git add docs/architecture/state-persistence.md
git commit -m "docs: 精细化 Arena 进度持久化 Schema + XMP 侧边栏导出方案

- ArenaProgress 新增 workflowMode/arena/autoSelect/reviewOnly 分支
- 精细化到 currentGroupIndex/currentRound/currentPairIndex
- 冷启动恢复横幅 + 恢复/放弃选择
- XMP 侧边栏：星级/Pick/Reject/色标/标签
- 原子写入 XMP 文件（temp + rename）
- 兼容 Lightroom/Capture One/Bridge"
```

---

## Self-Review Checklist

### 1. Spec Coverage

| Killshot P0 项 | 对应 Task | 状态 |
|----------------|----------|------|
| 增加混合工作流模式（AI Auto-Select / Arena / Review-Only） | Task 1 | ✅ 完整 |
| Speed Mode 本地化实现方案 | Task 2 | ✅ 完整 |
| RAW/DNG 缩略图支持 | Task 2 | ✅ 完整 |
| 免费版取消 100 张限制 | Task 3 | ✅ 完整 |
| XMP 侧边栏导出 | Task 5 | ✅ 完整 |
| MVP 功能表更新 | Task 4 | ✅ 完整 |
| Quick Win 按钮 | Task 1 | ✅ 完整 |
| 跨平台路线 | Task 3 | ✅ 完整 |
| 精细化 Arena 进度持久化 | Task 5 | ✅ 完整 |

### 2. Placeholder Scan

- ✅ 无 "TBD" / "TODO" / "implement later"
- ✅ 无 "add appropriate error handling" / "handle edge cases"
- ✅ 无 "write tests" without test code（此为文档任务，无测试步骤）
- ✅ 无 "similar to Task N"
- ✅ 所有步骤含具体内容

### 3. Type Consistency

- `ArenaProgress.workflowMode` 类型 `'auto_select' | 'arena' | 'review_only'` — 与 Task 1 的三个模式名称一致
- `ArenaGroupState.status` 类型含 `'skipped'` — 与 Task 1 的 Quick Win 跳过逻辑一致
- `getMimeType()` 和 `isRawFormat()` 在 Task 2 中引用但未定义 — 这是示意代码，MVP 实现时会创建
- `generateXmp()` 函数签名与 `ExportPhoto` 接口在 Task 5 中自洽

---

## Summary

| Task | 文件 | 核心改动 |
|------|------|---------|
| 1 | arena-pk-design.md | 新增三种工作流模式 + Quick Win 机制 |
| 2 | harmony-tech-stack.md | Speed Mode 本地化算法方案 + RAW 支持策略 |
| 3 | competitive-analysis.md | 免费版取消数量限制 + 跨平台路线图 |
| 4 | mvp-scope.md | P0 功能表/时间线/验收标准全面更新 |
| 5 | state-persistence.md | 精细化 Arena Schema + XMP 导出方案 |

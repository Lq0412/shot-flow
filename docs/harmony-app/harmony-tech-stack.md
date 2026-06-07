# 鸿蒙端技术栈与 API 映射

> 从桌面版功能到鸿蒙 API 的映射关系

## Web 版 → 鸿蒙端功能映射

| Web 版功能 | 鸿蒙 App 策略 | 鸿蒙 API |
|-----------|--------------|----------|
| 极速模式（本地 pHash + ORB） | **鸿蒙端本地实现** | ArkTS 实现 pHash 算法 |
| 专家模式（DINOv2 + NIMA + InsightFace） | **云端 API** | `@ohos.net.http` 上传缩略图 |
| 土豪模式（LLM 视觉判定） | **云端 API** | 复用云端 `llm_judge.py` |
| 擂台 PK ←/→/↑/↓ | **鸿蒙原生手势** | `GestureGroup` / `PinchGesture` |
| 缩放 1x/2x/4x | **鸿蒙原生缩放** | `PinchGesture` 比丝滑 |
| 水印 11 样式 | **MVP 只做 2-3 种** | `Canvas` + `CanvasRenderingContext2D` |
| 照片导入（手动粘贴路径） | **PhotoPicker 直接选相册** | `photoAccessHelper.PhotoViewPicker` |
| Flask localhost 服务 | **不需要** | 鸿蒙原生 App |
| RAW 支持 | **MVP 支持 DNG + 嵌入式预览** | DNG 直读 + CR3/NEF/ARW 嵌入 JPEG 预览提取 |
| 多 Session | **首版单 Session** | 后续版本加 Tab 多任务 |

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

## 鸿蒙端关键 API 速查

### 照片访问

```typescript
// 选择照片（无需权限）
import { photoAccessHelper } from '@kit.MediaLibraryKit';

const picker = new photoAccessHelper.PhotoViewPicker();
const result = await picker.select({
  MIMETypes: ['image/jpeg', 'image/png', 'image/heif', 'image/dng'],
  maxSelectNumber: 500
});
```

### EXIF 读取

```typescript
import { image } from '@kit.ImageKit';

const imageSource = image.createImageSource(filePath);
const exif = await imageSource.getImageProperty(imagePropertyKey.F_NUMBER);
// 可读取：DateTime, FNumber, ExposureTime, ISO, FocalLength, LensModel 等
```

### Canvas 水印渲染

```typescript
// Canvas 绘制水印
@Builder
WatermarkCanvas() {
  Canvas(this.context)
    .width('100%')
    .height('100%')
    .onReady(() => {
      this.context.drawImage(img, 0, 0, width, height)
      this.context.font = '24px sans-serif'
      this.context.fillStyle = '#FFFFFF'
      this.context.fillText(`f/${fNumber}  ${shutterSpeed}s  ISO${iso}`, x, y)
    })
}
```

### 状态管理

```typescript
// 擂台进度状态
@Entry
@Component
struct ArenaPage {
  @State currentGroup: number = 0
  @State currentRound: number = 0
  @State groups: PhotoGroup[] = []
  @State undoStack: ArenaDecision[] = []

  // AppStorage 跨页面共享
  @StorageProp('totalGroups') totalGroups: number = 0
}
```

### 网络请求

```typescript
import { http } from '@kit.NetworkKit';

// 上传图片到云端
const uploadTask = http.createUploadTask(
  'https://api.shotflow.com/v1/analyze',
  [{ name: 'images', type: 'multipart', filePath: photoPath }]
)
```

### 本地持久化

```typescript
import { preferences } from '@kit.ArkData';

// 保存用户偏好
const pref = await preferences.getPreferences(context, 'shot_flow_prefs')
await pref.put('last_strength', 'standard')
await pref.put('preference_weights', JSON.stringify(weights))
await pref.flush()
```

## 水印样式（MVP 选 3 种）

从 11 种样式中优先实现：

1. **样式 A（标准白框）** — 最通用，底部显示 EXIF + 品牌 Logo
2. **样式 G（极简线）** — 底部一条细线 + 小字 EXIF，年轻人喜欢
3. **样式 D（圆角白框）** — 圆角边框 + EXIF，社媒分享感强

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
| RAW 支持 | **首版不支持** | 后续用 `image.createImageSource()` 扩展 |
| 多 Session | **首版单 Session** | 后续版本加 Tab 多任务 |

## 鸿蒙端关键 API 速查

### 照片访问

```typescript
// 选择照片（无需权限）
import { photoAccessHelper } from '@kit.MediaLibraryKit';

const picker = new photoAccessHelper.PhotoViewPicker();
const result = await picker.select({
  MIMETypes: ['image/jpeg', 'image/png', 'image/heif'],
  maxSelectNumber: 200
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

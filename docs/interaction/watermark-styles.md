# 水印样式设计思路

> 从桌面版 11 种样式中提取的设计思路，鸿蒙 MVP 选 3 种

## 核心思路

水印是摄影师的"签名"，但大多数水印工具要么太朴素（纯文字），要么太花哨（占据画面）。

**片刻的水印设计理念**：模拟相机 LCD 回放画面——既是信息展示，也是摄影文化的表达。

## 样式概览

### 样式 A — 标准白框（MVP）

```
┌──────────────────────────────────────┐
│                                      │
│           [ 照片主体 ]                 │
│                                      │
├──────────────────────────────────────┤
│  [品牌Logo]  f/2.8  1/500s  ISO400  │
│  FUJIFILM X-T5  XF 56mm F1.2        │
└──────────────────────────────────────┘
```

- 白色底部条
- 品牌 Logo + EXIF 信息
- 最通用，适合所有场景

### 样式 G — 极简线（MVP）

```
┌──────────────────────────────────────┐
│                                      │
│           [ 照片主体 ]                 │
│                                      │
│  ─────────────────────────────────── │
│  f/2.8 · 1/500s · ISO400 · 56mm    │
└──────────────────────────────────────┘
```

- 底部一条细线分隔
- 小字 EXIF 信息
- 年轻人/社交媒体风格

### 样式 D — 圆角白框（MVP）

```
┌──────────────────────────────────────┐
│  ╭──────────────────────────────╮    │
│  │                              │    │
│  │        [ 照片主体 ]           │    │
│  │                              │    │
│  ╰──────────────────────────────╯    │
│         f/2.8 · 1/500s · ISO400     │
└──────────────────────────────────────┘
```

- 圆角内框 + 白色背景
- EXIF 在外框底部
- 社媒分享感强

## 品牌 Logo 自动匹配

桌面版已支持的品牌：

| 品牌 | Logo | 说明 |
|------|------|------|
| Fujifilm | ✓ | 含 X-T5 机身图（样式 H） |
| Canon | ✓ | |
| Nikon | ✓ | |
| Sony | ✓ | |
| Leica | ✓ | |
| Hasselblad | ✓ | |
| Olympus | ✓ | 2 个变体 |
| Panasonic | ✓ | 2 个变体 |
| Pentax | ✓ | |
| Ricoh | ✓ | |
| Apple | ✓ | |
| Xmage | ✓ | 华为影像 |

**匹配逻辑**：从 EXIF `Make` 字段自动匹配品牌，无需用户手动选择。

## 鸿蒙端实现要点

### Canvas 绘制

```typescript
// 核心步骤
1. 加载原图 → Canvas
2. 计算边框尺寸 + 照片缩放
3. 绘制背景（白色/黑色）
4. 绘制照片到指定区域
5. 读取 EXIF 数据
6. 绘制品牌 Logo（自动抠背景）
7. 绘制 EXIF 文字
8. 导出 JPEG/PNG
```

### 品牌Logo自动抠背景

桌面版的做法：
- 加载 Logo 图片
- 取左上角像素颜色作为背景色
- 将该颜色的 alpha 设为 0（透明）
- 叠加到水印条上

鸿蒙端可以用 `CanvasRenderingContext2D` 的像素操作实现类似效果。

### EXIF 数据提取

```typescript
const imageSource = image.createImageSource(filePath)
const make = await imageSource.getImageProperty('ImagePropertyKey.MAKE')     // FUJIFILM
const model = await imageSource.getImageProperty('ImagePropertyKey.MODEL')   // X-T5
const fNumber = await imageSource.getImageProperty('ImagePropertyKey.F_NUMBER') // 2.8
const exposureTime = await imageSource.getImageProperty('ImagePropertyKey.EXPOSURE_TIME')
const iso = await imageSource.getImageProperty('ImagePropertyKey.ISO_SPEED_RATINGS')
const focalLength = await imageSource.getImageProperty('ImagePropertyKey.FOCAL_LENGTH')
```

## 后续版本可扩展的样式

- **样式 H — 相机 LCD 回放**：模拟相机屏幕，最炫但实现复杂
- **杂志风格**：自动提取照片主色调做配色
- **竖版水印**：侧边栏式，不遮挡照片底部

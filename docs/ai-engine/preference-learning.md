# 用户偏好学习机制

> 让 AI 越用越懂你——最强的用户粘性机制

## 核心思路

每个摄影师的审美偏好不同：
- 有的偏好锐利清晰（拍建筑）
- 有的偏好柔和虚化（拍人像）
- 有的偏好明亮通透
- 有的偏好暗调情绪

**偏好学习**：记录用户在擂台中的每一次选择，累积推断用户的审美倾向，用于调整后续 AI 的推荐。

## 数据采集

### 每次擂台决策记录

```
{
  "winner": {
    "aesthetic": 8.1,
    "sharpness": 6.5,
    "face_quality": 7.0,
    "technical": 7.8,
    "brightness": 0.65
  },
  "loser": {
    "aesthetic": 7.9,
    "sharpness": 8.2,
    "face_quality": 6.5,
    "technical": 8.0,
    "brightness": 0.55
  },
  "user_chose": "left"   // 用户选了 winner
}
```

### 偏好向量更新

```
delta = winner.signals - loser.signals

如果用户持续选择"锐度低但美学分高"的照片：
  → pref_aesthetic += 0.01
  → pref_sharpness -= 0.005

偏好向量：
{
  "aesthetic": 0.15,     // 偏好美学
  "sharpness": -0.05,    // 不太在意锐度
  "brightness": 0.08,    // 偏好明亮
  "face_quality": 0.02   // 轻微关注人脸质量
}
```

## 偏好如何影响系统

### 1. AI 推荐标记

擂台中，AI 推荐的照片不再只看综合分，而是加入偏好权重：

```
personalized_score = composite + dot_product(preference, signals)
```

偏好锐利的用户 → 锐利照片得分提升 → AI 推荐更合口味

### 2. 初筛权重调整

```
默认权重：  aesthetic=0.50, sharpness=0.20, face=0.15, tech=0.15

用户偏好锐利后：
            aesthetic=0.45, sharpness=0.28, face=0.15, tech=0.12
```

不太关注美学的用户 → 美学权重降低 → 不会因为"不够美"被误淘汰

### 3. 跨 Session 累积

```
Session 1: 30 次决策 → 偏好初步形成
Session 2: 50 次决策 → 偏好更准确
Session 10: 500 次决策 → 偏好稳定，推荐准确率明显提升
```

## 持久化策略

### 桌面版的教训

桌面版已实现偏好学习（`pref_aesthetic`, `pref_sharpness`, `pref_brightness`），但**未被持久化**。每次重启 App 后偏好数据丢失。

这是最大的浪费——代码写好了但没用起来。

### 鸿蒙版必须做的

```
1. 本地持久化（Preferences）
   - 每次决策后更新偏好向量
   - 跨 App 启动保留

2. 云端同步（API）
   - 按用户 ID（设备 ID）聚合
   - 跨设备同步
   - 隐私友好：只存偏好向量，不存照片

3. 冷启动策略
   - 新用户使用默认权重
   - 30 次决策后开始影响推荐
   - 100 次决策后完全个性化
```

## 为什么这是最强 Lock-in

**关键洞察**：

> 用户积累 1000+ 次决策后，切换到竞品 = "AI 从零开始不懂你的审美了"。

| 竞品 | 偏好学习 |
|------|---------|
| Aftershoot | 固定 AI，不可学习 |
| Narrative | 固定 AI，不可学习 |
| FilterPixel | 固定 AI，不可学习 |
| **Shot Flow** | **可学习，越用越准** |

这是所有竞品都没有的能力。一旦用户用习惯了，切换成本极高。

## 未来扩展

### 客户/主角人脸记忆

```
跨 session 保存主角人脸 ID：
  婚礼客户 A → 偏好明亮 + 正脸
  客户 B → 偏好暗调 + 侧脸

下次拍摄同一客户 → 自动切换偏好配置
```

### 偏好模型导出/分享

```
导出自己的偏好模型：
  → 分享给同行"我的审美配置"
  → 团队统一选片标准
```

### 本地美学模型微调（长期）

```
擂台决策数据是天然训练集：
  winner = positive sample
  loser = negative sample

长期可以用这些数据微调本地美学模型，
从"规则匹配"进化到"真正理解你的审美"
```

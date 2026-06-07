# 状态持久化与恢复模式

> 从桌面版提取的状态管理思路，适用于鸿蒙端和云端

## 状态机设计

```
pending → scanning → hashing → grouping → done
  │          │          │          │
  └──── error ◄─────────┘          │
  └──── cancelled ◄────────────────┘
```

## 桌面版的做法

### JSON 状态文件

```json
{
  "version": 6,
  "folder": "/path/to/photos",
  "mode": "expert",
  "groups": [
    {
      "group_id": "g1",
      "photos": ["img1.jpg", "img2.jpg"],
      "status": "pending",
      "winner": null,
      "loser_files": []
    }
  ],
  "pref_aesthetic": 0.0,
  "pref_sharpness": 0.0,
  "pref_brightness": 0.0,
  "runtime": "auto"
}
```

### 原子写入

```
1. 写入临时文件 .pic_selecter_state.tmp
2. fsync() 确保刷盘
3. rename() 原子替换
```

**关键洞察**：直接写 JSON 文件在断电/崩溃时会损坏。temp + rename 是文件系统层面的原子操作。

### Schema 版本迁移

```
v4 → v5: 加 pref_brightness 字段
v5 → v6: 加 runtime 字段
太旧（< v4）: 报错 + 引导用户重新扫描
```

## 鸿蒙端适用的部分

### 本地状态（Preferences + JSON）

```typescript
// 任务状态
interface TaskState {
  taskId: string
  status: 'pending' | 'uploading' | 'analyzing' | 'arena' | 'done' | 'error'
  photos: string[]
  groups: GroupState[]
  arenaProgress: ArenaProgress
  preferences: UserPreferences
}

// 保存到本地
const pref = await preferences.getPreferences(context, 'shot_flow')
await pref.put('current_task', JSON.stringify(taskState))
await pref.flush()
```

### 擂台撤销栈

```typescript
interface ArenaDecision {
  groupId: string
  winnerIndex: number
  loserIndex: number
  timestamp: number
}

// 多级撤销
@State undoStack: ArenaDecision[] = []

// 撤销：弹出栈顶，恢复状态
undo(): void {
  const last = this.undoStack.pop()
  if (last) {
    this.restoreDecision(last)
  }
}
```

### 云端状态（API）

```
鸿蒙端:
  - 擂台进度（当前组、当前轮）
  - 撤销栈（最近 N 步）
  - 用户偏好

云端:
  - 任务状态（分析进度、分组结果）
  - 分析结果（画质评分、分组映射）
  - 偏好模型（累积的权重）
```

## 偏好学习的持久化

### 核心数据

```
用户每次擂台决策：
  - winner 的信号值（美学分、锐度分、脸质分、技术分）
  - loser 的信号值
  - 用户选了哪个

累积后得到偏好方向：
  - 偏好更锐利的？pref_sharpness += delta
  - 偏好更亮的？pref_brightness += delta
  - 偏好更美的？pref_aesthetic += delta
```

### 持久化策略

```
鸿蒙端本地:
  - 存储到 Preferences
  - 每次决策后更新
  - 跨 App 启动保留

云端:
  - 按用户 ID 聚合
  - 跨设备同步
  - 用于动态调整初筛权重
```

### 偏好学习是最强 Lock-in

**洞察**：用户积累 1000+ 次决策后，切换竞品 = "AI 从零开始不懂你的审美了"。

这个机制在桌面版已经实现（`pref_*` 字段），但未被持久化到跨 session。鸿蒙版必须从第一天就持久化。

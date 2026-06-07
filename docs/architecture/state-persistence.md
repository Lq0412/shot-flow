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
          <rdf:li>{champion|runner_up|eliminated}</rdf:li>
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

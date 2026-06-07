# 云端 AI 服务设计

> FastAPI 后端，复用桌面版 AI 算法逻辑，提供 HTTP API 给鸿蒙端调用

## 架构思路

```
鸿蒙 App  ──HTTPS──▶  Nginx (SSL + 限流)
                        │
                        ▼
                    FastAPI (Gunicorn)
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
          初筛服务   分组服务   LLM判断
          quality   clustering  llm_judge
```

## 核心 API 设计

```
POST /api/v1/analyze
  Input:  multipart (图片文件列表 + options)
  Output: { images: [{ path, quality_score, flags, auto_reject, reject_reason }] }

POST /api/v1/group
  Input:  { images: [{ path, quality }], options: { threshold } }
  Output: { groups: [[path, ...], ...] }

POST /api/v1/prescreen
  Input:  multipart (图片文件列表 + strength)
  Output: { rejected: [path], reasons: { path: reason }, kept: [path] }

POST /api/v1/judge (土豪模式)
  Input:  { image_base64, model, strength }
  Output: { verdict: "pass"|"reject", reason: "..." }

GET  /api/v1/preferences/{user_id}
  Output: { aesthetic_weight, sharpness_weight, brightness_weight }

POST /api/v1/preferences/{user_id}
  Input:  { decision: { winner, loser, signals } }
  Output: { updated_weights }
```

## 关键设计原则

### 1. 无静默降级

**思路**：每个模块在依赖缺失时明确抛异常，不返回空结果。

原因：照片选片工具的静默失败 = 漏掉好照片。这比报错更危险。

实际做法：
- AI 模型加载失败 → 抛 `VisionUnavailable` 错误
- LLM API 不可用 → 抛 `LLMJudgeError` 错误
- 任务启动时预检所有依赖（`prewarm_all()`）
- 前端收到明确错误信息，引导用户切换模式

### 2. 自适应并发限流器

**思路**：LLM API 调用需要限流，但固定并发数太保守。

实际做法：
- 起始并发 = 10
- 收到 429 (Rate Limit) → 并发减半（最低 1）
- 连续 30 次成功 + 稳定 10 秒 → 并发 +1
- 基于条件变量的线程安全实现

### 3. Per-job 日志

**思路**：每次任务创建独立日志文件，而非一个不断增长的 log.txt。

好处：
- 调试特定任务时直接找对应文件
- 日志不会无限增长
- 可以按时间清理旧日志

### 4. 图片处理完即删

**思路**：云端不存储用户照片，处理完立即删除。

- 缩略图上传后，分析完成即删除
- 不写数据库，不持久化
- 隐私友好，合规

## 安全设计

- HTTPS 强制
- API Key 认证（每个 App 安装生成唯一 key）
- 图片上传限流（防止滥用）
- Origin/Referer 校验
- 文件类型白名单（仅 JPG/PNG/HEIF）

## 部署方案

| 组件 | 规格 | 月成本 |
|------|------|--------|
| 轻量云服务器 | 2C4G Ubuntu | ¥100-200 |
| SSL 证书 | Let's Encrypt 自动续期 | 免费 |
| 域名 + 备案 | .com 域名 | ¥60/年 |

### 内存预算

| 组件 | 占用 |
|------|------|
| 系统 + Nginx | ~300MB |
| FastAPI (2 worker) | ~400MB |
| AI 模型 (DINOv2-Small + OpenCV) | ~400MB |
| 临时图片处理 | ~200MB |
| 预留 | ~700MB |
| **合计** | **~2GB** |

## LLM 判断的 Prompt 工程思路

### "骨架 vs 皮肉" 框架

- **骨架（不可修复）**：焦点不实、表情失误、时机错过 → 直接淘汰
- **皮肉（可修复）**：乱发、皮肤瑕疵、路人入镜 → 标记"可修复"通过
- 两个力度等级（standard / advanced），不同阈值
- 大量边缘案例示例（Cases A-M）
- 明确反模式："不要写'平庸'或'没有亮点'——那些是借口，不是观察"

### 土豪模式费用控制

- 2000 张照片全跑 LLM 可能花上百元
- 需要预估金额显示 + 实时累计 + 到达预算自动暂停
- 这在桌面版是缺失的，鸿蒙版必须加上

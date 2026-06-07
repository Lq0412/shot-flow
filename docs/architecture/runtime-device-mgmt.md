# GPU/CPU 运行时管理思路

> 从借鉴项目中提取的设备管理设计模式，适用于云端 AI 服务

## 核心思路：分层运行时传播

```
用户选择 (UI)
    ↓ runtime: auto | cpu | gpu
API 参数
    ↓ 环境变量 / 配置
模型加载层
    ↓ device: cuda | mps | cpu
每个 AI 组件独立感知设备
```

**为什么需要这个？**

AI 应用中常见问题：
- 一半组件跑 GPU，另一半跑 CPU → 内存浪费 / 速度不一致
- MPS（Apple Silicon）上某些模型会爆 VRAM → 需要单独路由到 CPU
- 用户想强制 CPU（调试/兼容）→ 不能只靠自动检测

## 模式一：设备自适应

```python
def _device():
    """根据运行时偏好 + 硬件能力选择设备"""
    if runtime == "cpu":
        return torch.device("cpu")
    if torch.cuda.is_available():
        return torch.device("cuda")
    if hasattr(torch.backends, "mps") and torch.backends.mps.is_available():
        return torch.device("mps")
    return torch.device("cpu")
```

## 模式二：组件级设备路由

```python
def _pyiqa_device():
    """pyiqa/CLIP 模型在 MPS 上会爆 VRAM，路由到 CPU"""
    if _device_type() == "mps":
        return torch.device("cpu")
    return _device()

def _select_onnx_providers():
    """ONNX Runtime 的 provider 选择"""
    if runtime == "cpu":
        return ["CPUExecutionProvider"]
    if _device_type() == "cuda":
        return ["CUDAExecutionProvider", "CPUExecutionProvider"]
    return ["CPUExecutionProvider"]
```

**关键洞察**：不是所有 AI 组件都能在同一个设备上安全运行。有些模型（特别是 CLIP 变体和 pyiqa 包装的模型）在 MPS 上会崩溃。需要按组件粒度控制设备。

## 模式三：加速器感知并发

```python
def _default_expert_workers():
    """根据设备类型调整并发"""
    env = os.environ.get("SHOT_FLOW_EXPERT_WORKERS")
    if env:
        return int(env)

    device_type = _device_type()
    if device_type in ("cpu", "mps"):
        return 1   # 单线程，避免 ONNX 崩溃
    if device_type == "cuda":
        return min(4, os.cpu_count() - 2)  # 多线程喂 GPU
    return 1
```

**关键洞察**：不要因为一个平台的限制就全局限制并发。CUDA GPU 显著受益于多线程供给（2-4 workers），而 MPS/CPU 的 ONNX 模型会因为多线程崩溃。

## 模式四：懒加载 + 预热

```
导入时 → 不加载模型（快速启动）
任务开始时 → prewarm_all()（一次性加载 + 验证）
模型首次使用时 → 真正初始化
后续使用 → 缓存实例
```

**关键洞察**：分离"导入成功"和"模型能跑"。下载权重失败不应在导入时报错，而应在预热时暴露。这样用户可以先看到界面，再处理模型问题。

## 对鸿蒙云端服务的应用

云端服务的设备管理比桌面端简单（服务器设备固定），但以下思路仍有价值：

1. **分层传播**：配置文件 → FastAPI 启动参数 → 模型加载
2. **组件级路由**：ONNX 模型用 CPU，PyTorch 模型用 GPU
3. **懒加载 + 预热**：FastAPI 启动时不加载模型，首次请求时加载 + 缓存
4. **自适应并发**：有 GPU 时多 worker，纯 CPU 时少 worker
5. **预检**：任务开始前检查所有模型可用，提前报错

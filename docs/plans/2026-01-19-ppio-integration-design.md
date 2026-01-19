# PPIO API 集成设计

## 概述

支持使用 PPIO API (`https://api.ppio.com/openai/v1`) 进行数据合成，可选模型包括：
- `pa/gpt-5.2`
- `pa/claude-opus-4-5-20251101`
- `pa/gemini-3-pro-preview`

## 使用方式

```bash
export OPENAI_API_KEY="your-ppio-api-key"

uv run python src/r2egym/agenthub/run/edit.py runagent_multiple \
  --traj_dir "./traj" \
  --max_workers 54 \
  --start_idx 0 \
  --k 2000 \
  --dataset "R2E-Gym/R2E-Gym-Lite" \
  --split "train" \
  --llm_name "pa/gpt-5.2" \
  --llm_base_url "https://api.ppio.com/openai/v1" \
  --use_fn_calling True \
  --exp_name r2egym-training-trajectories \
  --temperature 0.2 \
  --max_steps 40
```

## 改动清单

### 1. src/r2egym/agenthub/run/edit.py

**runagent 函数** - 添加 `llm_base_url` 参数：

```python
def runagent(
    ds,
    exp_name: Optional[str] = None,
    max_steps=40,
    num_restarts=1,
    max_steps_absolute=50,
    llm_name="gpt-4o",
    llm_base_url: Optional[str] = None,  # 新增
    temperature=0,
    use_fn_calling: bool = True,
    # ... 其他参数不变
)
```

传递给 AgentArgs：

```python
agent_args.llm_name = llm_name
agent_args.llm_base_url = llm_base_url  # 新增
```

**runagent_multiple 函数** - 同样添加参数并传递。

### 2. src/r2egym/agenthub/agent/agent.py

**llm_base_url 初始化** - 优先使用传入的值：

```python
self.llm_base_url = args.llm_base_url or (
    os.environ.get("LLM_BASE_URL", "http://localhost:8000/v1")
    if ("openai/" in self.llm_name) or ("hosted_vllm" in self.llm_name)
    else None
)
```

**function calling 检测** - 添加 `pa/` 前缀支持：

```python
support_fn_calling = (
    "gpt" in self.llm_name
    or "sonnet" in self.llm_name
    or "o3" in self.llm_name
    or "o4" in self.llm_name
    or self.llm_name.startswith("pa/")
) and "qwen" not in self.llm_name
```

## 设计决策

1. **`--llm_base_url` 参数** - 灵活通用，可用于任何 OpenAI 兼容的 API
2. **`OPENAI_API_KEY` 环境变量** - 复用现有机制，LiteLLM 默认读取
3. **`pa/` 前缀检测** - 假设所有 PPIO 模型都支持 function calling
4. **运算符优先级修正** - 原有 `and "qwen"` 应包裹整个条件

## 不需要改动

- `AgentArgs` dataclass - 已有 `llm_base_url` 字段
- LiteLLM 调用 - 已支持 `api_base` 参数
- API Key 处理 - 使用现有 `OPENAI_API_KEY`

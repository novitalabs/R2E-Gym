# R2E-Gym Training Run: Claude Opus 4.5

## 运行配置

**日期**: 2026-01-20
**模型**: `pa/claude-opus-4-5-20251101` (via PPIO API)
**数据集**: `R2E-Gym/R2E-Gym-Lite` (train split)

### 命令

```bash
HTTPS_PROXY=http://172.17.0.1:1081 HTTP_PROXY=http://172.17.0.1:1081 \
uv run --active python src/r2egym/agenthub/run/edit.py runagent_multiple \
  --traj_dir "./traj" \
  --max_workers 54 \
  --start_idx 0 \
  --k 2000 \
  --dataset "R2E-Gym/R2E-Gym-Lite" \
  --split "train" \
  --llm_name "openai/pa/claude-opus-4-5-20251101" \
  --llm_base_url "https://api.ppio.com/openai/v1" \
  --use_fn_calling True \
  --exp_name r2egym-training-trajectories \
  --temperature 0.2 \
  --max_steps 40 \
  --backend docker
```

### 关键参数说明

| 参数 | 值 | 说明 |
|------|-----|------|
| `llm_name` | `openai/pa/claude-opus-4-5-20251101` | 需要 `openai/` 前缀让 LiteLLM 识别为 OpenAI 兼容 API |
| `llm_base_url` | `https://api.ppio.com/openai/v1` | PPIO API endpoint |
| `max_workers` | 54 | 并行 Docker 容器数 |
| `max_steps` | 40 | 每个 agent 最大步数 |
| `backend` | `docker` | 使用本地 Docker (非 Kubernetes) |
| `temperature` | 0.2 | 低温度确保一致性 |

## 遇到的问题与解决

### 1. Module Not Found
**问题**: `ModuleNotFoundError: No module named 'r2egym'`
**解决**: 使用 `uv pip install -e .` 安装包

### 2. HuggingFace 连接超时
**问题**: `ConnectionError: Couldn't reach 'R2E-Gym/R2E-Gym-Lite'`
**解决**: 设置 `HTTPS_PROXY=http://172.17.0.1:1081`

### 3. Kubernetes 配置缺失
**问题**: `Invalid kube-config file. No configuration found.`
**解决**: 添加 `--backend docker` 使用本地 Docker

### 4. LLM Provider 未识别
**问题**: `litellm.BadRequestError: LLM Provider NOT provided`
**原因**: 模型名 `pa/claude-opus-4-5-20251101` 无法被 LiteLLM 识别
**解决**: 改为 `openai/pa/claude-opus-4-5-20251101`

## 轨迹分析

### 输出文件
- **路径**: `./traj/r2egym-training-trajectories.jsonl`
- **格式**: JSONL (每行一个完整轨迹)

### 轨迹结构

```json
{
  "trajectory_steps": [...],      // 步骤列表
  "problem_statement": "...",     // 问题描述
  "docker_image": "...",          // Docker 镜像
  "exp_name": "...",              // 实验名称
  "env_args": {
    "ds": {
      "repo_name": "pandas",      // 仓库名
      "docker_image": "...",
      "commit_hash": "..."
    }
  },
  "reward": 1.0,                  // 奖励 (1.0=成功, 0.0=失败)
  "test_output": "..."            // 测试输出
}
```

### 初步统计 (运行中)

| 指标 | 值 |
|------|-----|
| 目标样本数 | 2000 |
| 成功率 | ~62.5% |
| 平均步数 | ~23 步 |
| 涉及仓库 | pillow, numpy, tornado, aiohttp, pyramid, pandas, scrapy, etc. |

## 监控命令

```bash
# 查看完成轨迹数
wc -l ./traj/r2egym-training-trajectories.jsonl

# 查看运行容器数
docker ps --format "{{.Names}}" | grep namanjain | wc -l

# 查看实时日志
tail -f /tmp/claude/-workspace-develop-agentic-data-R2E-Gym/tasks/b9d9625.output

# 查看错误日志
grep -i "error\|failed" /tmp/claude/-workspace-develop-agentic-data-R2E-Gym/tasks/b9d9625.output | tail -20
```

## 预估时间

基于初始观察:
- 当前速率: ~2-3 轨迹/分钟
- 目标: 2000 轨迹
- 预估总时间: ~12-16 小时

## 后续工作

1. 监控训练完成
2. 分析成功/失败案例
3. 评估轨迹质量
4. 可选: 使用其他模型 (gpt-5.2, gemini-3-pro) 生成对比数据

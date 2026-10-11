简体中文 | [English](README.en.md)

# LLM-as-a-Verifier

**面向智能体的通用细粒度推理期验证、进度追踪与反馈评估框架**。

`LLM-as-a-Verifier` 是一个无需额外微调、通用于多模态与多种智能体架构的统一验证框架。它通过构建精细的评分维度解构、在评分 Token 的完整 Logprob 对数概率分布上求取期望收益，并结合成对奖励模型（Pairwise Reward Model）与重复评估缩放，为编程、机器人、医疗等复杂智能体任务提供高灵敏度、连续平滑的即时评估反馈，在 Terminal-Bench、SWE-Bench Verified 等基准测试中取得 SOTA 表现。

---

## 核心机理与数学直觉

传统的大模型验证（LLM-as-a-Judge）往往依赖输出离散的布尔值或整数评分，极易引发打分漂移与方差剧增。`LLM-as-a-Verifier` 引入了三大核心支柱：

```
[智能体候选轨迹 / 代码方案] ──> [多维度准则解构 (Criteria Decomposition)]
                                              │
                                              ▼
[平滑期望收益 E[Reward]] <── [Logprob 全概率分布积分] <── [成对比较与重复评估]
         │
         ▼
[三大应用形态: Best-of-N 测试期缩放 | 逐步进度追踪 (track) | RL 强化学习奖励]
```

1. **准则精细分解（Criteria Decomposition）**：将复合目标拆解为可独立核验的原子准则（如代码正确性、边界条件处理、副作用约束等）。
2. **Logprob 全概率分布期望（Expectation over Full Logprob Distribution）**：不再仅截取贪婪采样的离散打分，而是通过对大模型输出评分 Token 的全部对数概率分布计算数学期望，将离散评判转化为高质量连续数值。
3. **测试期推理缩放（Test-Time Scaling）**：支持 Best-of-N 最优候选筛选、成对偏好对比与轨迹单步增量进度追踪。

---

## 快速开始

### 安装

```bash
# 从 PyPI 安装
pip install llm-verifier

# 或从源码安装最新开发版
pip install -e .
```

---

## 核心接口使用示例

### 1. Best-of-N 候选最优选择 (`select`)

```python
import llm_verifier

problem = "编写一个反转字符串的 Python 函数。"
candidates = [
    "def rev(s): return s[::-1]",
    "def rev(s): return s",
    "def rev(s): return ''.join(sorted(s))",
]

result = llm_verifier.select(
    problem=problem,
    candidates=candidates,
    criteria={"Correctness": "代码是否真正反转了输入字符串？"},
)

print(result.index)   # 最佳方案索引: 0
print(result.scores)  # 各方案连续期望得分: [0.73104, 0.38446, 0.38449]
```

### 2. 成对方案细粒度偏好对比 (`compare`)

```python
reward_a, reward_b = llm_verifier.compare(
    problem, 
    candidates[0], 
    candidates[1],
    criteria={"Overall": "方案是否正确且高效地解决了问题？"},
)
print(reward_a, reward_b)  # 输出连续奖励值，范围 [0, 1]
```

### 3. 智能体单步进度连续追踪 (`track`)

在智能体长程运行中，可逐步量化当前步骤是否有效逼近目标：

```python
progress = llm_verifier.track(
    goal="修复 issue #42 中的连接池泄漏缺陷",
    step_history=[
        "定位到 connection.py 中的未释放上下文",
        "使用 try...finally 包装连接释放逻辑"
    ]
)
print(progress.step_scores)
```

---

## 许可证

本项目基于 [MIT License](LICENSE) 开源。

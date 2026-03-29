---

# trade_agent：基于多智能体 LLM 的金融交易框架

## trade_agent 框架

trade_agent 是一个多智能体交易框架，模拟了真实交易公司的运作模式。通过部署由 LLM 驱动的专业智能体——从基本面分析师、情绪分析专家、技术分析师到交易员、风险管理团队——各智能体协同评估市场状况并辅助交易决策。此外，这些智能体通过动态讨论来确定最优策略。

<p align="center">
  <img src="assets/schema.png" style="width: 100%; height: auto;">
</p>

> trade_agent 框架仅用于研究目的。交易表现可能因多种因素而异，包括所选骨干语言模型、模型温度、交易周期、数据质量及其他非确定性因素。[本框架不构成任何金融、投资或交易建议。](https://tauric.ai/disclaimer/)

我们的框架将复杂的交易任务分解为专业化角色，确保系统在市场分析和决策方面实现稳健、可扩展的方法。

### 分析师团队
- **基本面分析师**：评估公司财务状况和业绩指标，识别内在价值和潜在风险信号。
- **情绪分析师**：使用情绪评分算法分析社交媒体和公众情绪，评估短期市场情绪。
- **新闻分析师**：监控全球新闻和宏观经济指标，解读事件对市场状况的影响。
- **技术分析师**：利用技术指标（如 MACD 和 RSI）检测交易模式并预测价格走势。

<p align="center">
  <img src="assets/analyst.png" width="100%" style="display: inline-block; margin: 0 2%;">
</p>

### 研究员团队
- 由看多和看空研究员组成，他们对分析师团队提供的见解进行批判性评估。通过结构化辩论，他们权衡潜在收益与固有风险。

<p align="center">
  <img src="assets/researcher.png" width="70%" style="display: inline-block; margin: 0 2%;">
</p>

### 交易员智能体
- 综合分析师和研究员的报告，做出明智的交易决策。基于全面的市场洞察，确定交易的时机和规模。

<p align="center">
  <img src="assets/trader.png" width="70%" style="display: inline-block; margin: 0 2%;">
</p>

### 风险管理与投资组合经理
- 通过评估市场波动性、流动性及其他风险因素，持续评估投资组合风险。风险管理团队评估并调整交易策略，向投资组合经理提供评估报告以供最终决策。
- 投资组合经理批准/拒绝交易提案。若获批准，订单将被发送至模拟交易所并执行。

<p align="center">
  <img src="assets/risk.png" width="70%" style="display: inline-block; margin: 0 2%;">
</p>

## 安装与命令行

### 安装

克隆 trade_agent：
```bash
git clone https://github.com/henyai/we_trade_agent.git
cd we_trade_agent
```

使用您喜欢的环境管理器创建虚拟环境：
```bash
conda create -n trade_agent python=3.13
conda activate trade_agent
```

安装依赖：
```bash
pip install -r requirements.txt
```

### 所需 API

trade_agent 支持多个 LLM 提供商。请设置所选提供商的 API 密钥：

```bash
export OPENAI_API_KEY=...          # OpenAI (GPT)
export GOOGLE_API_KEY=...          # Google (Gemini)
export ANTHROPIC_API_KEY=...       # Anthropic (Claude)
export XAI_API_KEY=...             # xAI (Grok)
export OPENROUTER_API_KEY=...      # OpenRouter
export ALPHA_VANTAGE_API_KEY=...   # Alpha Vantage
```

对于本地模型，请在配置中设置 `llama_provider: "ollama"` 来配置 Ollama。

或者，将 `.env.example` 复制为 `.env` 并填入您的密钥：
```bash
cp .env.example .env
```

### 命令行使用

您可以直接运行以下命令来体验 CLI：
```bash
python -m cli.main
```
您将看到一个界面，可以选择所需的股票代码、日期、LLM、研究深度等。

<p align="center">
  <img src="assets/cli/cli_init.png" width="100%" style="display: inline-block; margin: 0 2%;">
</p>

界面将实时显示加载结果，让您可以追踪智能体的运行进度。

<p align="center">
  <img src="assets/cli/cli_news.png" width="100%" style="display: inline-block; margin: 0 2%;">
</p>

<p align="center">
  <img src="assets/cli/cli_transaction.png" width="100%" style="display: inline-block; margin: 0 2%;">
</p>

## trade_agent 包

### 实现细节

我们使用 LangGraph 构建 trade_agent，以确保灵活性和模块化。该框架支持多个 LLM 提供商：OpenAI、Google、Anthropic、xAI、OpenRouter 和 Ollama。

### Python 使用方法

要在代码中使用 trade_agent，您可以导入 `trade_agent` 模块并初始化一个 `TradeAgentGraph()` 对象。`.propagate()` 函数将返回一个决策。您可以运行 `main.py`，以下是一个简单示例：

```python
from trade_agent.graph.trading_graph import TradeAgentGraph
from trade_agent.default_config import DEFAULT_CONFIG

ta = TradeAgentGraph(debug=True, config=DEFAULT_CONFIG.copy())

# 前向传播
_, decision = ta.propagate("NVDA", "2026-01-15")
print(decision)
```

您还可以调整默认配置来设置您自己选择的 LLM、辩论轮数等。

```python
from trade_agent.graph.trading_graph import TradeAgentGraph
from trade_agent.default_config import DEFAULT_CONFIG

config = DEFAULT_CONFIG.copy()
config["llm_provider"] = "openai"  # openai, google, anthropic, xai, openrouter, ollama
config["deep_think_llm"] = "gpt-5.2"  # 用于复杂推理的模型
config["quick_think_llm"] = "gpt-5-mini"  # 用于快速任务的模型
config["max_debate_rounds"] = 2

ta = TradeAgentGraph(debug=True, config=config)
_, decision = ta.propagate("NVDA", "2026-01-15")
print(decision)
```

有关所有配置选项，请参见 `trade_agent/default_config.py`。

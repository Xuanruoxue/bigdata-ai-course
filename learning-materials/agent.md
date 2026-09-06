# AI Agent(智能体)概念讲义

> 适用范围:大数据与人工智能课程 · 概念学习与资料核查
> 整理日期:2026-09-03
> 说明:本文为概念讲解存档,所有引用来源见文末"参考资料"。

---

## 目录

1. 概念的个人解释
2. 核心机制与组成
3. 具体应用场景
4. 容易混淆的问题与使用边界
5. 参考资料(可核查来源)

---

## 一、概念的个人解释

字面上 Agent 是"行动者、代理人"。在 AI 领域,可以把它浓缩为一句话:

> **Agent = LLM(大脑)+ 工具(手)+ 记忆(记事本)+ 自主决策循环(做事的节奏)**,是能"自己决定怎么做"并"真的去做"的系统。

### 1.1 一个通俗类比

| 对象 | 类比 | 特点 |
|---|---|---|
| 聊天机器人 | 只动口的"顾问" | 问什么答什么,不能动手办事,没有行动闭环 |
| **Agent** | 有手有脚的"实习生" | 领到目标后自己拆解任务、查资料、调系统、验证结果,失败会换方案 |

关键差异不在"会不会说话",而在 **自主性(Autonomy)** 与 **行动闭环**:感知 → 思考 → 行动 → 观察反馈 → 再思考。

### 1.2 概念边界提示

2024 年之后,业界(尤以 Anthropic 为代表)把这一大范畴统称为 **"Agentic Systems(智能体系统)"**,并在其中**严格区分两种架构**:

- **Workflow(工作流)**:LLM 与工具通过**预先写死的代码路径**编排。
- **Agent(智能体)**:LLM **在运行时动态决定**自己的流程与工具使用,保持对"如何完成任务"的控制。

通俗理解:区别在于"路径由代码定"还是"路径由模型定"。详见第四节。

---

## 二、核心机制与组成

### 2.1 五个组成要素

| 组成 | 作用 | 通俗说法 |
|---|---|---|
| **大模型 LLM** | 推理引擎:理解目标、拆解步骤、决定下一步 | 大脑 |
| **工具 Tools** | 执行具体动作:查数据库、写代码、搜网页、调 API;通过 Function Calling 或 MCP(统一工具协议)接入 | 手 |
| **记忆 / 状态** | 短期(上下文窗口)记录过程;长期(数据库 / 向量库)记录用户偏好与历史 | 记事本 |
| **决策循环** | 核心机制,业界称 **ReAct 模式**:Reason(思考)→ Act(行动)→ Observe(观察) | 做事的节奏 |
| **运行时与护栏** | 编排循环、错误处理、Guardrails(输入输出校验)、Human-in-the-loop(人工审批点) | 监工与刹车 |

### 2.2 核心决策循环(伪代码)

```python
state = 初始上下文(用户目标 + 历史记忆 + 可用工具清单)
while not done:
    thought = LLM(state)          # 思考:下一步做什么?
    if thought == "直接回答":      # 已足够,结束循环
        break
    result = call_tool(thought.tool, thought.args)   # 行动:调用某个工具
    state += result               # 观察:把反馈写回状态,进入下一轮
```

这就是"智能体化"的本质——不是一次性生成答案,而是**多轮"想一步、做一步、看一步"**的循环。

### 2.3 关键理论与主流框架

- 理论源头:**ReAct 论文**(2022,Google),提出"让模型边推理边行动边观察"的模式。
- 主流工程框架:LangGraph、OpenAI Agents SDK、微软 AutoGen、Claude Agent SDK 等。
- 工具接入标准:**MCP(Model Context Protocol)**,由 Anthropic 提出,用于统一外部工具/数据的接入协议。

---

## 三、具体应用场景:数据分析 Agent

以本课程的大数据背景为例,描述一个"**季度销量下滑归因分析 Agent**":

1. **下达开放式目标**:用户说"帮我分析三季度 A 系列销量为什么比二季度下滑,并出报告。"
2. **Agent 规划**:需要订单明细、去年同期对比、是否改价 / 停促销等信息。
3. **行动 1**:调用 SQL 工具查询订单库,得到分区销量 → **观察**:下滑集中在华东区域。
4. **再思考 + 行动 2**:写 Python 工具做"品类 × 区域"交叉统计并出图。
5. **再行动 3**:数据仍解释不了,调用外部搜索 / 舆情工具查该品类近期负面信息。
6. **汇总**:综合所有工具结果,输出带图表的归因报告与下一步建议。

关键点:**第 3~5 步"先查什么、要不要再查"都由模型根据上一步结果动态决定**,而不是程序员预先写死的 if-else。

### 更复杂的形态:多 Agent 协作

当任务足够复杂,可拆成多个专职 Agent 协作,例如:

- "数据工程 Agent":负责取数清洗;
- "分析师 Agent":负责统计建模;
- "汇报 Agent":负责撰写报告。

多个 Agent 之间通过 Handoff(交接)或共享状态协调工作(如 OpenAI Agents SDK 的多 Agent 模式)。

---

## 四、容易混淆的问题与使用边界

### 4.1 四组高频混淆点

**1. Agent vs 聊天机器人 / 单次工具调用**

会接 Function Calling、会"查一次资料再作答"的多轮对话,不一定是 Agent。Agent 的判断标准是**多步、由模型主导的决策闭环**。单步"检索后作答"通常叫 **增强型 LLM(Augmented LLM)**。

**2. Agent vs RAG(检索增强生成)**

- RAG 只解决"从知识库取相关文本"这一个动作,是一种工具 / 能力;
- Agent 是"编排者",可以把 RAG、搜索、代码执行都当作工具调用。
- 两者不是同一层概念:**RAG 常被 Agent 当作其中一条工具**。

**3. Agent vs Workflow / Chain(流水线)**

判断标准:**控制权在代码还是在模型?**

| | Workflow(工作流) | Agent(智能体) |
|---|---|---|
| 执行路径 | 代码预先写死,步骤固定 | 模型运行时动态决定 |
| 确定性 | 高 | 低 |
| 适用 | 任务固定、步骤可预测 | 开放式、步数不可预测 |
| 成本与延迟 | 较低、可控 | 较高,随步数增长 |

工程建议(Anthropic):**能用简单 Workflow 就不要上真 Agent**;自主性换来的是不确定的成本与错误复合风险。

**4. 单 Agent vs 多 Agent;自主 ≠ 智能**

- 多 Agent 不一定更强:通信开销、成本、错误复合都可能增加,除非子任务边界清晰(不同角色分工)才值得。
- Agent **没有"意图"或"意识"**,本质是"LLM 决定调哪个工具的循环"。
- **自主性是一个刻度盘**(从 0 人工到全自动),不是有 / 无的开关。

### 4.2 使用边界:什么时候不该用 Agent

- **任务固定、步骤可预测**:用 Workflow 或普通程序,更快、更便宜、更可控。
- **对延迟 / 吞吐敏感**(如高并发在线推理):LLM 决策循环太慢。
- **错误不可逆或代价高**(资金操作、删除数据、对外发言):必须加 Human-in-the-loop 审批点与护栏。
- **缺少可校验反馈**:每步模型决策都会放大错误率,若没有环境反馈(如代码测试、SQL 结果)或评估手段,不宜放开自主。
- **长任务超上下文**:需额外的记忆压缩 / 外挂存储设计。

### 4.3 什么情况适合用 Agent

1. 开放式问题、所需步数不可预测;
2. 每一步都能从环境拿到"真实反馈"来校验进展;
3. 已有护栏与停止条件(如最大迭代次数),可随时叫停。

---

## 五、参考资料(可核查来源)

权威与一手来源(可用于核查原始定义、机制与最佳实践):

1. **OpenAI 官方 Agent 定义**
   "Agents are applications that plan, call tools, collaborate across specialists, and keep enough state to complete multi-step work."
   https://developers.openai.com/api/docs/guides/agents

2. **OpenAI Agents SDK(Python)**
   含 Agent、Handoff、Guardrails、Sessions 等核心概念与代码示例。
   https://openai.github.io/openai-agents-python/

3. **LangChain Agents 概念文档**
   Agent = Model + Harness,以及工具、状态、护栏等组成说明。
   https://docs.langchain.com/oss/python/langchain/agents

4. **Anthropic《Building Effective Agents》**
   工程实践"设计圣经",重点区分 Workflow 与 Agent、何时该用 Agent。
   https://www.anthropic.com/engineering/building-effective-agents

5. **ReAct 原论文**
   Reasoning and Acting in Language Models —— Agent 决策循环(Reason → Act → Observe)机制出处。
   https://arxiv.org/abs/2210.03629

6. **《A Survey on Large Language Model based Autonomous Agents》**
   LLM 智能体的系统综述(规划、记忆、工具、多智能体等全景)。
   https://arxiv.org/abs/2308.11432

7. **Model Context Protocol(MCP)官方文档**
   统一 Agent 接入外部工具与数据的开放标准。
   https://modelcontextprotocol.io/

8. **经典人工智能教材概念**
   "Intelligent Agent"感知 - 决策 - 行动(PEAS)定义的理论源头。
   https://en.wikipedia.org/wiki/Intelligent_agent

---

## 附:快速记忆卡

```
一句话:Agent = 大脑(LLM)+ 手(工具)+ 记事本(记忆)+ 自己定节奏(决策循环)
核心循环:思考 → 行动 → 观察 → 再思考(ReAct)
关键判据:路径由代码定 = Workflow;路径由模型定 = Agent
最佳实践:先试 Workflow,不够再上 Agent;时刻留人工刹车
```

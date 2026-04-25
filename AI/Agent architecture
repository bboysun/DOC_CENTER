Agent 架构

Agent 架构深度解析
一、Agent 的本质
Agent 的核心是一个 感知-推理-行动 循环：LLM 作为"大脑"，通过工具与外部世界交互，根据反馈持续调整行为，直到完成目标。

                    ┌─────────────┐
                    │   用户目标    │
                    └──────┬──────┘
                           ▼
              ┌────────────────────────┐
              │      LLM（大脑）        │
              │  ┌──────────────────┐  │
              │  │ 推理 / 规划 / 反思 │  │
              │  └──────────────────┘  │
              └─────┬──────────┬──────┘
                    │          │
              ┌─────▼──┐  ┌───▼──────┐
              │  记忆   │  │   工具    │
              │短期/长期 │  │API/DB/搜索│
              └────────┘  └──────────┘
不同的 Agent 架构，本质上是 推理与行动的编排方式 不同。

二、ReAct 架构（Reasoning + Acting）
2.1 核心原理
ReAct 来自 2022 年 Yao et al. 的论文，核心思想：让 LLM 交替进行推理（Thought）和行动（Action），用推理指导行动，用行动结果修正推理。

这解决了两个单独方法的问题：

纯推理（Chain-of-Thought）：无法获取外部信息，容易幻觉
纯行动（Act-only）：没有推理过程，行动盲目
2.2 执行流程
用户问题: "贵州茅台最新季度的营收是多少？同比增长如何？"
    │
    ▼
┌─────────────────────────────────────────────────────┐
│ Step 1                                              │
│ Thought: 我需要查询贵州茅台最新的财报数据。先搜索     │
│          最新季度的营收数据。                         │
│ Action:  search("贵州茅台 2026Q1 财报 营收")         │
│ Observation: 贵州茅台2026Q1营收 412.37亿元...        │
├─────────────────────────────────────────────────────┤
│ Step 2                                              │
│ Thought: 拿到了Q1营收412.37亿。现在需要去年同期数据   │
│          来计算同比增长率。                           │
│ Action:  search("贵州茅台 2025Q1 财报 营收")         │
│ Observation: 贵州茅台2025Q1营收 378.52亿元...        │
├─────────────────────────────────────────────────────┤
│ Step 3                                              │
│ Thought: 2026Q1: 412.37亿, 2025Q1: 378.52亿         │
│          同比增长 = (412.37-378.52)/378.52 = 8.94%   │
│          数据齐全，可以回答了。                       │
│ Action:  finish(最终答案)                            │
└─────────────────────────────────────────────────────┘
2.3 Prompt 模板结构
You are a financial analyst agent. You have access to the following tools:
- search(query): Search financial databases
- calculator(expression): Perform calculations
- lookup_filing(company, period): Get SEC/exchange filings
Use the following format:
Question: the input question
Thought: reason about what to do next
Action: tool_name(arguments)
Observation: the result of the action
... (repeat Thought/Action/Observation as needed)
Thought: I now have enough information
Final Answer: the final answer
Question: {user_question}
2.4 优缺点
优点	缺点
简单直观，易于实现	线性执行，无法并行
推理过程可解释	容易陷入循环（反复调用相同工具）
每步都有反思，减少幻觉	长任务时上下文窗口容易爆满
适合大多数中等复杂度任务	无全局规划，"走一步看一步"
2.5 适用金融场景
单一股票/基金的信息查询与分析
简单的财务指标计算
实时行情问答
三、Plan-and-Execute 架构
3.1 核心原理
Plan-and-Execute 将 Agent 拆分为两个角色：

Planner（规划者）：先制定完整计划
Executor（执行者）：按计划逐步执行
核心洞察：复杂任务不应边想边做，而应先想清楚再做。规划和执行用不同的 LLM 调用（甚至不同的模型），避免上下文污染。

3.2 架构图
                        用户任务
                           │
                           ▼
                ┌─────────────────────┐
                │     Planner LLM     │
                │  "制定完整执行计划"   │
                └──────────┬──────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │        Task Plan          │
              │  Step 1: 收集财报数据      │
              │  Step 2: 收集行业数据      │
              │  Step 3: 计算关键指标      │
              │  Step 4: 同行对比分析      │
              │  Step 5: 生成分析报告      │
              └────┬─────────────────────┘
                   │
          ┌────────┼────────────────┐
          ▼        ▼                ▼
     ┌─────────┐ ┌─────────┐  ┌─────────┐
     │Executor │ │Executor │  │Executor │    每个 Step 可以
     │ Step 1  │ │ Step 2  │  │ Step N  │    是一个 ReAct Agent
     └────┬────┘ └────┬────┘  └────┬────┘
          │           │            │
          ▼           ▼            ▼
      Observation  Observation  Observation
          │           │            │
          └─────────┬─┘            │
                    ▼              │
          ┌─────────────────┐     │
          │   Replanner     │◄────┘
          │ "是否需要调整计划"│
          └────────┬────────┘
                   │
              ┌────┴────┐
              │ 计划OK？ │
              ├── Yes ──→ 继续执行下一步
              └── No ──→ 生成新计划（回到 Planner）
3.3 核心流程伪代码
def plan_and_execute(task: str):
    # Phase 1: Planning
    plan = planner_llm.generate_plan(task)
    # plan = ["收集茅台Q1财报", "收集五粮液Q1财报", "对比分析", "生成报告"]
    
    results = []
    for i, step in enumerate(plan.steps):
        # Phase 2: Execute each step (每一步可以是一个小型 ReAct Agent)
        result = executor_agent.execute(
            step=step,
            context=results  # 前序步骤的结果作为上下文
        )
        results.append(result)
        
        # Phase 3: Replan (检查是否需要调整后续计划)
        should_replan = replanner_llm.evaluate(
            original_plan=plan,
            completed=results,
            remaining=plan.steps[i+1:]
        )
        if should_replan.needed:
            plan = should_replan.new_plan  # 动态调整
    
    # Phase 4: Synthesize
    return synthesizer_llm.summarize(task, results)
3.4 与 ReAct 的本质区别
ReAct（扁平循环）:
  Think → Act → Observe → Think → Act → Observe → ... → Done
  所有步骤在同一个 LLM 调用上下文中
Plan-and-Execute（分层架构）:
  Plan: [Step1, Step2, Step3, Step4]    ← 高层规划（一次 LLM 调用）
    │
    ├─ Execute Step1: Think→Act→Obs     ← 底层执行（独立 LLM 调用）
    ├─ Replan: 计划是否需要调整？         ← 反思（独立 LLM 调用）
    ├─ Execute Step2: Think→Act→Obs
    ├─ ...
    └─ Synthesize: 汇总所有结果
3.5 优缺点
优点	缺点
全局视野，不会迷失方向	初始规划可能不准确（信息不足时）
每步独立上下文，不会窗口爆满	多次 LLM 调用，延迟和成本更高
支持动态调整计划（Replan）	Replan 的判断本身可能出错
适合复杂、多步骤任务	简单任务过度工程化
各步骤可并行执行	步骤间的依赖关系管理复杂
3.6 适用金融场景
完整的投研分析报告生成
跨市场/跨资产的综合分析
尽职调查（Due Diligence）流程自动化
多步骤合规审查
四、Multi-Agent 架构
4.1 核心原理
Multi-Agent 的核心思想：与其让一个超级 Agent 做所有事，不如让多个专家 Agent 分工协作。每个 Agent 有自己的角色定义、工具集、专业知识。

这模拟了真实团队的工作模式：分析师做分析、合规官审合规、经理做决策。

4.2 三大框架对比
CrewAI — 角色驱动型
                    ┌─────────────────────┐
                    │       Crew          │
                    │   (团队管理者)       │
                    └──────────┬──────────┘
                               │ 分配任务
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
        ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
        │  Agent 1     │ │  Agent 2     │ │  Agent 3     │
        │  角色: 分析师  │ │  角色: 研究员  │ │  角色: 审核员  │
        │  目标: 财务分析│ │  目标: 数据收集│ │  目标: 合规审查│
        │  工具: 计算器  │ │  工具: 搜索   │ │  工具: 规则库  │
        │  背景: 10年经验│ │  背景: 数据专家│ │  背景: 合规专家│
        └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
               │                │                │
               ▼                ▼                ▼
           Task 1           Task 2           Task 3
        "分析财务指标"    "收集行业数据"    "检查合规性"
               │                │                │
               └──────→ 结果传递 →──────→ 结果传递 →──→ 最终输出
CrewAI 的核心概念：

Agent：有角色（role）、目标（goal）、背景故事（backstory）的独立实体
Task：分配给 Agent 的具体任务，有明确的期望输出
Crew：Agent 集合 + 任务编排方式（顺序/并行）
Process：Sequential（流水线）或 Hierarchical（层级式，有经理 Agent 协调）
# CrewAI 示例
from crewai import Agent, Task, Crew, Process
analyst = Agent(
    role="资深金融分析师",
    goal="对上市公司进行深度财务分析",
    backstory="你有15年A股分析经验,擅长财务指标解读和行业对比",
    tools=[financial_db, calculator],
    llm=gemini_pro
)
researcher = Agent(
    role="行业研究员",
    goal="收集和整理行业数据与新闻",
    backstory="你精通Wind和Bloomberg终端,擅长数据挖掘",
    tools=[wind_api, news_search],
    llm=gemini_flash  # 研究员可以用更快更便宜的模型
)
compliance = Agent(
    role="合规审查官",
    goal="确保所有分析内容符合监管要求",
    backstory="你熟悉证监会、银保监会所有相关法规",
    tools=[regulation_db],
    llm=gemini_pro
)
# 任务定义 — 顺序执行，上一步输出作为下一步输入
task1 = Task(description="收集{company}最近4个季度的财报数据", agent=researcher)
task2 = Task(description="基于财报数据进行财务分析", agent=analyst)
task3 = Task(description="审查分析报告的合规性", agent=compliance)
crew = Crew(
    agents=[researcher, analyst, compliance],
    tasks=[task1, task2, task3],
    process=Process.sequential  # 流水线式
)
result = crew.kickoff(inputs={"company": "贵州茅台"})
AutoGen — 对话驱动型
            ┌────────────────────────────────────────────┐
            │           Group Chat Manager               │
            │        (控制发言顺序和终止条件)               │
            └──────────────────┬─────────────────────────┘
                               │
                 ┌─────────────┼─────────────┐
                 ▼             ▼             ▼
          ┌──────────┐  ┌──────────┐  ┌──────────┐
          │ Agent A  │  │ Agent B  │  │ Agent C  │
          │ 分析师    │←→│ 风控官   │←→│ 人类代理  │
          └──────────┘  └──────────┘  └──────────┘
                 ↑             ↑             ↑
                 └─────────────┴─────────────┘
                        多轮对话
                    A: "我认为该股PE偏高..."
                    B: "从风控角度,需要关注..."
                    C: "人工确认: 同意,继续"
                    A: "综合意见,最终建议..."
AutoGen 的核心概念：

ConversableAgent：所有 Agent 的基类，天然支持多轮对话
GroupChat：多个 Agent 在群聊中讨论，由 Manager 控制发言
Human Proxy：人类可以随时介入对话
核心理念：Agent 之间通过"聊天"协作，而非"任务分配"
与 CrewAI 的本质区别：CrewAI 是"经理分配任务给下属"，AutoGen 是"专家们围坐讨论"。

LangGraph — 状态图驱动型
                    ┌──────────────┐
                    │  START       │
                    └──────┬──────┘
                           ▼
                    ┌──────────────┐
                    │  Classifier  │ ← 判断任务类型
                    └──┬───────┬──┘
                       │       │
              ┌────────▼─┐  ┌─▼────────┐
              │ 简单查询   │  │ 深度分析   │
              │ (ReAct)   │  │ (Plan)    │
              └────┬──────┘  └─────┬────┘
                   │               │
                   │     ┌─────────▼──────────┐
                   │     │    并行执行          │
                   │     │  ┌──────┐ ┌──────┐  │
                   │     │  │数据   │ │行业   │  │
                   │     │  │收集   │ │研究   │  │
                   │     │  └──┬───┘ └──┬───┘  │
                   │     │     └───┬────┘      │
                   │     └─────────┼───────────┘
                   │               ▼
                   │        ┌──────────────┐
                   │        │   分析合成     │
                   │        └──────┬───────┘
                   │               ▼
                   │        ┌──────────────┐
                   │        │   合规审查     │
                   │        └──────┬───────┘
                   │               │
                   │        ┌──────▼───────┐
                   │        │  质量达标？    │
                   │        └──┬────────┬──┘
                   │      Yes  │        │ No
                   │           │    ┌───▼───────┐
                   │           │    │  修订循环   │──→ 回到分析合成
                   │           │    └───────────┘
                   ▼           ▼
                ┌──────────────────┐
                │    输出结果       │
                └──────────────────┘
LangGraph 的核心概念：

StateGraph：有向图，节点是函数/Agent，边是状态转移
State：全局共享状态（TypedDict），所有节点读写同一个 State
Conditional Edge：根据状态动态选择下一个节点（路由逻辑）
Checkpoint：自动持久化状态，支持暂停/恢复/时间旅行
# LangGraph 示例
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
class AnalysisState(TypedDict):
    task: str
    task_type: str              # "simple" | "deep"
    financial_data: dict        # 财务数据
    industry_data: dict         # 行业数据
    analysis: str               # 分析结果
    compliance_check: dict      # 合规审查结果
    quality_score: float        # 质量评分
    revision_count: int         # 修订次数
def classify_task(state: AnalysisState) -> AnalysisState:
    """判断任务复杂度"""
    # LLM 判断是简单查询还是深度分析
    state["task_type"] = llm.classify(state["task"])
    return state
def collect_data(state: AnalysisState) -> AnalysisState:
    """收集财务数据"""
    state["financial_data"] = financial_api.query(state["task"])
    return state
def analyze(state: AnalysisState) -> AnalysisState:
    """核心分析"""
    state["analysis"] = analyst_llm.analyze(
        state["financial_data"], 
        state["industry_data"]
    )
    return state
def compliance_review(state: AnalysisState) -> AnalysisState:
    """合规审查"""
    state["compliance_check"] = compliance_llm.review(state["analysis"])
    return state
def quality_gate(state: AnalysisState) -> str:
    """质量门控 — 决定走向"""
    if state["quality_score"] >= 0.8 or state["revision_count"] >= 3:
        return "output"
    return "revise"
# 构建状态图
graph = StateGraph(AnalysisState)
graph.add_node("classify", classify_task)
graph.add_node("collect_data", collect_data)
graph.add_node("analyze", analyze)
graph.add_node("compliance", compliance_review)
graph.add_node("revise", revise_analysis)
graph.set_entry_point("classify")
graph.add_conditional_edges("classify", route_by_type, {
    "simple": "simple_query",
    "deep": "collect_data"
})
graph.add_edge("collect_data", "analyze")
graph.add_edge("analyze", "compliance")
graph.add_conditional_edges("compliance", quality_gate, {
    "output": END,
    "revise": "revise"
})
graph.add_edge("revise", "analyze")  # 修订后重新分析
app = graph.compile(checkpointer=MemorySaver())
4.3 三大框架对比总结
维度	CrewAI	AutoGen	LangGraph
核心隐喻	公司团队	专家圆桌	工作流引擎
协作模式	任务分配	对话讨论	状态流转
控制粒度	粗（角色级）	中（对话级）	细（节点级）
灵活性	低（框架约束多）	中	高（完全自定义）
学习曲线	最低	中	最高
可调试性	低（黑盒多）	中	高（图可视化）
生产就绪度	低	中	最高（LangChain 生态）
适合场景	快速原型、角色明确	需要讨论/辩论	生产级复杂流程
金融推荐	投研报告生成（原型）	投委会模拟	生产级金融 Agent
五、Tool-Use（工具调用）
5.1 核心原理
Tool-Use 是所有 Agent 架构的 基础能力层。LLM 本身只能生成文本，通过工具调用才能：

获取实时数据（搜索、API）
执行精确计算（计算器、代码执行）
操作外部系统（数据库、交易系统）
5.2 Function Calling 机制
用户: "帮我计算茅台的市盈率"
         │
         ▼
┌──────────────────────────────────────────────────────┐
│ LLM 推理                                            │
│                                                      │
│ 我需要两个数据: 股价和每股收益                          │
│ 应该调用 get_stock_price 和 get_eps                   │
│                                                      │
│ 输出（结构化 JSON，不是自然语言）:                      │
│ {                                                    │
│   "tool_calls": [                                    │
│     {                                                │
│       "function": "get_stock_price",                 │
│       "arguments": {"symbol": "600519.SH"}           │
│     },                                               │
│     {                                                │
│       "function": "get_eps_ttm",                     │
│       "arguments": {"symbol": "600519.SH"}           │
│     }                                                │
│   ]                                                  │
│ }                                                    │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼  框架层执行工具调用
              ┌─────────────────────┐
              │  Tool Executor      │
              │  get_stock_price()  │──→ Wind API ──→ 1850.00
              │  get_eps_ttm()     │──→ Wind API ──→ 58.73
              └─────────┬───────────┘
                        │
                        ▼  将结果返回给 LLM
┌──────────────────────────────────────────────────────┐
│ LLM 第二轮                                           │
│                                                      │
│ 股价: 1850.00, EPS(TTM): 58.73                       │
│ PE = 1850.00 / 58.73 = 31.50                         │
│                                                      │
│ "贵州茅台当前市盈率(PE-TTM)为 31.50 倍"                │
└──────────────────────────────────────────────────────┘
5.3 工具定义标准（OpenAI Function Calling 格式）
{
  "type": "function",
  "function": {
    "name": "get_financial_statement",
    "description": "获取上市公司的财务报表数据。支持资产负债表、利润表、现金流量表。",
    "parameters": {
      "type": "object",
      "properties": {
        "symbol": {
          "type": "string",
          "description": "股票代码，如 600519.SH"
        },
        "statement_type": {
          "type": "string",
          "enum": ["balance_sheet", "income_statement", "cash_flow"],
          "description": "报表类型"
        },
        "period": {
          "type": "string",
          "description": "报告期，如 2026Q1"
        }
      },
      "required": ["symbol", "statement_type", "period"]
    }
  }
}
5.4 金融场景的工具体系设计
┌─────────────────────────────────────────────────────────┐
│                    Tool Registry                        │
│                  (工具注册中心)                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  数据获取层                                              │
│  ├── get_stock_price(symbol)          实时/历史股价       │
│  ├── get_financial_statement(...)     三大财务报表        │
│  ├── get_industry_data(sector)        行业数据           │
│  ├── search_news(query, date_range)   新闻搜索           │
│  └── get_analyst_reports(symbol)      券商研报           │
│                                                         │
│  计算分析层                                              │
│  ├── calculate(expression)            数学计算           │
│  ├── run_python(code)                 Python 沙箱执行    │
│  ├── statistical_analysis(data, method)  统计分析        │
│  └── generate_chart(data, chart_type)    图表生成        │
│                                                         │
│  知识检索层                                              │
│  ├── query_knowledge_base(query)      内部知识库 RAG     │
│  ├── lookup_regulation(keyword)       法规查询           │
│  └── get_company_profile(symbol)      公司基本面         │
│                                                         │
│  操作执行层（需权限控制）                                  │
│  ├── create_alert(condition)          设置预警           │
│  ├── generate_report(template, data)  生成报告           │
│  └── submit_order(...)                下单（需审批）      │
│                                                         │
└─────────────────────────────────────────────────────────┘
5.5 工具调用的安全设计（金融场景特别重要）
                 LLM 输出 tool_call
                        │
                        ▼
               ┌─────────────────┐
               │  参数校验层      │ ← 类型检查、范围检查、注入防护
               └────────┬────────┘
                        ▼
               ┌─────────────────┐
               │  权限控制层      │ ← 只读 vs 读写、审批流程
               │  - 查询类: 自动  │
               │  - 交易类: 人审  │
               └────────┬────────┘
                        ▼
               ┌─────────────────┐
               │  频率限制层      │ ← 防止 Agent 失控循环调用
               │  - 单工具: 10/分 │
               │  - 总调用: 50/次 │
               └────────┬────────┘
                        ▼
               ┌─────────────────┐
               │  审计日志层      │ ← 全部调用记录，可追溯
               └────────┬────────┘
                        ▼
                   执行工具调用
六、架构选型决策树（金融场景）
                        你的任务是什么？
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
            简单查询     结构化分析    需要讨论/辩论
            │           │           │
            ▼           ▼           ▼
          ReAct    复杂度如何？     AutoGen
                        │         (投委会模拟)
                ┌───────┼───────┐
                ▼               ▼
          中等复杂           高度复杂
          (单一分析)       (跨域/多步骤)
                │               │
                ▼               ▼
        Plan-and-Execute   需要生产部署？
                                │
                        ┌───────┼───────┐
                        ▼               ▼
                      Yes             No
                        │               │
                        ▼               ▼
                   LangGraph         CrewAI
                (状态图+检查点)    (快速原型验证)
实战建议：

大多数金融场景的生产系统 → LangGraph：控制粒度最细，支持持久化、错误恢复、人工审批节点
快速验证想法 → CrewAI：30 分钟搭出原型
需要多角色辩论（如投资决策） → AutoGen：天然支持多轮讨论
简单工具调用场景 → ReAct：不要过度设计
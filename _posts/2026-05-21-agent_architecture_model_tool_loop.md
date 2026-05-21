# 从 LangGraph 回到 Model-Tool Loop：更聪明的模型，正在让 Agent 架构重新变简单

这两年 Agent 框架的发展，有一个很有意思的反转。

一开始，大家觉得只靠大语言模型和工具调用不够可靠，于是拼命发明各种 Agent 架构：Planner、Executor、Router、SubAgent、Workflow、State Machine、LangGraph、多 Agent 协作、人工审批节点、条件边、状态持久化……

看起来越来越工程化，也越来越像传统软件系统。

但现在，一个相反的趋势正在出现：

> 模型越来越强之后，很多复杂 Agent 架构反而显得多余。最终最通用、最强大的形态，可能又回到了最经典的 `Model <> Tool` 循环。

这听起来有点反直觉。毕竟过去一两年，很多人都在强调：Agent 不是简单的工具调用，真正的 Agent 应该有规划、有状态、有图、有工作流。

但问题是：这些复杂结构到底是在增强模型，还是在束缚模型？

我越来越倾向于后者。

---

## 1. 早期 Agent 框架为什么会走向 LangGraph？

我们先不要急着批评 LangGraph。

早期显式 Agent 框架的出现是合理的。

在大语言模型能力还不稳定的时候，企业级应用最怕的不是模型不够聪明，而是模型不够听话。

它可能不按格式输出，可能跳过步骤，可能忘记系统提示，可能随便编工具参数，可能在应该调用工具的时候直接胡说，可能在应该等待人工确认的时候擅自执行。

那时候，如果你只是写一大段 prompt：

```text
You are a helpful enterprise assistant.
Please follow these rules...
Step 1...
Step 2...
Step 3...
Never do X...
Always do Y...
```

结果很可能是：今天能用，明天崩；简单任务能用，复杂任务崩；demo 能用，生产环境崩。

于是工程师自然会想：

> 既然模型不可靠，那就不要让它决定流程。我们把流程写死。

这就是 LangGraph 这类框架的土壤。

它们本质上是在用传统软件工程的方式约束 LLM：

```text
分类节点 -> 路由节点 -> 工具节点 -> 审批节点 -> 总结节点 -> 输出节点
```

每一步都有明确状态，每条边都有条件，每个节点都可以独立测试。

从企业工程视角看，这很诱人。因为它看起来可控、可观测、可审计、可恢复。

所以早期 Agent 框架不是错的。它们是在特定历史阶段，对“不可靠模型”的一种工程补丁。

---

## 2. 但问题来了：Prompt 爆炸被转移成了 Graph 爆炸

早期大家抱怨 prompt 爆炸。

所有规则都塞进 system prompt，最后 prompt 越来越长，越来越难维护。不同业务逻辑、工具说明、安全策略、输出格式、异常处理，全都混在一起。

于是我们引入了 graph，希望把逻辑拆开。

但拆着拆着，新的问题出现了：

> Prompt 爆炸没有消失，只是变成了 Graph 爆炸。

原来一个自然语言很容易表达的规则：

```text
当用户要求发邮件时，如果意图明确且收件人明确，可以发送；
如果用户只是让你帮忙写一封邮件，就只创建草稿；
如果内容敏感，需要人工确认。
```

如果写成图，就可能变成：

```text
intent_detection_node
recipient_validation_node
sensitivity_check_node
draft_or_send_router
human_approval_node
send_email_node
final_response_node
```

然后你还要定义 state schema、edge condition、error branch、retry branch、fallback branch。

这时候系统看起来很工程化，但也开始变得僵硬。

更讽刺的是，很多这种规则，本来正是 LLM 最擅长理解的东西。我们却用一堆代码节点把它提前硬编码了。

这就引出一个核心问题：

> 你到底是在利用 LLM 的智能，还是在努力绕开 LLM 的智能？

---

## 3. 严格代码流程的泛化能力，很多时候不如 LLM

传统软件工程有一个默认假设：明确流程优于模糊推理。

这在普通业务系统里当然成立。银行转账、库存扣减、权限校验、订单状态机，都应该由确定性代码控制。

但 Agent 面对的问题不完全一样。

Agent 经常处理的是半结构化任务：

```text
帮我整理这批邮件
看一下这个项目有没有风险
帮我分析这些日志
根据这几份文档写个回复
检查这段代码哪里可能有问题
```

这类任务的难点不在于“流程不够确定”，而在于“情况太多，无法提前枚举”。

如果你用 graph 强行拆流程，就会遇到一个问题：

> 你写下的每一条边，都是对未来场景的一次假设。

假设越多，系统越脆。

而 LLM 的优势恰恰是能在没有完全枚举规则的情况下，根据上下文做出合理判断。

所以对于大量通用 Agent 场景，严格工作流并不会让系统更聪明，只会让系统更像一个复杂但笨重的表单流程。

这就是为什么很多看起来很漂亮的多 Agent / LangGraph demo，真实落地时会变得很尴尬：

```text
图很复杂，效果一般。
节点很多，泛化很差。
架构很漂亮，维护很痛苦。
```

---

## 4. Skill 的出现，是一个关键转折

我认为 Skill 概念的出现非常重要。

因为它解决的是早期 Agent 框架真正想解决的问题：如何让模型稳定地执行某类任务。

但 Skill 的解决方式和 Graph 不一样。

Graph 的思路是：

> 你不可靠，所以我用代码管住你。

Skill 的思路是：

> 你已经足够聪明，所以我给你一份可执行的行为说明书。

这两者差别很大。

一个 Skill 可以包含：

```text
任务目标
操作步骤
注意事项
工具使用规则
输入输出格式
常见错误
示例
边界条件
```

它不是简单 prompt，也不是硬编码流程。它更像一个“可加载的专业操作手册”。

关键是：模型现在越来越能读懂并遵守这种手册。

早期模型可能看完 Skill 也乱来，所以工程师必须写 graph。

但如果模型已经能比较稳定地遵循 Skill，那么大量显式 graph 就不再必要。

此时更好的结构是：

```text
Model + Tools + Skills + Memory + Policy
```

而不是：

```text
Model hidden inside a giant workflow graph
```

---

## 5. LangChain 的 Middleware，其实暴露了这个趋势

LangChain 现在引入 middleware，是一个很有意思的信号。

表面上看，这是为了增强 `create_agent`。

但从架构角度看，它说明了一件事：

> 官方也意识到，不应该让用户为了扩展一个标准 Agent，就去手写完整 LangGraph。

`create_agent` 本质上是一个固定的标准 Agent loop。

大概就是：

```text
model -> tool -> model -> tool -> ... -> final answer
```

而 middleware 负责处理横切逻辑：

```text
before_model
after_model
wrap_model_call
wrap_tool_call
human-in-the-loop
PII detection
summarization
retry
fallback
logging
```

这其实很像 Web 框架。

在 Web 开发里，我们不会为了加日志、鉴权、限流、压缩、异常处理，就重新设计 HTTP 请求流程。我们会用 middleware、filter、interceptor。

Agent 也是一样。

如果标准 `Model <> Tool` loop 已经足够通用，那么大部分扩展都不应该改变图结构，而应该作为 middleware 插入运行时。

所以我甚至觉得 LangChain 现在的架构是在“去 LangGraph 化”：

```text
LangGraph 仍然是底层运行时，
但普通用户不应该直接面对它。
```

这不是说 LangGraph 没用。

而是说：

> LangGraph 不应该成为普通 Agent 扩展的默认接口。

---

## 6. “Email Agent” 是一个典型的抽象膨胀

LangChain 文档里有一类示例，会把 email 做成一个独立 agent，然后再放进 LangGraph workflow。

技术上当然可以。

但从抽象建模看，这很值得怀疑。

email 到底是什么？

如果只是发邮件、读邮件、搜索邮件，那它显然是 Tool 或 MCP Tool：

```text
read_email
search_email
send_email
create_draft
```

如果是“如何写一封专业邮件”“如何回复客户投诉”“如何按照公司语气写邮件”，那更像 Skill。

只有当任务变成：

```text
帮我处理今天所有邮件，判断哪些要回复，哪些要归档，哪些要提醒我。
```

这时候 email 才值得成为一个 Agent。

因为它有自己的目标、循环、判断、工具集合和中间状态。

所以问题不是“email agent 能不能做”。当然能。

问题是：

> 我们是不是太容易把一个能力域包装成 Agent 了？

这几年 Agent 框架里有一种抽象膨胀：

```text
File Agent
Email Agent
Calendar Agent
Browser Agent
GitHub Agent
Database Agent
CRM Agent
```

听起来很高级。

但很多时候，它们只是工具集合外面套了一层 LLM。

真正的判断标准应该是：

> 这个东西是否需要独立的目标驱动循环？

如果不需要，它就是 Tool。

如果需要专业说明，它是 Skill。

如果需要连接外部系统，它是 MCP。

如果需要自治循环，它才是 Agent。

---

## 7. Model-Tool Loop 为什么又回来了？

最经典的 Agent 结构其实非常简单：

```text
while not done:
    model reads context
    model decides whether to call a tool
    tool returns observation
    model continues or answers
```

也就是：

```text
Model <> Tool
```

早期大家觉得它太简单，不够工程化。

但现在回头看，它反而是最强的基本结构。

原因很简单：它保留了 LLM 的泛化能力。

你不需要提前规定所有分支。

你只需要给模型：

```text
有哪些工具
什么时候可以用
什么不能做
相关 Skill 是什么
用户上下文是什么
权限边界是什么
观察结果是什么
```

然后让模型自己组合。

这听起来像“把权力交给模型”。

是的。

但这正是 Agent 的意义。

如果所有流程都已经被代码写死，那你需要的不是 Agent，而是 workflow engine。

Agent 的价值恰恰在于：

> 它能在未完全预定义的场景中，根据上下文选择行动。

---

## 8. Graph 没死，但它应该退回正确的位置

这里必须避免另一个极端。

我不是说 LangGraph、workflow、state machine 都没用。

它们当然有用。

只是它们不应该成为所有 Agent 应用的默认表达方式。

Graph 适合什么？

适合这些场景：

```text
强合规流程
多阶段审批
确定性业务状态机
长周期任务恢复
人工节点
多 Agent 并行协作
需要审计和重放的企业流程
```

比如贷款审批、保险理赔、医疗流程、财务报销、企业采购，这些流程本来就应该被图化。

因为这里最重要的不是泛化，而是：

```text
可审计
可恢复
可证明
可监管
可回滚
```

但大量通用 AI 助手任务不属于这个范围。

对于这些任务，强行画图只会让系统更复杂。

更合理的分层应该是：

```text
Model-Tool Loop: 默认 Agent 运行时
Skills: 任务级行为说明
Tools/MCP: 外部能力接口
Memory: 长期和短期上下文
Middleware: 横切扩展
Graph: 例外情况下的外层编排
```

也就是说：

> Graph 应该编排 Agent，而不是替代 Agent 的思考。

---

## 9. 更聪明的模型，会让框架更薄

这可能是接下来几年 Agent 架构最大的趋势。

当模型弱的时候，框架必须很厚。

框架要帮它规划，帮它路由，帮它记忆，帮它纠错，帮它拆任务，帮它决定下一步。

但当模型强的时候，框架应该变薄。

框架只需要提供：

```text
上下文
工具
权限
记忆
技能
观察性
安全边界
```

至于如何完成任务，应该尽量交给模型。

这和传统软件架构有点反过来。

传统软件中，核心逻辑在代码里，模型只是一个函数。

而 Agent 系统中，模型越来越像核心 runtime，代码则变成它的工具、护栏和环境。

所以未来好的 Agent 平台，可能不是看它能画多复杂的图，而是看它能不能优雅地回答几个问题：

```text
如何给模型正确的工具？
如何按需加载 Skill？
如何管理上下文和记忆？
如何控制权限？
如何观察和调试模型行为？
如何在人类需要介入时介入？
如何让模型在边界内自由泛化？
```

这些问题，比“我能不能画一个复杂的 Agent 图”更重要。

---

## 10. 结论：当模型弱时，框架替模型思考；当模型强时，框架给模型边界

我对 Agent 架构演进的判断可以总结成一句话：

> 当模型弱时，框架要替模型思考；当模型强时，框架应该给模型边界。

早期 LangGraph、多 Agent、Planner-Executor 这些设计，是为了弥补模型能力不足。

它们在当时很合理。

但随着模型越来越能遵循指令、理解 Skill、使用工具、处理长上下文，过度复杂的图结构会越来越像一种历史包袱。

最终，最经典的 `Model <> Tool` loop 可能重新成为主流。

不是因为大家放弃了工程化。

恰恰相反，是因为工程化成熟之后，我们终于意识到：

> 真正应该被工程化的，不是模型的每一步思考路径，而是模型周围的运行环境。

工具要工程化。

权限要工程化。

Skill 加载要工程化。

Memory 要工程化。

观测和审计要工程化。

安全策略要工程化。

但模型的泛化能力，不应该被过早写死在一张图里。

所以，未来的 Agent 架构也许不是越来越复杂，而是越来越克制：

```text
一个强模型，
一组好工具，
一套清晰 Skill，
一个可靠运行时，
再加上必要的边界。
```

这可能比十几个 Agent、几十个节点、上百条边，更接近 Agent 系统的本质。

因为 Agent 的核心不是图。

Agent 的核心是：

> 在边界内，让模型行动。


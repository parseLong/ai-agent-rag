---
title: 万字详解面试题库｜框架篇（LangChain/LangGraph）
source: https://mp.weixin.qq.com/s?__biz=MzUxOTAwNTM2MQ==&mid=2247488426&idx=1&sn=ab9bf4c2e9e91c0888956d7a53cd043b&scene=21&poc_token=HDZMDGqjXqYdanLz_ecJMx7nskMqbJ1JcEqytITV
author:
published:
created: 2026-05-19
description: LangChain和LangGraph是目前构建LLM应用的主流框架，也是AI面试中的必考内容。一文搞定所有面试题。
tags:
  - 参考资料
  - 工具与框架
  - 面试备考
  - LangGraph
---

## 万字详解面试题库 - 框架篇

**LangChain** 和 **LangGraph** 是目前构建LLM应用的主流框架，也是AI面试中的必考内容。我今天整理了18道核心面试题，涵盖LangChain基础组件、Chain与Agent、LCEL表达式、LangGraph状态流、节点编排、条件分支等高频考点，每道题都配有详细参考答案和延伸追问，帮助大家系统掌握这两大框架。

---

## 题目一：LangChain的核心定位是什么，解决了什么问题

**参考答案：**

LangChain是一个专门开发大语言模型应用的框架。它主要解决三个问题：

第一， **标准化封装** 。LLM应用开发涉及模型调用、向量库、工具集成等多个环节，每个环节都有不同的提供商和API。LangChain把这些都封装成统一的接口，开发者不用关心底层差异。

第二， **快速组装** 。LangChain把常见功能抽象成独立组件，比如模型接口、Prompt模板、文档加载器、文本分割器、向量存储等。开发者可以像搭积木一样组合这些组件，几行代码就能搭出一个RAG应用或Agent。

第三， **复杂流程编排** 。除了基础组件，LangChain还提供了Chain链式调用、Memory记忆管理、Agent自主决策等高级能力，让复杂任务的实现变得简单。

LangChain降低了LLM应用开发的门槛，是目前最流行的LLM开发框架之一。

**延伸追问：**

面试官可能会问LangChain的 **优缺点** 。回答要点是优点是生态丰富、组件齐全、快速上手。缺点是过度封装隐藏细节、版本迭代快API变化大、生产环境有一定性能开销。

---

## 题目二：LangChain的五大核心组件是什么

**参考答案：**

LangChain的五大核心组件分别是：

**Models（模型）** ：封装各类LLM和Embedding模型，支持OpenAI、Anthropic、本地模型等多种提供商，可以无缝切换。

**Prompts（提示词）** ：提供PromptTemplate模板管理，支持变量插值、FewShot示例、ChatMessage消息模板等，让Prompt管理更规范。

**Chains（链）** ：把多个组件串联起来形成执行流程，前一个组件的输出作为后一个组件的输入。LangChain预置了很多常用Chain，比如LLMChain、RetrievalQA、SQLDatabaseChain等。

**Memory（记忆）** ：在对话中保持上下文，支持BufferMemory完整保存、SummaryMemory自动总结、VectorStoreMemory向量检索等多种实现。

**Tools & Agents（工具与智能体）** ：Tools封装外部功能供Agent调用，Agent则根据用户输入自主决策是否调用工具、调用哪个工具、传什么参数。

这五个组件覆盖了LLM应用开发的核心环节。

**延伸追问：**

可能会问Chain和Agent的区别。回答要点是Chain是固定的执行流程，数据按预设路径流动。Agent是 **动态决策** ，模型自己判断要不要调用工具，更灵活但也更难控制。

---

## 题目三：LLM和ChatModel有什么区别

**参考答案：**

LangChain中区分LLM和ChatModel主要是适配不同类型的模型接口：

**LLM** 是传统的文本补全模型，输入输出都是纯字符串。比如早期的GPT-3，你给它一段文本，它继续生成后面的内容。

**ChatModel** 是对话模型，基于消息结构交互。输入是HumanMessage、SystemMessage、AIMessage等角色消息组成的列表，输出也是消息对象。ChatGPT、通义千问、文心一言都属于这类。

实际开发中ChatModel是主流用法，因为它支持角色区分，可以用SystemMessage设定模型行为，用HumanMessage传递用户输入，对话管理更清晰。

LangChain对两者做了统一封装，但内部处理逻辑不同。选型时要看模型提供商的API类型，选择对应的封装类。

**延伸追问：**

可能会问什么时候还用LLM。回答要点是一些传统文本生成场景，比如文章续写、代码补全，或者调用一些老版本的模型API时可能用到。

---

## 题目四：LangChain实现RAG的核心流程是什么

**参考答案：**

LangChain实现RAG的流程分为两个阶段：

**索引阶段（离线）：** 第一步，文档加载。用Document Loader加载各种格式的文档，比如PDF、Word、网页等。 第二步，文本分割。用TextSplitter把长文档切分成小块Chunk，适配模型上下文限制。 第三步，向量化。用Embedding模型把文本Chunk转换成向量。 第四步，存储索引。把向量存入VectorStore向量数据库，同时保存原始文本作为元数据。

**查询阶段（在线）：** 第一步，问题向量化。把用户问题用同样的Embedding模型转成向量。 第二步，相似度检索。在向量库中搜索最相似的几个文档片段。 第三步，上下文拼接。把检索到的片段和用户问题一起拼成Prompt。 第四步，生成答案。调用LLM基于上下文生成回答。

LangChain的RetrievalQA链把查询阶段的后三步封装在一起，几行代码就能实现RAG问答。

**延伸追问：**

可能会问文本分割的Chunk大小怎么选。回答要点是要考虑Embedding模型的上下文限制，考虑检索粒度，通常几百个Token比较合适，太大检索不精确，太小信息不完整。

---

## 题目五：LangChain的Memory机制有哪些实现方式

**参考答案：**

LangChain提供了多种Memory实现来应对不同场景：

**ConversationBufferMemory** 是最简单的方式，完整保存全部对话历史，直接拼接到Prompt中。适合对话轮数少的场景，缺点是历史太长会超出模型上下文限制。

**ConversationBufferWindowMemory** 只保留最近K轮对话，通过滑动窗口控制长度。解决了超长问题，但可能丢失早期的重要信息。

**ConversationSummaryMemory** 用LLM定期总结对话历史，把长篇对话压缩成简短摘要。既保留了关键信息又控制了长度，但增加了调用成本。

**VectorStoreRetrieverMemory** 把对话历史存到向量数据库，检索时根据问题召回相关的历史片段。适合需要长期记忆的场景。

实际使用中要根据对话长度、成本预算、记忆需求来选择合适的Memory类型，也可以组合使用。

**延伸追问：**

可能会问Memory和Prompt长度冲突怎么处理。回答要点是可以设置max\_token\_limit限制Memory占用的Token数，或者用SummaryMemory压缩历史，或者用WindowMemory截断保留最近几轮。

---

## 题目六：什么是LCEL，它有什么优势

**参考答案：**

LCEL是LangChain Expression Language的缩写，是LangChain推出的声明式链式语法。它用管道符 `|` 来组合组件，写法类似Unix管道。

举个例子，一个完整的RAG链用LCEL写大概是这样：

```
retriever | format_docs | prompt | llm | output_parser
```

LCEL的优势有几个：

**语法简洁** 。几行代码就能表达复杂的链式流程，比传统的类继承方式清晰很多。

**支持流式输出** 。LCEL链天然支持流式响应，不用额外配置。

**支持异步执行** 。组件可以异步调用，提高整体性能。

**可观测性好** 。每个步骤的输入输出都能方便地查看和调试。

**易于组合复用** 。定义好的LCEL片段可以像函数一样在其他地方复用。

LCEL是LangChain推荐的新式写法，新项目中建议优先使用。

**延伸追问：**

可能会问LCEL和传统Chain的区别。回答要点是LCEL更轻量、更灵活、写法更现代，但传统Chain在一些复杂场景可能更成熟稳定，两者可以混用。

---

## 题目七：Stuff、MapReduce、Refine三种文档链有什么区别

**参考答案：**

这三种都是LangChain处理长文档的文档链策略，区别主要在于怎么处理多个文档片段：

**Stuff** 是最简单的方式，把所有检索到的文档片段一次性塞进Prompt，然后调用LLM生成答案。优点是简单高效，只调用一次模型。缺点是受限于模型上下文长度，片段太多会超限制。适合文档较短或检索结果少的场景。

**MapReduce** 是分而治之的思路。先把每个文档片段分别传给模型处理（Map阶段），得到中间结果，然后再把这些中间结果汇总生成最终答案（Reduce阶段）。适合超长文档，但会调用多次模型，成本较高。

**Refine** 是迭代优化的思路。先基于第一个片段生成初始答案，然后逐个把后续片段和当前答案一起传给模型进行优化，不断 refine 直到处理完所有片段。上下文连贯性好，但耗时最长，适合需要深度整合多个文档内容的场景。

选型时要权衡文档长度、成本预算、答案质量要求。

**延伸追问：**

可能会问MapReduce的Reduce阶段怎么做。回答要点是把Map阶段生成的多个中间结果拼接或总结，再传给模型生成最终答案，也可以用递归的方式多层Reduce。

---

## 题目八：LangChain Agent的工作原理是什么

**参考答案：**

LangChain Agent的核心是 **让LLM自主决策** 。

Agent的工作流程是这样的：

第一步，接收用户输入，结合对话历史和可用工具描述，生成Prompt传给LLM。

第二步，LLM判断是直接回答还是调用工具。如果需要调用工具，输出遵循ReAct格式，包含Thought（思考过程）和Action（工具调用指令）。

第三步，框架解析Action，提取工具名和参数，执行对应的工具函数，获取Observation（观察结果）。

第四步，把Observation追加到上下文，再次传给LLM，让模型基于工具返回的结果继续决策。

第五步，循环执行直到LLM决定输出最终答案。

整个过程的关键是模型拥有自主决策权，自己判断什么时候调用工具、调用哪个、传什么参数。LangChain预置了多种Agent类型，比如zero-shot-react-description适合通用任务，structured-chat-zero-shot支持多参数工具调用。

**延伸追问：**

可能会问Agent调用工具失败怎么办。回答要点是框架会捕获异常，把错误信息作为Observation传给模型，让模型决定重试、换工具还是直接回答。

---

## 题目九：如何在LangChain中自定义工具

**参考答案：**

LangChain定义工具有两种方式：

**第一种是用@tool装饰器** ，这是最简洁的推荐方式。把一个普通函数用@tool装饰，LangChain会自动从函数名和docstring生成工具描述。

```
@tool
def search_api(query: str) -> str:
    """搜索API，输入查询词返回搜索结果"""
    return requests.get(f"https://api.example.com/search?q={query}").text
```

**第二种是继承BaseTool类** ，适合需要更精细控制的场景。需要定义name、description、args\_schema等属性，重写\_run方法实现工具逻辑。

```
class SearchTool(BaseTool):
    name = "search"
    description = "搜索工具"
    args_schema = SearchInput
    
    def _run(self, query: str) -> str:
        return requests.get(f"https://api.example.com/search?q={query}").text
```

工具定义好后，把工具列表传给Agent，Agent就能在运行时调用它们。工具的描述要写清楚功能、参数含义、返回值格式，让模型能正确选择和使用。

**延伸追问：**

可能会问工具描述怎么写效果好。回答要点是描述要说清楚工具功能是什么、参数分别代表什么、返回值是什么格式、在什么场景下使用，越具体越好。

---

## 题目十：LangGraph和LangChain Chain的最大区别是什么

**参考答案：**

LangGraph和LangChain Chain的最大区别在于 **架构范式** 。

**LangChain Chain** 是基于链式结构的，数据线性流动，从A到B到C，或者简单的分支。适合顺序执行、流程固定的场景。Chain一旦定义好，执行路径就确定了。

**LangGraph** 是基于图结构的，支持复杂的分支、循环、并行。节点之间通过边连接，可以有条件边根据状态动态决定下一步去哪。适合多步骤复杂任务、需要循环迭代的场景、多Agent协作。

打个比方，Chain像是一条流水线，物料按固定顺序经过各个工位。LangGraph像是一张交通网，车辆可以根据路况选择不同路线，可以绕圈，可以并行多条路。

LangGraph专为复杂Agent设计，可以建模反思优化、审批流程、多角色协作等复杂场景。两者可以无缝集成，LangGraph的节点里可以使用LangChain的Chain和Agent。

**延伸追问：**

可能会问什么场景适合用LangGraph。回答要点是多步骤审批流程、多角色协作任务、需要循环迭代的任务（比如生成-测试-修复）、条件分支复杂的流程。

---

## 题目十一：LangGraph的三大核心概念是什么

**参考答案：**

LangGraph的三大核心概念是State、Node、Edge。

**State** （状态）是全局共享的数据结构，贯穿整个图流程。所有节点都可以读写State，用来保存对话历史、工具执行结果、中间变量等。State用TypedDict定义，类型安全且支持自动合并。

**Node** （节点）是最小执行单元，代表一个具体的处理步骤。Node可以是函数调用、LLM调用、工具执行等。Node接收State作为输入，返回对State的更新。LangGraph中一切操作都在Node里完成。

**Edge** （边）定义节点之间的流转规则。普通Edge是无条件跳转，从一个节点固定到下一个节点。条件Edge根据State的值动态决定下一个节点去哪，实现分支逻辑。

这三个概念配合起来，State负责数据共享，Node负责逻辑执行，Edge负责流程控制，构成了LangGraph完整的工作流编排能力。

**延伸追问：**

可能会问State的数据结构怎么设计。回答要点是用TypedDict定义，字段类型要明确，通常包含messages消息列表、中间结果字段、控制标志等，根据业务需求来设计。

---

## 题目十二：LangGraph的StateGraph是什么

**参考答案：**

StateGraph是LangGraph的核心构图类，相当于整个图的"画布"。

StateGraph的主要作用有四个：

第一， **定义状态结构** 。初始化StateGraph时要传入State的TypedDict定义，确定整个图共享的数据结构。

第二， **添加节点** 。用add\_node方法注册各个业务节点，每个节点绑定一个处理函数。

第三， **配置边** 。用add\_edge添加普通边，用add\_conditional\_edges添加条件边，定义节点之间的流转规则。

第四， **编译运行** 。调用compile方法把图定义编译成可运行的实例，然后传入初始State触发执行。

StateGraph是声明式的，先定义好图的结构，再一次性编译执行。这种设计让复杂工作流的定义变得清晰，也便于可视化理解和调试。

**延伸追问：**

可能会问StateGraph和普通Graph有什么区别。回答要点是StateGraph强调状态驱动，所有节点共享同一个State对象，而普通Graph可能只是定义节点连接关系，不一定有统一的状态管理。

---

## 题目十三：LangGraph的条件边是什么，怎么用

**参考答案：**

条件边Conditional Edge是LangGraph实现动态分支的核心机制。

普通边是固定的，A节点执行完一定到B节点。条件边则根据当前State的值动态决定下一步去哪。

使用条件边需要三个要素：

**源节点** ：条件边从哪个节点出发。

**条件函数** ：接收State作为参数，返回一个字符串标识。

**映射字典** ：把字符串标识映射到目标节点。

举个例子，一个审批流程中，条件函数检查State中的审批结果字段，如果通过返回"approved"，映射到结束节点；如果拒绝返回"rejected"，映射到修改节点。

条件边让图具备了运行时决策能力，是实现循环、分支、多路径的关键。条件函数要保持纯函数特性，只依赖State不依赖外部状态，确保可预测性。

**延伸追问：**

可能会问条件分支和RouterChain的区别。回答要点是RouterChain是Chain内部的路由，条件边是图级别的流程控制，LangGraph更灵活，可以实现更复杂的流转逻辑。

---

## 题目十四：LangGraph如何实现循环和迭代

**参考答案：**

LangGraph通过 **边的循环引用** 实现循环执行。

具体做法是在add\_edge时，把目标节点设为之前执行过的节点，形成闭环。比如：

```
graph.add_edge("generate", "evaluate")  # 生成后到评估
graph.add_edge("evaluate", "generate")  # 评估后回到生成，形成循环
```

实际使用中循环要有终止条件，否则会变成死循环。通常的做法是：

在State中设置计数器或标志位，比如retry\_count记录迭代次数，is\_complete标记是否完成。

条件函数检查这些字段，如果达到最大迭代次数或任务已完成，返回结束标识跳出循环；否则返回继续标识保持循环。

这种循环机制让LangGraph可以处理需要多轮迭代的任务，比如代码生成中的生成-测试-修复循环，内容创作中的生成-评估-优化循环。

**延伸追问：**

可能会问如何防止无限循环。回答要点是在State中设置最大迭代次数上限，条件函数检查次数，达到上限强制退出；或者设置超时机制，防止长时间运行。

---

## 题目十五：LangGraph如何实现多Agent协作

**参考答案：**

LangGraph构建多Agent系统的典型模式是： **每个Agent作为一个Node，通过State共享信息，通过Edge协调执行** 。

具体实现步骤：

第一步，定义Agent Node。每个Node内部封装一个Agent，可以是LangChain的Agent，也可以是自定义逻辑。

第二步，设计State结构。包含messages消息列表、任务分配字段、各Agent的中间结果等，确保Agent之间能交换信息。

第三步，编排执行流程。用条件边实现任务分配，比如根据问题类型路由到不同Agent；用普通边定义Agent之间的执行顺序。

常见的协作模式有：

**监督者模式** ：一个Supervisor Agent负责协调多个Worker Agent，决定任务分配给哪个Worker，汇总Worker的结果。

**流水线模式** ：多个Agent按顺序处理，前一个Agent的输出作为后一个的输入，像流水线一样传递。

**多轮协商模式** ：多个Agent反复讨论，通过State交换意见，直到达成共识。

**延伸追问：**

可能会问Agent之间如何通信。回答要点是通过State共享，一个Agent把结果写入State的特定字段，其他Agent从State读取；也可以通过消息列表字段传递对话式的消息。

---

## 题目十六：LangGraph的持久化机制是什么

**参考答案：**

LangGraph的持久化机制用于保存图执行的State和会话记录，支持断点续传和历史会话恢复。

持久化的核心概念是 **检查点Checkpoint** 。在图执行的关键节点，框架会自动把当前State保存到存储中。如果执行中断，下次可以从检查点恢复继续执行。

LangGraph支持多种存储后端：

**内存存储** ：数据存在内存中，重启丢失，适合开发调试。

**SQLite** ：本地文件存储，轻量便捷，适合小规模应用。

**PostgreSQL/Redis** ：生产级存储，支持分布式部署，数据可靠性高。

持久化的价值在于：

第一， **容错恢复** 。长时间运行的任务如果中断，可以从检查点恢复而不是从头开始。

第二， **历史回溯** 。可以查看任意时刻的State状态，方便调试和审计。

第三， **会话保持** 。用户的对话历史可以持久化，下次访问时恢复上下文。

**延伸追问：**

可能会问检查点对性能的影响。回答要点是会增加IO开销，可以配置检查点策略，比如只保存关键节点、异步保存、批量保存等来减少影响。

---

## 题目十七：LangGraph的完整执行流程是怎样的

**参考答案：**

LangGraph的完整执行流程分为六个步骤：

**第一步，定义State** 。用TypedDict定义全局状态的数据结构，包含消息列表、中间变量、控制标志等字段。

**第二步，初始化StateGraph** 。创建StateGraph实例，绑定State类型。

**第三步，注册Node** 。用add\_node添加各个业务节点，配置每个节点的处理函数。节点可以是LLM调用、工具执行、自定义函数等。

**第四步，配置边** 。设置入口节点，用add\_edge添加普通流转边，用add\_conditional\_edges添加条件分支边，定义完整的流程图。

**第五步，编译图** 。调用compile方法把图定义编译成可运行的实例。

**第六步，触发执行** 。传入初始State，调用invoke或stream方法启动执行。节点按边的规则流转，直到到达结束节点，返回最终的State。

整个流程是声明式的，先定义图的结构，再执行。这种设计让复杂工作流的逻辑清晰可控。

**延伸追问：**

可能会问stream和invoke的区别。回答要点是invoke是同步执行，返回最终结果；stream是流式执行，可以实时获取每个节点的输出，适合需要展示中间过程的场景。

---

## 题目十八：LangChain和LlamaIndex有什么区别，如何选择

**参考答案：**

LangChain和LlamaIndex都是LLM应用开发框架，但侧重点不同：

**LangChain更通用** ，提供完整的组件体系，覆盖模型调用、Prompt管理、Chain编排、Agent实现、工具集成等全流程。设计理念是灵活组合，开发者可以自由搭建各种架构。生态更丰富，社区更活跃。

**LlamaIndex更专注** ，聚焦于RAG和知识检索场景。提供数据加载、索引构建、查询引擎等专业化组件。设计理念是开箱即用，针对检索场景做了深度优化。索引类型更丰富，支持向量索引、树形索引、列表索引、关键词索引等。

具体选择建议：

如果你的项目是做通用LLM应用，涉及多种功能组合，选LangChain。

如果你的项目专注做RAG知识库，对检索质量要求高，选LlamaIndex。

两者也可以组合使用，比如用LlamaIndex做文档索引和检索，用LangChain做Chain编排和Agent实现，各取所长。

**延伸追问：**

可能会问两个框架的学习曲线如何。回答要点是LangChain组件多、概念多，学习曲线较陡但功能更全面。LlamaIndex更聚焦，上手相对简单，但做复杂流程可能不够灵活。
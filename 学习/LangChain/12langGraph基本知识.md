LangGraph和LangChain同宗同源，底层架构完全相同、接口完全相通。LangGraph本质上是LangChain的更高级的编排工具，核心是graph，主要功能是搭建图状的工作流。
LangGraph是基于LangChain进行的构建，**无论图结构多复杂，单独每个任务执行链路仍然是线性的，其背后仍然是靠着LangChain的Chain来实现的**。
LangGraph的本质是DAG（有向无环图），但它扩展支持了“有状态循环图”，本质上是一种 可控状态机（State Machine）。
LangGraph是一个用于构建具有LLMs的有状态、多角色应用程序的库，用于创建代理和多代理工作流。与其他LLM框架相比，它提供了以下核心优势：循环、可控性(子图)和持久性(记忆)
LangGraph的灵感来自Pregel和Apache Beam。公共接口借鉴了NetworkX.
LangGraph的高层API主要分为两层，其一是Agent API，用于将大模型、提示词模板、外部工具等关键元素快速封装为图中的一些节点，而更高一层的封装，则是进一步创建一些预构建的Agent、也就是预构建好的图，开发者只需要带入设置好要带入的工具和模型，三行代码就能借助这些预构建好的图，创建一个完整的Agent：
```python
tools=[search_tool,python_inter,fig_inter]
model=ChatDeepSeek(model="deepseek-chat")
graph=create_react_agent(model=model,tools=tools)
```
![[Pasted image 20250710112112.png]]

**运行监控框架：LangSmith** https://docs.smith.langchain.com/
LangSmith 是一款用于构建、调试、可视化和评估 LLM 工作流的全生命周期开发平台。

| 功能类别                           | 描述                                   | 场景                   |
| ------------------------------ | ------------------------------------ | -------------------- |
| 🧪 调试追踪（Trace Debugging）       | 可视化展示每个 LLM 调用、工具调用、Prompt、输入输出      | Agent 调试、Graph 调用链分析 |
| 📊 评估（Evaluation）              | 支持自动评估多个输入样本的回答质量，可自定义评分维度           | 批量测试 LLM 表现、A/B 对比   |
| 🧵 会话记录（Sessions / Runs）       | 每次 chain 或 agent 的运行都会被记录为一个 Run，可溯源 | Agent 问题诊断、用户问题分析    |
| 🔧 Prompt 管理器（Prompt Registry） | 保存、版本控制、调用历史 prompt                  | 多版本 prompt 迭代测试      |
| 📈 流量监控（Telemetry）             | 实时查看运行次数、错误率、响应时间等                   | 在生产环境中监控 Agent 质量    |
| 📁 Dataset 管理                  | 管理自定义测试集样本，支持自动化评估                   | 微调前评估、数据对比实验         |
| 📜 LangGraph 可视化               | 对 LangGraph 中每个节点运行情况进行实时可视化展示       | Graph 执行追踪           |

**可视化与调试框架：LangGraph Studio** https://www.langgraph.dev/studio
LangGraph Studio 是一个用于可视化构建、测试、分享和部署智能体流程图的图形化 IDE + 运行平台。

| 功能模块            | 说明                                        | 应用场景             |
| --------------- | ----------------------------------------- | ---------------- |
| 🧩 Graph 编辑器    | 以拖拽方式创建节点（工具、模型、Router）并连接                | 零代码构建 LangGraph  |
| 🔍 节点配置器        | 每个节点可配置 LLM、工具、Router 逻辑、Memory           | 灵活定制 Agent 控制流   |
| ▶️ 即时测试         | 输入 prompt 可在浏览器中运行整个图                     | 实时测试执行结果         |
| 💾 云端保存 / 分享    | 将构建的 Graph 保存为公共 URL / 私人项目               | 团队协作，Demo 分享     |
| 📎 Tool 插件管理    | 可连接自定义工具（MCP）、HTTP API、Python 工具          | 插件式扩展 Agent 功能   |
| 🔁 Router 分支节点  | 创建条件分支，支持 if/else 路由                      | 决策型智能体           |
| 📦 上传文档 / 多模态   | 可以上传文件（如 PDF）并嵌入进图中处理流程                   | RAG 结构、OCR、图文问答等 |
| 🧠 Prompt 输入/预览 | 编辑 prompt 并观察其运行效果                        | Prompt 工程调试      |
| 📤 一键部署         | 将 Graph 部署为可被 Agent Chat UI 使用的 Assistant | 快速集成到前端          |

**服务部署工具：LangGraph Cli** https://www.langgraph.dev/
LangGraph CLI 是用于本地启动、调试、测试和托管 LangGraph 智能体图的开发者命令行工具。

| 功能类别               | 命令示例                                                  | 说明                                      |
| ------------------ | ----------------------------------------------------- | --------------------------------------- |
| ✅ 启动 Graph 服务      | `langgraph dev`                                       | 启动 Graph 的开发服务器，供前端（如 Agent Chat UI）调用  |
| 🧪 测试 Graph 输入     | `langgraph run graph:graph --input '{"input": "你好"}'` | 本地 CLI 输入测试，输出结果                        |
| 🧭 管理项目结构          | `langgraph init`                                      | 初始化一个标准 Graph 项目目录结构                    |
| 📦 部署 Graph（未来）    | `langgraph deploy`（预留）                                | 发布 graph 至 LangGraph 云端（已对接 Studio）     |
| 🧱 显示 Assistant 列表 | `langgraph list`                                      | 显示当前 graph 中有哪些 assistant（即 entrypoint） |
| 🔄 重载运行时           | 自动热重载                                                 | 修改 `graph.py` 时，`dev` 模式自动重启生效          |
官方也提供了云托管服务，借助LangGraph Platform，开发者可以将构建的智能体 Graph部署到云端，并允许公开访问，同时支持支持长时间运行、文件上传、外部 API 调用、Studio 集成等功能。

**前端可视化工具：Agent Chat UI** https://langchain-ai.github.io/langgraph/agents/ui/
Agent Chat UI 是 LangGraph/LangChain 官方提供的多智能体前端对话面板，用于与后端 Agent（Graph 或 Chain）进行实时互动，支持上传文件、多工具协同、结构化输出、多轮对话、调试标注等功能。

| 功能模块                  | 描述                            | 应用场景                       |
| --------------------- | ----------------------------- | -------------------------- |
| 💬 多轮对话框              | 类似 ChatGPT 的输入区域，支持多轮上下文      | 用户提问，Agent 回复              |
| 🛠 工具调用轨迹显示           | 显示每个调用的工具、参数、结果（结构化）          | Agent 推理透明化                |
| 📄 上传 PDF / 图片        | 支持上传文档、图片、嵌入多模态输入             | RAG、OCR、图像问答               |
| 📁 文件面板               | 可查看上传历史文件、删除、重新引用             | 管理文档输入                     |
| 🧭 Assistant 切换       | 支持切换不同 Assistant（Graph entry） | 一键切换模型能力（如 math / weather） |
| 🧩 插件支持               | 与 MCP 工具、LangGraph 图打通        | 工具式 Agent 调用               |
| 🔍 调试视图               | 显示每轮 Agent 的思维过程和中间状态         | Prompt 调试、模型行为分析           |
| 🌐 云部署支持              | 支持接入远端 Graph API（如 dev 服务器）   | 前后端分离部署                    |
| 🧪 与 LangSmith 对接（可选） | 若后端启用 tracing，可同步显示运行 trace   | 调试闭环                       |

OpenAI的Agents SDK最核心的优势还是在于和OpenAI的GPT系列模型深度集成，能够非常便捷的调用GPT模型的很多内置工具（如网络搜索、RAG、Computer use等），并且上手难度非常低，能够非常简单的完成Agent创建。因此如果是GPT模型开发者，Agents SDK还是非常合适的。
谷歌的ADK的优势在于和Gemini多模态功能深度适配，可以说是原生的多模态Agent开发框架，原生就支持图像、文本、音频等多模态交互功能，并且和谷歌云在线API深度集成，可以直接调用谷歌云API（例如谷歌云盘、谷歌邮箱API、谷歌搜索等）来完成智能体开发。也提供了调试前端UI页面以及智能体运行追踪等功能，同时支持一键部署上线

#### R1+qwen3+mcp
内容可以参考MCP文件夹下面的qwen3系列笔记
配置所使用的模型服务
```Python
llm_cfg = {
    'model': 'deepseek-reasoner',
    'model_server': 'https://api.deepseek.com',
    'api_key':DS_API_KEY
    'fncall_prompt_type': 'nous', # 使用nous格式避免推理模型遇到stop符号停止
}
```
核心代码和Qwen-Agent接入MCP工具的qwen3_mcp_sqlite.py文件一样，只需要修改llm_cfg.

#### OpenAI Agents SDK 接入DS-R1
GitHub官网：https://github.com/openai/openai-agents-python/
 Agents SDK博客：https://openai.github.io/openai-agents-python/
 **OpenAI Agents SDK** 让你能够通过一个轻量、易用、抽象极少的工具包来构建基于智能体的 AI 应用。它是我们此前 Agent 实验项目 **Swarm** 的一个面向生产环境的升级版本。该 SDK 仅包含极少量的原语（基础构件）：
- **Agent（智能体）**：即带有指令和工具的大语言模型（LLM）
- **Handoff（交接）**：允许智能体将特定任务委托给其他智能体
- **Guardrail（护栏）**：用于对输入内容进行验证

结合 Python 使用时，这些原语足够强大，能够表达工具与智能体之间的复杂关系，并让你在没有高学习成本的前提下构建真实可用的应用程序。此外，SDK 自带内置的追踪功能，可以帮助你可视化和调试智能体的执行流程，同时也支持对流程进行评估，甚至用于模型的微调。
OpenAI的Agents SDK 的设计遵循两个核心原则：
1. **功能足够强大，值得使用，但原语足够少，容易上手**
2. **默认配置即可很好地运行，但你也可以完全自定义行为逻辑**
以下是该 SDK 的主要特性：
- **Agent 循环机制**：内置的智能体循环逻辑，自动处理工具调用、结果返回给 LLM、直到任务完成的全过程
- **Python 优先**：使用原生 Python 语言特性来编排与串联智能体，而无需学习新的抽象概念
- **Handoff（智能体间任务交接）**：强大的功能，可在多个智能体间协调与任务委派
- **Guardrail（输入验证护栏）**：支持与智能体并行运行输入验证逻辑，若验证失败可提前中断流程
- **函数工具化**：可以将任何 Python 函数转为工具，自动生成 Schema，并支持基于 Pydantic 的验证机制
- **追踪系统（Tracing）**：内置追踪功能，可视化、调试、监控你的智能体流程，并结合 OpenAI 的评估、微调与蒸馏工具一同使用

安装agents sdk
```
pip install openai-agents
```
##### OpenAI Agents SDK接入MCP服务器
仅需要在构建`Agent`对象时，像传递通过`@function_tool`装饰器定义的工具一样，将`MCP`服务器作为参数传递给`Agent`对象即可。只不过，不再通过`tools`参数，而是借助`mcp_servers`参数来传递
给Agent对象传递MCP服务器
```Python
    mcp_agent=Agent(
        name="智能代理",
        instructions="可以调用MCP服务器中的工具来完成任务",
        mcp_servers=[mcp_server_1, mcp_server_2],  # server_1和server_2是MCP服务器的实例
        mcp_config={
            "convert_schemas_to_strict": False  # 默认是 False
        }
    )
```
name:代理的名称
instructions：代理的指令
**mcp_servers**：该参数用于接收一个或多个`Model Context Protocol (MCP)`服务器的实例。当`Agent`运行时，它会自动加载`MCP`服务器，并获取`MCP`服务中定义的可用工具列表。
 **mcp_config**：该参数用于配置 `MCP` 服务器的相关设置。

`OpenAI Agents SDK`框架中兼容的是基于[Model Context Protocol](https://github.com/modelcontextprotocol)的开放协议规范，而该协议根据所支持和使用的传输协议不同，目前主要分为如下三种类型的服务器：
- **stdio 服务器**：作为应用程序的子进程运行。我们一般将这种类型的服务器视为“本地”运行。
- **http over sse 服务器**：作为独立的进程运行，并使用`sse`协议进行通信，可以通过`url`来访问。
- **StreamableHTTP 服务器**：可流式传输的`HTTP`传输,正在取代 `SSE`传输。
`OpenAI Agents SDK`框架目前仅支持`stdio`和`http over sse`这两种类型。

与`Stdio`类似，`MCPServerSse`通过接收`MCPServerSseParams`中定义的参数来连接`MCP Server`。
MCPServerSseParams参数汇总

|   |   |   |   |
|---|---|---|---|
|参数|类型|描述|使用场景|
|`url`|`str`|服务器的 URL。|指定要连接的服务器地址。|
|`headers`|`dict` (可选)|发送到服务器的请求头。|用于传递认证信息或其他自定义头部。|
|`timeout`|`float` (可选)|HTTP 请求的超时时间，默认为 5 秒。|控制请求的最大等待时间，避免长时间无响应。|
|`sse_read_timeout`|`float` (可选)|SSE 连接的超时时间，默认为 5 分钟。|控制服务器推送事件的最大等待时间，确保连接的稳定性。|

  在 `SSE (Server-Sent Events)` 模式 下，我们可以把 `MCP` 服务器当成一个已经在网络上跑着的「工具 API」，用 `URL` 建立事件流、再用 `POST /messages` 发指令。当前主流的使用形式可以归纳为两大类：**自托管启动与平台托管直连。**

- **自托管启动（Self-hosted）**
  这种方式主要是应用`[MCP官方的Python / TypeScript MCP SDK](https://github.com/modelcontextprotocol)` 把 `Server` 代码跑起来（FastAPI / Express 集成）, 程序启动后会打印或返回一个本地或内网 `SSE URL`，然后便可以用这个 `URL` 建立事件流接入到`OpenAI Agents SDK` 框架中。如果需要使用`MCP` 官方的`Python` SDK构建`SSE` 模式的`MCP` 服务器。

- **平台托管直连（Platform-hosted）**
  诸如 `[ModelScope的 MCP 广场](https://www.modelscope.cn/mcp)`、[mcp.run](https://docs.mcp.run/integrating/tutorials/mcp-run-sse-openai-agents)等社区维护大量 `MPC` 服务并支持一键启动`MCP`的`SSE` 模式。使用的方法非常简单，只需要在平台首页选好`MCP Server` → 生成一个`API-Key`，一键生成 `SSE URL` ，即可直接接入使用。

  对于国内开发者，通过“一键复制 URL + API-Key”就能拿到 `SSE` 端点的热门网站大致可以分为四类：
1. 专门的 MCP 服务市场/目录（如 MCP Market、AIbase、MCP Server Hub）；
2. 云厂商推出的 官方托管平台（阿里云 “百炼” 与 Higress Marketplace）；
3. 行业 API 已内置 MCP & SSE（百度地图、腾讯位置服务等）；
4. IDE / 客户端集市 把第三方 SSE URL 聚合到本地（Cherry Studio、Cursor）。

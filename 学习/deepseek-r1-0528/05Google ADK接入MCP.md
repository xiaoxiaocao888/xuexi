**谷歌 Agent Development Kit (ADK)** 旨在让开发者像进行软件开发一样来创建和部署自主智能体系统，使构建从简单任务到复杂工作流的多代理应用更为容易。

ADK **天生支持多代理 (multi-agent)** 架构，在设计上鼓励将应用拆分为多个专职的代理，并通过层次化的管理和协作来完成复杂任务。典型模式是采用**层次化的主从代理模型**：可以定义一个主代理（如“调度/主管”代理）负责统筹，根据用户请求的意图将子任务**委派**给不同功能特长的子代理处理。这种设计允许代理间**自动分工与协调**：在处理用户消息时，主代理的底层大模型会参考自己及子代理的能力描述，判断是否将工作**转交**（handoff）给某个子代理。例如，一个 WeatherAgent 主管可根据用户输入动态决定是自己调用天气查询工具回答天气问题，还是将对话移交给 GreetingAgent 来处理问候语。为此，每个代理在定义时都需要提供清晰的角色说明和能力描述，以帮助大模型决策**职责路由**。这种通过 LLM 推断驱动的代理转移机制使 ADK 的多代理系统更具**自适应性**，能够根据对话内容灵活调用最合适的代理处理。除此之外，ADK 还支持**多代理并行和循环**等复杂编排模式：内置了**顺序 (Sequential)**、**并行 (Parallel)**、**循环**
  
**模型支持方面**，ADK 采用**模型无关、插件式**设计。通过内置的 _LiteLLM_ 适配层，ADK 可以无缝对接 Vertex AI 模型园中各种模型，甚至包括 Anthropic、Meta、Mistral、AI21 等多家提供商的大模型。这意味着企业可根据需求选择最适合的模型或使用私有模型，而无需更改 ADK 的使用方式，避免供应商锁定。

**工具扩展方面**，ADK 拥有丰富的**内置工具**和**第三方工具**整合能力。框架自带了诸如网络搜索、代码执行等常用工具模块，并通过**开放 API**支持开发者注册任意 Python 函数作为自定义工具。ADK 特别提供了**模型上下文协议 (MCP)** 工具接口和对 **OpenAPI** 服务的集成，方便调用外部系统。ADK 还能够把**其他代理作为工具**使用：例如可以将一个由 LangChain 或 CrewAI 构建的代理包装为 ADK 工具供另一个代理调用，从而充分利用社区已有成果。通过这种插件化设计，ADK 的功能可**无限扩...

- ADK项目主页：https://github.com/google/adk-python
- ADK项目说明文档：https://google.github.io/adk-docs/

##### ADK默认的大模型调用工具——LiteLlm
- 项目官网：https://github.com/BerriAI/litellm
- - 项目参考文档：https://docs.litellm.ai/docs/
**LiteLLM** 是一个轻量级的 Python 库，它特别适用于开发者在多种云服务提供商（如 OpenAI、DeepSeek）或本地部署模型（如 Ollama）上运行大语言模型的需求。**LiteLLM 核心功能如下：**
- **跨平台模型兼容性**：通过统一接口调用多个大语言模型，无论是云端提供的 API（如 OpenAI）还是本地托管的模型（如 Ollama）。它内置支持常见的大型模型和多模型集成，如 GPT-3.5、GPT-4、DeepSeek 等，也能支持定制模型或自己托管的 LLM。
    
- **简化的 API 调用**：LiteLLM 提供了一个简单直观的接口，用户只需关注模型名称和输入输出内容，LiteLLM 会自动处理与模型的连接、API 调用以及响应解析工作。
    
- **多模型支持**：LiteLLM 支持通过配置文件和代码直接切换不同的模型。它支持从 OpenAI、Anthropic、Meta、DeepSeek 等多个模型提供商调用，还允许开发者轻松集成自定义模型。例如，您可以在一个项目中同时使用 OpenAI 的 GPT 模型和 DeepSeek 的推理模型，而无需额外的集成工作。
    
- **灵活的代理支持**：LiteLLM 可以配置为代理模型的 API，通过代理层进行调用，支持本地化部署和代理服务，例如与 Ollama 等工具结合，运行本地大语言模型。
    
- **自定义 API 路由（Base URL）**：LiteLLM 允许用户设置自定义的 `base_url`，以支持通过代理或内部 API 调用模型。
    
- **流式响应与事件处理**： 支持流式响应，允许开发者在模型生成回复的同时进行处理。能够实时接收和处理模型输出，适用于需要低延迟响应的应用场景，如实时对话或多轮交互。
    
- **工具和任务管理**：在集成外部工具（如搜索引擎、数据库查询等）时，LiteLLM 通过简洁的配置支持多任务执行。开发者可以将外部工具与模型结合，形成复杂的自动化任务流，例如通过模型生成的查询请求，调用外部 API 完成任务并返回结果。
    
- **灵活的配置和扩展性**：LiteLLM 提供了多种配置选项，允许开发者根据具体需求调优模型的行为。用户可以通过传入配置文件来调整模型的推理设置、响应格式、温度参数等，此外，LiteLLM 还支持通过插件扩展其功能，适应更多的使用场景。
    
安装litellm
```
pip install litellm==1.67.2
```

```Python
# 使用 LiteLLM 调用 DeepSeek 模型
from litellm import completion
response = completion(
    model="deepseek/deepseek-chat",  
    messages=[{"role": "user", "content": "你好，好久不见！"}],
    api_base=api_base,
    api_key=api_key
)
print(response)
print(response.choices[0].message.content)
```
使用最简模式构建一个Agent代理
```python
init_agent = Agent( name="chatbot", model=model, instruction="你是一个乐于助人的中文助手。", )
```

ADK内置了非常多的工具，包括Gemini系列模型自带的谷歌搜索、文件检索和代码解释器等功能，同时ADK还拥有内置对话前端，方便开发者更加直观的感受Agent的对话流程，并且实时追踪Agent的events流。而如果Agent中存在外部工具调用，则在内置前端中，还能进一步观察到Agent调用外部工具完整流程，以及各环节调用信息，方便开发者即时debug。如果是使用Gemini模型，这个前端还支持语音和视频实时交互

ADK内置前端开启流程:创建一个ADK_Chat文件夹：(手动创建基本结构)
```CSS
    ADK_Chat/
    ├── test_agent/
    │   ├── agent.py
    │   ├── __init__.py
    │   ├── .env
    ├── __init__.py
    └── ...
```
 **init.py**
写入如下内容：`from . import agent`
**.env**
```Bash
DS_BASE_URL=https://api.deepseek.com
DS_API_KEY=YOUR_API_KRY
```
**agent.py**  使用`Google ADK`框架中的`MCPToolset`方法接入`MCP Server`
```Python
from google.adk.agents import Agent
from google.adk.models.lite_llm import LiteLlm

from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, SseServerParams
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, StdioServerParameters

import os
from dotenv import load_dotenv
load_dotenv(override=True)

# step 1. 设置Filesystem MCP Server 所需要的文件路径
TARGET_FOLDER_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), "file_data")

# 确保目录存在
os.makedirs(TARGET_FOLDER_PATH, exist_ok=True)

DS_API_KEY = os.getenv("DS_API_KEY")
DS_BASE_URL = os.getenv("DS_BASE_URL")

model = LiteLlm(
    model="deepseek/deepseek-reasoner",  
    api_base=DS_BASE_URL,
    api_key=DS_API_KEY
)

# 根据 GitHub issue #893 的解决方案，使用本地 MCP 服务器
root_agent = Agent(
    name="filesystem_assistant",
    model=model, 
    description="用于管理本地文件的智能体",
    instruction="你是一个文件管理助手，可以帮助用户列出文件、读取文件内容等。",
    tools=[
        MCPToolset(
            connection_params=StdioServerParameters(
                command='npx.cmd',  # Windows 环境使用 npx.cmd
                args=[
                    "-y",
                    "@modelcontextprotocol/server-filesystem",
                    os.path.abspath(TARGET_FOLDER_PATH)
                ],
            )
        )
    ],
)
```

adk其实是伴随着安装包一起安装的调用测试命令。在使用adk命令时，我们无需单独设置主函数，只需要按照要求创建一个名为`root_agent`的主agent，即可顺利开启对话。同时当前项目结构中，`.env`文件用于保存一些关键变量信息，而`__init__.py`则负责将当前项目文件`test_agent`包装为一个可执行的Python文件。这也就是为何我们可以直接输入adk run test_agent的原因。

  接下来即可开启前端了，我们在项目根目录下输入：

```Bash
    adk web --port 8002
    adk web --no-reload # windows使用这个命令
```
在浏览器输入：`http://0.0.0.0:8002/`即可开启对话
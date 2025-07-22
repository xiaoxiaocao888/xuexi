```bash
pip install langgraph -i https://pypi.tuna.tsinghua.edu.cn/simple
pip show langgraph
pip install python-dotenv openai
pip install langchain-deepseek
```
这里一样要在.env文件写入DEEPSEEK_API_KEY

在langgraph中，是通过一个`结构化工具函数`来说明外部函数的参数输入（包括参数类型），以确保在实际调用过程中大模型能够准确识别外部函数的参数类型及其实际含义

其中Pydantic 提供的 BaseModel 负责定义外部函数的参数结构，相当于是告诉模型外部函数参数名是 city、类型是字符串（str）、描述为“城市名称”，有了这个结构，模型能更好地理解如何调用该函数。
```python
from langchain_core.tools import tool 
from pydantic import BaseModel, Field 
class WeatherQuery(BaseModel): 
       city: str = Field(description="The location name of the city") @tool(args_schema = WeatherQuery)
def fetch_weather(city: str) -> dict[str, Any] | None:
```
如果是将外部工具接入OpenAI Agents SDK
```python
from agents import function_tool 
@function_tool 
def get_weather(loc):
```
创建LangGraph智能体
```python
tools = [fetch_weather]
from langgraph.prebuilt import create_react_agent
from langchain.chat_models import init_chat_model
model = init_chat_model(model="deepseek-chat", model_provider="deepseek")  
agent = create_react_agent(model=model, tools=tools)
response = agent.invoke({"messages": [{"role": "user", "content": "请问北京今天天气如何？"}]})
print(response["messages"][-1].content)
```
借助LangGraph React Agent，无需任何额外设置，即可完成多工具串联和并联调用。

 LangChain工具集： https://python.langchain.com/docs/integrations/tools/
主要的内置工具有

|   |   |   |
|---|---|---|
|功能类别|工具名称|简要说明|
|🔎 搜索工具|`TavilySearchResults`|快速搜索实时网络信息|
||`SerpAPIWrapper`|基于 SerpAPI 的搜索结果工具|
||`GoogleSearchAPIWrapper`|调用 Google 可编程搜索引擎|
|🧠 计算工具|`PythonREPLTool`|执行 Python 表达式并返回结果|
||`LLMMathTool`|结合 LLM 和数学推理能力|
||`WolframAlphaQueryRun`|基于 Wolfram Alpha 的计算引擎|
|🗂 数据工具|`SQLDatabaseToolkit`|构建 SQL 数据库查询工具集|
||`PandasDataframeTool`|用于在 Agent 中操作表格数据|
|🌐 网络/API|`RequestsGetTool` / `RequestsPostTool`|执行 HTTP 请求|
||`BrowserTool` / `PlaywrightBrowserToolkit`|自动化网页浏览与抓取|
|💾 文件处理|`ReadFileTool`|读取本地文件内容|
||`WriteFileTool`|写入文本到指定文件中|
|📚 检索工具|`FAISSRetriever`|基于向量的文档检索工具|
||`ChromaRetriever`|使用 ChromaDB 的检索器|
||`ContextualCompressionRetriever`|上下文压缩检索器，适合长文档|
|🧠 LLM 工具|`ChatOpenAI` / `OpenAIFunctionsTool`|使用 OpenAI 模型作为工具调用|
||`ChatAnthropic`|Anthropic Claude 模型封装工具|
|🔧 自定义工具|`@tool` 装饰器|任意函数可封装为 Agent 可调用工具|
||`Tool` 类继承|自定义更复杂逻辑的工具实现|
使用内置工具tavily，和08笔记类似。区别是使用langgraph的agent方法
```python
import os
from dotenv import load_dotenv 
load_dotenv(override=True)
from langchain_tavily import TavilySearch
from langgraph.prebuilt import create_react_agent
from langchain.chat_models import init_chat_model
model = init_chat_model(model="deepseek-chat", model_provider="deepseek")  
search = TavilySearch(max_results=5,topic="general")
tools = [search]
agent = create_react_agent(model=model, tools=tools)
response=agent.invoke({"messages": [{"role": "user", "content": "请帮我搜索最近OpenAI CEO在访谈中的核心观点。"}]})
print(response["messages"][-1].content)
```
![[Pasted image 20250710151815.png]]

**在Agent运行的时候设置{"recursion_limit": X}，即可限制智能体自主执行任务时的步数。**
x等于`len(response["messages"])`
```Python
try:
    response = agent.invoke(
        {"messages": [{"role": "user", "content": "请问北京今天天气如何？"}]},
        {"recursion_limit": 4},
    )
except GraphRecursionError:
    print("Agent stopped due to max iterations.")
```
防止陷入死循环。一般是4步，接受用户消息，发起function call message，收到function response message,给用户发送final response。
**智能体记忆管理与多轮对话方法**
保存的不仅仅是历史对话，还可以是图结构运行状态的一次快照
使用：导入库，实例化，在创建智能体时，加入checkpointer=实例化对象。还要有线程。根据线程id区分对话
```Python
from langgraph.checkpoint.memory import InMemorySaver
checkpointer = InMemorySaver()
tools = [fetch_weather]
model = init_chat_model(model="deepseek-chat", model_provider="deepseek")  
agent = create_react_agent(model=model, 
                           tools=tools,
                           checkpointer=checkpointer)
config = {
    "configurable": {
        "thread_id": "1"  
    }
}
response = agent.invoke(
    {"messages": [{"role": "user", "content": "你好，我叫陈明，好久不见！"}]},
    config
)
print(response['messages'])
#此时记忆自动保存在当前Agent和线程中
latest = agent.get_state(config)
print(latest)
#当我们再次进行对话时，直接带入线程ID，即可带入此前对话记忆：
response1 = agent.invoke(
    {"messages": [{"role": "user", "content": "你好，请问你还记得我叫什么名字么？"}]},
    config
)
print(response1['messages'])
latest = agent.get_state(config)
print(latest)
#如果更新线程ID，则会重新开启对话
config2 = {
    "configurable": {
        "thread_id": "2"  
    }
}
response3 = agent.invoke(
    {"messages": [{"role": "user", "content": "你好，你还记得我叫什么名字么？"}]},
    config2
)
print(response3['messages'])
```
![[Pasted image 20250710154756.png]]
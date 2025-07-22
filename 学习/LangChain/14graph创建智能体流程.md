LangSmith是用于跟踪和分析大模型的使用情况，而LangGraph Studio比LangSmith更加方便和高效的可视化调试工具平台
我们可以使用LangGraph Platform把应用、Agent、Workflow部署成Server。完整架构如下
![[Pasted image 20250710170424.png]]
- **LangGraph Studio**：桌面版应用（`目前仅支持Mac`）和本地运行（适用于所有操作系统）；
- **LangServer**：最终构建出来的服务，提供`Assistant API`接口；
- **Python/JS SDK**：通过接口可以直接和 `LangServer` 提供的各个`API`接口连接；
- **Remote Graph**：类似于之前`LangServe`的用法，可以直接用 `Graph` 的接口去调用，这样拿到的`Graph`就是一个 `Runable`对象，就可以去调用它的`invoke`，`batch` 等。
 
 `LangGraph Studio` 是专为 `LangGraph` 图式代理打造的本地/云端 `IDE`，具备可视化节点和状态及实时调试功能。`LangGraph Studio` 在本地可视化运行时会自动把调用过程上传到 `LangSmith`；而在 `LangSmith` 网页端查看任何 `Trace` 时，又能一键`Run in Studio`回放整条执行链，所以它是通过统一 `Trace SDK` 与 `LangSmith` 紧密集成。而LangGraph CLI则是构建这个项目的关键

#### 创建完整LangGraph智能体项目流程
**1.** 新建一个文件夹，我们使用之前的文件夹。新建一个requirements.txt文件，写需要安装的依赖。
依赖管理
```Bash
langgraph
langchain-core
langchain-deepseek
python-dotenv
langsmith
pydantic
matplotlib
seaborn
pandas
IPython
langchain_mcp_adapters
uv
```
**2.注册LangSmith**
借助LangSmith进行追踪（会将智能体运行情况实时上传到LangGraph官网并进行展示）。
https://smith.langchain.com/  在该网站进行注册和登录。
**3.在.env文件里面增加**
环境变量
```env
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=XXX
LANGSMITH_PROJECT=文件名
```
**4.创建graph.py核心文件**
新建一个`16graph03hexin.py`文件，在该文件中编写构建图的具体运行逻辑，如状态、节点、变、图的编译等。注意：这里不要使用`init_chat_model`方法，而是使用`ChatDeepSeek`方法，因为会存在异步阻塞问题。此外，在使用LangGraph CLI创建智能体项目时，会自动设置记忆相关内容，并进行持久化记忆存储，无需手动设置。因此此时智能体代码如下所示：
```python
import httpx
from typing import Any
from langchain_tavily import TavilySearch
from dotenv import load_dotenv
from langchain_deepseek import ChatDeepSeek
from langgraph.graph.message import add_messages
from langchain.chat_models import init_chat_model
from langgraph.prebuilt import create_react_agent
from langchain_core.tools import tool
from pydantic import BaseModel, Field
# 加载环境变量
load_dotenv(override=True)
# 内置搜索工具
search_tool = TavilySearch(max_results=5, topic="general")
TIANQI_API_BASE = "http://gfeljm.tianqiapi.com/api"
class WeatherQuery(BaseModel):
    city: str = Field(description="The location name of the city")
@tool(args_schema = WeatherQuery)
def fetch_weather(city: str) -> dict[str, Any] | None:
    """
    从 Tianqi API 获取天气信息。
    :param city: 城市名称
    :return: 天气数据字典；若出错返回包含 error 信息的字典
    """
    params = {
        "version":"v61",
        "appid":"78875865",
        "appsecret": "vnm0pO1N",
        "city": city,
    }
    with httpx.Client() as client:
        try:
            response = client.get(TIANQI_API_BASE, params=params, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            return {"error": f"HTTP 错误: {e.response.status_code}"}
        except Exception as e:
            return {"error": f"请求失败: {str(e)}"}
tools = [search_tool, fetch_weather]
# 创建模型
model = ChatDeepSeek(model="deepseek-chat")
# 创建图
graph = create_react_agent(model=model, tools=tools)
```
**5.创建langgraph.json文件** 配置文件
在该`json`文件中配置项目信息，遵循规范如下所示：
- 必须包含 `dependencies` 和 `graphs` 字
- `graphs` 字段格式："图名": "文件路径:变量名"
- 配置文件必须放在与Python文件同级或更高级的目录
  注意: 项目文件的名称必须为`langgraph.json`。
```json
{
    "dependencies":["./"],
    "graphs":{
        "chatbot":"./graph.py:graph"
    },
    "env":".env"
}
```
- `dependencies`: ["./"] - 告诉`LangGraph`在当前目录查找依赖项（会自动读取`requirements.txt`）
- `chatbot`: "./graph.py:graph" - 定义图名为`chatbot`，来自`graph.py`文件中的`graph`变量
- `env`: ".env" - 指定环境变量文件位置
**6.安装依赖**
```Bash
pip install -U "langgraph-cli[inmem]" -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install -r requirements.txt
```
进入文件夹里面，执行该命令即可启动项目
```Bash
langgraph dev
```

![[Pasted image 20250711100644.png]]
启动成功后能看到三个连接，其中第一个连接是当前部署完成后的服务端口(后端服务地址)，第二个是LangGraph Studio的可视化页面，第三个端口是端口说明。
我们可以点击第二个链接，可以进行对话
带虚线的叫条件边
![[Pasted image 20250711101539.png]]
**LangGraph Agent后端接入Agent Chat UI完整流程**
- 项目主页：https://github.com/langchain-ai/agent-chat-ui
1.克隆项目
```Bash
git clone https://github.com/langchain-ai/agent-chat-ui.git
cd agent-chat-ui
```
不成功，可以关闭防火墙试试
2.安装npm
node.js官网：https://nodejs.org
```Bash
npm install -g pnpm
pnpm -v
```
3.安装前端项目依赖
```Bash
pnpm install
```
4.开启Chat Agent UI  前端
前端和后端都启动着，就可以使用前端端口直接访问后端服务
```Bash
pnpm dev
```
![[Pasted image 20250711151309.png]]
复制链接的时候会出现一个页面，要填写前端URL和agent名字(要和langgraph.json里面的graphs的一样)还有langsmith的密钥
![[Pasted image 20250711151247.png]]
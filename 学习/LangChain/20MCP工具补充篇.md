MCP的SDK（开发工具），指的是官方提供的用于开发MCP工具的第三方库，截至目前，MCP SDK已支持Python、TypeScript、Java、Kotlin和C#等编程语言进行客户端和服务器创建。
MCP项目官网： https://github.com/modelcontextprotocol
主流的MCP集成平台
- MCP官方服务器合集： https://github.com/modelcontextprotocol/servers
- MCP Github热门导航： https://github.com/punkpeye/awesome-mcp-servers
- Smithery：https://smithery.ai/
- MCP导航：https://mcp.so/
- 阿里云百炼：https://bailian.console.aliyun.com/?tab=mcp
- 魔搭社区MCP广场：https://www.modelscope.cn/mcp
- mcp.run：https://www.mcp.run/
- MCP服务市场 https://mcpmarket.cn/
- GitHub 上的热门 MCP Server 并直接给出 SSE 地址与使用教程 https://mcp.aibase.cn/
- 国际化，标签筛选 + “Copy URL” 操作流畅，数千条 SSE Server https://mcpserverhub.com/


本地运行，也被成为离线运行，MCP服务器和MCP客户端在同一台电脑上进行运行，其二则是在线运行，或者异地部署运行，指的是MCP服务器在远程服务器或者云端运行，然后借助HTTP网络通信来进行响应。
如果是调用在线MCP服务，开发者只能了解其功能，而如果是离线的MCP服务，则对应的MCP工具是完全开源的，在每次调用之前，开发者都需要将其源码先下载到本地，然后再运行。

在inspecotr选择SSE模式，选择默认运行地址：`http://localhost:8000/sse`

使用LangGraph接入MCP工具，核心需要使用langchain_mcp_adapters库，该库可以将MCP工具信息进行解析，并让LangGraph顺利识别。识别后即可像任意其他工具一样接入LangGraph中并搭建智能体。其中是把它转换为LangChain工具来使用
和前面使用uv来搭建环境，使用MCP工具是一样的
创建servers_config.json文件，写入外部工具信息
```JSON
{
  "mcpServers": {
    "weather": {
      "command": "python",
      "args": ["weather_server.py"],
      "transport": "stdio"
    },
    "write": {
      "command": "python",
      "args": ["write_server.py"],
      "transport": "stdio"
    }
  }
}
```
在.env文件写入一些地址、模型、密钥信息
```Plaintext
BASE_URL=https://api.deepseek.com
MODEL=deepseek-chat
LLM_API_KEY=DeepSeek API KEY
OPENWEATHER_API_KEY=OpenWeather API KEY
```
创建agent_prompts.txt文件来写提示词模板，方便更改提示词。
创建主函数client.py.读取提示词\ 基本步骤
和langchain接入MCP是差不多的，只是创建智能体函数不一样
```python
with open("agent_prompts.txt", "r", encoding="utf-8") as f: 
     prompt = f.read()
from langchain_mcp_adapters.client import MultiServerMCPClient 
from langgraph.checkpoint.memory import InMemorySaver
# 设置记忆存储 
checkpointer = InMemorySaver()
cfg = Configuration()
@staticmethod 
def load_servers(file_path: str = "servers_config.json") -> Dict[str, Any]: 
   with open(file_path, "r", encoding="utf-8") as f: 
        return json.load(f).get("mcpServers", {})
servers_cfg = Configuration.load_servers() 
# 1️ 连接多台 MCP 服务器 
mcp_client = MultiServerMCPClient(servers_cfg) 
tools = await mcp_client.get_tools() # LangChain Tool 对象列表
# 2️ 初始化大模型（DeepSeek / OpenAI / 任意兼容 OpenAI 协议的模型） 
DeepSeek_API_KEY = os.getenv("DEEPSEEK_API_KEY") 
model = ChatDeepSeek(model="deepseek-chat") 
# 3 构造 LangGraph Agent 
agent = create_react_agent(model=model, tools=tools, prompt=prompt, checkpointer=checkpointer)
```

我们创建新的文件connect.py(视频是说修改client.py，我是创建新的)
引入stdio_client是为了让client.py和server.py同时运行，实时通信
运行
```
uv run connect.py server.py
```
![[Pasted image 20250511173750.png]]
导入依赖
```python
import asyncio
import os
import json
from typing import Optional
from contextlib import AsyncExitStack

from openai import OpenAI  
from dotenv import load_dotenv

from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

# 加载 .env 文件，确保 API Key 受到保护
load_dotenv()
class MCPClient:
```
##### 创建类MCPClient的函数
###### 初始化MCP客户端
主要作用是为客户端建立与 OpenAI 服务和 MCP 服务器通信的基础设施。
```python
def __init__(self):
    """初始化 MCP 客户端"""
    # 初始化异步资源管理栈（用于管理连接池、会话等）
    self.exit_stack = AsyncExitStack()  
    
    # 从环境变量加载 OpenAI 配置
    self.openai_api_key = os.getenv("OPENAI_API_KEY")  # API 密钥
    self.base_url = os.getenv("BASE_URL")             # OpenAI 服务地址（可能为代理地址）
    self.model = os.getenv("MODEL")                   # 使用的模型（如 gpt-3.5-turbo）
    
    # 强制校验 OpenAI API Key 是否存在
    if not self.openai_api_key:
        raise ValueError("❌ 未找到 OpenAI API Key，请在 .env 文件中设置 OPENAI_API_KEY")
    
    # 创建 OpenAI 官方客户端实例
    self.client = OpenAI(
        api_key=self.openai_api_key, 
        base_url=self.base_url  # 兼容官方API或第三方代理
    )
    
    # 初始化 MCP 会话对象（稍后通过 connect_to_server 方法填充）
    self.session: Optional[ClientSession] = None  
```
###### 连接到 MCP 服务器并获取可用工具列表
`server_script_path` 参数是 服务器脚本路径
创建 `StdioServerParameters` 对象配置服务器启动参数
    - `command`：执行命令
    - `args`：命令参数（脚本路径）
    - `env`：环境变量（此处为 None，使用默认环境）
**启动服务器并建立通信通道**
- `stdio_client(server_params)`：通过标准输入输出（STDIO）启动服务器进程并建立客户端连接
- `exit_stack.enter_async_context()`：将资源加入异步资源管理栈，确保自动清理
- `ClientSession`：创建客户端会话对象，用于与服务器通信
     - `stdio_transport` 是 `stdio_client(server_params)` 返回的对象，它实际上是一个**二元元组（Tuple）**，包含两个元素：
        - 第一个元素：标准输入输出流对象（通常用于接收数据）
        - 第二个元素：写入函数（通常用于发送数据），所以是对应的赋值。
- `session.initialize()`：初始化客户端与服务器的会话
- `session.list_tools()`：向MCP服务器请求可用工具列表，大概是
```python
 tools=[
        Tool(name="翻译", description="将文本翻译成指定语言"),
        Tool(name="计算", description="执行数学计算"),
        # ...
    ]
```

```python
async def connect_to_server(self, server_script_path: str):
        """连接到 MCP 服务器并列出可用工具"""
        is_python = server_script_path.endswith('.py')
        is_js = server_script_path.endswith('.js')
        if not (is_python or is_js):
            raise ValueError("服务器脚本必须是 .py 或 .js 文件")
        command = "python" if is_python else "node"
        server_params = StdioServerParameters(
            command=command,
            args=[server_script_path],
            env=None
        )
        # 启动 MCP 服务器并建立通信
        stdio_transport = await self.exit_stack.enter_async_context(stdio_client(server_params))
        self.stdio, self.write = stdio_transport
        self.session = await self.exit_stack.enter_async_context(ClientSession(self.stdio, self.write))
        await self.session.initialize()
        # 列出 MCP 服务器上的工具
        response = await self.session.list_tools()
        tools = response.tools
        print("\n已连接到服务器，支持以下工具:", [tool.name for tool in tools])
```
###### 处理用户关于城市信息的查询，智能调用外部工具获取数据，并生成最终回答
.arguments是一个字典，包含了模型生成的工具参数，如
```python
tool_args = {"city": "北京", "date": "2023-05-11"}
```
content.message.model_dump()将 OpenAI 模型返回的 `content.message` 对象（通常是一个 Pydantic 模型实例）转换为字典
```python
async def process_query(self, city: str) -> str:
        """使用大模型处理查询并调用可用的 MCP 工具 (Function Calling)"""
        messages = [{"role": "user", "content": f"关于{city}的信息，请始终用中文城市名称"}]
        response=await self.session.list_tools()
        available_tools = [{
            "type": "function",
            "function": {
                "name": tool.name,
                "description": tool.description,
                "input_schema": tool.inputSchema
            }
        } for tool in response.tools]
        # print(available_tools)
        # 第一次请求，判断是否需要工具
        response = self.client.chat.completions.create(
            model=self.model,            
            messages=messages,
            tools=available_tools    
        )
        # 处理返回的内容
        content = response.choices[0]
        if content.finish_reason == "tool_calls":
            # 如果是需要使用工具，就解析工具
            tool_call = content.message.tool_calls[0]
            tool_name = tool_call.function.name
            tool_args = json.loads(tool_call.function.arguments)
            # 执行工具
            result = await self.session.call_tool(tool_name, tool_args)
            print(f"\n\n[Calling tool {tool_name} with args {tool_args}]\n\n")
            # 将模型返回的调用哪个工具数据和工具执行完成后的数据都存入messages中
            messages.append(content.message.model_dump())
            messages.append({
                "role": "tool",
                "content": result.content[0].text,
                "tool_call_id": tool_call.id,
            })
            # 第二次请求，将上面的结果再返回给大模型用于生产最终的结果
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
            )
            return response.choices[0].message.content
        return content.message.content
```

###### 交互式聊天循环
```python
async def chat_loop(self):
        """运行交互式聊天循环"""
        print("\n🤖 MCP 客户端已启动！输入 'quit' 退出")
        while True:
            try:
                city = input("\n你: ").strip()
                if city.lower() == 'quit':
                    break
                  # 处理查询，显示结果
                response = await self.process_query(city)  # 发送用户输入到 OpenAI API
                print(f"\n🤖 OpenAI: {response}")
            except Exception as e:
                print(f"\n⚠️ 发生错误: {str(e)}")
```
###### 清理资源，MCPClient类创建就到这里了
```python
async def cleanup(self):
        """清理资源"""
        await self.exit_stack.aclose()
```
主体
```python
async def main():
    if len(sys.argv) < 2:
        print("Usage: python client.py <path_to_server_script>")
        sys.exit(1)
    client = MCPClient()
    try:
        await client.connect_to_server(sys.argv[1])
        await client.chat_loop()
    finally:
        await client.cleanup()
if __name__ == "__main__":
    import sys
    asyncio.run(main())
```
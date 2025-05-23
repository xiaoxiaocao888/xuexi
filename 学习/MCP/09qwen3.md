qwen3不仅拥有非常强悍的推理和对话性能，还原生支持MCP功能。
对于Qwen3全系列模型，不仅支持Function calling，还支持工具的并联、串联调用甚至是自动debug
MCP能力指的是外部工具识别和调用能力，即Function calling能力。
我们可以通过观察模型的内置提示词模版，看出模型是如何识别外部工具的。内置提示词模版可以在模型的tokenizer_config.json中查看

去.env文件里面写入
```
BASE_URL=http://localhost:11434/v1/
MODEL=qwen3:0.6b
LLM_API_KEY=ollama
```

修改main.py文件，把文件里面的所有内容都替换了。（九天的Qwen3接入MCP技术实战（上））
main.py文件里面详细介绍了qwen3模型如何构建MCP的客户端，如何接入外部工具，以及响应。
main.py文件具备的功能：
1. 根据`.env`配置，自由切换底层模型，如可接入ollama、vLLM驱动的模型，也可以接入OpenAI、DeepSeek等在线模型；
2. 根据`servers_config.json`读取多个MCP服务，可以读取本地MCP工具，也可以下载在线MCP工具并进行使用；
3. 能够进行基于命令行的实现多轮对话；
4. 能够同时连接并调用多个MCP server上的多个工具，并能实现多工具的并联和串联调用；

###### 配置MCP服务器
创建一个servers_config.json脚本，写入我们希望调用的MCP工具，如Filesystem MCP，借助Filesystem，可以高效便捷操作本地文件夹。Filesystem也是一个js项目，源代码托管在npm平台。所以命令使用npx
写入内容
-y表示不用用户确认来 自动执行。@xx指的是我当前MCP库的名字
最后一行是要操作的文件夹路径
```JSON
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "D:/mcp-client/"
      ]
    }
  }
}
```

此时执行 就可以看到filesystem  MCP工具的外部函数
```
uv run main.py
```
![[Pasted image 20250511195200.png]]
![[Pasted image 20250511195844.png]]

创建write_server.py文件：写入本地文档的MCP服务器
```Python
import json
import httpx
from typing import Any
from mcp.server.fastmcp import FastMCP

# 初始化 MCP 服务器
mcp = FastMCP("WriteServer")
USER_AGENT = "write-app/1.0"

@mcp.tool()
async def write_file(content: str) -> str:
    """
    将指定内容写入本地文件。
    :param content: 必要参数，字符串类型，用于表示需要写入文档的具体内容。
    :return：是否成功写入
    """
    return "已成功写入本地文件。"

if __name__ == "__main__":
    # 以标准 I/O 方式运行 MCP 服务器
    mcp.run(transport='stdio')
```

##### Qwen3 MCP Client接入自定义的函数
在servers_config.json配置文件里面添加
```json
"weather": { "command": "python", "args": ["weather_server.py"] },
"write": { "command": "python", "args": ["write_server.py"] }
```
重新执行 ,可以看到加载了新工具进来
```
uv run main.py
```
![[Pasted image 20250511201943.png]]
可以实现并联操作
![[Pasted image 20250511204641.png]]
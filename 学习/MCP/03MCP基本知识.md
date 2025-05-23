**核心作用**：统一调用工具流程；大幅提高开发效率
**MCP是一套标准协议，任何MCP工具(MCP Server)都可以无缝接入到任何MCP智能体中，即我开发的工具大家都可以用，你开发的工具我也可以用**

MCP GitHub主页：
```
https://github.com/modelcontextprotocol
```

MCP 全称是Model Context Protocol,模拟上下文协议。
MCP把大模型运行环境称为MCP Client，即MCP客户端；把外部函数运行环境称为MCP Server，即MCP服务器。就是说外部函数的定义和相关函数代码都是写在server.py文件

**Client**:用户交互入口，将用户输入（自然语言）发送给OpenAI，从Server处获取动态构造工具列表(每次处理用户查询时都会从 Server 获取最新工具列表)，将Server工具的描述发送给OpenAI，解析OpenAI的tool_calls响应，通过stdio启动Server进程，使用MCP协议调用远程工具

**OpenAI**：根据用户输入判断是否需要调用工具，结合client发送过来的Server工具描述，生成符合工具调用规范的JSON请求(tool_calls,基于工具描述的 `input_schema` 生成合规参数)，    最终将工具返回的结构化数据转换为自然语言

**Server**：提供具体工具（如天气查询），通过`@mcp.tool()`装饰器暴露出工具列表，执行真实API调用(如OpenWeather)，返回标准化格式的结果(必须包含工具执行状态码（成功/失败）和清洗后的数据)


`response.choices[0].finish_reason:
`.choices[0]`表示模型生成的多个候选回复中的第一个(通常也是唯一一个)使用索引，是因为API始终返回的是列表结构
`.finish_reason`是解释模型为何停止生成文本，
值                含义                                                    触发场景示例 
"stop"         模型正常结束生成                        回复完整生成，遇到停止标记
"length"      达到 token 上限                         `max_tokens` 参数限制
"tool_calls"  模型决定需要调用工具               用户查询需要外部工具（如天气查询、计算器等）
"content_filter" 内容被安全系统过滤          生成内容违反 OpenAI 使用策略


当调用 `client.chat.completions.create()` 时：
1. OpenAI 服务器生成候选回复（数量由 `n` 参数决定）
2. 所有候选回复被封装到 `choices` 列表中
3. 即使 `n=1`，也返回单元素列表以保持数据结构一致性

##### 初始化 MCP 服务器
```python
mcp = FastMCP("WeatherServer")
```
后面的工具注册依赖此实例
```python
@mcp.tool()
```
启动服务器
```
mcp.run(transport='stdio')
```

**自定义User-agent**：
是为了让服务端知道是哪个应用在调用，- 服务端日志可区分不同版本客户端的请求，便于排查兼容性问题
格式为<应用名>/<版本号> (<附加信息>)
```
USER_AGENT = "weather-app/1.0 (+https://example.com/docs)"
```

`response.raise_for_status()`:
自动检查HTTP响应状态码，如果状态码表示错误（4XX客户端错误，5XX服务端错误）会抛出异常，若是200不执行任何操作。

`async def fetch_weather(city: str) -> dict[str, Any] | None:`
异步函数定义接收参数为str类型的city，返回类型为字典类型，键为字符串，值为任意类型，| None表示可能返回none（如请求失败时）
是需要导入
```
from typing import Any
```

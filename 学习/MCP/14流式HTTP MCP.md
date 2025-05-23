**HTTP流式传输**：更加先进的客户端和服务器异地的通信方式。相比SSE传输方式，HTTP流式传输并发更高、通信更加稳定，同时也更容易集成和部署，更能适用于企业级调用

**SDK**：是一套工具、库、文档和示例代码的集合，专门为开发者设计，用于简化特定平台、系统或服务的应用开发流程。核心组成有API(用于程序接口)、开发工具、文档和示例、依赖库与框架。

##### 基于HTTP流式传输的MCP服务器开发流程
- 创建项目文件(参照04uv文档) 采用src_layer的风格进行项目文件编排，要删除main.py
我们直接使用上面创建的文档D:\mcp-client目录下新建目录\src\mcp_weather_http。在该目录下新建三个文件__init__.py,__main__.py,server.py
`__init__.py`可以把当前文件夹包装成一个脚本(包初始化与接口暴露)，将server.py中的main函数提升为包级接口；外部调用时只需from mcp_weather_http import main，无需知道内部模块结构；未来修改核心逻辑文件位置时，只需调整此处导入路径
```python
from .server import main #从server.py文件中导入main函数
```
`__main__.py`主程序入口(命令行模块化入口)
```python
from mcp_weather_http import main
main()
```
`server.py`核心脚本
导入的功能组件：
httpx:用来联网查天气                              mcp:提供了让大模型调用工具的能力
starlette:帮你搭建HTTP网络服务             click：帮你写命令行参数，如--api-key=xxx
查询天气的函数：
fetch_weather,根据城市名去OpenWeather网站查询天气，使用httpx.AsyncClient工具联网发请求
运行服务器  替换成自己OpenWeather的APIKEY
密钥源优先级：命令行参数>环境变量>代码默认值
如果我们的appid,appsecret要动态传入，那么在fetch_weather函数那里加上appid:str,appsecret:str
然后在下方的@click增加。 还要在main()函数参数那里增加appid:str,appsecret:str。那么在执行命令时就要加上--appid xxx    --appsecret xxx
```Bash
@click.option( "--appid", envvar="TIANQI_APPID", # 通过环境变量或命令行传入
required=True, help="天行数据API的 appid（或设置 TIANQI_APPID 环境变量）", )

python weather_mcp_streamable_http_server.py --port 3000 --api-key xxxx
```
实时往客户端推送消息。就是“流式日志通知”
````Python
await ctx.session.send_log_message(...)
````
流式会话管理器配置session__manager=xxx。这段代码创建了MCP的“HTTP会话处理中心”，负责处理所有/mcp路由的请求；stateless=True表示不保存历史对话，每次都是新请求；json_response=False表示用流式SSE(可以改为一次性JSON响应)
构建Starlette Web应用starlette_app=xxx 这是将MCP服务挂载到你的网站/mcp路径上；用户访问这个路径时，就会进入刚才创建的MCP会话管理器
```Python
import contextlib
import logging
import os
from collections.abc import AsyncIterator

import anyio
import click
import httpx  # 异步HTTP客户端
import mcp.types as types 
from mcp.server.lowlevel import Server
from mcp.server.streamable_http_manager import StreamableHTTPSessionManager
from starlette.applications import Starlette # ASGI框架
from starlette.routing import Mount
from starlette.types import Receive, Scope, Send

# ---------------------------------------------------------------------------
# Weather helpers
# ---------------------------------------------------------------------------
OPENWEATHER_URL = "http://gfeljm.tianqiapi.com/api"
DEFAULT_UNITS = "metric"  # use Celsius by default
DEFAULT_LANG = "zh_cn"  # Chinese descriptions


async def fetch_weather(city: str, api_key: str) -> dict[str, str]:
    """Call OpenWeather API and return a simplified weather dict.

    Raises:
        httpx.HTTPStatusError: if the response has a non-2xx status.
    """
    params = {
        "version":"v61",
        "appid":"78875865",
        "appsecret": "vnm0pO1N",
        "city": city,
    }
    async with httpx.AsyncClient(timeout=10) as client:
        r = await client.get(OPENWEATHER_URL, params=params)
        r.raise_for_status()
        data = r.json()
    # Extract a concise summary
    tianqi=data.get("wea")
    maxtem=data.get('tem1')
    mintem=data.get('tem2')
    humidity = data.get("humidity")
    win_speed=data.get("win_speed")
    return {
        "city": city,
        "weather": f"{tianqi}",
        "maxtemp": f"{maxtem}°C",
        "mintemp": f"{mintem}°C",
        "humidity": f"{humidity}",
        "win_speed":f"{win_speed}",
    }

@click.command()
@click.option("--port", default=3000, help="Port to listen on for HTTP")
@click.option(
    "--api-key",
    envvar="OPENWEATHER_API_KEY",
    required=True,
    help="OpenWeather API key (or set OPENWEATHER_API_KEY env var)",
)
@click.option(
    "--log-level",
    default="INFO",
    help="Logging level (DEBUG, INFO, WARNING, ERROR, CRITICAL)",
)
@click.option(
    "--json-response",
    is_flag=True,
    default=False,
    help="Enable JSON responses instead of SSE streams",
) # 切换响应格式
def main(port: int, api_key: str, log_level: str, json_response: bool) -> int:
    """Run an MCP weather server using Streamable HTTP transport."""

    # ---------------------- Configure logging ----------------------
    logging.basicConfig(
        level=getattr(logging, log_level.upper()),
        format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    )
    logger = logging.getLogger("weather-server")

    # ---------------------- Create MCP Server ----------------------
    app = Server("mcp-streamable-http-weather")

    # ---------------------- Tool implementation -------------------
    @app.call_tool()
    async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
        """Handle the 'get-weather' tool call."""
        ctx = app.request_context
        city = arguments.get("location")
        if not city:
            raise ValueError("'location' is required in arguments")

        # Send an initial log message so the client sees streaming early.
        await ctx.session.send_log_message(
            level="info",
            data=f"Fetching weather for {city}…",
            logger="weather",
            related_request_id=ctx.request_id,
        )

        try:
            weather = await fetch_weather(city, api_key)
        except Exception as err:
            # Stream the error to the client and re-raise so MCP returns error.
            await ctx.session.send_log_message(
                level="error",
                data=str(err),
                logger="weather",
                related_request_id=ctx.request_id,
            )
            raise

        # Stream a success notification (optional)
        await ctx.session.send_log_message(
            level="info",
            data="Weather data fetched successfully!",
            logger="weather",
            related_request_id=ctx.request_id,
        )

        # Compose human-readable summary for the final return value.
        summary = (
            f"{weather['city']}：{weather['weather']}，最低温度 {weather['mintemp']}，"
            f"最高温度 {weather['maxtemp']}，湿度 {weather['humidity']},风速{weather['win_speed']}。"
        )

        return [
            types.TextContent(type="text", text=summary),
        ]

    # ---------------------- Tool registry -------------------------
    @app.list_tools()
    async def list_tools() -> list[types.Tool]:
        """Expose available tools to the LLM."""
        return [
            types.Tool(
                name="get-weather",
                description="查询指定城市的实时天气（OpenWeather 数据）",
                inputSchema={
                    "type": "object",
                    "required": ["location"],
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "城市的中文名称",
                        }
                    },
                },
            )
        ]

    # ---------------------- Session manager -----------------------
    session_manager = StreamableHTTPSessionManager(
        app=app,
        event_store=None,  # 无状态；不保存历史事件
        json_response=json_response,
        stateless=True,
    )

    async def handle_streamable_http(scope: Scope, receive: Receive, send: Send) -> None:  # noqa: D401,E501
        await session_manager.handle_request(scope, receive, send)

    # ---------------------- Lifespan Management --------------------
    @contextlib.asynccontextmanager
    async def lifespan(app: Starlette) -> AsyncIterator[None]:
        async with session_manager.run():
            logger.info("Weather MCP server started! 🚀")
            try:
                yield
            finally:
                logger.info("Weather MCP server shutting down…")

    # ---------------------- ASGI app + Uvicorn ---------------------
    starlette_app = Starlette(
        debug=False,
        routes=[Mount("/mcp", app=handle_streamable_http)],
        lifespan=lifespan,
    )

    import uvicorn

    uvicorn.run(starlette_app, host="0.0.0.0", port=port)

    return 0


if __name__ == "__main__":
    main()
```
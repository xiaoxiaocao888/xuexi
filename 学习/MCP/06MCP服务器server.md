###### 根据MCP协议定义，Server可以提供三种类型的标准能力
- Resources:资源，类似于文件数据读取，可以是文件资源或是API响应返回的内容
- Tools：工具，第三方服务、功能函数，通过此可控制LLM可调用哪些函数
- Prompts：提示词，为用户预先定义好的完成特定任务的模版

根据MCP的规范，当前支持两种传输方式：标准输入输出(stdio)和基于HTTP的服务器推送事件(SSE)
通信方式：
- HTTP：采用请求-响应模式，客户端发送请求，服务器返回响应，每次请求都是独立的。
- SSE：允许服务器通过单个持久的HTTP连接，持续向客户端推送数据
![[Pasted image 20250507214922.png]]

服务器依赖安装。因为我们需要使用http请求来查询天气
```
uv add mcp httpx
```
server.py
```python
import json
import httpx
from typing import Any
from mcp.server.fastmcp import FastMCP
import os
# 初始化 MCP 服务器
mcp = FastMCP("WeatherServer")
# OpenWeather API 配置
TIANQI_API_BASE = "http://gfeljm.tianqiapi.com/api"
#API_KEY =os.getenv("OPENWEATHER_API_KEY")  # 请替换为你自己的 OpenWeather API Key
#USER_AGENT = "weather-app/1.0"
async def fetch_weather(city: str) -> dict[str, Any] | None:
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
    #headers = {"User-Agent": USER_AGENT}
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(TIANQI_API_BASE, params=params, timeout=30.0)
            response.raise_for_status()
            #response.encoding='gbk'
            return response.json()  # 返回字典类型
        except httpx.HTTPStatusError as e:
            return {"error": f"HTTP 错误: {e.response.status_code}"}
        except Exception as e:
            return {"error": f"请求失败: {str(e)}"}

def format_weather(data: dict[str, Any] | str) -> str:
    """
    将天气数据格式化为易读文本。
    :param data: 天气数据（可以是字典或 JSON 字符串）
    :return: 格式化后的天气信息字符串
    """
    # 如果传入的是字符串，则先转换为字典
    if isinstance(data, str):
        try:
            data = json.loads(data)
        except Exception as e:
            return f"无法解析天气数据: {e}"
    # 如果数据中包含错误信息，直接返回错误提示
    if "error" in data:
        return f"⚠️ {data['error']}"
    print(data)
    # # 提取数据时做容错处理
    # city = data.get("name", "未知")
    # country = data.get("sys", {}).get("country", "未知")
    # temp = data.get("main", {}).get("temp", "N/A")
    # humidity = data.get("main", {}).get("humidity", "N/A")
    # wind_speed = data.get("wind", {}).get("speed", "N/A")
    # # weather 可能为空列表，因此用 [0] 前先提供默认字典
    # weather_list = data.get("weather", [{}])
    # description = weather_list[0].get("description", "未知")
    date=data.get("date")
    week=data.get("week")
    city=data.get("city")
    tianqi=data.get("wea")
    maxtem=data.get('tem1')
    mintem=data.get('tem2')
    humidity = data.get("humidity")
    win_speed=data.get("win_speed")
    return (
        f"日期：{date}\n"
        f"星期：{week}\n"
        f"🌍 {city}\n"
        f"🌡 最高温度: {maxtem}°C  最低温度：{mintem}°C\n"
        f"💧 湿度: {humidity}%\n"
        f"🌬 风速: {win_speed} m/s\n"
        f"🌤 天气: {tianqi}\n"
    )
    # return (
    #     f"🌍 {city}, {country}\n"
    #     f"🌡 温度: {temp}°C\n"
    #     f"💧 湿度: {humidity}%\n"
    #     f"🌬 风速: {wind_speed} m/s\n"
    #     f"🌤 天气: {description}\n"
    # )
@mcp.tool()
async def query_weather(city: str) -> str:
    """
    输入指定城市的中文名称，不需要带市，返回今日天气查询结果。
    :param city: 城市名称
    :return: 格式化后的天气信息
    """
    data = await fetch_weather(city)
    return format_weather(data)
    #return data
if __name__ == "__main__":
    # 以标准 I/O 方式运行 MCP 服务器
    mcp.run(transport='stdio')
```
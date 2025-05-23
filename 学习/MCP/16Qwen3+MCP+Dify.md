登录魔搭社区
```
https://modelscope.cn/mcp
```
点击MCP广场--选择高德地图、力扣、今天吃什么、Tavily智搜
去到对应的平台获取API，然后粘贴上去，就可以出现这样的SSE URL连接的地址
![[Pasted image 20250523091635.png]]
接着去MCP实验厂，测试一下这几个MCP工具是否可用
点击配置--点击添加MCP Server--点击JSON，输入前面的url--点击确定
```
{
    "mcpServers": {
        "amap-maps": {
            "type": "sse",
            "url": "https://mcp.api-inference.modelscope.cn/sse/3d3c7a54be7348"
        },
        "tavily-mcp": {
            "type": "sse",
            "url": "https://mcp.api-inference.modelscope.cn/sse/4c60860a92cb45"
        },
        "howtocook-mcp": {
            "type": "sse",
            "url": "https://mcp.api-inference.modelscope.cn/sse/c58786eaa32d48"
        },
        "leetcode-mcp-server": {
            "type": "sse",
            "url": "https://mcp.api-inference.modelscope.cn/sse/df1fd01070444d"
        }
    }
}
```
点击配置旁边的实验厂--滑到下面有个对话窗口，可以点击小扳手图案，开启或关闭默认的工具
输入一些对应工具的问题，如适合早八人的早餐食谱，从长沙火车站到梅溪湖步步高的公交路线。可以看到对应的输出，测试成功。
去Dify上绘制工作流
点击插件--点击安装插件--点击Marketplace--在搜索处输入AGENT--安装AGENT策略(支持MCP工具)
![[Pasted image 20250523095646.png]]
在地图Agent那里点击Agent策略--选择支持MCP工具的Agent--点击React
模型配置为qwen3，工具列表选择和时间相关的获取时间戳
MCP配置那里输入前面配置的高德地图的SSE URL 
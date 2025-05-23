首先在`github`上找到公开的高德MCP服务器：https://github.com/sugarforever/amap-mcp-server
去https://lbs.amap.com/该网站登录，并创建新应用来添加APIKEY.
其次找能抓取网页数据的MCP，这里可以选择Firecrawl MCP Server ：https://github.com/mendableai/firecrawl-mcp-server

Firecrawl MCP Server实现与 [Firecrawl](https://github.com/mendableai/firecrawl) 集成以实现网页抓取功能。支持的功能如下：
- 抓取、爬取、搜索、提取、深入研究和批量抓取支持
- 使用 JS 渲染进行网页抓取
- URL 发现和抓取
- 带有内容提取的网页搜索
需要注册Firecrawl的API： https://www.firecrawl.dev/signin/password_signin
登录就可以找得到API
运行结果。地址复制粘贴到浏览器就可以看得到一个前端页面，可以在页面和模型聊天
![[Pasted image 20250513123738.png]]
```Python
    def init_agent_service():
        # 使用 Ollama 运行的模型
        llm_cfg = {
            'model': 'qwen3:0.6b',
            'model_server': 'http://localhost:11434/v1',
            'api_key': 'EMPTY',
        }

        tools = [{
            "mcpServers": {
                'time': {
                    'command': 'uvx',
                    'args': ['mcp-server-time', '--local-timezone=Asia/Shanghai']
                },
                # "fetch": {
                #     "command": "uvx",
                #     "args": ["mcp-server-fetch"]
                # },
                
                "sqlite" : {
                    "command": "uvx",
                    "args": [
                        "mcp-server-sqlite",
                        "--db-path",
                        "test.db"
                    ]
                },

                "firecrawl-mcp": {
                    "command": "npx",
                    "args": ["-y", "firecrawl-mcp"],
                    "env": {
                        "FIRECRAWL_API_KEY": "fc-08f4598cdc6247b291e26ef38f1f2a4c"
                    },
                },

                "amap-mcp-server": {
                    "command": "uvx",
                    "args": [
                        "amap-mcp-server"
                    ],
                    "env": {
                        "AMAP_MAPS_API_KEY": "7ff110a641580368fa8e24ffd709c3b3"
                    }
                }
            },
            
            
        },
        # pip install "qwen-agent[code_interpreter]"
        'code_interpreter',
        ]

        system = """
        你是一个规划师和数据分析师 \
            你可以调用高德地图规划旅行路线，同时可以提取网页信息进行数据分析
        """

        bot = Assistant(
            llm=llm_cfg,
            name='智能助理',
            description='具备查询高德地图、提取网页信息、数据分析的能力',
            system_message=system,
            function_list=tools,
        )

        return bot

    def run_query(query=None):
        # 定义数据库助手
        bot = init_agent_service()

        from qwen_agent.gui import WebUI

        chatbot_config = {
            'prompt.suggestions': [
                '请你帮我写一个关于人工智能的论文',
                "https://github.com/orgs/QwenLM/repositories 提取这一页的Markdown 文档，然后绘制一个柱状图展示每个项目的收藏量",
                '帮我查询从故宫去颐和园的路线',
            ]
        }
        WebUI(
            bot,
            chatbot_config=chatbot_config,
        ).run()
```
  

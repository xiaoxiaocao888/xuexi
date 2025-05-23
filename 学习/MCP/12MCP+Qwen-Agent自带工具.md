大模型不擅长数学运算、数据可视化等精确计算任务，因此我们一般会使用代码解释器，使智能体编写和运行 Python 程序，从而逐步解决复杂的数据问题。如果智能体编写的代码未能成功运行，它还会不断调整代码，尝试不同的方法，直到代码能够顺利执行。
Qwen-Agent内置集成了 code_interpreter 组件，是一个用于​**​动态执行代码、处理数据并生成结果​**​的核心工具。可以运行代码，处理结构化数据csv/Excel,集成Pandas/Numpy等库，进行数据清洗、统计分析和复杂计算。通过 ​**​Matplotlib、Seaborn​**​ 等库自动生成图表（折线图、柱状图、热力图等），直观展示数据洞察等等。
因此需要先执行安装
```Bash
    uv pip install "qwen-agent[code_interpreter]"
```
运行结果
![[Pasted image 20250512214657.png]]
![[Pasted image 20250512214717.png]]
我这里没有看到图形，可能是我的模型问题吧。
```Python
    # qwen3_mcp_multi.py
    from qwen_agent.agents.assistant import Assistant
    from qwen_agent.utils.output_beautify import typewriter_print

    def init_agent_service():

        # 使用 Ollama 运行的模型
        llm_cfg = {
            'model': 'Qwen3:8B',
            'model_server': 'http://localhost:11434/v1',
            'api_key': 'EMPTY',
        }

        system = ('你是一个数据分析师，具有对本地数据库的增删改查能力，同时具有使用python代码的能力')

        tools = [{
            "mcpServers": {
                "sqlite" : {
                    "command": "uvx",
                    "args": [
                        "mcp-server-sqlite",
                        "--db-path",
                        "test.db"
                    ]
                },
            
            },
            
            
        },
        # pip install "qwen-agent[code_interpreter]"
        'code_interpreter',
        ]

        bot = Assistant(
            llm=llm_cfg,
            name='数据分析师',
            description='可以对本地数据库进行增删改查，也',
            system_message=system,
            function_list=tools,
        )

        return bot

    def run_query(query=None):
        # 定义数据库助手
        bot = init_agent_service()

        # 执行对话逻辑
        messages = []
        messages.append({'role': 'user', 'content': [{'text': query}]})

        # 跟踪前一次的输出，用于增量打印
        previous_text = ""
        
        print('数据库管理员: ', end='', flush=True)

        for response in bot.run(messages):
            previous_text = typewriter_print(response, previous_text)


    # 命令行接口，便于直接运行
    if __name__ == '__main__':

        query = '帮我随机生成10条学生的成绩数据, 插入到students表中,\
                然后绘制一个折线图展示所有成绩变化趋势'
        
        # 执行查询
        run_query(query)
        
```
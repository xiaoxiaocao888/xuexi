Assistant类中的function_list参数可以通过tool_use的方式传入MCP格式的调用示例，就像定义Function Calling的 JsonSchema 一样。其中 MCP 调用格式如下所示：
要把MCP工具封装成tools对象。sqlite就是MCP工具。uvx指的是python项目从pip上下载
```Python
tools = [{
    "mcpServers": {
        "sqlite" : {
            "command": "uvx",
            "args": [
                "mcp-server-sqlite",
                "--db-path",
                "test.db"
            ]
        }
    }
}]
```
这个 MCP 服务器配置的功能是启动一个 SQLite 数据库服务器，允许通过 MCP 协议与该数据库进行交互。具体功能包括：

- 数据库连接: 允许其他服务或客户端连接到 SQLite 数据库。
- 执行 SQL 查询: 通过 MCP 协议发送 SQL 查询到 SQLite 数据库并获取结果。
- 数据管理: 允许对数据库中的数据进行增、删、改、查等操作
  其中我们配置了三个参数：
- "mcp-server-sqlite": 这是指定要启动的 MCP 服务器类型，表示启动一个 SQLite 服务器。
- "--db-path": 这是一个选项，后面跟着数据库文件的路径。
- "test.db": 这是 SQLite 数据库的文件名，表示要使用的数据库文件。
准备好了MCP服务器后，接下来传递给`Assistant`对象。

目前创建MCP智能体的代码无法在Jupyter环境下运行
创建qwen3_mcp_sqlite.py文件
运行结果如下，可以看到运行成功后在本目录下生成了test.db文件，这就是MCP自动创建的数据库文件
![[Pasted image 20250512201847.png]]
![[Pasted image 20250512202501.png]]
```Python
    # qwen3_mcp_sqlite.py.py
    from qwen_agent.agents.assistant import Assistant
    from qwen_agent.utils.output_beautify import typewriter_print

    def init_agent_service():
        
        # 使用 Ollama 运行的模型
        llm_cfg = {
            'model': 'qwen3:0.6b',
            'model_server': 'http://localhost:11434/v1',
            'api_key': 'EMPTY',
        }

        system = ('你扮演一个数据库助手，你具有查询数据库的能力')

        tools = [{
            "mcpServers": {
                "sqlite" : {
                    "command": "uvx",
                    "args": [
                        "mcp-server-sqlite",
                        "--db-path",
                        "test.db"
                    ]
                }
            }
        }]
        
        bot = Assistant(
            llm=llm_cfg,
            name='数据库管理员',
            description='你是一位数据库管理员，具有对本地数据库的增删改查能力',
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

        query = '帮我创建一个学生表,表名是students,包含id, name, age, gender, score字段,然后插入一条数据,id为1,name为张三,age为20,gender为男,score为95'
        
        # 执行查询
        run_query(query)
```
创建query_sql_table.py文件来查询数据表的内容
```Python
import sqlite3

conn = sqlite3.connect('D:\\mcp-client\\test.db')
cursor = conn.cursor()

# 先查看数据库中有哪些表
cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
tables = cursor.fetchall()
print("数据库中的表:", tables)

# 如果有表，则查询第一个表的数据
if tables:
    table_name = tables[0][0]
    cursor.execute(f"SELECT * FROM {table_name}")
    print(f"{table_name} 表中的数据:", cursor.fetchall())
else:
    print("数据库中没有表，需要先创建表并插入数据")

conn.close()
```
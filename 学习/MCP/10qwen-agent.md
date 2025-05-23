Qwen-Agent是开源智能体开发框架，自带知识库检索。
借助Qwen-Agent，Qwen系列模型能够更好的开启Function calling功能（包括多步Function calling和自动debug等）、能够更好的进行模型记忆管理、能够更加便捷的连接MCP工具、能够进行长文档检索等，此外Qwen-Agent还提供了便捷的前端交互页面。

Qwen-Agent核心特性如下：
- 更强的工具调用（Function Calling）能力:框架支持智能体自动调用外部工具或函数，包括内置的代码解释器、浏览器助手等，也支持开发者自定义工具，扩展智能体的能力。
    
- 便捷的MCP工具接入流程：最新版的Qwen-Agent已经集成了MCP工具接入流程，我们仅需写入MCP配置，即可在Qwen-Agent中调用MCP工具：
![[Pasted image 20250511215823.png]]
- 规划与记忆能力： 智能体具备任务规划能力，能够根据用户需求自动制定执行步骤。同时，具备上下文记忆功能，能在对话中保持状态，提供连贯的交互体验。
    
- 长文本处理与 RA： Qwen-Agent 集成了检索增强生成（RAG）机制，支持处理从 8K 到 100 万 tokens 的长文档，通过文档分块和相关性检索，提升上下互与展示
    
- UI前端交互与展示 http://localhost:7860

##### Qwen-Agent下载与创建多轮对话机器人
```Python
uv pip install -U "qwen-agent[rag,code_interpreter,gui,mcp]"
```
- [gui] 用于提供基于 Gradio 的 GUI 支持；前端页面
- [rag] 用于支持 RAG；围绕本地知识库文档进行检索的插件
- [code_interpreter] 用于提供代码解释器相关支持；
- [mcp] 用于支持 MCP。工具识别

Qwen-Agent提供了接入Qwen系列模型，及在线API或者通过`Ollama`、`Vllm`启动本地模型等任意兼容`OpenAI API`接口规范的模型。其中通过`Assistant`组件，可以实现工具调用、Agent编排、MCP接入等等一系列功能。注意: 不同于类似`OpenAI Agents SDK`，`Assisttant` 类仅仅是`Qwen-Agent`构建不同形式代理的一个子类，主要用于实现tool_use和RAG的工作流。
Assistant参数介绍：
llm指定要使用的语言模型配置或实例.system_message系统消息，用于定义助手的行为和角色
name助手的名字
运行结果
![[Pasted image 20250512194354.png]]
```Python
from qwen_agent.agents import Assistant
from qwen_agent.utils.output_beautify import typewriter_print

# 步骤 2：配置您所使用的 LLM。
llm_cfg = {
    # 使用 ollama 提供的模型服务：
    'model': 'qwen3:8B',
    'model_server': 'http://localhost:11434/v1',
    "api_key":"ollama",

    # （可选） LLM 的超参数：
    'generate_cfg': {
        'temperature': 0.7,
    }
}
# 步骤3：创建智能体对象
bot = Assistant(llm=llm_cfg,
                system_message="你是一位乐于助人的小助理",
                name="智能助理"
                )

# 步骤 4：作为聊天机器人运行智能体。
messages = []  # 这里储存聊天历史。
while True:
    # 输入请求 。
    query = input('\n用户请求: 输入“退出”中止对话')
    if query == '退出':
        break
    else:
        # 将用户请求添加到聊天历史。
        messages.append({'role': 'user', 'content': query})
        response = []
        response_plain_text = ''
        
        print('AI 回复:')
        for response in bot.run(messages=messages):
            # 流式输出。
            response_plain_text = typewriter_print(response, response_plain_text)
        # 将机器人的回应添加到聊天历史。
        messages.extend(response)
```

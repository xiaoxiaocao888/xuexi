可以流式传输的数据主要有三类：
1. **工作流进度**——执行每个图形节点后获取状态更新。
2. **LLM 令牌**— 流式传输语言模型令牌。从任何地方捕获令牌流：内部节点、子图或工具。
3. **自定义更新**——发出用户定义的信号（例如，“获取 10/100 条记录”）。

**Reducers(归约器)** 更新状态的组件，就是更新状态的函数。方法有.copy复制状态。.update更新状态
**Context Reducer(上下文归约器)**
可以使用Context通道来定义共享资源(例如数据库连接)，这些资源在图节点之外进行管理，且不包括在检查点中。

**流式输出**可以查看中文文档 https://www.aidoczh.com/langgraph/how-tos/#_3
`.stream` 和 `.astream` 是同步和异步方法，用于从图形运行中流式返回输出。当调用这些方法时，可以指定几种不同的模式（例如 `graph.stream(..., mode="...")`）：
- `"values"`：在图形的每一步之后，流式传输状态的完整值。返回全部过程的值
- `"updates"`：在图形的每一步之后，流式传输对状态的更新。如果在同一步中进行了多次更新（例如，运行多个节点），则这些更新将分别流式传输。返回更新的值。
- `"custom"`：从图形节点内部流式传输自定义数据。
- `"messages"`：对于调用 LLM 的图形节点，流式传输 LLM 令牌和元数据。
- `"debug"`：在图形执行过程中，尽可能多地流式传输信息。
将多个流式模式作为列表传递来同时指定多个流式模式。流式输出将是元组 `(stream_mode, data)`。例如：graph是agent
```python
graph.stream(..., stream_mode=["updates", "messages"])
('messages', (AIMessageChunk(content='Hi'), {'langgraph_step': 3, 'langgraph_node': 'agent', ...}))
('updates', {'agent': {'messages': [AIMessage(content="Hi, how can I help you?")]}})
#也可以
async for chunk in graph.astream(inputs,stream_mode="updates"):
    for node,values in chunk.items():#查看节点，边的信息
```
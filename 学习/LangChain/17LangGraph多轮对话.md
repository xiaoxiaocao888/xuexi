对于消息序列格式state，使用LangGraph预构建的add_messages函数，即对于全新的消息，会附加到现有列表，同时也会正确处理现有消息的更新。
```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages
class State(TypedDict):
   messages:Annotated[list,add_messages]
graph_builder=StateGraph(State)
```
add_messages的核心逻辑是合并两个消息列表，按ID更新现有消息。参数
left(messages)--消息的基本列表  right(messsages)--要合并到基本列表的消息列表或单个列表。
如果right和left消息具有相同的ID，则right替换left的消息。否则作为一条新的消息进行追加。
```python
from langgraph.graph.message import add_messages
from langchain_core.messages import AIMessage,HumanMessage
msgs1=[HumanMessage(content="你好。",id="1")]
msgs2=[AIMessage(content="你好，很高心认识你。",id="2")]
print(add_messages(msgs1,msgs2))
msgs3=[HumanMessage(content="你好你好",id="1")]
print(add_messages(msgs1,msgs3))
```
![[Pasted image 20250714222053.png]]
手动构建多轮机器人对话
```python
import os 
from dotenv import load_dotenv
from langchain.chat_models import init_chat_model
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph,START,END
from langgraph.graph.message import add_messages
from langchain_core.messages import AIMessage,HumanMessage,SystemMessage
load_dotenv(override=True)
DeepSeek_API_KEY=os.getenv("DEEPSEEK_API_KEY")
model=init_chat_model(model="deepseek-chat",model_provider="deepseek")
class State(TypedDict):
   messages:Annotated[list,add_messages]
graph_builder=StateGraph(State)
def chatbot(state:State):
   return {"messages":[model.invoke(state["messages"])]}
graph_builder.add_node("chatbot",chatbot)
graph_builder.add_edge(START,"chatbot")
graph=graph_builder.compile()
from IPython.display import Image,display
#对图结构进行可视化
display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
final_state=graph.invoke({"messages":["你好，我叫丽丽，好久不见"]})
print(final_state)
messages_list=[
   HumanMessage(content="你好，我叫丽丽，好久不见。"),
   AIMessage(content="你好呀，我是小智，一名乐于助人的AI助手，很高心认识您！"),
   HumanMessage(content="请问，你还记得我叫什么名字么？"),
]
final_state1=graph.invoke({"messages":messages_list})
print(final_state)
messages_list1=[]
#手动构建多轮对话机器人
while True:
   try:
      user_input=input("用户提问： ")
      if user_input.lower() in ["exit"]:
         print("下次再见！")
         break
      messages_list1.append(HumanMessage(content=user_input))
      final_state1=graph.invoke({"messages":messages_list1})
      print("小智：",final_state1['messages'][-1].content)
      messages_list1.append(final_state1['messages'][-1])
      messages_list1=messages_list1[-50:]
   except:
      break
```
**借助MemorySaver高效搭建多轮对话机器人**
MemorySaver的核心功能
- 短期记忆(线程级记忆)，MemorySaver为每个thread_id保存和恢复对话状态(state)，实现在同一会话中的历史上下文记忆
- 状态持久化在每个节点运行后，State会自动存储；再次调用时，如果使用相同的thread_id，MemorySaver会回复此前保存的状态，无需手动传递历史信息
- 多对话隔离 通过不同的thread_id可实现会话隔离，允许多个用户并发交互且各自的对话互不干扰
- 图状态快照与恢复  不仅包括对话历史，还保存整个工作流状态，可用于错误恢复、时间旅行、断点续跑、Human-in-the-loop等高级场景
![[Pasted image 20250715143824.png]]
```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph,START,END
from langgraph.graph.message import add_messages
from langchain.chat_models import init_chat_model
from langgraph.checkpoint.memory import MemorySaver
#1.定义状态类(会自动合并messages)
class State(TypedDict):
   messages:Annotated[list,add_messages]
#2.初始化模型
model=init_chat_model(model="deepseek-chat",model_provider="deepseek")
#3.定义聊天节点
def chatbot(state:State)->State:
   reply=model.invoke(state["messages"])
   return {"messages":[reply]}
#4.构建带MemorySaver图
builder=StateGraph(State)
builder.add_node("chatbot",chatbot)
builder.add_edge(START,"chatbot")
memory=MemorySaver()
graph=builder.compile(checkpointer=memory)
from IPython.display import Image,display
#对图结构进行可视化
display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
#5.运行多轮对话，使用相同thread_id实现记忆
thread_config={"configurable":{"thread_id":"session_10"}}
state1=graph.invoke({"messages":[{"role":"user","content":"你好，好久不见。我叫丽丽。"}]},config=thread_config)
print(state1)
state2=graph.invoke({"messages":[{"role":"user","content":"你好呀，你还记得我的名字吗？"}]},config=thread_config)
print(state2['messages'])
#使用不同thread_id，会开启全新对话
state3=graph.invoke({"messages":[{"role":"user","content":"你好呀，你还记得我的名字吗？"}]},config=thread_config)
print(state3['messages'][-1].content)
#查看记忆
latest=graph.get_state(thread_config)
print(latest.value["messages"])
```
同样，可以对agent智能体加入记忆
```python
checkpointer = InMemorySaver()
agent = create_react_agent(model="anthropic:claude-3-7-sonnet-latest",tools=[get_weather], checkpointer=checkpointer)
config = {"configurable": {"thread_id": "1"}} 
sf_response = agent.invoke( {"messages": [{"role": "user", "content": "what is the weather in sf"}]}, config)
```
同样，agent也可以配置结构化输出.使用response_format参数，值为结构化的函数名
```python
agent=create_react_agent(model=model,tools=tools,response_format=State)
```
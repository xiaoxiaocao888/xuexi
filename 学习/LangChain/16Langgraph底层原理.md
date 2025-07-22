借助底层API，能够更加灵活的完成各类智能体的开发，而且在某些场景下，入实现人在闭环(Human in the loop)或者搭建多智能体(multi Agent)系统时，必须使用更加底层的API才能够完成。
人在闭环指的是在整个AI智能运行过程当中去借助一些人工协助来完成一些功能，如编写SQL代码后直接操作数据库(如询问你是否要调用或执行，确认了再去做)

Langgraph框架通过组合Nodes和Edges去构建复杂的循环工作流程，通过消息传递的方式串联所以的节点形成一个通路。通过State维持消息能够及时更新并向该去的地方传递。每次执行都会启动一个状态，节点在处理时会传递和修改该状态。
定义图时首先定义图State。状态表示会随着图计算的进行而维护和更新的上下文或记忆。它用来确保图中的每个步骤都可以访问先前步骤的相关信息，从而可以根据整个过程中积累的数据进行动态决策。使用StateGraph类来创建state状态
构建state:可以将图的状态设计为一个字典，用于在不同节点间共享和修改数据，然后使用StateGraph类进行图的实例化。
builder可以理解为图构建器对象，用于逐步添加节点、边、控制流逻辑，最终编译成可执行的LangGraph图，图构建器需要通过带入一个状态对象来创建
addition节点是一个加法逻辑，接收当前状态StateGraph(dict)，将字典中x的值增加1，并返回新的状态。
subtraction节点是一个减法逻辑，接收从addition节点传来的状态StateGraph（dict),从字典中的x值减去2，创建并返回一个新的键
图结构可视化的好处是：直接从代码中生成图形化的表示，可以检查图的执行逻辑是否符合构建的预期。
下载安装图可视化依赖的第三方库
```bash
pip install pyppeteer ipython -i https://pypi.tuna.tsinghua.edu.cn/simple
```
在vscode上面直接运行是看不到图片的，然后在扩展那里安装了jupyter，新增.ipynb结尾的文件，然后点击左边的箭头就可以运行成功
![[Pasted image 20250715125629.png]]
```python
from langgraph.graph import StateGraph
#使用stategraph接收一个字典
builder=StateGraph(dict)
def addition(state):
    #注意：这里接收到的是初始状态
    print(f"init_state:{state}")
    return {"x":state["x"]+1}
def subtraction(state):
    #注意，这里接收到的是上一个节点的状态
    print(f"addition_state:{state}")
    return {"x":state["x"]-2}
# START 和END节点表示图开始和结束
from langgraph.graph import START,END
#向图中添加两个节点 名字，函数
builder.add_node("addition",addition)
builder.add_node("subtraction",subtraction)
#构建节点之间的边
builder.add_edge(START,"addition")
builder.add_edge("addition","subtraction")
builder.add_edge("subtraction",END)
#可以通过下面方法查看边节点信息
print(builder.edges)
print(builder.nodes)
#通过compile()方法将这些设置编译成可执行的图
graph=builder.compile()
from IPython.display import Image,display
#对图结构进行可视化
display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
#编译后的graph对象提供了invoke方法，用来启动图的执行，可以传递初始状态（参数），作为输入
#定义一个初始化的状态
initial_state={"x":10}
print(graph.invoke(initial_state))
```

##### 借助Pydantic对象创建图
pydantic是一个用于创建“数据模型”的python库，可以自动校验数据类型，并将字典数据转换为结构化对象。
```python
state={"x":1,"y":"hello"} #传入没有类型限制
state["x"]="abc" #不小心改错也不会报错
from pydantic import BaseModel
class MyState(BaseModel):
   x:int
   y:str="default" #设置默认值
#自动校验
state=MyState(x=1)
print(state.x)
print(state.y)
state=MyState(x="abx") #会报错
```
![[Pasted image 20250715130108.png]]
所以我们可以在上面那个代码可以加上x的格式定义
```python
from pydantic import BaseModel
class CalcState(BaseModel):
   x:int
def addition(state:CalcState)->CalcState:
...
    return CalcState(x=state.x+1)
builder=StateGraph(CalcState)
initial_state=CalcState(x=10)
```

##### 创建条件分支图
![[Pasted image 20250715134149.png]]
```python
from typing import Optional
from pydantic import BaseModel
from langgraph.graph import StateGraph,START,END
#定义结构化状态
def MyState(BaseModel):
   x:int
   result:Optional[str]=None  #字符串可选属性
#定义各节点处理逻辑
def check_x(state:MyState)->MyState:
   print(f"[check_x] Received state:{state}")
   return state
def is_even(state:MyState)->bool:
   return state.x%2==0
def handle_even(state:MyState)->MyState:
   print("[handle_even]x是偶数")
   return Mystate(x=state.x,result="even")
def handle_odd(state:MyState)->MyState:
   print("[handle_odd]x是奇数")
   return MyState(x=state.x,result="odd")
#构建图
builder=StateGraph(MyState)
builder.add_node("check_x",check_x)
builder.add_node("handle_even",handle_even)
builder.add_node("handle_odd",handle_odd)
#添加条件分支 判别由边完成
builder.add_conditional_edges("check_x",is_even,{
   True:"handle_even",
   False:"handle_odd"
})
#衔接开始和结束
builder.add_edge(START,"check_x")
builder.add_edge("handle_even",END)
builder.add_edge("handle_odd",END)
#编译图
graph=builder.compile()
#执行测试
print("\n 测试x=4（偶数）")
print(graph.invoke(MyState(x=4)))
print("\n 测试x=3（奇数）")
from IPython.display import display,Image
display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
```

##### 创建循环图
![[Pasted image 20250715134342.png]]
```python
from pydantic import BaseModel
from langgraph.graph import StateGraph,START,END
#1.定义结构化状态模型
class LoopState(BaseModel):
   x:int
#2.定义节点逻辑 函数名和属性名不可同
def increment(state:LoopState)->LoopState:
   print(f"[inctement]当前x={state.x}")
   return LoopState(x=state.x+1)
def is_done(state:LoopState)->bool:
   return state.x>10
#3.构建图
builder=StateGraph(LoopState)
builder.add_node("increment",increment)
#4.设置循环控制，is_done为True则结束，否则继续
builder.add_conditional_edges("increment",is_done,{
   True:END,
   False:"increment"
})
builder.add_edge(START,"increment")
graph=builder.compile()
#5.测试执行
print("\n 执行循环直到x>10")
final_state=graph.invoke(LoopState(x=6))
print(f"[最终结果]->x={final_state['x']}")
from IPython.display import display,Image
display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
```
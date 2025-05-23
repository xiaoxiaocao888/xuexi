为了支持调用OpenAI模型，以及在环境变量中读取API-KEY等信息，需先安装
```
uv add mcp openai python-dotenv
```
在mcp-client文件夹下创建.env文件，并写入OpenAI的API-KEY，以及反向代理地址。借助反向代理，国内可以无门槛直连OpenAI官方服务器，并调用官方API。写入如下内容
```python
BASE_URL="反向代理地址" #https://api.deepseek.com
MODEL=gpt-4o  #可更换为DeepSeek或本地模型
OPENAI_API_KEY="OpenAI-API-Key" #"DeepSeek API-Key
```
修改client.py代码，全部代码如下
这是运行结果
![[Pasted image 20250507213038.png]]
```python
import asyncio
#from mcp import  ClientSession
import os
from openai import OpenAI
from dotenv import load_dotenv
from contextlib import AsyncExitStack
#加载.env文件，确保API Key 受到保护
load_dotenv()
class MCPClient:
    def __init__(self):
        """初始化MCP客户端"""
        #self.session=None
        self.exit_stack=AsyncExitStack()
        self.openai_api_key=os.getenv("OPENAI_API_KEY")#读取openai key
        self.base_url=os.getenv("BASE_URL")
        self.model=os.getenv("MODEL")
        if not self.openai_api_key:
            raise ValueError("未找到OpenAI API Key，请在.env文件中设置OPENAI_API_KEY")
        self.client=OpenAI(api_key=self.openai_api_key,base_url=self.base_url)
    async def process_qury(self,query:str)->str:
        """调用OpenAI API 处理用户查询"""
        messages=[{"role":"system","content":"你是一个智能助手，帮助用户解决问题"},
                  {"role":"user","content":query}]
        try:
            #调用OpenAI API
            response=await asyncio.get_event_loop().run_in_executor(
                None,
                lambda:self.client.chat.completions.create(
                    model=self.model,
                    messages=messages
                )
            )
            return response.choices[0].message.content
        except Exception as e:
            return f"调用OpenAI API时出错：{str(e)}"
    async  def chat_loop(self):
        """运行交互式聊天循环"""
        print("\nMCP 客户端已启动！输入'quit'退出")
        while True:
            try:
                query=input("\n你: ").strip()
                if query.lower()=='quit':
                    break
                response=await self.process_qury(query)#发送用户输入到openai
                print(f"\n OpenAI:{response}")
            except Exception as e:
                print(f"\n 发生错误：{str(e)}")
    async  def cleanup(self):
        """清理资源"""
        await  self.exit_stack.aclose()
async def main():
    client=MCPClient()
    try:
        await client.chat_loop()
    finally:
        await client.cleanup()
if __name__=="__main__":
    asyncio.run(main())
```
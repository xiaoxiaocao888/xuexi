下载
```
pip install aiohttp
```
requests.get()同步的代码-->异步操作aiohttp
使用with语句，是后面执行完后，对应的对象会自动关闭，如文件f
```python
import asyncio
import aiohttp
urls=[
    "https://www.umei.cc/d/file/20230616/8d21a66a232b9d163a379a6a962c9d62.jpg",
    "https://www.umei.cc/d/file/20230823/c62654b744cd93951de5d60c15716a2d.jpg",
    "https://www.umei.cc/d/file/20221104/398dc39206d36ea9021293c815672fb6.jpg"
]
async def aiodownload(url):
    #发送请求-->得到图片内容-->保存到文件
    name=url.rsplit("/",1)[1]#从右边切，切一次，得到[1]位置的内容
    async with aiohttp.ClientSession() as session:#request
        async with session.get(url) as resp: #resp=requests.get()
            #请求回来了，写入文件
            #可以去学习aiofiles模块
            print(resp.status)
            with open("./img/"+name,mode="wb") as f:
                f.write(await resp.content.read())#读取内容是异步的，需要await挂起，有些是用resp.text()要加括号
    print(name,"下载OK")
async def main():
    tasks=[]
    for url in urls:
        tasks.append(aiodownload(url))
    await asyncio.wait(tasks)
if __name__=='__main__':
    asyncio.run(main())
```
前情提要可复习MCP文件夹的外部函数
在第一次请求里面，与对话类模型不一样(比如DeepSeek-v3)，对话模型当触发函数调用时模型回复结果为空，但是deepseek-r1即使触发了函数调用，其`content`中依然会返回响应内容。如下所示：
```Python
response.choices[0].message.content

'我将调用函数查询北京的天气情况。'
```
生成的"tool_calls"的list包含了当前调用外部函数的全部信息
```Python
response_message.tool_calls

[ChatCompletionMessageToolCall(id='call_0_08e1ba29-18e9-428d-be3e-1b840c48dff6', function=Function(arguments='{"loc": "Beijing"}', name='get_weather'), type='function', index=0)]
```
对于当前CompletionMessageToolCall对象，id为外部函数调用发起请求id，function则表示调用外部函数基本信息，而type则代表了当前当前调用外部函数类型，function代表调用自定义的外部函数。

##### r1-0528和其他不一样的地方
在第一二次请求之间对messages进行拼接那里，要把reasoning_content删除。
```Python
for message in messages:
    if 'reasoning_content' in message:
        del message['reasoning_content']
```
##### 多函数并联调用，一次性构建完整回复
我之前在qwen3+mcp那里的前端页面可以直接得到并联结果，并不需要其他的函数。
```Python
import json
import requests

def create_function_response_messages(messages, response):
    function_call_messages = response.choices[0].message.tool_calls
    messages.append(response.choices[0].message.model_dump())
    for function_call_message in function_call_messages:
        tool_name = function_call_message.function.name
        tool_args = json.loads(function_call_message.function.arguments)
        
        # 运行外部函数
        fuction_to_call = available_functions[tool_name]
        try:
            function_response = fuction_to_call(**tool_args)
        except Exception as e:
            function_response = "函数运行报错如下:" + str(e)

        # 拼接消息队列
        messages.append(
            {
                "role": "tool",
                "content": function_response,
                "tool_call_id": function_call_message.id,
            }
        )

        # 删除reasoning_content字段
        for message in messages:
            if 'reasoning_content' in message:
                del message['reasoning_content']
    return messages    
messages2 = create_function_response_messages(messages2, response2)
response = client.chat.completions.create( model="deepseek-reasoner", messages=messages2, tools=tools, ) 
```

##### 串联调用
就是在available_functions那里加上函数的映射，在tools加上函数的描述

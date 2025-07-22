下载数据集
```
modelscope download --dataset swift/self-cognition self_cognition.jsonl --local_dir ./dataset/
```
格式是这样的
![[Pasted image 20250605111706.png]]
要把格式替换成和sft_mini_512.jsonl的一样。我们可以先选中要替换的内容如`{"query":`然后按ctrl+H ,粘贴上要替换的内容，点下面右下角那个，就可以全部都替换啦
![[Pasted image 20250605113157.png]]
还要处理结尾的}，要把它替换成}]}。就是Ctrl+H，然后点击`.*`符号，输入正则化表达式`\}$`就是匹配每行以}结尾的,然后再下方替换处写为}]}即可。
![[Pasted image 20250605114013.png]]
这样我们的self_cognition.jsonl文件的格式就都是这样的啦
```
{"conversations": [{"role": "user", "content": "你是？"}, {"role": "assistant", "content": "我是{{NAME}}，由{{AUTHOR}}训练的人工智能助手。我的目标是为用户提供有用、准确和及时的信息，并通过各种方式帮助用户进行有效的沟通。请告诉我有什么可以帮助您的呢？", "tag": "zh"}]}
```
在train_lora.py文件里面main函数参数是`data_path`的文件名修改为self_cognition.jsonl
把init_model函数里面的ckp参数的rlhf修改为pretrain，还要在out目录下新建lora文件夹
运行该文件，就会生成out/lora_identity_512.pth模型文件
![[Pasted image 20250605192826.png]]
![[Pasted image 20250605192918.png]]
去做模型评估，eval_model.py文件里面的lora_name，的默认值修改为lora_identity，运行
![[Pasted image 20250605193212.png]]

把train_lora.py文件复制粘贴到`https://chat.deepseek.com/`，并加上一句话，请将以上代码修改为通过peft包来进行LoRA微调，发送。然后复制deepseek生成的代码，粘贴到新建的train_lora_peft.py文件
然后我们拿train_lora_peft.py和train_lora.py文件做对比
1.这个地方是做梯度裁剪，为防止网络训练时出现梯度爆炸，以参数梯度的L2范数和给定阈值来进行调节，虽然左边的代码包含所有的代码，但是基座模型的梯度计算是冻结了的，所以不受影响。函数名的下划线表示这是一个原地操作函数，如lora_params
![[Pasted image 20250605145006.png]]
2.大模型生成的代码，它是新增加这段代码，使用LoraConfig类来定义它的相关的一些参数。
通过get_peft_model函数来初始化微调模型。打印可训练参数
![[Pasted image 20250605145613.png]]
3.大模型这里创建了lora目录，我们自定义的包里面是漏了的
![[Pasted image 20250605150213.png]]
4.我们自己写的代码，下面才是获取参数以及打印相关参数，以自定义的方式设置了哪些参数不需要做梯度计算。
![[Pasted image 20250605150424.png]]

运行train_lora_peft.py文件，要是看到报这样的错误q_proj,v_proj找不到，那可能是要把它修改为wq,wv，因为不同的会有不同的命名
运行成功，可以看到out/lora下面生成了新的文件
![[Pasted image 20250605193411.png]]
![[Pasted image 20250605193349.png]]
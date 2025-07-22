**策略模型pi:**
**参考模型ref**
**偏好数据三元组**
**温度系数beta**

修改train_dpo.py文件main函数里面的参数
`log_interval、save_interval`都改为1，默认是100
`max_seq_len`改为100，默认为1024

下载数据集
```
modelscope download --dataset gongjy/minimind_dataset dpo.jsonl --local_dir ./dataset/
```

观察train_dpo.py文件
我们可以看到这个是多了一个ref_model参考模型
```
model, ref_model, tokenizer = init_model(lm_config)
```
进入init_model这个类我们可以看到，参数模型多了后面两句：设置成评估模型，设置成不需要梯度
![[Pasted image 20250605163609.png]]
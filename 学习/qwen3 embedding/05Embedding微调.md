##### 1.创建虚拟环境
```
conda create -n swift_venv python=3.12 -y
conda init bash && source /root/.bashrc
conda activate swift_venv
conda install ipykernel
ipython kernel install --user --name=swift_venv
```
##### 2.安装ms-swift
```bash
pip install git+https://github.com/modelscope/ms-swift.git
#如果是在云服务器上还要科学上网，在前面增加
source /etc/network_turbo
```
##### 3.安装其他依赖
```bash
pip install deepspeed liger-kernel
pip install scikit-learn
pip install -U sentence-transformers
# 下面可能要科学上网才能下载好
source /etc/network_turbo
pip install flash-attn --no-build-isolation
```
##### 4.下载训练数据集
```python
#设置下面命令，就无需科学上网
export HF_ENDPOINT=https://hf-mirror.com
pip install --upgrade huggingface_hub
huggingface-cli download --repo-type dataset --resume-download microsoft/ms_marco --local-dir /root/xxx
```
##### 5.开始embedding模型训练
###### 方法一：全参数训练
```bash
bash embedding_sft.sh
```
该文件命令为
```bash
#!/bin/bash
#nproc 通常指 "number of processes"。
nproc_per_node=1 
#设置为环境变量，告诉 swift 框架，你打算在单个GPU上运行这个训练任务。
NPROC_PER_NODE=$nproc_per_node \
swift sft \
    --model /data/home/zengxiangxi/xuexi/qwen3/model/qwen3_embedding_0.6b \
    --task_type embedding \
    --model_type qwen3_emb \
    --train_type full \
    #指定要训练的数据集，表示只使用正面数据集
    --dataset sentence-transformers/stsb:positive  \
   #只使用整个数据集的5%
	--split_dataset_ratio 0.05 \
    --eval_strategy steps \
    --output_dir output \
    --eval_steps 200 \
    --num_train_epochs 5 \
    --save_steps 200 \
    --per_device_train_batch_size 8 \
    --per_device_eval_batch_size 4 \
    --gradient_accumulation_steps 2 \
    --learning_rate 6e-6 \
    --loss_type cosine_similarity  \
    --label_names labels \
    --dataloader_drop_last true 
```
![[Pasted image 20250708171919.png]]
运行完我们要测试一下这个模型，我们可以把`qwen3_embedding_transformers.py`文件里面加载的模型地址修改为我们上面微调训练结束后保存的模型地址。
![[Pasted image 20250708172332.png]]
**损失函数对比**
MS-SWIFT框架支持多种损失函数，用于训练Embedding。选择合适的损失函数对模型性能至关重要，不同的损失函数适用于不同的数据格式和训练目标。
**InfoNCE损失**
原理：对比学习损失函数，最大化正样本对相似度，最小化负样本对相似度；使用批内对比学习策略，将同批次内其他样本作为负样本
数据格式
```
{"query":"sentence1","response":"sentence1-positive","rejected_response":["sentence1-negative1","sentence1-negative2"]}
```
环境变量配置
```bash
export INFONCE_TEMPERATURE=0.01 #温度参数，控制相似度分布锐利度
export INFONCE_USE_BATCH=True   #使用批内负样本
export INFONCE_HARD_NEGATIVES=False #困难负样本数量标准化
export INFONCE_MASK_FAKE_NEGATIVE=False #屏蔽假负样本
```

**余弦相似度损失**
原理：直接优化预测相似度与真实相似度标签的差异；使用MSE损失计算`||input_label-cosine_sim(u,v)||_2`
数据格式：
```
{"query":"sentence1","response":"sentence2","label":0.8}
```

**对比学习损失**
原理：经典的对比学习损失，正样本拉近，负样本推远；需要设置margin参数
数据格式：
```
{"query":"sentence1","response":"sentence2","label":1} #1正样本，0负样本
```

**在线对比学习损失**
原理：对比学习的改进版本，选择困难正样本和困难负样本；通常比标准对比学习效果更好
数据格式：
```
{"query":"sentence1","response":"sentence2","label":1} #1表正0表负
```

**训练数据类型对应的损失函数**


| 数据类型    | 推荐损失函数                         | 原因            |
| ------- | ------------------------------ | ------------- |
| 只有正样本对  | infonce                        | 利用批内负样本，训练信号强 |
| 正负样本对   | contrastive或online_contrastive | 直接利用标注的负样本    |
| 连续相似度分数 | cosine_similarity              | 直接优化相似度预测     |
###### 方法二：lora微调
```bash
bash embedding_lora.sh
```
文件详细内容为
```bash
#!/bin/bash

nproc_per_node=1 
NPROC_PER_NODE=$nproc_per_node \
swift sft \
    --model /data/home/zengxiangxi/xuexi/qwen3/model/qwen3_embedding_0.6b \
    --task_type embedding \
    --model_type qwen3_emb \
    --train_type lora \
    --lora_rank 16 \
    --lora_alpha 32 \
    --lora_dropout 0.1 \
    --dataset sentence-transformers/stsb:positive  \
    --split_dataset_ratio 0.05 \
    --eval_strategy steps \
    --output_dir output \
    --eval_steps 200 \
    --num_train_epochs 5 \
    --save_steps 200 \
    --per_device_train_batch_size 8 \
    --per_device_eval_batch_size 4 \
    --gradient_accumulation_steps 2 \
    --learning_rate 6e-6 \
    --loss_type cosine_similarity  \
    --label_names labels \
    --dataloader_drop_last true 


```
![[Pasted image 20250709165408.png]]
训练完后，将Lora权重和模型进行合并，会输出一个合并之后的这个模型的权重
```bash
swift export --adapters 上面训练完保存的lora权重路径 --merge_lora true --output_dir xxx --safe_serialization true --exist_ok true
```
把合并后的模型复制粘贴到`qwen3_embedding_transformers.py`文件里面替换，测试模型
```
swift export --adapters /data/home/zengxiangxi/xuexi/qwen3/output/v7-20250709-164716/checkpoint-520 --merge_lora true --output_dir/data/home/zengxiangxi/xuexi/qwen3/model/mergemodel --safe_serialization true --exist_ok true
```
输出目录要求不存在，要是存在，就不会保存或更新
![[Pasted image 20250709171132.png]]
![[Pasted image 20250709171348.png]]
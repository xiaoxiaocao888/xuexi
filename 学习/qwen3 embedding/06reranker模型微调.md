**项目简介**：对qwen3-reranker-0.6b模型的监督微调，通过Lora进行参数高效微调，用于改善查询-文档匹配的重排效果
**项目结构**：
![[Pasted image 20250707163656.png]]

**环境配置**
在embedding模型虚拟环境的基础上。基于下面这个框架来微调的
```bash
pip install -U FlagEmbedding[finetune]
```

**数据格式**
训练数据应为JSONL格式，每行包含一个样本
```bash
{
"query":"查询文本",
"pos":["正例文档1"],
"neg:["负例文档1","负例文档2","负例文档3"]
}
```

#### 使用方法
##### 1.准备数据
将训练数据准备成JSONL格式，即toy_dataset.jsonl文件
##### 2.配置训练参数
修改`reranker_sft.sh`中的参数
就是要修改为自己的`train_data model_name_or_path output_dir`
###### 3.开始训练
```bash
bash reranker_sft.sh
```
文件内容
```bash
#!/bin/bash

# 数据和训练参数
train_data="/data/home/zengxiangxi/xuexi/qwen3/toy_dataset.jsonl"
# 训练轮数
num_train_epochs=3
per_device_train_batch_size=4
# 梯度累积步数
gradient_accumulation_steps=1
 # 训练组大小
train_group_size=6
# GPU数量
num_gpus=1


# 模型参数（使用LoRA进行参数高效微调）
model_args="\
    --model_name_or_path /data/home/zengxiangxi/xuexi/qwen3/model/qwen3_reranker_0.6b \
    --use_lora True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --use_flash_attn True \
    --target_modules q_proj k_proj v_proj o_proj \
    --save_merged_lora_model True \
    --model_type decoder \
"


# 数据参数（包含指令格式设置）
data_args="\
    --train_data $train_data \
    --cache_path ~/.cache \
    --train_group_size $train_group_size \
    --query_max_len 512 \
    --passage_max_len 512 \
    --pad_to_multiple_of 8
"


# 训练参数
training_args="\
    --output_dir /data/home/zengxiangxi/xuexi/qwen3/output \
    --overwrite_output_dir \
    --learning_rate 2e-4 \
    --bf16 \
    --num_train_epochs $num_train_epochs \
    --per_device_train_batch_size $per_device_train_batch_size \
    --gradient_accumulation_steps $gradient_accumulation_steps \
    --dataloader_drop_last True \
    --warmup_ratio 0.1 \
    --gradient_checkpointing \
    --weight_decay 0.01 \
    --deepspeed /root/autodl-tmp/Qwen3-Reranker-SFT/ds_stage0.json \
    --logging_steps 1 \
    --save_steps 1000 \
"

# 执行训练
torchrun --nproc_per_node $num_gpus \
    -m FlagEmbedding.finetune.reranker.decoder_only.base \
    $model_args \
    $data_args \
    $training_args
```
##### 4.测试模型
注意里面加载的模型地址要是你上面训练结束保存的模型地址
```
python test.py
CUDA_VISIBLE_DEVICES=5 python your_script.py
```
我们写提示词的时候，最好在提示词后面再加一个引子即请问，xx是：
当然，即使我们写了提示词，大模型也可能不会输出我们想要的格式，那就是要训练
使用ensure_ascii=False，这样就不会转码不会转成unicode，这样就保持原始的中文

去https://github.com/owenliang/qwen-sft/blob/main这里面下载好city.txt文件
在服务器上面damoxing文件夹下面新建qwen_datacreate.py文件，执行该文件生成train.xlsx文件。
```python
import random
import json
import time 

Q_arr=[]
A_arr=[]

Q_list=[
    ('{city}{year}年{month}月{day}日的天气','%Y-%m-%d'),
    ('{city}{year}年{month}月{day}号的天气','%Y-%m-%d'),
    ('{city}{month}月{day}日的天气','%m-%d'),
    ('{city}{month}月{day}号的天气','%m-%d'),

    ('{year}年{month}月{day}日{city}的天气','%Y-%m-%d'),
    ('{year}年{month}月{day}号{city}的天气','%Y-%m-%d'),
    ('{month}月{day}日{city}的天气','%m-%d'),
    ('{month}月{day}号{city}的天气','%m-%d'),

    ('你们{year}年{month}月{day}日去{city}玩吗？','%Y-%m-%d'),
    ('你们{year}年{month}月{day}号去{city}玩么？','%Y-%m-%d'),
    ('你们{month}月{day}日去{city}玩吗？','%m-%d'),
    ('你们{month}月{day}号去{city}玩吗？','%m-%d'),
]
with open('city.txt','r',encoding='utf-8') as fp:
    city_list=fp.readlines()
    city_list=[line.strip().split(' ')[1] for line in city_list]
prompt_template='''
给定一句话：“%s”，请你按步骤要求工作。

步骤1：识别这句话中的城市和日期共2个信息
步骤2：根据城市和日期信息，生成JSON字符串，格式为{"city":城市,"date":日期}

请问，这个JSON字符串是：
'''
# 生成一批"1月2号"、"1月2日"、"2023年1月2号", "2023年1月2日", "2023-02-02", "03-02"之类的话术, 教会它做日期转换
for i in range(1000):
    Q=Q_list[random.randint(0,len(Q_list)-1)]
    city=city_list[random.randint(0,len(city_list)-1)]
    year=random.randint(1990,2025)
    month=random.randint(1,12)
    day=random.randint(1,28)
    time_str='{}-{}-{}'.format(year,month,day)
    date_field=time.strftime(Q[1],time.strptime(time_str,'%Y-%m-%d'))
    Q=Q[0].format(city=city,year=year,month=month,day=day) # 问题
    A=json.dumps({'city':city,'date':date_field},ensure_ascii=False)  # 回答

    Q_arr.append(prompt_template%(Q,))
    A_arr.append(A)

import pandas as pd 

df=pd.DataFrame({'Prompt':Q_arr,'Completion':A_arr})
df.to_excel('train.xlsx')
```
#### 新建qwen_json.py文件，来训练模型
将 pandas DataFrame 转换为 Hugging Face 的 Dataset 对象，使其可以使用 Hugging Face 生态系统的各种功能。
```python
from datasets import Dataset
dataset = Dataset.from_pandas(df)  # 将pandas DataFrame转换为Hugging Face数据集对象
```
`create_training_instance`：将原始数据转换为对话格式
`remove_columns`：移除不需要的原始列   `df.columns.tolist()` 获取列名，转成列表形式
`load_from_cache_file=False` 强制每次运行都重新执行映射函数，而不是使用之前保存的缓存结果。如果这次的函数修改了，就避免使用上一次运行的结果
```python
formatted_dataset = dataset.map(create_training_instance, remove_columns=df.columns.tolist(),load_from_cache_file=False)
```
将文本转换为模型可理解的 token ID，添加 attention mask（指示哪些位置是有效内容），创建 labels（训练目标）， 应用批处理提高效率
```python
tokenized_dataset = formatted_dataset.map(
    tokenize_and_prepare_labels, 
    batched=True,
    batch_size=4,
    remove_columns=["text"]
)
```
Hugging Face 数据集处理的核心价值在于：
- **内存优化**：使用内存映射技术，即使处理超大数据集也不会耗尽内存
- **惰性加载**：只在需要时才加载数据，节省资源
- **高效处理**：自动并行化和批处理加速数据处理
- **标准化接口**：与 Trainer 类无缝集成
- **缓存机制**：自动缓存处理结果，避免重复计算
加载分词器或模型时，要写上参数`trust_remote_code=True` 表示加载那个地址里面的自定义代码
在加载模型时，`device_map="auto"` 表示自动分配模型到可用设备(GPU/CPU),多GPU/CPU混合使用；`torch_dtype=torch.bfloat16` 是 PyTorch 中用于控制模型计算精度的关键参数。
参数控制：
1. **模型权重的存储格式**：权重从磁盘加载后转为BF16格式存储在显存中
2. **前向传播的计算精度**：所有计算使用BF16进行
3. **激活值的存储格式**：中间结果使用BF16存储
如果不确定能不能使用，我们可以 .都是True就可以
```python
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"BF16 supported: {torch.cuda.is_bf16_supported()}")
```

| 数据类型           | 位宽  | 指数位 | 尾数位 | 动态范围 | 精度  | 模型加载选项    |
| -------------- | --- | --- | --- | ---- | --- | --------- |
| float32(FP32)  | 32位 | 8位  | 32位 | 大    | 高   | 高精度但耗显存   |
| float16(FP16)  | 16位 | 5位  | 10位 | 小    | 中   | 省显存但易数值溢出 |
| bfloat16(BF16) | 16位 | 8位  | 7位  | 大    | 低   | 平衡显存与数值范围 |

快速分词器是 Hugging Face 用 Rust 语言重新实现的分词器版本

| 特性     | 慢速分词器 (Python) | 快速分词器 (Rust) |
| ------ | -------------- | ------------ |
| 实现语言   | Python         | Rust         |
| 执行速度   | 慢（解释型语言）       | **快**（编译型语言） |
| 内存占用   | 高              | 低            |
| 特殊字符处理 | 基础             | **高级处理**     |
| 并行能力   | 有限             | **原生多线程**    |
| 功能完整性  | 基本功能           | **完整功能**     |
| 错误处理   | 简单             | 健壮           |
###### 添加token代码解释
核心作用是**确保分词器(Tokenizer)具有完整且可用的特殊Token**，特别是对于中文模型训练至关重要。主要完成三个关键任务：
1. **填充Token(PAD)设置**：确保分词器有填充Token，用于批处理时统一序列长度
2. **结束Token(EOS)设置**：确保模型知道何时结束文本生成
3. **词汇表扩展**：当添加新Token时，扩展分词器的词汇表

GPT系列模型：必须定义EOS（结束符）
BERT系列模型：必须定义PAD（填充符）
所有模型：训练时都需要处理批次序列对齐问题
Qwen3-0.6B 已经预定义了以下关键特殊 Token：
- **EOS Token**: `<|endoftext|>` (ID: 151643)
- **PAD Token**: **未明确定义**，但通常使用 EOS Token 作为填充
- **其他特殊 Token**:
    - `<|im_start|>`: 对话开始标记
    - `<|im_end|>`: 对话结束标记
    - `<|system|>`: 系统提示标记
可是适当简化
不需要添加额外的`[PAD] Token`,因为Qwen推荐使用eos_token作为pad_token
不需要扩展词汇表,Qwen已包含所有必要特殊Token
###### 调整模型大小以匹配分词器
模型嵌入矩阵维度 = (分词器词汇表大小, 隐藏层维度)
模型嵌入层需要匹配分词器的词汇量,所以添加了新Token就需要调整模型嵌入层


```python
import pandas as pd
import torch
import re
import numpy as np
from datasets import Dataset
from peft import LoraConfig, get_peft_model, TaskType, PeftModel
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    TrainingArguments,
    Trainer,
    DataCollatorForLanguageModeling,
)

# --- 辅助函数：确保分词器和模型配置一致 ---
def prepare_tokenizer_and_model(model_path, add_special_tokens=True):
    """统一处理分词器和模型的初始化，确保配置一致"""
    print("正在加载分词器...")
    tokenizer = AutoTokenizer.from_pretrained( #pretrained预训练的
        model_path,
        trust_remote_code=True,
        use_fast=True,  # 使用快速分词器更好处理特殊字符
    )
    
    # 添加特殊tokens
    if add_special_tokens:
        special_tokens_dict = {}
        if tokenizer.pad_token is None:
            special_tokens_dict['pad_token'] = '[PAD]'
            print("设置pad_token为[PAD]")
        
        if tokenizer.eos_token is None:
            tokenizer.eos_token = '<|endoftext|>'
            print(f"设置eos_token为: {tokenizer.eos_token}")
        
        if special_tokens_dict:
            print(f"添加特殊token: {special_tokens_dict}")
            # 保存原始vocab_size用于检查
            original_vocab_size = len(tokenizer)
            tokenizer.add_special_tokens(special_tokens_dict)
            new_vocab_size = len(tokenizer)
            print(f"分词器词汇表大小从 {original_vocab_size} 增加到 {new_vocab_size}")
    
    print("正在加载模型...")
    model = AutoModelForCausalLM.from_pretrained(
        model_path,
        trust_remote_code=True,
        device_map="auto",
        torch_dtype=torch.bfloat16,
    )
    
    # 调整模型大小以匹配分词器
    old_vocab_size = model.config.vocab_size # 模型原始词汇表大小
    new_vocab_size = len(tokenizer) # 当前分词器词汇表大小
    
    if old_vocab_size != new_vocab_size:
        print(f"调整模型embedding大小: {old_vocab_size} -> {new_vocab_size}")
        model.resize_token_embeddings(new_vocab_size) #扩展模型输入/输出嵌入层的维度
        
        # 初始化新添加token的embedding
        with torch.no_grad():
            new_token_embeddings = model.get_input_embeddings().weight
            # 确保有新的token需要初始化
            if new_vocab_size > old_vocab_size:
                # 使用现有token的平均值初始化新token
                new_token_embeddings[old_vocab_size:] = torch.mean(new_token_embeddings[:old_vocab_size], dim=0, keepdim=True)
        
        # 更新模型配置 确保模型知道哪个Token是填充Token
        model.config.pad_token_id = tokenizer.pad_token_id
    
    # 验证词表一致性
    validate_vocab_size(tokenizer, model)
    
    return tokenizer, model

def validate_vocab_size(tokenizer, model):
    """验证分词器和模型的词汇表大小是否一致"""
    tokenizer_size = len(tokenizer)
    model_size = model.config.vocab_size
    
    if tokenizer_size != model_size:
        raise ValueError(f"分词器词汇表大小 ({tokenizer_size}) 与模型词汇表大小 ({model_size}) 不一致！")
    
    print(f"分词器和模型词汇表大小一致: {tokenizer_size}")

# --- 数据清洗函数 ---
def clean_text(text):
    """
    清洗文本数据：
    1. 移除非BMP字符（Unicode > 0xFFFF）
    2. 移除零宽空格等特殊字符
    3. 替换连续换行符
    """
    if not isinstance(text, str):
        return ""
    
    # 移除非BMP字符（如emoji、特殊符号）
    cleaned = ''.join(c for c in text if ord(c) <= 0xFFFF)
    
    # 替换特定问题字符
    cleaned = cleaned.replace('\u200b', ' ')   # 零宽空格
    cleaned = cleaned.replace('\ufffd', '')     # 替换字符
    cleaned = cleaned.replace('\x00', '')       # 空字符
    
    # 标准化空白字符
    cleaned = re.sub(r'\s+', ' ', cleaned).strip() #正则表达式是把连续的空格替换成一个空格
    
    return cleaned

# --- 1. 配置参数 ---
model_path = "/data/home/zengxiangxi/hf_model/qwen3-0.6b"
data_file_path = "train.xlsx"
output_dir = "./qwen3_template_finetune"

# --- 2. 加载模型和分词器 ---
tokenizer, model = prepare_tokenizer_and_model(model_path)

# 打印实际词表大小用于调试
print(f"实际词表大小: {tokenizer.vocab_size}")
print(f"Pad token ID: {tokenizer.pad_token_id}")
print(f"EOS token ID: {tokenizer.eos_token_id}")

# --- 3. 加载和预处理数据 ---
print("正在加载和处理数据...")
df = pd.read_excel(data_file_path)
df.dropna(subset=['Prompt', 'Completion'], inplace=True)

if df.empty:
    raise ValueError(f"加载数据后，DataFrame 为空。")
print(f"数据加载成功，共 {len(df)} 条有效数据。")

# 数据清洗：应用清洗函数
print("正在清洗数据...")
df['Prompt'] = df['Prompt'].apply(clean_text)
df['Completion'] = df['Completion'].apply(clean_text)
print("数据清洗完成。")

# 转换数据格式
def create_training_instance(row):
    user_prompt = row['Prompt']
    assistant_completion = row['Completion']
    formatted_text = (
        f"<|im_start|>user\n{user_prompt}<|im_end|>\n"
        f"<|im_start|>assistant\n{assistant_completion}<|im_end|>"
    )
    return {"text": formatted_text}

dataset = Dataset.from_pandas(df)
formatted_dataset = dataset.map(create_training_instance, remove_columns=df.columns.tolist(), load_from_cache_file=False)

# 打印样本检查
print("\n--- 样本检查 ---")
for i in range(min(3, len(formatted_dataset))):
    sample = formatted_dataset[i]
    print(f"样本 {i} 长度: {len(sample['text'])}")
    print(f"前100字符: {sample['text'][:100]}...")
    print("-" * 50)

# --- 4. 分词与标签化（关键修复）---
def tokenize_and_prepare_labels(examples):
    # 确保文本字段存在且不为空
    texts = examples['text']
    for i, text in enumerate(texts):
        if text is None or text.strip() == "":
            print(f"警告: 样本 {i} 文本为空，使用默认文本")
            texts[i] = "<|im_start|>user\n空文本<|im_end|>\n<|im_start|>assistant\n空文本<|im_end|>"
    
    # 分词处理 输出格式 input_ids：Token ID数组 attention_mask：注意力掩码数组
    tokenized_output = tokenizer(
        texts,
        truncation=True, #当文本超过max_length时自动截断
        max_length=512,
        padding="max_length", #将所有样本填充到相同长度
        return_tensors="np"  # 使用numpy数组避免None问题
    )
    
    # 转换为Python列表以便后续处理
    tokenized_output = {k: v.tolist() for k, v in tokenized_output.items()}
#结果位{"input_ids": [[101, 234, 345, ..., 102],  # 样本1的token ID [101, 456, 789, ..., 102]   # 样本2的token ID],
#    "attention_mask": [[1, 1, 1, ..., 0],  # 样本1的掩码（1=真实token，0=填充）
#        [1, 1, 1, ..., 0]   # 样本2的掩码   ]}

    # 获取词汇表大小
    vocab_size = tokenizer.vocab_size
    
    # --- 关键修复：强制替换所有越界token ID ---
    for i in range(len(tokenized_output["input_ids"])):
        input_ids = tokenized_output["input_ids"][i]
        
        # 替换所有越界ID为UNK
        tokenized_output["input_ids"][i] = [
            idx if idx < vocab_size else tokenizer.unk_token_id
            for idx in input_ids
        ]
    
    # 初始化标签
    labels = tokenized_output["input_ids"].copy()
    
    # 查找并屏蔽非回答部分的标签 禁止添加额外特殊Token `tokenizer.encode()`会自动添加：BOS（开始符）、EOS（结束符）  定位助理回答起始位置。损失掩码准备：只计算助理回答部分的损失
    assistant_token_ids = tokenizer.encode("<|im_start|>assistant\n", add_special_tokens=False)
    
    for i in range(len(labels)):
        input_ids = tokenized_output["input_ids"][i]
        assistant_pos = -1
        
        # 查找assistant标记的位置
        for j in range(len(input_ids) - len(assistant_token_ids) + 1):
            if input_ids[j:j+len(assistant_token_ids)] == assistant_token_ids:
                assistant_pos = j
                break
        
        # 只训练assistant部分
        if assistant_pos != -1:
            # 计算需要屏蔽的长度
            mask_length = assistant_pos + len(assistant_token_ids)
            # 确保mask_length不超过序列长度
            if mask_length <= len(labels[i]):
                # 生成与屏蔽长度相同的-100列表
                mask_value = [-100] * mask_length
                # 正确赋值给切片 为什么是[:mask_length]，因为我们也要屏蔽前面用户提升部分
                labels[i][:mask_length] = mask_value
            else:
                print(f"警告: 样本 {i} 的mask_length ({mask_length}) 超过序列长度 ({len(labels[i])})")
        else:
            print(f"警告: 样本 {i} 未找到assistant标记，整个序列将被屏蔽")
            labels[i] = [-100] * len(labels[i])
    
    tokenized_output["labels"] = labels
    
    # 确保所有字段都是列表且没有None值
    for key in tokenized_output:
        for i in range(len(tokenized_output[key])):
            if any(x is None for x in tokenized_output[key][i]):
                print(f"警告: 样本 {i} 的字段 {key} 包含None值，使用0替换")
                tokenized_output[key][i] = [0 if x is None else x for x in tokenized_output[key][i]]
    
    return tokenized_output

print("开始数据分词处理...")
try:
    tokenized_dataset = formatted_dataset.map(
        tokenize_and_prepare_labels, 
        batched=True, 
        batch_size=4,  # 减小批大小
        remove_columns=["text"],  #移除text列
        load_from_cache_file=False
    )
    print("数据分词与标签化处理完毕。")
except Exception as e:
    print(f"分词过程中发生错误: {e}")
    # 尝试逐个样本处理
    print("尝试逐个样本处理...")
    tokenized_samples = []
    for i in range(len(formatted_dataset)):
        try:
            sample = formatted_dataset[i]
            # 修复这里的括号问题
            batch_data = {'text': [sample['text']]}  # 创建一个包含单个样本的批次
            tokenized_sample = tokenize_and_prepare_labels(batch_data)
            # 提取单个样本
            tokenized_samples.append({k: v[0] for k, v in tokenized_sample.items()})
        except Exception as e:
            print(f"样本 {i} 处理失败: {e}")
            # 创建空样本作为占位符
            tokenized_samples.append({
                'input_ids': [tokenizer.pad_token_id] * 512,
                'attention_mask': [1] * 512,
                'labels': [-100] * 512
            })
    
    # 从样本列表创建数据集
    tokenized_dataset = Dataset.from_dict({
        'input_ids': [s['input_ids'] for s in tokenized_samples],
        'attention_mask': [s['attention_mask'] for s in tokenized_samples],
        'labels': [s['labels'] for s in tokenized_samples]
    })
    print("已处理所有样本，创建分词后数据集")

# 打印分词后的样本检查
print("\n--- 分词后样本检查 ---")
for i in range(min(3, len(tokenized_dataset))):
    sample = tokenized_dataset[i]
    print(f"样本 {i} input_ids 长度: {len(sample['input_ids'])}")
    print(f"input_ids 前10: {sample['input_ids'][:10]}")
    print(f"labels 前10: {sample['labels'][:10]}")
    print("-" * 50)

# --- 5. 配置PEFT (LoRA) ---
print("\n正在配置LoRA...")
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM, # 任务类型：因果语言模型
    r=16, # LoRA秩 值越小参数越少
    lora_alpha=32, # LoRA缩放因子，控制低秩矩阵的缩放程度，通常设置为 r 的 2-4 倍
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"], #引用LoRA的目标模块 o_proj注意力输出投影，后面几个是FFN层的前馈网络
    lora_dropout=0.05, #防止过拟合
    bias="none",
    modules_to_save=["embed_tokens", "lm_head"],  # 需要完整训练（不应用LoRA）的模块
)
# LoRA微调模型，注入LoRA适配器
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()

# --- 6. 配置并开始训练 ---
print("\n正在配置训练参数...")
training_args = TrainingArguments(
    output_dir=output_dir,
    num_train_epochs=5, #训练轮数(完整遍历数据集的次数)
    per_device_train_batch_size=2, #每个GPU/CPU的批次大小，显存占用 ∝ batch_size × 序列长度
    gradient_accumulation_steps=8, #梯度累积步数，相当于batch_size=2*8=16
    learning_rate=2e-4,
    lr_scheduler_type="cosine", #学习率调度策略
    warmup_ratio=0.1,
    logging_steps=5,
    save_strategy="epoch", #保存策略(每轮结束时保存)
    bf16=True,
    report_to="none", #禁用外部报告
    # 添加以下参数减少内存使用
    gradient_checkpointing=True, #梯度检查点技术，牺牲计算时间换取显存
    optim="adafactor", #使用该优化器
)

# 自定义数据整理器
class SafeDataCollator:
    def __init__(self, tokenizer):
        self.tokenizer = tokenizer
        self.collator = DataCollatorForLanguageModeling(tokenizer=tokenizer, mlm=False)
    
    def __call__(self, features):
        # 确保所有字段都存在且没有None值
        processed_features = []
        for feature in features:
            processed_feature = {}
            for key in ['input_ids', 'attention_mask', 'labels']:
                # 确保键存在
                if key not in feature:
                    print(f"警告: 特征缺少 {key} 字段，使用默认值")
                    if key == 'labels':
                        processed_feature[key] = [-100] * 512
                    else:
                        processed_feature[key] = [self.tokenizer.pad_token_id] * 512
                else:
                    # 确保值不是None
                    if feature[key] is None:
                        print(f"警告: 特征 {key} 为None，使用默认值")
                        if key == 'labels':
                            processed_feature[key] = [-100] * 512
                        else:
                            processed_feature[key] = [self.tokenizer.pad_token_id] * 512
                    else:
                        # 确保是整数列表
                        processed_feature[key] = [int(x) for x in feature[key]]
            processed_features.append(processed_feature)
        
        # 使用标准整理器
        return self.collator(processed_features)

# 使用安全的数据整理器
data_collator = SafeDataCollator(tokenizer)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset,
    data_collator=data_collator,
)

print("\n--- 开始模型微调 ---")
try:
    trainer.train()
    print("--- 模型微调完成！ ---")
except Exception as e:
    print(f"训练过程中发生错误: {e}")
    print("尝试保存当前进度...")
    final_adapter_path = f"{output_dir}/interrupted_adapter"
    model.save_pretrained(final_adapter_path)
    tokenizer.save_pretrained(final_adapter_path)
    print(f"已保存中断的适配器到: {final_adapter_path}")
    raise

# --- 7. 保存与推理 ---
final_adapter_path = f"{output_dir}/final_adapter"
model.save_pretrained(final_adapter_path)
tokenizer.save_pretrained(final_adapter_path)
print(f"\n最终的LoRA适配器已保存到: {final_adapter_path}")

print("\n--- 开始推理测试，验证微调效果 ---")
del model, trainer
torch.cuda.empty_cache()

# 加载推理模型
print("加载推理模型...")
tokenizer_for_inference, base_model = prepare_tokenizer_and_model(model_path)

# 加载LoRA权重
inference_model = PeftModel.from_pretrained(base_model, final_adapter_path)
inference_model.eval()

test_prompt = """给定一句话："我们2025年春节去哈尔滨旅游怎么样？"，请你按步骤要求工作。

步骤1：识别这句话中的城市和日期共2个信息
步骤2：根据城市和日期信息，生成JSON字符串，格式为{"city":城市,"date":日期}

请问，这个JSON字符串是："""

messages = [{"role": "user", "content": test_prompt}]
input_text = tokenizer_for_inference.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
print(f"\n发送给模型的完整输入:\n{input_text}")

model_inputs = tokenizer_for_inference([input_text], return_tensors="pt").to(inference_model.device)

# 确保生成参数中的eos_token_id有效
valid_eos_token_ids = [tokenizer_for_inference.eos_token_id]
if tokenizer_for_inference.pad_token_id is not None:
    valid_eos_token_ids.append(tokenizer_for_inference.pad_token_id)

generated_ids = inference_model.generate(
    **model_inputs,
    max_new_tokens=100,
    eos_token_id=valid_eos_token_ids,
    do_sample=False
)

response_ids = generated_ids[0][model_inputs.input_ids.shape[1]:]
response = tokenizer_for_inference.decode(response_ids, skip_special_tokens=True)
print(f"\n模型的回答:\n{response.strip()}")
```
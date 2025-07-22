因为assistant的内容为空，所以训练出来的效果不好。经过一番查证，发现是处理输入数据，我们可以使用新版的
```python
messages = [
                    {"role": "system", "content": self.system_prompt},
                    {"role": "user", "content": input_text},
                    {"role": "assistant", "content": assistant_content_with_tool} # 使用包含工具调用的内容
                ]

                # 使用 apply_chat_template 格式化对话
                formatted_text = self.tokenizer.apply_chat_template(
                    messages,
                    tokenize=False,
                    add_generation_prompt=False, # 训练时通常不需要生成提示
                                   )
                                   ```
把qwen_json.py文件里面的内容修改为
```python
import pandas as pd
import torch
import re
from datasets import Dataset
from peft import LoraConfig, get_peft_model, TaskType, PeftModel
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    TrainingArguments,
    Trainer,
    DataCollatorForLanguageModeling,
)

# --- 1. 配置参数 ---
# 注意：请确认你的模型路径是正确的。这里使用一个常见的Qwen2模型作为示例。
# 你的路径可能是 'Qwen/Qwen1.5-0.5B-Chat' 或 'Qwen/Qwen2-0.5B-Instruct' 等
model_path = "/data/home/zengxiangxi/hf_model/qwen3-0.6b" 
data_file_path = "newdata.xlsx"
output_dir = "./qwen2_instruct_template_finetune"

# --- 2. 加载模型和分词器 (基本保持不变，你的函数很棒) ---
def prepare_tokenizer_and_model(model_path):
    """统一处理分词器和模型的初始化，确保配置一致"""
    print(f"正在从 '{model_path}' 加载分词器和模型...")
    tokenizer = AutoTokenizer.from_pretrained(
        model_path,
        trust_remote_code=True,
        use_fast=False, # Qwen2 推荐 use_fast=False
    )
    
    # Qwen2 的 special tokens 已经预设好，通常不需要手动添加
    # eos_token='<|im_end|>', pad_token='<|endoftext|>'
    if tokenizer.pad_token is None:
        tokenizer.pad_token = tokenizer.eos_token
        print(f"设置 pad_token 为 eos_token: {tokenizer.eos_token}")

    model = AutoModelForCausalLM.from_pretrained(
        model_path,
        trust_remote_code=True,
        device_map="auto",
        torch_dtype=torch.bfloat16,
    )
    
    # 调整模型大小以匹配分词器 (如果需要)
    if len(tokenizer) != model.config.vocab_size:
        print(f"调整模型embedding大小: {model.config.vocab_size} -> {len(tokenizer)}")
        model.resize_token_embeddings(len(tokenizer))
    
    # 确保模型的pad_token_id设置正确
    model.config.pad_token_id = tokenizer.pad_token_id
    
    print("分词器和模型加载并准备完成。")
    return tokenizer, model

tokenizer, model = prepare_tokenizer_and_model(model_path)


# --- 3. 加载和预处理数据 ---
print("正在加载和处理数据...")
df = pd.read_excel(data_file_path)
df.dropna(subset=['Prompt', 'Completion'], inplace=True)

if df.empty:
    raise ValueError("加载数据后，DataFrame 为空。")
print(f"数据加载成功，共 {len(df)} 条有效数据。")

# 数据清洗函数 (你的函数很好，保持不变)
def clean_text(text):
    if not isinstance(text, str): return ""
    cleaned = ''.join(c for c in text if ord(c) <= 0xFFFF)
    cleaned = cleaned.replace('\u200b', ' ').replace('\ufffd', '').replace('\x00', '')
    cleaned = re.sub(r'\s+', ' ', cleaned).strip()
    return cleaned

print("正在清洗数据...")
df['Prompt'] = df['Prompt'].apply(clean_text)
df['Completion'] = df['Completion'].apply(clean_text)
print("数据清洗完成。")


# --- 核心修改部分：使用 apply_chat_template ---

def create_message_list(row):
    """将DataFrame的行转换为Hugging Face聊天模板所需的messages格式"""
    return {
        "messages": [
            {"role": "user", "content": row['Prompt']},
            {"role": "assistant", "content": row['Completion']}
        ]
    }

# 将整个数据集转换为messages格式
dataset = Dataset.from_pandas(df)
formatted_dataset = dataset.map(create_message_list, remove_columns=df.columns.tolist())

# 打印样本检查
print("\n--- 格式化后样本检查 (messages格式) ---")
print(formatted_dataset[0]['messages'])
print("-" * 50)


# --- 修正后的、真正支持批处理的函数 ---

def tokenize_and_prepare_labels(examples):
    """
    一个真正的批处理版分词函数。
    它一次性处理整个批次，以保证速度和正确性。
    """
    # 1. 使用列表推导式准备好整个批次的文本数据
    # 然后一次性调用 tokenizer 进行处理，获得带填充和截断的 input_ids 和 attention_mask
    full_texts = [tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=False) for messages in examples["messages"]]
    
    full_tokenized = tokenizer(
        full_texts,
        truncation=True,
        max_length=512,
        padding="max_length",
        return_tensors="pt", # 直接返回 PyTorch tensor
    )
    input_ids = full_tokenized["input_ids"]
    attention_mask = full_tokenized["attention_mask"]
    
    # 2. 同样地，一次性准备好所有用户提问部分的文本
    # 我们不进行填充，因为我们只需要知道每个提问的原始长度
    user_prompts = [tokenizer.apply_chat_template([messages[0]], tokenize=False, add_generation_prompt=True) for messages in examples["messages"]]
    
    # 获取每个用户提问部分的 token 长度
    user_prompt_lens = [len(ids) for ids in tokenizer(user_prompts, add_special_tokens=False)["input_ids"]]

    # 3. 创建标签，并使用高效的向量化操作进行屏蔽
    # 首先，完全复制 input_ids
    labels = input_ids.clone()

    # 其次，根据每个样本的提问长度，屏蔽掉提问部分
    for i in range(len(user_prompt_lens)):
        prompt_len = user_prompt_lens[i]
        labels[i, :prompt_len] = -100  # -100 在损失计算中会被忽略

    # 最后，屏蔽掉所有填充部分（pad_token）对应的标签
    # 这是非常高效的PyTorch操作，远胜于循环
    labels[input_ids == tokenizer.pad_token_id] = -100
    
    # 返回一个字典，注意要将 tensor 转回 list
    return {
        "input_ids": input_ids.tolist(),
        "attention_mask": attention_mask.tolist(),
        "labels": labels.tolist(),
    }

print("开始数据分词与标签化处理...")
tokenized_dataset = formatted_dataset.map(
    tokenize_and_prepare_labels, 
    batched=True, 
    batch_size=10, # 可以适当调整批大小
    remove_columns=["messages"],
    load_from_cache_file=False
)
print("数据分词与标签化处理完毕。")

# 打印分词后的样本检查
print("\n--- 分词后样本检查 ---")
sample = tokenized_dataset[0]
print(f"样本 input_ids (前20): {sample['input_ids'][:20]}")
print(f"样本 labels (前40，注意-100的屏蔽部分): {sample['labels'][:40]}")
# 解码查看效果
print(f"解码后的输入: {tokenizer.decode(sample['input_ids'][:40])}")
print(f"解码后的标签 (忽略-100): {tokenizer.decode([l for l in sample['labels'][:40] if l != -100])}")
print("-" * 50)


# --- 5. 配置PEFT (LoRA) ---
print("\n正在配置LoRA...")
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
)
model = get_peft_model(model, lora_config)
if hasattr(model, "enable_input_require_grads"):
    model.enable_input_require_grads()
else:
    def make_inputs_require_grad(module, input, output):
        output.requires_grad_(True)
    model.get_input_embeddings().register_forward_hook(make_inputs_require_grad)
model.print_trainable_parameters()


# --- 6. 配置并开始训练 ---
print("\n正在配置训练参数...")
training_args = TrainingArguments(
    output_dir=output_dir,
    num_train_epochs=5,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,
    learning_rate=2e-4,
    lr_scheduler_type="cosine",
    warmup_ratio=0.1,
    logging_steps=5,
    save_strategy="epoch",
    bf16=True, # 如果你的GPU支持bf16
    report_to="none",
    gradient_checkpointing=True,
    optim="adafactor",
)

# 使用默认的数据整理器即可，因为我们已经手动完成了所有处理
data_collator = DataCollatorForLanguageModeling(tokenizer=tokenizer, mlm=False)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset,
    data_collator=data_collator,
)

print("\n--- 开始模型微调 ---")
trainer.train()
print("--- 模型微调完成！ ---")


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
base_model = AutoModelForCausalLM.from_pretrained(
    model_path,
    trust_remote_code=True,
    device_map="auto",
    torch_dtype=torch.bfloat16,
)
tokenizer_for_inference = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)

# 加载LoRA权重
inference_model = PeftModel.from_pretrained(base_model, final_adapter_path)
inference_model.eval()

# 你的测试用例
test_prompt = '想去北京看冬奥会，时间是2026年2月' # 直接用你数据集里的例子

messages = [{"role": "user", "content": test_prompt}]
# 注意：推理时 add_generation_prompt=True
input_text = tokenizer_for_inference.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
print(f"\n发送给模型的完整输入:\n{input_text}")

model_inputs = tokenizer_for_inference([input_text], return_tensors="pt").to(inference_model.device)

generated_ids = inference_model.generate(
    **model_inputs,
    max_new_tokens=100,
    eos_token_id=tokenizer_for_inference.eos_token_id,
    do_sample=False
)

response_ids = generated_ids[0][model_inputs.input_ids.shape[1]:]
response = tokenizer_for_inference.decode(response_ids, skip_special_tokens=True)
print(f"\n模型的回答:\n{response.strip()}")
# 预期输出: {"city": "北京", "date": "2026年2月"}
```
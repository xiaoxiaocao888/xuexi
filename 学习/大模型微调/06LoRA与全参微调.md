在model_lora.py文件中的类LoRA
A×B是一个全零的矩阵，即初始化的时候完全是原始的模型权重，LoRA对它是没有任何影响的。
forward方法是可以去做梯度计算和参数优化的，这里是返回AB相乘
![[Pasted image 20250605092914.png]]
apply_lora函数：对model里面的所有模块做一个for循环，把module和name提取出来，如果模型的类型是一个线性层，而且权重的第一个维度和第二个维度相等的时候，就应用到LoRA微调。`forward_with_lora`函数是在原来模型的基础上又增加了LoRA 微调
```python
def apply_lora(model, rank=16):
    for name, module in model.named_modules():
        if isinstance(module, nn.Linear) and module.weight.shape[0] == module.weight.shape[1]:
            lora = LoRA(module.weight.shape[0], module.weight.shape[1], rank=rank).to(model.device)
            setattr(module, "lora", lora)
            original_forward = module.forward
            # 显式绑定
            def forward_with_lora(x, layer1=original_forward, layer2=lora):
                return layer1(x) + layer2(x)
            module.forward = forward_with_lora
            ```

##### 对于train_lora.py文件的一些方法
**CrossEntropyLoss函数**:是PyTorch中用于​**​多分类任务​**​的核心损失函数，其核心功能是​**​衡量模型预测概率分布与真实标签之间的差异​**​。CrossEntropyLoss 结合了 `Softmax` 和 `负对数似然损失（NLLLoss）` 两步操作
- ​**​Softmax​**​：将模型输出的原始分数（logits）转换为概率分布（所有类别概率和为1）。
- ​**​NLLLoss​**​：计算真实标签对应概率的负对数，惩罚低置信度的正确预测。 ![[Pasted image 20250605095654.png]]
- 关键参数

| 参数              | 作用                                                                            | 示例                            |
| --------------- | ----------------------------------------------------------------------------- | ----------------------------- |
| weight          | 为不同类别分配权重，解决类别不平衡问题                                                           | torch.tensor([1.0, 2.0, 0.5]) |
| ignore_index    | 忽略指定类别索引的样本（如无效标签 `-100`），不参与损失计算                                             | ignore_index=-100             |
| reduction       | 控制损失输出形式：<br>- `'mean'`（默认，批损失均值）<br>- `'sum'`（批损失总和）<br>- `'none'`（返回每个样本损失） | reduction='sum'               |
| label_smoothing | 标签平滑（取值 0~1），防止模型对预测过度自信，提升泛化能力                                               | label_smoothing=0.1           |

`scaler.scale(loss).backward()`：对损失做一个反向传播，同时考虑混合精度下面的梯度缩放
`if (step + 1) % args.accumulation_steps == 0:`设置梯度累积，训练多个步骤，然后统一做一个参数优化。
`scaler.step(optimizer)`:使用step方法保存梯度
`model.eval()  model.train()`:就是将模型设置为评估模式/训练模式
`model = MiniMindLM(lm_config)`初始化预训练模型
`ckp = f'./out/rlhf_{lm_config.dim}{moe_path}.pth'`：ckp从rlhf文件里面获取，即默认是从DPO对齐的模型再去做LoRA 微调
`init_distributed_mode`分布式模式
`train_epoch`:开始做训练

##### LoRA微调和全参微调对比 train_lora.py和train_full_sft.py
1.lora微调是对lora_params参数去做梯度裁剪，全参是对模型整个参数去做裁剪
![[Pasted image 20250605102039.png]]
2.lora只保存lora的权重，全参是保存模型的所有权重
![[Pasted image 20250605102446.png]]
3.模型初始化函数里面的模型来源不同。全参是从预训练模型里面去加载的；lora微调是从rlhf，就是从人类反馈的强化学习的这样的模型来加载。在这个minimind工程里面是在做完DPO之后保存的权重。全参还多了个日志输出。
4.lora微调使用`apply_lora(model)`这个函数来决定哪些模块进行lora微调
5.进行判断，如果不是lora部分，直接把它的梯度计算设置为false
将lora相关的参数保存在lora_params里面。
lora微调只是针对lora参数去设置优化器，全参是针对整个模型去设置优化器
![[Pasted image 20250605103241.png]]
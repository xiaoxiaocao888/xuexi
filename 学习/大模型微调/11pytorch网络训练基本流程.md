首先在模型练习新建一个pytorch_model.py文件，然后我们这个网络就是弄个一层的卷积层
​`padding`参数用于控制输入特征图（如图像）边缘的填充像素数量​**​，其核心作用是​**​维持卷积操作前后特征图的空间尺寸​**​（宽高），同时​**​减少边缘信息丢失​**​。
未填充时​**​（`padding=0`）：角落像素仅被卷积核覆盖一次，导致边缘信息利用率低
**填充后​**​（`padding=1`）：边缘像素被零值填充，确保所有像素（包括边缘）参与卷积计算的次数更均衡
当`kernel_size=3`时，`padding=1`可保持特征图尺寸不变（计算公式：`output_size = (input_size + 2*padding - kernel_size)/stride + 1`）
`unsqueeze(0)`：添加批处理维度（batch_size=1）
```python
import torch
import numpy as np
class Mymodel(torch.nn.Module):
    def __init__(self,):
        super().__init__()
        self.layer=torch.nn.Conv2d(
            in_channels=3,#输入通道数(RGB图像)
            out_channels=1,#输出特征图数量(分割结果)
            kernel_size=3,#卷积核尺寸
            stride=1, #步长
            padding=1)
    def forward(self,x):
        return self.layer(x)
if __name__=="__main__":
    model=Mymodel()
    x=np.ones((3,256,256)) #这个是图片的数据，全是1说明是一张纯白的图片，什么数据都没有
    x=torch.tensor(x).unsqueeze(0).float()# 添加batch维度：[1, 3, 256, 256]
    print(x.shape) #输出torch.Size([1, 3, 256, 256]) 通道是3，图像大小是256 256
    output=model(x)
    print(output.shape) #out_channels为3输出torch.Size([1, 3, 256, 256])为1输出torch.Size([1, 1, 256, 256])
    ```
然后现在网络有了，我们就要有data,新建一个pytorch_data.py文件
当前实现将所有数据预加载到内存（`self.imgs.append(...)`），对于大型数据集建议改为：
```python
def __getitem__(self, idx):
    img_path = os.path.join(self.imgDir, self.imgNames[idx])
    return torch.from_numpy(cv2.imread(img_path)).permute(2,0,1).float()/255.0  
    ```
 实现实时加载，避免内存溢出（尤其处理高分辨率图像时）
```python
from torch.utils.data import Dataset
import cv2
import os
import numpy as np
import torch
class MyData(Dataset):
    def __init__(self,imgDir,maskDir):
        super().__init__()
        imgNames=os.listdir(imgDir)
        self.imgs,self.masks=[],[]
        for imgName in imgNames:
            # imgDir="./data/img/"
            #imgName="000.jpg"  
            #-->./data/img/000.jpg
            img=cv2.imread(imgDir+imgName) #图片1080,1920,3
            mask=cv2.imread(maskDir+imgName) #1080,1920,1
            #print(mask.shape)
            img=np.transpose(img,(2,0,1))#3,1080,1920.就是把通道3放在第一个位置
            mask=np.transpose(mask(2,0,1))
            mask=mask[0] #只保留一个通道
            self.imgs.append(torch.tensor(img).float()/255.0)
            self.masks.append(torch.tensor(mask).float()/255.0)
    def __len__(self):
        return len(self.imgs)
    def __getitem__(self, idx):
        return self.imgs[idx],self.masks[idx]
if __name__=="__main__":
    data=MyData("./data/img/","./data/mask/")
    ```
标准的dataset类就是长这样
![[Pasted image 20250605202733.png]]
一般我们要把所有的数据读到内存里面，新建文件pytorch_train.py
```python
import numpy as np
import pytorch_model
import pytorch_data
import torch
import cv2
myModel=pytorch_model.Mymodel()
trainData=pytorch_data.MyData("./data/img","./data/mask/")
trainLoader=torch.utils.data.DataLoader(trainData,batch_size=1,shuffle=True)
optimizer=torch.optim.Adam(myModel.parameters(),lr=1e-3)
lr_scheduler=torch.optim.lr_scheduler.StepLR(optimizer,step_size=10,gamma=0.1)
lossFunc=torch.nn.MSELoss() #输出要和真实的mask做比较嘛
for epoch in range(10):
    for img,mask in trainLoader:
        #img b,3,1080,1920,torch.tensor
        #mask b,1,1080,1920,torch.tensor
        optimizer.zero_grad() #清零历史梯度
        output=myModel(img) #输出
        loss=lossFunc(output,mask) #也是torch.tensor,是一个标量的，只有一个数值
        print(loss.item())
        loss.backward() #反向传播，计算当前梯度
        optimizer.step() #根据梯度更新权重
    lr_scheduler.step() #学习率下降
img=cv2.imread("0012.jpg")#1080,1920,3
img=np.transpose(img,(2,0,1))#3,1080,1920
img=torch.tensor(img).float().unsqueeze(0)/255.0 #因为多了一个维度为1的
output=myModel(img)#1,1,1080,1920
outMask=output.squeeze().detach().numpy()*255.0 #squeeze去掉维度为1的维度，detach复制(切断计算图，节省内存)，numpy数组
outMask=outMask.astype(np.uint8) #浮点数转0-255整数
cv2.imwrite("outMask.jpg",outMask)
```

PyTorch网络是基于PyTorch框架构建的深度学习模型，其核心是通过神经网络层处理数据并自动优化参数。
PyTorch网络的核心构成：神经网络层和动态计算图
神经网络层：处理输入数据的计算单元
**​参数​**​：每层包含可学习的权重（Weights）和偏置（Biases），通过训练自动调整。
- **卷积层(`nn.Conv2d`）​**​：提取图像空间特征（如边缘、纹理）
- ​**​全连接层（`nn.Linear`）​**​：连接所有输入与输出神经元，用于分类或回归
- ​**​激活函数（如`nn.ReLU`）​**​：引入非线性，使网络能拟合复杂模式（例如：`ReLU`将负数归零，保留正数）
**动态计算图（Dynamic Computational Graph）​**​
- PyTorch的​**​核心优势​**​：网络结构在代码运行时动态构建，无需预先定义完整计算图

1. **前向传播（Forward）​**​
    - 数据从输入层逐层传递至输出层，生成预测结果（如代码中的myModel(img)）
2. ​**​反向传播（Backward）与自动微分​**​
    - ​**​自动梯度计算​**​：PyTorch通过`autograd`引擎自动追踪所有运算，调用`loss.backward()`时自动计算梯度
    - ​**​优化器更新​**​：优化器（如Adam）根据梯度调整参数（`optimizer.step()`）
3. ​**​损失函数（Loss Function）​**​
    - 衡量预测值与真实值的差距，常见类型：
        - ​**​回归任务​**​：均方误差（`nn.MSELoss`）。
        - ​**​分类任务​**​：交叉熵损失（`nn.CrossEntropyLoss`）

Tensor(张量)是PyTorch中的基本数据结构，类似于NumPy数组。Module是所有神经网络模块的基类。
**数学本质**
张量通过​**​阶数​**​区分维度，可视为标量、向量、矩阵的高维扩展

| 阶数   | 名称   | 示例                         | 数据结构 |
| ---- | ---- | -------------------------- | ---- |
| 0阶   | 标量   | 温度值5.0                     | 单一数值 |
| 1阶   | 向量   | 坐标[1.2,3.4,-0.9]           | 一维数组 |
| 2阶   | 矩阵   | 图像灰度矩阵`[[0,255],[128,64]]` | 二维表格 |
| >=3阶 | 高阶张量 | RGB图像(高度×宽度×3通道)           | 多维数组 |
在PyTorch、TensorFlow等框架中，张量是​**​数据表示与计算的核心载体​**​，具有以下技术特性
1. ​**​数据类型与设备支持​**​
    - 数据类型：`float32`、`int64`等，通过 `.dtype` 查看。
    - 设备支持：CPU/GPU自动切换（`.to('cuda')`）。
2. ​**​自动微分（Autograd）​**​
    - 张量可跟踪计算历史（`requires_grad=True`），支持反向传播自动求导
3. ​**​广播机制（Broadcasting）​**​
    - 不同形状张量运算时自动扩展维度（如 `[3,1] + [1,4] → [3,4]`）
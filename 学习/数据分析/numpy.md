### 导入numpy包
import numpy
import numpy as np
from numpy import *
### 导入numpy的原因是，操作麻烦
`a=[1,2,3,4]  [x+1 for x in a]  就会变成[2,3,4,5]`
`两列表里面的数字相加a=[1,2,3,4] b=[2,3,4,5] [x+y for (x,y) in zip(a,b)] 变成[3,5,7,9]`

### 使用numpy
`a=np.array([1,2,3,4]) a+1 结果为array([2,3,4,5]) b=[2,3,4,5] a+b 结果为array([3,5,7,9])`
### 产生数组
#### 从列表产生数组
`l=[0,1,2,3] a=np.array(l) a为array([0,1,2,3])
#### 从列表传入
`a=np.array([1,2,3,4]) a为array([1,2,3,4])`
#### 生成全0 或1 的数组
` np.zeros(5) 结果为array([0.,0.,0.,0.,0.])`
`np.ones(5,dtype='int') 结果为arry([1,1,1,1,1])`
#### 使用fill方法将数组设为指定值
`a=np.array([1,2,3]) a.fill(5) a变为array([5,5,5]) 如果是a.fill(2.5) a为[2,2,2]因为它们都是整数类型。需要转换为浮点数a=a.astype('float') a.fill(2.5) a为array([2.5,2.5,2.5])`
#### 生成整数序列
`a=np.arange(1,10，2)  a为array([1,3,5,7,9]) 左闭右开，步长为2，若步长为1可省略`
#### 生成等差数列
`a=np.linspace(1,10,10)    a为array([1,2,3,4,5,6,7,8,9,10]) 第一个数字为开始值，第二个数字是可以包含的，第三个数字是数列的个数。意思就是从1到10生成有10个数字的等差数列
#### 生成随机数
`np.random.rand(10) 生成了10个随机浮点数 0-1
np.random.randn(10) 生成了10个服从正态分布的浮点数
np.random.randint(1,10,10)生成了10个从1到10的随机整数
### 数组属性
#### 查看类型
type(a)
#### 查看数组中的数据类型
a.dtype
#### 查看形状，会返回一个数组，每个元素代表这一维的元素数目
a.shape
#### 查看数组里面元素的数目
a.size
#### 查看数组的维度
a.ndim
#### 切片和列表那些一样，也支持负索引
区别：数组的切片在内存中使用的是引用机制
`a=np.array([0,1,2,3,4]) b=a[2:4] b[0]=10 a结果为array([0,1,10,3,4])`
没有为b分配空间，而是让b指向a所分配的内存空间，所以b会改变a的值
可以使用copy()方法解决这个问题,这个复制会申请新的内存`b=a[2:4].copy()`
#### 产生多维数组
`a=np.array([[1,2,3,4],[2,3,4,5]]) a[1,3]为5 a[:,1]为array([2,3]) 逗号前面为行`
#### 花式索引需要指定索引位置
`a=np.arange(0,100,10) index=[1,2,-3] y=a[index] print(y) 结果为[10 20 70]`
使用布尔数组
`mask=np.array([0,2,2,0,0,1,0,0,1,0]) a[mask]结果为array([10,20,50,80])
与切片不同，花式索引返回的是一个复制而不是引用
#### 判断数组中的元素是不是大于0
`a=np.array([0,0,3,4])  a>0 结果为array([False, False,  True,  True])`
#### 数组中大于x的元素的索引位置
`np.where(a>0) 结果为(array([2, 3], dtype=int64),)  where返回值是一个元组，返回的是索引位置
#### 返回数组大于x的元素
`a[a>0] 或者a[np.where(a>0)] 结果都是array([3, 4])`
#### 类型转换
`a=np.array([1,2,3])  np.asarray(a,dtype=float) 结果为array([1., 2., 3.])`
`a=np.array([1,5,-3],dtype=float) a为array([ 1.,  5., -3.])`
`a=np.array([1,2,3])
`a.astype(float) 结果为array([1., 2., 3.])但是a任然为array([1, 2, 3]) astype方法返回的是一个新数组`
### 数组操作
#### 数组排序
`a=np.array([4,2,3,7,1])  np.sort(a) 返回array([1, 2, 3, 4, 7]) a任然为array([4, 2, 3, 7, 1])`
`order=np.argsort(a)  order为array([4, 1, 2, 0, 3], dtype=int64) argsort返回从小到大的排列在数组中的索引位置`
#### 数组求和 最值 均值 标准差 相关系数矩阵
`np.sum(a)或a.sum() 结果为17  max min mean均值 std标准差同理`
`对于一维数组，cov返回的是方差 如果是二维数组，返回的是i行j列的协方差矩阵，其中每个元素（i，j）是a中第i列和第j列之间的协方差`
#### 数组形状
`a=np.arange(6) a.shape=2,3 a为array([[0, 1, 2],[3, 4, 5]]) a.shape结果为(2, 3)   第一个shape是把数组变为2行3列
reshape方法是不会修改原来的值，而是返回一个新的数组
转置
a.T 或 a.transpose()
#### 数组连接
`x=np.array([[0,1,2],[10,11,12]])
`y=np.array([[50,51,52],[60,61,62]])
`z=np.concatenate((x,y))
`z结果为array([[ 0,  1,  2], [10, 11, 12],  [50, 51, 52], [60, 61, 62]]) 默认为横向连接 也可以使用np.vstack((x,y))
`如果是z=np.concatenate((x,y),axis=1) 结果为array([[ 0,  1,  2, 50, 51, 52],  [10, 11, 12, 60, 61, 62]]) 纵向连接 也可以使用np.hstack((x,y))
改变维度
`np.array((x,y)) 结果为array([[[ 0,  1,  2],  [10, 11, 12]],[[50, 51, 52],[60, 61, 62]]])
`np.dstack((x,y)) 结果为array([[[ 0,  1,  2], [10, 11, 12]],[[50, 51, 52],[60, 61, 62]]])
numpy内置函数
abs exp指数 中值median 
a=np.array([2,-1,3,4]) np.cumsum(a) 结果为array([2, 1, 4, 8]) 求累加和，第一个数字，第一个和第二个数字之和，前三个数字之和，等等。

np.nan是空置
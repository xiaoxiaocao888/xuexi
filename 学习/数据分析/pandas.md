#### 导入包
import pandas as pd
import numpy as np
#### 两种基本类型
##### series
一维数组，与numpy中的一维数组类似。series能保存不同种数据类型
初始化：`a=pd.Series([1,3,5,np.nan,6,8]) print(s) 结果为
0    1.0
1    3.0
2    5.0
3    NaN
4    6.0
5    8.0
dtype: float64
`s=pd.Series([1,3,5,np.nan,6,8],index=['a','b','c','d','e','f']) print(s) 结果为
a    1.0
b    3.0
c    5.0
d    NaN
e    6.0
f    8.0
dtype: float64
索引：`a.index 结果为RangeIndex(start=0, stop=6, step=1)
`a.values结果为array([ 1.,  3.,  5., nan,  6.,  8.]) a[0]为1.0
切片操作与数组的一样
索引赋值
`a.index.name='索引' a变为
索引
0    1.0
1    3.0
2    5.0
3    NaN
4    6.0
5    8.0
dtype: float64
`a.index=list('abcdef') a和上面的s一样
`a['a':'b'] 结果为
a    1.0
b    3.0
dtype: float64
以字符作为索引下标的，左右都是闭区间
##### DataFrame
二维的表格型数据结构。与excel类型
eg:首先构造一组时间序列，作为我们第一维的下标（转换成时间格式的语法：`pd.to_datetime(df['列名'])`
`date=pd.date_range('20180101',periods=6) print(date) 结果为DatetimeIndex(['2018-01-01', '2018-01-02', '2018-01-03', '2018-01-04',  '2018-01-05', '2018-01-06'],dtype='datetime64[ns]', freq='D')
然后创建一个DataFrame结构：
`df=pd.DataFrame(np.random.randn(6,4),index=date,columns=list('ABCD')) df结果为
                             A                     B                      C                         D
2018-01-01      1.428573        -2.057036       0.567368            -0.672844
2018-01-02      -0.875757       1.278290        -0.931197          0.898402
2018-01-03      1.796392         -0.480099      -0.815437          0.753969 2018-01-04      0.949939         0.687148        0.873199          0.522366 2018-01-05       1.239382        1.004287        -0.168719         -0.778457 2018-01-06       -1.787429       -1.076466       0.086963         -1.242108
默认情况下，如果不指定index参数和columns，那么他们的值将用从0开始的数字替代
除了向DataFrame中传入二维数组，我们也可以使用字典传入数据：
`df2=pd.DataFrame({'A':1,'B':pd.Timestamp('20181001'),'C':pd.Series(1,index=list(range(4)),dtype=float),'D':np.array([3]*4,dtype=int),'E':pd.Categorical(["test","train","test","train"]),'F':'abc'}) df2为
   A        B                C   D     E         F
0  1  2018-10-01   1.0   3   test    abc  
1  1  2018-10-01   1.0   3   train   abc
2  1  2018-10-01   1.0   3   test    abc
3  1  2018-10-01   1.0   3   train   abc
字典的每个key代表一列，其value可以是各种能够转化为Series的对象。DataFrame只要求每一列数据的格式相同
查看数据的 前几行和后几行
df.head()  df.tail(3)看最后3行   都默认为前/后五行
下标使用index属性查看
列标使用columns属性查看
数据使用values查看
#### 读取数据
df=pd.read_excel(r'c:路径')
读行操作：`df.iloc[0:2]  打印出前两行的数据 df.loc[0:2]前三行
添加一行：eg:`dit=['名称':'复仇者联盟','投票人数':12345]  s=pd.Series(dit) s.name=3 df=df.append(s) name的属性是索引还有Series都有的 上面那个相当于他的下标
删除一行：`df=df.drop(x)  x为下标
列操作：`df.columns 结果为列名称  df['名称'][:5] 结果为名称那列的前五行数据  df['名称','类型'][:5] 显示名称和类型两列的前五行
增加一列：`df['序号']=range(1,len(df)+1) 
删除一列：df=df.drop('序号',axis=1) 删除序号那列，axis=1表示纵向，=0是默认横向
通过标签选择数据：`df.loc[[index],[colunm]]
`eg:df.loc[1,'名称'] 显示第一行，名称那列的值  df.loc[[1,3,5],['名称'，'类型']]显示1 3 5行名称 类型的值
条件选择
eg:·`df[df['场地']=='美国'] 显示出场地是美国的数据
`df[(df.场地='美国') &(df.评分>9)] 或用|
#### 缺失值及异常值处理
dropna  根据标签中的缺失值进行过滤，删除缺失值
fillna       对缺失值进行填充
isnull      判断哪些值是缺失值，返回一个布尔值对象
notnull   isnull的否定式
判断缺失值：pd.isnull(df) 或df.isnull()
`df[df['名字'].isnull()][:10]显示前10个名字为空的数据
填充缺失值：`df[df[‘评分’].isnull()][:10]   df['评分'].fillna(np.mean(df['评分']),inplace=True) 直接在原始数据上进行修改
删除缺失值：df.dropna()
参数： how='all' 删除全为空值的行或列  inplace=True 覆盖之前的数据 axis=0 选择行或列
处理异常值：找到并删除或修改 `df=df[df.投票人数>0] 
数据保存：`df.to_excel('路径')
#### 数据格式转换
查看格式：eg:`df['投票人数'].dtype`
修改格式：eg:`df['投票人数']=df['投票人数'].astype('int')`
#### 数据排序
`df.sort_values(by='投票人数',ascending=False)[:5] False表示降序 True表示升序，是默认值  多值排序by=['评分','投票人数'] 就是先按评分排序再按投票人数排序
### 基本统计分析
#### 描述统计
dataframe.describe()对dataframe中的数值型数据进行描述统计,即显示出各数值型数据的总数 平均值 标准差 最小值 四分位数 最大值
通过描述性统计，可以发现一些异常值
`df['投票人数'].max() 
max min mean median中值 方差var 标准差std  sum 相关系数corr 协方差cov
查看数据的多少种类型unique  计数` len(df['产地'].unique())
数据替换 `df['产地'].replace('USA','美国',inplace=True) 在原始数据上将USA改为美国
计算每一年电影的数量`df['年代'].value_counts() 结果是降序
### 数据透视 pivot_table
`pd.pivot_table(df,index=["年代"]) 以年份作为索引，显示数值型属性
`在前面加上这两段代码pd.set_option('max_columns'.100) pd.set_option('max_rows'.500) 让数据显示完整，不会出现那个省略号
`多个索引 index=["年份","产地"]
指定需要统计汇总的数据` pivot_table=pd.pivot_table(df,index=['年代',"产地"],values=["评分"]) 出现的数据只有年代产地评分
指定函数，来统计不同的统计量`pd.pivot_table(df,index=["年代","产地"],values=["投票人数"],aggfunc=np.sum)
通过对”投票人数“列和”评分“列进行对应分组，对”产地“实现数据聚合和总结
`pd.pivot_table(df,index=["产地"],values=["投票人数","评分"],aggfunc=[np.sum,np.mean])
将非数值NaN设置为0
`index=["产地"],values=["投票人数","评分"],aggfunc=[np.sum,np.mean],fill_value=0)
在后面加上,margins=True 可以在下方显示一些总和数据
对不同的值执行不同的函数：可以向aggfunc传递一个字典，副作用就是将标签做的更加简洁才行 `pd.pivot_table(df,index=["产地"],values=["投票人数","评分"],aggfunc={"投票人数":np.sum,"评分":np.mean},fill_value=0)
`table=pd.pivot_table(df,index=["产地"],values=["投票人数","评分"],aggfunc={"投票人数":np.sum,"评分":np.mean},fill_value=0) type(table)结果为pandas.core.frame.DataFrame
table.sort_values("评分",ascending=False)按照评分降序排列

### 数据重塑和轴线旋转
#### 层次化索引
Series：`s=pd.Series(np.arange(1,10),index=[['a','a','a','b','b','c','c','d','d'],[1,2,3,1,2,3,1,2,3]]) s为
a  1    1
   2    2
   3    3
b  1    4
   2    5
c  3    6
   1    7
d  2    8
   3    9
dtype: int32
将s Series转化为DataFrame:  s.unstack()
再转回来s.unstack().stack()
DataFrame:行和列都可以进行层次化索引
`data=pd.DataFrame(np.arange(12).reshape(4,3),index=[['a','a','b','b'],[1,2,1,2]],columns=[['A','A','B'],['Z','X','C']])           data为
                           A           B
                  Z         X          C
    a    1      0         1          2
          2      3         4          5
    b    1      6         7          8
          2      9        10        11
设置行 列级别名称` data.index.names=['row1','row2'] data.columns.names=['col1','col2']  结果为
                col1            A      B
                col2    Z      X       C
row1        row2
a                 1        0     1       2
                  2       3     4        5
b                1        6      7       8
                2         9     10      11
row1 row2调换位置：data.swaplevel('row1','row2')
set_index可以把列变成索引  `df=df.set_index(['产地','年代']) 每一个索引都是一个元组df.index[0] 结果为('美国', 1994) 读取产地是美国df.loc['美国']
reset_index是把索引变成列 `df=df.reset_index() 结果变为之前的那样
#### 数据旋转
行和列转换data.T
转化为层次化索引Series， data.stack()   变回来data.stack().unstack()
#### 数据分组，分组计算(只对数值变量)
按照电影产地进行分组
先定义一个分组变量`group=df.groupby(df['产地'])
计算每年的平均评分`df['评分'].groupby(df['年代']).mean()
将年代转化为字符串`df['年代']=df['年代'].astype('str')
传入多个分组变量`df['评分'].groupby([df['产地'],df['年代']]).mean()
### 离散化操作
离散化也称为分组、区间化
pd.cut(x,bins,right=True,labels=None,retbins=False,precision=3,include_lowest=False)
x:需要离散化的数组、Series、DataFrame对象    bins:分组的依据   默认左开右闭
`pd.cut(df['评分'],[0,3,5,7,9,10],labels=['E','D','C','B','A']) 0-3分为E
对bins进行赋值`bins=np.percentile(df['投票人数'],[0,20,40,60,80,100])
合并数据集
1.append
`df_usa=df[df.产地=='美国']  df_china=df[df.产地=='中国大陆'] df_china.append(df_usa)
2.merge  可做横向拼接
`pd.merge(left,right,how='inner',on=None,left_on=None,right_on=None,left_index=False,right_index=False,sort=True,suffixes=('_x','_y'),copy=True,indicator=False)
left right 为对象
on要加入的列（名称），必须在左右综合对象中找到。如果不能通过left_index=False,right_index=False，将推断DataFrame中的列的交叉点为连接键
left_on:从左边找键列
right_on:从正确的综合找纵列
how:左连接left，右连接right，内连接就是共同的
` df1=df.loc[:5] df2=df.loc[:5][['名字','产地']] df2['票房']=[123444,2345,444,5555,333,222] df2=df2.sample(frac=1) df2.index=range(len(df2))
`pd.merge(df1,df2,how='inner',on='名字')`   frac那里是把顺序打乱
3.concat
`df1=df[:10]  df2=df[100:120]   df3=df[200:210]   pd.concat([df1,df2,df3]) 批量合并
默认是横向合并，想纵向合并axis=1
Dify是一个开源的LLM应用开发平台。

|      | 推理模型                               | 非推理模型                               |
| ---- | ---------------------------------- | ----------------------------------- |
| 区别   | 推理能力强，输出包含思考过程，推理过程复杂，易产生错误        | 纯数据驱动，基于大规模语言模型训练。直接给出答案            |
| 使用场景 | 复杂计算、代码生成、复杂文档分析                   | 基础事实回答或语言理解、代码注释、文本摘要/问答/分类/提取，生成文案 |
| 模型名称 | DeepSeek-R1、GPT-4o、Claude 3、Gemini | GPT-3、BERT、OPT、LLaMA-2、DeepSeek-v3  |
**文本分段**：
- 适配模型的输入限制。有数字限制，分段的小文本能更好的适配模型输入要求，还要兼顾到这个向量化嵌入的效率，分段后的短文本生成的向量，聚焦于单一语义，可以提升相似性检索的精准度，
- 提升语义的相关性，避免信息的稀碎
- 优化检索的性能。

**索引方式**：
- 嵌入模型索引知识：支持语义级别的检索，可以搜索到相近意思的词语
- 通过关键字索引知识

可以去reddit社区看什么场景下使用嵌入模型更好。
https://www.reddit.com/r/LocalLLaMA/comments/18j39qt/what_embedding_models_are_you_using_for_rag/?rdt=35958
嵌入式模型排行：
https://huggingface.co/spaces/mteb/leaderboard
DeepWiki:https://deepwiki.com/
将github上的开源项目生成结构化(将对应的github项目地址放到搜索栏)，交互式的文档与知识图谱，可以快速理解复杂代码库的核心逻辑和架构。核心功能有一键生成项目文档。这里面还有对该项目的对话功能
![[Pasted image 20250529110836.png]]

使用数据库流程就是：工作流--python程序--MYSQL
拉取镜像
```
docker pull mysql:5.7
```
启动MYSQL数据库
```
docker run --name mysql5.7 -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:5.7
```
登录mysql
```
docker exec -it mysql5.7 mysql -u root -p
```
输入密码进入命令行
```
create database lab;
use lab;
create table student(
id int primary key auto_increment,
name varchar(50) not null
);
insert into student (name) values('Alice'),('Bob'),('Charlie');
select * from student;
```
准备后台程序环境
```
conda create -n dify-mysql python=3.10
conda activate dify-mysql
pip install pymysql flask request
```
编写后台程序编码，就是与数据库进行连接，和要进行的连接操作
编写dify工作流，开始节点--代码--结束节点。其中代码节点就是去调用后台程序的代码
个人觉得这种调用还是不好的，每次使用场景不一样，倒不如直接在数据库里面进行命令行执行了
大模型节点的system是模型的角色定位，点击添加信息就会有user面板，就是我们用户要大模型做的事情

自然语言转SQL，github上面可以deepwiki.com/jaguarliuu/rookie_text2data
了解这个项目
在dify上面安装该插件
点击插件--点击探索Marketplace--搜索text2data--进行安装
工作流：开始--工具：rookie_text2datal(即根据自然语言生成sql)--rookie_excute_sql（执行sql)--结束

工具的配置里面返回数据格式选择TEXT，数据库IP/域名填写host.docker.internal
模型要选择非推理模型，如deepseek-r1
在生成SQL节点添加自定义提升：生成sql语句过程中，不要自动给表添加schema等前缀
使用langchain 搭建rag是比较简单的，里面使用开源框架来做的。
直接检索的问题有：无效信息多，越多对大模型后续的推理影响越大；存在最大token限制。
解决：把存放着原始数据的知识库(knowledge)中的每一个raw data，切分成一个一个的小块，这些小块可以是一个段落，也可以是数据库中某个索引对应的值。该过程称为"分块"。我们最终得到一个充满块的数据(chunks)的新的知识库(repository)。当再接收到query，就会在新知识库里面查找，返回一个或多个特定的chunk，信息量就更小且更精确。然后这些chunk被加入到prompt中，作为上下文信息与用于原始query共同输入到大模型进行处理，生成最终的回答。

在上述将原始数据（raw data）转化为chunk的过程中，就包含**构建RAG**的第一部分开发工作：
做数据清洗，如去除停用词、标点符号等。和如何选择合适的`split`方法来进行数据切分的一系列技术。
在NLP中去计算文本相似度的有效的方法就是Embedding，即将这些chunks转换成向量（vector）形式。
构建RAG的第三部分工作：我们要去了解和学习如何选择、使用向量数据库。（解决搜索效率和计算相似度优化算法）
#### 一个基础的RAG架构会只要包含以下几方面的开发工作：
1. 如何将原始数据转化成chunks；
2. 如何将chunks转化成Vector；
3. 如何选择计算向量相似度的算法；
4. 如何利用向量数据库提升搜索效率；
5. 如何把找到的chunks与原始query拼接在一起，产生最终的Prompt；

![[Pasted image 20250703104249.png]]

##### 支持 PDF 文件解析的智能RAG 问答
我们通过 `Streamlit` 前端界面，结合 `LangChain` 框架 与 `DashScope` 向量嵌入服务，实现了一个轻量化的 `RAG（Retrieval-Augmented Generation）` 智能问答系统，支持上传多个 `PDF` 文档，系统将自动完成文本提取、分块、向量化，并构建基于 `FAISS` 的检索数据库。用户随后可以在页面中输入任意问题，系统会调用大语言模型（如 `DeepSeek-Chat`）对 `PDF` 内容进行语义理解和回答生成。

```Python
pip install streamlit PyPDF2 dashscope faiss-cpu
```
要获取DASHSCOPE_API_KEY=xxx，写入.env
能够实现：
- **LangChain 的多模块能力**（向量搜索 + Agent工具）
- **Streamlit 前端交互**
- **FAISS 向量数据库**
- **DashScope Embedding + DeepSeek 模型接入**
- 并完成了完整的 RAG（检索增强生成）流程

 `PyPDF2` 用来读取 PDF 文本。
 设置 `KMP_DUPLICATE_LIB_OK` 避免某些 MKL 多线程报错。
 用阿里云 DashScope 提供的 `text-embedding-v1` 将文本转为向量表示，用于相似度搜索。
`pdf_read`：逐页读取 PDF 内容并拼接。
`get_chunks`：将长文本切片为多个段落（chunk），每段 1000 字，重叠 200 字。(避免一些重要信息被截断)
`vector_store`：用 FAISS 建立向量索引，并保存到本地 `faiss_db/`。
`user_input`: 加载本地 FAISS 向量库； 将其转为 LangChain 的检索工具； 交由 Agent 调用完成回答。
主函数：页面标题与界面配置。`st.columns` 分栏：左边显示提示，右边放置“清空数据库”按钮。主输入框：`st.text_input("请输入问题")` 只有当数据库存在时才能提问。    
侧边栏： PDF 上传器；提交按钮（处理上传的 PDF → 分片 → 向量化 → 存储）。
提交PDF后执行的逻辑：if process_button:- 当点击“提交并处理”后：
读取上传的 PDF； 切片文本；向量化入库；弹出气球提示，并 `st.rerun()` 刷新页面状态。
##### LangChain RAG核心功能：
- 使用 `st.file_uploader()` 组件支持多文件上传，并通过 `PyPDF2.PdfReader` 对每页内容进行提取，组合为整体文本。
- 使用 `RecursiveCharacterTextSplitter` 将长文档切割为固定长度（1000字）+ 重叠（200字）的小块，将文本块通过 `DashScopeEmbeddings` 嵌入为向量，使用 `FAISS` 本地存储向量数据库。
- 通过 `Streamlit` 获取用户输入问题，如果向量数据库存在，则加载 `FAISS` 检索器，使用 `create_retriever_tool()` 构建 `LangChain` 工具，交由 `AgentExecutor` 执行，自动调用检索器并生成答案。
运行命令：streamlit run d:/LangChainlianxi/12rag.py
若是没有出现地址，可以按enter键
![[Pasted image 20250703120357.png]]
FAISS词向量知识库用来存储长文档切成短文档的最终结果
retriever是可以把rag系统里面检索部分的功能封装成工具，这个工具就可以放到agent里面
```Python
    import streamlit as st
    from PyPDF2 import PdfReader
    from langchain.text_splitter import RecursiveCharacterTextSplitter
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_community.vectorstores import FAISS
    from langchain.tools.retriever import create_retriever_tool
    from langchain.agents import AgentExecutor, create_tool_calling_agent
    from langchain_community.embeddings import DashScopeEmbeddings
    from langchain.chat_models import init_chat_model
    import os
    from dotenv import load_dotenv 
    load_dotenv(override=True)


    DeepSeek_API_KEY = os.getenv("DEEPSEEK_API_KEY")
    dashscope_api_key = os.getenv("dashscope_api_key")

    os.environ["KMP_DUPLICATE_LIB_OK"]="TRUE"


    embeddings = DashScopeEmbeddings(
        model="text-embedding-v1", dashscope_api_key=dashscope_api_key
    )

    def pdf_read(pdf_doc):
        text = ""
        for pdf in pdf_doc:
            pdf_reader = PdfReader(pdf)
            for page in pdf_reader.pages:
                text += page.extract_text()
        return text


    def get_chunks(text):
        text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
        chunks = text_splitter.split_text(text)
        return chunks

    def vector_store(text_chunks):
        vector_store = FAISS.from_texts(text_chunks, embedding=embeddings)
        vector_store.save_local("faiss_db")

    def get_conversational_chain(tools, ques):
        llm = init_chat_model("deepseek-chat", model_provider="deepseek")
        prompt = ChatPromptTemplate.from_messages([
            (
                "system",
                """你是AI助手，请根据提供的上下文回答问题，确保提供所有细节，如果答案不在上下文中，请说"答案不在上下文中"，不要提供错误的答案""",
            ),
            ("placeholder", "{chat_history}"),
            ("human", "{input}"),
            ("placeholder", "{agent_scratchpad}"),
        ])
        
        tool = [tools]
        agent = create_tool_calling_agent(llm, tool, prompt)
        agent_executor = AgentExecutor(agent=agent, tools=tool, verbose=True)
        
        response = agent_executor.invoke({"input": ques})
        print(response)
        st.write("🤖 回答: ", response['output'])

    def check_database_exists():
        """检查FAISS数据库是否存在"""
        return os.path.exists("faiss_db") and os.path.exists("faiss_db/index.faiss")

    def user_input(user_question):
        # 检查数据库是否存在
        if not check_database_exists():
            st.error("❌ 请先上传PDF文件并点击'Submit & Process'按钮来处理文档！")
            st.info("💡 步骤：1️⃣ 上传PDF → 2️⃣ 点击处理 → 3️⃣ 开始提问")
            return
        
        try:
            # 加载FAISS数据库
            new_db = FAISS.load_local("faiss_db", embeddings, allow_dangerous_deserialization=True)
            
            retriever = new_db.as_retriever()
            retrieval_chain = create_retriever_tool(retriever, "pdf_extractor", "This tool is to give answer to queries from the pdf")
            get_conversational_chain(retrieval_chain, user_question)
            
        except Exception as e:
            st.error(f"❌ 加载数据库时出错: {str(e)}")
            st.info("请重新处理PDF文件")

    def main():
        st.set_page_config("🤖 LangChain B站公开课 By九天Hector")
        st.header("🤖 LangChain B站公开课 By九天Hector")
        
        # 显示数据库状态
        col1, col2 = st.columns([3, 1])
        
        with col1:
            if check_database_exists():
            pass
            else:
                st.warning("⚠️ 请先上传并处理PDF文件")
        
        with col2:
            if st.button("🗑️ 清除数据库"):
                try:
                    import shutil
                    if os.path.exists("faiss_db"):
                        shutil.rmtree("faiss_db")
                    st.success("数据库已清除")
                    st.rerun()
                except Exception as e:
                    st.error(f"清除失败: {e}")

        # 用户问题输入
        user_question = st.text_input("💬 请输入问题", 
                                    placeholder="例如：这个文档的主要内容是什么？",
                                    disabled=not check_database_exists())

        if user_question:
            if check_database_exists():
                with st.spinner("🤔 AI正在分析文档..."):
                    user_input(user_question)
            else:
                st.error("❌ 请先上传并处理PDF文件！")

        # 侧边栏
        with st.sidebar:
            st.title("📁 文档管理")
            
            # 显示当前状态
            if check_database_exists():
                st.success("✅ 数据库状态：已就绪")
            else:
                st.info("📝 状态：等待上传PDF")
            
            st.markdown("---")
            
            # 文件上传
            pdf_doc = st.file_uploader(
                "📎 上传PDF文件", 
                accept_multiple_files=True,
                type=['pdf'],
                help="支持上传多个PDF文件"
            )
            
            if pdf_doc:
                st.info(f"📄 已选择 {len(pdf_doc)} 个文件")
                for i, pdf in enumerate(pdf_doc, 1):
                    st.write(f"{i}. {pdf.name}")
            
            # 处理按钮
            process_button = st.button(
                "🚀 提交并处理", 
                disabled=not pdf_doc,
                use_container_width=True
            )
            
            if process_button:
                if pdf_doc:
                    with st.spinner("📊 正在处理PDF文件..."):
                        try:
                            # 读取PDF内容
                            raw_text = pdf_read(pdf_doc)
                            
                            if not raw_text.strip():
                                st.error("❌ 无法从PDF中提取文本，请检查文件是否有效")
                                return
                            
                            # 分割文本
                            text_chunks = get_chunks(raw_text)
                            st.info(f"📝 文本已分割为 {len(text_chunks)} 个片段")
                            
                            # 创建向量数据库
                            vector_store(text_chunks)
                            
                            st.success("✅ PDF处理完成！现在可以开始提问了")
                            st.balloons()
                            st.rerun()
                            
                        except Exception as e:
                            st.error(f"❌ 处理PDF时出错: {str(e)}")
                else:
                    st.warning("⚠️ 请先选择PDF文件")
            
            # 使用说明
            with st.expander("💡 使用说明"):
                st.markdown("""
                **步骤：**
                1. 📎 上传一个或多个PDF文件
                2. 🚀 点击"Submit & Process"处理文档
                3. 💬 在主页面输入您的问题
                4. 🤖 AI将基于PDF内容回答问题
                
                **提示：**
                - 支持多个PDF文件同时上传
                - 处理大文件可能需要一些时间
                - 可以随时清除数据库重新开始
                """)

    if __name__ == "__main__":
        main()
```
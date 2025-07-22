这里介绍的是使用tavily，进行下载
```bash
pip install -qU langchain-tavily -i https://pypi.tuna.tsinghua.edu.cn/simple
```
去该网站注册并登录，获取API KEY https://app.tavily.com/home
代码
```Python
import os
from dotenv import load_dotenv 
load_dotenv(override=True)
from langchain_community.tools.tavily_search import TavilySearchResults
search = TavilySearchResults(max_results=2)
print(search.invoke("苹果2025WWDC发布会"))

from langchain.agents import AgentExecutor, create_tool_calling_agent, tool
from langchain_core.prompts import ChatPromptTemplate
from langchain.chat_models import init_chat_model

tools = [search]
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "你是一名助人为乐的助手，并且可以调用工具进行网络搜索，获取实时信息。"),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)

# 初始化模型
model = init_chat_model("deepseek-chat", model_provider="deepseek")
agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
print(agent_executor.invoke({"input": "请问苹果2025WWDC发布会召开的时间是？"}))

```
![[Pasted image 20250702152223.png]]

推荐的Agent创建函数有

| 函数名                                       | 功能描述            | 适用场景       |
| ----------------------------------------- | --------------- | ---------- |
| **create_tool_calling_agent**             | 创建使用工具的Agent    | 通用工具调用     |
| **create_openai_tools_agent**             | 创建OpenAI工具Agent | OpenAI模型专用 |
| **create_openai_functions_agent**         | 创建OpenAI函数Agent | OpenAI函数调用 |
| **create_react_agent**                    | 创建ReAct推理Agent  | 推理+行动模式    |
| **create_structured_chat_agent**          | 创建结构化聊天Agent    | 多输入工具支持    |
| **create_conversational_retrieval_agent** | 创建对话检索Agent     | 检索增强对话     |
| **create_json_chat_agent**                | 创建JSON聊天Agent   | JSON格式交互   |
| **create_xml_agent**                      | 创建XML格式Agent    | XML逻辑格式    |
| **create_self_ask_with_search_agent**     | 创建自问自答搜索Agent   | 自主搜索推理     |
##### 使用`create_openai_tools_agent`来实际开发一个浏览器自动化代理
```bash
pip install playwright lxml langchain_community beautifulsoup4 reportlab
playwright install
```
安装过程是下载并安装 Playwright 支持的浏览器内核（注意：这里不是用我们本机已有的浏览器），包括`Chromium`（类似 Chrome）、`Firefox`、`WebKit`（类似 Safari），并将这些浏览器下载到本地的 `.cache/ms-playwright` 目录或项目的 `~/.playwright` 目录中，以便 Playwright 使用稳定一致的运行环境。
用代理工具初始化同步 `Playwright` 浏览器
```Python
    from langchain_community.agent_toolkits import PlayWrightBrowserToolkit
    from langchain_community.tools.playwright.utils import create_sync_playwright_browser
    from langchain import hub
    from langchain.agents import AgentExecutor, create_openai_tools_agent
    from langchain.chat_models import init_chat_model
    import os
    from dotenv import load_dotenv 
    load_dotenv(override=True)

    DeepSeek_API_KEY = os.getenv("DEEPSEEK_API_KEY")
    # print(DeepSeek_API_KEY)  # 可以通过打印查看
    # 初始化 Playwright 浏览器：
    sync_browser = create_sync_playwright_browser()
    toolkit = PlayWrightBrowserToolkit.from_browser(sync_browser=sync_browser)
    tools = toolkit.get_tools()
    # 通过 LangChain Hub 拉取提示词模版
    prompt = hub.pull("hwchase17/openai-tools-agent")
    # 初始化模型
    model = init_chat_model("deepseek-chat", model_provider="deepseek")
    # 通过 LangChain 创建 OpenAI 工具代理
    agent = create_openai_tools_agent(model, tools, prompt)
    # 通过 AgentExecutor 执行代理
    agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

    if __name__ == "__main__":
        # 定义任务
        command = {
            "input": "访问这个网站 https://github.com/fufankeji/MateGen/blob/main/README_zh.md 并帮我总结一下这个网站的内容"
        }
        # 执行任务
        response = agent_executor.invoke(command)
        print(response)
```
![[Pasted image 20250702154342.png]]

将`Playwright Agent`封装成工具函数，并结合`LangChain`的`LCEL`串行链，实现一个更加复杂的浏览器自动化代理。简单的串行链由`Playwright Agent`和 `generate_pdf Agent`组成，即先爬取网页的内容，然后将网页中的内容写入到本地的`PDF`文件中。再定一个摘要工具，在使用`Playwright`工具访问网页后，根据爬取到的网页内容先使用大模型进行摘要总结，再调用`generate_pdf`工具将总结内容写入到本地的`PDF`文件中
![[Pasted image 20250702155931.png]]
```Python
    from langchain_community.agent_toolkits import PlayWrightBrowserToolkit
    from langchain_community.tools.playwright.utils import create_sync_playwright_browser
    from langchain import hub
    from langchain.agents import AgentExecutor, create_openai_tools_agent
    from langchain.chat_models import init_chat_model
    from langchain_core.tools import tool
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_core.output_parsers import StrOutputParser
    from reportlab.lib.pagesizes import letter, A4
    from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
    from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
    from reportlab.lib.enums import TA_JUSTIFY, TA_CENTER
    from reportlab.pdfbase import pdfmetrics
    from reportlab.pdfbase.ttfonts import TTFont
    import os
    from datetime import datetime
    import os
    from dotenv import load_dotenv 
    load_dotenv(override=True)


    DeepSeek_API_KEY = os.getenv("DEEPSEEK_API_KEY")

    # 1. 创建网站总结工具
    @tool
    def summarize_website(url: str) -> str:
        """访问指定网站并返回内容总结"""
        try:
            # 创建浏览器实例
            sync_browser = create_sync_playwright_browser()
            toolkit = PlayWrightBrowserToolkit.from_browser(sync_browser=sync_browser)
            tools = toolkit.get_tools()
            
            # 初始化模型和Agent
            model = init_chat_model("deepseek-chat", model_provider="deepseek")
            prompt = hub.pull("hwchase17/openai-tools-agent")
            agent = create_openai_tools_agent(model, tools, prompt)
            agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=False)
            
            # 执行总结任务
            command = {
                "input": f"访问这个网站 {url} 并帮我详细总结一下这个网站的内容，包括主要功能、特点和使用方法"
            }
            
            result = agent_executor.invoke(command)
            return result.get("output", "无法获取网站内容总结")
            
        except Exception as e:
            return f"网站访问失败: {str(e)}"

    # 2. 创建PDF生成工具
    @tool  
    def generate_pdf(content: str) -> str:
        """将文本内容生成为PDF文件"""
        try:
            # 生成文件名（带时间戳）
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filename = f"website_summary_{timestamp}.pdf"
            
            # 创建PDF文档
            doc = SimpleDocTemplate(filename, pagesize=A4)
            styles = getSampleStyleSheet()
            
            # 注册中文字体（如果系统有的话）
            try:
                # Windows 系统字体路径
                font_paths = [
                    "C:/Windows/Fonts/simhei.ttf",  # 黑体
                    "C:/Windows/Fonts/simsun.ttc",  # 宋体
                    "C:/Windows/Fonts/msyh.ttc",    # 微软雅黑
                ]
                
                chinese_font_registered = False
                for font_path in font_paths:
                    if os.path.exists(font_path):
                        try:
                            pdfmetrics.registerFont(TTFont('ChineseFont', font_path))
                            chinese_font_registered = True
                            print(f"✅ 成功注册中文字体: {font_path}")
                            break
                        except:
                            continue
                            
                if not chinese_font_registered:
                    print("⚠️ 未找到中文字体，使用默认字体")
                    
            except Exception as e:
                print(f"⚠️ 字体注册失败: {e}")
            
            # 自定义样式 - 支持中文
            title_style = ParagraphStyle(
                'CustomTitle',
                parent=styles['Heading1'],
                fontSize=18,
                alignment=TA_CENTER,
                spaceAfter=30,
                fontName='ChineseFont' if 'chinese_font_registered' in locals() and chinese_font_registered else 'Helvetica-Bold'
            )
            
            content_style = ParagraphStyle(
                'CustomContent',
                parent=styles['Normal'],
                fontSize=11,
                alignment=TA_JUSTIFY,
                leftIndent=20,
                rightIndent=20,
                spaceAfter=12,
                fontName='ChineseFont' if 'chinese_font_registered' in locals() and chinese_font_registered else 'Helvetica'
            )
            
            # 构建PDF内容
            story = []
            
            # 标题
            story.append(Paragraph("网站内容总结报告", title_style))
            story.append(Spacer(1, 20))
            
            # 生成时间
            time_text = f"生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
            story.append(Paragraph(time_text, styles['Normal']))
            story.append(Spacer(1, 20))
            
            # 分隔线
            story.append(Paragraph("=" * 50, styles['Normal']))
            story.append(Spacer(1, 15))
            
            # 主要内容 - 改进中文处理
            if content:
                # 清理和处理内容
                content = content.replace('\r\n', '\n').replace('\r', '\n')
                paragraphs = content.split('\n')
                
                for para in paragraphs:
                    if para.strip():
                        # 处理特殊字符，确保PDF可以正确显示
                        clean_para = para.strip()
                        # 转换HTML实体
                        clean_para = clean_para.replace('&lt;', '<').replace('&gt;', '>').replace('&amp;', '&')
                        
                        try:
                            story.append(Paragraph(clean_para, content_style))
                            story.append(Spacer(1, 8))
                        except Exception as para_error:
                            # 如果段落有问题，尝试用默认字体
                            try:
                                fallback_style = ParagraphStyle(
                                    'Fallback',
                                    parent=styles['Normal'],
                                    fontSize=10,
                                    leftIndent=20,
                                    rightIndent=20,
                                    spaceAfter=10
                                )
                                story.append(Paragraph(clean_para, fallback_style))
                                story.append(Spacer(1, 8))
                            except:
                                # 如果还是有问题，记录错误但继续
                                print(f"⚠️ 段落处理失败: {clean_para[:50]}...")
                                continue
            else:
                story.append(Paragraph("暂无内容", content_style))
            
            # 页脚信息
            story.append(Spacer(1, 30))
            story.append(Paragraph("=" * 50, styles['Normal']))
            story.append(Paragraph("本报告由 Playwright PDF Agent 自动生成", styles['Italic']))
            
            # 生成PDF
            doc.build(story)
            
            # 获取绝对路径
            abs_path = os.path.abspath(filename)
            print(f"📄 PDF文件生成完成: {abs_path}")
            return f"PDF文件已成功生成: {abs_path}"
            
        except Exception as e:
            error_msg = f"PDF生成失败: {str(e)}"
            print(error_msg)
            return error_msg


    # 3. 创建串行链
    print("=== 创建串行链：网站总结 → PDF生成 ===")

    # 方法1：简单串行链
    simple_chain = summarize_website | generate_pdf

    # 方法2：带LLM优化的串行链
    optimization_prompt = ChatPromptTemplate.from_template(
        """请优化以下网站总结内容，使其更适合PDF报告格式：

    原始总结：
    {summary}

    请重新组织内容，包括：
    1. 清晰的标题和结构
    2. 要点总结
    3. 详细说明
    4. 使用要求等

    优化后的内容："""
    )

    model = init_chat_model("deepseek-chat", model_provider="deepseek")

    # 带优化的串行链：网站总结 → LLM优化 → PDF生成
    optimized_chain = (
        summarize_website 
        | (lambda summary: {"summary": summary})
        | optimization_prompt 
        | model 
        | StrOutputParser() 
        | generate_pdf
    )

    # 4. 测试函数
    def test_simple_chain(url: str):
        """测试简单串行链"""
        print(f"\n🔄 开始处理URL: {url}")
        print("📝 步骤1: 网站总结...")
        print("📄 步骤2: 生成PDF...")
        
        result = simple_chain.invoke(url)
        print(f"✅ 完成: {result}")
        return result

    def test_optimized_chain(url: str):
        """测试优化串行链"""
        print(f"\n🔄 开始处理URL (优化版): {url}")
        print("📝 步骤1: 网站总结...")
        print("🎨 步骤2: 内容优化...")
        print("📄 步骤3: 生成PDF...")
        
        result = optimized_chain.invoke(url)
        print(f"✅ 完成: {result}")
        return result

    # 5. 创建交互式函数
    def create_website_pdf_report(url: str, use_optimization: bool = True):
        """创建网站PDF报告的主函数"""
        print("=" * 60)
        print("🤖 网站内容PDF生成器")
        print("=" * 60)
        
        try:
            if use_optimization:
                result = test_optimized_chain(url)
            else:
                result = test_simple_chain(url)
                
            print("\n" + "=" * 60)
            print("🎉 任务完成！")
            print("=" * 60)
            return result
            
        except Exception as e:
            error_msg = f"❌ 处理失败: {str(e)}"
            print(error_msg)
            return error_msg

    # 6. 主程序入口
    if __name__ == "__main__":
        # 测试URL
        test_url = "https://github.com/fufankeji/MateGen/blob/main/README_zh.md"
        
        print("选择处理方式:")
        print("1. 简单串行链（直接总结 → PDF）")
        print("2. 优化串行链（总结 → 优化 → PDF）")
        
        choice = input("请选择 (1/2): ").strip()
        
        if choice == "1":
            create_website_pdf_report(test_url, use_optimization=False)
        elif choice == "2":
            create_website_pdf_report(test_url, use_optimization=True)
        else:
            print("使用默认优化模式...")
            create_website_pdf_report(test_url, use_optimization=True) 
```
RAG项目-----服装商品智能客服

📁 项目结构说明
RAG开发/
├── chat_history/          # 对话历史持久化存储目录
├── chroma_db/             # Chroma 向量数据库持久化目录
├── data/                  # 待入库的知识库原始文件（如 .txt/.md）
├── app_file_uploader.py   # 文件上传 & 知识库构建模块
├── app_qa.py              # 主程序：Streamlit 对话界面
├── config_data.py         # 全局配置（API Key、模型名称等）
├── file_history_store.py  # 对话历史持久化工具
├── knowledge_base.py      # 知识库构建工具（文档加载、分块、入库）
├── md5.text               # 文件校验或其他辅助文件
├── rag.py                 # RAG 核心服务（Chain 构建、prompt 模板）
└── vector_stores.py       # 向量数据库服务（Chroma 初始化、检索器）

离线流程:
本地知识文件加载和读取->文本切分->向量数据库

用户上传文件离线流程
1. 用户交互阶段（app_file_upload.py）
├─ 用户通过 WEB 网页上传文件
├─ Streamlit 的 st.file_uploader() 接收文件
├─ uploader_file.get_value() 获取文件原始内容
└─ 将文件内容传递给 st.session_state 中的 KnowledgeBaseService 实例
2. 核心服务处理阶段（knowledge_base.py）
├─ 第一步：MD5防重复校验
│  ├─ get_string_md5(str)：计算文件内容的MD5值
│  ├─ check_md5()：读取 md5.text，检查该文件是否已上传过
│  └─ 若为新文件，调用 save_md5() 将新MD5写入 md5.text
└─ 第二步：向量入库
   ├─ upload_by_str(self, data, filename)：处理文件内容
   ├─ self.spliter：将文件文本分割为适合向量化的块
   ├─ self.chroma：连接Chroma向量库客户端
   └─ 将分割后的文本块向量化并写入 Chroma 向量库

在线流程:
用户提问 -> 问题向量化 -> 向量库检索服装资料 -> 拼接提示词(资料+历史+问题) -> LLM流式生成答案 -> 界面实时输出 -> 保存对话历史

用户在线流程详细步骤：
1. 用户交互阶段（app_qa.py）
├─ 用户在聊天框输入问题
├─ 前端实时显示用户消息
└─ 消息存入会话状态 st.session_state
2. RAG 核心处理阶段（rag.py）
├─ 第1步：将用户问题送入 chain.stream
├─ 第2步：检索模块启动
│  ├─ 用户问题向量化
│  ├─ 在 chroma_db 中匹配最相似的服装资料
│  └─ 返回商品材质、尺码、款式、价格等参考信息
├─ 第3步：提示词组装
│  ├─ 系统提示：基于服装资料专业回答
│  ├─ 注入检索到的知识库内容
│  ├─ 注入历史对话记录
│  └─ 加入用户当前问题
└─ 第4步：LLM生成流式回答
3. 前端展示阶段（app_qa.py）
├─ capture() 缓存每一段输出
├─ st.write_stream 实现打字机效果
└─ 拼接完整答案存入对话历史
4. 对话历史持久化
├─ 对话存入 chat_history 目录
└─ 下次对话自动加载上下文，实现多轮问答


# AI Hub 智能应用平台 - 需求规格说明书

## 1. 项目概述

开发一个名为 **AI Hub** 的智能应用中心，核心功能包括：

- **AI 聊天室**：支持多轮对话、记忆持久化、RAG知识库检索、工具调用与MCP服务调用。
- **AI 超级智能体**：能够根据用户复杂需求自主推理、规划并执行多步骤操作，直至完成目标。
- **内置工具集**：联网搜索、文件操作、网页抓取、资源下载、终端操作、PDF生成。
- **MCP 服务**：从指定网站（如Pexels、Unsplash）搜索图片。

界面风格参考所给图片：深色/科技感主题，卡片式布局，包含“AI聊天”、“哄哄模拟器”等入口（本项目聚焦于“AI聊天”和“超级智能体”，其他卡片可预留或作为示例）。

技术栈：

- 后端：Java 21 + Maven + Spring AI + LangChain4j + PGVector + 云数据库（如阿里云RDS PG）
- 前端：Vue 3 + Vite + 炫酷UI库（如Naive UI / Element Plus + 动画/粒子背景）
- AI模型：支持OpenAI、Azure、国产模型（通义千问、智谱）以及本地部署模型（Ollama + Llama3）
- 部署：容器化 + Serverless（如阿里云函数计算FC）

---

## 2. 前端需求

### 2.1 页面结构

整体采用单页面应用（SPA），包含以下主要视图：

1. **首页 / 应用中心**：参照图片，顶部导航栏（Logo、登录状态），中间为卡片网格展示各类AI应用（AI聊天、超级智能体、哄哄模拟器等）。点击卡片进入对应功能页。
2. **AI聊天室页面**：与智能助手进行自然对话，支持多轮、记忆、文件上传（RAG）、工具调用可视化。
3. **超级智能体页面**：用户输入复杂任务，智能体展示推理步骤和执行过程，实时反馈结果。
4. **设置/个人中心**（可选）：API Key配置、对话历史管理。

### 2.2 核心交互组件

- **对话窗口**：消息气泡（用户/AI），支持Markdown渲染、代码高亮、图片/文件显示。
- **输入框**：支持多行文本、附件上传（PDF/Word/TXT等用于RAG）、语音输入（可选）。
- **工具调用指示器**：显示AI正在调用哪些工具（如“正在联网搜索…”），并展示工具返回的摘要。
- **智能体工作流面板**：以步骤卡片或流程图形式展示智能体的思考链（Thought → Action → Observation）。
- **侧边栏**：对话历史列表（可新建、删除、重命名）、知识库管理入口。

### 2.3 前端与后端API集成

| 功能 | API端点 | 请求方式 | 说明 |
|------|---------|----------|------|
| 发送聊天消息 | `/api/chat/message` | POST | 支持流式响应（SSE或WebSocket） |
| 获取对话历史 | `/api/chat/history/{sessionId}` | GET | 分页加载 |
| 删除对话 | `/api/chat/history/{sessionId}` | DELETE | |
| 上传文档构建知识库 | `/api/rag/upload` | POST | 返回文档ID |
| 创建智能体任务 | `/api/agent/run` | POST | 非流式或流式 |
| 获取智能体任务状态 | `/api/agent/status/{taskId}` | GET | 轮询或SSE |
| 调用MCP搜索图片 | `/api/mcp/search-image` | POST | 参数：关键词、数量 |
| 文件下载/预览 | `/api/tool/download?url=...` | GET | 触发资源下载 |

### 2.4 样式要求

- 深色主题（#0A0F1F背景，霓虹蓝/紫色点缀）。
- 卡片悬浮动画、玻璃态模糊效果。
- 消息输入框支持 @ 唤起工具选择（如@搜索、@PDF生成）。
- 响应式布局：PC优先，适配平板。

---

## 3. 后端需求

### 3.1 总体架构

采用模块化Maven多模块项目：

- `ai-hub-core`：公共实体、工具类、异常处理。
- `ai-hub-chat`：聊天室核心逻辑（对话记忆、RAG、工具调用路由）。
- `ai-hub-agent`：超级智能体引擎（基于ReAct或Plan-and-Solve模式）。
- `ai-hub-tools`：具体工具实现（联网搜索、文件操作等）。
- `ai-hub-mcp`：MCP服务（图片搜索）。
- `ai-hub-web`：Spring MVC控制器、WebSocket/SSE端点。

### 3.2 数据库设计

使用 **PostgreSQL + pgvector** 扩展存储向量，云数据库服务（如阿里云RDS PG）。

**主要表结构：**

```sql
-- 对话会话表
CREATE TABLE conversation (
    id UUID PRIMARY KEY,
    user_id VARCHAR(64),
    title VARCHAR(255),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- 消息表（支持多轮记忆）
CREATE TABLE message (
    id UUID PRIMARY KEY,
    session_id UUID REFERENCES conversation(id),
    role VARCHAR(20), -- user, assistant, tool
    content TEXT,
    tool_calls JSONB,
    created_at TIMESTAMP
);

-- 知识库文档表
CREATE TABLE knowledge_doc (
    id UUID PRIMARY KEY,
    file_name VARCHAR(255),
    content TEXT,
    embedding vector(1536), -- 根据模型维度调整
    metadata JSONB
);

-- 智能体任务表
CREATE TABLE agent_task (
    id UUID PRIMARY KEY,
    user_query TEXT,
    status VARCHAR(20), -- RUNNING, COMPLETED, FAILED
    steps JSONB, -- 存储推理过程
    final_answer TEXT,
    created_at TIMESTAMP
);
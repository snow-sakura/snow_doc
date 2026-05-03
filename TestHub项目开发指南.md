# TestHub 智能测试管理平台 - 项目开发完整指南

## 📋 目录

1. [项目概述](#1-项目概述)
2. [技术架构](#2-技术架构)
3. [环境配置](#3-环境配置)
4. [后端开发](#4-后端开发)
5. [前端开发](#5-前端开发)
6. [AI大模型配置详解](#6-ai大模型配置详解)
7. [提示词配置与完善](#7-提示词配置与完善)
8. [核心功能模块](#8-核心功能模块)
9. [部署指南](#9-部署指南)

---

## 1. 项目概述

### 1.1 项目简介

**TestHub** 是一个功能强大的智能测试管理平台，集成了 **AI 需求分析**、**测试用例管理**、**API 测试**、**UI 自动化测试** 等多个模块，旨在提升测试效率和质量。

### 1.2 核心特性

- 🤖 **AI 智能化能力**: AI 需求分析、智能测试用例生成、Dify AI 助手、多模型支持
- 🔐 **安全机制**: JWT 双 Token 认证、自动刷新、Token 黑名单
- ⚙️ **统一配置中心**: 环境检测、驱动管理、AI 模型配置
- 📋 **测试用例管理**: 完整生命周期管理、灵活组织、评审流程
- 🌐 **API 测试**: HTTP/WebSocket、项目集合、环境变量、定时任务
- 🖥️ **UI 自动化测试**: Selenium/Playwright 双引擎、元素管理、页面对象模式
- 📱 **APP 自动化测试**: Airtest 框架、设备管理、组件化编排
- 🏭 **数据工厂**: 多种数据生成和转换工具

---

## 2. 技术架构

### 2.1 后端技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **Django** | 4.2.7 | Web 框架 |
| **Django REST Framework** | 3.14.0 | RESTful API |
| **Python** | 3.12+ | 编程语言 |
| **MySQL** | 8.0+ | 数据库 |
| **Celery** | 5.3.4 | 异步任务队列 |
| **Redis** | 5.0.1 | 缓存/消息队列 |
| **Channels** | 4.3.2 | WebSocket 支持 |

### 2.2 前端技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **Vue.js** | 3.3.4 | 前端框架 |
| **Vite** | 7.3.1 | 构建工具 |
| **Element Plus** | 2.3.9 | UI 组件库 |
| **Pinia** | 2.1.6 | 状态管理 |
| **Vue Router** | 4.2.4 | 路由管理 |
| **Axios** | 1.5.0 | HTTP 客户端 |
| **ECharts** | 5.4.3 | 数据可视化 |
| **Monaco Editor** | 0.53.0 | 代码编辑器 |

### 2.3 项目目录结构

```
testhub_platform/
├── backend/                          # Django 后端
│   ├── __init__.py
│   ├── asgi.py                      # ASGI 配置 (Channels)
│   ├── celery.py                    # Celery 配置
│   ├── middleware.py                # 中间件
│   ├── settings.py                  # Django 配置
│   ├── urls.py                      # URL 路由
│   └── wsgi.py                      # WSGI 配置
│
├── apps/                            # Django 应用模块
│   ├── api_testing/                 # API 接口测试
│   ├── ui_automation/               # Web UI 自动化测试
│   ├── app_automation/              # APP 自动化测试 (Android)
│   ├── requirement_analysis/         # 需求分析与 AI 生成 ⭐
│   ├── assistant/                   # AI 智能助手
│   ├── data_factory/                # 数据工厂工具
│   ├── core/                        # 核心公共模块
│   ├── users/                       # 用户管理
│   ├── projects/                    # 项目管理
│   ├── testcases/                   # 测试用例
│   ├── testsuites/                  # 测试套件
│   ├── executions/                  # 测试执行
│   ├── reports/                     # 测试报告
│   ├── reviews/                     # 用例评审
│   └── versions/                    # 版本管理
│
├── frontend/                        # Vue 前端
│   ├── src/
│   │   ├── api/                     # API 调用
│   │   ├── components/              # 公共组件
│   │   ├── views/                   # 页面视图
│   │   ├── stores/                  # Pinia 状态
│   │   ├── router/                  # 路由配置
│   │   ├── locales/                 # 国际化
│   │   └── utils/                   # 工具函数
│   ├── package.json
│   └── vite.config.js
│
├── docs/                            # 文档
├── allure/                          # Allure 配置
├── media/                           # 媒体文件
├── logs/                            # 日志文件
├── requirements.txt                 # Python 依赖
└── manage.py                        # Django 管理脚本
```

---

## 3. 环境配置

### 3.1 环境要求

- **Python**: 3.12+
- **Node.js**: 18+
- **MySQL**: 8.0+
- **Redis**: 6.0+ (用于 Celery 和 Channels)
- **Git**

### 3.2 配置文件 (.env.example)

```bash
# ================================
# Django配置
# ================================
SECRET_KEY=your-secret-key-here-change-in-production
DEBUG=True
ALLOWED_HOSTS=*
CORS_ALLOWED_ORIGINS=http://localhost,http://127.0.0.1

# ================================
# 数据库配置
# ================================
DB_NAME=testhub
DB_USER=root
DB_PASSWORD=your-database-password
DB_PORT=3306
DB_HOST=127.0.0.1

# ================================
# Redis配置
# ================================
REDIS_URL=redis://127.0.0.1:6379/0

# ================================
# 邮件配置
# ================================
EMAIL_HOST=smtp.163.com
EMAIL_PORT=465
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your_email@gmail.com
EMAIL_HOST_PASSWORD=your_email_password

# ================================
# 本地化配置
# ================================
LANGUAGE_CODE=zh-hans
TIME_ZONE=Asia/Shanghai
```

### 3.3 创建 .env 文件

```bash
cd /Users/snow/snow-sakura/snow-python/testhub_platform
cp .env.example .env
# 编辑 .env 填入实际配置
```

---

## 4. 后端开发

### 4.1 安装依赖

```bash
# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt
```

### 4.2 数据库迁移

```bash
# 生成迁移文件
python manage.py makemigrations

# 执行迁移
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser
```

### 4.3 启动后端服务

```bash
# 开发模式
python manage.py runserver 0.0.0.0:8000

# 生产模式 (需要先配置 Nginx)
daphne -b 0.0.0.0 -p 8000 backend.asgi:application
```

### 4.4 Celery 异步任务服务

```bash
# 启动 Celery Worker
celery -A backend worker -l info

# 启动 Celery Beat (定时任务)
celery -A backend beat -l info
```

### 4.5 核心应用说明

#### 需求分析模块 (requirement_analysis)
- **ai_models.py**: AI 模型配置和服务
- **advanced_analyzer.py**: 高级需求分析器
- **models.py**: 数据模型定义
- **views.py**: API 视图实现
- **services.py**: 业务服务层

#### UI 自动化模块 (ui_automation)
- **ai_base.py**: AI 自动化基础类 (96KB，核心 AI 功能)
- **ai_agent.py**: AI Agent 实现
- **selenium_engine.py**: Selenium 执行引擎
- **playwright_engine.py**: Playwright 执行引擎
- **test_executor.py**: 测试执行器

#### 智能助手模块 (assistant)
- **views.py**: Dify API 集成
- **views_config.py**: 配置视图

---

## 5. 前端开发

### 5.1 安装依赖

```bash
cd frontend
npm install
```

### 5.2 开发模式启动

```bash
npm run dev
# 访问 http://localhost:3000
```

### 5.3 构建生产版本

```bash
npm run build
# 输出到 dist/ 目录
```

### 5.4 前端核心页面

| 路径 | 功能 |
|------|------|
| `/views/requirement-analysis/AIModelConfig.vue` | AI 模型配置 |
| `/views/requirement-analysis/PromptConfig.vue` | 提示词配置 |
| `/views/configuration/AIIntelligentModeConfig.vue` | AI 智能模式配置 |
| `/views/configuration/DifyConfig.vue` | Dify 配置 |
| `/views/ui-automation/ai/` | UI 自动化 AI 功能 |
| `/views/assistant/AssistantView.vue` | AI 智能助手 |

---

## 6. AI大模型配置详解

### 6.1 支持的 AI 模型提供商

| 模型类型 | 模型名称 | Base URL 示例 |
|----------|----------|---------------|
| **DeepSeek** | deepseek-chat | https://api.deepseek.com |
| **通义千问** | qwen-turbo / qwen-plus | https://dashscope.aliyuncs.com |
| **硅基流动** | various models | https://api.siliconflow.cn |
| **智谱 AI** | glm-4 | https://open.bigmodel.cn |
| **其他** | 自定义 | 用户指定 |

### 6.2 AI 模型配置模型 (Django Model)

位置: `apps/requirement_analysis/ai_models.py`

```python
class AIModelConfig(models.Model):
    """AI模型配置模型"""
    MODEL_CHOICES = [
        ('deepseek', 'DeepSeek'),
        ('qwen', '通义千问'),
        ('siliconflow', '硅基流动'),
        ('other', '其他'),
    ]
    
    ROLE_CHOICES = [
        ('writer', '测试用例编写专家'),
        ('reviewer', '测试评审专家'),
    ]
    
    name = models.CharField(max_length=100, verbose_name='配置名称')
    model_type = models.CharField(max_length=20, choices=MODEL_CHOICES)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES)
    api_key = models.CharField(max_length=200, verbose_name='API Key')
    base_url = models.URLField(verbose_name='API Base URL')
    model_name = models.CharField(max_length=100, verbose_name='模型名称')
    max_tokens = models.IntegerField(default=4096)
    temperature = models.FloatField(default=0.7)
    top_p = models.FloatField(default=0.9)
    is_active = models.BooleanField(default=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
```

### 6.3 API 调用服务

```python
class AIModelService:
    """AI模型服务类"""
    
    @staticmethod
    async def call_openai_compatible_api(config: AIModelConfig, messages: List[Dict[str, str]]) -> Dict[str, Any]:
        """调用OpenAI兼容格式的API"""
        headers = {
            'Authorization': f'Bearer {config.api_key}',
            'Content-Type': 'application/json'
        }
        
        data = {
            'model': config.model_name,
            'messages': messages,
            'max_tokens': config.max_tokens,
            'temperature': config.temperature,
            'top_p': config.top_p,
            'stream': False
        }
        
        # 智能补全 URL 路径
        base_url = config.base_url.rstrip('/')
        if not base_url.endswith('/chat/completions'):
            if base_url.endswith('/v1'):
                url = f"{base_url}/chat/completions"
            else:
                url = f"{base_url}/v1/chat/completions"
        
        async with httpx.AsyncClient(timeout=120.0) as client:
            response = await client.post(url, headers=headers, json=data)
            response.raise_for_status()
            return response.json()
```

### 6.4 推荐的 AI 模型配置

#### DeepSeek 配置
```yaml
模型类型: deepseek
角色: writer (测试用例编写专家)
API Key: sk-xxxxxxxxxxxxxxxxxxxx
Base URL: https://api.deepseek.com
模型名称: deepseek-chat
最大Token: 4096
温度参数: 0.7
Top P: 0.9
```

#### 通义千问配置
```yaml
模型类型: qwen
角色: reviewer (测试评审专家)
API Key: sk-xxxxxxxxxxxxxxxxxxxx
Base URL: https://dashscope.aliyuncs.com
模型名称: qwen-plus
最大Token: 4096
温度参数: 0.5
Top P: 0.9
```

### 6.5 配置页面路径

- **前端配置页面**: `frontend/src/views/requirement-analysis/AIModelConfig.vue`
- **智能模式配置**: `frontend/src/views/configuration/AIIntelligentModeConfig.vue`
- **API 调用**: `frontend/src/api/requirement-analysis.js`

---

## 7. 提示词配置与完善

### 7.1 提示词配置模型

位置: `apps/requirement_analysis/ai_models.py`

```python
class PromptConfig(models.Model):
    """提示词配置模型"""
    PROMPT_CHOICES = [
        ('writer', '用例编写提示词'),
        ('reviewer', '用例评审提示词'),
    ]
    
    name = models.CharField(max_length=100, verbose_name='配置名称')
    prompt_type = models.CharField(max_length=20, choices=PROMPT_CHOICES)
    content = models.TextField(verbose_name='提示词内容')
    is_active = models.BooleanField(default=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
```

### 7.2 测试用例编写提示词 (writer)

这是一个完整的提示词模板，建议直接使用或根据业务需求调整：

```markdown
# 测试用例编写专家提示词

## 角色定义
你是一位经验丰富的测试工程师，专注于根据需求文档编写高质量的测试用例。

## 核心要求

### 1. 测试用例格式
请严格按照以下格式输出测试用例，使用 Markdown 表格：

| 用例编号 | 用例标题 | 优先级 | 前置条件 | 测试步骤 | 预期结果 |
|----------|----------|--------|----------|----------|----------|
| TC-001 | [用例标题] | P0/P1/P2 | [前置条件] | [步骤1<br>步骤2<br>步骤3] | [预期结果] |

### 2. 编号规则
- 编号必须连续，中间不能有遗漏
- 格式: TC-001, TC-002, TC-003...
- 同一模块的用例可以使用模块前缀: TC-LOGIN-001, TC-PAY-001

### 3. 优先级定义
- **P0**: 核心功能，冒烟测试用例，必须优先测试
- **P1**: 重要功能，正常路径测试
- **P2**: 辅助功能，边界和异常测试

### 4. 测试类型覆盖
为每个需求生成以下类型的测试用例：
1. **正常路径测试**: 验证基本功能正常工作
2. **异常路径测试**: 验证异常情况正确处理
3. **边界值测试**: 验证边界条件处理正确
4. **性能测试**: 验证性能指标符合要求
5. **安全测试**: 验证安全控制有效

### 5. 测试步骤规范
- 每个步骤描述要清晰、具体、可执行
- 包含必要的测试数据
- 预期结果要明确、可验证
- 步骤之间要有明确的依赖关系说明

### 6. 特殊字符处理（关键）
- 如果在表格内容中出现管道符 '|'，必须转义为 '\|'
- 否则会导致表格列错位，无法解析
- 示例：应输入 'a\|b' 而不是 'a|b'

### 7. 需求理解要点
- 仔细阅读需求文档，理解业务背景
- 识别关键业务流程和用户角色
- 关注功能边界和异常场景
- 考虑非功能性需求（性能、安全、兼容性）

### 8. 输出要求
- 所有用例必须一次性完整输出，不能中断
- 保持格式一致性
- 确保每个用例都有明确的预期结果

## 示例输出格式

```markdown
## 用户登录功能测试用例

### 1. 正常路径测试

| 用例编号 | 用例标题 | 优先级 | 前置条件 | 测试步骤 | 预期结果 |
|----------|----------|--------|----------|----------|----------|
| TC-LOGIN-001 | 正确账号密码登录成功 | P0 | 用户已注册 | 1. 打开登录页面<br>2. 输入正确用户名<br>3. 输入正确密码<br>4. 点击登录按钮 | 登录成功，跳转至首页 |
```

---

### 7.3 测试用例评审提示词 (reviewer)

```markdown
# 测试用例评审专家提示词

## 角色定义
你是一位资深的测试评审专家，负责审查和优化测试用例的质量。

## 评审维度

### 1. 完整性检查
- [ ] 是否覆盖了所有需求点
- [ ] 是否包含正常路径和异常路径
- [ ] 是否覆盖了边界值场景
- [ ] 是否考虑了性能和安全因素

### 2. 正确性检查
- [ ] 测试步骤是否清晰可执行
- [ ] 预期结果是否明确可验证
- [ ] 前置条件是否完整
- [ ] 测试数据是否合理

### 3. 可维护性检查
- [ ] 用例编号是否规范
- [ ] 用例标题是否清晰
- [ ] 步骤描述是否简洁
- [ ] 是否存在重复用例

### 4. 可执行性检查
- [ ] 步骤是否可以在实际环境中执行
- [ ] 依赖关系是否明确
- [ ] 环境要求是否可满足

### 5. 评审输出格式

```markdown
## 评审报告

### 问题列表
| 序号 | 用例编号 | 问题类型 | 问题描述 | 建议修改 |
|------|----------|----------|----------|----------|
| 1 | TC-001 | 完整性 | 缺少边界值测试 | 建议添加最大/最小值测试用例 |

### 改进建议
1. [具体改进建议...]
2. [具体改进建议...]

### 通过评审的用例
- [通过评审的用例列表...]
```
```

---

### 7.4 AI 智能模式提示词 (UI 自动化)

位置: `apps/ui_automation/ai_base.py`

```python
# Browser-use 集成使用的系统提示词
SYSTEM_PROMPT = """
You are an expert web testing agent. Your task is to:
1. Understand the user's testing goal
2. Analyze the current page structure
3. Generate appropriate browser actions
4. Verify the test results

Available actions:
- navigate: Go to a URL
- click: Click on an element
- input: Enter text into an input field
- wait: Wait for an element to appear
- assert: Verify an element's content
- screenshot: Take a screenshot

Always prioritize reliability and precision in your actions.
"""
```

### 7.5 提示词配置页面

- **前端配置页面**: `frontend/src/views/requirement-analysis/PromptConfig.vue`
- **默认提示词加载**: 通过前端 `loadDefaultPrompts()` 方法
- **提示词模板**: 存储在数据库 PromptConfig 表中

---

## 8. 核心功能模块

### 8.1 需求分析模块

#### 流程图
```
需求文档上传 → 文本提取 → AI 需求分析 → 业务需求生成 → 测试用例生成 → 用例评审
```

#### API 端点

| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/requirement/documents/` | GET/POST | 文档列表/上传 |
| `/api/requirement/documents/{id}/analyze/` | POST | 分析文档 |
| `/api/requirement/documents/{id}/extract_text/` | GET | 提取文本 |
| `/api/requirement/ai-model-configs/` | GET/POST | AI 模型配置 |
| `/api/requirement/prompt-configs/` | GET/POST | 提示词配置 |

### 8.2 UI 自动化模块

#### 支持的定位策略
- ID
- XPath
- CSS Selector
- Name
- Class Name
- Tag Name
- Link Text
- Partial Link Text

#### 支持的浏览器
- Chrome
- Firefox
- Edge

#### AI 智能模式功能
- 基于 Browser-use 框架
- 支持文本模式和视觉模式
- 智能任务规划和步骤生成
- 自动处理复杂交互

### 8.3 APP 自动化模块

#### 支持的功能
- Airtest 框架集成
- ADB 设备管理
- 图像识别定位
- 坐标/区域定位
- UI Flow 编排
- 多分辨率适配

---

## 9. 部署指南

### 9.1 开发环境部署

```bash
# 1. 克隆项目
git clone <repository-url>
cd testhub_platform

# 2. 配置环境
cp .env.example .env
# 编辑 .env 填入配置

# 3. 安装后端依赖
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 4. 安装前端依赖
cd frontend
npm install
cd ..

# 5. 数据库初始化
python manage.py migrate
python manage.py createsuperuser

# 6. 启动服务
# 终端1: 后端
python manage.py runserver 0.0.0.0:8000

# 终端2: Celery
celery -A backend worker -l info

# 终端3: 前端
cd frontend
npm run dev
```

### 9.2 生产环境部署

#### 使用 Docker 部署

```bash
# 构建镜像
docker build -t testhub:latest .

# 运行容器
docker run -d -p 8000:8000 -p 3000:3000 \
  --env-file .env \
  testhub:latest
```

#### 使用 Nginx + Gunicorn/Daphne

```nginx
# /etc/nginx/sites-available/testhub
upstream testhub_backend {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name your-domain.com;

    # 前端静态文件
    location / {
        root /path/to/testhub/frontend/dist;
        try_files $uri $uri/ /index.html;
    }

    # API 代理
    location /api/ {
        proxy_pass http://testhub_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # WebSocket 支持
    location /ws/ {
        proxy_pass http://testhub_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 9.3 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| 数据库连接失败 | 检查 MySQL 服务和 .env 配置 |
| AI API 调用失败 | 检查 API Key 和 Base URL 配置 |
| 前端无法访问后端 | 检查 CORS 配置和代理设置 |
| Celery 任务不执行 | 检查 Redis 连接和 Celery 服务 |
| 文件上传失败 | 检查 media 目录权限 |

---

## 📞 联系方式

如有问题，请参考项目中的 `docs/` 目录下的详细文档。

---

*文档生成时间: 2026-04-12*
*项目版本: v1.0.1*

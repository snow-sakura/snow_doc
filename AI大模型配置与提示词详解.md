# AI 大模型配置与提示词详解

## 📋 目录

1. [AI 模型配置概述](#1-ai-模型配置概述)
2. [支持的 AI 模型类型](#2-支持的-ai-模型类型)
3. [模型配置详解](#3-模型配置详解)
4. [提示词配置详解](#4-提示词配置详解)
5. [完整配置示例](#5-完整配置示例)
6. [高级配置](#6-高级配置)
7. [常见问题](#7-常见问题)

---

## 1. AI 模型配置概述

### 1.1 配置架构

```
┌─────────────────────────────────────────────────────────────┐
│                    前端配置页面                              │
│  AIModelConfig.vue / PromptConfig.vue                      │
└─────────────────────────┬─────────────────────────────────┘
                          │ API 调用
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Django REST Framework                    │
│                   requirement_analysis                       │
└─────────────────────────┬─────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    AI Model Service                         │
│                 AIModelService                              │
└─────────────────────────┬─────────────────────────────────┘
                          │ HTTP 调用
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   AI 模型提供商                             │
│         DeepSeek / 通义千问 / 硅基流动 / 其他               │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 核心文件位置

| 文件 | 位置 | 功能 |
|------|------|------|
| AI模型模型 | `apps/requirement_analysis/ai_models.py` | AIModelConfig 模型定义 |
| 提示词模型 | `apps/requirement_analysis/ai_models.py` | PromptConfig 模型定义 |
| AI服务类 | `apps/requirement_analysis/ai_models.py` | AIModelService 服务类 |
| 需求分析器 | `apps/requirement_analysis/advanced_analyzer.py` | 高级分析器 |
| 前端模型配置 | `frontend/src/views/requirement-analysis/AIModelConfig.vue` | 模型配置页面 |
| 前端提示词配置 | `frontend/src/views/requirement-analysis/PromptConfig.vue` | 提示词配置页面 |
| 前端智能模式 | `frontend/src/views/configuration/AIIntelligentModeConfig.vue` | AI 智能模式 |
| UI自动化AI | `apps/ui_automation/ai_base.py` | UI自动化AI功能 |
| 智能助手 | `apps/assistant/views.py` | Dify助手集成 |

---

## 2. 支持的 AI 模型类型

### 2.1 模型类型列表

| model_type | 显示名称 | Base URL | 特点 |
|------------|----------|----------|------|
| `deepseek` | DeepSeek | https://api.deepseek.com | 性价比高，中文支持好 |
| `qwen` | 通义千问 | https://dashscope.aliyuncs.com | 阿里云，稳定可靠 |
| `siliconflow` | 硅基流动 | https://api.siliconflow.cn | 聚合多家模型 |
| `zhipu` | 智谱AI | https://open.bigmodel.cn | 中文理解强 |
| `other` | 其他 | 用户指定 | 支持任意 OpenAI 兼容接口 |

### 2.2 推荐的模型组合

#### 方案一：DeepSeek 组合（推荐）
```yaml
编写专家:
  model_type: deepseek
  model_name: deepseek-chat
  base_url: https://api.deepseek.com
  temperature: 0.7
  max_tokens: 4096

评审专家:
  model_type: deepseek
  model_name: deepseek-chat
  base_url: https://api.deepseek.com
  temperature: 0.5
  max_tokens: 4096
```

#### 方案二：通义千问 + DeepSeek 组合
```yaml
编写专家:
  model_type: qwen
  model_name: qwen-plus
  base_url: https://dashscope.aliyuncs.com
  temperature: 0.7
  max_tokens: 4096

评审专家:
  model_type: deepseek
  model_name: deepseek-chat
  base_url: https://api.deepseek.com
  temperature: 0.5
  max_tokens: 4096
```

---

## 3. 模型配置详解

### 3.1 AIModelConfig 模型字段

```python
class AIModelConfig(models.Model):
    """AI模型配置模型"""
    
    # 基本信息
    name = models.CharField(max_length=100, verbose_name='配置名称')
    
    # 模型类型
    model_type = models.CharField(max_length=20, choices=MODEL_CHOICES)
    # 可选值: 'deepseek', 'qwen', 'siliconflow', 'other'
    
    # 角色
    role = models.CharField(max_length=20, choices=ROLE_CHOICES)
    # 可选值: 'writer' (用例编写专家), 'reviewer' (用例评审专家)
    
    # API 配置
    api_key = models.CharField(max_length=200, verbose_name='API Key')
    base_url = models.URLField(verbose_name='API Base URL')
    model_name = models.CharField(max_length=100, verbose_name='模型名称')
    
    # 生成参数
    max_tokens = models.IntegerField(default=4096, verbose_name='最大Token数')
    temperature = models.FloatField(default=0.7, verbose_name='温度参数')
    top_p = models.FloatField(default=0.9, verbose_name='Top P参数')
    
    # 状态
    is_active = models.BooleanField(default=True, verbose_name='是否启用')
    
    # 元数据
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

### 3.2 参数说明

| 参数 | 说明 | 推荐值 | 备注 |
|------|------|--------|------|
| `temperature` | 温度参数，控制随机性 | 0.5-0.9 | 越低越确定性，越高越创造性 |
| `max_tokens` | 最大生成 Token 数 | 2048-8192 | 根据输出长度需求调整 |
| `top_p` | 核采样参数 | 0.9 | 与 temperature 配合使用 |

### 3.3 URL 自动补全逻辑

```python
# AIModelService 中的 URL 补全逻辑
base_url = config.base_url.rstrip('/')

if not base_url.endswith('/chat/completions'):
    if base_url.endswith('/v1'):
        # 已有 /v1，添加 /chat/completions
        url = f"{base_url}/chat/completions"
    else:
        # 添加完整的 /v1/chat/completions
        url = f"{base_url}/v1/chat/completions"
else:
    url = base_url  # 已经完整
```

### 3.4 各平台配置详情

#### DeepSeek
```yaml
模型类型: deepseek
Base URL: https://api.deepseek.com
可用模型:
  - deepseek-chat (通用对话)
  - deepseek-coder (代码专用)
API Key 获取: https://platform.deepseek.com/api_keys
```

#### 通义千问
```yaml
模型类型: qwen
Base URL: https://dashscope.aliyuncs.com
可用模型:
  - qwen-turbo (快速响应)
  - qwen-plus (更强能力)
  - qwen-max (最强能力)
API Key 获取: https://dashscope.console.aliyun.com/apiKey
```

#### 硅基流动
```yaml
模型类型: siliconflow
Base URL: https://api.siliconflow.cn
可用模型:
  - Qwen/Qwen2.5-7B-Instruct
  - deepseek-ai/DeepSeek-V2.5
  - Anthropic/claude-3.5-sonnet
API Key 获取: https://www.siliconflow.cn/api-keys
```

---

## 4. 提示词配置详解

### 4.1 PromptConfig 模型

```python
class PromptConfig(models.Model):
    """提示词配置模型"""
    
    # 类型选择
    PROMPT_CHOICES = [
        ('writer', '用例编写提示词'),
        ('reviewer', '用例评审提示词'),
    ]
    
    # 基本信息
    name = models.CharField(max_length=100, verbose_name='配置名称')
    prompt_type = models.CharField(max_length=20, choices=PROMPT_CHOICES)
    content = models.TextField(verbose_name='提示词内容')
    
    # 状态
    is_active = models.BooleanField(default=True, verbose_name='是否启用')
    
    # 元数据
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

### 4.2 提示词调用流程

```python
# apps/requirement_analysis/ai_models.py

class AIModelService:
    @staticmethod
    async def generate_test_cases(task: TestCaseGenerationTask) -> str:
        """生成测试用例"""
        # 获取提示词配置
        writer_prompt = task.writer_prompt_config.content
        
        # 构建用户消息
        user_message = (
            f"请根据以下需求生成测试用例：\n"
            f"【需求文档内容】\n{task.requirement_text}"
        )
        
        # 构建消息列表
        messages = [
            {"role": "system", "content": writer_prompt},
            {"role": "user", "content": user_message}
        ]
        
        # 调用 AI 服务
        response = await AIModelService.call_openai_compatible_api(
            task.writer_model_config, messages
        )
        
        return response['choices'][0]['message']['content']
    
    @staticmethod
    async def review_test_cases(task: TestCaseGenerationTask, test_cases: str) -> str:
        """评审测试用例"""
        # 获取评审提示词
        reviewer_prompt = task.reviewer_prompt_config.content
        
        user_message = f"请评审以下测试用例：\n\n{test_cases}"
        
        messages = [
            {"role": "system", "content": reviewer_prompt},
            {"role": "user", "content": user_message}
        ]
        
        response = await AIModelService.call_openai_compatible_api(
            task.reviewer_model_config, messages
        )
        
        return response['choices'][0]['message']['content']
```

### 4.3 完整提示词模板

#### 4.3.1 用例编写提示词（优化版）

```markdown
# 测试用例编写专家 - 系统提示词

## 角色定位
你是一位拥有10年以上经验的资深测试工程师，精通各类软件测试方法和最佳实践。你的职责是根据需求文档生成高质量、全面、可执行的测试用例。

## 核心能力
- 深入理解业务需求和用户故事
- 识别潜在的功能和非功能需求
- 设计覆盖全面的测试场景
- 编写清晰、可执行的测试步骤
- 考虑边界条件和异常情况

## 输出格式要求

### Markdown 表格格式
```markdown
| 用例编号 | 用例标题 | 优先级 | 前置条件 | 测试步骤 | 预期结果 |
|----------|----------|--------|----------|----------|----------|
| TC-XXX-001 | [清晰简洁的标题] | P0/P1/P2 | [具体的前置条件] | [详细步骤] | [明确的预期] |
```

### 编号规范
- 格式: TC-{模块}-{序号}
- 例如: TC-LOGIN-001, TC-PAY-002
- 模块建议: LOGIN(登录), USER(用户), ORDER(订单), PAY(支付), ADMIN(管理)

### 优先级定义
- **P0 (冒烟测试)**: 核心功能，必须通过
  - 主要业务流程
  - 关键功能点
  - 高风险场景

- **P1 (核心测试)**: 重要功能，推荐测试
  - 次要业务流程
  - 功能分支
  - 数据验证

- **P2 (扩展测试)**: 辅助功能，按需测试
  - 边界值
  - 异常场景
  - 兼容性

## 测试用例设计原则

### 1. 完整性原则
每个需求应覆盖：
- ✅ 正常路径（Happy Path）
- ✅ 异常路径（Error Path）
- ✅ 边界值（Boundary Value）
- ✅ 等价类（Equivalence Class）

### 2. 可执行原则
- ✅ 步骤清晰，无歧义
- ✅ 测试数据具体
- ✅ 预期结果可验证
- ✅ 无外部依赖或明确说明依赖

### 3. 独立性原则
- ✅ 用例之间相互独立
- ✅ 可单独执行
- ✅ 无执行顺序要求（除非明确说明）

## 特殊要求

### 管道符转义（关键）
⚠️ **重要**: 如果在任何表格单元格中需要使用管道符 `|`，必须转义为 `\|`

正确示例：
- 预期结果: "显示 \"成功\|完成\" 状态"
- 前置条件: "文件格式为 CSV\|TSV"

错误示例（会导致表格错位）：
- 预期结果: "显示 '成功|完成' 状态"
- 前置条件: "文件格式为 CSV|TSV"

### 步骤格式
测试步骤使用 `<br>` 换行：
```
1. 打开登录页面<br>2. 输入用户名: test@example.com<br>3. 输入密码: Test123456<br>4. 点击登录按钮
```

## 输出示例

### 示例1：登录功能

```markdown
## 登录功能测试用例

### P0 - 冒烟测试

| 用例编号 | 用例标题 | 优先级 | 前置条件 | 测试步骤 | 预期结果 |
|----------|----------|--------|----------|----------|----------|
| TC-LOGIN-001 | 正确账号密码登录成功 | P0 | 用户已注册账号 test@example.com | 1. 打开登录页面<br>2. 输入用户名: test@example.com<br>3. 输入密码: Test123456<br>4. 点击"登录"按钮 | 1. 页面跳转到首页<br>2. 显示用户头像<br>3. 显示"欢迎回来"提示 |
| TC-LOGIN-002 | 记住登录状态 | P0 | 用户已注册 | 1. 勾选"记住我"<br>2. 输入正确账号密码<br>3. 登录成功<br>4. 关闭浏览器<br>5. 重新打开登录页面 | 自动填充用户名，无需重新登录 |

### P1 - 核心功能测试

| 用例编号 | 用例标题 | 优先级 | 前置条件 | 测试步骤 | 预期结果 |
|----------|----------|--------|----------|----------|----------|
| TC-LOGIN-003 | 错误密码登录失败 | P1 | 用户已注册账号 test@example.com | 1. 输入用户名: test@example.com<br>2. 输入密码: WrongPassword123<br>3. 点击"登录"按钮 | 1. 显示错误提示"用户名或密码错误"<br>2. 不跳转页面<br>3. 密码框清空 |
| TC-LOGIN-004 | 不存在账号登录 | P1 | 无 | 1. 输入用户名: notexist@example.com<br>2. 输入密码: Test123456<br>3. 点击"登录"按钮 | 显示错误提示"用户名或密码错误" |

### P2 - 边界值测试

| 用例编号 | 用例标题 | 优先级 | 前置条件 | 测试步骤 | 预期结果 |
|----------|----------|--------|----------|----------|----------|
| TC-LOGIN-005 | 空用户名登录 | P2 | 无 | 1. 用户名留空<br>2. 输入密码: Test123456<br>3. 点击"登录"按钮 | 显示提示"请输入用户名" |
| TC-LOGIN-006 | 空密码登录 | P2 | 无 | 1. 输入用户名: test@example.com<br>2. 密码留空<br>3. 点击"登录"按钮 | 显示提示"请输入密码" |
| TC-LOGIN-007 | 超长用户名输入 | P2 | 无 | 1. 输入超长用户名(256字符)<br>2. 输入正确密码<br>3. 点击"登录"按钮 | 显示提示"用户名长度不能超过100字符" |
```
```

---

#### 4.3.2 用例评审提示词（优化版）

```markdown
# 测试用例评审专家 - 系统提示词

## 角色定位
你是一位资深测试架构师，拥有丰富的测试用例评审经验。你的职责是对生成的测试用例进行全面、深入的评审，发现问题并提出改进建议。

## 评审维度

### 1. 覆盖率评审
检查测试用例是否覆盖了需求的各个方面：
- ✅ 功能完整性
- ✅ 业务流程覆盖
- ✅ 正常路径覆盖
- ✅ 异常路径覆盖
- ✅ 边界值覆盖
- ✅ 非功能需求覆盖（性能、安全、兼容性）

### 2. 正确性评审
检查测试用例的技术正确性：
- ✅ 测试步骤逻辑正确
- ✅ 测试数据合理
- ✅ 预期结果可验证
- ✅ 依赖关系明确
- ✅ 无歧义描述

### 3. 可执行性评审
检查测试用例的执行可行性：
- ✅ 环境要求明确
- ✅ 步骤可操作
- ✅ 无外部依赖或依赖已满足
- ✅ 执行顺序合理

### 4. 可维护性评审
检查测试用例的可维护性：
- ✅ 编号规范
- ✅ 命名清晰
- ✅ 无重复用例
- ✅ 模块划分合理

## 评审输出格式

### 评审摘要
```markdown
## 评审摘要

| 评审维度 | 评分 | 说明 |
|----------|------|------|
| 覆盖率 | 85% | 缺少性能测试用例 |
| 正确性 | 90% | 步骤清晰准确 |
| 可执行性 | 95% | 依赖说明完整 |
| 可维护性 | 88% | 个别编号不规范 |

**综合评分**: 89/100
```

### 问题列表
```markdown
## 发现的问题

### 🔴 严重问题

| 序号 | 用例编号 | 问题类型 | 问题描述 | 建议修改 |
|------|----------|----------|----------|----------|
| 1 | TC-PAY-003 | 覆盖缺失 | 未覆盖支付密码错误3次锁定场景 | 添加支付密码锁定测试用例 |
| 2 | TC-ORDER-007 | 逻辑错误 | 预期结果与实际流程不符 | 修改预期结果为"订单状态变为已取消" |

### 🟡 中等问题

| 序号 | 用例编号 | 问题类型 | 问题描述 | 建议修改 |
|------|----------|----------|----------|----------|
| 1 | TC-LOGIN-005 | 边界值 | 未测试最大用户名长度 | 添加长度=100字符的测试用例 |
| 2 | TC-USER-012 | 数据不足 | 测试数据过于简单 | 使用更真实的测试数据 |

### 🟢 建议改进

| 序号 | 用例编号 | 问题类型 | 建议 |
|------|----------|----------|------|
| 1 | TC-*-* | 命名规范 | 建议使用更清晰的模块前缀 |
| 2 | TC-*-* | 文档完善 | 建议添加用例间的依赖说明 |
```

### 改进建议
```markdown
## 总体改进建议

1. **覆盖率提升**
   - 建议添加性能测试用例（响应时间、并发处理）
   - 建议添加安全测试用例（SQL注入、XSS攻击）
   - 建议添加兼容性测试用例（浏览器、设备）

2. **用例优化**
   - 统一编号格式，使用模块前缀
   - 完善前置条件说明
   - 细化预期结果的验证点

3. **文档完善**
   - 添加测试数据字典
   - 添加测试环境要求说明
   - 完善用例间的依赖关系图
```

### 通过评审的用例
以下用例通过评审，可以直接使用：
- TC-LOGIN-001: 正确账号密码登录成功
- TC-LOGIN-002: 记住登录状态
- TC-LOGIN-004: 不存在账号登录
- TC-PAY-001: 正常支付流程

---

## 5. 完整配置示例

### 5.1 前端配置页面使用

#### AI 模型配置页面
访问路径: `配置中心 → AI 模型配置`

```javascript
// 推荐的配置组合

// 编写专家配置
{
  name: "DeepSeek-测试用例编写",
  model_type: "deepseek",
  role: "writer",
  api_key: "sk-xxxxxxxxxxxxxxxx",
  base_url: "https://api.deepseek.com",
  model_name: "deepseek-chat",
  max_tokens: 4096,
  temperature: 0.7,
  top_p: 0.9,
  is_active: true
}

// 评审专家配置
{
  name: "DeepSeek-用例评审",
  model_type: "deepseek",
  role: "reviewer",
  api_key: "sk-xxxxxxxxxxxxxxxx",
  base_url: "https://api.deepseek.com",
  model_name: "deepseek-chat",
  max_tokens: 4096,
  temperature: 0.5,
  top_p: 0.9,
  is_active: true
}
```

#### 提示词配置页面
访问路径: `需求分析 → 提示词配置`

```javascript
// 编写提示词配置
{
  name: "标准测试用例提示词",
  prompt_type: "writer",
  content: "# 测试用例编写专家...\n\n## 完整提示词内容见上方",
  is_active: true
}

// 评审提示词配置
{
  name: "标准评审提示词",
  prompt_type: "reviewer",
  content: "# 测试用例评审专家...\n\n## 完整提示词内容见上方",
  is_active: true
}
```

### 5.2 API 调用示例

```python
# 调用 AI 服务生成测试用例
import asyncio
from apps.requirement_analysis.ai_models import AIModelService, AIModelConfig, PromptConfig

async def generate_test_cases():
    # 假设从数据库获取配置
    model_config = AIModelConfig.objects.get(role='writer', is_active=True)
    prompt_config = PromptConfig.objects.get(prompt_type='writer', is_active=True)
    
    # 构建消息
    messages = [
        {"role": "system", "content": prompt_config.content},
        {"role": "user", "content": "请根据以下需求生成测试用例：\n用户登录功能需求..."}
    ]
    
    # 调用服务
    result = await AIModelService.call_openai_compatible_api(model_config, messages)
    
    return result['choices'][0]['message']['content']

# 执行
result = asyncio.run(generate_test_cases())
print(result)
```

---

## 6. 高级配置

### 6.1 多模型协作

```python
# 使用不同模型处理不同任务
class MultiModelCoordinator:
    """多模型协调器"""
    
    # 快速草稿生成 - 使用快速模型
    DRAFT_MODEL = {
        'type': 'qwen',
        'name': 'qwen-turbo',
        'temperature': 0.8,
        'max_tokens': 2048
    }
    
    # 精细生成 - 使用强模型
    FINE_GRAIN_MODEL = {
        'type': 'qwen',
        'name': 'qwen-plus',
        'temperature': 0.7,
        'max_tokens': 4096
    }
    
    # 评审优化 - 使用批判性模型
    REVIEW_MODEL = {
        'type': 'deepseek',
        'name': 'deepseek-chat',
        'temperature': 0.5,
        'max_tokens': 4096
    }
```

### 6.2 流式响应配置

```python
# 支持 SSE 流式输出
async def generate_streaming_test_cases(task):
    model_config = task.writer_model_config
    
    data = {
        'model': model_config.model_name,
        'messages': messages,
        'stream': True  # 启用流式
    }
    
    async with httpx.AsyncClient(timeout=120.0) as client:
        async with client.stream('POST', url, json=data) as response:
            async for line in response.aiter_lines():
                if line.startswith('data: '):
                    yield json.loads(line[6:])
```

### 6.3 重试机制

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
async def call_with_retry(config, messages):
    """带重试的 API 调用"""
    return await AIModelService.call_openai_compatible_api(config, messages)
```

---

## 7. 常见问题

### Q1: API 调用返回 401 错误
**原因**: API Key 无效或过期
**解决**: 
1. 检查 API Key 是否正确
2. 确认 API Key 是否有效
3. 检查账户余额

### Q2: URL 配置不正确
**原因**: Base URL 格式错误
**解决**:
- DeepSeek: `https://api.deepseek.com`
- 通义千问: `https://dashscope.aliyuncs.com`
- 硅基流动: `https://api.siliconflow.cn`

### Q3: 输出格式不规范
**原因**: 提示词不够详细
**解决**: 参考上方的完整提示词模板优化提示词

### Q4: 生成内容被截断
**原因**: max_tokens 设置过小
**解决**: 增加 max_tokens 值到 4096 或更高

### Q5: 输出不稳定
**原因**: temperature 设置过高
**解决**: 降低 temperature 到 0.5-0.7

### Q6: 表格格式错位
**原因**: 内容中包含未转义的管道符
**解决**: 确保提示词中包含管道符转义规则，并在生成结果后进行预处理

---

*文档更新时间: 2026-04-12*

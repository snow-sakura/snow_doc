# TestHub 后端模块设计文档

> 本文档用于提交给AI，让AI根据后端模块设计对应的前端页面
> 代码路径：`/Users/snow/snow-sakura/snow-python/testhub_platform`

---

## 一、模块总览

| 模块名称 | 应用路径 | 主要功能 | 核心数据模型 |
|---------|---------|---------|------------|
| 用户认证 | `apps/users/` | 用户管理、认证授权、个人配置 | User, UserProfile |
| 项目管理 | `apps/projects/` | 项目创建、成员管理、环境配置 | Project, ProjectMember, ProjectEnvironment |
| 测试用例 | `apps/testcases/` | 用例编写、步骤管理、评论协作 | TestCase, TestCaseStep, TestCaseAttachment, TestCaseComment |
| 测试套件 | `apps/testsuites/` | 用例套件管理、分类组织 | TestSuite, TestSuiteMembership |
| 测试执行 | `apps/executions/` | 测试计划、用例执行、结果记录 | TestPlan, TestRun, TestRunCase, TestRunCaseHistory |
| 测试报告 | `apps/reports/` | 执行报告生成、结果分析 | TestReport, ReportArtifact |
| 测试评审 | `apps/reviews/` | 用例评审流程、评审意见 | TestReview, ReviewComment |
| 版本管理 | `apps/versions/` | 版本规划、版本对比 | Version, VersionMilestone |
| AI助手 | `apps/assistant/` | AI对话、知识库管理 | DifyConfig, AssistantSession, ChatMessage, KnowledgeBase |
| 需求分析 | `apps/requirement_analysis/` | AI需求分析、用例自动生成 | RequirementDocument, RequirementAnalysis, BusinessRequirement, GeneratedTestCase |
| API测试 | `apps/api_testing/` | API接口测试、自动化测试 | ApiProject, ApiCollection, ApiRequest, Environment, TestSuite, TestExecution |
| UI自动化 | `apps/ui_automation/` | Web UI自动化测试 | UiProject, Element, ElementGroup, TestScript, PageObject, TestSuite |
| APP自动化 | `apps/app_automation/` | 移动APP自动化测试 | AppProject, AppDevice, AppElement, AppTestSuite, AppTestExecution |
| 数据工厂 | `apps/data_factory/` | 测试数据生成 | DataFactoryRecord |
| 核心模块 | `apps/core/` | 通知配置、系统设置 | UnifiedNotificationConfig |

---

## 二、用户认证模块 (`apps/users/`)

### 2.1 模块作用
提供完整的用户认证和权限管理功能，包括用户注册登录、JWT令牌管理、个人资料维护、主题语言偏好设置。

### 2.2 核心功能
- **用户管理**：用户注册、登录、登出、密码修改
- **JWT认证**：Token生成与刷新
- **个人配置**：主题、语言、时区、通知偏好
- **用户资料**：头像、电话、部门、职位

### 2.3 数据模型

```
User (继承AbstractUser)
├── id (UUID, 主键)
├── username (用户名)
├── email (邮箱)
├── password (密码)
├── avatar (头像URL)
├── phone (电话)
├── department (部门)
├── position (职位)
├── is_active (是否激活)
├── created_at (创建时间)
└── updated_at (更新时间)

UserProfile (用户扩展配置)
├── id (UUID, 主键)
├── user (OneToOne → User)
├── theme (主题: light/dark)
├── language (语言: zh-CN/en)
├── timezone (时区)
└── notifications (JSON, 通知配置)
```

### 2.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/users/register/` | POST | 用户注册 |
| `/api/users/login/` | POST | 用户登录 |
| `/api/users/logout/` | POST | 用户登出 |
| `/api/users/profile/` | GET/PUT/PATCH | 获取/更新个人资料 |
| `/api/users/change-password/` | POST | 修改密码 |
| `/api/users/token/refresh/` | POST | 刷新Token |

### 2.5 前端设计要点
- 登录注册表单：支持邮箱/用户名登录
- 个人中心页面：头像上传、资料编辑、偏好设置
- 设置面板：主题切换、语言选择、通知开关

---

## 三、项目管理模块 (`apps/projects/`)

### 3.1 模块作用
管理系统中的测试项目，支持多项目隔离、项目成员协作、环境配置管理。

### 3.2 核心功能
- **项目管理**：项目创建、编辑、删除、归档
- **成员管理**：成员邀请、角色分配、权限控制
- **环境配置**：测试/预发布/生产等多环境管理
- **项目统计**：用例数量、执行进度、缺陷统计

### 3.3 数据模型

```
Project (项目)
├── id (UUID, 主键)
├── name (项目名称)
├── description (项目描述)
├── status (状态: active/archived)
├── owner (外键 → User)
├── members (多对多 → User, through ProjectMember)
├── created_at
└── updated_at

ProjectMember (项目成员)
├── id (UUID, 主键)
├── project (外键 → Project)
├── user (外键 → User)
├── role (角色: owner/admin/member/viewer)
├── joined_at
└── invited_by (外键 → User)

ProjectEnvironment (项目环境)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (环境名称: dev/staging/prod)
├── base_url (基础URL)
├── variables (JSON, 环境变量)
├── is_default (是否默认)
└── created_at
```

### 3.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/projects/` | GET/POST | 列表/创建项目 |
| `/api/projects/{id}/` | GET/PUT/DELETE | 项目详情/更新/删除 |
| `/api/projects/{id}/members/` | GET/POST | 成员列表/添加成员 |
| `/api/projects/{id}/members/{user_id}/` | DELETE | 移除成员 |
| `/api/projects/{id}/environments/` | GET/POST | 环境列表/创建环境 |
| `/api/projects/{id}/environments/{env_id}/` | GET/PUT/DELETE | 环境操作 |
| `/api/projects/{id}/stats/` | GET | 项目统计 |

### 3.5 前端设计要点
- 项目卡片展示：名称、描述、成员头像、状态标签
- 成员管理弹窗：角色选择、权限说明
- 环境配置面板：变量表格编辑、URL输入
- 项目仪表盘：统计数据可视化

---

## 四、测试用例模块 (`apps/testcases/`)

### 4.1 模块作用
管理测试用例库，支持用例编写、步骤拆解、附件上传、评论协作。

### 4.2 核心功能
- **用例管理**：创建、编辑、复制、删除、导入导出
- **步骤管理**：用例步骤拆分、预期结果设置
- **附件支持**：上传截图、文档等附件
- **评论协作**：用例讨论、评审意见
- **标签分类**：用例分类、优先级、状态管理
- **批量操作**：批量编辑、批量移动

### 4.3 数据模型

```
TestCase (测试用例)
├── id (UUID, 主键)
├── title (用例标题)
├── project (外键 → Project)
├── module (外键 → TestModule, nullable)
├── priority (优先级: P0/P1/P2/P3)
├── type (类型: functional/performance/security)
├── precondition (前置条件)
├── created_by (外键 → User)
├── assigned_to (外键 → User, nullable)
├── status (状态: draft/active/deprecated)
├── version (版本号)
├── created_at
└── updated_at

TestCaseStep (用例步骤)
├── id (UUID, 主键)
├── testcase (外键 → TestCase)
├── step_number (步骤序号)
├── action (操作描述)
├── expected_result (预期结果)
├── test_data (测试数据)
└── order (排序)

TestCaseAttachment (用例附件)
├── id (UUID, 主键)
├── testcase (外键 → TestCase)
├── file_name (文件名)
├── file_path (文件路径)
├── file_type (文件类型)
├── file_size (文件大小)
└── uploaded_by (外键 → User)

TestCaseComment (用例评论)
├── id (UUID, 主键)
├── testcase (外键 → TestCase)
├── user (外键 → User)
├── content (评论内容)
├── parent (自关联, nullable)
├── created_at
└── updated_at

TestModule (用例模块/目录)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (模块名称)
├── parent (自关联, nullable)
├── order (排序)
└── created_at
```

### 4.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/testcases/` | GET/POST | 用例列表/创建 |
| `/api/testcases/{id}/` | GET/PUT/DELETE | 用例详情/更新/删除 |
| `/api/testcases/{id}/steps/` | GET/POST | 用例步骤 |
| `/api/testcases/{id}/attachments/` | GET/POST | 附件管理 |
| `/api/testcases/{id}/comments/` | GET/POST | 评论列表/添加 |
| `/api/testcases/bulk-update/` | POST | 批量更新 |
| `/api/testcases/import/` | POST | Excel导入 |
| `/api/testcases/export/` | GET | Excel导出 |
| `/api/modules/` | GET/POST | 模块列表/创建 |

### 4.5 前端设计要点
- 用例列表页：表格视图、树形视图切换
- 用例编辑器：富文本编辑、步骤拖拽排序
- 步骤编辑卡片：操作/预期结果/数据分离
- 批量操作工具栏：勾选、批量移动/删除/标签
- 评论气泡：对话式评论、@提及

---

## 五、测试套件模块 (`apps/testsuites/`)

### 5.1 模块作用
组织和分组测试用例，支持套件分类、动态用例选择、套件执行。

### 5.2 核心功能
- **套件管理**：创建、编辑、删除测试套件
- **用例分组**：将用例按模块/功能分组
- **动态筛选**：基于标签、模块、创建者等筛选用例
- **执行统计**：套件内用例通过/失败统计

### 5.3 数据模型

```
TestSuite (测试套件)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (套件名称)
├── description (描述)
├── filter_criteria (JSON, 动态筛选条件)
├── included_cases (多对多 → TestCase, through TestSuiteMembership)
├── created_by (外键 → User)
├── created_at
└── updated_at

TestSuiteMembership (套件-用例关联)
├── id (UUID, 主键)
├── testsuite (外键 → TestSuite)
├── testcase (外键 → TestCase)
└── order (排序)
```

### 5.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/testsuites/` | GET/POST | 套件列表/创建 |
| `/api/testsuites/{id}/` | GET/PUT/DELETE | 套件详情/更新/删除 |
| `/api/testsuites/{id}/cases/` | GET/POST/DELETE | 套件用例管理 |
| `/api/testsuites/{id}/execute/` | POST | 执行套件 |

### 5.5 前端设计要点
- 套件卡片：名称、描述、包含用例数量、状态
- 用例选择器：多选表格、支持拖拽
- 筛选器：标签选择、模块树、优先级过滤

---

## 六、测试执行模块 (`apps/executions/`)

### 6.1 模块作用
管理测试计划和执行，支持计划创建、定时执行、实时进度、结果记录。

### 6.2 核心功能
- **测试计划**：创建执行计划、选择用例套件
- **手动执行**：立即执行选定用例
- **定时执行**：配置Cron表达式自动执行
- **执行进度**：实时显示执行状态
- **历史记录**：每次执行的结果对比
- **失败重跑**：仅重跑失败用例

### 6.3 数据模型

```
TestPlan (测试计划)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (计划名称)
├── description (描述)
├── test_suite (外键 → TestSuite, nullable)
├── environment (外键 → ProjectEnvironment)
├── scheduled_time (定时执行时间, nullable)
├── status (状态: draft/pending/running/completed/cancelled)
├── created_by (外键 → User)
├── created_at
└── updated_at

TestRun (测试执行记录)
├── id (UUID, 主键)
├── test_plan (外键 → TestPlan)
├── test_suite (外键 → TestSuite, nullable)
├── environment (外键 → ProjectEnvironment)
├── status (状态: pending/running/paused/completed)
├── started_at (开始时间)
├── completed_at (结束时间)
├── total_cases (总用例数)
├── passed_cases (通过数)
├── failed_cases (失败数)
├── blocked_cases (阻塞数)
├── created_by (外键 → User)
└── trigger_type (触发类型: manual/scheduled/api)

TestRunCase (执行-用例记录)
├── id (UUID, 主键)
├── test_run (外键 → TestRun)
├── test_case (外键 → TestCase)
├── test_step (外键 → TestCaseStep, nullable)
├── status (状态: pending/running/passed/failed/blocked/skipped)
├── actual_result (实际结果)
├── duration (执行耗时, 秒)
├── screenshot (截图路径, nullable)
├── logs (JSON, 执行日志)
├── executed_at (执行时间)
└── executed_by (外键 → User)

TestRunCaseHistory (执行历史快照)
├── id (UUID, 主键)
├── test_run_case (外键 → TestRunCase)
├── snapshot_data (JSON, 历史数据快照)
├── created_at
```

### 6.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/plans/` | GET/POST | 测试计划列表/创建 |
| `/api/plans/{id}/` | GET/PUT/DELETE | 计划详情/更新/删除 |
| `/api/plans/{id}/execute/` | POST | 立即执行计划 |
| `/api/plans/{id}/cancel/` | POST | 取消执行 |
| `/api/runs/` | GET | 执行记录列表 |
| `/api/runs/{id}/` | GET | 执行详情 |
| `/api/runs/{id}/cases/` | GET | 执行用例列表 |
| `/api/runs/{id}/cases/{case_id}/` | PUT | 更新用例执行状态 |
| `/api/runs/{id}/retry-failed/` | POST | 重跑失败用例 |
| `/api/runs/{id}/stop/` | POST | 停止执行 |

### 6.5 前端设计要点
- 测试计划日历：可视化定时任务
- 执行仪表盘：实时进度环形图、统计数据
- 执行详情页：步骤级通过/失败状态
- 用例执行卡片：状态图标、截图预览、日志展开
- 历史对比视图：多次执行结果差异高亮

---

## 七、测试报告模块 (`apps/reports/`)

### 7.1 模块作用
生成和管理测试报告，提供详细的执行分析报告、趋势图表、导出分享功能。

### 7.2 核心功能
- **报告生成**：基于执行记录自动生成报告
- **趋势分析**：通过率趋势、缺陷趋势
- **报告导出**：PDF/HTML/Excel格式导出
- **报告分享**：生成分享链接、邮件推送
- **自定义模板**：报告模板配置

### 7.3 数据模型

```
TestReport (测试报告)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (报告名称)
├── test_run (外键 → TestRun)
├── summary (JSON, 汇总数据)
├── statistics (JSON, 统计数据)
├── charts (JSON, 图表配置)
├── created_by (外键 → User)
├── created_at
└── updated_at

ReportArtifact (报告产物)
├── id (UUID, 主键)
├── report (外键 → TestReport)
├── file_name (文件名)
├── file_path (文件路径)
├── file_type (类型: pdf/html/excel)
├── file_size (文件大小)
└── created_at
```

### 7.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/reports/` | GET/POST | 报告列表/创建 |
| `/api/reports/{id}/` | GET/DELETE | 报告详情/删除 |
| `/api/reports/{id}/export/` | GET | 导出报告 |
| `/api/reports/{id}/share/` | POST | 生成分享链接 |
| `/api/reports/trend/` | GET | 趋势数据 |

### 7.5 前端设计要点
- 报告卡片列表：名称、执行时间、通过率徽章
- 报告详情页：摘要卡片、图表区域、详细表格
- 趋势图表：折线图、柱状图、饼图
- 导出面板：格式选择、模板预览

---

## 八、测试评审模块 (`apps/reviews/`)

### 8.1 模块作用
管理测试用例评审流程，支持评审创建、意见提交、评审状态跟踪。

### 8.2 核心功能
- **评审创建**：选择用例创建评审任务
- **评审流程**：待评审→评审中→已通过/已拒绝
- **意见反馈**：评审人提交具体意见
- **用例修订**：根据评审意见修改用例
- **评审历史**：查看评审记录和讨论

### 8.3 数据模型

```
TestReview (测试评审)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (评审名称)
├── description (描述)
├── test_cases (多对多 → TestCase)
├── reviewers (多对多 → User)
├── creator (外键 → User)
├── status (状态: pending/in_progress/approved/rejected)
├── deadline (截止日期)
├── created_at
└── updated_at

ReviewComment (评审意见)
├── id (UUID, 主键)
├── review (外键 → TestReview)
├── test_case (外键 → TestCase)
├── user (外键 → User)
├── line_number (行号/步骤号)
├── content (意见内容)
├── severity (严重程度: suggestion/major/critical)
├── status (状态: open/resolved/dismissed)
├── created_at
└── updated_at
```

### 8.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/reviews/` | GET/POST | 评审列表/创建 |
| `/api/reviews/{id}/` | GET/PUT/DELETE | 评审详情/更新 |
| `/api/reviews/{id}/cases/` | GET/POST | 评审用例管理 |
| `/api/reviews/{id}/comments/` | GET/POST | 评审意见 |
| `/api/reviews/{id}/approve/` | POST | 通过评审 |
| `/api/reviews/{id}/reject/` | POST | 拒绝评审 |

### 8.5 前端设计要点
- 评审看板：状态泳道（待评审/评审中/已完成）
- 用例评审视图：左侧用例列表、右侧评审意见区
- 意见标注：行号高亮、意见气泡
- 评审进度条：参与人、完成度统计

---

## 九、版本管理模块 (`apps/versions/`)

### 9.1 模块作用
管理项目版本规划，支持版本创建、里程碑设置、版本对比功能。

### 9.2 核心功能
- **版本规划**：创建版本、设置发布时间
- **里程碑**：版本内关键节点标记
- **用例关联**：用例与版本的关联管理
- **版本对比**：两个版本的用例差异对比
- **版本统计**：版本内用例覆盖率统计

### 9.3 数据模型

```
Version (版本)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (版本名称)
├── version_number (版本号: 1.0.0)
├── release_date (发布日期)
├── status (状态: planning/development/testing/released)
├── description (描述)
├── created_by (外键 → User)
├── created_at
└── updated_at

VersionMilestone (版本里程碑)
├── id (UUID, 主键)
├── version (外键 → Version)
├── name (里程碑名称)
├── target_date (目标日期)
├── status (状态: pending/in_progress/completed)
├── created_at
└── updated_at
```

### 9.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/versions/` | GET/POST | 版本列表/创建 |
| `/api/versions/{id}/` | GET/PUT/DELETE | 版本详情/更新 |
| `/api/versions/{id}/milestones/` | GET/POST | 里程碑管理 |
| `/api/versions/compare/` | GET | 版本对比 |

### 9.5 前端设计要点
- 版本时间线：横向时间轴展示版本规划
- 版本详情页：里程碑进度、用例统计
- 版本对比视图：左右两栏用例列表、差异高亮

---

## 十、AI助手模块 (`apps/assistant/`)

### 10.1 模块作用
提供AI智能对话功能，集成Dify等AI平台，支持知识库管理、上下文对话。

### 10.2 核心功能
- **AI对话**：与AI进行自然语言交流
- **上下文记忆**：支持多轮对话、话题延续
- **知识库集成**：关联业务知识库提升回答准确性
- **会话管理**：历史会话、收藏对话
- **Dify集成**：对接Dify AI平台

### 10.3 数据模型

```
DifyConfig (Dify配置)
├── id (UUID, 主键)
├── project (外键 → Project, nullable)
├── name (配置名称)
├── api_key (API密钥)
├── base_url (Dify服务地址)
├── app_id (应用ID)
├── is_default (是否默认)
├── created_by (外键 → User)
├── created_at
└── updated_at

AssistantSession (AI会话)
├── id (UUID, 主键)
├── project (外键 → Project, nullable)
├── user (外键 → User)
├── title (会话标题)
├── config (外键 → DifyConfig)
├── context (JSON, 上下文数据)
├── is_pinned (是否置顶)
├── created_at
└── updated_at

ChatMessage (聊天消息)
├── id (UUID, 主键)
├── session (外键 → AssistantSession)
├── role (角色: user/assistant/system)
├── content (消息内容)
├── metadata (JSON, 元数据)
├── created_at
└── updated_at

KnowledgeBase (知识库)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (知识库名称)
├── description (描述)
├── documents (JSON, 文档列表)
├── created_by (外键 → User)
├── created_at
└── updated_at
```

### 10.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/assistant/sessions/` | GET/POST | 会话列表/创建 |
| `/api/assistant/sessions/{id}/` | GET/DELETE | 会话详情/删除 |
| `/api/assistant/sessions/{id}/messages/` | GET/POST | 消息列表/发送 |
| `/api/assistant/configs/` | GET/POST | Dify配置管理 |
| `/api/assistant/knowledge/` | GET/POST | 知识库管理 |
| `/api/assistant/chat/` | POST | 发送消息(流式) |

### 10.5 前端设计要点
- 聊天界面：消息气泡、Markdown渲染、代码高亮
- 会话列表：侧边栏、搜索、会话分组
- 配置面板：API Key输入、连接测试
- 知识库管理：文档上传、向量库状态

---

## 十一、需求分析模块 (`apps/requirement_analysis/`)

### 11.1 模块作用
利用AI自动分析需求文档，生成测试用例，支持需求结构化、用例智能生成、覆盖率分析。

### 11.2 核心功能
- **需求上传**：支持多种格式（txt/md/docx/pdf）
- **AI解析**：自动提取业务需求、功能点、非功能需求
- **用例生成**：基于需求自动生成测试用例
- **覆盖率分析**：分析需求覆盖情况
- **用例关联**：用例与需求的关联映射
- **批量生成**：支持批量处理大量需求

### 11.3 数据模型

```
RequirementDocument (需求文档)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (文档名称)
├── file_path (文件路径)
├── file_type (文件类型)
├── content (文档内容)
├── status (状态: pending/processing/completed/failed)
├── created_by (外键 → User)
├── created_at
└── updated_at

RequirementAnalysis (需求分析结果)
├── id (UUID, 主键)
├── document (外键 → RequirementDocument)
├── summary (JSON, 需求摘要)
├── business_requirements (JSON, 业务需求列表)
├── functional_requirements (JSON, 功能需求列表)
├── non_functional_requirements (JSON, 非功能需求)
├── created_at
└── updated_at

BusinessRequirement (业务需求)
├── id (UUID, 主键)
├── analysis (外键 → RequirementAnalysis)
├── requirement_id (需求ID)
├── title (标题)
├── description (描述)
├── priority (优先级)
├── type (类型)
├── acceptance_criteria (JSON, 验收标准)
├── created_at
└── updated_at

GeneratedTestCase (AI生成的测试用例)
├── id (UUID, 主键)
├── document (外键 → RequirementDocument)
├── requirement (外键 → BusinessRequirement, nullable)
├── title (用例标题)
├── priority (优先级)
├── precondition (前置条件)
├── steps (JSON, 测试步骤)
├── expected_result (预期结果)
├── ai_model (AI模型名称)
├── confidence (置信度)
├── status (状态: draft/approved/rejected/merged)
├── source (来源: requirement/testcase_review)
├── created_at
└── updated_at

AnalysisTask (分析任务)
├── id (UUID, 主键)
├── project (外键 → Project)
├── document (外键 → RequirementDocument)
├── status (状态: pending/running/completed/failed)
├── progress (进度百分比)
├── error_message (错误信息)
├── started_at
├── completed_at
└── created_at

AIModelConfig (AI模型配置)
├── id (UUID, 主键)
├── project (外键 → Project, nullable)
├── name (配置名称)
├── provider (提供商: openai/anthropic/deepseek/dify)
├── model_name (模型名称)
├── api_key (API密钥)
├── base_url (API地址)
├── parameters (JSON, 模型参数)
├── is_default (是否默认)
├── created_by (外键 → User)
├── created_at
└── updated_at

PromptConfig (提示词配置)
├── id (UUID, 主键)
├── project (外键 → Project, nullable)
├── name (配置名称)
├── prompt_type (类型: analysis/generation/review)
├── system_prompt (系统提示词)
├── user_prompt_template (用户提示模板)
├── parameters (JSON, 参数定义)
├── is_active (是否启用)
├── created_at
└── updated_at

GenerationConfig (用例生成配置)
├── id (UUID, 主键)
├── project (外键 → Project)
├── name (配置名称)
├── model_config (外键 → AIModelConfig)
├── prompt_config (外键 → PromptConfig)
├── generation_rules (JSON, 生成规则)
├── output_format (输出格式)
├── is_active (是否启用)
├── created_at
└── updated_at

TestCaseGenerationTask (用例生成任务)
├── id (UUID, 主键)
├── project (外键 → Project)
├── requirement (外键 → RequirementDocument)
├── config (外键 → GenerationConfig)
├── status (状态: pending/running/completed/failed)
├── total_count (总生成数)
├── success_count (成功数)
├── failed_count (失败数)
├── generated_cases (多对多 → GeneratedTestCase)
├── error_message (错误信息)
├── started_at
├── completed_at
└── created_at
```

### 11.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/requirement/documents/` | GET/POST | 文档列表/上传 |
| `/api/requirement/documents/{id}/` | GET/DELETE | 文档详情/删除 |
| `/api/requirement/documents/{id}/analyze/` | POST | 触发分析 |
| `/api/requirement/documents/{id}/analysis/` | GET | 分析结果 |
| `/api/requirement/generate/` | POST | 批量生成用例 |
| `/api/requirement/generated-cases/` | GET | 生成用例列表 |
| `/api/requirement/generated-cases/{id}/` | GET/PUT | 用例详情/更新 |
| `/api/requirement/generated-cases/{id}/approve/` | POST | 审核通过 |
| `/api/requirement/generated-cases/{id}/reject/` | POST | 审核拒绝 |
| `/api/requirement/generated-cases/{id}/merge/` | POST | 合并到用例库 |
| `/api/requirement/configs/models/` | GET/POST | AI模型配置 |
| `/api/requirement/configs/prompts/` | GET/POST | 提示词配置 |
| `/api/requirement/configs/generation/` | GET/POST | 生成配置 |

### 11.5 前端设计要点
- 需求文档上传区：拖拽上传、格式提示、进度条
- 分析进度面板：实时状态、解析步骤展示
- 需求结构树：层级展示业务需求→功能点
- 用例生成卡片：预览、编辑、批量操作
- 覆盖率热力图：需求与用例的关联矩阵
- AI模型配置：多模型切换、参数调整

---

## 十二、API测试模块 (`apps/api_testing/`)

### 12.1 模块作用
提供API接口自动化测试功能，支持接口管理、请求调试、环境变量、测试套件、定时任务。

### 12.2 核心功能
- **项目管理**：API项目独立管理
- **集合管理**：按模块/功能组织API
- **请求调试**：完整的HTTP请求构建器
- **环境变量**：多环境配置、变量引用
- **前置/后置脚本**：JavaScript脚本处理
- **测试套件**：用例化测试编排
- **定时任务**：自动化回归测试
- **Mock服务**：本地Mock服务器
- **结果报告**：详细执行结果和日志

### 12.3 数据模型

```
ApiProject (API项目)
├── id (UUID, 主键)
├── name (项目名称)
├── description (描述)
├── base_url (基础URL)
├── global_headers (JSON, 全局请求头)
├── created_by (外键 → User)
├── created_at
└── updated_at

ApiCollection (API集合/文件夹)
├── id (UUID, 主键)
├── project (外键 → ApiProject)
├── name (集合名称)
├── description (描述)
├── parent (自关联, nullable)
├── order (排序)
└── created_at

ApiRequest (API请求)
├── id (UUID, 主键)
├── collection (外键 → ApiCollection)
├── name (请求名称)
├── method (HTTP方法: GET/POST/PUT/DELETE/PATCH)
├── endpoint (请求路径)
├── description (描述)
├── headers (JSON, 请求头)
├── query_params (JSON, 查询参数)
├── path_params (JSON, 路径参数)
├── body_type (body类型: none/json/form-data/raw/binary)
├── body_content (请求体内容)
├── pre_request_script (前置脚本)
├── test_script (测试脚本)
├── expected_status (预期状态码)
├── timeout (超时时间, 秒)
├── order (排序)
├── created_by (外键 → User)
├── created_at
└── updated_at

Environment (环境配置)
├── id (UUID, 主键)
├── project (外键 → ApiProject)
├── name (环境名称: dev/staging/prod)
├── variables (JSON, 环境变量)
├── is_active (是否启用)
├── created_at
└── updated_at

RequestHistory (请求历史)
├── id (UUID, 主键)
├── api_request (外键 → ApiRequest)
├── environment (外键 → Environment, nullable)
├── request_data (JSON, 请求详情)
├── response_data (JSON, 响应详情)
├── status_code (状态码)
├── duration (响应时间, ms)
├── test_results (JSON, 测试结果)
├── created_at
└── created_by (外键 → User)

TestSuite (API测试套件)
├── id (UUID, 主键)
├── project (外键 → ApiProject)
├── name (套件名称)
├── description (描述)
├── requests (多对多 → ApiRequest)
├── pre_script (套件前置脚本)
├── post_script (套件后置脚本)
├── created_by (外键 → User)
├── created_at
└── updated_at

TestExecution (API测试执行)
├── id (UUID, 主键)
├── test_suite (外键 → TestSuite)
├── environment (外键 → Environment)
├── status (状态: pending/running/completed/failed)
├── total_requests (总请求数)
├── passed (通过数)
├── failed (失败数)
├── results (JSON, 执行结果详情)
├── duration (总耗时, ms)
├── executed_at
└── executed_by (外键 → User)

ScheduledTask (定时任务)
├── id (UUID, 主键)
├── project (外键 → ApiProject)
├── test_suite (外键 → TestSuite)
├── environment (外键 → Environment)
├── name (任务名称)
├── cron_expression (Cron表达式)
├── is_active (是否启用)
├── last_run (上次执行时间)
├── next_run (下次执行时间)
├── notifications (JSON, 通知配置)
├── created_by (外键 → User)
├── created_at
└── updated_at
```

### 12.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/api_testing/projects/` | GET/POST | API项目列表/创建 |
| `/api/api_testing/projects/{id}/` | GET/PUT/DELETE | 项目详情 |
| `/api/api_testing/collections/` | GET/POST | 集合列表/创建 |
| `/api/api_testing/collections/{id}/` | GET/PUT/DELETE | 集合详情 |
| `/api/api_testing/requests/` | GET/POST | 请求列表/创建 |
| `/api/api_testing/requests/{id}/` | GET/PUT/DELETE | 请求详情 |
| `/api/api_testing/requests/{id}/send/` | POST | 发送请求 |
| `/api/api_testing/environments/` | GET/POST | 环境列表/创建 |
| `/api/api_testing/environments/{id}/` | GET/PUT/DELETE | 环境详情 |
| `/api/api_testing/suites/` | GET/POST | 测试套件列表/创建 |
| `/api/api_testing/suites/{id}/` | GET/PUT/DELETE | 套件详情 |
| `/api/api_testing/suites/{id}/execute/` | POST | 执行套件 |
| `/api/api_testing/executions/` | GET | 执行记录列表 |
| `/api/api_testing/executions/{id}/` | GET | 执行详情 |
| `/api/api_testing/scheduled/` | GET/POST | 定时任务管理 |
| `/api/api_testing/history/` | GET | 请求历史 |

### 12.5 前端设计要点
- API列表视图：方法标签(GET绿色/POST蓝色等)、路径、状态
- 请求构建器：Tab切换(Params/Headers/Body/Auth/Scripts)
- 环境变量面板：左侧边栏、变量高亮显示
- 响应查看器：Pretty/Raw/Preview多视图、Header/Body/Cookie分Tab
- 测试脚本编辑器：语法高亮、控制台输出
- 定时任务日历：Cron可视化配置
- 执行报告：火焰图、时间线、失败定位

---

## 十三、UI自动化模块 (`apps/ui_automation/`)

### 13.1 模块作用
提供Web UI自动化测试功能，支持元素定位、页面对象模型、脚本录制、跨浏览器测试。

### 13.2 核心功能
- **项目管理**：UI自动化项目独立管理
- **元素库**：统一管理页面元素定位器
- **定位策略**：支持多种定位方式(xpath/css/id/text)
- **页面对象**：POM设计模式支持
- **元素分组**：按模块/页面组织元素
- **测试脚本**：Python/JavaScript脚本编写
- **执行引擎**：基于Selenium/Playwright
- **跨浏览器**：Chrome/Firefox/Safari多浏览器支持
- **测试套件**：用例编排和执行
- **执行报告**：截图、日志、录屏

### 13.3 数据模型

```
UiProject (UI自动化项目)
├── id (UUID, 主键)
├── name (项目名称)
├── description (描述)
├── framework (框架: selenium/playwright)
├── base_url (基础URL)
├── default_browser (默认浏览器)
├── created_by (外键 → User)
├── created_at
└── updated_at

LocatorStrategy (定位策略配置)
├── id (UUID, 主键)
├── name (策略名称: xpath/css/id/name)
├── description (描述)
├── pattern (匹配模式)
├── priority (优先级)
└── created_at

Element (页面元素)
├── id (UUID, 主键)
├── project (外键 → UiProject)
├── name (元素名称)
├── description (描述)
├── page_object (外键 → PageObject, nullable)
├── locator_type (定位类型)
├── locator_value (定位值)
├── fallback_locators (JSON, 备用定位器)
├── timeout (超时时间, 秒)
├── created_at
└── updated_at

ElementGroup (元素分组)
├── id (UUID, 主键)
├── project (外键 → UiProject)
├── name (分组名称)
├── description (描述)
├── parent (自关联, nullable)
├── order (排序)
└── created_at

TestScript (测试脚本)
├── id (UUID, 主键)
├── project (外键 → UiProject)
├── name (脚本名称)
├── description (描述)
├── page_object (外键 → PageObject)
├── code (脚本代码)
├── language (语言: python/javascript)
├── preconditions (JSON, 前置条件)
├── test_data (JSON, 测试数据)
├── tags (JSON, 标签)
├── created_by (外键 → User)
├── created_at
└── updated_at

PageObject (页面对象)
├── id (UUID, 主键)
├── project (外键 → UiProject)
├── name (页面名称)
├── url_pattern (URL模式)
├── description (描述)
├── elements (多对多 → Element)
├── created_at
└── updated_at

TestSuite (UI测试套件)
├── id (UUID, 主键)
├── project (外键 → UiProject)
├── name (套件名称)
├── description (描述)
├── scripts (多对多 → TestScript)
├── browsers (JSON, 浏览器配置)
├── parallelism (并行数量)
├── retry_count (失败重试次数)
├── created_by (外键 → User)
├── created_at
└── updated_at

TestExecution (UI测试执行)
├── id (UUID, 主键)
├── test_suite (外键 → TestSuite)
├── status (状态: pending/running/completed/failed)
├── browser (浏览器类型)
├── total_scripts (总脚本数)
├── passed (通过数)
├── failed (失败数)
├── results (JSON, 执行结果)
├── screenshots (JSON, 截图列表)
├── logs (TEXT, 执行日志)
├── duration (总耗时, 秒)
├── executed_at
└── executed_by (外键 → User)
```

### 13.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/ui_automation/projects/` | GET/POST | 项目列表/创建 |
| `/api/ui_automation/projects/{id}/` | GET/PUT/DELETE | 项目详情 |
| `/api/ui_automation/elements/` | GET/POST | 元素列表/创建 |
| `/api/ui_automation/elements/{id}/` | GET/PUT/DELETE | 元素详情 |
| `/api/ui_automation/elements/validate/` | POST | 定位器验证 |
| `/api/ui_automation/groups/` | GET/POST | 元素分组 |
| `/api/ui_automation/pageobjects/` | GET/POST | 页面对象列表/创建 |
| `/api/ui_automation/pageobjects/{id}/` | GET/PUT/DELETE | 页面对象详情 |
| `/api/ui_automation/scripts/` | GET/POST | 测试脚本列表/创建 |
| `/api/ui_automation/scripts/{id}/` | GET/PUT/DELETE | 脚本详情 |
| `/api/ui_automation/scripts/{id}/execute/` | POST | 执行单个脚本 |
| `/api/ui_automation/suites/` | GET/POST | 测试套件列表/创建 |
| `/api/ui_automation/suites/{id}/` | GET/PUT/DELETE | 套件详情 |
| `/api/ui_automation/suites/{id}/execute/` | POST | 执行套件 |
| `/api/ui_automation/executions/` | GET | 执行记录列表 |
| `/api/ui_automation/executions/{id}/` | GET | 执行详情 |
| `/api/ui_automation/executions/{id}/screenshots/` | GET | 执行截图 |

### 13.5 前端设计要点
- 元素管理表格：名称、定位器、类型、所属页面
- 定位器可视化：截图上标注元素位置
- 脚本编辑器：代码高亮、代码提示、自动补全
- 页面对象结构树：页面→元素层级展示
- 执行控制台：实时日志流、截图滚动
- 执行报告：瀑布图、失败步骤高亮、录屏回放

---

## 十四、APP自动化模块 (`apps/app_automation/`)

### 14.1 模块作用
提供移动APP自动化测试功能，支持设备管理、元素定位、组件识别、APP测试套件。

### 14.2 核心功能
- **项目管理**：APP自动化项目独立管理
- **设备管理**：连接管理Android/iOS设备
- **元素识别**：原生元素定位、图像识别
- **组件库**：常用组件模板管理
- **测试用例**：APP操作脚本编写
- **测试套件**：用例编排和执行
- **执行监控**：实时日志、截图、录屏
- **多设备并行**：同时在多设备执行

### 14.3 数据模型

```
AppProject (APP自动化项目)
├── id (UUID, 主键)
├── name (项目名称)
├── description (描述)
├── platform (平台: android/ios)
├── app_package (应用包名)
├── app_activity (启动Activity)
├── created_by (外键 → User)
├── created_at
└── updated_at

AppDevice (设备配置)
├── id (UUID, 主键)
├── project (外键 → AppProject)
├── name (设备名称)
├── device_id (设备ID/UDID)
├── platform (平台)
├── os_version (系统版本)
├── resolution (分辨率)
├── status (状态: online/offline/busy)
├── capabilities (JSON, 设备能力)
├── last_seen (最后在线时间)
└── created_at

AppElement (APP元素)
├── id (UUID, 主键)
├── project (外键 → AppProject)
├── name (元素名称)
├── description (描述)
├── component (外键 → AppComponent)
├── locator_type (定位类型: id/xpath/accessibility/hierarchy)
├── locator_value (定位值)
├── fallback_locators (JSON, 备用定位器)
├── screenshot (截图标记)
├── created_at
└── updated_at

AppComponent (APP组件)
├── id (UUID, 主键)
├── project (外键 → AppProject)
├── name (组件名称)
├── description (描述)
├── component_type (组件类型: button/input/list/card/dialog)
├── template (JSON, 组件模板)
├── created_at
└── updated_at

AppTestSuite (APP测试套件)
├── id (UUID, 主键)
├── project (外键 → AppProject)
├── name (套件名称)
├── description (描述)
├── devices (多对多 → AppDevice)
├── parallelism (并行数量)
├── retry_count (失败重试次数)
├── created_by (外键 → User)
├── created_at
└── updated_at

AppTestCase (APP测试用例)
├── id (UUID, 主键)
├── suite (外键 → AppTestSuite)
├── name (用例名称)
├── description (描述)
├── steps (JSON, 测试步骤)
├── test_data (JSON, 测试数据)
├── expected_results (JSON, 预期结果)
├── created_at
└── updated_at

AppTestExecution (APP测试执行)
├── id (UUID, 主键)
├── test_suite (外键 → AppTestSuite)
├── device (外键 → AppDevice)
├── status (状态: pending/running/completed/failed)
├── total_cases (总用例数)
├── passed (通过数)
├── failed (失败数)
├── results (JSON, 执行结果)
├── screenshots (JSON, 截图列表)
├── video_url (录屏视频URL)
├── logs (TEXT, 执行日志)
├── duration (总耗时, 秒)
├── executed_at
└── executed_by (外键 → User)
```

### 14.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/app_automation/projects/` | GET/POST | 项目列表/创建 |
| `/api/app_automation/projects/{id}/` | GET/PUT/DELETE | 项目详情 |
| `/api/app_automation/devices/` | GET/POST | 设备列表/添加 |
| `/api/app_automation/devices/{id}/` | GET/PUT/DELETE | 设备详情 |
| `/api/app_automation/devices/{id}/status/` | GET | 设备状态检查 |
| `/api/app_automation/elements/` | GET/POST | 元素列表/创建 |
| `/api/app_automation/elements/{id}/` | GET/PUT/DELETE | 元素详情 |
| `/api/app_automation/components/` | GET/POST | 组件列表/创建 |
| `/api/app_automation/suites/` | GET/POST | 测试套件列表/创建 |
| `/api/app_automation/suites/{id}/` | GET/PUT/DELETE | 套件详情 |
| `/api/app_automation/suites/{id}/cases/` | GET/POST | 套件用例管理 |
| `/api/app_automation/suites/{id}/execute/` | POST | 执行套件 |
| `/api/app_automation/executions/` | GET | 执行记录列表 |
| `/api/app_automation/executions/{id}/` | GET | 执行详情 |
| `/api/app_automation/executions/{id}/screenshots/` | GET | 执行截图 |

### 14.5 前端设计要点
- 设备状态面板：卡片展示在线/离线/忙碌状态
- 设备详情弹窗：设备信息、截图预览、实时画面
- 元素选择器：截图上点击标注元素
- 组件模板库：拖拽式组件拼装
- 用例编辑器：步骤卡片拖拽排序
- 执行监控台：实时日志、设备矩阵视图
- 执行报告：截图时间线、失败对比、录屏播放

---

## 十五、数据工厂模块 (`apps/data_factory/`)

### 15.1 模块作用
提供测试数据生成功能，支持多种类型数据的快速生成，包括个人信息、地址、联系方式等。

### 15.2 核心功能
- **数据生成**：51种数据生成工具
- **自定义模板**：用户自定义数据规则
- **批量生成**：指定数量批量生成
- **格式转换**：支持JSON/CSV/Excel格式
- **数据预览**：实时预览生成结果
- **使用历史**：记录生成历史

### 15.3 数据模型

```
DataFactoryRecord (数据工厂记录)
├── id (UUID, 主键)
├── user (外键 → User)
├── name (记录名称)
├── tool_name (工具名称)
├── parameters (JSON, 生成参数)
├── output_format (输出格式: json/csv/excel)
├── result_preview (TEXT, 结果预览)
├── result_file (文件路径, nullable)
├── usage_count (使用次数)
├── created_at
└── updated_at
```

### 15.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/data_factory/generate/` | POST | 生成数据 |
| `/api/data_factory/records/` | GET/POST | 生成记录列表/创建 |
| `/api/data_factory/records/{id}/` | GET/DELETE | 记录详情/删除 |
| `/api/data_factory/preview/` | POST | 数据预览 |
| `/api/data_factory/export/` | POST | 导出数据文件 |

### 15.5 前端设计要点
- 工具分类列表：左侧分类、右侧工具网格
- 工具配置面板：参数表单、实时预览
- 生成结果展示：表格视图、JSON视图
- 批量配置：数量输入、格式选择
- 历史记录列表：时间排序、搜索过滤

---

## 十六、核心模块 (`apps/core/`)

### 16.1 模块作用
提供系统级别的核心功能，包括通知配置、系统设置等。

### 16.2 核心功能
- **通知配置**：统一的通知渠道管理
- **消息模板**：通知消息模板配置
- **通知规则**：触发规则和过滤条件

### 16.3 数据模型

```
UnifiedNotificationConfig (统一通知配置)
├── id (UUID, 主键)
├── name (配置名称)
├── channel (渠道: email/slack/webhook/wechat/dingtalk)
├── config (JSON, 渠道配置)
├── notification_rules (JSON, 通知规则)
├── is_active (是否启用)
├── created_by (外键 → User)
├── created_at
└── updated_at
```

### 16.4 API接口

| 接口路径 | 方法 | 功能描述 |
|---------|------|---------|
| `/api/core/notifications/configs/` | GET/POST | 通知配置列表/创建 |
| `/api/core/notifications/configs/{id}/` | GET/PUT/DELETE | 配置详情/更新 |
| `/api/core/notifications/test/` | POST | 测试通知发送 |

### 16.5 前端设计要点
- 通知渠道卡片：图标、状态开关、配置入口
- 配置表单：Webhook地址/Token/密钥等
- 规则编辑器：事件选择、条件配置
- 测试面板：输入测试内容、发送测试通知

---

## 十七、通用API设计规范

### 17.1 响应格式

```json
{
  "code": 200,
  "message": "success",
  "data": { ... },
  "pagination": {
    "total": 100,
    "page": 1,
    "page_size": 20,
    "total_pages": 5
  }
}
```

### 17.2 分页参数
- `page`: 页码 (默认1)
- `page_size`: 每页数量 (默认20, 最大100)
- `ordering`: 排序字段 (如 `-created_at`)

### 17.3 认证方式
- JWT Token认证
- Header: `Authorization: Bearer <token>`

### 17.4 通用错误码
| 错误码 | 说明 |
|-------|------|
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

---

## 十八、前端设计建议

### 18.1 整体风格
- **配色方案**：深色主题为主，专业科技感
- **主色调**：蓝色系 (#3B82F6)，辅助橙色 (#F59E0B) 表示警告/失败
- **字体**：无衬线字体，如 Inter、Noto Sans SC
- **图标库**：Lucide Icons 或 Heroicons

### 18.2 布局建议
- **左侧导航**：固定侧边栏，支持折叠
- **顶部栏**：项目选择器、搜索、通知、用户菜单
- **主内容区**：响应式布局，适配不同屏幕

### 18.3 组件库建议
- 使用 Tailwind CSS 或 UnoCSS
- 组件库：shadcn/ui 或 Headless UI
- 图表库：ECharts 或 Recharts
- 表格：TanStack Table
- 拖拽：dnd-kit 或 vue-draggable

### 18.4 状态管理
- 全局状态：Pinia (Vue) 或 Zustand (React)
- 服务端状态：TanStack Query (React) 或 Vue Query

---

## 十九、技术栈建议

### 19.1 前端技术栈
| 层级 | 推荐方案 |
|-----|---------|
| 框架 | React 18+ / Vue 3+ |
| UI组件库 | shadcn/ui 或 Element Plus |
| 构建工具 | Vite |
| 路由 | React Router / Vue Router |
| 状态管理 | Zustand / Pinia |
| HTTP客户端 | Axios |
| 图表 | ECharts / Recharts |
| 表格 | TanStack Table |
| 表单 | React Hook Form / VeeValidate |
| 拖拽 | dnd-kit / vue-draggable |
| Markdown | react-markdown / markdown-it |
| 代码高亮 | Shiki / Prism.js |
| 虚拟化 | TanStack Virtual |

### 19.2 后端技术栈
| 层级 | 推荐方案 |
|-----|---------|
| 框架 | Django 4+ + Django REST Framework |
| 认证 | simplejwt |
| 任务队列 | Celery + Redis |
| WebSocket | Django Channels |
| AI集成 | LangChain / Dify SDK |
| 自动化框架 | Selenium / Playwright |
| 数据库 | PostgreSQL |

---

*文档生成时间: 2026-04-12*
*代码路径: /Users/snow/snow-sakura/snow-python/testhub_platform*

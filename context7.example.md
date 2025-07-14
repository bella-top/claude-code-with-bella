> /mcp-prompt is running… Amazon Bedrock RAG库应该怎么使用

⏺ I'll help you understand how to use Amazon Bedrock RAG libraries. Let me search for the most current documentation and examples.

⏺ context7 - resolve-library-id (MCP)(libraryName: "Amazon Bedrock RAG")
⎿  Available Libraries (top matches):

     Each result includes:
     … +108 lines (ctrl+r to expand)

⏺ Perfect! I found the Amazon Bedrock RAG library. Let me get the detailed documentation for you.

⏺ context7 - get-library-docs (MCP)(context7CompatibleLibraryID: "/aws-samples/amazon-bedrock-rag", tokens: 8000)
⎿ TITLE: Deploying AWS CDK Stack to Default Account/Region                                                                                                                                             
DESCRIPTION: This command deploys the AWS CDK stack defined in the project to the AWS account and region configured as default in your environment. It provisions or updates the AWS resources as spe
cified in your CDK application.
… +186 lines (ctrl+r to expand)


⏺ 让我再获取一些更详细的架构和实现信息：

⏺ context7 - get-library-docs (MCP)(context7CompatibleLibraryID: "/aws-samples/amazon-bedrock-rag", tokens: 10000, topic: "architecture implementation setup")
⎿ TITLE: Installing Frontend Dependencies (npm)                                                                                                                                                        
DESCRIPTION: This command installs all required Node.js packages for the frontend application, ensuring all dependencies are available for the application to run. It should be executed from the fro
ntend directory.
… +186 lines (ctrl+r to expand)


⏺ 让我通过 DeepWiki 获取更详细的 Amazon Bedrock RAG 架构信息：

⏺ deepwiki - read_wiki_contents (MCP)(repoName: "aws-samples/amazon-bedrock-rag")
⎿  # Page: Introduction to Amazon Bedrock RAG

     # Introduction to Amazon Bedrock RAG
     … +277 lines (ctrl+r to expand)
⎿  # Page: System Architecture Overview

     # System Architecture Overview
     … +397 lines (ctrl+r to expand)
⎿  # Page: Key Concepts and Terminology

     # Key Concepts and Terminology
     … +270 lines (ctrl+r to expand)
⎿  # Page: Getting Started

     # Getting Started
     … +297 lines (ctrl+r to expand)
⎿  # Page: Prerequisites and Installation

     # Prerequisites and Installation
     … +298 lines (ctrl+r to expand)
⎿  # Page: Configuration and Deployment

     # Configuration and Deployment
     … +261 lines (ctrl+r to expand)
⎿  # Page: Backend Architecture

     # Backend Architecture
     … +267 lines (ctrl+r to expand)
⎿  # Page: CDK Infrastructure

     # CDK Infrastructure
     … +279 lines (ctrl+r to expand)
⎿  # Page: Lambda Functions

     # Lambda Functions
     … +243 lines (ctrl+r to expand)
⎿  # Page: Query Lambda

     # Query Lambda
     … +223 lines (ctrl+r to expand)
⎿  # Page: Data Source and Ingestion Lambdas

     # Data Source and Ingestion Lambdas
     … +288 lines (ctrl+r to expand)
⎿  # Page: Frontend Architecture

     # Frontend Architecture
     … +316 lines (ctrl+r to expand)
⎿  # Page: React Components

     # React Components
     … +223 lines (ctrl+r to expand)
⎿  # Page: App Component

     # App Component
     … +254 lines (ctrl+r to expand)
⎿  # Page: Chat and QA Components

     # Chat and QA Components
     … +229 lines (ctrl+r to expand)
⎿  # Page: RAG Models Integration

     # RAG Models Integration
     … +247 lines (ctrl+r to expand)
⎿  # Page: Dependencies and Build System

     # Dependencies and Build System
     … +236 lines (ctrl+r to expand)
⎿  # Page: Data Flow and Processing

     # Data Flow and Processing
     … +262 lines (ctrl+r to expand)
⎿  # Page: Document Ingestion Flow

     # Document Ingestion Flow
     … +226 lines (ctrl+r to expand)
⎿  # Page: Query Processing Flow

     # Query Processing Flow
     … +243 lines (ctrl+r to expand)
⎿  # Page: Web Crawler Configuration

     # Web Crawler Configuration
     … +247 lines (ctrl+r to expand)
⎿  # Page: Development Guide

     # Development Guide
     … +338 lines (ctrl+r to expand)
⎿  # Page: Extending the Backend

     # Extending the Backend
     … +327 lines (ctrl+r to expand)
⎿  # Page: Extending the Frontend

     # Extending the Frontend
     … +385 lines (ctrl+r to expand)
⎿  # Page: Adding New Data Sources

     # Adding New Data Sources
     … +421 lines (ctrl+r to expand)
⎿  # Page: Contribution Guidelines

     # Contribution Guidelines
     … +252 lines (ctrl+r to expand)

⏺ Amazon Bedrock RAG库的使用指南如下：

系统概述

Amazon Bedrock RAG 是一个基于知识库的检索增强生成(RAG)解决方案，使用 Amazon Bedrock 服务提供企业级的问答功能。该解决方案包含前端界面、后端API和知识库管理功能。

主要组件

前端组件：
- React 应用程序，提供用户界面
- 支持模型选择和问答交互
- 网页数据源配置界面

后端组件：
- API Gateway 提供 RESTful 接口
- Lambda 函数处理查询和数据摄取
- WAF 提供安全防护

知识库：
- Amazon Bedrock 知识库存储文档向量
- OpenSearch Serverless 作为向量存储
- 支持 S3 文档和网页爬虫数据源

快速开始

1. 环境准备

# 安装依赖
npm install -g aws-cdk
aws configure

# 克隆项目
git clone <repository-url>
cd amazon-bedrock-rag

2. 后端部署

cd backend
npm install

# 部署，需要替换为你的IP地址
cdk deploy --context allowedip="xxx.xxx.xxx.xxx/32"

部署后记录输出的：
- API Gateway URL
- S3 存储桶名称

3. 前端启动

cd frontend
npm install
npm start

4. 配置使用

1. 配置API端点：在前端输入 API Gateway URL
2. 选择模型：选择要使用的基础模型（默认：Claude Instant）
3. 上传文档：将文档上传到 S3 存储桶
4. 配置网页数据源：通过界面配置要爬取的网站URL
5. 开始提问：在聊天界面输入问题

数据源管理

S3 文档数据源

- 直接上传 PDF、TXT 等文档到 S3 存储桶
- 系统自动处理文档并添加到知识库
- 支持自动分块（500 tokens，20% 重叠）

网页爬虫数据源

- 通过 UI 配置种子 URL
- 支持包含/排除过滤器
- 每日自动爬取更新

API 接口

| 端点        | 方法   | 功能        |
  |-----------|------|-----------|
| /docs     | POST | 提交问题并获取答案 |
| /urls     | GET  | 获取网页爬虫配置  |
| /web-urls | POST | 更新网页爬虫配置  |

支持的基础模型

- 默认模型：Anthropic Claude Instant
- 嵌入模型：Amazon Titan Embeddings G1 - Text
- 支持通过前端界面选择不同模型

查询处理流程

1. 用户在前端输入问题
2. 系统从知识库检索相关内容
3. 使用选定的基础模型生成回答
4. 返回答案及引用来源

安全特性

- WAF 基于 IP 地址的访问控制
- S3 存储桶加密和 SSL 强制
- Lambda 函数最小权限原则
- API 流量限制和监控

扩展开发

添加新数据源

1. 创建数据源连接器
2. 实现数据摄取 Lambda 函数
3. 更新知识库配置
4. 添加前端管理界面

自定义前端组件

1. 使用 React 和 Material UI
2. 遵循现有组件模式
3. 集成新的 API 功能

故障排查

- API 调用失败：检查 baseUrl 配置和 WAF 设置
- 文档摄取失败：检查 S3 权限和文档格式
- 模型访问错误：确保在 Bedrock 控制台中启用了相应模型

清理资源

使用以下命令删除所有资源：
cdk destroy

这个解决方案提供了一个完整的企业级 RAG 实现，支持多种数据源和基础模型，具有良好的安全性和可扩展性。
╭───────────────────────────────────────────────────────╮
│ ✻ Welcome to Claude Code!                             │
│                                                       │
│   /help for help, /status for your current setup      │
│                                                       │
│   cwd: /Users/saizhuolin/test/claude-code-with-bella  │
│                                                       │
│   ─────────────────────────────────────────────────── │
│                                                       │
│   Overrides (via env):                                │
│                                                       │
│   • API Base URL: https://openapi-ait.ke.com          │
╰───────────────────────────────────────────────────────╯

> /mcp-prompt is running… Amazon Bedrock RAG的库应该怎么使用

⏺ I'll help you understand how to use Amazon Bedrock RAG libraries. Let me search for the most current documentation and examples.

⏺ context7 - resolve-library-id (MCP)(libraryName: "Amazon Bedrock RAG")
⎿  Available Libraries (top matches):

     Each result includes:
     … +108 lines (ctrl+r to expand)

⏺ Perfect! I found the Amazon Bedrock RAG library. Let me get the detailed documentation for you.

⏺ context7 - get-library-docs (MCP)(context7CompatibleLibraryID: "/aws-samples/amazon-bedrock-rag", tokens: 8000)
⎿ TITLE: Deploying AWS CDK Stack to Default Account/Region                                                                                                                                             
DESCRIPTION: This command deploys the AWS CDK stack defined in the project to the AWS account and region configured as default in your environment. It provisions or updates the AWS resources as spe
cified in your CDK application.
… +186 lines (ctrl+r to expand)


⏺ 让我再获取一些更详细的架构和实现信息：

⏺ context7 - get-library-docs (MCP)(context7CompatibleLibraryID: "/aws-samples/amazon-bedrock-rag", tokens: 10000, topic: "architecture implementation setup")
⎿ TITLE: Installing Frontend Dependencies (npm)                                                                                                                                                        
DESCRIPTION: This command installs all required Node.js packages for the frontend application, ensuring all dependencies are available for the application to run. It should be executed from the fro
ntend directory.
… +186 lines (ctrl+r to expand)


⏺ 让我通过 DeepWiki 获取更详细的 Amazon Bedrock RAG 架构信息：

⏺ deepwiki - read_wiki_contents (MCP)(repoName: "aws-samples/amazon-bedrock-rag")
⎿  # Page: Introduction to Amazon Bedrock RAG

     # Introduction to Amazon Bedrock RAG
     … +277 lines (ctrl+r to expand)
⎿  # Page: System Architecture Overview

     # System Architecture Overview
     … +397 lines (ctrl+r to expand)
⎿  # Page: Key Concepts and Terminology

     # Key Concepts and Terminology
     … +270 lines (ctrl+r to expand)
⎿  # Page: Getting Started

     # Getting Started
     … +297 lines (ctrl+r to expand)
⎿  # Page: Prerequisites and Installation

     # Prerequisites and Installation
     … +298 lines (ctrl+r to expand)
⎿  # Page: Configuration and Deployment

     # Configuration and Deployment
     … +261 lines (ctrl+r to expand)
⎿  # Page: Backend Architecture

     # Backend Architecture
     … +267 lines (ctrl+r to expand)
⎿  # Page: CDK Infrastructure

     # CDK Infrastructure
     … +279 lines (ctrl+r to expand)
⎿  # Page: Lambda Functions

     # Lambda Functions
     … +243 lines (ctrl+r to expand)
⎿  # Page: Query Lambda

     # Query Lambda
     … +223 lines (ctrl+r to expand)
⎿  # Page: Data Source and Ingestion Lambdas

     # Data Source and Ingestion Lambdas
     … +288 lines (ctrl+r to expand)
⎿  # Page: Frontend Architecture

     # Frontend Architecture
     … +316 lines (ctrl+r to expand)
⎿  # Page: React Components

     # React Components
     … +223 lines (ctrl+r to expand)
⎿  # Page: App Component

     # App Component
     … +254 lines (ctrl+r to expand)
⎿  # Page: Chat and QA Components

     # Chat and QA Components
     … +229 lines (ctrl+r to expand)
⎿  # Page: RAG Models Integration

     # RAG Models Integration
     … +247 lines (ctrl+r to expand)
⎿  # Page: Dependencies and Build System

     # Dependencies and Build System
     … +236 lines (ctrl+r to expand)
⎿  # Page: Data Flow and Processing

     # Data Flow and Processing
     … +262 lines (ctrl+r to expand)
⎿  # Page: Document Ingestion Flow

     # Document Ingestion Flow
     … +226 lines (ctrl+r to expand)
⎿  # Page: Query Processing Flow

     # Query Processing Flow
     … +243 lines (ctrl+r to expand)
⎿  # Page: Web Crawler Configuration

     # Web Crawler Configuration
     … +247 lines (ctrl+r to expand)
⎿  # Page: Development Guide

     # Development Guide
     … +338 lines (ctrl+r to expand)
⎿  # Page: Extending the Backend

     # Extending the Backend
     … +327 lines (ctrl+r to expand)
⎿  # Page: Extending the Frontend

     # Extending the Frontend
     … +385 lines (ctrl+r to expand)
⎿  # Page: Adding New Data Sources

     # Adding New Data Sources
     … +421 lines (ctrl+r to expand)
⎿  # Page: Contribution Guidelines

     # Contribution Guidelines
     … +252 lines (ctrl+r to expand)

⏺ Amazon Bedrock RAG库的使用指南如下：
```
系统概述

Amazon Bedrock RAG 是一个基于知识库的检索增强生成(RAG)解决方案，使用 Amazon Bedrock 服务提供企业级的问答功能。该解决方案包含前端界面、后端API和知识库管理功能。

主要组件

前端组件：
- React 应用程序，提供用户界面
- 支持模型选择和问答交互
- 网页数据源配置界面

后端组件：
- API Gateway 提供 RESTful 接口
- Lambda 函数处理查询和数据摄取
- WAF 提供安全防护

知识库：
- Amazon Bedrock 知识库存储文档向量
- OpenSearch Serverless 作为向量存储
- 支持 S3 文档和网页爬虫数据源

快速开始

1. 环境准备

# 安装依赖
npm install -g aws-cdk
aws configure

# 克隆项目
git clone <repository-url>
cd amazon-bedrock-rag

2. 后端部署

cd backend
npm install

# 部署，需要替换为你的IP地址
cdk deploy --context allowedip="xxx.xxx.xxx.xxx/32"

部署后记录输出的：
- API Gateway URL
- S3 存储桶名称

3. 前端启动

cd frontend
npm install
npm start

4. 配置使用

1. 配置API端点：在前端输入 API Gateway URL
2. 选择模型：选择要使用的基础模型（默认：Claude Instant）
3. 上传文档：将文档上传到 S3 存储桶
4. 配置网页数据源：通过界面配置要爬取的网站URL
5. 开始提问：在聊天界面输入问题

数据源管理

S3 文档数据源

- 直接上传 PDF、TXT 等文档到 S3 存储桶
- 系统自动处理文档并添加到知识库
- 支持自动分块（500 tokens，20% 重叠）

网页爬虫数据源

- 通过 UI 配置种子 URL
- 支持包含/排除过滤器
- 每日自动爬取更新

API 接口

| 端点        | 方法   | 功能        |
  |-----------|------|-----------|
| /docs     | POST | 提交问题并获取答案 |
| /urls     | GET  | 获取网页爬虫配置  |
| /web-urls | POST | 更新网页爬虫配置  |

支持的基础模型

- 默认模型：Anthropic Claude Instant
- 嵌入模型：Amazon Titan Embeddings G1 - Text
- 支持通过前端界面选择不同模型

查询处理流程

1. 用户在前端输入问题
2. 系统从知识库检索相关内容
3. 使用选定的基础模型生成回答
4. 返回答案及引用来源

安全特性

- WAF 基于 IP 地址的访问控制
- S3 存储桶加密和 SSL 强制
- Lambda 函数最小权限原则
- API 流量限制和监控

扩展开发

添加新数据源

1. 创建数据源连接器
2. 实现数据摄取 Lambda 函数
3. 更新知识库配置
4. 添加前端管理界面

自定义前端组件

1. 使用 React 和 Material UI
2. 遵循现有组件模式
3. 集成新的 API 功能

故障排查

- API 调用失败：检查 baseUrl 配置和 WAF 设置
- 文档摄取失败：检查 S3 权限和文档格式
- 模型访问错误：确保在 Bedrock 控制台中启用了相应模型

清理资源

使用以下命令删除所有资源：
cdk destroy

这个解决方案提供了一个完整的企业级 RAG 实现，支持多种数据源和基础模型，具有良好的安全性和可扩展性。
```

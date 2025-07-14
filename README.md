# Claude Code With Bella Openapi In IDE

[![Static Badge](https://img.shields.io/badge/UseIn-Github-blue?style=flat-square)](README_GITHUB_ACTION.md)

## Overview

- Claude Code是由anthropic-ai推出的一款强大的Code Agent，其主要特点如下：
  - 功能齐全：集成了主流IDE和GitHub中的cr, issues，在GitHub中使用：[Claude Code With GitHubActions](README_GITHUB_ACTION.md)
  - 设计合理：其执行任务的过程，完美复刻了一个程序员在进行项目开发时的思考过程，在执行复杂任务上表现很好
  - 不依赖Code Embeddings: 与依赖RAG的Code Agent不同，不需要针对代码训练Embedding模型，只需要LLM模型即可使用，将类似`README`的文件作为Code Base也更像一个人在进行开发
  - 安全性：Code Base不依赖向量库，确保了不会在服务端存储向量化的代码
  - 单一Agent的设计理念：参考 [devin.ai的Agent设计理念](https://cognition.ai/blog/dont-build-multi-agents#principles-of-context-engineering)，其中以Claude Code的实现作为例子阐述理念
  
- Bella Openapi 
  - 实现了Claude Code依赖的 `/v1/messsages` 接口
  - 所有在Bella-Openapi中接入的LLM协议均可使用Claude Code，不仅仅支持Claude系列模型，同时支持了Openai全系列、Gemini、DeepSeek、Qwen、Doubao等主流模型。
  - 可以为Claude Code使用单独的Apikey，控制成本，且每次请求产生的费用开销在日志中清晰可见
  - 企业级用户可以在使用统一网关的同时，享受最强Code Agent的全部特性

- 对比Cursor和Windsurf，Claude Code更适合异步编程的场景。Claude Code的Planning更为复杂，执行速度较慢，但是能力更强

## Tech Background
- [Agent设计](agent_design.md)
- [Code-Base设计](codebase_Cursor_vs_Claude.md)

## Dependency

### Bella-openapi

- [GitHub](https://github.com/LianjiaTech/bella-openapi)
- [文档](https://doc.bella.top/docs/bella-openapi/intro)

### Claude Code 

- [官网地址](https://docs.anthropic.com/en/docs/claude-code/overview)

- 安装claud-code:

    ```shell
    npm install -g @anthropic-ai/claude-code
    ```
    - IDE集成：
        - JetBrains IDE： Plugin Market安装
        - VS Code(包括Cursor和windsurf)： 
          - 在IDE中打开终端执行 `claude` 命令自动安装 （完成配置后才能执行）
          - 或者，打开插件市场安装

- 配置：
    ```shell
    cp setting.example.json ~/.claude/settings.json
    ```
    - 如果希望claude直接修改代码，不需要经过同意可以添加`allow`权限
      ```
            "Edit(**)",
            "Write(**)"
      ```
    - 修改 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_AUTH_TOKEN`
    - `ANTHROPIC_MODEL` 用于执行用户的推理任务，可配置为自己想用的模型
    - `ANTHROPIC_SMALL_FAST_MODEL` 用于执行一些简单的后台任务，比如`/clear`和diff写入文件等，可配置为自己想用的模型
    - 根据 [权限说明](#工具和权限说明) 配置 `permissions`

- 安装完成后重启IDE，打开终端执行 `claude` 命令即可启动。
- 在输入框输入：`/config`命令
- 将diff工具设置为`auto`，会自动进行IDE 检测
- 输入框执行 `/ide` 命令，检测ide是否存在

## 工作流程

### CodeBase
- 通过`/init` 生成 `CLAUDE.md` 文件，作为每个任务的起始提示词，相当于给claude提供进行项目开发的 `README`。[示例](./CLAUDE.example.md)
- claude code的user messages中会加入`CLAUDE.md` 文件的全量内容，示例：
```json
{
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "<system-reminder>\nAs you answer the user's questions, you can use the following context:\n# claudeMd\nCodebase and user instructions are shown below. Be sure to adhere to these instructions. IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them exactly as written.\n\nContents of /Users/saizhuolin/github/bella-openapi/api/CLAUDE.md (project instructions, checked into the codebase):\n\n# CLAUDE.md\n\nThis file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.\n\n## Project Overview\nBella OpenAPI is a multi-module Maven project that provides an OpenAPI gateway for AI services. It acts as a unified interface supporting multiple AI providers (OpenAI, AWS Bedrock, Alibaba Cloud, Huoshan, etc.) through a pluggable adapter pattern.\n\n## Module Architecture\n- **sdk/**: Core SDK with protocol definitions, DTOs, and client interfaces\n- **spi/**: Service Provider Interface module with authentication and session management (CAS/OAuth)\n- **server/**: Main Spring Boot application with REST endpoints and business logic\n\n## Build & Development Commands\n\n### Basic Maven Commands\n```bash\n# Build all modules\nmvn clean compile\n\n# Run tests\nmvn test\n\n# Package without tests\nmvn package -DskipTests\n\n# Install to local repository\nmvn install\n```\n\n### Development Scripts\n```bash\n# Build the application\n./build.sh\n\n# Run the application\n./run.sh\n```\n\n## Key Architecture Patterns\n\n### Protocol Adapter Pattern\nThe core architecture uses the **AdaptorManager** pattern to route requests to different AI providers:\n- `AdaptorManager` manages protocol adapters for each endpoint\n- `IProtocolAdaptor` interface defines how to communicate with each provider\n- Adapters are organized by endpoint type (completion, embedding, tts, asr, etc.)\n\n### Multi-Layer Architecture\n1. **Controllers** (`endpoints/`): REST API endpoints\n2. **Interceptors** (`intercept/`): Cross-cutting concerns (auth, quotas, metrics)\n3. **Protocol Layer** (`protocol/`): Provider-specific adapters\n4. **Repository Layer** (`db/repo/`): Database access with jOOQ\n\n### Database Integration\n- Uses jOOQ for database access with code generation\n- Generated POJOs and Records in `server/src/codegen/`\n- Database schema initialization scripts in `server/sql/`\n\n## Configuration\n- Main config: `server/src/main/resources/application.yml`\n- Multi-environment support with profile-specific configs\n- Uses Redis for caching (Redisson) and JetCache for L1/L2 caching\n- Apollo configuration center support (optional)\n\n## Testing\n- Test configuration: `application-ut.yml`\n- API test files in `server/src/test/resources/`\n- Mock implementations available for each protocol adapter\n\n## Key Directories\n- `server/src/main/java/com/ke/bella/openapi/endpoints/`: REST controllers\n- `server/src/main/java/com/ke/bella/openapi/protocol/`: Provider adapters\n- `server/src/main/java/com/ke/bella/openapi/intercept/`: Request/response interceptors\n- `sdk/src/main/java/com/ke/bella/openapi/protocol/`: Protocol DTOs and interfaces\n- `server/src/main/resources/lua/`: Lua scripts for Redis operations\n\n## Development Notes\n- When adding new AI providers, implement `IProtocolAdaptor` and register with `AdaptorManager`\n- Protocol-specific DTOs go in SDK module, implementation in server module\n- Cost calculation and metrics collection are handled through dedicated handlers\n- Some endpoints support both streaming and non-streaming responses\n\n# important-instruction-reminders\nDo what has been asked; nothing more, nothing less.\nNEVER create files unless they're absolutely necessary for achieving your goal.\nALWAYS prefer editing an existing file to creating a new one.\nNEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.\n\n      \n      IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context or otherwise consider it in your response unless it is highly relevant to your task. Most of the time, it is not relevant.\n</system-reminder>\n"
        },
        {
          "type": "text",
          "text": "为TransferUtils写一个测试用例"
        },
        {
          "type": "text",
          "text": "The user selected the following lines from /Users/saizhuolin/github/bella-openapi/api/sdk/src/main/java/com/ke/bella/openapi/protocol/message/TransferUtils.java:\nTransferUtils"
        }
      ]
    }
  ]
}
```
- 重复执行`/init`可以修改当前的`CLAUDE.md` 文件，常用于更新代码后
- Code Base优化
 - 示例CodeBase中的[Key Directories](./CLAUDE.example.md#key-directories)和[Key Architecture Patterns](./CLAUDE.example.md#key-architecture-patterns)用于提示LLM如何查找项目文件
 - 如果要加速推理，就需要优化这部分的提示词，比如给出更详细的介绍。可以加一个`Key Files`，涵盖重要且修改代码时阅读频率高的文件
 - 如果模型上下文足够大，甚至可以提供全量的`File Description`
 - 对于当前实现的特定任务，也可以临时添加对应的`Key Files`来加速LLM的推理
 - `File Description`的生成工具可以使用：[bella-issues-bot#初始化文件记忆系统](https://github.com/bella-top/bella-issues-bot/blob/main/quick-start.md#%E5%88%9D%E5%A7%8B%E5%8C%96%E6%96%87%E4%BB%B6%E8%AE%B0%E5%BF%86%E7%B3%BB%E7%BB%9F)

### file choose
`@[filename]` 通过 `@` 选择文件，加速推理。同时如果在终端中粘贴很长的提示词，比如一个很长的请求让claude code进行debug，可能会出现问题，可以把请求内容粘贴到一个文件里，告诉claude code读文件获取请求详情。

- 示例： 上文json中的第三个text就是选择文件后添加的提示词，相当于告诉LLM从哪开始看起

### 整体流程
[示例](./workflow.example.json)是完成一个任务的最后一步请求，这个请求后就会进行代码的实现。
这个请求过程展示了一个完整的任务执行的流程。若有兴趣可通过每一步的请求和响应来复现整个过程，理解其实现。


## Claude Code常用命令

**注意**：启动claude code后，在输入框输入`/`后，会出现快捷命令列表，可以前缀匹配，用`方向键`选择快捷命令，点击`tab`，可以选中快捷命令，点击`enter`会直接执行命令。

- Debug启动：`claude -d`
- help：`claude -h`
- 继续上一次的对话：`claude -c`
- 选择对话继续: `claude -r`
- code base: `/init` 生成 CLAUDE.md 文件，作为每个任务的起始提示词，相当于给claude提供进行项目开发的 `README`
- 清除历史上下文: `/clear` 相当于开启新会话，避免之前的执行历史的干扰
- 总结历史上下文: `/compact` 当大项目执行轮次过多时，上下文可能超长，需要将历史上下文进行总结。可以提供用于总结的提示词，告诉claude code怎么总结： `/compact [你的提示词]`
- 选择文件: `@[filename]` 通过 `@` 选择文件，加速推理。同时如果在终端中粘贴很长的提示词，比如一个很长的请求让claude code进行debug，可能会出现问题，可以把请求内容粘贴到一个文件里，告诉claude code读文件获取请求详情。
- 自动修改模式：按一次`shift + Tab`，进入`自动修改模式`，修改文件前不会询问
- 规划模式：按两次`shift + Tab`，进入`planning模式`，先和使用者制度计划，确认计划后再执行，适用于复杂任务场景
- 开启深度思考：提你的示词输入完成后，追加输入`think hard`或`think harder`
- 添加额外的工作目录：`/add-dir`，通常用于多个项目的联合开发，或者依赖其他项目的背景知识

## 工具和权限说明
如果不配置权限，Claude Code执行终端命令时会询问用户。如果不想过多询问（尤其是在异步进行编程时，可能不会一直盯着执行任务的终端界面），可以将权限配置在 `permissions` 的 `allow`之下。


### Claude Code的工具集合

| Tool             | Description                                          | Permission Required |
| :--------------- | :--------------------------------------------------- | :------------------ |
| **Agent**        | Runs a sub-agent to handle complex, multi-step tasks | No                  |
| **Bash**         | Executes shell commands in your environment          | Yes                 |
| **Edit**         | Makes targeted edits to specific files               | Yes                 |
| **Glob**         | Finds files based on pattern matching                | No                  |
| **Grep**         | Searches for patterns in file contents               | No                  |
| **LS**           | Lists files and directories                          | No                  |
| **MultiEdit**    | Performs multiple edits on a single file atomically  | Yes                 |
| **NotebookEdit** | Modifies Jupyter notebook cells                      | Yes                 |
| **NotebookRead** | Reads and displays Jupyter notebook contents         | No                  |
| **Read**         | Reads the contents of files                          | No                  |
| **TodoRead**     | Reads the current session's task list                | No                  |
| **TodoWrite**    | Creates and manages structured task lists            | No                  |
| **WebFetch**     | Fetches content from a specified URL                 | Yes                 |
| **WebSearch**    | Performs web searches with domain filtering          | Yes                 |
| **Write**        | Creates or overwrites files                          | Yes                 |

### 隐私文件的管理
比如包含密钥的文件，禁止修改并且没有读取意义的.lock文件等等，可以根据需要配置在 `permissions` 的 `deny`之下
示例：
```json
{
  "permissions": {
    "deny": [
      "Read(*.lock)",
      "Grep(*.lock)",
      "Edit(*.lock)",
      "Read(**/*application-dev.yml)",
      "Read(**/*application-prod.yml)",
      "Read(**/*application-test.yml)",
      "Read(**/*application-preview.yml)",
      "Grep(**/*application-dev.yml)",
      "Grep(**/*application-prod.yml)",
      "Grep(**/*application-test.yml)",
      "Grep(**/*application-preview.yml)",
      "Read(**/.env)",
      "Read(**/.env.*)",
      "Grep(**/.env)",
      "Grep(**/.env*)"
    ]
  }
}
```

## 实用的高级功能

### 自定义指令
**官方文档**：https://docs.anthropic.com/zh-CN/docs/claude-code/slash-commands#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%96%9C%E6%9D%A0%E5%91%BD%E4%BB%A4
```shell
mkdir ~/.claude/commands
echo "按照我们的编码标准修复GitHub Issues #$ARGUMENTS" > ~/.claude/commands/fix-issue.md
```
使用：
```shell
/project:fix-issue 123
```
**注意**：可以将一些常用提示词，作为命令输入

### 自定义mcp
官方文档：https://docs.anthropic.com/zh-CN/docs/claude-code/mcp

#### 推荐实用的mcp工具

**1.** 获取依赖代码库的最新版本文档和代码示例： https://github.com/upstash/context7
```shell
claude mcp add -s user context7 -- npx -y @upstash/context7-mcp
```

**2.** deepwiki：https://cognition.ai/blog/deepwiki-mcp-server
```shell
claude mcp add -s user -t http deepwiki https://mcp.deepwiki.com/mcp
```

**可以添加使用以上两个工具的提示词：**
```shell
echo "- When the user requests code examples, setup or configuration steps, or library/API documentation, you can use context7 tool. 
- When the user need to conduct research on GitHub's open-source code repository, you can use the DeepWiki tool" > ~/.claude/commands/mcp-tool.md
```

**使用：**
```shell
/user:mcp-tool Amazon Bedrock RAG库应该怎么使用
```
**使用示例：**
[context7使用示例](context7.example.md)

**3.将Claude Code集成为mcp服务**： 
不仅可以在Claude Code中配置MCP服务，还可以将Claude Code启动为MCP服务器，命令：`claude mcp serve`

### 非交互式命令
```shell
claude -p "按照我们的编码标准修复GitHub Issues 123"
```
**注意**：此命令可以在自己的服务中直接调用Claude Code，实现自己需要的功能，附[使用攻略](integration-guide.md)


## 常见问题
### IDE中新增的文件没有显示
**原因**：这是由于这些IDE的文件索引未刷新导致的，原因大概率是其刷新索引依赖的文件系统的事件通知丢失了或者有延迟，且IDE自身轮询刷新的时间间隔较长。在使用JetBrains的IDE时这个问题比较明显，因为文件索引机制更为复杂。

**解决方案**：手动刷新，右键项目目录 → "Reload from Disk"

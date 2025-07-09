# Claude Code CLI 代码集成完整攻略

## 目录
1. [概述](#概述)
2. [前置准备](#前置准备)
3. [核心实现](#核心实现)
4. [完整代码](#完整代码)
5. [使用示例](#使用示例)
6. [最佳实践](#最佳实践)
7. [故障排除](#故障排除)

## 概述

本攻略提供了在Node.js应用中直接调用Claude Code CLI的完整解决方案，基于对Claude Code Base Action的深度分析，提取了所有核心技术要点。

### 为什么需要直接调用

- **灵活性**: 在任何Node.js环境中使用，不限于GitHub Actions
- **集成性**: 可以整合到现有的应用程序和工作流中
- **可控性**: 完全控制执行参数、输出处理和错误处理
- **扩展性**: 可以根据需求自定义功能和输出格式

## 前置准备

### 1. 环境要求

[按照Claude-Code-Cli](README.md)

### 2. Node.js依赖

```json
{
  "dependencies": {
    "@types/node": "^20.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

### 3. 系统要求

- Unix/Linux/macOS (支持`mkfifo`命令)
- Node.js 18+
- 已安装`jq`命令行工具 (用于JSON处理)

## 核心实现

### 关键技术要点

1. **工作目录管理**: Claude必须在正确的项目目录下运行
2. **Named Pipes通信**: 使用命名管道避免stdin/stdout复杂性
3. **流式输出处理**: 实时解析结构化JSON输出
4. **进程生命周期管理**: 超时控制和优雅终止
5. **错误处理和资源清理**: 确保系统稳定性

### 接口设计

```typescript
export interface ClaudeExecutionOptions {
  prompt: string;                    // 执行的prompt
  model?: string;                    // 模型名称
  maxTurns?: number;                 // 最大对话轮数
  allowedTools?: string[];           // 允许的工具列表
  disallowedTools?: string[];        // 禁用的工具列表
  systemPrompt?: string;             // 系统提示词
  appendSystemPrompt?: string;       // 追加系统提示词
  mcpConfig?: string;                // MCP配置路径
  timeoutMinutes?: number;           // 超时时间(分钟)
  outputFile?: string;               // 输出文件路径
  workingDirectory?: string;         // 工作目录(关键!)
}

export interface ClaudeExecutionResult {
  success: boolean;                  // 执行是否成功
  exitCode: number;                  // 退出代码
  output: string;                    // 原始输出
  executionLog?: any[];              // 结构化执行日志
  outputFile?: string;               // 输出文件路径
}
```

## 完整代码

```typescript
import { spawn, exec } from 'child_process';
import { promisify } from 'util';
import { writeFile, unlink, mkdir } from 'fs/promises';
import { createWriteStream } from 'fs';
import path from 'path';

const execAsync = promisify(exec);

export class ClaudeCodeRunner {
  private tempDir: string;
  
  constructor(tempDir: string = '/tmp/claude-runner') {
    this.tempDir = tempDir;
  }

  /**
   * 执行Claude Code命令
   */
  async runClaude(options: ClaudeExecutionOptions): Promise<ClaudeExecutionResult> {
    // 保存原始工作目录
    const originalCwd = process.cwd();
    
    try {
      // 1. 切换到指定工作目录(关键步骤!)
      if (options.workingDirectory) {
        console.log(`🔄 Changing to working directory: ${options.workingDirectory}`);
        process.chdir(options.workingDirectory);
      }
      
      // 2. 确保临时目录存在
      await mkdir(this.tempDir, { recursive: true });
      
      const promptPath = path.join(this.tempDir, `prompt-${Date.now()}.txt`);
      const pipePath = path.join(this.tempDir, `claude_pipe-${Date.now()}`);
      const outputFile = options.outputFile || 
        path.join(this.tempDir, `claude-output-${Date.now()}.json`);
      
      try {
        // 3. 准备prompt文件
        await writeFile(promptPath, options.prompt);
        console.log(`📝 Prompt written to: ${promptPath}`);
        
        // 4. 创建命名管道
        await this.createNamedPipe(pipePath);
        console.log(`🔗 Named pipe created: ${pipePath}`);
        
        // 5. 构建Claude命令参数
        const claudeArgs = this.buildClaudeArgs(options);
        console.log(`⚙️  Claude args: ${claudeArgs.join(' ')}`);
        
        // 6. 执行Claude进程
        const result = await this.executeClaudeProcess(
          claudeArgs,
          promptPath,
          pipePath,
          outputFile,
          options
        );
        
        return result;
      } finally {
        // 7. 清理临时文件
        await this.cleanup([promptPath, pipePath]);
      }
    } finally {
      // 8. 恢复原始工作目录
      process.chdir(originalCwd);
    }
  }

  /**
   * 创建命名管道
   */
  private async createNamedPipe(pipePath: string): Promise<void> {
    try {
      await unlink(pipePath);
    } catch (e) {
      // 忽略文件不存在的错误
    }
    await execAsync(`mkfifo "${pipePath}"`);
  }

  /**
   * 构建Claude命令行参数
   */
  private buildClaudeArgs(options: ClaudeExecutionOptions): string[] {
    const args = ['-p', '--verbose', '--output-format', 'stream-json'];
    
    if (options.model) {
      args.push('--model', options.model);
    }
    if (options.maxTurns) {
      args.push('--max-turns', options.maxTurns.toString());
    }
    if (options.allowedTools?.length) {
      args.push('--allowedTools', options.allowedTools.join(','));
    }
    if (options.disallowedTools?.length) {
      args.push('--disallowedTools', options.disallowedTools.join(','));
    }
    if (options.systemPrompt) {
      args.push('--system-prompt', options.systemPrompt);
    }
    if (options.appendSystemPrompt) {
      args.push('--append-system-prompt', options.appendSystemPrompt);
    }
    if (options.mcpConfig) {
      args.push('--mcp-config', options.mcpConfig);
    }
    
    return args;
  }

  /**
   * 执行Claude进程
   */
  private async executeClaudeProcess(
    claudeArgs: string[],
    promptPath: string,
    pipePath: string,
    outputFile: string,
    options: ClaudeExecutionOptions
  ): Promise<ClaudeExecutionResult> {
    return new Promise((resolve) => {
      let output = '';
      let resolved = false;
      
      console.log(`🚀 Starting Claude process...`);
      
      // 启动Claude进程 - 继承当前工作目录
      const claudeProcess = spawn('claude', claudeArgs, {
        stdio: ['pipe', 'pipe', 'inherit'],
        // 不需要显式设置cwd，会继承process.cwd()
        env: {
          ...process.env,
          // 可以在这里添加其他环境变量
        }
      });

      // 设置超时
      const timeoutMs = (options.timeoutMinutes || 10) * 60 * 1000;
      const timeoutId = setTimeout(() => {
        if (!resolved) {
          console.error(`⏰ Claude process timed out after ${timeoutMs / 1000} seconds`);
          claudeProcess.kill('SIGTERM');
          setTimeout(() => claudeProcess.kill('SIGKILL'), 5000);
          resolved = true;
          resolve({
            success: false,
            exitCode: 124,
            output,
            outputFile
          });
        }
      }, timeoutMs);

      // 处理输出
      claudeProcess.stdout.on('data', (data) => {
        const text = data.toString();
        output += text;
        
        // 实时输出处理
        this.processStreamOutput(text);
      });

      // 处理进程退出
      claudeProcess.on('close', async (code) => {
        if (!resolved) {
          clearTimeout(timeoutId);
          resolved = true;
          
          console.log(`✅ Claude process completed with exit code: ${code}`);
          
          // 处理输出并保存到文件
          const executionLog = await this.processOutputToJson(output, outputFile);
          
          resolve({
            success: code === 0,
            exitCode: code || 0,
            output,
            executionLog,
            outputFile
          });
        }
      });

      claudeProcess.on('error', (error) => {
        if (!resolved) {
          console.error('❌ Claude process error:', error);
          clearTimeout(timeoutId);
          resolved = true;
          resolve({
            success: false,
            exitCode: 1,
            output,
            outputFile
          });
        }
      });

      // 通过命名管道发送prompt
      this.sendPromptThroughPipe(promptPath, pipePath, claudeProcess);
    });
  }

  /**
   * 处理流式输出
   */
  private processStreamOutput(text: string): void {
    const lines = text.split('\n');
    lines.forEach((line) => {
      if (line.trim() === '') return;
      
      try {
        const parsed = JSON.parse(line);
        console.log('📊', JSON.stringify(parsed, null, 2));
      } catch (e) {
        console.log('📝', line);
      }
    });
  }

  /**
   * 通过命名管道发送prompt
   */
  private async sendPromptThroughPipe(
    promptPath: string, 
    pipePath: string, 
    claudeProcess: any
  ): Promise<void> {
    // 启动cat进程发送prompt到管道
    const catProcess = spawn('cat', [promptPath], {
      stdio: ['ignore', 'pipe', 'inherit']
    });
    
    const pipeStream = createWriteStream(pipePath);
    catProcess.stdout.pipe(pipeStream);

    // 从管道读取并发送到Claude
    const pipeProcess = spawn('cat', [pipePath]);
    pipeProcess.stdout.pipe(claudeProcess.stdin);

    // 错误处理
    catProcess.on('error', (error) => {
      console.error('❌ Cat process error:', error);
      pipeStream.destroy();
    });

    pipeProcess.on('error', (error) => {
      console.error('❌ Pipe process error:', error);
      claudeProcess.kill('SIGTERM');
    });
  }

  /**
   * 处理输出为JSON格式
   */
  private async processOutputToJson(output: string, outputFile: string): Promise<any[]> {
    try {
      // 将输出保存到临时文件
      const tempFile = outputFile.replace('.json', '.txt');
      await writeFile(tempFile, output);
      
      // 使用jq处理为JSON数组
      const { stdout: jsonOutput } = await execAsync(`jq -s '.' "${tempFile}"`);
      await writeFile(outputFile, jsonOutput);
      
      console.log(`💾 Execution log saved to: ${outputFile}`);
      return JSON.parse(jsonOutput);
    } catch (e) {
      console.warn(`⚠️  Failed to process output to JSON: ${e}`);
      return [];
    }
  }

  /**
   * 清理临时文件
   */
  private async cleanup(paths: string[]): Promise<void> {
    for (const path of paths) {
      try {
        await unlink(path);
      } catch (e) {
        // 忽略清理错误
      }
    }
    console.log('🧹 Temporary files cleaned up');
  }

  /**
   * 便捷方法：将结果输出到Markdown文件
   */
  async runAndSaveToMarkdown(
    options: ClaudeExecutionOptions,
    markdownPath: string
  ): Promise<ClaudeExecutionResult> {
    const result = await this.runClaude(options);
    
    // 创建Markdown内容
    const markdownContent = this.formatResultAsMarkdown(result, options);
    await writeFile(markdownPath, markdownContent);
    
    console.log(`📄 Results saved to Markdown: ${markdownPath}`);
    return result;
  }

  /**
   * 格式化结果为Markdown
   */
  private formatResultAsMarkdown(
    result: ClaudeExecutionResult,
    options: ClaudeExecutionOptions
  ): string {
    const timestamp = new Date().toISOString();
    
    return `# Claude Code Execution Result

## Execution Info
- **Timestamp**: ${timestamp}
- **Success**: ${result.success ? '✅' : '❌'}
- **Exit Code**: ${result.exitCode}
- **Model**: ${options.model || 'default'}
- **Max Turns**: ${options.maxTurns || 'unlimited'}
- **Working Directory**: ${options.workingDirectory || process.cwd()}

## Prompt
\`\`\`
${options.prompt}
\`\`\`

## Output
\`\`\`
${result.output}
\`\`\`

## Execution Log
${result.executionLog ? '```json\n' + JSON.stringify(result.executionLog, null, 2) + '\n```' : 'No execution log available'}

---
*Generated by Claude Code Runner at ${timestamp}*
`;
  }
}
```

## 使用示例

### 1. 基本使用

```typescript
import { ClaudeCodeRunner } from './claude-code-runner';

async function basicExample() {
  const runner = new ClaudeCodeRunner();
  
  try {
    const result = await runner.runClaude({
      prompt: "分析当前项目的结构，生成一个项目概述",
      model: "claude-3-5-sonnet-20241022",
      maxTurns: 5,
      allowedTools: ["Read", "Glob", "Grep", "Bash"],
      timeoutMinutes: 15,
      workingDirectory: process.cwd() // 明确指定项目目录
    });
    
    console.log('✅ Execution completed:', result.success);
    if (result.success) {
      console.log('📊 Output preview:', result.output.substring(0, 500) + '...');
    } else {
      console.error('❌ Execution failed with exit code:', result.exitCode);
    }
  } catch (error) {
    console.error('❌ Error:', error);
  }
}
```

### 2. 保存到Markdown文件

```typescript
async function markdownExample() {
  const runner = new ClaudeCodeRunner();
  
  try {
    const result = await runner.runAndSaveToMarkdown({
      prompt: "帮我优化这个函数的性能，并解释优化原理",
      model: "claude-3-5-sonnet-20241022",
      systemPrompt: "你是一个性能优化专家，专注于提供详细的代码分析和优化建议。",
      maxTurns: 3,
      allowedTools: ["Read", "Edit", "Grep"],
      timeoutMinutes: 10,
      workingDirectory: "/path/to/your/project"
    }, './claude-optimization-result.md');
    
    console.log('📄 Results saved to: ./claude-optimization-result.md');
  } catch (error) {
    console.error('❌ Error:', error);
  }
}
```

### 3. 批量处理

```typescript
async function batchProcessing() {
  const runner = new ClaudeCodeRunner();
  const tasks = [
    {
      name: "代码审查",
      prompt: "审查src目录下的所有TypeScript文件，找出潜在问题",
      outputFile: "./reviews/code-review.md"
    },
    {
      name: "文档生成",
      prompt: "为主要的API函数生成文档",
      outputFile: "./docs/api-docs.md"
    },
    {
      name: "测试建议",
      prompt: "分析测试覆盖率，提供测试改进建议",
      outputFile: "./reports/test-suggestions.md"
    }
  ];

  for (const task of tasks) {
    console.log(`🔄 Processing: ${task.name}`);
    
    try {
      const result = await runner.runAndSaveToMarkdown({
        prompt: task.prompt,
        model: "claude-3-5-sonnet-20241022",
        maxTurns: 5,
        allowedTools: ["Read", "Glob", "Grep"],
        timeoutMinutes: 20,
        workingDirectory: process.cwd()
      }, task.outputFile);
      
      console.log(`✅ ${task.name} completed: ${result.success}`);
    } catch (error) {
      console.error(`❌ ${task.name} failed:`, error);
    }
  }
}
```

## 最佳实践

### 1. 工作目录管理

```typescript
// ✅ 推荐：明确指定工作目录
const result = await runner.runClaude({
  workingDirectory: "/absolute/path/to/project",
  // ... 其他选项
});

// ❌ 避免：依赖当前目录
const result = await runner.runClaude({
  // workingDirectory 未指定
});
```

### 2. 工具权限控制

```typescript
// ✅ 推荐：明确指定允许的工具
const result = await runner.runClaude({
  allowedTools: ["Read", "Glob", "Grep"], // 只读操作
  prompt: "分析代码结构"
});

// ⚠️  谨慎：允许写入操作
const result = await runner.runClaude({
  allowedTools: ["Read", "Write", "Edit", "Bash"], // 包含写入操作
  prompt: "优化代码"
});
```

### 3. 超时和资源管理

```typescript
// ✅ 推荐：设置合理的超时时间
const result = await runner.runClaude({
  timeoutMinutes: 15, // 根据任务复杂度调整
  maxTurns: 5,        // 限制对话轮数
});
```

### 4. 错误处理

```typescript
async function robustExecution() {
  const runner = new ClaudeCodeRunner();
  
  try {
    const result = await runner.runClaude(options);
    
    if (!result.success) {
      console.error(`Claude execution failed with exit code: ${result.exitCode}`);
      console.error(`Output: ${result.output}`);
      return;
    }
    
    // 处理成功结果
    console.log('Success!', result.output);
    
  } catch (error) {
    if (error.code === 'ENOENT') {
      console.error('❌ Claude CLI not found. Install with: npm install -g @anthropic-ai/claude-code');
    } else if (error.code === 'EACCES') {
      console.error('❌ Permission denied. Check file permissions.');
    } else {
      console.error('❌ Unexpected error:', error);
    }
  }
}
```

## 故障排除

### 常见问题及解决方案

#### 1. Claude CLI未找到

```bash
Error: spawn claude ENOENT
```

**解决方案**:
```bash
npm install -g @anthropic-ai/claude-code@latest
which claude  # 验证安装路径
```

#### 2. 权限问题

```bash
Error: EACCES: permission denied
```

**解决方案**:
```bash
# 检查临时目录权限
ls -la /tmp/claude-runner/
chmod 755 /tmp/claude-runner/

# 或使用不同的临时目录
const runner = new ClaudeCodeRunner('./temp/claude');
```

#### 3. Named Pipe创建失败

```bash
Error: mkfifo command not found
```

**解决方案**:
- 确保在Unix/Linux/macOS系统上运行
- Windows用户需要使用WSL或Docker

#### 4. jq命令未找到

```bash
Error: jq: command not found
```

**解决方案**:
```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# CentOS/RHEL
sudo yum install jq
```

### 调试技巧

#### 1. 启用详细日志

```typescript
// 在环境变量中启用调试
process.env.DEBUG = 'claude-runner:*';
```

#### 2. 保留临时文件

```typescript
class DebugClaudeCodeRunner extends ClaudeCodeRunner {
  private async cleanup(paths: string[]): Promise<void> {
    console.log('🐛 Debug mode: Skipping cleanup');
    console.log('📁 Temp files:', paths);
    // 不删除临时文件，便于调试
  }
}
```

#### 3. 输出分析

```typescript
const result = await runner.runClaude(options);

// 分析原始输出
console.log('Raw output length:', result.output.length);
console.log('Exit code:', result.exitCode);
console.log('Execution log entries:', result.executionLog?.length || 0);

// 保存调试信息
await writeFile('./debug-output.txt', result.output);
```

---

## 总结

这个完整攻略提供了在Node.js中直接调用Claude Code CLI的所有必要信息：

1. **核心实现**: 基于Named Pipes的稳定进程间通信
2. **工作目录管理**: 确保Claude在正确的项目上下文中运行
3. **完整的错误处理**: 超时控制、资源清理和异常处理
4. **灵活的输出处理**: 支持实时输出和结构化日志
5. **便捷的集成接口**: 易于集成到现有应用中

通过遵循这个攻略，你可以在任何Node.js项目中稳定地使用Claude Code的强大功能。

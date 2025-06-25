# Claude Code With GitHub Actions

## 概述

本文档介绍如何在GitHub Actions中集成Claude Code。通过GitHub Actions，可以在merger request或创建issues时自动触发Claude Code执行相关操作。

## 核心优势

- **自动化代码审查**: 在PR中自动分析代码质量，提供改进建议
- **Issues自动解决**: 当Issues被创建后，可立即自动处理
- **文档自动更新**: 可以扩展GitHub Action实现项目文档的自动更新
- **安全漏洞检测**: 自动扫描潜在的安全问题

## 配置要求

### 1. GitHub Secrets 配置

在仓库的Settings -> Secrets and variables -> Actions中添加以下secrets：

```Bash
ANTHROPIC_BASE_URL    # Bella OpenAPI的基础URL
ANTHROPIC_AUTH_TOKEN  # API访问令牌
```

### 2. 工作流权限

确保GitHub Actions使用的Tokens具有必要的权限：

```yaml
permissions:
  contents: Read And Write
  pull-requests: Read And Write
  issues: Read And Write
```

## 配置流程

```shell
# 进入项目目录
mkdir -p .github/workflows
```
核心Actions:
```shell
#Job: Run Claude Code中的model和ANTHROPIC_SMALL_FAST_MODEL替换为自己使用的模型名
cat > .github/workflows/claude.yml << 'EOF'
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
      (github.event_name == 'issues' && (contains(github.event.issue.body, '@claude') || contains(github.event.issue.title, '@claude')))
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
    steps:
      - name: Get PR details
        id: pr
        uses: actions/github-script@v7
        if: github.event_name == 'issue_comment' && github.event.issue.pull_request
        with:
          github-token: ${{ secrets.GIT_TOKEN }}
          script: |
            const { data: pr } = await github.rest.pulls.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number
            });
            core.setOutput('head_sha', pr.head.sha);
            core.setOutput('head_ref', pr.head.ref);
            core.setOutput('head_repo', pr.head.repo.full_name);

      - name: Checkout repository
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4
        with:
          repository: ${{ steps.pr.outputs.head_repo || github.repository }}
          ref: ${{ steps.pr.outputs.head_sha || github.sha }}
          fetch-depth: 0
          token: ${{ secrets.GIT_TOKEN }}

      - name: Run Claude Code
        id: claude
        uses: anthropics/claude-code-action@beta
        with:
          model: "claude-4-sonnet"
          github_token: ${{ secrets.GIT_TOKEN }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_AUTH_TOKEN }}
          claude_env: |
            ANTHROPIC_AUTH_TOKEN: ${{ secrets.ANTHROPIC_AUTH_TOKEN }}
            ANTHROPIC_SMALL_FAST_MODEL: claude-3-5-haiku
            CLAUDE_CODE_MAX_OUTPUT_TOKENS: 64000
        env:
          ANTHROPIC_BASE_URL: ${{ secrets.ANTHROPIC_BASE_URL }}
EOF
```

自动触发PR:
```shell
# 将 branches 改为代码仓中的默认分支
cat > .github/workflows/auto-review.yml << 'EOF'
name: Auto Review Request

on:
  pull_request_target:
    types: [opened]
    branches: [develop]

jobs:
  auto-comment:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Add review request comment
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GIT_TOKEN }}
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '@claude 检查提交的代码'
            })
EOF
```

## 功能用法及展示

### 解决issues:
在issues的标题、body、comment中 `@claude` 均会触发。可以实现代码，也可以向其咨询问题。

- 使用示例：
  - [Bella-Openapi实现空闲拉取任务模式](https://github.com/LianjiaTech/bella-openapi/issues/59) 复杂问题，先咨询问题（增加思考过程），后写代码
  - [Bella-Openapi支持apikey绑定service_id](https://github.com/LianjiaTech/bella-openapi/issues/89) 简单问题，直接写代码

## Code Review:
在MR的body、comment中`@claude`均会触发，添加`auto-review.yml`后提MR会自动触发CR。

- CR生成的内容大致包含：代码修改总结、优点、问题、修改建议/直接merge。
- 对CR的修改建议有疑问，可以在comment中`@claude`进行确认或纠正。
- 如果采纳建议，可以直接复制修改建议在comment中`@claude`，告知其进行修改。或者将修改建议复制到本地的Claude Code Cli，让其帮助实现代码。
- 修改测试完毕后，在comment中`@claude`告知其再次CR。例如`修改完成xxxx，请再次Review`
- 使用示例：
  - [Bella-Openapi适配native message api](https://github.com/LianjiaTech/bella-openapi/pull/87) CR建议修改 -> 对建议提出纠正 -> 达成一致 -> update -> CR
  - [Bella-Openapi支持apikey绑定service_id](https://github.com/LianjiaTech/bella-openapi/pull/90) CR建议直接merge
  
## 故障排除

1. **认证失败**: 检查ANTHROPIC_AUTH_TOKEN是否正确配置
2. **权限不足**: 确保GitHub Actions具有必要的仓库权限

通过合理配置GitHub Actions和Claude Code，可以显著提升开发效率，实现真正的AI驱动开发工作流。

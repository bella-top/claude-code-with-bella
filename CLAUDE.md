# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository provides documentation, examples, and configuration templates for integrating Claude Code with Bella OpenAPI. It serves as a guide for setting up and using Claude Code as a powerful code agent with various LLM providers through the Bella OpenAPI gateway.

## Repository Structure

- **README.md**: Main documentation in Chinese explaining Claude Code integration with Bella OpenAPI
- **README_GITHUB_ACTION.md**: Documentation for GitHub Actions integration (currently minimal)
- **settings.example.json**: Example configuration file for Claude Code settings
- **workflow.example.json**: Example workflow demonstrating task execution flow
- **CLAUDE.example.md**: Example CLAUDE.md file showing how to document a Java/Maven project (Bella OpenAPI)

## Key Components

### Configuration Templates
- **settings.example.json**: Contains example environment variables and permissions for Claude Code
  - Environment variables: `ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, model configurations
  - Permissions: Defines allowed/denied operations for security
  - Model settings: Primary model (`ANTHROPIC_MODEL`) and fast model (`ANTHROPIC_SMALL_FAST_MODEL`)

### Documentation Examples
- **CLAUDE.example.md**: Comprehensive example showing how to document a complex Java project
  - Maven build commands and development scripts
  - Multi-module architecture documentation
  - Key directories and architectural patterns
  - Request processing flow and development notes

## Usage Guidelines

### Setting up Claude Code with Bella OpenAPI
1. Copy `settings.example.json` to `~/.claude/settings.json`
2. Update the `ANTHROPIC_BASE_URL` to point to your Bella OpenAPI instance
3. Configure `ANTHROPIC_AUTH_TOKEN` with your API key
4. Adjust permissions based on your security requirements

### Creating Project Documentation
Use the pattern from `CLAUDE.example.md` when documenting new projects:
- Include build commands and development workflows
- Document architectural patterns and key directories
- Explain request/processing flows for complex systems
- Provide development notes for extending the system

### Security Considerations
The example configuration includes security restrictions:
- Denies access to environment files (`.env*`)
- Blocks access to production configuration files
- Restricts dangerous bash commands
- Prevents access to lock files

## Key Features Highlighted

### Multi-Provider Support
- Supports various LLM providers through Bella OpenAPI (OpenAI, Claude, Gemini, DeepSeek, etc.)
- Cost-effective with separate API key management
- Enterprise-grade gateway capabilities

### Development Workflow
- Uses `/init` command to generate CLAUDE.md files for projects
- Supports file selection with `@filename` syntax
- Provides context management with `/clear` and `/compact` commands
- Enables asynchronous development workflows

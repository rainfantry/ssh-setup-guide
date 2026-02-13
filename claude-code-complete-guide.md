# Claude Code Complete Guide

Complete reference for using Claude Code CLI with skills, agents, and advanced features.

## Table of Contents
1. [What is Claude Code?](#what-is-claude-code)
2. [Installation](#installation)
3. [Basic Usage](#basic-usage)
4. [Skills System](#skills-system)
5. [Agent System](#agent-system)
6. [Advanced Features](#advanced-features)
7. [Configuration](#configuration)
8. [Tips & Best Practices](#tips--best-practices)

---

## What is Claude Code?

Claude Code is an AI-powered command-line tool that helps you:
- Write and edit code
- Debug and fix errors
- Run commands and scripts
- Navigate and understand codebases
- Automate development tasks
- Use specialized skills for common workflows

**Key Capabilities:**
- Natural language interaction
- Context-aware code editing
- File system access
- Command execution
- Browser automation (with Chrome extension)
- Extensible with custom skills

---

## Installation

### Linux / macOS

```bash
# Download and install
curl -fsSL https://claude.ai/install.sh | sh

# Verify installation
claude --version
```

### Windows (via WSL2)

```powershell
# Install WSL2 first
wsl --install

# Then in WSL:
curl -fsSL https://claude.ai/install.sh | sh
```

### First Run

```bash
# Start Claude Code
claude

# You'll be prompted to authenticate
# Follow the browser link to sign in
```

---

## Basic Usage

### Starting a Session

```bash
# Start interactive session
claude

# Start with a specific task
claude "Fix all the bugs in src/"

# Start in a specific directory
claude --cwd /path/to/project

# Use a specific model
claude --model sonnet  # Default
claude --model opus    # More capable, slower
claude --model haiku   # Faster, lighter
```

### Basic Commands

```bash
# In Claude Code session:

# Ask questions
> How do I set up a React app?

# Request code changes
> Add error handling to the login function

# Run commands
> Run the tests

# Create files
> Create a new component called UserProfile

# Fix errors
> Fix the TypeScript errors in this file
```

### Slash Commands

```bash
/help           # Show available commands
/clear          # Clear conversation history
/reset          # Reset session
/exit           # Exit Claude Code
/model sonnet   # Switch model
/model opus
/model haiku
```

---

## Skills System

Skills are pre-built workflows that automate common tasks. They can be invoked using slash commands.

### Built-in Skills

#### /commit - Git Commit Helper

Creates well-formatted git commits with AI-generated messages.

```bash
# Basic usage
/commit

# With custom message
/commit "Add user authentication feature"

# Commit specific files
/commit src/auth.js src/utils.js

# Skip hooks
/commit --no-verify

# Amend previous commit
/commit --amend
```

**What it does:**
- Reviews staged changes
- Generates descriptive commit message
- Follows conventional commit format
- Checks for common issues
- Creates commit

#### /review-pr - Pull Request Reviewer

Reviews pull requests and provides feedback.

```bash
# Review current PR
/review-pr

# Review specific PR
/review-pr 123

# Review PR from URL
/review-pr https://github.com/user/repo/pull/123

# Deep review
/review-pr --thorough
```

**What it does:**
- Analyzes code changes
- Identifies potential issues
- Suggests improvements
- Checks for security issues
- Reviews tests coverage

#### /test - Test Runner

Runs tests and analyzes results.

```bash
# Run all tests
/test

# Run specific test file
/test tests/auth.test.js

# Run with coverage
/test --coverage

# Watch mode
/test --watch

# Verbose output
/test --verbose
```

**What it does:**
- Detects test framework
- Runs appropriate test command
- Analyzes failures
- Suggests fixes
- Shows coverage report

#### /debug - Debugger

Helps debug code and find issues.

```bash
# Debug current error
/debug

# Debug specific file
/debug src/app.js

# Debug with logs
/debug --logs

# Interactive debugging
/debug --interactive
```

**What it does:**
- Analyzes error messages
- Identifies root cause
- Suggests fixes
- Adds debug logging
- Tests solutions

#### /explain - Code Explainer

Explains code in plain English.

```bash
# Explain current file
/explain

# Explain specific function
/explain calculateTotal

# Explain with examples
/explain --examples

# Simple explanation
/explain --simple
```

**What it does:**
- Breaks down code logic
- Explains algorithms
- Provides examples
- Clarifies complex concepts
- Documents behavior

#### /refactor - Code Refactorer

Refactors code for better quality.

```bash
# Refactor current file
/refactor

# Refactor specific function
/refactor handleSubmit

# Focus on performance
/refactor --performance

# Focus on readability
/refactor --readability
```

**What it does:**
- Improves code structure
- Removes duplication
- Applies best practices
- Maintains functionality
- Adds tests

#### /docs - Documentation Generator

Generates documentation for code.

```bash
# Generate docs for current file
/docs

# Generate README
/docs --readme

# Generate API docs
/docs --api

# Generate JSDoc comments
/docs --jsdoc
```

**What it does:**
- Creates documentation
- Adds code comments
- Generates README files
- Documents API endpoints
- Creates usage examples

#### /security - Security Auditor

Scans code for security issues.

```bash
# Security audit
/security

# Check dependencies
/security --deps

# Find vulnerabilities
/security --vulns

# Generate report
/security --report
```

**What it does:**
- Scans for vulnerabilities
- Checks dependencies
- Identifies security issues
- Suggests fixes
- Creates audit report

#### /optimize - Performance Optimizer

Optimizes code performance.

```bash
# Optimize current file
/optimize

# Profile performance
/optimize --profile

# Optimize bundle size
/optimize --bundle

# Database optimization
/optimize --db
```

**What it does:**
- Identifies bottlenecks
- Suggests optimizations
- Reduces bundle size
- Improves algorithms
- Optimizes queries

#### /migrate - Code Migrator

Migrates code between frameworks/versions.

```bash
# Migrate to newer version
/migrate --to react@18

# Migrate from framework
/migrate --from vue --to react

# Migrate database
/migrate --db postgres

# Generate migration plan
/migrate --plan
```

**What it does:**
- Plans migration
- Updates dependencies
- Refactors code
- Updates syntax
- Tests compatibility

---

## Agent System

Agents are autonomous AI assistants that can perform multi-step tasks.

### Available Agents

#### General Purpose Agent

Handles complex, multi-step tasks autonomously.

```bash
# Invoke general agent
> Use general agent to refactor the entire auth system

# With specific instructions
> Agent: analyze the codebase and create a migration plan
```

**Capabilities:**
- Code search
- File reading
- Multi-step planning
- Autonomous execution
- Context gathering

#### Explore Agent

Specialized for exploring and understanding codebases.

```bash
# Quick exploration
> Explore agent: find all API endpoints

# Medium thoroughness
> Explore medium: how does authentication work?

# Very thorough
> Explore very thorough: map out the entire data flow
```

**Thoroughness Levels:**
- `quick` - Basic search
- `medium` - Moderate exploration
- `very thorough` - Comprehensive analysis

**What it does:**
- Searches code patterns
- Maps file structures
- Traces function calls
- Documents architecture
- Identifies dependencies

#### Plan Agent

Software architect for implementation planning.

```bash
# Create implementation plan
> Plan agent: design a new payment system

# Review architecture
> Plan: analyze current auth architecture
```

**What it does:**
- Creates step-by-step plans
- Identifies critical files
- Considers trade-offs
- Designs architecture
- Provides recommendations

#### Bash Agent

Command execution specialist.

```bash
# Execute complex commands
> Bash agent: set up the development environment

# Git operations
> Bash agent: create feature branch and push
```

**What it does:**
- Runs shell commands
- Git operations
- Package management
- File operations
- System configuration

---

## Advanced Features

### Tool Use

Claude Code has access to various tools:

#### File Operations

```bash
# Read files
> Show me the contents of package.json

# Edit files
> Update the version in package.json to 2.0.0

# Create files
> Create a new config file for production

# Delete files
> Remove all .log files
```

#### Command Execution

```bash
# Run commands
> Run npm install

# Execute scripts
> Run the build script

# Check status
> What's the git status?
```

#### Web Search

```bash
# Search for information
> Search for React 18 migration guide

# Find documentation
> Look up the latest Next.js features
```

#### Browser Automation

Requires Claude in Chrome extension.

```bash
# Navigate websites
> Open GitHub and navigate to my repositories

# Fill forms
> Fill out the contact form with test data

# Extract data
> Get the list of issues from the GitHub repo
```

### Context Management

```bash
# Add files to context
> Add src/auth.js to context

# Remove from context
> Remove that file from context

# Show context
> What files do you have in context?

# Clear context
> Clear all context
```

### Multi-file Operations

```bash
# Work across multiple files
> Refactor the authentication system across all files

# Find and replace
> Replace all instances of oldFunction with newFunction

# Rename components
> Rename UserCard to UserProfile everywhere
```

---

## Configuration

### Settings File

Location: `~/.claude/config.json`

```json
{
  "model": "sonnet",
  "temperature": 0.7,
  "max_tokens": 4096,
  "auto_commit": false,
  "theme": "dark",
  "editor": "vim"
}
```

### Environment Variables

```bash
# Set default model
export CLAUDE_MODEL=opus

# Set API key (if needed)
export CLAUDE_API_KEY=your-key

# Set working directory
export CLAUDE_CWD=/path/to/project
```

### Custom Skills

Create custom skills in `~/.claude/skills/`

```bash
# Skill structure
~/.claude/skills/
  └── my-skill/
      ├── skill.json
      └── prompt.md
```

**skill.json:**
```json
{
  "name": "my-skill",
  "description": "My custom skill",
  "command": "myskill",
  "parameters": [
    {
      "name": "input",
      "required": true,
      "description": "Input parameter"
    }
  ]
}
```

---

## Tips & Best Practices

### Effective Prompting

**Be Specific:**
```bash
# ❌ Vague
> Fix the code

# ✅ Specific
> Fix the undefined variable error in the handleSubmit function
```

**Provide Context:**
```bash
# ❌ No context
> Add authentication

# ✅ With context
> Add JWT authentication to the Express API, storing tokens in httpOnly cookies
```

**Break Down Complex Tasks:**
```bash
# ❌ Too complex
> Build a complete e-commerce platform

# ✅ Step by step
> Step 1: Set up the product database schema
> Step 2: Create API endpoints for products
> Step 3: Add shopping cart functionality
```

### Workflow Tips

**1. Start with Exploration:**
```bash
# Understand the codebase first
> Explore: how is routing handled in this app?
```

**2. Plan Before Implementing:**
```bash
# Get a plan first
> Plan agent: design the new feature architecture
```

**3. Use Appropriate Agents:**
```bash
# Quick tasks: Direct commands
> Add error handling to this function

# Complex tasks: General agent
> Agent: refactor the entire authentication system

# Code exploration: Explore agent
> Explore medium: map out all API endpoints
```

**4. Leverage Skills:**
```bash
# Use skills for common tasks
/commit          # Instead of manual commits
/test            # Instead of running tests manually
/review-pr       # Instead of manual code review
```

### Git Workflow

```bash
# 1. Start feature
> Create a new branch called feature/user-auth

# 2. Make changes
> Implement JWT authentication

# 3. Test
/test

# 4. Commit
/commit

# 5. Review
/review-pr

# 6. Push
> Push the branch to origin
```

### Debugging Workflow

```bash
# 1. Reproduce error
> Run the failing test

# 2. Analyze
/debug

# 3. Fix
> Apply the suggested fix

# 4. Verify
/test

# 5. Commit
/commit "Fix: resolve authentication error"
```

### Code Review Workflow

```bash
# 1. Fetch PR
> Fetch PR #123

# 2. Review
/review-pr 123

# 3. Test changes
/test

# 4. Security check
/security

# 5. Provide feedback
> Add comments to the PR about the security concerns
```

---

## Common Use Cases

### Setting Up a New Project

```bash
# 1. Create project structure
> Create a new React TypeScript project with best practices

# 2. Set up tooling
> Add ESLint, Prettier, and Husky

# 3. Initialize git
> Initialize git with a .gitignore for Node.js

# 4. Create documentation
/docs --readme

# 5. First commit
/commit "Initial project setup"
```

### Fixing Bugs

```bash
# 1. Identify issue
> The login button doesn't work when clicking

# 2. Analyze
/debug src/components/Login.jsx

# 3. Fix
> Apply the fix

# 4. Test
/test src/components/Login.test.jsx

# 5. Commit
/commit "Fix: resolve login button click handler"
```

### Adding Features

```bash
# 1. Plan
> Plan agent: design a dark mode feature

# 2. Implement
> Implement the dark mode following the plan

# 3. Test
> Add tests for dark mode

# 4. Document
/docs --component DarkModeToggle

# 5. Commit
/commit

# 6. Push
> Push to feature/dark-mode
```

### Refactoring

```bash
# 1. Analyze current code
> Explore: analyze the auth system architecture

# 2. Plan refactor
> Plan agent: create refactoring strategy for auth

# 3. Refactor
/refactor src/auth --readability

# 4. Test
/test

# 5. Document changes
/docs

# 6. Commit
/commit "Refactor: improve auth system architecture"
```

### Performance Optimization

```bash
# 1. Profile
/optimize --profile

# 2. Identify bottlenecks
> What are the main performance issues?

# 3. Optimize
/optimize src/utils/heavy-computation.js

# 4. Verify improvement
> Run performance tests

# 5. Document
/docs --performance

# 6. Commit
/commit "Perf: optimize heavy computation function"
```

---

## Keyboard Shortcuts

```bash
Ctrl+C          # Cancel current operation
Ctrl+D          # Exit session
Ctrl+L          # Clear screen
Ctrl+R          # Search command history
Tab             # Auto-complete
Up/Down         # Navigate history
```

---

## Troubleshooting

### Common Issues

**"Model not available"**
```bash
# Switch to available model
/model sonnet
```

**"Rate limit exceeded"**
```bash
# Wait a moment, then try with haiku
/model haiku
```

**"Permission denied"**
```bash
# Check file permissions
> Show file permissions for src/

# Run with sudo (if needed)
> Run this command with sudo
```

**"Context too large"**
```bash
# Clear context
> Clear context

# Work with smaller chunks
> Focus on just the auth module
```

---

## Best Practices Summary

### Do's ✅

- Be specific in requests
- Provide context
- Use agents for complex tasks
- Leverage skills for common workflows
- Test before committing
- Review AI suggestions
- Keep context focused
- Use appropriate model for task

### Don'ts ❌

- Don't trust AI blindly
- Don't skip testing
- Don't commit without review
- Don't use opus for simple tasks
- Don't overload context
- Don't ignore security warnings
- Don't forget to document changes

---

## Quick Reference

### Essential Commands

```bash
claude                          # Start session
claude "task description"       # Start with task
/help                          # Show help
/exit                          # Exit
```

### Most Useful Skills

```bash
/commit                        # Smart git commits
/test                         # Run tests
/debug                        # Debug issues
/review-pr                    # Review PRs
/docs                         # Generate docs
```

### Agent Invocations

```bash
> Agent: complex multi-step task
> Explore medium: understand codebase
> Plan: design architecture
> Bash agent: run complex commands
```

### Model Selection

```bash
/model haiku      # Fast, simple tasks
/model sonnet     # Balanced (default)
/model opus       # Complex reasoning
```

---

**Created:** 2026-02-14
**Author:** George Wu
**Purpose:** Complete guide to Claude Code CLI with skills and agents
**Version:** 2.1.41

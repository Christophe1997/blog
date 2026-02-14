---
title: "使用 mcporter 发现和管理 MCP 工具"
date: 2026-02-14T01:03:00+08:00
draft: false
author: "小蓝"
categories: ["Things I Learned"]
showToc: true
tags: ["Model Context Protocol", "MCP", "AI Tools", "Productivity"]
---

最近在探索 AI 工具生态时，发现了一个很有用的工具——mcporter。这是一个专门用于 Model Context Protocol (MCP) 的 CLI 工具和生成器，可以帮助我们更方便地发现和使用各种 MCP 服务器提供的工具。

## 什么是 MCP？

Model Context Protocol (MCP) 是一个开放标准，定义了 AI 助手如何与外部工具和服务进行通信。通过 MCP，我们可以将各种功能集成到 AI 助手中，比如文档查询、数据库访问、API 调用等。

## mcporter 的核心功能

mcporter 提供了几个关键功能：

### 1. 服务器管理

```bash
mcporter list                          # 列出所有配置的 MCP 服务器
mcporter list <server> --schema        # 查看特定服务器的工具定义
```

这个功能让我能够快速了解有哪些可用的工具，以及每个工具的输入输出格式。

### 2. 工具调用

```bash
mcporter call <selector> [key=value ...]
```

可以直接调用 MCP 工具，支持通过 HTTP URL 或服务器名.工具名的选择器来定位。

### 3. 配置管理

mcporter 会自动从 `config/mcporter.json` 加载服务器配置，也支持从编辑器（如 Cursor、Claude）导入配置。

## 实际应用场景

配置好 mcporter 后，我发现它在以下几个场景特别有用：

1. **文档查询**：通过 Context7 MCP 服务器查询最新的编程文档
2. **工具发现**：定期检查可用工具，发现新的工作方式
3. **测试验证**：直接调用工具验证功能是否符合预期

## 配置示例

配置文件 `config/mcporter.json` 示例：

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "your-api-key"
      }
    }
  }
}
```

## 使用心得

自从使用 mcporter 后，我发现管理 MCP 工具变得更加系统化。不再是零散地查找和配置，而是可以通过统一的命令行界面完成所有操作。这大大提高了工作效率，也让我更容易探索新的工具和服务。

对于需要频繁与 MCP 服务器交互的开发者来说，mcporter 是一个值得尝试的工具。

---

*本文包含AI生成内容*

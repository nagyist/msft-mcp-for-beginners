## 入门  

[![构建您的第一个 MCP 服务器](../../../translated_images/zh-CN/04.0ea920069efd979a.webp)](https://youtu.be/sNDZO9N4m9Y)

_(点击上方图片观看本课程的视频)_

本章节包含多个课程：

- **1 您的第一个服务器**，在第一课中，您将学习如何创建第一个服务器并使用检查工具进行检查，这是测试和调试服务器的宝贵方式，[查看课程](01-first-server/README.md)

- **2 客户端**，本课中，您将学习如何编写可以连接到服务器的客户端，[查看课程](02-client/README.md)

- **3 带 LLM 的客户端**，更好的客户端编写方式是添加一个大语言模型（LLM），这样它可以与服务器“协商”要执行的操作，[查看课程](03-llm-client/README.md)

- **4 在 Visual Studio Code 中使用服务器 GitHub Copilot Agent 模式**。本节介绍如何在 Visual Studio Code 内运行 MCP 服务器，[查看课程](04-vscode/README.md)

- **5 stdio 传输服务器**，stdio 传输是用于本地 MCP 服务器与客户端通信的推荐标准，提供基于子进程的安全通信和内置的进程隔离，[查看课程](05-stdio-server/README.md)

- **6 使用 MCP 的 HTTP 流（可流式 HTTP）**。了解现代 HTTP 流传输（根据 [MCP 规范 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/transports/#streamable-http) 推荐的远程 MCP 服务器方案）、进度通知，以及如何使用可流式 HTTP 实现可扩展的实时 MCP 服务器和客户端，[查看课程](06-http-streaming/README.md)

- **7 利用 VSCode 的 AI 工具包** 来使用和测试 MCP 客户端和服务器，[查看课程](07-aitk/README.md)

- **8 测试**，本节重点介绍多种测试服务器和客户端的方法，[查看课程](08-testing/README.md)

- **9 部署**，本章节探讨 MCP 解决方案的多种部署方式，[查看课程](09-deployment/README.md)

- **10 高级服务器用法**，本章涵盖高级服务器使用技巧，[查看课程](./10-advanced/README.md)

- **11 认证**，本章介绍如何添加简单认证，从基础认证到使用 JWT 和 RBAC。推荐从该章节开始，然后查看第 5 章中的高级主题，并通过第 2 章中的建议进行额外的安全加固，[查看课程](./11-simple-auth/README.md)

- **12 MCP 主机**，配置并使用流行的 MCP 主机客户端，包括 Claude Desktop、Cursor、Cline 和 Windsurf。学习传输类型和故障排除，[查看课程](./12-mcp-hosts/README.md)

- **13 MCP 检查器**，使用 MCP 检查器工具交互式调试和测试 MCP 服务器。学习故障排除工具、资源和协议消息，[查看课程](./13-mcp-inspector/README.md)

Model Context Protocol (MCP) 是一个开放协议，用于标准化应用程序向大语言模型（LLM）提供上下文的方式。可以将 MCP 想象成 AI 应用的 USB-C 接口 —— 它提供一种标准化方式，将 AI 模型连接到不同的数据源和工具。

## 学习目标

完成本课程后，您将能够：

- 为 C#、Java、Python、TypeScript 和 JavaScript 配置 MCP 开发环境
- 构建和部署具备自定义功能（资源、提示和工具）的基础 MCP 服务器
- 创建连接 MCP 服务器的主机应用
- 对 MCP 实现进行测试和调试
- 理解常见的设置难点及其解决方案
- 将 MCP 实现连接到流行的大语言模型服务

## 配置您的 MCP 环境

开始使用 MCP 之前，准备好开发环境和理解基本工作流程很重要。本节将指导您完成初始配置步骤，确保 MCP 的顺利入门。

### 前提条件

开始 MCP 开发前，请确保您具备：

- **开发环境**：针对所选编程语言（C#、Java、Python、TypeScript 或 JavaScript）
- **集成开发环境/编辑器**：Visual Studio、Visual Studio Code、IntelliJ、Eclipse、PyCharm 或任何现代代码编辑器
- **包管理器**：NuGet、Maven/Gradle、pip 或 npm/yarn
- **API 密钥**：用于您主机应用中计划使用的任何 AI 服务

### 官方 SDK

接下来的章节将展示使用 Python、TypeScript、Java 和 .NET 构建的解决方案。以下是所有官方支持的 SDK。

MCP 官方提供多语言 SDK（符合 [MCP 规范 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)）：
- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk) - 与微软合作维护
- [Java SDK](https://github.com/modelcontextprotocol/java-sdk) - 与 Spring AI 合作维护
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) - 官方 TypeScript 实现
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk) - 官方 Python 实现（FastMCP）
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk) - 官方 Kotlin 实现
- [Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - 与 Loopwork AI 合作维护
- [Rust SDK](https://github.com/modelcontextprotocol/rust-sdk) - 官方 Rust 实现
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk) - 官方 Go 实现

## 关键要点

- 利用语言特定的 SDK，配置 MCP 开发环境简单快捷
- 构建 MCP 服务器需要创建并注册具备明确定义架构的工具
- MCP 客户端连接服务器和模型以利用增强功能
- 测试和调试对实现可靠 MCP 功能至关重要
- 部署方案涵盖本地开发到云端解决方案多种方式

## 练习

我们提供了一组样例，与本节所有章节中的练习相辅相成。每个章节还有各自的练习和作业。

- [Java 计算器](./samples/java/calculator/README.md)
- [.Net 计算器](../../../03-GettingStarted/samples/csharp)
- [JavaScript 计算器](./samples/javascript/README.md)
- [TypeScript 计算器](./samples/typescript/README.md)
- [Python 计算器](../../../03-GettingStarted/samples/python)

## 额外资源

- [在 Azure 上使用 Model Context Protocol 构建 Agent](https://learn.microsoft.com/azure/developer/ai/intro-agents-mcp)
- [使用 Azure 容器应用的远程 MCP（Node.js/TypeScript/JavaScript）](https://learn.microsoft.com/samples/azure-samples/mcp-container-ts/mcp-container-ts/)
- [.NET OpenAI MCP Agent](https://learn.microsoft.com/samples/azure-samples/openai-mcp-agent-dotnet/openai-mcp-agent-dotnet/)

## 接下来做什么

从第一课开始学习：[创建您的第一个 MCP 服务器](01-first-server/README.md)

完成本模块后，继续学习：[第 4 模块：实用实现](../04-PracticalImplementation/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：  
本文件通过 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 进行翻译。虽然我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。以原始语言的原文文件为权威来源。对于重要信息，建议使用专业人工翻译。对于因使用本翻译产生的任何误解或曲解，我们概不负责。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
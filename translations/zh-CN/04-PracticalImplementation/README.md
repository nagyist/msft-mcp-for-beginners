# 实践实现

[![如何使用真实工具和工作流构建、测试和部署 MCP 应用](../../../translated_images/zh-CN/05.64bea204e25ca891.webp)](https://youtu.be/vCN9-mKBDfQ)

_(点击上方图片观看本课视频)_

实践实现是模型上下文协议（MCP）威力变得具体的地方。虽然理解 MCP 的理论和架构非常重要，但真正的价值出现在你应用这些概念来构建、测试和部署解决实际问题的方案时。本章架起概念知识与实际开发之间的桥梁，指导你完成基于 MCP 应用的落地过程。

无论你是在开发智能助手，将 AI 集成到业务工作流中，还是构建用于数据处理的自定义工具，MCP 都提供了灵活的基础。其语言无关的设计以及针对主流编程语言的官方 SDK，使其易于被各种开发者使用。通过利用这些 SDK，你可以快速原型开发、迭代和在不同平台与环境中扩展你的方案。

在接下来的章节中，你将找到实际示例、示范代码和部署策略，展示如何在 C#、Java（Spring）、TypeScript、JavaScript 和 Python 中实现 MCP。你还将学习如何调试和测试 MCP 服务器、管理 API，以及如何使用 Azure 将解决方案部署到云端。这些实战资源旨在加快你的学习进程，帮助你自信地构建健壮、可投入生产的 MCP 应用。

## 概述

本课聚焦于跨多种编程语言的 MCP 实现实践。我们将探讨如何在 C#、Java（Spring）、TypeScript、JavaScript 和 Python 中使用 MCP SDK 来构建稳健的应用程序，调试和测试 MCP 服务器，并创建可复用的资源、提示和工具。

## 学习目标

完成本课后，你将能够：

- 使用官方 SDK 在多种编程语言中实现 MCP 解决方案
- 系统性地调试和测试 MCP 服务器
- 创建和使用服务器功能（资源、提示和工具）
- 设计用于复杂任务的高效 MCP 工作流
- 优化 MCP 实现以提升性能和可靠性

## 官方 SDK 资源

模型上下文协议为多种语言提供官方 SDK（对应 [MCP 规范 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)）：

- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk)
- [Java（Spring）SDK](https://github.com/modelcontextprotocol/java-sdk) **注意：** 依赖 [Project Reactor](https://projectreactor.io)。详情见 [讨论问题 246](https://github.com/orgs/modelcontextprotocol/discussions/246)。
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk)
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk)

## 使用 MCP SDK

本节提供在多种编程语言中实现 MCP 的实用示例。你可以在 `samples` 目录下找到按语言分类的示范代码。

### 可用示例

代码库包含以下语言的[示例实现](../../../04-PracticalImplementation/samples)：

- [C#](./samples/csharp/README.md)
- [Java（Spring）](./samples/java/containerapp/README.md)
- [TypeScript](./samples/typescript/README.md)
- [JavaScript](./samples/javascript/README.md)
- [Python](./samples/python/README.md)

每个示例演示该语言及生态系统中的核心 MCP 概念和实现模式。

### 实用指南

其他实用 MCP 实现指南：

- [分页与大型结果集](./pagination/README.md) - 处理工具、资源和大型数据集的基于游标的分页

## 核心服务器功能

MCP 服务器可实现以下任意组合的功能：

### 资源

资源为用户或 AI 模型提供上下文和数据：

- 文档仓库
- 知识库
- 结构化数据源
- 文件系统

### 提示

提示是为用户设计的模板化消息和工作流：

- 预定义对话模板
- 引导式交互模式
- 专门的对话结构

### 工具

工具是 AI 模型可执行的函数：

- 数据处理工具
- 外部 API 集成
- 计算功能
- 搜索功能

## 示例实现：C# 实现

官方 C# SDK 仓库包含多个示例实现，演示 MCP 的不同方面：

- **基础 MCP 客户端**：简单示例，展示如何创建 MCP 客户端并调用工具
- **基础 MCP 服务器**：极简服务器实现，包含基本工具注册
- **高级 MCP 服务器**：全功能服务器，含工具注册、认证和错误处理
- **ASP.NET 集成**：展示与 ASP.NET Core 集成的示例
- **工具实现模式**：多种工具实现模式，涵盖不同复杂度

MCP C# SDK 处于预览阶段，API 可能会变动。我们将持续更新本博客以反映 SDK 的发展。

### 主要功能

- [C# MCP Nuget ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol)
- 构建你的[第一个 MCP 服务器](https://devblogs.microsoft.com/dotnet/build-a-model-context-protocol-mcp-server-in-csharp/)

有关完整的 C# 实现示例，请访问[官方 C# SDK 示例仓库](https://github.com/modelcontextprotocol/csharp-sdk)

## 示例实现：Java（Spring）实现

Java（Spring）SDK 提供企业级特性的稳健 MCP 实现选项。

### 主要功能

- Spring 框架集成
- 强类型安全
- 响应式编程支持
- 全面的错误处理

完整的 Java（Spring）实现示例见样例目录中的 [Java（Spring）示例](samples/java/containerapp/README.md)。

## 示例实现：JavaScript 实现

JavaScript SDK 提供轻量且灵活的 MCP 实现方式。

### 主要功能

- 支持 Node.js 和浏览器
- 基于 Promise 的 API
- 便于与 Express 等框架集成
- 支持 WebSocket 流式传输

完整的 JavaScript 实现示例见样例目录中的 [JavaScript 示例](samples/javascript/README.md)。

## 示例实现：Python 实现

Python SDK 以 Pythonic 方式实现 MCP，且与优秀的机器学习框架有良好集成。

### 主要功能

- 使用 asyncio 支持 async/await
- FastAPI 集成
- 简单的工具注册
- 与流行机器学习库的本地集成

完整的 Python 实现示例见样例目录中的 [Python 示例](samples/python/README.md)。

## API 管理

Azure API 管理是保护 MCP 服务器的绝佳方案。思路是在你的 MCP 服务器前面放置一个 Azure API 管理实例，让它负责你可能需要的功能，如：

- 限流
- 令牌管理
- 监控
- 负载均衡
- 安全

### Azure 示例

这里有一个 Azure 示例，即 [创建 MCP 服务器并使用 Azure API 管理保护它](https://github.com/Azure-Samples/remote-mcp-apim-functions-python)。

下图展示了授权流程：

![APIM-MCP](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/mcp-client-authorization.gif?raw=true)

图片中发生的过程：

- 使用 Microsoft Entra 进行身份验证/授权。
- Azure API 管理充当网关，并使用策略来引导和管理流量。
- Azure Monitor 记录所有请求以供后续分析。

#### 授权流程

详细查看授权流程：

![Sequence Diagram](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/infra/app/apim-oauth/diagrams/images/mcp-client-auth.png?raw=true)

#### MCP 授权规范

了解更多关于 [MCP 授权规范](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/authorization/)

## 将远程 MCP 服务器部署到 Azure

让我们来试着部署前面提到的示例：

1. 克隆仓库

    ```bash
    git clone https://github.com/Azure-Samples/remote-mcp-apim-functions-python.git
    cd remote-mcp-apim-functions-python
    ```

1. 注册 `Microsoft.App` 资源提供者。

   - 如果你使用 Azure CLI，运行 `az provider register --namespace Microsoft.App --wait`。
   - 如果你使用 Azure PowerShell，运行 `Register-AzResourceProvider -ProviderNamespace Microsoft.App`。稍后运行 `(Get-AzResourceProvider -ProviderNamespace Microsoft.App).RegistrationState` 检查注册是否完成。

1. 运行此 [azd](https://aka.ms/azd) 命令来配置 API 管理服务、函数应用（含代码）及所有其他必需的 Azure 资源

    ```shell
    azd up
    ```

    该命令将部署所有 Azure 云资源

### 使用 MCP Inspector 测试服务器

1. 在**新终端窗口**中安装并运行 MCP Inspector

    ```shell
    npx @modelcontextprotocol/inspector
    ```

    你应看到类似以下界面：

    ![Connect to Node inspector](../../../translated_images/zh-CN/connect.141db0b2bd05f096.webp)

1. 按住 CTRL 键点击，从应用显示的 URL（例如 [http://127.0.0.1:6274/#resources](http://127.0.0.1:6274/#resources)）加载 MCP Inspector Web 应用
1. 将传输类型设置为 `SSE`
1. 将 URL 设置为执行 `azd up` 后显示的运行中的 API 管理 SSE 端点，并**连接**：

    ```shell
    https://<apim-servicename-from-azd-output>.azure-api.net/mcp/sse
    ```

1. **列出工具**。点击某个工具并**运行工具**。

如果所有步骤成功，你现在应该已连接到 MCP 服务器，并能成功调用工具。

## Microsoft Azure 的 MCP 服务器

[Remote-mcp-functions](https://github.com/Azure-Samples/remote-mcp-functions-dotnet)：该系列仓库是基于 Azure Functions 使用 Python、C# .NET 和 Node/TypeScript 构建和部署自定义远程 MCP 服务器的快速启动模板。

该示例提供完整解决方案，帮助开发者：

- 本地构建和运行：在本地机器上开发和调试 MCP 服务器
- 部署到 Azure：通过简化的 azd up 命令轻松部署到云端
- 从客户端连接：从包括 VS Code 的 Copilot 代理模式和 MCP Inspector 工具在内的多个客户端连接 MCP 服务器

### 主要功能

- 设计即安全：MCP 服务器使用密钥和 HTTPS 实现安全
- 认证选项：支持内置认证和/或 API 管理的 OAuth
- 网络隔离：支持 Azure 虚拟网络（VNET）实现网络隔离
- 无服务器架构：利用 Azure Functions 提供可扩展的事件驱动执行
- 本地开发：提供全面的本地开发和调试支持
- 简化部署：简化的部署流程至 Azure

仓库包含所有必要的配置文件、源代码和基础设施定义，方便快速启动生产级 MCP 服务器实现。

- [Azure 远程 MCP Functions Python](https://github.com/Azure-Samples/remote-mcp-functions-python) - 使用 Azure Functions 和 Python 实现 MCP 的示例

- [Azure 远程 MCP Functions .NET](https://github.com/Azure-Samples/remote-mcp-functions-dotnet) - 使用 Azure Functions 和 C# .NET 实现 MCP 的示例

- [Azure 远程 MCP Functions Node/Typescript](https://github.com/Azure-Samples/remote-mcp-functions-typescript) - 使用 Azure Functions 和 Node/TypeScript 实现 MCP 的示例

## 关键要点

- MCP SDK 提供针对不同语言的工具，用于实现稳健的 MCP 方案
- 调试和测试过程对可靠的 MCP 应用至关重要
- 可复用的提示模板保障 AI 交互的一致性
- 精心设计的工作流能够协调多个工具完成复杂任务
- 实现 MCP 方案需考虑安全、性能和错误处理

## 练习

设计一个解决你领域现实问题的实用 MCP 工作流：

1. 确定 3-4 个可用于解决该问题的工具
2. 绘制展示这些工具如何交互的工作流图
3. 使用你偏好的语言实现其中一个工具的基础版本
4. 创建一个提示模板，帮助模型有效使用你的工具

## 附加资源

---

## 接下来

下一章：[高级主题](../05-AdvancedTopics/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：  
本文件由人工智能翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译而成。虽然我们力求准确，但请注意，自动翻译可能存在错误或不准确之处。原始文档的母语版本应被视为权威来源。对于重要信息，建议采用专业人工翻译。因使用本翻译版本而产生的任何误解或误释，本公司概不负责。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
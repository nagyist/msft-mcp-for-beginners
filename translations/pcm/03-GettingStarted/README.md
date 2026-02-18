## Getting Started  

[![Build Your First MCP Server](../../../translated_images/pcm/04.0ea920069efd979a.webp)](https://youtu.be/sNDZO9N4m9Y)

_(Click di image wey dey top to watch video for dis lesson)_

Dis section get plenty lessons:

- **1 Your first server**, for dis first lesson, you go learn how to create your first server and check am wit inspector tool, one beta way to test and debug your server, [to the lesson](01-first-server/README.md)

- **2 Client**, for dis lesson, you go learn how to write client wey fit connect to your server, [to the lesson](02-client/README.md)

- **3 Client with LLM**, better way to write client na to add LLM to am so e fit "negotiate" wit your server on wetin e go do, [to the lesson](03-llm-client/README.md)

- **4 Consuming a server GitHub Copilot Agent mode in Visual Studio Code**. Here, we dey look how to run our MCP Server inside Visual Studio Code, [to the lesson](04-vscode/README.md)

- **5 stdio Transport Server** stdio transport na di recommended standard for local MCP server-to-client communication, e dey provide secure subprocess-based communication wit built-in process isolation [to the lesson](05-stdio-server/README.md)

- **6 HTTP Streaming with MCP (Streamable HTTP)**. Learn about modern HTTP streaming transport (di recommended approach for remote MCP servers per [MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/transports/#streamable-http)), progress notifications, and how to implement scalable, real-time MCP servers and clients using Streamable HTTP. [to the lesson](06-http-streaming/README.md)

- **7 Utilising AI Toolkit for VSCode** to consume and test your MCP Clients and Servers [to the lesson](07-aitk/README.md)

- **8 Testing**. Here we go focus specially how we fit test our server and client for different ways, [to the lesson](08-testing/README.md)

- **9 Deployment**. Dis chapter go look different ways to deploy your MCP solutions, [to the lesson](09-deployment/README.md)

- **10 Advanced server usage**. Dis chapter dey cover advanced server usage, [to the lesson](./10-advanced/README.md)

- **11 Auth**. Dis chapter dey cover how to add simple auth, from Basic Auth to using JWT and RBAC. We dey encourage you to start here and den check Advanced Topics for Chapter 5 and do extra security hardening as dem recommend for Chapter 2, [to the lesson](./11-simple-auth/README.md)

- **12 MCP Hosts**. Configure and use popular MCP host clients like Claude Desktop, Cursor, Cline, and Windsurf. Learn transport types and troubleshooting, [to the lesson](./12-mcp-hosts/README.md)

- **13 MCP Inspector**. Debug and test your MCP servers interactively using MCP Inspector tool. Learn how to troubleshoot tools, resources, and protocol messages, [to the lesson](./13-mcp-inspector/README.md)

The Model Context Protocol (MCP) na open protocol wey dey standardize how applications dey provide context to LLMs. Think of MCP like USB-C port for AI applications - e dey provide one standard way to connect AI models to different data sources and tools.

## Learning Objectives

By di end of dis lesson, you go fit:

- Set up development environments for MCP for C#, Java, Python, TypeScript, and JavaScript
- Build and deploy basic MCP servers wit custom features (resources, prompts, and tools)
- Create host applications wey connect to MCP servers
- Test and debug MCP implementations
- Understand common setup wahala and how to solve am
- Connect your MCP implementations to popular LLM services

## Setting Up Your MCP Environment

Before you start to dey work wit MCP, e important to prepare your development environment and sabi di basic workflow. Dis section go guide you through di initial setup steps make you start smoothly wit MCP.

### Prerequisites

Before you waka enter MCP development, make sure say you get:

- **Development Environment**: For your chosen language (C#, Java, Python, TypeScript, or JavaScript)
- **IDE/Editor**: Visual Studio, Visual Studio Code, IntelliJ, Eclipse, PyCharm, or any modern code editor
- **Package Managers**: NuGet, Maven/Gradle, pip, or npm/yarn
- **API Keys**: For any AI services wey you want use for your host applications


### Official SDKs

For the next chapters you go see solutions wey dem build using Python, TypeScript, Java, and .NET. Dis na all di official supported SDKs.

MCP dey provide official SDKs for plenty languages (wey dem align wit [MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)):
- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk) - Dem dey maintain am with Microsoft
- [Java SDK](https://github.com/modelcontextprotocol/java-sdk) - Dem dey maintain am with Spring AI
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) - Official TypeScript implementation
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk) - Official Python implementation (FastMCP)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk) - Official Kotlin implementation
- [Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - Dem dey maintain am with Loopwork AI
- [Rust SDK](https://github.com/modelcontextprotocol/rust-sdk) - Official Rust implementation
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk) - Official Go implementation

## Key Takeaways

- Setting up MCP development environment easy wit language-specific SDKs
- Building MCP servers mean to create and register tools wit clear schemas
- MCP clients dey connect to servers and models to take advantage of extended features
- Testing and debugging na important for secure MCP implementations
- Deployment options dey from local development reach cloud-based solutions

## Practicing

We get samples wey go complement di exercises wey you go see for all chapters for dis section. Each chapter also get dia own exercises and assignments

- [Java Calculator](./samples/java/calculator/README.md)
- [.Net Calculator](../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](./samples/javascript/README.md)
- [TypeScript Calculator](./samples/typescript/README.md)
- [Python Calculator](../../../03-GettingStarted/samples/python)

## Additional Resources

- [Build Agents using Model Context Protocol on Azure](https://learn.microsoft.com/azure/developer/ai/intro-agents-mcp)
- [Remote MCP with Azure Container Apps (Node.js/TypeScript/JavaScript)](https://learn.microsoft.com/samples/azure-samples/mcp-container-ts/mcp-container-ts/)
- [.NET OpenAI MCP Agent](https://learn.microsoft.com/samples/azure-samples/openai-mcp-agent-dotnet/openai-mcp-agent-dotnet/)

## What's next

Start wit di first lesson: [Creating your first MCP Server](01-first-server/README.md)

After you don finish dis module, continue to: [Module 4: Practical Implementation](../04-PracticalImplementation/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis dokument don translate by AI translation service wey dem dey call [Co-op Translator](https://github.com/Azure/co-op-translator). Even though we dey try make am correct, abeg sabi say automated translations fit get mistake or no too correct. Di original dokument for im own language na di correct one. If na serious tin, make you use professional human translation. We no go responsible for any wrong understanding or wrong sense wey fit come from dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
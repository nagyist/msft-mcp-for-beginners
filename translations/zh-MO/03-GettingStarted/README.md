## Getting Started  

[![Build Your First MCP Server](../../../translated_images/zh-MO/04.0ea920069efd979a.webp)](https://youtu.be/sNDZO9N4m9Y)

_(點擊上方圖片觀看本課程影片)_

本節包含多個課程：

- **1 Your first server**，在第一課中，你將學習如何建立你的第一個伺服器，並使用檢查器工具進行檢查，這是測試及除錯伺服器的重要方式，[前往課程](01-first-server/README.md)

- **2 Client**，在這課中，你將學習如何撰寫可連接至伺服器的客戶端，[前往課程](02-client/README.md)

- **3 Client with LLM**，撰寫客戶端更佳的方法是加入LLM，使它能與伺服器「協商」應該進行的操作，[前往課程](03-llm-client/README.md)

- **4 Consuming a server GitHub Copilot Agent mode in Visual Studio Code**。此處，我們將探討如何在Visual Studio Code內部執行MCP伺服器，[前往課程](04-vscode/README.md)

- **5 stdio Transport Server** stdio傳輸是本地MCP伺服器與客戶端通訊的推薦標準，提供安全的子程序基礎通訊和內建程序隔離，[前往課程](05-stdio-server/README.md)

- **6 HTTP Streaming with MCP (Streamable HTTP)**。了解現代HTTP串流傳輸（依據[MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/transports/#streamable-http)建議的遠端MCP伺服器方式）、進度通知，以及如何使用可串流HTTP實作可擴充的即時MCP伺服器與客戶端。[前往課程](06-http-streaming/README.md)

- **7 Utilising AI Toolkit for VSCode** 以便使用AI工具包測試與使用你的MCP客戶端與伺服器 [前往課程](07-aitk/README.md)

- **8 Testing**。本章特別著重於多樣方式測試伺服器與客戶端，[前往課程](08-testing/README.md)

- **9 Deployment**。本章將介紹多種部署MCP方案的方法，[前往課程](09-deployment/README.md)

- **10 Advanced server usage**。本章涵蓋進階伺服器用法，[前往課程](./10-advanced/README.md)

- **11 Auth**。本章介紹如何新增簡易認證，從基礎認證到使用JWT與RBAC。建議先從此章開始，然後參考第5章的進階主題，以及第2章的安全強化建議，[前往課程](./11-simple-auth/README.md)

- **12 MCP Hosts**。設定並使用熱門的MCP主機客戶端，包含Claude Desktop, Cursor, Cline和Windsurf。了解傳輸類型及故障排除，[前往課程](./12-mcp-hosts/README.md)

- **13 MCP Inspector**。使用MCP檢查器工具互動式除錯與測試你的MCP伺服器。學習故障排除工具、資源及協定訊息，[前往課程](./13-mcp-inspector/README.md)

Model Context Protocol (MCP) 是一個開放協定，標準化應用程式如何提供上下文給大型語言模型。可以將MCP想像成AI應用的USB-C埠 — 它提供了將AI模型與不同資料源及工具連接的標準化方式。

## Learning Objectives

完成本課程後，你將能夠：

- 設置C#、Java、Python、TypeScript和JavaScript的MCP開發環境
- 建置並部署帶有自訂功能（資源、提示與工具）的基本MCP伺服器
- 建立能連接至MCP伺服器的主機應用程式
- 測試與除錯MCP實作
- 瞭解常見設定挑戰及其解決方案
- 連接你的MCP實作至熱門大型語言模型服務

## Setting Up Your MCP Environment

在開始使用MCP之前，準備好開發環境並了解基本工作流程非常重要。本節將引導你完成初始設定，以確保你能順利開始使用MCP。

### Prerequisites

開始MCP開發之前，要確定你已具備：

- **開發環境**：依你選擇的程式語言（C#, Java, Python, TypeScript或JavaScript）
- **IDE/編輯器**：Visual Studio、Visual Studio Code、IntelliJ、Eclipse、PyCharm或任何現代程式編輯器
- **套件管理工具**：NuGet、Maven/Gradle、pip或npm/yarn
- **API 金鑰**：用於你規劃在主機應用中使用的任何AI服務

### Official SDKs

接下來的章節將展示使用Python、TypeScript、Java和.NET的解決方案。這裡列出所有官方支持的SDK。

MCP提供多語言官方SDK（符合[MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)）：
- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk) — 與Microsoft合作維護
- [Java SDK](https://github.com/modelcontextprotocol/java-sdk) — 與Spring AI合作維護
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) — 官方TypeScript實作
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk) — 官方Python實作 (FastMCP)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk) — 官方Kotlin實作
- [Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) — 與Loopwork AI合作維護
- [Rust SDK](https://github.com/modelcontextprotocol/rust-sdk) — 官方Rust實作
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk) — 官方Go實作

## Key Takeaways

- 使用語言特定的SDK設置MCP開發環境相當簡單
- 建置MCP伺服器涉及創建並註冊具明確架構的工具
- MCP客戶端連接至伺服器和模型以發揮擴展功能
- 測試和除錯是保證MCP實作可靠的關鍵
- 部署選項從本地開發到雲端方案皆有支援

## Practicing

我們提供一組範例，搭配本節中所有章節的練習使用。此外，每章也有其專屬練習和作業：

- [Java 計算器](./samples/java/calculator/README.md)
- [.Net 計算器](../../../03-GettingStarted/samples/csharp)
- [JavaScript 計算器](./samples/javascript/README.md)
- [TypeScript 計算器](./samples/typescript/README.md)
- [Python 計算器](../../../03-GettingStarted/samples/python)

## Additional Resources

- [在 Azure 上使用 Model Context Protocol 構建代理](https://learn.microsoft.com/azure/developer/ai/intro-agents-mcp)
- [Azure 容器應用遠端MCP (Node.js/TypeScript/JavaScript)](https://learn.microsoft.com/samples/azure-samples/mcp-container-ts/mcp-container-ts/)
- [.NET OpenAI MCP 代理](https://learn.microsoft.com/samples/azure-samples/openai-mcp-agent-dotnet/openai-mcp-agent-dotnet/)

## What's next

從第一課開始： [建立你的第一個MCP伺服器](01-first-server/README.md)

完成本模組後，繼續學習：[模組4：實務應用](../04-PracticalImplementation/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件係使用人工智能翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確度，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議採用專業人工翻譯。對於因使用本翻譯而引起的任何誤解或誤釋，我們概不負責。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
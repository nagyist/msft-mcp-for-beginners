## Getting Started  

[![Build Your First MCP Server](../../../translated_images/zh-HK/04.0ea920069efd979a.webp)](https://youtu.be/sNDZO9N4m9Y)

_(點擊上面圖片觀看本課程視頻)_

本章包括多個課程：

- **1 你的第一個伺服器**，在第一課中，你將學習如何創建你的第一個伺服器並使用檢查器工具檢視它，這是測試和調試伺服器的寶貴方法，[到課程](01-first-server/README.md)

- **2 客戶端**，在此課程中，你將學習如何編寫可以連接到伺服器的客戶端，[到課程](02-client/README.md)

- **3 帶 LLM 的客戶端**，更好的編寫客戶端方法是為其添加 LLM，讓它能“協商”伺服器需要執行的操作，[到課程](03-llm-client/README.md)

- **4 在 Visual Studio Code 中使用 GitHub Copilot Agent 模式 消費伺服器**。這裡介紹如何在 Visual Studio Code 內運行 MCP 伺服器，[到課程](04-vscode/README.md)

- **5 stdio 傳輸伺服器** stdio 傳輸是 MCP 伺服器與客戶端本地通信的推薦標準，通過子進程通信實現安全隔離 [到課程](05-stdio-server/README.md)

- **6 使用 MCP 的 HTTP 串流（可串流的 HTTP）**。了解現代 HTTP 串流傳輸（根據 [MCP 規範 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/transports/#streamable-http) 為遠程 MCP 伺服器推薦方法）、進度通知，以及如何使用可串流的 HTTP 實現可擴展、實時 MCP 伺服器與客戶端，[到課程](06-http-streaming/README.md)

- **7 利用 AI 工具包於 VSCode**，消費與測試你的 MCP 客戶端與伺服器，[到課程](07-aitk/README.md)

- **8 測試**。重點介紹如何以不同方式測試伺服器與客戶端，[到課程](08-testing/README.md)

- **9 部署**。探討部署 MCP 解決方案的不同方法，[到課程](09-deployment/README.md)

- **10 進階伺服器用法**。涵蓋進階伺服器使用技巧，[到課程](./10-advanced/README.md)

- **11 認證**。介紹如何加入簡單認證，從 Basic 認證到使用 JWT 與 RBAC。建議由此開始，然後參考第 5 章的進階主題，並透過第 2 章的推薦進行額外安全加固，[到課程](./11-simple-auth/README.md)

- **12 MCP 主機**。配置並使用流行的 MCP 主機客戶端，如 Claude Desktop、Cursor、Cline 及 Windsurf。學習傳輸類型及故障排查，[到課程](./12-mcp-hosts/README.md)

- **13 MCP 檢查器**。使用 MCP 檢查器工具互動式調試和測試你的 MCP 伺服器。學習故障排查工具、資源和協議訊息，[到課程](./13-mcp-inspector/README.md)

模型上下文協議（MCP）是一種開放協議，標準化應用程序如何向 LLM 提供上下文。可以把 MCP 想像成 AI 應用的 USB-C 埠——它提供一種標準化方式將 AI 模型連接到不同資料來源和工具。

## 學習目標

完成本課程後，你將能夠：

- 設定 C#、Java、Python、TypeScript 和 JavaScript 的 MCP 開發環境
- 建立並部署帶有自訂功能（資源、提示及工具）的基本 MCP 伺服器
- 創建連接 MCP 伺服器的主機應用
- 測試和調試 MCP 的實現
- 瞭解常見安裝挑戰及其解決方案
- 將你的 MCP 實現連接到流行的 LLM 服務

## 設定你的 MCP 環境

開始 MCP 工作前，重要的是準備你的開發環境並了解基本工作流程。本節將引導你完成初始設定步驟，確保順利開始使用 MCP。

### 先決條件

開始 MCP 開發前，請確保你已經具備：

- **開發環境**：依據你選擇的語言 (C#、Java、Python、TypeScript 或 JavaScript)
- **IDE/編輯器**：Visual Studio、Visual Studio Code、IntelliJ、Eclipse、PyCharm 或任何現代程式碼編輯器
- **套件管理器**：NuGet、Maven/Gradle、pip 或 npm/yarn
- **API 金鑰**：用於計劃在主機應用中使用的任何 AI 服務

### 官方 SDKs

未來各章節會展示使用 Python、TypeScript、Java 與 .NET 的方案。以下是所有官方支持的 SDK。

MCP 提供多種語言官方 SDK（符合 [MCP 規範 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)）：
- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk) - 與 Microsoft 合作維護
- [Java SDK](https://github.com/modelcontextprotocol/java-sdk) - 與 Spring AI 合作維護
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) - 官方 TypeScript 實現
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk) - 官方 Python 實現（FastMCP）
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk) - 官方 Kotlin 實現
- [Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - 與 Loopwork AI 合作維護
- [Rust SDK](https://github.com/modelcontextprotocol/rust-sdk) - 官方 Rust 實現
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk) - 官方 Go 實現

## 重要摘要

- 使用語言專屬 SDK，搭建 MCP 開發環境相當簡單
- 建立 MCP 伺服器涉及創建並註冊具清晰結構的工具
- MCP 客戶端連接伺服器和模型以擴展功能
- 測試和調試對可靠的 MCP 實現至關重要
- 部署方案涵蓋本地開發到雲端解決方案

## 練習

我們提供了一組示例，補充本節所有章節的練習。此外，每章也包含自己專屬的練習與作業

- [Java 計算器](./samples/java/calculator/README.md)
- [.Net 計算器](../../../03-GettingStarted/samples/csharp)
- [JavaScript 計算器](./samples/javascript/README.md)
- [TypeScript 計算器](./samples/typescript/README.md)
- [Python 計算器](../../../03-GettingStarted/samples/python)

## 額外資源

- [在 Azure 上使用模型上下文協議構建代理](https://learn.microsoft.com/azure/developer/ai/intro-agents-mcp)
- [Azure 容器應用上的遠端 MCP (Node.js/TypeScript/JavaScript)](https://learn.microsoft.com/samples/azure-samples/mcp-container-ts/mcp-container-ts/)
- [.NET OpenAI MCP 代理](https://learn.microsoft.com/samples/azure-samples/openai-mcp-agent-dotnet/openai-mcp-agent-dotnet/)

## 下一步

從第一課開始：[建立你的第一個 MCP 伺服器](01-first-server/README.md)

完成本模組後，繼續： [模組 4：實踐應用](../04-PracticalImplementation/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：  
本文件係使用人工智能翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我哋致力確保準確性，但請注意，自動翻譯可能包含錯誤或不準確之處。原文版本應視為具權威性嘅資料來源。對於重要資訊，建議採用專業人工翻譯。我哋對因使用此翻譯而引致嘅任何誤解或誤釋概不負責。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
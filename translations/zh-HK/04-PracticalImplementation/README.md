# 實踐應用

[![如何使用真實工具和工作流程構建、測試及部署 MCP 應用程式](../../../translated_images/zh-HK/05.64bea204e25ca891.webp)](https://youtu.be/vCN9-mKBDfQ)

_(點擊上方圖片觀看本課影片)_

實踐應用是模型上下文協議（MCP）威力具體顯現的時刻。理解 MCP 的理論及架構固然重要，但真正的價值是在於你將這些概念應用於構建、測試及部署解決實際問題的方案。本章連接概念知識與實作開發，指導你如何將基於 MCP 的應用程式帶入現實。

無論你是在開發智慧助理、將 AI 整合入商業工作流，還是構建自訂資料處理工具，MCP 都提供了靈活的基礎。它的語言無關設計與多種熱門程式語言的官方 SDK，都讓廣大開發者能輕鬆入門。透過這些 SDK，你可快速原型設計、迭代，並擴展你的方案至不同平台與環境。

以下章節將提供實務範例、示範程式碼與部署策略，說明如何使用 C#、Java 搭配 Spring、TypeScript、JavaScript 與 Python 實現 MCP。你還將學會如何除錯與測試 MCP 伺服器、管理 API，以及利用 Azure 部署方案。這些動手資源旨在加速你的學習，幫助你自信地建立健全且能投入生產的 MCP 應用程式。

## 概覽

本課聚焦於 MCP 在多種程式語言中實作的實務面。我們將探討如何使用 C#、Java 搭配 Spring、TypeScript、JavaScript 及 Python 等 MCP SDK，來建構穩健的應用程式、除錯並測試 MCP 伺服器，以及建立可重用的資源、提示與工具。

## 學習目標

本課結束後，你將能夠：

- 使用多種程式語言的官方 SDK 實作 MCP 解決方案
- 系統化除錯與測試 MCP 伺服器
- 創建及使用伺服器功能（資源、提示與工具）
- 設計有效的 MCP 工作流程來處理複雜任務
- 針對效能與可靠性優化 MCP 實現

## 官方 SDK 資源

模型上下文協議提供多種語言的官方 SDK（符合 [MCP 規範 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)）：

- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk)
- [Java 搭配 Spring SDK](https://github.com/modelcontextprotocol/java-sdk) **注意：** 需要依賴 [Project Reactor](https://projectreactor.io)。（見 [討論議題 246](https://github.com/orgs/modelcontextprotocol/discussions/246)。）
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk)
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk)

## 使用 MCP SDK

本節提供多種程式語言的 MCP 實務範例。你可於 `samples` 目錄找到依語言組織的範本程式碼。

### 可用範例

本程式庫包含以下 [範例實作](../../../04-PracticalImplementation/samples)：

- [C#](./samples/csharp/README.md)
- [Java 搭配 Spring](./samples/java/containerapp/README.md)
- [TypeScript](./samples/typescript/README.md)
- [JavaScript](./samples/javascript/README.md)
- [Python](./samples/python/README.md)

每個範例演示該語言與生態系中 MCP 的關鍵概念與實作模式。

### 實務指南

更多實際 MCP 實作指南：

- [分頁與巨量結果集](./pagination/README.md) - 處理以游標分頁的工具、資源與大型資料集

## 核心伺服器功能

MCP 伺服器可實作以下任意組合的功能：

### 資源

提供上下文與資料供使用者或 AI 模型使用：

- 文件庫
- 知識庫
- 結構化資料源
- 檔案系統

### 提示

為使用者提供範本訊息與工作流程：

- 預先定義的對話範本
- 引導式互動範例
- 專門設計的對話結構

### 工具

讓 AI 模型可執行的函式：

- 資料處理工具
- 外部 API 整合
- 計算能力
- 搜尋功能

## 範例實作：C# 實作

官方 C# SDK 程式庫包含多個示範 MCP 不同面向的範例：

- **基本 MCP 用戶端**：示範如何建立 MCP 用戶端並呼叫工具的簡單範例
- **基本 MCP 伺服器**：具備簡易工具註冊的最小伺服器實作
- **進階 MCP 伺服器**：完整功能伺服器，包括工具註冊、認證和錯誤處理
- **ASP.NET 整合**：展示與 ASP.NET Core 整合的範例
- **工具實作模式**：涵蓋不同複雜度工具的多樣實作模式

MCP C# SDK 尚處於預覽階段，API 可能變動。隨著 SDK 進化，我們將持續更新本部落格。

### 主要功能

- [C# MCP Nuget ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol)
- 建置你的 [第一個 MCP 伺服器](https://devblogs.microsoft.com/dotnet/build-a-model-context-protocol-mcp-server-in-csharp/)。

完整 C# 實作範例，請參見 [官方 C# SDK 範例程式庫](https://github.com/modelcontextprotocol/csharp-sdk)

## 範例實作：Java 搭配 Spring 實作

Java 搭配 Spring SDK 提供企業級功能的堅固 MCP 實作選項。

### 主要功能

- Spring 框架整合
- 強類型安全
- 反應式編程支援
- 全面錯誤處理

有關完整 Java 搭配 Spring 實作範例，請見 samples 目錄下的 [Java 搭配 Spring 範例](samples/java/containerapp/README.md)。

## 範例實作：JavaScript 實作

JavaScript SDK 提供輕量且靈活的 MCP 實作方案。

### 主要功能

- 支援 Node.js 與瀏覽器
- 以 Promise 為基礎的 API
- 易於與 Express 及其他框架整合
- 支援 WebSocket 串流

完整 JavaScript 實作範例，請見 samples 目錄下的 [JavaScript 範例](samples/javascript/README.md)。

## 範例實作：Python 實作

Python SDK 提供 Python 風格的 MCP 實作，並與優秀的機器學習框架深度整合。

### 主要功能

- 支援 asyncio 的 async/await
- 與 FastAPI 整合``
- 簡易的工具註冊
- 原生整合熱門機器學習庫

完整 Python 實作範例，請見 samples 目錄下的 [Python 範例](samples/python/README.md)。

## API 管理

Azure API 管理是保護 MCP 伺服器的良好方案。概念是將 Azure API 管理實例置於 MCP 伺服器前端，讓它處理你可能需要的特性，包括：

- 限流
- 令牌管理
- 監控
- 負載平衡
- 安全性

### Azure 範例

這裡有一個 Azure 範例，正是如此操作，也就是 [建立 MCP 伺服器並用 Azure API 管理保護它](https://github.com/Azure-Samples/remote-mcp-apim-functions-python)。

以下圖片示範授權流程：

![APIM-MCP](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/mcp-client-authorization.gif?raw=true)

圖片中發生的情況：

- 使用 Microsoft Entra 進行驗證/授權。
- Azure API 管理作為閘道，透過策略引導及管理流量。
- Azure 監控記錄所有請求供後續分析。

#### 授權流程

以下是授權流程的詳細說明：

![Sequence Diagram](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/infra/app/apim-oauth/diagrams/images/mcp-client-auth.png?raw=true)

#### MCP 授權規範

深入瞭解 [MCP 授權規範](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/authorization/)

## 部署遠端 MCP 伺服器到 Azure

讓我們嘗試部署前述的範例：

1. 複製程式庫

    ```bash
    git clone https://github.com/Azure-Samples/remote-mcp-apim-functions-python.git
    cd remote-mcp-apim-functions-python
    ```

1. 註冊 `Microsoft.App` 資源提供者。

   - 若使用 Azure CLI，執行 `az provider register --namespace Microsoft.App --wait`。
   - 若使用 Azure PowerShell，執行 `Register-AzResourceProvider -ProviderNamespace Microsoft.App`。稍後執行 `(Get-AzResourceProvider -ProviderNamespace Microsoft.App).RegistrationState` 確認註冊完成。

1. 執行此 [azd](https://aka.ms/azd) 指令，佈建 API 管理服務、函式應用（含程式碼）以及所有其他必要的 Azure 資源

    ```shell
    azd up
    ```

    此指令應會在 Azure 上部署所有雲端資源

### 使用 MCP Inspector 測試你的伺服器

1. 於 **新終端機視窗** 安裝並執行 MCP Inspector

    ```shell
    npx @modelcontextprotocol/inspector
    ```

    你將看到類似介面：

    ![Connect to Node inspector](../../../translated_images/zh-HK/connect.141db0b2bd05f096.webp)

1. 按 CTRL 點擊，從 App 顯示的網址載入 MCP Inspector 網頁應用（例如 [http://127.0.0.1:6274/#resources](http://127.0.0.1:6274/#resources)）
1. 將傳輸類型設定為 `SSE`
1. 將 URL 設為 `azd up` 後顯示的 API 管理 SSE 端點並 **連線**：

    ```shell
    https://<apim-servicename-from-azd-output>.azure-api.net/mcp/sse
    ```

1. **列出工具**。點擊某工具並 **執行工具**。

若以上所有步驟都正常，代表你已成功連接 MCP 伺服器並成功呼叫工具。

## 適用 Azure 的 MCP 伺服器

[Remote-mcp-functions](https://github.com/Azure-Samples/remote-mcp-functions-dotnet)：此系列程式庫為快速啟動範本，協助開發者使用 Azure Functions 搭配 Python、C# .NET 或 Node/TypeScript 構建並部署自訂遠端 MCP（模型上下文協議）伺服器。

這些範例提供完整方案，讓開發者能：

- 在本機建置及執行：於本機開發與除錯 MCP 伺服器
- 部署至 Azure：使用簡單 azd up 指令輕鬆部署至雲端
- 從用戶端連接：支援包括 VS Code Copilot 代理模式及 MCP Inspector 工具等多種用戶端連線

### 主要功能

- 安全設計：MCP 伺服器使用金鑰和 HTTPS 保護
- 認證選項：支援 OAuth（透過內建認證及/或 API 管理）
- 網路隔離：可使用 Azure 虛擬網路（VNET）實現隔離
- 無伺服器架構：運用 Azure Functions 實現可擴充、事件驅動的執行
- 本地開發：全面的本地開發與除錯支援
- 簡易部署：簡化的 Azure 部署流程

程式庫包含所有必要的組態檔、原始碼及基礎架構定義，可快速著手打造生產等級的 MCP 伺服器實現。

- [Azure Remote MCP Functions Python](https://github.com/Azure-Samples/remote-mcp-functions-python) - 使用 Azure Functions 與 Python 的 MCP 範例實作

- [Azure Remote MCP Functions .NET](https://github.com/Azure-Samples/remote-mcp-functions-dotnet) - 使用 Azure Functions 與 C# .NET 的 MCP 範例實作

- [Azure Remote MCP Functions Node/Typescript](https://github.com/Azure-Samples/remote-mcp-functions-typescript) - 使用 Azure Functions 與 Node/TypeScript 的 MCP 範例實作。

## 主要結論

- MCP SDK 提供特定語言的工具，協助構建穩健的 MCP 解決方案
- 除錯與測試流程對穩定的 MCP 應用至關重要
- 可重用的提示範本促進一致的 AI 互動
- 精心設計的工作流程可協調多工具完成複雜任務
- 實作 MCP 解決方案須考量安全性、效能及錯誤處理

## 練習

設計一個針對你領域中真實問題的實務 MCP 工作流程：

1. 識別 3-4 種有助於解決此問題的工具
2. 創建工作流程圖，展示這些工具如何互動
3. 使用你偏好的程式語言實作一個工具的基本版本
4. 建立一個提示範本，幫助模型有效地使用你的工具

## 其他資源

---

## 下一步

下一章：[進階主題](../05-AdvancedTopics/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：  
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們致力於確保翻譯的準確性，但請注意自動翻譯可能存在錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議採用專業人工翻譯。我們對因使用此翻譯而導致的任何誤解或誤釋不承擔任何責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
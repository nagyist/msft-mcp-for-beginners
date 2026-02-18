# 部署 MCP 伺服器

部署你的 MCP 伺服器讓其他人能在本地環境之外存取其工具和資源。根據你對可擴充性、可靠性和管理便利性的需求，有數種部署策略可以考慮。以下你會找到針對本地、容器及雲端部署 MCP 伺服器的指南。

## 概覽

本課程涵蓋如何部署你的 MCP 伺服器應用程式。

## 學習目標

課程結束後，你將可以：

- 評估不同的部署方法。
- 部署你的應用程式。

## 本地開發與部署

如果你的伺服器是設計讓使用者在自己的機器上運行並使用，可以遵循以下步驟：

1. **下載伺服器**。如果你不是寫伺服器的開發者，請先下載它到你的機器。  
1. **啟動伺服器程序**：執行你的 MCP 伺服器應用程式。

針對 SSE（標準 IO 類型伺服器不需要）：

1. **設定網路**：確保伺服器能在預期的連接埠被存取。  
1. **連接客戶端**：使用本地連結 URL，例如 `http://localhost:3000`。

## 雲端部署

MCP 伺服器可以部署到各種雲端平台：

- **無伺服器函式**：將輕量 MCP 伺服器部署為無伺服器函式。  
- **容器服務**：使用 Azure Container Apps、AWS ECS 或 Google Cloud Run 等服務。  
- **Kubernetes**：部署及管理 MCP 伺服器至 Kubernetes 叢集提供高可用性。

### 範例：Azure Container Apps

Azure Container Apps 支援部署 MCP 伺服器，目前仍在持續開發中，現階段支援 SSE 伺服器。

你可以按照下列方式操作：

1. 克隆一個程式庫：

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```


1. 本地執行測試：

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```


1. 想在本地試用，在 *.vscode* 資料夾建立 *mcp.json* 檔案並加入以下內容：

  ```json
  {
      "inputs": [
          {
              "type": "promptString",
              "id": "weather-api-key",
              "description": "Weather API Key",
              "password": true
          }
      ],
      "servers": {
          "weather-sse": {
              "type": "sse",
              "url": "http://localhost:8000/sse",
              "headers": {
                  "x-api-key": "${input:weather-api-key}"
              }
          }
      }
  }
  ```


  一旦 SSE 伺服器啟動，你可以點擊 JSON 檔案中的播放圖示，GitHub Copilot 就會抓取伺服器上的工具，會看到工具圖示。

1. 進行部署，執行以下命令：

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```


就這麼簡單，透過以上步驟，你可以本地部署，也可以部署到 Azure。

## 其他資源

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)  
- [Azure Container Apps 文章](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)  
- [Azure Container Apps MCP 程式庫](https://github.com/anthonychu/azure-container-apps-mcp-sample)  


## 下一步

- 下一篇：[進階伺服器主題](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件係使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件之母語版本應視為權威來源。對於重要資訊，建議採用專業人工翻譯。我們不對因使用本翻譯所導致之任何誤解或誤譯負責。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
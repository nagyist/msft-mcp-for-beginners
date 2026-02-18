# 部署 MCP 伺服器

部署你的 MCP 伺服器允許其他人存取該伺服器的工具和資源，超越你的本地環境。根據你的可擴展性、可靠性和管理便利性的需求，有多種部署策略可供考慮。以下將為你提供在本地、容器和雲端部署 MCP 伺服器的指引。

## 概述

本課程涵蓋如何部署你的 MCP 伺服器應用程式。

## 學習目標

完成本課程後，你將能夠：

- 評估不同的部署方法。
- 部署你的應用程式。

## 本地開發與部署

如果你的伺服器是為了在使用者機器上運行而設計，則可遵循以下步驟：

1. **下載伺服器**。如果你沒有自己撰寫伺服器，請先將其下載到你的機器上。
1. **啟動伺服器程序**：運行你的 MCP 伺服器應用程式。

針對 SSE（標準輸入輸出類型伺服器則無需此步驟）

1. **設定網路**：確保伺服器可在預期的埠口存取。
1. **連接客戶端**：使用本地連線 URL，例如 `http://localhost:3000`。

## 雲端部署

MCP 伺服器可部署至各種雲端平台：

- **無伺服器函數**：將輕量級 MCP 伺服器作為無伺服器函數部署。
- **容器服務**：使用 Azure Container Apps、AWS ECS 或 Google Cloud Run 等服務。
- **Kubernetes**：將 MCP 伺服器部署並管理於 Kubernetes 叢集，以達高可用性。

### 範例：Azure Container Apps

Azure Container Apps 支援部署 MCP 伺服器。此功能仍在開發中，目前支援 SSE 伺服器。

操作步驟如下：

1. 複製一個代碼庫：

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. 在本機執行以測試：

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. 若要在本機嘗試，請於 *.vscode* 目錄下建立 *mcp.json* 檔案，並加入以下內容：

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


  啟動 SSE 伺服器後，可點擊 JSON 檔案中的播放圖示，現在你應該會看到 GitHub Copilot 偵測到伺服器上的工具，請參見工具圖示。

1. 部署時，執行以下指令：

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```


如上所述，你可以先本地部署，再透過此流程部署到 Azure。

## 附加資源

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps 文章](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP 代碼庫](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## 接下來的內容

- 下一課： [進階伺服器主題](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：  
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 提供翻譯。雖然我們力求準確，但請注意自動翻譯可能包含錯誤或不準確之處。以文件的原始語言版本為唯一權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們對因使用本翻譯而導致的任何誤解或曲解概不負責。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
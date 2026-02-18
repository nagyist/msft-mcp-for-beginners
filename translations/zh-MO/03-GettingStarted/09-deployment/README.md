# 部署 MCP 服务器

部署您的 MCP 服务器让其他人能够访问其工具和资源，超越本地环境。根据您对可扩展性、可靠性和管理便利性的需求，有多种部署策略可供选择。以下内容为您提供了在本地、容器中和云端部署 MCP 服务器的指导。

## 概述

本课题涵盖如何部署您的 MCP 服务器应用。

## 学习目标

完成本课后，您将能够：

- 评估不同的部署方法。
- 部署您的应用。

## 本地开发和部署

如果您的服务器旨在被用户机器上运行和使用，可以按照以下步骤操作：

1. **下载服务器**。如果您不是编写该服务器，则首先将其下载到您的机器。  
1. **启动服务器进程**：运行您的 MCP 服务器应用。

对于 SSE（stdio 类型服务器不需要）

1. **配置网络**：确保服务器在预期端口可访问。  
1. **连接客户端**：使用本地连接 URL，如 `http://localhost:3000`。

## 云端部署

MCP 服务器可以部署到多种云平台：

- **无服务器函数**：将轻量的 MCP 服务器作为无服务器函数部署。  
- **容器服务**：使用 Azure Container Apps、AWS ECS 或 Google Cloud Run 等服务。  
- **Kubernetes**：在 Kubernetes 集群中部署和管理 MCP 服务器，以实现高可用性。

### 示例：Azure Container Apps

Azure Container Apps 支持部署 MCP 服务器。该功能仍在开发中，目前支持 SSE 服务器。

操作步骤如下：

1. 克隆一个仓库：

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. 本地运行测试：

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. 若要在本地尝试，请在 *.vscode* 目录中创建一个 *mcp.json* 文件，添加以下内容：

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

  SSE 服务器启动后，您可以点击 JSON 文件中的播放图标，现在应该能看到 GitHub Copilot 检测到服务器上的工具，见工具图标。

1. 部署时，运行以下命令：

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

就这样，通过这些步骤您可以在本地或者通过 Azure 部署。

## 附加资源

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps 文章](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP 仓库](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## 接下来

- 下一课: [高级服务器主题](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件已使用人工智能翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。儘管我們致力於確保準確性，但請注意，自動翻譯可能包含錯誤或不準確之處。原始語言的文件應被視為權威來源。對於重要資訊，建議採用專業人工翻譯。我們對因使用本翻譯而引起的任何誤解或錯誤解釋概不負責。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
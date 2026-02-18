# 部署 MCP 服务器

部署您的 MCP 服务器使其他人可以访问其工具和资源，而不限于您的本地环境。根据您对可扩展性、可靠性和易管理性的需求，可以考虑多种部署策略。以下内容将为您提供在本地、容器及云端部署 MCP 服务器的指导。

## 概述

本课程涵盖如何部署您的 MCP Server 应用。

## 学习目标

完成本课程后，您将能够：

- 评估不同的部署方法。
- 部署您的应用。

## 本地开发和部署

如果您的服务器旨在通过用户机器运行进行访问，您可以按照以下步骤操作：

1. **下载服务器**。如果您不是编写该服务器，请先将其下载到您的机器。  
1. **启动服务器进程**：运行您的 MCP 服务器应用

对于 SSE（stdio 类型服务器不需要）

1. **配置网络**：确保服务器在预期端口可访问  
1. **连接客户端**：使用如 `http://localhost:3000` 的本地连接 URL

## 云端部署

MCP 服务器可以部署到各种云平台：

- **无服务器函数**：将轻量级 MCP 服务器部署为无服务器函数  
- **容器服务**：使用 Azure Container Apps、AWS ECS 或 Google Cloud Run 等服务  
- **Kubernetes**：在 Kubernetes 集群中部署和管理 MCP 服务器，实现高可用性

### 示例：Azure Container Apps

Azure Container Apps 支持部署 MCP 服务器。目前此功能仍在开发中，并且当前支持 SSE 服务器。

操作步骤如下：

1. 克隆一个仓库：

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. 本地运行以测试：

  ```sh
  uv venv
  uv sync

  # Linux/macOS
  export API_KEYS=<AN_API_KEY>
  # Windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. 若要在本地调试，请在 *.vscode* 目录下创建一个 *mcp.json* 文件并添加以下内容：

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

  启动 SSE 服务器后，您可以点击 JSON 文件中的播放图标，您应该会看到服务器上的工具被 GitHub Copilot 识别，参见工具图标。

1. 部署时，运行以下命令：

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

就是这么简单，通过这些步骤即可在本地部署，也可以部署到 Azure。

## 其他资源

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps 文章](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP 仓库](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## 后续内容

- 下一节：[高级服务器主题](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译而成。尽管我们努力保证准确性，但请注意，自动翻译可能包含错误或不准确之处。原始语言版本的文件应被视为权威来源。对于重要信息，建议使用专业人工翻译。因使用本翻译而产生的任何误解或误释，我们概不负责。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
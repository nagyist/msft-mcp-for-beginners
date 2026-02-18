# Deploying MCP Servers

Deploying your MCP server dey allow oda people to access im tools and resources beyond your local environment. E get different deployment strategies wey you fit consider, depend on how you want am for scalability, reliability, and how e go easy to manage. Below you go find guidance for deploying MCP servers locally, inside containers, and for the cloud.

## Overview

Dis lesson go show how to deploy your MCP Server app.

## Learning Objectives

By di end of dis lesson, you go fit:

- Check different deployment ways.
- Deploy your app.

## Local development and deployment

If your server na to run for user machine, you fit follow these steps:

1. **Download the server**. If you no write di server, abeg download am first for your machine.  
1. **Start the server process**: Run your MCP server application 

For SSE (no need am for stdio type server)

1. **Configure networking**: Make sure say the server dey accessible on di port wey dem expect  
1. **Connect clients**: Use local connection URLs like `http://localhost:3000`

## Cloud Deployment

MCP servers fit deploy for different cloud platforms:

- **Serverless Functions**: Deploy light MCP servers as serverless functions  
- **Container Services**: Use services like Azure Container Apps, AWS ECS, or Google Cloud Run  
- **Kubernetes**: Deploy and manage MCP servers for Kubernetes clusters to get better availability

### Example: Azure Container Apps

Azure Container Apps fit deploy MCP Servers. E still dey work progress and e currently dey support SSE servers.

Na so you fit take do am:

1. Clone the repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Run am locally to test di thing dem:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. To try am locally, create *mcp.json* file inside *.vscode* directory and put dis content inside:

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

  Once the SSE server don start, you fit click the play icon for the JSON file, you go fit see tools for the server dey picked up by GitHub Copilot, check the Tool icon.

1. To deploy am, run dis command:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Na so e be, deploy am locally, deploy am to Azure through these steps.

## Additional Resources

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps article](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## What's Next

- Next: [Advanced Server Topics](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even though we dey try make am accurate, abeg sabi say automated translations fit get errors or mistakes. The original document wey e dey for im own language na im be the correct one. If na serious matter, e better make person wey sabi human translation do am. We no go responsible if person no understand well or if person misunderstand from this translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
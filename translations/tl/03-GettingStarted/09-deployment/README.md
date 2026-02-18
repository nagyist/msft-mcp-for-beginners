# Pagdedepoy ng mga MCP Server

Ang pagdedepoy ng iyong MCP server ay nagpapahintulot sa iba na ma-access ang mga tool at resources nito lampas sa iyong lokal na kapaligiran. Mayroong ilang mga estratehiya sa pagdedepoy na dapat isaalang-alang, depende sa iyong mga pangangailangan para sa scalability, pagiging maasahan, at kadalian sa pamamahala. Sa ibaba makikita mo ang gabay para sa pagdedepoy ng mga MCP server nang lokal, sa mga container, at sa cloud.

## Pangkalahatang Ideya

Tinutukoy ng leksyon na ito kung paano i-deploy ang iyong MCP Server app.

## Mga Layunin ng Pagkatuto

Sa pagtatapos ng leksyon na ito, magagawa mong:

- Suriin ang iba't ibang pamamaraan ng pagdedepoy.
- I-deploy ang iyong app.

## Lokal na pagdevelop at pagdedepoy

Kung ang iyong server ay nilalayong gamitin sa pamamagitan ng pagpapatakbo sa makina ng mga user, maaari mong sundin ang mga sumusunod na hakbang:

1. **I-download ang server**. Kung hindi ikaw ang gumawa ng server, i-download muna ito sa iyong makina.  
1. **Simulan ang proseso ng server**: Patakbuhin ang iyong MCP server application

Para sa SSE (hindi kinakailangan para sa stdio type server)

1. **I-configure ang networking**: Tiyaking naa-access ang server sa inaasahang port  
1. **Ikonekta ang mga kliyente**: Gumamit ng mga lokal na connection URL tulad ng `http://localhost:3000`

## Cloud Deployment

Maaaring ideploy ang mga MCP server sa iba't ibang cloud platform:

- **Serverless Functions**: Ideploy ang magagaan na MCP server bilang serverless functions  
- **Container Services**: Gumamit ng mga serbisyo tulad ng Azure Container Apps, AWS ECS, o Google Cloud Run  
- **Kubernetes**: Ideploy at pamahalaan ang mga MCP server sa Kubernetes clusters para sa mataas na availability

### Halimbawa: Azure Container Apps

Sinusuportahan ng Azure Container Apps ang pagdedepoy ng MCP Servers. Ito ay isang work in progress at kasalukuyang sinusuportahan ang mga SSE server.

Narito kung paano mo ito magagawa:

1. I-clone ang isang repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Patakbuhin ito nang lokal upang subukan:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Para subukan ito nang lokal, gumawa ng isang *mcp.json* file sa isang *.vscode* na direktor at ilagay ang sumusunod na nilalaman:

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

  Kapag ang SSE server ay nasimulan na, maaari mong i-click ang play icon sa JSON file, makikita mo na ngayon ang mga tool sa server na kinikilala ng GitHub Copilot, tingnan ang Tool icon. 

1. Para mag-deploy, patakbuhin ang sumusunod na command:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Ayan na, i-deploy ito nang lokal, i-deploy ito sa Azure sa pamamagitan ng mga hakbang na ito.

## Karagdagang mga Mapagkukunan

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps article](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## Ano ang Susunod

- Susunod: [Mga Advanced na Paksa sa Server](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Paalala**:  
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI na pagsasalin na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagamat nagsusumikap kaming maging tumpak, pakatandaan na ang mga awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o kamalian. Ang orihinal na dokumento sa kanyang sariling wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang hindi pagkakaunawaan o maling interpretasyon na maaaring magmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
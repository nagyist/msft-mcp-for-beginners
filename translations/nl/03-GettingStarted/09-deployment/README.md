# Deployen van MCP-servers

Het deployen van je MCP-server maakt het mogelijk voor anderen om toegang te krijgen tot de tools en bronnen ervan buiten je lokale omgeving. Er zijn verschillende deploy-strategieën om te overwegen, afhankelijk van je eisen voor schaalbaarheid, betrouwbaarheid en beheerbaarheid. Hieronder vind je richtlijnen voor het deployen van MCP-servers lokaal, in containers en in de cloud.

## Overzicht

Deze les behandelt hoe je jouw MCP Server-app kunt deployen.

## Leerdoelen

Aan het einde van deze les kun je:

- Verschillende deploy-benaderingen evalueren.
- Je app deployen.

## Lokale ontwikkeling en deployment

Als je server bedoeld is om lokaal op de machine van gebruikers te draaien, kun je de volgende stappen volgen:

1. **Download de server**. Als je de server niet zelf hebt geschreven, download deze dan eerst naar je machine.
1. **Start het serverproces**: Voer je MCP-serverapplicatie uit.

Voor SSE (niet nodig voor stdio-type server)

1. **Configureer netwerken**: Zorg dat de server bereikbaar is op de verwachte poort.
1. **Verbind clients**: Gebruik lokale verbindings-URL's zoals `http://localhost:3000`.

## Cloud-Deployment

MCP-servers kunnen op diverse cloudplatformen gedeployed worden:

- **Serverless Functions**: Deploy lichte MCP-servers als serverless functions.
- **Container Services**: Gebruik services zoals Azure Container Apps, AWS ECS, of Google Cloud Run.
- **Kubernetes**: Deploy en beheer MCP-servers in Kubernetes-clusters voor hoge beschikbaarheid.

### Voorbeeld: Azure Container Apps

Azure Container Apps ondersteunen het deployen van MCP Servers. Het is nog in ontwikkeling en ondersteunt momenteel SSE-servers.

Zo pak je het aan:

1. Clone een repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Run het lokaal om dingen te testen:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Om het lokaal te proberen, maak een *mcp.json*-bestand aan in een *.vscode*-directory en voeg de volgende inhoud toe:

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

  Zodra de SSE-server gestart is, kun je op het play-icoon in het JSON-bestand klikken; je zou nu moeten zien dat tools op de server worden opgepikt door GitHub Copilot, zie het Tool-icoon.

1. Om te deployen, voer het volgende commando uit:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Daar heb je het, deploy het lokaal of deploy het naar Azure via deze stappen.

## Aanvullende bronnen

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps artikel](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## Wat is het volgende

- Volgende: [Geavanceerde Serveronderwerpen](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI-vertalingsdienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat automatische vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet als de gezaghebbende bron worden beschouwd. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
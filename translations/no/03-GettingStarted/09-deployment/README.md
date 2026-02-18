# Distribuere MCP-servere

Distribuering av MCP-serveren din gjør det mulig for andre å få tilgang til verktøyene og ressursene dens utover ditt lokale miljø. Det finnes flere distribueringsstrategier å vurdere, avhengig av dine krav til skalerbarhet, pålitelighet og enkel administrasjon. Nedenfor finner du veiledning for distribusjon av MCP-servere lokalt, i containere og til skyen.

## Oversikt

Denne leksjonen dekker hvordan du distribuerer MCP Server-appen din.

## Læringsmål

Når du er ferdig med denne leksjonen, vil du kunne:

- Vurdere ulike distribueringsmåter.
- Distribuere appen din.

## Lokal utvikling og distribusjon

Hvis serveren din skal kjøres på brukernes maskin, kan du følge disse stegene:

1. **Last ned serveren**. Hvis du ikke skrev serveren, last den ned til maskinen din først.  
1. **Start serverprosessen**: Kjør MCP-serverapplikasjonen din

For SSE (ikke nødvendig for stdio type server)

1. **Konfigurer nettverk**: Sørg for at serveren er tilgjengelig på forventet port  
1. **Koble klienter til**: Bruk lokale tilkoblings-URLer som `http://localhost:3000`

## Distribuering i skyen

MCP-servere kan distribueres til ulike skyplattformer:

- **Serverløse funksjoner**: Distribuer lette MCP-servere som serverløse funksjoner  
- **Container-tjenester**: Bruk tjenester som Azure Container Apps, AWS ECS eller Google Cloud Run  
- **Kubernetes**: Distribuer og administrer MCP-servere i Kubernetes-klynger for høy tilgjengelighet

### Eksempel: Azure Container Apps

Azure Container Apps støtter distribusjon av MCP-servere. Det er fortsatt under utvikling og støtter foreløpig SSE-servere.

Slik kan du gå frem:

1. Klon et repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Kjør det lokalt for å teste:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. For å prøve lokalt, lag en *mcp.json*-fil i en *.vscode*-mappe og legg til følgende innhold:

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

 Når SSE-serveren er startet, kan du klikke på spilleikonet i JSON-filen. Nå skal verktøy på serveren bli plukket opp av GitHub Copilot, se verktøyikonet.

1. For å distribuere, kjør følgende kommando:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Der har du det, distribuer lokalt, og distribuer til Azure gjennom disse stegene.

## Ekstra ressurser

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps artikkel](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Hva nå

- Neste: [Avanserte serveremner](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vennligst vær oppmerksom på at automatiserte oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal anses som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår som følge av bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
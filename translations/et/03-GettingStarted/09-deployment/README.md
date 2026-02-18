# MCP serverite juurutamine

MCP serveri juurutamine võimaldab teistel pääseda ligi selle tööriistadele ja ressurssidele väljaspool teie kohalikku keskkonda. Saadaval on mitu juurutusstrateegiat, sõltuvalt teie nõuetest skaleeritavuse, töökindluse ja haldamise lihtsuse osas. Allpool leiate juhiseid MCP serverite kohalikuks juurutamiseks, konteineritesse ja pilve.

## Ülevaade

See õppetund käsitleb, kuidas oma MCP Serveri rakendust juurutada.

## Õpieesmärgid

Selle õppetunni lõpuks oskad sa:

- Hinnata erinevaid juurutusmeetodeid.
- Juurutada oma rakendust.

## Kohalik arendus ja juurutus

Kui teie server on mõeldud töötama kasutaja masina peal, saate järgida järgmisi samme:

1. **Laadige server alla**. Kui te ei kirjutanud serverit, siis laadige see esmalt oma masinasse.  
1. **Käivitage serveriprotsess**: Käivitage oma MCP serveri rakendus.

SSE jaoks (ei ole vajalik stdio tüüpi serveri puhul)

1. **Konfigureerige võrgustik**: Veenduge, et server oleks juurdepääsetav oodatud pordil  
1. **Ühendage kliendid**: Kasutage kohalikke ühenduse URL-e nagu `http://localhost:3000`

## Pilve juurutamine

MCP servereid saab juurutada mitmetesse pilveplatvormidesse:

- **Serverita funktsioonid**: Juurutage kergekaalulisi MCP servereid serverita funktsioonidena  
- **Konteinerite teenused**: Kasutage teenuseid nagu Azure Container Apps, AWS ECS või Google Cloud Run  
- **Kubernetes**: Juurutage ja haldage MCP servereid Kubernetes klastri sees kõrge kättesaadavuse tagamiseks

### Näide: Azure Container Apps

Azure Container Apps toetavad MCP serverite juurutamist. See on veel pooleli olev projekt ja hetkel toetatakse SSE servereid.

Siin on, kuidas seda teha:

1. Kopeerige repositorium:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Käivitage see kohalikult, et asju testida:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Kohalikuks proovimiseks looge *.vscode* kataloogi *mcp.json* fail ja lisage järgmine sisu:

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

  Kui SSE server on käivitatud, saate JSON-failis klõpsata taasesituse ikoonil; nüüd peaks GitHub Copilot serveri tööriistad üles võtma, vaadake tööriista ikooni.

1. Juurutamiseks käivitage järgmine käsk:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Nii see käibki, juurutage see kohalikult, juurutage see Azure'i nende sammude kaudu.

## Lisamaterjalid

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps artikkel](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Mis järgmiseks

- Järgmine: [Täpsemad serveriteemad](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastutusest loobumine**:
See dokument on tõlgitud kasutades tehisintellektil põhinevat tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi püüame tagada täpsust, tuleb arvestada, et automatiseeritud tõlked võivad sisaldada vigu või ebatäpsusi. Algne dokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise info puhul soovitatakse professionaalset inimtõlget. Me ei vastuta tõlke kasutamisest tingitud arusaamatuste ega valesti mõistmiste eest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
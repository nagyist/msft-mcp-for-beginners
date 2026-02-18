# Postavljanje MCP servera

Postavljanje vašeg MCP servera omogućuje drugima pristup njegovim alatima i resursima izvan vašeg lokalnog okruženja. Postoji nekoliko strategija postavljanja koje treba razmotriti, ovisno o vašim zahtjevima za skalabilnošću, pouzdanošću i jednostavnošću upravljanja. Dolje ćete pronaći upute za postavljanje MCP servera lokalno, u kontejnerima i u oblaku.

## Pregled

Ova lekcija pokriva kako postaviti vašu MCP Server aplikaciju.

## Ciljevi učenja

Do kraja ove lekcije moći ćete:

- Procijeniti različite pristupe postavljanju.
- Postaviti svoju aplikaciju.

## Lokalni razvoj i postavljanje

Ako je vaš server namijenjen za korištenje na korisničkom računalu, možete slijediti sljedeće korake:

1. **Preuzmite server**. Ako niste napisali server, prvo ga preuzmite na svoje računalo.  
1. **Pokrenite server proces**: Pokrenite svoju MCP server aplikaciju.

Za SSE (nije potrebno za stdio tip servera)

1. **Konfigurirajte mrežu**: Osigurajte da je server dostupan na očekivanom portu  
1. **Povežite klijente**: Koristite lokalne veze poput `http://localhost:3000`

## Postavljanje u oblaku

MCP serveri mogu se postaviti na različite platforme u oblaku:

- **Serverless funkcije**: Postavite lagane MCP servere kao serverless funkcije  
- **Container Services**: Koristite usluge poput Azure Container Apps, AWS ECS ili Google Cloud Run  
- **Kubernetes**: Postavite i upravljajte MCP serverima u Kubernetes klasterima radi visoke dostupnosti

### Primjer: Azure Container Apps

Azure Container Apps podržava postavljanje MCP servera. Još je u razvoju, a trenutno podržava SSE servere.

Evo kako to možete učiniti:

1. Klonirajte repozitorij:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Pokrenite ga lokalno da testirate:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Da biste ga probali lokalno, stvorite *mcp.json* datoteku u *.vscode* direktoriju i dodajte sljedeći sadržaj:

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

  Nakon što je SSE server pokrenut, možete kliknuti ikonu za reproduciranje u JSON datoteci, sada biste trebali vidjeti alate na serveru koje prepoznaje GitHub Copilot, pogledajte ikonu Alata.

1. Za postavljanje, pokrenite sljedeću naredbu:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

To je to, postavite ga lokalno ili postavite u Azure kroz ove korake.

## Dodatni resursi

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Članak o Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repozitorij](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Što slijedi

- Sljedeće: [Napredne teme servera](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Odricanje od odgovornosti**:
Ovaj dokument preveden je korištenjem AI usluge prevođenja [Co-op Translator](https://github.com/Azure/co-op-translator). Iako nastojimo postići točnost, imajte na umu da automatski prijevodi mogu sadržavati pogreške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za kritične informacije preporučuje se profesionalni ljudski prijevod. Ne odgovaramo za bilo kakve nesporazume ili kriva tumačenja koja proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
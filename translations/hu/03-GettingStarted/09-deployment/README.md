# MCP szerverek telepítése

Az MCP szerver telepítése lehetővé teszi mások számára, hogy az eszközeit és erőforrásait a helyi környezeteden túli módon is elérhessék. Több telepítési stratégia létezik, amelyeket a skálázhatóság, a megbízhatóság és a kezelés egyszerűsége szerint érdemes megfontolni. Lentebb útmutatást találsz MCP szerverek helyi, konténeres és felhőbeli telepítéséhez.

## Áttekintés

Ez a lecke azt mutatja be, hogyan lehet telepíteni az MCP Server alkalmazást.

## Tanulási célok

A lecke végére képes leszel:

- Különböző telepítési megközelítéseket értékelni.
- Telepíteni az alkalmazásodat.

## Helyi fejlesztés és telepítés

Ha a szerver arra van szánva, hogy a felhasználók gépén fusson, kövesd az alábbi lépéseket:

1. **Töltsd le a szervert**. Ha nem te írtad a szervert, először töltsd le a gépedre.  
1. **Indítsd el a szerver folyamatot**: Futtasd az MCP szerver alkalmazásodat.

SSE esetén (nem szükséges stdio típusú szervernél)

1. **Konfiguráld a hálózatot**: Győződj meg róla, hogy a szerver az elvárt porton elérhető.  
1. **Kapcsolódó kliensek**: Használj helyi kapcsolódási URL-eket, például `http://localhost:3000`.

## Felhőbeli telepítés

Az MCP szerverek különböző felhőplatformokra telepíthetők:

- **Serverless Functions**: Könnyű súlyú MCP szervereket telepíthetsz szerver nélküli funkcióként.  
- **Konténer szolgáltatások**: Használj olyan szolgáltatásokat, mint az Azure Container Apps, AWS ECS vagy Google Cloud Run.  
- **Kubernetes**: Telepítsd és kezeld az MCP szervereket Kubernetes klaszterekben a nagy rendelkezésre állás érdekében.

### Példa: Azure Container Apps

Az Azure Container Apps támogatja az MCP szerverek telepítését. Még fejlesztés alatt áll, és jelenleg SSE szervereket támogat.

Így csinálhatod:

1. Klónozz egy repót:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Futtasd helyileg a teszteléshez:

  ```sh
  uv venv
  uv sync

  # Linux/macOS
  export API_KEYS=<AN_API_KEY>
  # Windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. A helyi próba érdekében hozz létre egy *mcp.json* fájlt a *.vscode* könyvtárban az alábbi tartalommal:

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

  Amint az SSE szerver elindult, kattints a lejátszás ikonra a JSON fájlban, ekkor a GitHub Copilot fel fogja ismerni a szerveren található eszközöket, nézd meg az Eszköz ikont.

1. A telepítéshez futtasd a következő parancsot:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Így van, helyileg telepíted, vagy ezeken a lépéseken keresztül telepíted Azure-ba.

## További források

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps cikk](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Mi következik

- Következő: [Speciális szerver témák](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Figyelmeztetés**:
Ezt a dokumentumot a [Co-op Translator](https://github.com/Azure/co-op-translator) AI fordító szolgáltatás segítségével fordítottuk. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum anyanyelvű változatát tekintse a hiteles forrásnak. Kritikus információk esetén profi emberi fordítást javaslunk. Nem vállalunk felelősséget a fordítás használatából eredő félreértésekért vagy félreértelmezésekért.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
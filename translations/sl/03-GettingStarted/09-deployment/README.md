# Nameščanje MCP strežnikov

Nameščanje vašega MCP strežnika omogoča drugim dostop do njegovih orodij in virov onkraj vašega lokalnega okolja. Obstaja več strategij nameščanja, odvisno od vaših zahtev za razširljivost, zanesljivost in enostavnost upravljanja. Spodaj boste našli navodila za nameščanje MCP strežnikov lokalno, v vsebnikih in v oblaku.

## Pregled

Ta lekcija zajema, kako namestiti vašo MCP strežniško aplikacijo.

## Cilji učenja

Do konca te lekcije boste znali:

- Oceniti različne pristope nameščanja.
- Namestiti svojo aplikacijo.

## Lokalni razvoj in nameščanje

Če je vaš strežnik namenjen uporabi na uporabniških računalnikih, lahko sledite naslednjim korakom:

1. **Prenesite strežnik**. Če strežnika niste napisali sami, ga najprej prenesite na svoj računalnik.
1. **Zaženite strežniški proces**: Zaženite vašo MCP strežniško aplikacijo

Za SSE (ni potrebno za stdio tip strežnika)

1. **Konfigurirajte omrežje**: Poskrbite, da je strežnik dostopen na pričakovani vratih
1. **Povežite odjemalce**: Uporabite lokalne povezovalne URL-je, kot je `http://localhost:3000`

## Nameščanje v oblaku

MCP strežnike lahko namestite na različne oblačne platforme:

- **Brezstrežniške funkcije (Serverless Functions)**: Namestite lahke MCP strežnike kot brezstrežniške funkcije
- **Storitve z vsebniki**: Uporabite storitve, kot so Azure Container Apps, AWS ECS ali Google Cloud Run
- **Kubernetes**: Namestite in upravljajte MCP strežnike v Kubernetes grozdih za visoko razpoložljivost

### Primer: Azure Container Apps

Azure Container Apps podpirajo nameščanje MCP strežnikov. Je še v razvoju in trenutno podpira SSE strežnike.

Tako lahko to izvedete:

1. Klonirajte repozitorij:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Zaženite ga lokalno za testiranje:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Za lokalno preizkušanje ustvarite datoteko *mcp.json* v imeniku *.vscode* in dodajte naslednjo vsebino:

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

  Ko se SSE strežnik zažene, lahko kliknete ikono za predvajanje v JSON datoteki, zdaj bi morali videti, da orodja na strežniku zazna GitHub Copilot, glejte ikono Orodja.

1. Za nameščanje zaženite naslednji ukaz:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Tako, imate ga, namestite lokalno ali na Azure preko teh korakov.

## Dodatni viri

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Članek o Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repozitorij](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Kaj sledi

- Naprej: [Napredne teme strežnikov](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Opozorilo**:
Ta dokument je bil preveden z uporabo storitve za prevajanje z umetno inteligenco [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, upoštevajte, da lahko avtomatizirani prevodi vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku velja za uradni vir. Za kritične informacije je priporočljivo uporabiti strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
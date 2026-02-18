# MCP serverių diegimas

Paleidus savo MCP serverį kiti gali pasiekti jo įrankius ir išteklius už lokalaus aplinkos ribų. Yra keletas diegimo strategijų, kurias reikėtų apsvarstyti, priklausomai nuo jūsų mastelio, patikimumo ir valdymo paprastumo reikalavimų. Žemiau rasite gaires, kaip diegti MCP serverius vietoje, konteineriuose ir debesyje.

## Apžvalga

Ši pamoka apima, kaip įdiegti jūsų MCP Server aplikaciją.

## Mokymosi tikslai

Pamokos pabaigoje galėsite:

- Įvertinti skirtingus diegimo būdus.
- Įdiegti savo aplikaciją.

## Vietinis kūrimas ir diegimas

Jei jūsų serveris skirtas būti naudojamas vartotojo kompiuteryje, galite sekti šiuos veiksmus:

1. **Atsisiųskite serverį**. Jei serverio nerašėte patys, pirmiausia atsisiųskite jį į savo kompiuterį.  
1. **Paleiskite serverio procesą**: paleiskite savo MCP serverio aplikaciją.

SSE atveju (nereikalinga stdio tipo serveriui):

1. **Suconfigūruokite tinklą**: užtikrinkite, kad serveris būtų pasiekiamas per numatytą prievadą.  
1. **Prijunkite klientus**: naudokite vietinius ryšio URL, pvz., `http://localhost:3000`.

## Debesų diegimas

MCP serveriai gali būti diegiami įvairiose debesų platformose:

- **Serverless Functions**: diegkite lengvus MCP serverius kaip serverless funkcijas.  
- **Konteinerių paslaugos**: naudokite paslaugas kaip Azure Container Apps, AWS ECS arba Google Cloud Run.  
- **Kubernetes**: diegkite ir valdykite MCP serverius Kubernetes klasteriuose aukšto prieinamumo užtikrinimui.

### Pavyzdys: Azure Container Apps

Azure Container Apps palaiko MCP serverių diegimą. Tai dar kūrimo stadijoje, šiuo metu palaikomi SSE serveriai.

Štai kaip tai padaryti:

1. Nuklonuokite repozitoriją:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Paleiskite lokaliai, kad išbandytumėte:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Norėdami išbandyti lokaliai, sukurkite *mcp.json* failą *.vscode* kataloge ir pridėkite šį turinį:

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

  Paleidus SSE serverį, galite paspausti grojimo piktogramą JSON faile, dabar turėtumėte matyti, kaip GitHub Copilot aptinka serverio įrankius, žr. Įrankio piktogramą.

1. Norėdami įdiegti, paleiskite šią komandą:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Štai taip galite diegti lokaliai arba į Azure šiais žingsniais.

## Papildomi ištekliai

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)  
- [Azure Container Apps straipsnis](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)  
- [Azure Container Apps MCP repozitorija](https://github.com/anthonychu/azure-container-apps-mcp-sample)  


## Kas toliau

- Toliau: [Išplėstiniai serverio temos](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors stengiamės užtikrinti tikslumą, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas gimtąja kalba turėtų būti laikomas autoritetingu šaltiniu. Svarbiai informacijai vertiname profesionalų žmogaus vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretavimą, kilusį dėl šio vertimo naudojimo.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
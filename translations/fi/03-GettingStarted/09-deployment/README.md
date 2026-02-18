# MCP-palvelimien käyttöönotto

MCP-palvelimesi käyttöönotto mahdollistaa sen työkalujen ja resurssien käytön paikallisen ympäristösi ulkopuolella. Käyttöönottoon on useita strategioita, riippuen skaalaustarpeistasi, luotettavuudesta ja hallinnan helppoudesta. Alta löydät ohjeita MCP-palvelimien käyttöönottoon paikallisesti, konteissa ja pilvessä.

## Yleiskatsaus

Tämä oppitunti kattaa, kuinka MCP Server -sovelluksesi otetaan käyttöön.

## Oppimistavoitteet

Tämän oppitunnin jälkeen osaat:

- Arvioida erilaisia käyttöönotto-menetelmiä.
- Ottaa sovelluksesi käyttöön.

## Paikallinen kehitys ja käyttöönotto

Jos palvelimesi on tarkoitettu käytettäväksi suoraan käyttäjän koneella, voit noudattaa seuraavia vaiheita:

1. **Lataa palvelin**. Jos et itse kirjoittanut palvelinta, lataa se ensin koneellesi.  
1. **Käynnistä palvelinprosessi**: Käynnistä MCP-palvelinsovelluksesi.

SSE:tä varten (ei tarvita stdio-tyyppiselle palvelimelle)

1. **Määritä verkkoasetukset**: Varmista, että palvelin on käytettävissä odotetulla portilla.  
1. **Yhdistä asiakkaat**: Käytä paikallisia yhteysosoitteita kuten `http://localhost:3000`.

## Pilvikäyttöönotto

MCP-palvelimia voi ottaa käyttöön erilaisilla pilvialustoilla:

- **Serverless-funktiot**: Ota käyttöön kevyitä MCP-palvelimia serverless-funktioina.  
- **Säiliöpalvelut**: Käytä palveluita kuten Azure Container Apps, AWS ECS tai Google Cloud Run.  
- **Kubernetes**: Ota MCP-palvelimet käyttöön ja hallinnoi niitä Kubernetes-klustereissa korkean saatavuuden takaamiseksi.

### Esimerkki: Azure Container Apps

Azure Container Apps tukee MCP-palvelimien käyttöönottoa. Se on vielä työn alla, ja tällä hetkellä tukee SSE-palvelimia.

Näin voit toimia:

1. Kloonaa repositorio:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Käynnistä paikallisesti testataksesi:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Paikallisen kokeilun tekemiseksi luo *mcp.json* -tiedosto *.vscode*-kansioon ja lisää seuraava sisältö:

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

  Kun SSE-palvelin on käynnistynyt, voit klikata JSON-tiedostossa olevaa toistopainiketta, nyt palvelimen työkalut pitäisi näkyä GitHub Copilotissa, näet työkalukuvakkeen.

1. Käyttöönottoa varten suorita seuraava komento:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Siinä se, ota käyttöön paikallisesti tai Azuren kautta näiden vaiheiden avulla.

## Lisämateriaalit

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps -artikkeli](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP -repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Seuraavaksi

- Seuraava: [Kehittyneet palvelinaiheet](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, pyydämme huomioimaan, että automaattikäännöksissä saattaa esiintyä virheitä tai epätarkkuuksia. Alkuperäistä asiakirjaa sen omalla kielellä tulisi pitää oikeana lähteenä. Tärkeissä tiedoissa suosittelemme ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä mahdollisesti aiheutuvista väärinkäsityksistä tai virhetulkinnasta.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
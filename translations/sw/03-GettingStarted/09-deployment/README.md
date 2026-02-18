# Kueneza Server za MCP

Kueneza server yako ya MCP kunawawezesha wengine kupata zana na rasilimali zake zaidi ya mazingira yako ya ndani. Kuna mbinu mbalimbali za kueneza zinazopaswa kuzingatiwa, kulingana na mahitaji yako ya kupanuka, kuaminika, na urahisi wa usimamizi. Hapa chini utapata mwongozo wa kueneza server za MCP ndani ya eneo lako la kazi, ndani ya kontena, na kwenye wingu.

## Muhtasari

Somo hili linashughulikia jinsi ya kueneza programu yako ya MCP Server.

## Malengo ya Kujifunza

Mwisho wa somo hili, utaweza:

- Kutathmini mbinu tofauti za kueneza.
- Kueneza programu yako.

## Maendeleo na uenezi wa ndani

Ikiwa server yako inakusudiwa kutumiwa kwa kuendeshwa kwenye kompyuta za watumiaji, unaweza kufuata hatua zifuatazo:

1. **Pakua server**. Ikiwa hujaunda server hiyo, basi pakua kwanza kwenye kompyuta yako.  
1. **Anzisha mchakato wa server**: Endesha programu yako ya MCP server

Kwa SSE (sio lazima kwa server aina ya stdio)

1. **Sanidi mtandao**: Hakikisha server inapatikana kwa bandari inayotarajiwa  
1. **Unganisha wateja**: Tumia URL za muunganisho wa ndani kama `http://localhost:3000`

## Uenezi wa Wingu

Server za MCP zinaweza kuenezwa kwenye majukwaa mbalimbali ya wingu:

- **Serverless Functions**: Kueneza server nyepesi za MCP kama serverless functions  
- **Huduma za Kontena**: Tumia huduma kama Azure Container Apps, AWS ECS, au Google Cloud Run  
- **Kubernetes**: Kueneza na kusimamia server za MCP kwenye vikundi vya Kubernetes kwa upatikanaji mkubwa

### Mfano: Azure Container Apps

Azure Container Apps zinaunga mkono uenezi wa Server za MCP. Bado ni kazi inayoendelea na kwa sasa zinaunga mkono server za SSE.

Hivi ndivyo unavyoweza kuifanya:

1. Nakili repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Iendeshe ndani ya eneo lako la kazi kufanyia majaribio:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Ili kujaribu ndani ya eneo lako, tengeneza faili *mcp.json* katika saraka ya *.vscode* na ongeza maudhui yafuatayo:

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

  Mara server ya SSE itakapoanzishwa, unaweza kubofya ikoni ya kucheza kwenye faili la JSON, sasa utaona zana za server zikichukuliwa na GitHub Copilot, ona ikoni ya Zana.

1. Kueneza, endesha amri ifuatayo:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Hivyo ndivyo ulivyo hivyo, kueneza ndani ya eneo lako, kueneza kwa Azure kupitia hatua hizi.

## Rasilimali Zaidi

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Makala ya Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Kile Kinachofuata

- Ifuatayo: [Mada Zaidi za Server](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kiarifu cha Kukataa**:
Nyaraka hii imetatuliwa kwa kutumia huduma ya kutafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kuwa tafsiri za kiotomatiki zinaweza kuwa na makosa au upotoshaji. Nyaraka ya asili katika lugha yake ya asili inapaswa kuchukuliwa kama chanzo cha kuaminika. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na watu inashauriwa. Hatubeba dhima kwa uelewa au tafsiri zisizo sahihi zinazotokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
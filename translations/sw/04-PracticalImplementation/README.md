# Utekelezaji wa Kivitendo

[![Jinsi ya Kujenga, Kupima, na Kutoa Programu za MCP kwa Vifaa na Mifumo Halisi](../../../translated_images/sw/05.64bea204e25ca891.webp)](https://youtu.be/vCN9-mKBDfQ)

_(Bofya picha hapo juu kutazama video ya somo hili)_

Utekelezaji wa kivitendo ndicho kinachoonekana nguvu ya Model Context Protocol (MCP). Wakati kuelewa nadharia na usanifu wa MCP ni muhimu, thamani halisi huibuka unapotekeleza dhana hizi kujenga, kupima, na kutoa suluhisho zinazotatua matatizo halisi. Sura hii inachanganya maarifa ya dhana na maendeleo ya mikono kwa mkono, ikikuongoza kupitia mchakato wa kuleta programu zinazotegemea MCP kuishi.

Kama unaunda wasaidizi wa akili, kuingiza AI katika taratibu za biashara, au kujenga vifaa maalum vya usindikaji data, MCP hutoa msingi unaobadilika. Muundo wake usiotegemea lugha na SDK rasmi kwa lugha za programu zinazojulikana hufanya uweze kufikiwa na waendelezaji mbalimbali. Kwa kutumia SDK hizi, unaweza haraka kufanya mbinu za awali, kurudia mchakato, na kupanua suluhisho zako katika majukwaa na mazingira tofauti.

Katika sehemu zilizofuata, utapata mifano ya kivitendo, msimbo wa kuiga, na mikakati ya utoaji inayothibitisha jinsi ya kutekeleza MCP katika C#, Java na Spring, TypeScript, JavaScript, na Python. Pia utajifunza jinsi ya kutatua hitilafu na kupima seva zako za MCP, kusimamia API, na kutoa suluhisho kwenye wingu kwa kutumia Azure. Vifaa hivi vya mikono kwa mkono vimeundwa kuharakisha ujifunzaji wako na kusaidia kujenga programu thabiti za MCP zinazotegemewa kwa uzalishaji kwa ujasiri.

## Muhtasari

Somo hili linazingatia vipengele vya kivitendo vya utekelezaji wa MCP katika lugha nyingi za programu. Tutachunguza jinsi ya kutumia SDK za MCP katika C#, Java na Spring, TypeScript, JavaScript, na Python kujenga programu thabiti, kutatua hitilafu na kupima seva za MCP, na kuunda rasilimali zinazoweza kutumika tena, maelekezo, na zana.

## Malengo ya Kujifunza

Mwisho wa somo hili, utaweza:

- Kutekeleza suluhisho za MCP kwa kutumia SDK rasmi katika lugha mbalimbali za programu
- Kutatua hitilafu na kupima seva za MCP kwa mfumo
- Kuunda na kutumia vipengele vya seva (Rasilimali, Maelekezo, na Zana)
- Kubuni taratibu bora za MCP kwa kazi ngumu
- Kuboresha utekelezaji wa MCP kwa utendaji na uaminifu

## Rasilimali Rasmi za SDK

Model Context Protocol inatoa SDK rasmi kwa lugha nyingi (kuzingatia [MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)):

- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk)
- [Java na Spring SDK](https://github.com/modelcontextprotocol/java-sdk) **Kumbuka:** inahitaji tegemezi ya [Project Reactor](https://projectreactor.io). (Angalia [jadili suala 246](https://github.com/orgs/modelcontextprotocol/discussions/246).)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk)
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk)

## Kazi na SDK za MCP

Sehemu hii inatoa mifano ya kivitendo ya utekelezaji wa MCP katika lugha nyingi za programu. Unaweza kupata msimbo wa kuiga katika saraka ya `samples` iliyopangwa kwa lugha.

### Sampuli Zinazopatikana

Hifadhidata ina [sampuli za utekelezaji](../../../04-PracticalImplementation/samples) katika lugha zifuatazo:

- [C#](./samples/csharp/README.md)
- [Java na Spring](./samples/java/containerapp/README.md)
- [TypeScript](./samples/typescript/README.md)
- [JavaScript](./samples/javascript/README.md)
- [Python](./samples/python/README.md)

Kila sampuli inaonyesha dhana kuu za MCP na mifumo ya utekelezaji kwa lugha na mazingira hayo hasa.

### Mwongozo wa Kivitendo

Mwongozo zaidi kwa utekelezaji wa MCP wa kivitendo:

- [Kuhusu Ukurasa na Mseto Mkubwa wa Matokeo](./pagination/README.md) - Tumia kipengele cha ukurasa kwa zana, rasilimali, na maktaba kubwa za data

## Vipengele Vikuu vya Seva

Seva za MCP zinaweza kutekeleza mchanganyiko wowote wa vipengele hivi:

### Rasilimali

Rasilimali hutoa muktadha na data kwa mtumiaji au mfano wa AI kutumia:

- Makusanyo ya nyaraka
- Maktaba za maarifa
- Vyanzo vya data iliyopangwa
- Mfumo wa faili

### Maelekezo

Maelekezo ni ujumbe na taratibu zilizobuniwa kwa mtumiaji:

- Mitindo iliyoandaliwa ya mazungumzo
- Mifumo ya mwingiliano wa mwelekeo
- Miundo maalum ya mazungumzo

### Zana

Zana ni kazi ambazo mfano wa AI hufanya:

- Zana za usindikaji data
- Muunganisho wa API za nje
- Uwezo wa kihesabu
- Kifaa cha utaftaji

## Sampuli za Utekelezaji: Utekelezaji wa C#

Hazina rasmi ya SDK ya C# ina sampuli kadhaa za utekelezaji unaoonyesha vipengele tofauti vya MCP:

- **Mteja wa MCP wa Msingi**: Mfano rahisi wa jinsi ya kuunda mteja wa MCP na kuita zana
- **Seva ya MCP ya Msingi**: Utekelezaji wa seva wa chini sana na usajili wa zana rahisi
- **Seva ya MCP ya Juu**: Seva kamili yenye usajili wa zana, uthibitishaji, na usimamizi wa makosa
- **Uunganisho wa ASP.NET**: Mifano inayoonyesha muunganisho na ASP.NET Core
- **Mifumo ya Utekelezaji wa Zana**: Mienendo tofauti ya kutekeleza zana za ngazi tofauti za ugumu

SDK ya C# ya MCP iko katika awamu ya maonyesho na API zinaweza kubadilika. Tutaendelea kusasisha blogi hii kadri SDK inavyotokea.

### Vipengele Muhimu

- [C# MCP Nuget ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol)
- Kujenga [seva yako ya kwanza ya MCP](https://devblogs.microsoft.com/dotnet/build-a-model-context-protocol-mcp-server-in-csharp/).

Kwa sampuli kamili za utekelezaji wa C#, tembelea [hifadhi rasmi ya sampuli za SDK ya C#](https://github.com/modelcontextprotocol/csharp-sdk)

## Sampuli ya utekelezaji: Utekelezaji wa Java na Spring

SDK ya Java na Spring hutoa chaguzi za utekelezaji thabiti za MCP zenye vipengele vya kiwango cha biashara.

### Vipengele Muhimu

- Muunganisho wa Spring Framework
- Usalama wa aina wa nguvu
- Msaada wa programu ya reactive
- Ushughulikiaji kamili wa makosa

Kwa sampuli kamili ya utekelezaji wa Java na Spring, angalia [sampuli ya Java na Spring](samples/java/containerapp/README.md) katika saraka ya sampuli.

## Sampuli ya utekelezaji: Utekelezaji wa JavaScript

SDK ya JavaScript hutoa njia nyepesi na inayobadilika kwa utekelezaji wa MCP.

### Vipengele Muhimu

- Msaada wa Node.js na kivinjari
- API inayotegemea ahadi (Promise-based)
- Muunganisho rahisi na Express na mifumo mingine
- Msaada wa WebSocket kwa mtiririko

Kwa sampuli kamili ya utekelezaji wa JavaScript, angalia [sampuli ya JavaScript](samples/javascript/README.md) katika saraka ya sampuli.

## Sampuli ya utekelezaji: Utekelezaji wa Python

SDK ya Python hutoa njia ya Pythonic kwa utekelezaji wa MCP pamoja na muunganisho bora wa mifumo ya ML.

### Vipengele Muhimu

- Msaada wa async/await na asyncio
- Muunganisho wa FastAPI``
- Usajili rahisi wa zana
- Muunganisho wa asili na maktaba maarufu za ML

Kwa sampuli kamili ya utekelezaji wa Python, angalia [sampuli ya Python](samples/python/README.md) katika saraka ya sampuli.

## Usimamizi wa API

Azure API Management ni jibu zuri jinsi tunavyoweza kuimarisha Seva za MCP. Wazo ni kuweka mfano wa Azure API Management mbele ya Seva yako ya MCP na kuruhusu isimamishe vipengele utakavyotaka kama:

- ukomo wa viwango
- usimamizi wa tokeni
- ufuatiliaji
- upangaji mzigo
- usalama

### Sampuli ya Azure

Hii ni Sampuli ya Azure inayofanya hayo kabisa, yaani [kuunda Seva ya MCP na kuilinda kwa Azure API Management](https://github.com/Azure-Samples/remote-mcp-apim-functions-python).

Tazama jinsi mzunguko wa idhini unavyotokea kwenye picha ifuatayo:

![APIM-MCP](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/mcp-client-authorization.gif?raw=true)

Katika picha iliyotangulia, yafanyika yafuatayo:

- Uthibitishaji/Idhini hutokea kwa kutumia Microsoft Entra.
- Azure API Management inafanya kazi kama lango na kutumia sera kuongoza na kusimamia trafiki.
- Azure Monitor inaandika maombi yote kwa uchambuzi zaidi.

#### Mzunguko wa Idhini

Tuchukulie mzunguko wa idhini kwa undani zaidi:

![Sequence Diagram](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/infra/app/apim-oauth/diagrams/images/mcp-client-auth.png?raw=true)

#### Maelezo ya Idhini ya MCP

Jifunze zaidi kuhusu [maelezo ya idhini ya MCP](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/authorization/)

## Toa Seva ya Remote MCP kwa Azure

Tuchunguze kama tunaweza kutoa sampuli tuliyotaja awali:

1. Nakili repo

    ```bash
    git clone https://github.com/Azure-Samples/remote-mcp-apim-functions-python.git
    cd remote-mcp-apim-functions-python
    ```

1. Sajili mtoa huduma wa rasilimali `Microsoft.App`.

   - Ikiwa unatumia Azure CLI, endesha `az provider register --namespace Microsoft.App --wait`.
   - Ikiwa unatumia Azure PowerShell, endesha `Register-AzResourceProvider -ProviderNamespace Microsoft.App`. Kisha endesha `(Get-AzResourceProvider -ProviderNamespace Microsoft.App).RegistrationState` baada ya muda mdogo ili kuangalia kama usajili umekamilika.

1. Endesha amri hii ya [azd](https://aka.ms/azd) kutoa huduma ya usimamizi wa api, app ya kazi (pamoja na msimbo) na rasilimali zote muhimu za Azure

    ```shell
    azd up
    ```

    Amri hizi zinapaswa kutoa rasilimali zote za wingu kwenye Azure

### Kupima seva yako na MCP Inspector

1. Katika **dirisha jipya la terminal**, sakinisha na endesha MCP Inspector

    ```shell
    npx @modelcontextprotocol/inspector
    ```

    Unapaswa kuona kiolesura kama:

    ![Connect to Node inspector](../../../translated_images/sw/connect.141db0b2bd05f096.webp)

1. Bofya CTRL ili pakua app ya wavuti ya MCP Inspector kutoka kwa URL inayoonyeshwa na app (mfano [http://127.0.0.1:6274/#resources](http://127.0.0.1:6274/#resources))
1. Weka aina ya usafirishaji kuwa `SSE`
1. Weka URL ya mwisho wa API Management SSE unaoendesha baada ya `azd up` na **Unganisha**:

    ```shell
    https://<apim-servicename-from-azd-output>.azure-api.net/mcp/sse
    ```

1. **Orodhesha Zana**. Bofya zana na **Endesha Zana**.  

Kama hatua zote zimefanikiwa, sasa unategemeana na seva ya MCP na umeweza kuita zana.

## Seva za MCP kwa Azure

[Remote-mcp-functions](https://github.com/Azure-Samples/remote-mcp-functions-dotnet): Mfululizo huu wa hifadhidata ni mfumo wa kuanza haraka kwa kujenga na kutoa seva za mbali za MCP (Model Context Protocol) kwa kutumia Azure Functions na Python, C# .NET au Node/TypeScript.

Sampuli hutoa suluhisho kamili inayoruhusu waendelezaji:

- Kujenga na kuendesha kwa ndani: Kuendeleza na kutatua hitilafu seva ya MCP kwenye mashine ya ndani
- Kutoa kwenye Azure: Kutambulisha kwa urahisi kwenye wingu kwa amri moja ya azd up
- Kuungana kutoka kwa wateja: Kuunganisha seva ya MCP kutoka kwa wateja tofauti ikiwa ni pamoja na hali ya wakala wa Copilot ya VS Code na kifaa cha MCP Inspector

### Vipengele Muhimu

- Usalama ulio ndani ya muundo: Seva ya MCP inalindwa kwa kutumia funguo na HTTPS
- Chaguzi za uthibitishaji: Inasaidia OAuth kwa kutumia uthibitishaji uliojengwa na/au API Management
- Kutengwa kwa mtandao: Inaruhusu kutengwa kwa mtandao kwa kutumia Azure Virtual Networks (VNET)
- Usanifu wa bila seva: Inatumia Azure Functions kwa utekelezaji unaoweza kupanuka unaotegemea matukio
- Maendeleo ya ndani: Msaada kamili wa maendeleo na kutatua hitilafu kwa ndani
- Utoaji rahisi: Mchakato rahisi wa kutoa sehemu kwenye Azure

Hifadhidata ina faili zote za usanidi, msimbo wa chanzo, na maelezo ya miundombinu ya kuanza haraka na utekelezaji wa seva ya MCP ya uzalishaji.

- [Azure Remote MCP Functions Python](https://github.com/Azure-Samples/remote-mcp-functions-python) - Sampuli ya utekelezaji wa MCP kwa kutumia Azure Functions na Python

- [Azure Remote MCP Functions .NET](https://github.com/Azure-Samples/remote-mcp-functions-dotnet) - Sampuli ya utekelezaji ya MCP kwa kutumia Azure Functions na C# .NET

- [Azure Remote MCP Functions Node/Typescript](https://github.com/Azure-Samples/remote-mcp-functions-typescript) - Sampuli ya utekelezaji wa MCP kwa kutumia Azure Functions na Node/TypeScript.

## Muhimu wa Kumbukumbu

- SDK za MCP hutoa zana maalum za lugha kwa kutekeleza suluhisho thabiti za MCP
- Mchakato wa kutatua hitilafu na kupima ni muhimu kwa programu za MCP zinazotegemewa
- Mitindo ya maelekezo inayoweza kutumika tena huruhusu mwingiliano thabiti wa AI
- Taratibu bora zinaweza kuratibu kazi ngumu kwa kutumia zana nyingi
- Kutekeleza suluhisho za MCP kunahitaji kuzingatia usalama, utendaji, na usimamizi wa makosa

## Zoho la Mazoezi

Buni mchakato wa kivitendo wa MCP unaoshughulikia tatizo halisi katika eneo lako:

1. Tambua zana 3-4 ambazo zitakuwa muhimu kutatua tatizo hili
2. Tengeneza mchoro wa mchakato unaoonyesha jinsi zana hizi zinavyoshirikiana
3. Tekeleza toleo la msingi la moja ya zana kwa lugha unayotaka
4. Tengeneza kiolezo cha maelekezo kitakachosaidia mfano kutumia zana yako kwa ufanisi

## Rasilimali Zaidi

---

## Ifuatayo

Ifuatayo: [Mada za Juu](../05-AdvancedTopics/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tangazo la Kukataa**:
Nyaraka hii imefasiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au kasoro. Nyaraka asili katika lugha yake ya asili inapaswa kuzingatiwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu ya binadamu inashauriwa. Hatubebwi na dhima yoyote kwa kutoelewana au tafsiri potofu zinazotokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
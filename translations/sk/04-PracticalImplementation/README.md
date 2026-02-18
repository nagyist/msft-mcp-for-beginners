# Praktická implementácia

[![Ako vytvoriť, otestovať a nasadiť MCP aplikácie s reálnymi nástrojmi a pracovnými postupmi](../../../translated_images/sk/05.64bea204e25ca891.webp)](https://youtu.be/vCN9-mKBDfQ)

_(Kliknite na obrázok vyššie pre zobrazenie videa tejto lekcie)_

Praktická implementácia je miestom, kde sa sila Model Context Protocol (MCP) stáva hmatateľnou. Aj keď je dôležité porozumieť teórii a architektúre MCP, skutočná hodnota sa objavuje, keď tieto koncepty použijete na vytváranie, testovanie a nasadzovanie riešení, ktoré riešia reálne problémy. Táto kapitola prekladá medzeru medzi koncepčným poznaním a praktickým vývojom a vedie vás procesom vytvárania aplikácií založených na MCP.

Či už vyvíjate inteligentných asistentov, integrujete AI do podnikových pracovných postupov alebo vytvárate vlastné nástroje na spracovanie dát, MCP poskytuje flexibilný základ. Jeho jazykovo-neutrálny dizajn a oficiálne SDK pre populárne programovacie jazyky ho robia dostupným pre širokú škálu vývojárov. Vďaka týmto SDK môžete rýchlo prototypovať, iterovať a škálovať riešenia naprieč rôznymi platformami a prostrediami.

V nasledujúcich častiach nájdete praktické príklady, ukážkový kód a stratégie nasadenia, ktoré ukazujú, ako implementovať MCP v C#, Jave so Springom, TypeScripte, JavaScripte a Pythone. Tiež sa naučíte, ako ladiť a testovať vaše MCP servery, spravovať API a nasadiť riešenia do cloudu pomocou Azure. Tieto praktické zdroje sú navrhnuté tak, aby urýchlili vaše učenie a pomohli vám dôverne vytvárať robustné MCP aplikácie pripravené do produkcie.

## Prehľad

Táto lekcia sa zameriava na praktické aspekty implementácie MCP v rôznych programovacích jazykoch. Preskúmame, ako používať MCP SDK v C#, Jave so Springom, TypeScripte, JavaScripte a Pythone na vytváranie robustných aplikácií, ladenie a testovanie MCP serverov a tvorbu znovupoužiteľných zdrojov, promptov a nástrojov.

## Výučbové ciele

Na konci tejto lekcie budete schopní:

- Implementovať MCP riešenia pomocou oficiálnych SDK v rôznych programovacích jazykoch
- Systematicky ladiť a testovať MCP servery
- Vytvárať a používať vlastnosti servera (Zdroje, Prompty a Nástroje)
- Navrhnúť efektívne MCP pracovné postupy pre komplexné úlohy
- Optimalizovať implementácie MCP pre výkon a spoľahlivosť

## Oficiálne SDK zdroje

Model Context Protocol ponúka oficiálne SDK pre viaceré jazyky (v súlade s [MCP Špecifikáciou 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)):

- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk)
- [Java so Spring SDK](https://github.com/modelcontextprotocol/java-sdk) **Poznámka:** vyžaduje závislosť na [Project Reactor](https://projectreactor.io). (Pozri [diskusný issue 246](https://github.com/orgs/modelcontextprotocol/discussions/246).)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk)
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk)

## Práca s MCP SDK

Táto sekcia poskytuje praktické príklady implementácie MCP v rôznych programovacích jazykoch. Ukážkový kód nájdete v adresári `samples` zoradený podľa jazyka.

### Dostupné ukážky

Repozitár obsahuje [ukážkové implementácie](../../../04-PracticalImplementation/samples) v nasledujúcich jazykoch:

- [C#](./samples/csharp/README.md)
- [Java so Spring](./samples/java/containerapp/README.md)
- [TypeScript](./samples/typescript/README.md)
- [JavaScript](./samples/javascript/README.md)
- [Python](./samples/python/README.md)

Každá ukážka demonštruje kľúčové koncepty MCP a implementačné vzory pre daný jazyk a ekosystém.

### Praktické príručky

Ďalšie príručky pre praktickú implementáciu MCP:

- [Paginácia a veľké výsledkové množiny](./pagination/README.md) - Rieši stránkovanie založené na kurzore pre nástroje, zdroje a veľké dátové množiny

## Hlavné vlastnosti servera

MCP servery môžu implementovať ľubovoľnú kombináciu týchto vlastností:

### Zdroje

Zdroje poskytujú kontext a dáta pre používateľa alebo AI model na použitie:

- Úložiská dokumentov
- Znalostné bázy
- Štruktúrované dátové zdroje
- Súborové systémy

### Prompty

Prompty sú šablónované správy a pracovné postupy pre používateľov:

- Preddefinované šablóny rozhovorov
- Riadené interakčné vzory
- Špecializované dialógové štruktúry

### Nástroje

Nástroje sú funkcie, ktoré AI model vykonáva:

- Utility na spracovanie dát
- Integrácie externých API
- Výpočtové schopnosti
- Vyhľadávacie funkcie

## Ukážkové implementácie: Implementácia v C#

Oficiálne repozitár C# SDK obsahuje niekoľko ukážkových implementácií demonštrujúcich rôzne aspekty MCP:

- **Základný MCP klient**: Jednoduchý príklad, ako vytvoriť MCP klienta a volať nástroje
- **Základný MCP server**: Minimálna implementácia servera so základnou registráciou nástrojov
- **Pokročilý MCP server**: Plnofunkčný server s registráciou nástrojov, autentifikáciou a spracovaním chýb
- **Integrácia ASP.NET**: Príklady ukazujúce integráciu s ASP.NET Core
- **Vzory implementácie nástrojov**: Rôzne vzory implementácie nástrojov s rôznou úrovňou zložitosti

MCP C# SDK je vo fáze preview a API sa môžu meniť. Tento blog budeme priebežne aktualizovať podľa vývoja SDK.

### Kľúčové vlastnosti

- [C# MCP Nuget ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol)
- Vytváranie [prvého MCP servera](https://devblogs.microsoft.com/dotnet/build-a-model-context-protocol-mcp-server-in-csharp/).

Pre kompletné ukážky implementácií v C# navštívte [oficiálny repozitár C# SDK ukážok](https://github.com/modelcontextprotocol/csharp-sdk)

## Ukážková implementácia: Implementácia v Java so Springom

Java so Spring SDK ponúka robustné možnosti implementácie MCP s funkciami na podnikovej úrovni.

### Kľúčové vlastnosti

- Integrácia so Spring Framework
- Silná typová bezpečnosť
- Podpora reaktívneho programovania
- Komplexné spracovanie chýb

Pre kompletnú ukážku implementácie v Java so Springom pozrite [Java so Spring ukážku](samples/java/containerapp/README.md) v adresári samples.

## Ukážková implementácia: Implementácia v JavaScript

JavaScript SDK poskytuje ľahký a flexibilný prístup k implementácii MCP.

### Kľúčové vlastnosti

- Podpora Node.js a prehliadačov
- API založené na PROMISE
- Jednoduchá integrácia s Express a inými frameworkami
- Podpora WebSocket pre streamovanie

Pre kompletnú ukážku implementácie v JavaScript pozrite [JavaScript ukážku](samples/javascript/README.md) v adresári samples.

## Ukážková implementácia: Implementácia v Pythone

Python SDK ponúka pythonický prístup k implementácii MCP s vynikajúcou integráciou ML frameworkov.

### Kľúčové vlastnosti

- Podpora async/await s asyncio
- Integrácia s FastAPI
- Jednoduchá registrácia nástrojov
- Nativna integrácia s populárnymi ML knižnicami

Pre kompletnú ukážku implementácie v Pythone pozrite [Python ukážku](samples/python/README.md) v adresári samples.

## Správa API

Azure API Management je skvelým riešením, ako zabezpečiť MCP servery. Myšlienka je umiestniť inštanciu Azure API Management pred váš MCP server a nechať ju riešiť funkcie, ktoré pravdepodobne chcete, ako napríklad:

- obmedzovanie rýchlosti
- správa tokenov
- monitorovanie
- vyvažovanie záťaže
- bezpečnosť

### Azure ukážka

Tu je Azure ukážka, ktorá robí presne to, t.j. [vytváranie MCP servera a zabezpečenie pomocou Azure API Management](https://github.com/Azure-Samples/remote-mcp-apim-functions-python).

Pozrite sa, ako prebieha autorizácia na obrázku nižšie:

![APIM-MCP](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/mcp-client-authorization.gif?raw=true)

Na predchádzajúcom obrázku prebieha:

- Autentifikácia/Autorizácia pomocou Microsoft Entra.
- Azure API Management funguje ako brána a používa pravidlá na smerovanie a správu prenosu.
- Azure Monitor zaznamenáva všetky požiadavky na ďalšiu analýzu.

#### Autorizácia - tok

Pozrime sa na tok autorizácie podrobnejšie:

![Sequence Diagram](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/infra/app/apim-oauth/diagrams/images/mcp-client-auth.png?raw=true)

#### Špecifikácia autorizácie MCP

Viac informácií o [špecifikácii autorizácie MCP](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/authorization/)

## Nasadenie vzdialeného MCP servera do Azure

Pozrime sa, či môžeme nasadiť ukážku, ktorú sme spomínali vyššie:

1. Naklonujte repozitár

    ```bash
    git clone https://github.com/Azure-Samples/remote-mcp-apim-functions-python.git
    cd remote-mcp-apim-functions-python
    ```

1. Zaregistrujte poskytovateľa zdrojov `Microsoft.App`.

   - Ak používate Azure CLI, spustite `az provider register --namespace Microsoft.App --wait`.
   - Ak používate Azure PowerShell, spustite `Register-AzResourceProvider -ProviderNamespace Microsoft.App`. Potom po chvíli spustite `(Get-AzResourceProvider -ProviderNamespace Microsoft.App).RegistrationState`, aby ste skontrolovali, či je registrácia kompletná.

1. Spustite tento príkaz [azd](https://aka.ms/azd) na vytvorenie služby správy API, funkčnej aplikácie (s kódom) a všetkých ďalších požadovaných Azure zdrojov

    ```shell
    azd up
    ```

    Tento príkaz by mal nasadiť všetky cloudové zdroje v Azure

### Testovanie vášho servera s MCP Inspector

1. V **novom terminálovom okne** nainštalujte a spustite MCP Inspector

    ```shell
    npx @modelcontextprotocol/inspector
    ```

    Mali by ste vidieť rozhranie podobné:

    ![Pripojenie k Node inspektoru](../../../translated_images/sk/connect.141db0b2bd05f096.webp)

1. Ctrl kliknite pre načítanie webovej aplikácie MCP Inspector z URL zobrazeného aplikáciou (napr. [http://127.0.0.1:6274/#resources](http://127.0.0.1:6274/#resources))
1. Nastavte typ transportu na `SSE`
1. Nastavte URL na bežiaci endpoint API Management SSE zobrazený po príkaze `azd up` a **Pripojte sa**:

    ```shell
    https://<apim-servicename-from-azd-output>.azure-api.net/mcp/sse
    ```

1. **Zoznam nástrojov**. Kliknite na nástroj a **Spustite nástroj**.

Ak všetky kroky prebehli úspešne, teraz by ste mali byť pripojení k MCP serveru a mohli ste zavolať nástroj.

## MCP servery pre Azure

[Remote-mcp-functions](https://github.com/Azure-Samples/remote-mcp-functions-dotnet): Táto sada repozitárov je šablóna na rýchly štart pre vytváranie a nasadzovanie vlastných vzdialených MCP (Model Context Protocol) serverov pomocou Azure Functions v Pythone, C# .NET alebo Node/TypeScript.

Ukážky poskytujú kompletné riešenie, ktoré umožňuje vývojárom:

- Vyvíjať a spúšťať lokálne: Vývoj a ladenie MCP servera na lokálnom počítači
- Nasadiť do Azure: Jednoduché nasadenie do cloudu pomocou príkazu azd up
- Pripojiť sa z klientov: Pripojenie k MCP serveru z rôznych klientov vrátane režimu agenta Copilot vo VS Code a nástroja MCP Inspector

### Kľúčové vlastnosti

- Bezpečnosť od základu: MCP server je zabezpečený pomocou kľúčov a HTTPS
- Možnosti autentifikácie: Podpora OAuth s vstavaným auth a/alebo API Management
- Izolácia siete: Umožňuje izoláciu siete pomocou Azure Virtual Networks (VNET)
- Serverless architektúra: Využíva Azure Functions pre škálovateľné, na udalostiach založené vykonávanie
- Lokálny vývoj: Komplexná podpora pre lokálny vývoj a ladenie
- Jednoduché nasadenie: Zjednodušený proces nasadenia do Azure

Repozitár obsahuje všetky potrebné konfiguračné súbory, zdrojový kód a definície infraštruktúry pre rýchly štart s implementáciou MCP servera pripraveného do produkcie.

- [Azure Remote MCP Functions Python](https://github.com/Azure-Samples/remote-mcp-functions-python) - Ukážková implementácia MCP pomocou Azure Functions s Pythonom

- [Azure Remote MCP Functions .NET](https://github.com/Azure-Samples/remote-mcp-functions-dotnet) - Ukážková implementácia MCP pomocou Azure Functions s C# .NET

- [Azure Remote MCP Functions Node/Typescript](https://github.com/Azure-Samples/remote-mcp-functions-typescript) - Ukážková implementácia MCP pomocou Azure Functions s Node/TypeScript.

## Kľúčové poznatky

- MCP SDK poskytujú nástroje špecifické pre jazyky na implementáciu robustných MCP riešení
- Proces ladenia a testovania je kľúčový pre spoľahlivé MCP aplikácie
- Znovupoužiteľné šablóny promptov umožňujú konzistentné AI interakcie
- Dobre navrhnuté pracovné postupy môžu orchestruje zložité úlohy pomocou viacerých nástrojov
- Implementácia MCP riešení vyžaduje zohľadnenie bezpečnosti, výkonu a spracovania chýb

## Cvičenie

Navrhnite praktický MCP pracovný postup, ktorý rieši reálny problém vo vašej doméne:

1. Identifikujte 3-4 nástroje, ktoré by boli užitočné na riešenie tohto problému
2. Vytvorte diagram pracovného postupu zobrazujúci, ako tieto nástroje interagujú
3. Implementujte základnú verziu jedného z nástrojov vo vašom preferovanom jazyku
4. Vytvorte šablónu promptu, ktorá by pomohla modelu efektívne použiť váš nástroj

## Dodatočné zdroje

---

## Čo nasleduje

Nasleduje: [Pokročilé témy](../05-AdvancedTopics/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zrieknutie sa zodpovednosti**:  
Tento dokument bol preložený pomocou AI prekladovej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Aj keď sa snažíme o presnosť, prosím, berte na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Originálny dokument v jeho pôvodnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre dôležité informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za akékoľvek nedorozumenia alebo nesprávne výklady vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
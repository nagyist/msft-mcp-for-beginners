# Nasazení MCP serverů

Nasazení vašeho MCP serveru umožňuje ostatním přístup k jeho nástrojům a zdrojům mimo vaše lokální prostředí. Existuje několik strategií nasazení, které je třeba zvážit v závislosti na vašich požadavcích na škálovatelnost, spolehlivost a snadnost správy. Níže najdete pokyny pro nasazení MCP serverů lokálně, v kontejnerech a do cloudu.

## Přehled

Tato lekce pokrývá, jak nasadit aplikaci MCP Server.

## Výukové cíle

Na konci této lekce budete schopni:

- Zhodnotit různé přístupy k nasazení.
- Nasadit vaši aplikaci.

## Lokální vývoj a nasazení

Pokud má být váš server používán běžící na počítači uživatele, můžete postupovat podle následujících kroků:

1. **Stáhněte server**. Pokud jste server nenapsali vy, stáhněte si ho nejprve do svého počítače.
1. **Spusťte proces serveru**: Spusťte vaši aplikaci MCP serveru.

Pro SSE (není potřeba u typu serveru stdio)

1. **Konfigurujte síťování**: Zajistěte, že je server přístupný na očekávaném portu
1. **Připojte klienty**: Používejte lokální připojovací URL jako `http://localhost:3000`

## Nasazení do cloudu

MCP servery lze nasadit na různé cloudové platformy:

- **Serverless Functions**: Nasazení lehkých MCP serverů jako serverless funkcí
- **Služby kontejnerů**: Používejte služby jako Azure Container Apps, AWS ECS nebo Google Cloud Run
- **Kubernetes**: Nasazení a správa MCP serverů v Kubernetes clusteru pro vysokou dostupnost

### Příklad: Azure Container Apps

Azure Container Apps podporují nasazení MCP Serverů. Je to stále ve vývoji a aktuálně podporuje SSE servery.

Tady je, jak na to:

1. Naklonujte repozitář:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Spusťte ho lokálně, abyste si věci otestovali:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Pro vyzkoušení lokálně vytvořte soubor *mcp.json* v adresáři *.vscode* a přidejte následující obsah:

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

  Jakmile bude SSE server spuštěn, můžete kliknout na ikonu přehrávání v JSON souboru, nyní byste měli vidět, že nástroje na serveru jsou zachyceny GitHub Copilot, viz ikona nástroje.

1. Pro nasazení spusťte následující příkaz:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

A je to, nasadíte ho lokálně, nasadíte ho do Azure tímto způsobem.

## Další zdroje

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Článek o Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Repo MCP pro Azure Container Apps](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## Co dál

- Dále: [Pokročilá témata serveru](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o vyloučení odpovědnosti**:  
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o přesnost, mějte prosím na paměti, že automatické překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho původním jazyce by měl být považován za závazný zdroj. Pro zásadní informace se doporučuje profesionální lidský překlad. Nebereme na sebe odpovědnost za jakékoli nedorozumění nebo nesprávné výklady vyplývající z použití tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
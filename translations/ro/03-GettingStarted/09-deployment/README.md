# Implementarea serverelor MCP

Implementarea serverului tău MCP permite altora să acceseze instrumentele și resursele sale dincolo de mediul tău local. Există mai multe strategii de implementare de luat în considerare, în funcție de cerințele tale privind scalabilitatea, fiabilitatea și ușurința gestionării. Mai jos vei găsi îndrumări pentru implementarea serverelor MCP local, în containere și în cloud.

## Prezentare generală

Această lecție acoperă modul de a implementa aplicația ta MCP Server.

## Obiective de învățare

La finalul acestei lecții, vei putea să:

- Evaluezi diferite abordări de implementare.
- Îți implementezi aplicația.

## Dezvoltare și implementare locală

Dacă serverul tău este destinat să fie utilizat rulând pe mașina utilizatorului, poți urma pașii următori:

1. **Descarcă serverul**. Dacă nu ai scris serverul, descarcă-l mai întâi pe mașina ta.
1. **Pornește procesul serverului**: Rulează aplicația ta MCP server

Pentru SSE (nu este necesar pentru serverele de tip stdio)

1. **Configurează rețeaua**: Asigură-te că serverul este accesibil pe portul așteptat
1. **Conectează clienții**: Folosește URL-uri de conexiune locale precum `http://localhost:3000`

## Implementare în cloud

Serverele MCP pot fi implementate pe diverse platforme cloud:

- **Funcții serverless**: Implementarea serverelor MCP ușoare ca funcții serverless
- **Servicii de containere**: Folosește servicii precum Azure Container Apps, AWS ECS sau Google Cloud Run
- **Kubernetes**: Implementarea și gestionarea serverelor MCP în clustere Kubernetes pentru disponibilitate ridicată

### Exemplu: Azure Container Apps

Azure Container Apps suportă implementarea serverelor MCP. Este încă în curs de dezvoltare și în prezent suportă serverele SSE.

Iată cum poți proceda:

1. Clonează un repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Rulează-l local pentru a testa:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Pentru a încerca local, creează un fișier *mcp.json* într-un director *.vscode* și adaugă următorul conținut:

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

  Odată ce serverul SSE este pornit, poți face clic pe pictograma de redare din fișierul JSON, acum ar trebui să vezi instrumentele de pe server preluate de GitHub Copilot, vezi pictograma Tool.

1. Pentru a implementa, rulează comanda următoare:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Asta este tot, implementează-l local, implementează-l pe Azure urmând acești pași.

## Resurse suplimentare

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Articol Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Repo Azure Container Apps MCP](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## Ce urmează

- Următorul: [Subiecte avansate despre server](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare de responsabilitate**:  
Acest document a fost tradus folosind serviciul de traducere automată AI [Co-op Translator](https://github.com/Azure/co-op-translator). Deși ne străduim pentru acuratețe, vă rugăm să țineți cont că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un specialist uman. Nu ne asumăm responsabilitatea pentru eventuale neînțelegeri sau interpretări greșite rezultate din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
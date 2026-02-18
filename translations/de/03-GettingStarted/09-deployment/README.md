# Bereitstellung von MCP-Servern

Die Bereitstellung Ihres MCP-Servers ermöglicht anderen den Zugriff auf dessen Tools und Ressourcen über Ihre lokale Umgebung hinaus. Es gibt verschiedene Bereitstellungsstrategien, die Sie je nach Ihren Anforderungen an Skalierbarkeit, Zuverlässigkeit und einfache Verwaltung berücksichtigen können. Nachfolgend finden Sie Anleitungen zur Bereitstellung von MCP-Servern lokal, in Containern und in der Cloud.

## Überblick

Diese Lektion behandelt, wie Sie Ihre MCP-Server-App bereitstellen.

## Lernziele

Am Ende dieser Lektion können Sie:

- Verschiedene Bereitstellungsansätze bewerten.
- Ihre App bereitstellen.

## Lokale Entwicklung und Bereitstellung

Wenn Ihr Server dazu gedacht ist, auf den Rechnern der Nutzer ausgeführt zu werden, können Sie folgende Schritte befolgen:

1. **Server herunterladen**. Wenn Sie den Server nicht selbst geschrieben haben, laden Sie ihn zuerst auf Ihren Rechner herunter.  
1. **Serverprozess starten**: Führen Sie Ihre MCP-Server-Anwendung aus.

Für SSE (nicht notwendig für stdio-Typ Server)

1. **Netzwerk konfigurieren**: Stellen Sie sicher, dass der Server auf dem erwarteten Port erreichbar ist.  
1. **Clients verbinden**: Verwenden Sie lokale Verbindungs-URLs wie `http://localhost:3000`.

## Cloud-Bereitstellung

MCP-Server können auf verschiedenen Cloud-Plattformen bereitgestellt werden:

- **Serverless Funktionen**: Leichtgewichtige MCP-Server als serverlose Funktionen bereitstellen  
- **Container-Dienste**: Verwenden Sie Dienste wie Azure Container Apps, AWS ECS oder Google Cloud Run  
- **Kubernetes**: MCP-Server in Kubernetes-Clustern für hohe Verfügbarkeit bereitstellen und verwalten

### Beispiel: Azure Container Apps

Azure Container Apps unterstützen die Bereitstellung von MCP-Servern. Dies ist noch in Arbeit und unterstützt derzeit SSE-Server.

So gehen Sie vor:

1. Klonen Sie ein Repository:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Führen Sie es lokal aus, um es zu testen:

  ```sh
  uv venv
  uv sync

  # Linux/macOS
  export API_KEYS=<AN_API_KEY>
  # Windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Um es lokal auszuprobieren, erstellen Sie eine *mcp.json*-Datei in einem *.vscode*-Verzeichnis und fügen Sie den folgenden Inhalt hinzu:

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

  Sobald der SSE-Server gestartet ist, können Sie im JSON-File auf das Wiedergabe-Symbol klicken; Sie sollten nun sehen, wie Werkzeuge auf dem Server von GitHub Copilot erkannt werden, siehe das Werkzeug-Symbol.

1. Um bereitzustellen, führen Sie den folgenden Befehl aus:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Da haben Sie es – stellen Sie es lokal bereit, oder deployen Sie es über diese Schritte zu Azure.

## Zusätzliche Ressourcen

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps Artikel](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP Repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## Was kommt als Nächstes

- Weiter mit: [Fortgeschrittene Serverthemen](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:  
Dieses Dokument wurde mithilfe des KI-Übersetzungsdienstes [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, kann es bei automatisierten Übersetzungen zu Fehlern oder Ungenauigkeiten kommen. Das Originaldokument in der ursprünglichen Sprache ist als maßgebliche Quelle anzusehen. Für wichtige Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Nutzung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
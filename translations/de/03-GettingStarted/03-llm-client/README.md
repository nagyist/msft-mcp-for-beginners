# Einen Client mit LLM erstellen

Bisher hast du gesehen, wie man einen Server und einen Client erstellt. Der Client konnte explizit den Server aufrufen, um seine Tools, Ressourcen und Prompts aufzulisten. Das ist jedoch keine sehr praktische Vorgehensweise. Dein Nutzer lebt in einem agentischen Zeitalter und erwartet, Prompts zu verwenden und mit einem LLM zu kommunizieren. Für deinen Nutzer ist es egal, ob du MCP verwendest oder nicht, um deine Fähigkeiten zu speichern, aber er erwartet, natürliche Sprache zu benutzen, um zu interagieren. Wie lösen wir das? Die Lösung besteht darin, dem Client ein LLM hinzuzufügen.

## Überblick

In dieser Lektion konzentrieren wir uns darauf, ein LLM hinzuzufügen, um deinen Client zu erstellen, und zeigen, wie dies eine viel bessere Erfahrung für deinen Nutzer bietet.

## Lernziele

Am Ende dieser Lektion wirst du in der Lage sein:

- Einen Client mit einem LLM zu erstellen.
- Nahtlos mit einem MCP-Server unter Verwendung eines LLM zu interagieren.
- Eine bessere Endnutzererfahrung auf der Client-Seite zu bieten.

## Vorgehensweise

Versuchen wir zu verstehen, welchen Ansatz wir verfolgen müssen. Ein LLM hinzuzufügen klingt einfach, aber werden wir das tatsächlich tun?

So wird der Client mit dem Server interagieren:

1. Verbindung mit dem Server herstellen.

1. Fähigkeiten, Prompts, Ressourcen und Tools auflisten und deren Schema speichern.

1. Ein LLM hinzufügen und die gespeicherten Fähigkeiten mit ihrem Schema in einem Format übergeben, das das LLM versteht.

1. Einen Nutzer-Prompt verarbeiten, indem er zusammen mit den vom Client aufgelisteten Tools an das LLM übergeben wird.

Großartig, jetzt verstehen wir, wie wir das auf hoher Ebene machen können, lassen wir es uns im folgenden Übungsteil ausprobieren.

## Übung: Einen Client mit einem LLM erstellen

In dieser Übung lernen wir, wie wir ein LLM zu unserem Client hinzufügen.

### Authentifizierung mit GitHub Personal Access Token

Das Erstellen eines GitHub-Tokens ist ein unkomplizierter Prozess. So funktioniert es:

- Gehe zu GitHub Einstellungen – Klicke auf dein Profilbild oben rechts und wähle Einstellungen.
- Navigiere zu Entwickler-Einstellungen – Scrolle nach unten und klicke auf Entwickler-Einstellungen.
- Wähle Persönliche Zugriffstoken – Klicke auf Fein abgestufte Token und dann auf Neues Token generieren.
- Konfiguriere dein Token – Füge eine Notiz zur Referenz hinzu, setze ein Ablaufdatum und wähle die erforderlichen Bereiche (Berechtigungen). Stelle sicher, dass du die Berechtigung „Models“ hinzufügst.
- Erzeuge und kopiere das Token – Klicke auf Token generieren und kopiere es sofort, da du es danach nicht erneut sehen wirst.

### -1- Mit dem Server verbinden

Lass uns zuerst unseren Client erstellen:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Importiere zod für Schema-Validierung

class MCPClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", 
            apiKey: process.env.GITHUB_TOKEN,
        });

        this.client = new Client(
            {
                name: "example-client",
                version: "1.0.0"
            },
            {
                capabilities: {
                prompts: {},
                resources: {},
                tools: {}
                }
            }
            );    
    }
}
```

Im obigen Code haben wir:

- Die benötigten Bibliotheken importiert.
- Eine Klasse mit zwei Mitgliedern, `client` und `openai`, erstellt, die uns helfen, einen Client zu verwalten und mit einem LLM zu interagieren.
- Unsere LLM-Instanz so konfiguriert, dass GitHub Models durch Setzen von `baseUrl` auf die Inference-API verwendet werden.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Serverparameter für StdIO-Verbindung erstellen
server_params = StdioServerParameters(
    command="mcp",  # Ausführbare Datei
    args=["run", "server.py"],  # Optionale Befehlszeilenargumente
    env=None,  # Optionale Umgebungsvariablen
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # Verbindung initialisieren
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

Im obigen Code haben wir:

- Die benötigten Bibliotheken für MCP importiert.
- Einen Client erstellt.

#### .NET

```csharp
using Azure;
using Azure.AI.Inference;
using Azure.Identity;
using System.Text.Json;
using ModelContextProtocol.Client;
using ModelContextProtocol.Protocol.Transport;
using System.Text.Json;

var clientTransport = new StdioClientTransport(new()
{
    Name = "Demo Server",
    Command = "/workspaces/mcp-for-beginners/03-GettingStarted/02-client/solution/server/bin/Debug/net8.0/server",
    Arguments = [],
});

await using var mcpClient = await McpClientFactory.CreateAsync(clientTransport);
```

#### Java

Zuerst musst du die LangChain4j-Abhängigkeiten zu deiner `pom.xml`-Datei hinzufügen. Füge diese Abhängigkeiten hinzu, um die MCP-Integration und die Unterstützung von GitHub Models zu ermöglichen:

```xml
<properties>
    <langchain4j.version>1.0.0-beta3</langchain4j.version>
</properties>

<dependencies>
    <!-- LangChain4j MCP Integration -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-mcp</artifactId>
        <version>${langchain4j.version}</version>
    </dependency>
    
    <!-- OpenAI Official API Client -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-open-ai-official</artifactId>
        <version>${langchain4j.version}</version>
    </dependency>
    
    <!-- GitHub Models Support -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-github-models</artifactId>
        <version>${langchain4j.version}</version>
    </dependency>
    
    <!-- Spring Boot Starter (optional, for production apps) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

Erstelle dann deine Java-Client-Klasse:

```java
import dev.langchain4j.mcp.McpToolProvider;
import dev.langchain4j.mcp.client.DefaultMcpClient;
import dev.langchain4j.mcp.client.McpClient;
import dev.langchain4j.mcp.client.transport.McpTransport;
import dev.langchain4j.mcp.client.transport.http.HttpMcpTransport;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.openaiofficial.OpenAiOfficialChatModel;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.service.tool.ToolProvider;

import java.time.Duration;
import java.util.List;

public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {        // Konfigurieren Sie das LLM für die Verwendung von GitHub-Modellen
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Erstellen Sie einen MCP-Transport zur Verbindung mit dem Server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP-Client erstellen
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

Im obigen Code haben wir:

- **LangChain4j-Abhängigkeiten hinzugefügt**: Erforderlich für die MCP-Integration, den offiziellen OpenAI-Client und die GitHub-Models-Unterstützung.
- **Die LangChain4j-Bibliotheken importiert**: Für MCP-Integration und OpenAI-Chatmodell-Funktionalität.
- **Ein `ChatLanguageModel` erstellt**: Konfiguriert, um GitHub Models mit deinem GitHub-Token zu verwenden.
- **HTTP-Transport eingerichtet**: Mit Server-Sent Events (SSE), um eine Verbindung zum MCP-Server herzustellen.
- **Einen MCP-Client erstellt**: Der die Kommunikation mit dem Server übernimmt.
- **Die eingebaute MCP-Unterstützung von LangChain4j genutzt**: Was die Integration zwischen LLMs und MCP-Servern vereinfacht.

#### Rust

Dieses Beispiel setzt voraus, dass du einen Rust-basierten MCP-Server laufen hast. Falls nicht, siehe die Lektion [01-first-server](../01-first-server/README.md), um den Server zu erstellen.

Sobald du deinen Rust MCP-Server hast, öffne ein Terminal und navigiere zum gleichen Verzeichnis wie der Server. Führe dann den folgenden Befehl aus, um ein neues LLM-Client-Projekt zu erstellen:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Füge folgende Abhängigkeiten zu deiner `Cargo.toml`-Datei hinzu:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Es gibt keine offizielle Rust-Bibliothek für OpenAI, allerdings ist das `async-openai` Crate eine [community-gepflegte Bibliothek](https://platform.openai.com/docs/libraries/rust#rust), die häufig verwendet wird.

Öffne die Datei `src/main.rs` und ersetze ihren Inhalt mit folgendem Code:

```rust
use async_openai::{Client, config::OpenAIConfig};
use rmcp::{
    RmcpError,
    model::{CallToolRequestParam, ListToolsResult},
    service::{RoleClient, RunningService, ServiceExt},
    transport::{ConfigureCommandExt, TokioChildProcess},
};
use serde_json::{Value, json};
use std::error::Error;
use tokio::process::Command;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    // Anfangsnachricht
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI-Client einrichten
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP-Client einrichten
    let server_dir = std::path::Path::new(env!("CARGO_MANIFEST_DIR"))
        .parent()
        .unwrap()
        .join("calculator-server");

    let mcp_client = ()
        .serve(
            TokioChildProcess::new(Command::new("cargo").configure(|cmd| {
                cmd.arg("run").current_dir(server_dir);
            }))
            .map_err(RmcpError::transport_creation::<TokioChildProcess>)?,
        )
        .await?;

    // TODO: MCP-Werkzeugliste abrufen

    // TODO: LLM-Unterhaltung mit Werkzeugaufrufen

    Ok(())
}
```

Dieser Code richtet eine grundlegende Rust-Anwendung ein, die sich mit einem MCP-Server und GitHub Models für LLM-Interaktionen verbindet.

> [!IMPORTANT]
> Stelle sicher, dass du die Umgebungsvariable `OPENAI_API_KEY` mit deinem GitHub-Token setzt, bevor du die Anwendung ausführst.

Super, unser nächster Schritt ist es, die Fähigkeiten auf dem Server aufzulisten.

### -2- Fähigkeiten des Servers auflisten

Jetzt verbinden wir uns mit dem Server und fragen nach seinen Fähigkeiten:

#### TypeScript

Füge in derselben Klasse die folgenden Methoden hinzu:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // Werkzeuge auflisten
    const toolsResult = await this.client.listTools();
}
```

Im obigen Code haben wir:

- Code zum Verbinden mit dem Server hinzugefügt, `connectToServer`.
- Eine Methode `run` erstellt, die für den Ablauf unserer App verantwortlich ist. Bisher listet sie nur die Werkzeuge auf, aber wir werden bald mehr hinzufügen.

#### Python

```python
# Verfügbare Ressourcen auflisten
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Verfügbare Werkzeuge auflisten
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Das haben wir hinzugefügt:

- Ressourcen und Tools aufgelistet und ausgegeben. Für Tools listen wir auch das `inputSchema`, das wir später verwenden.

#### .NET

```csharp
async Task<List<ChatCompletionsToolDefinition>> GetMcpTools()
{
    Console.WriteLine("Listing tools");
    var tools = await mcpClient.ListToolsAsync();

    List<ChatCompletionsToolDefinition> toolDefinitions = new List<ChatCompletionsToolDefinition>();

    foreach (var tool in tools)
    {
        Console.WriteLine($"Connected to server with tools: {tool.Name}");
        Console.WriteLine($"Tool description: {tool.Description}");
        Console.WriteLine($"Tool parameters: {tool.JsonSchema}");

        // TODO: convert tool definition from MCP tool to LLm tool     
    }

    return toolDefinitions;
}
```

Im obigen Code haben wir:

- Die auf dem MCP-Server verfügbaren Tools aufgelistet.
- Für jedes Tool Name, Beschreibung und dessen Schema aufgelistet. Letzteres verwenden wir, um bald die Tools aufzurufen.

#### Java

```java
// Erstellen Sie einen Tool-Anbieter, der MCP-Tools automatisch entdeckt
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// Der MCP-Tool-Anbieter übernimmt automatisch:
// - Auflisten verfügbarer Tools vom MCP-Server
// - Konvertieren von MCP-Tool-Schemas in das LangChain4j-Format
// - Verwaltung der Tool-Ausführung und Antworten
```

Im obigen Code haben wir:

- Einen `McpToolProvider` erstellt, der alle Tools vom MCP-Server automatisch entdeckt und registriert.
- Der Tool-Provider übernimmt intern die Konvertierung zwischen MCP-Tool-Schemas und dem Tool-Format von LangChain4j.
- Dieser Ansatz abstrahiert den manuellen Prozess des Tool-Auflistens und der Konvertierung.

#### Rust

Das Abrufen von Tools vom MCP-Server erfolgt über die Methode `list_tools`. Füge in deiner `main`-Funktion, nachdem du den MCP-Client eingerichtet hast, folgenden Code hinzu:

```rust
// MCP Werkzeugauflistung abrufen
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Fähigkeiten des Servers in LLM-Tools umwandeln

Der nächste Schritt nach dem Auflisten der Server-Fähigkeiten ist, sie in ein Format zu konvertieren, das das LLM versteht. Sobald wir das tun, können wir diese Fähigkeiten als Tools für unser LLM bereitstellen.

#### TypeScript

1. Füge folgenden Code hinzu, um die Antwort vom MCP-Server in ein Tool-Format umzuwandeln, das das LLM verwenden kann:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Erstellen Sie ein Zod-Schema basierend auf dem Eingabe_Schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Setzen Sie den Typ explizit auf "Funktion"
            function: {
            name: tool.name,
            description: tool.description,
            parameters: {
            type: "object",
            properties: tool.input_schema.properties,
            required: tool.input_schema.required,
            },
            },
        };
    }

    ```

    Der obige Code nimmt eine Antwort vom MCP-Server und wandelt sie in ein Tool-Definitionsformat um, das das LLM versteht.

1. Lass uns nun die Methode `run` aktualisieren, um die Server-Fähigkeiten aufzulisten:

    ```typescript
    async run() {
        console.log("Asking server for available tools");
        const toolsResult = await this.client.listTools();
        const tools = toolsResult.tools.map((tool) => {
            return this.openAiToolAdapter({
            name: tool.name,
            description: tool.description,
            input_schema: tool.inputSchema,
            });
        });
    }
    ```

    Im obigen Code haben wir die Methode `run` aktualisiert, um das Ergebnis zu mappen und für jeden Eintrag `openAiToolAdapter` aufzurufen.

#### Python

1. Zuerst erstellen wir die folgende Umwandlungsfunktion:

    ```python
    def convert_to_llm_tool(tool):
        tool_schema = {
            "type": "function",
            "function": {
                "name": tool.name,
                "description": tool.description,
                "type": "function",
                "parameters": {
                    "type": "object",
                    "properties": tool.inputSchema["properties"]
                }
            }
        }

        return tool_schema
    ```

    In der Funktion `convert_to_llm_tools` nehmen wir eine MCP-Tool-Antwort und wandeln sie in ein Format um, das das LLM versteht.

1. Als Nächstes aktualisieren wir unseren Client-Code, um diese Funktion wie folgt zu verwenden:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Hier fügen wir einen Aufruf zu `convert_to_llm_tool` hinzu, um die MCP-Tool-Antwort in etwas umzuwandeln, das wir dem LLM später übergeben können.

#### .NET

1. Lass uns Code hinzufügen, um die MCP-Tool-Antwort in etwas umzuwandeln, das das LLM versteht

```csharp
ChatCompletionsToolDefinition ConvertFrom(string name, string description, JsonElement jsonElement)
{ 
    // convert the tool to a function definition
    FunctionDefinition functionDefinition = new FunctionDefinition(name)
    {
        Description = description,
        Parameters = BinaryData.FromObjectAsJson(new
        {
            Type = "object",
            Properties = jsonElement
        },
        new JsonSerializerOptions() { PropertyNamingPolicy = JsonNamingPolicy.CamelCase })
    };

    // create a tool definition
    ChatCompletionsToolDefinition toolDefinition = new ChatCompletionsToolDefinition(functionDefinition);
    return toolDefinition;
}
```

Im obigen Code haben wir:

- Eine Funktion `ConvertFrom` erstellt, die Name, Beschreibung und Eingabeschema übernimmt.
- Funktionalität definiert, die eine FunctionDefinition erzeugt, welche an eine ChatCompletionsDefinition übergeben wird. Letzteres versteht das LLM.

1. Lass uns sehen, wie wir bestehenden Code aktualisieren, um diese Funktion zu nutzen:

    ```csharp
    async Task<List<ChatCompletionsToolDefinition>> GetMcpTools()
    {
        Console.WriteLine("Listing tools");
        var tools = await mcpClient.ListToolsAsync();

        List<ChatCompletionsToolDefinition> toolDefinitions = new List<ChatCompletionsToolDefinition>();

        foreach (var tool in tools)
        {
            Console.WriteLine($"Connected to server with tools: {tool.Name}");
            Console.WriteLine($"Tool description: {tool.Description}");
            Console.WriteLine($"Tool parameters: {tool.JsonSchema}");

            JsonElement propertiesElement;
            tool.JsonSchema.TryGetProperty("properties", out propertiesElement);

            var def = ConvertFrom(tool.Name, tool.Description, propertiesElement);
            Console.WriteLine($"Tool definition: {def}");
            toolDefinitions.Add(def);

            Console.WriteLine($"Properties: {propertiesElement}");        
        }

        return toolDefinitions;
    }
    ```    In the preceding code, we've:

    - Update the function to convert the MCP tool response to an LLm tool. Let's highlight the code we added:

        ```csharp
        JsonElement propertiesElement;
        tool.JsonSchema.TryGetProperty("properties", out propertiesElement);

        var def = ConvertFrom(tool.Name, tool.Description, propertiesElement);
        Console.WriteLine($"Tool definition: {def}");
        toolDefinitions.Add(def);
        ```

        The input schema is part of the tool response but on the "properties" attribute, so we need to extract. Furthermore, we now call `ConvertFrom` with the tool details. Now we've done the heavy lifting, let's see how it call comes together as we handle a user prompt next.

#### Java

```java
// Erstellen Sie eine Bot-Schnittstelle für die Interaktion in natürlicher Sprache
public interface Bot {
    String chat(String prompt);
}

// Konfigurieren Sie den KI-Dienst mit LLM- und MCP-Tools
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

Im obigen Code haben wir:

- Ein einfaches `Bot`-Interface für natürliche Sprachinteraktionen definiert.
- LangChain4j's `AiServices` genutzt, um das LLM automatisch mit dem MCP-Tool-Provider zu verbinden.
- Das Framework handhabt automatisch die Umwandlung von Tool-Schemas und Funktionsaufrufen im Hintergrund.
- Dieser Ansatz eliminiert die manuelle Tool-Konvertierung – LangChain4j übernimmt die gesamte Komplexität der Umwandlung von MCP-Tools in ein LLM-kompatibles Format.

#### Rust

Um die MCP-Tool-Antwort in ein Format umzuwandeln, das das LLM versteht, fügen wir eine Hilfsfunktion hinzu, die die Tool-Liste formatiert. Füge folgenden Code in deine `main.rs`-Datei unterhalb der `main`-Funktion ein. Diese Funktion wird bei Anfragen an das LLM verwendet:

```rust
async fn format_tools(tools: &ListToolsResult) -> Result<Vec<Value>, Box<dyn Error>> {
    let tools_json = serde_json::to_value(tools)?;
    let Some(tools_array) = tools_json.get("tools").and_then(|t| t.as_array()) else {
        return Ok(vec![]);
    };

    let formatted_tools = tools_array
        .iter()
        .filter_map(|tool| {
            let name = tool.get("name")?.as_str()?;
            let description = tool.get("description")?.as_str()?;
            let schema = tool.get("inputSchema")?;

            Some(json!({
                "type": "function",
                "function": {
                    "name": name,
                    "description": description,
                    "parameters": {
                        "type": "object",
                        "properties": schema.get("properties").unwrap_or(&json!({})),
                        "required": schema.get("required").unwrap_or(&json!([]))
                    }
                }
            }))
        })
        .collect();

    Ok(formatted_tools)
}
```

Super, wir sind bereit, Benutzeranfragen zu bearbeiten, also machen wir mit dem nächsten Schritt weiter.

### -4- Verarbeitung von Benutzer-Prompt-Anfragen

In diesem Teil des Codes werden wir Benutzeranfragen verarbeiten.

#### TypeScript

1. Füge eine Methode hinzu, die dazu dient, unser LLM aufzurufen:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Rufen Sie das Werkzeug des Servers auf
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Machen Sie etwas mit dem Ergebnis
        // TODO

        }
    }
    ```

    Im obigen Code haben wir:

    - Eine Methode `callTools` hinzugefügt.
    - Die Methode nimmt eine LLM-Antwort und prüft, welche Tools aufgerufen wurden, falls vorhanden:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // Werkzeug aufrufen
        }
        ```

    - Ruft ein Tool auf, falls das LLM angibt, es soll aufgerufen werden:

        ```typescript
        // 2. Rufe das Werkzeug des Servers auf
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Mache etwas mit dem Ergebnis
        // TODO
        ```

1. Aktualisiere die Methode `run`, um Aufrufe an das LLM und an `callTools` einzuschließen:

    ```typescript

    // 1. Erstellen Sie Nachrichten, die Eingaben für das LLM sind
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. Aufruf des LLM
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Gehen Sie die LLM-Antwort durch, prüfen Sie für jede Auswahl, ob sie Werkzeugaufrufe enthält
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Super, hier der komplette Code:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Importiere zod für Schema-Validierung

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // Möglicherweise muss die URL in Zukunft geändert werden: https://models.github.ai/inference
            apiKey: process.env.GITHUB_TOKEN,
        });

        this.client = new Client(
            {
                name: "example-client",
                version: "1.0.0"
            },
            {
                capabilities: {
                prompts: {},
                resources: {},
                tools: {}
                }
            }
            );    
    }

    async connectToServer(transport: Transport) {
        await this.client.connect(transport);
        this.run();
        console.error("MCPClient started on stdin/stdout");
    }

    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
          }) {
          // Erstelle ein zod-Schema basierend auf dem input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Setze den Typ explizit auf "Funktion"
            function: {
              name: tool.name,
              description: tool.description,
              parameters: {
              type: "object",
              properties: tool.input_schema.properties,
              required: tool.input_schema.required,
              },
            },
          };
    }
    
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
      ) {
        for (const tool_call of tool_calls) {
          const toolName = tool_call.function.name;
          const args = tool_call.function.arguments;
    
          console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);
    
    
          // 2. Rufe das Werkzeug des Servers auf
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Mache etwas mit dem Ergebnis
          // TODO
    
         }
    }

    async run() {
        console.log("Asking server for available tools");
        const toolsResult = await this.client.listTools();
        const tools = toolsResult.tools.map((tool) => {
            return this.openAiToolAdapter({
              name: tool.name,
              description: tool.description,
              input_schema: tool.inputSchema,
            });
        });

        const prompt = "What is the sum of 2 and 3?";
    
        const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

        console.log("Querying LLM: ", messages[0].content);
        let response = this.openai.chat.completions.create({
            model: "gpt-4.1-mini",
            max_tokens: 1000,
            messages,
            tools: tools,
        });    

        let results: any[] = [];
    
        // 1. Gehe die LLM-Antwort durch, überprüfe bei jeder Wahl, ob Werkzeugaufrufe enthalten sind
        (await response).choices.map(async (choice: { message: any; }) => {
          const message = choice.message;
          if (message.tool_calls) {
              console.log("Making tool call")
              await this.callTools(message.tool_calls, results);
          }
        });
    }
    
}

let client = new MyClient();
 const transport = new StdioClientTransport({
            command: "node",
            args: ["./build/index.js"]
        });

client.connectToServer(transport);
```

#### Python

1. Füge einige Importe hinzu, die benötigt werden, um ein LLM aufzurufen:

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Als Nächstes fügen wir die Funktion hinzu, die das LLM aufruft:

    ```python
    # llm

    def call_llm(prompt, functions):
        token = os.environ["GITHUB_TOKEN"]
        endpoint = "https://models.inference.ai.azure.com"

        model_name = "gpt-4o"

        client = ChatCompletionsClient(
            endpoint=endpoint,
            credential=AzureKeyCredential(token),
        )

        print("CALLING LLM")
        response = client.complete(
            messages=[
                {
                "role": "system",
                "content": "You are a helpful assistant.",
                },
                {
                "role": "user",
                "content": prompt,
                },
            ],
            model=model_name,
            tools = functions,
            # Optionale Parameter
            temperature=1.,
            max_tokens=1000,
            top_p=1.    
        )

        response_message = response.choices[0].message
        
        functions_to_call = []

        if response_message.tool_calls:
            for tool_call in response_message.tool_calls:
                print("TOOL: ", tool_call)
                name = tool_call.function.name
                args = json.loads(tool_call.function.arguments)
                functions_to_call.append({ "name": name, "args": args })

        return functions_to_call
    ```

    Im obigen Code haben wir:

    - Unsere Funktionen, die wir auf dem MCP-Server gefunden und umgewandelt haben, an das LLM übergeben.
    - Dann das LLM mit diesen Funktionen aufgerufen.
    - Anschließend das Ergebnis geprüft, um herauszufinden, welche Funktionen gegebenenfalls aufgerufen werden sollen.
    - Schließlich eine Liste von Funktionen zur Ausführung übergeben.

1. Letzter Schritt, wir aktualisieren unseren Hauptcode:

    ```python
    prompt = "Add 2 to 20"

    # frage das LLM, welche Werkzeuge, falls vorhanden, alle sind
    functions_to_call = call_llm(prompt, functions)

    # rufe vorgeschlagene Funktionen auf
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Damit war der letzte Schritt erledigt. Im obigen Code:

    - Rufen wir ein MCP-Tool via `call_tool` auf, indem wir eine Funktion verwenden, von der das LLM aufgrund unseres Prompts denkt, dass sie aufgerufen werden soll.
    - Wir geben das Ergebnis des Tool-Aufrufs an den MCP-Server aus.

#### .NET

1. Hier ein Beispielcode für eine LLM-Prompt-Anfrage:

    ```csharp
    var tools = await GetMcpTools();

    for (int i = 0; i < tools.Count; i++)
    {
        var tool = tools[i];
        Console.WriteLine($"MCP Tools def: {i}: {tool}");
    }

    // 0. Define the chat history and the user message
    var userMessage = "add 2 and 4";

    chatHistory.Add(new ChatRequestUserMessage(userMessage));

    // 1. Define tools
    ChatCompletionsToolDefinition def = CreateToolDefinition();


    // 2. Define options, including the tools
    var options = new ChatCompletionsOptions(chatHistory)
    {
        Model = "gpt-4.1-mini",
        Tools = { tools[0] }
    };

    // 3. Call the model  

    ChatCompletions? response = await client.CompleteAsync(options);
    var content = response.Content;

    ```

    Im obigen Code haben wir:

    - Tools vom MCP-Server abgerufen, `var tools = await GetMcpTools()`.
    - Einen Nutzer-Prompt `userMessage` definiert.
    - Ein Optionsobjekt erstellt, das Modell und Tools spezifiziert.
    - Eine Anfrage an das LLM gemacht.

1. Ein letzter Schritt, wir prüfen, ob das LLM denkt, dass wir eine Funktion aufrufen sollen:

    ```csharp
    // 4. Check if the response contains a function call
    ChatCompletionsToolCall? calls = response.ToolCalls.FirstOrDefault();
    for (int i = 0; i < response.ToolCalls.Count; i++)
    {
        var call = response.ToolCalls[i];
        Console.WriteLine($"Tool call {i}: {call.Name} with arguments {call.Arguments}");
        //Tool call 0: add with arguments {"a":2,"b":4}

        var dict = JsonSerializer.Deserialize<Dictionary<string, object>>(call.Arguments);
        var result = await mcpClient.CallToolAsync(
            call.Name,
            dict!,
            cancellationToken: CancellationToken.None
        );

        Console.WriteLine(result.Content.First(c => c.Type == "text").Text);

    }
    ```

    Im obigen Code haben wir:

    - Eine Liste von Funktionsaufrufen durchlaufen.
    - Für jeden Tool-Aufruf Name und Argumente geparst und das Tool auf dem MCP-Server mittels MCP-Client aufgerufen. Am Ende geben wir die Resultate aus.

Hier der vollständige Code:

```csharp
using Azure;
using Azure.AI.Inference;
using Azure.Identity;
using System.Text.Json;
using ModelContextProtocol.Client;
using ModelContextProtocol.Protocol.Transport;
using System.Text.Json;

var endpoint = "https://models.inference.ai.azure.com";
var token = Environment.GetEnvironmentVariable("GITHUB_TOKEN"); // Your GitHub Access Token
var client = new ChatCompletionsClient(new Uri(endpoint), new AzureKeyCredential(token));
var chatHistory = new List<ChatRequestMessage>
{
    new ChatRequestSystemMessage("You are a helpful assistant that knows about AI")
};

var clientTransport = new StdioClientTransport(new()
{
    Name = "Demo Server",
    Command = "/workspaces/mcp-for-beginners/03-GettingStarted/02-client/solution/server/bin/Debug/net8.0/server",
    Arguments = [],
});

Console.WriteLine("Setting up stdio transport");

await using var mcpClient = await McpClientFactory.CreateAsync(clientTransport);

ChatCompletionsToolDefinition ConvertFrom(string name, string description, JsonElement jsonElement)
{ 
    // convert the tool to a function definition
    FunctionDefinition functionDefinition = new FunctionDefinition(name)
    {
        Description = description,
        Parameters = BinaryData.FromObjectAsJson(new
        {
            Type = "object",
            Properties = jsonElement
        },
        new JsonSerializerOptions() { PropertyNamingPolicy = JsonNamingPolicy.CamelCase })
    };

    // create a tool definition
    ChatCompletionsToolDefinition toolDefinition = new ChatCompletionsToolDefinition(functionDefinition);
    return toolDefinition;
}



async Task<List<ChatCompletionsToolDefinition>> GetMcpTools()
{
    Console.WriteLine("Listing tools");
    var tools = await mcpClient.ListToolsAsync();

    List<ChatCompletionsToolDefinition> toolDefinitions = new List<ChatCompletionsToolDefinition>();

    foreach (var tool in tools)
    {
        Console.WriteLine($"Connected to server with tools: {tool.Name}");
        Console.WriteLine($"Tool description: {tool.Description}");
        Console.WriteLine($"Tool parameters: {tool.JsonSchema}");

        JsonElement propertiesElement;
        tool.JsonSchema.TryGetProperty("properties", out propertiesElement);

        var def = ConvertFrom(tool.Name, tool.Description, propertiesElement);
        Console.WriteLine($"Tool definition: {def}");
        toolDefinitions.Add(def);

        Console.WriteLine($"Properties: {propertiesElement}");        
    }

    return toolDefinitions;
}

// 1. List tools on mcp server

var tools = await GetMcpTools();
for (int i = 0; i < tools.Count; i++)
{
    var tool = tools[i];
    Console.WriteLine($"MCP Tools def: {i}: {tool}");
}

// 2. Define the chat history and the user message
var userMessage = "add 2 and 4";

chatHistory.Add(new ChatRequestUserMessage(userMessage));


// 3. Define options, including the tools
var options = new ChatCompletionsOptions(chatHistory)
{
    Model = "gpt-4.1-mini",
    Tools = { tools[0] }
};

// 4. Call the model  

ChatCompletions? response = await client.CompleteAsync(options);
var content = response.Content;

// 5. Check if the response contains a function call
ChatCompletionsToolCall? calls = response.ToolCalls.FirstOrDefault();
for (int i = 0; i < response.ToolCalls.Count; i++)
{
    var call = response.ToolCalls[i];
    Console.WriteLine($"Tool call {i}: {call.Name} with arguments {call.Arguments}");
    //Tool call 0: add with arguments {"a":2,"b":4}

    var dict = JsonSerializer.Deserialize<Dictionary<string, object>>(call.Arguments);
    var result = await mcpClient.CallToolAsync(
        call.Name,
        dict!,
        cancellationToken: CancellationToken.None
    );

    Console.WriteLine(result.Content.First(c => c.Type == "text").Text);

}

// 5. Print the generic response
Console.WriteLine($"Assistant response: {content}");
```

#### Java

```java
try {
    // Führen Sie Anfragen in natürlicher Sprache aus, die automatisch MCP-Werkzeuge verwenden
    String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
    System.out.println(response);

    response = bot.chat("What's the square root of 144?");
    System.out.println(response);

    response = bot.chat("Show me the help for the calculator service");
    System.out.println(response);
} finally {
    mcpClient.close();
}
```

Im obigen Code haben wir:

- Einfache natürliche Sprachprompts verwendet, um mit den MCP-Server-Tools zu interagieren.
- Das LangChain4j-Framework handhabt automatisch:
  - Die Umwandlung von Nutzer-Prompts in Tool-Aufrufe, wenn notwendig.
  - Den Aufruf der passenden MCP-Tools basierend auf der Entscheidung des LLM.
  - Das Management des Gesprächsflusses zwischen LLM und MCP-Server.
- Die Methode `bot.chat()` gibt natürliche Sprachantworten zurück, die Ergebnisse von MCP-Toolausführungen enthalten können.
- Dieser Ansatz bietet eine nahtlose Nutzererfahrung, bei der die Nutzer nichts über die zugrunde liegende MCP-Implementierung wissen müssen.

Komplette Beispielcode:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .timeout(Duration.ofSeconds(60))
                .build();

        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        try {
            String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
            System.out.println(response);

            response = bot.chat("What's the square root of 144?");
            System.out.println(response);

            response = bot.chat("Show me the help for the calculator service");
            System.out.println(response);
        } finally {
            mcpClient.close();
        }
    }
}
```

#### Rust

Hier findet die meiste Arbeit statt. Wir rufen das LLM mit dem initialen Nutzer-Prompt auf und verarbeiten die Antwort, um zu prüfen, ob Tools aufgerufen werden müssen. Falls ja, rufen wir diese Tools auf und setzen das Gespräch mit dem LLM fort, bis keine weiteren Tool-Aufrufe mehr nötig sind und wir eine endgültige Antwort haben.

Wir werden mehrere Aufrufe an das LLM machen, daher definieren wir eine Funktion, die den LLM-Aufruf übernimmt. Füge folgende Funktion zu deiner `main.rs`-Datei hinzu:

```rust
async fn call_llm(
    client: &Client<OpenAIConfig>,
    messages: &[Value],
    tools: &ListToolsResult,
) -> Result<Value, Box<dyn Error>> {
    let response = client
        .completions()
        .create_byot(json!({
            "messages": messages,
            "model": "openai/gpt-4.1",
            "tools": format_tools(tools).await?,
        }))
        .await?;
    Ok(response)
}
```

Diese Funktion nimmt den LLM-Client, eine Liste von Nachrichten (inklusive Nutzer-Prompt), Tools vom MCP-Server und sendet eine Anfrage an das LLM, dabei gibt sie die Antwort zurück.
Die Antwort des LLM enthält ein Array von `choices`. Wir müssen das Ergebnis verarbeiten, um zu sehen, ob `tool_calls` vorhanden sind. Dies zeigt uns, dass das LLM anfragt, ob ein bestimmtes Tool mit Argumenten aufgerufen werden soll. Füge den folgenden Code am Ende deiner `main.rs`-Datei hinzu, um eine Funktion zu definieren, die die LLM-Antwort verarbeitet:

```rust
async fn process_llm_response(
    llm_response: &Value,
    mcp_client: &RunningService<RoleClient, ()>,
    openai_client: &Client<OpenAIConfig>,
    mcp_tools: &ListToolsResult,
    messages: &mut Vec<Value>,
) -> Result<(), Box<dyn Error>> {
    let Some(message) = llm_response
        .get("choices")
        .and_then(|c| c.as_array())
        .and_then(|choices| choices.first())
        .and_then(|choice| choice.get("message"))
    else {
        return Ok(());
    };

    // Inhalt drucken, falls verfügbar
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Werkzeugs Aufrufe behandeln
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Assistenten Nachricht hinzufügen

        // Jeden Werkzeugaufruf ausführen
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Werkzeugergebnis zu Nachrichten hinzufügen
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Gespräch mit Werkzeugergebnissen fortsetzen
        let response = call_llm(openai_client, messages, mcp_tools).await?;
        Box::pin(process_llm_response(
            &response,
            mcp_client,
            openai_client,
            mcp_tools,
            messages,
        ))
        .await?;
    }
    Ok(())
}
```

Wenn `tool_calls` vorhanden sind, werden die Tool-Informationen extrahiert, der MCP-Server mit der Tool-Anfrage aufgerufen und die Ergebnisse zu den Konversationsnachrichten hinzugefügt. Danach wird die Konversation mit dem LLM fortgesetzt, und die Nachrichten werden mit der Antwort des Assistenten und den Tool-Aufrufergebnissen aktualisiert.

Um Tool-Aufrufinformationen zu extrahieren, die das LLM für MCP-Aufrufe zurückgibt, fügen wir eine weitere Hilfsfunktion hinzu, die alles extrahiert, was für den Aufruf benötigt wird. Füge den folgenden Code am Ende deiner `main.rs`-Datei hinzu:

```rust
fn extract_tool_call_info(tool_call: &Value) -> Result<(String, String, String), Box<dyn Error>> {
    let tool_id = tool_call
        .get("id")
        .and_then(|id| id.as_str())
        .unwrap_or("")
        .to_string();
    let function = tool_call.get("function").ok_or("Missing function")?;
    let name = function
        .get("name")
        .and_then(|n| n.as_str())
        .unwrap_or("")
        .to_string();
    let args = function
        .get("arguments")
        .and_then(|a| a.as_str())
        .unwrap_or("{}")
        .to_string();
    Ok((tool_id, name, args))
}
```

Mit allen Bausteinen an Ort und Stelle können wir nun den initialen Nutzereingabe auffangen und das LLM aufrufen. Aktualisiere deine `main`-Funktion mit folgendem Code:

```rust
// LLM-Konversation mit Werkzeugaufrufen
let response = call_llm(&openai_client, &messages, &tools).await?;
process_llm_response(
    &response,
    &mcp_client,
    &openai_client,
    &tools,
    &mut messages,
)
.await?;
```

Dies fragt das LLM mit der initialen Nutzeranfrage nach der Summe zweier Zahlen ab und verarbeitet die Antwort, um Tool-Aufrufe dynamisch zu handhaben.

Super, du hast es geschafft!

## Aufgabe

Nimm den Code aus der Übung und baue den Server mit weiteren Tools aus. Erstelle dann einen Client mit einem LLM, so wie in der Übung, und teste ihn mit verschiedenen Eingaben, um sicherzustellen, dass alle Server-Tools dynamisch aufgerufen werden. Diese Art, einen Client zu bauen, sorgt für eine großartige Nutzererfahrung, da der Endanwender Eingaben verwenden kann, anstatt exakte Client-Befehle eingeben zu müssen, und davon nichts mitbekommt, dass ein MCP-Server aufgerufen wird.

## Lösung

[Lösung](/03-GettingStarted/03-llm-client/solution/README.md)

## Wichtige Erkenntnisse

- Die Integration eines LLM in deinen Client ermöglicht eine bessere Nutzerinteraktion mit MCP-Servern.
- Du musst die Antwort des MCP-Servers in etwas umwandeln, das das LLM verstehen kann.

## Beispiele

- [Java Rechner](../samples/java/calculator/README.md)
- [.Net Rechner](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Rechner](../samples/javascript/README.md)
- [TypeScript Rechner](../samples/typescript/README.md)
- [Python Rechner](../../../../03-GettingStarted/samples/python)
- [Rust Rechner](../../../../03-GettingStarted/samples/rust)

## Zusätzliche Ressourcen

## Was kommt als Nächstes

- Nächstes: [Server mit Visual Studio Code konsumieren](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:  
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner ursprünglichen Sprache gilt als maßgebliche Quelle. Für wichtige Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die durch die Nutzung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
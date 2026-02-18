# Oprettelse af en klient med LLM

Indtil videre har du set, hvordan man opretter en server og en klient. Klienten har kunnet kalde serveren eksplicit for at liste dens værktøjer, ressourcer og prompts. Det er dog ikke en særlig praktisk tilgang. Din bruger lever i den agentiske æra og forventer at bruge prompts og kommunikere med en LLM for at gøre det. For din bruger er det ligegyldigt, om du bruger MCP eller ej til at gemme dine kapaciteter, men de forventer at bruge naturligt sprog til at interagere. Hvordan løser vi dette? Løsningen er at tilføje en LLM til klienten.

## Oversigt

I denne lektion fokuserer vi på at tilføje en LLM til din klient og vise, hvordan dette giver en meget bedre oplevelse for din bruger.

## Læringsmål

Når denne lektion er slut, vil du kunne:

- Oprette en klient med en LLM.
- Sømløst interagere med en MCP-server ved hjælp af en LLM.
- Give en bedre slutbrugeroplevelse på klientsiden.

## Fremgangsmåde

Lad os prøve at forstå den fremgangsmåde, vi skal tage. At tilføje en LLM lyder simpelt, men vil vi faktisk gøre det?

Sådan interagerer klienten med serveren:

1. Etablér forbindelse til serveren.

1. List kapaciteter, prompts, ressourcer og værktøjer og gem deres skema.

1. Tilføj en LLM og videregiv de gemte kapaciteter og deres skema i et format, som LLM'en forstår.

1. Håndtér en brugerprompt ved at sende den til LLM sammen med de værktøjer, som klienten har listet.

Fint, nu hvor vi forstår, hvordan vi kan gøre dette på et overordnet niveau, så lad os prøve det i nedenstående øvelse.

## Øvelse: Oprettelse af en klient med en LLM

I denne øvelse lærer vi at tilføje en LLM til vores klient.

### Autentificering ved hjælp af GitHub Personal Access Token

At oprette et GitHub-token er en ligetil proces. Sådan gør du:

- Gå til GitHub Settings – Klik på dit profilbillede i øverste højre hjørne og vælg Settings.
- Navigér til Developer Settings – Scroll ned og klik på Developer Settings.
- Vælg Personal Access Tokens – Klik på Fine-grained tokens og derefter Generate new token.
- Konfigurer dit token – Tilføj en note til reference, sæt en udløbsdato, og vælg de nødvendige scopes (tilladelser). I dette tilfælde skal du sikre dig at tilføje Models-tilladelsen.
- Generér og kopier token – Klik på Generate token, og sørg for at kopiere det med det samme, da du ikke kan se det igen.

### -1- Opret forbindelse til server

Lad os først oprette vores klient:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Importer zod til skemavalidering

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

I den foregående kode har vi:

- Importeret de nødvendige biblioteker.
- Oprettet en klasse med to medlemmer, `client` og `openai`, som hjælper os med at administrere en klient og interagere med en LLM henholdsvis.
- Konfigureret vores LLM-instans til at bruge GitHub Models ved at sætte `baseUrl` til at pege på inference API'en.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Opret serverparametre til stdio-forbindelse
server_params = StdioServerParameters(
    command="mcp",  # Udførbar fil
    args=["run", "server.py"],  # Valgfrie kommandolinjeparametre
    env=None,  # Valgfrie miljøvariabler
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # Initialiser forbindelsen
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

I den foregående kode har vi:

- Importeret de nødvendige biblioteker til MCP.
- Oprettet en klient.

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

Først skal du tilføje LangChain4j-afhængighederne til din `pom.xml`-fil. Tilføj disse afhængigheder for at aktivere MCP-integration og support for GitHub Models:

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

Opret derefter din Java-klientklasse:

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
    
    public static void main(String[] args) throws Exception {        // Konfigurer LLM til at bruge GitHub-modeller
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Opret MCP-transport til forbindelse til server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // Opret MCP-klient
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

I den foregående kode har vi:

- **Tilføjet LangChain4j-afhængigheder**: Påkrævet for MCP-integration, OpenAI officiel klient og support for GitHub Models.
- **Importeret LangChain4j-biblioteker**: Til MCP-integration og OpenAI chat model-funktionalitet.
- **Oprettet en `ChatLanguageModel`**: Konfigureret til at bruge GitHub Models med dit GitHub-token.
- **Opsat HTTP-transport**: Bruger Server-Sent Events (SSE) til at forbinde til MCP-serveren.
- **Oprettet en MCP-klient**: Som håndterer kommunikation med serveren.
- **Brugt LangChain4j's indbyggede MCP-support**: Som forenkler integrationen mellem LLM'er og MCP-servere.

#### Rust

Dette eksempel antager, at du har en Rust-baseret MCP-server kørende. Hvis du ikke har en, kan du gå tilbage til [01-first-server](../01-first-server/README.md) lektionen for at oprette serveren.

Når du har din Rust MCP-server, skal du åbne en terminal og navigere til samme mappe som serveren. Kør derefter følgende kommando for at oprette et nyt LLM-klientprojekt:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Tilføj følgende afhængigheder til din `Cargo.toml`-fil:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Der findes ikke et officielt Rust-bibliotek til OpenAI, men `async-openai`-craten er et [fællesskabsvedligeholdt bibliotek](https://platform.openai.com/docs/libraries/rust#rust), som ofte bruges.

Åbn `src/main.rs`-filen og erstat dens indhold med følgende kode:

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
    // Startbesked
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // Opsæt OpenAI-klient
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // Opsæt MCP-klient
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

    // TODO: Hent MCP værktøjsliste

    // TODO: LLM samtale med værktøjsopkald

    Ok(())
}
```

Denne kode opsætter en grundlæggende Rust-applikation, der vil forbinde til en MCP-server og GitHub Models for LLM-interaktioner.

> [!IMPORTANT]
> Sørg for at sætte miljøvariablen `OPENAI_API_KEY` med dit GitHub-token, inden du kører applikationen.

Fint, til næste trin skal vi liste kapaciteterne på serveren.

### -2- Liste serverkapaciteter

Nu opretter vi forbindelse til serveren og spørger efter dens kapaciteter:

#### Typescript

I samme klasse tilføjer du følgende metoder:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // opregning af værktøjer
    const toolsResult = await this.client.listTools();
}
```

I den foregående kode har vi:

- Tilføjet kode til at oprette forbindelse til serveren, `connectToServer`.
- Oprettet en `run` metode, der håndterer appens flow. Indtil videre lister den kun værktøjerne, men vi vil snart tilføje mere.

#### Python

```python
# Liste over tilgængelige ressourcer
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Liste over tilgængelige værktøjer
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Her er hvad vi tilføjede:

- Liste ressourcer og værktøjer og udskrev dem. For værktøjer listede vi også `inputSchema`, som vi bruger senere.

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

I den foregående kode har vi:

- Listet de værktøjer, der er tilgængelige på MCP-serveren.
- For hvert værktøj listet navn, beskrivelse og dets skema. Sidstnævnte skal vi bruge til snart at kalde værktøjerne.

#### Java

```java
// Opret en værktøjsudbyder, der automatisk opdager MCP-værktøjer
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP-værktøjsudbyderen håndterer automatisk:
// - Oplistning af tilgængelige værktøjer fra MCP-serveren
// - Konvertering af MCP-værktøjs-skemaer til LangChain4j-format
// - Håndtering af værktøjseksekvering og svar
```

I den foregående kode har vi:

- Oprettet en `McpToolProvider`, som automatisk opdager og registrerer alle værktøjer fra MCP-serveren.
- Tool provideren håndterer internt konverteringen mellem MCP tools skemaer og LangChain4j’s tool format.
- Denne tilgang abstraherer den manuelle proces med listings og konvertering af værktøjer.

#### Rust

At hente værktøjer fra MCP-serveren gøres med metoden `list_tools`. Tilføj følgende kode i din `main`-funktion, efter at MCP-klienten er opsat:

```rust
// Get MCP tool listing
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Konverter serverkapaciteter til LLM-værktøjer

Næste skridt efter at have listet serverens kapaciteter er at konvertere dem til et format, som LLM'en forstår. Når det er gjort, kan vi give disse kapaciteter videre som værktøjer til vores LLM.

#### TypeScript

1. Tilføj følgende kode til at konvertere responsen fra MCP-serveren til et værktøjsformat, som LLM kan bruge:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Opret et zod-skema baseret på input_schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Angiv eksplicit typen til "function"
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

    Ovenstående kode tager et svar fra MCP-serveren og konverterer det til et værktøjsdefinitionsformat, som LLM kan forstå.

1. Lad os opdatere `run`-metoden for at liste serverkapaciteter:

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

    I den foregående kode har vi opdateret `run`-metoden til at iterere over resultatet og for hver post kaldt `openAiToolAdapter`.

#### Python

1. Først laver vi denne konverteringsfunktion:

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

    I funktionen `convert_to_llm_tools` tager vi et MCP tool-svar og konverterer det til et format, som LLM kan forstå.

1. Dernæst opdaterer vi vores klientkode til at bruge denne funktion således:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Her tilføjer vi et kald til `convert_to_llm_tool` for at konvertere MCP tool-svaret til noget, vi kan give til LLM senere.

#### .NET

1. Lad os tilføje kode for at konvertere MCP tool-svaret til noget, LLM kan forstå:

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

I den foregående kode har vi:

- Oprettet en funktion `ConvertFrom`, der tager navn, beskrivelse og input-skema.
- Defineret funktionalitet, som opretter en FunctionDefinition, der sendes til en ChatCompletionsDefinition. Sidstnævnte kan LLM forstå.

1. Lad os se, hvordan vi kan opdatere noget eksisterende kode til at bruge funktionen ovenfor:

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
// Opret en botgrænseflade til naturlig sproginteraktion
public interface Bot {
    String chat(String prompt);
}

// Konfigurer AI-tjenesten med LLM og MCP værktøjer
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

I den foregående kode har vi:

- Defineret et simpelt `Bot` interface for interaktion med naturligt sprog.
- Brugte LangChain4j’s `AiServices` til automatisk at binde LLM med MCP tool provideren.
- Frameworket håndterer automatisk konvertering af tool skemaer og kald af funktioner bag kulisserne.
- Denne tilgang eliminerer manuel værktøjskonvertering – LangChain4j håndterer al kompleksiteten i at konvertere MCP værktøjer til LLM-kompatibelt format.

#### Rust

For at konvertere MCP tool-svaret til et format, som LLM kan forstå, tilføjer vi en hjælpefunktion, der formaterer tools-listingen. Tilføj følgende kode til din `main.rs` fil under `main` funktionen. Den kaldes, når der laves forespørgsler til LLM:

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

Flot, nu er vi sat op til at håndtere brugerforespørgsler, så lad os tage fat på det næste.

### -4- Håndtér brugerprompt-forespørgsel

I denne del af koden vil vi håndtere brugerforespørgsler.

#### TypeScript

1. Tilføj en metode, der skal bruges til at kalde vores LLM:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Kald serverens værktøj
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Gør noget med resultatet
        // TO-DO

        }
    }
    ```

    I den foregående kode har vi:

    - Tilføjet en metode `callTools`.
    - Metoden tager et LLM-respons og tjekker, hvilke værktøjer der er kaldt, hvis nogen:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // kald værktøj
        }
        ```

    - Kalder et værktøj, hvis LLM indikerer, at det skal kaldes:

        ```typescript
        // 2. Kald serverens værktøj
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Gør noget med resultatet
        // TODO
        ```

1. Opdater `run`-metoden til at inkludere kald til LLM og `callTools`:

    ```typescript

    // 1. Opret meddelelser, der er input til LLM'en
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. Kalder LLM'en
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Gennemgå LLM-svaret, for hver valgmulighed, kontroller om det har værktøjskald
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Fint, lad os liste koden i sin fulde form:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Importer zod til skemavalidering

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // måske skal ændres til denne url i fremtiden: https://models.github.ai/inference
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
          // Opret et zod-skema baseret på input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Angiv eksplicit typen til "funktion"
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
    
    
          // 2. Kald serverens værktøj
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Gør noget med resultatet
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
    
        // 1. Gennemgå LLM-responsen, for hver valgmulighed, tjek om den har værktøjsopkald
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

1. Lad os tilføje nogle nødvendige imports til at kalde en LLM:

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Dernæst tilføjer vi funktionen, som kalder LLM:

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
            # Valgfrie parametre
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

    I den foregående kode har vi:

    - Videregivet vores funktioner, som vi fandt på MCP-serveren og konverterede, til LLM.
    - Kaldt LLM med disse funktioner.
    - Derefter undersøgt resultatet for at se, hvilke funktioner vi skal kalde, hvis nogen.
    - Til sidst sender vi et array af funktioner, der skal kaldes.

1. Endeligt opdaterer vi vores hovedkode:

    ```python
    prompt = "Add 2 to 20"

    # spørg LLM hvilke værktøjer, hvis nogen, der skal bruges
    functions_to_call = call_llm(prompt, functions)

    # kald foreslåede funktioner
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Det var det sidste trin, i koden ovenfor:

    - Kalder et MCP værktøj via `call_tool` ved hjælp af en funktion, som LLM mente vi skulle kalde baseret på prompten.
    - Printer resultatet af værktøjskaldet til MCP-serveren.

#### .NET

1. Lad os vise noget kode til at lave en LLM-promptforespørgsel:

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

    I den foregående kode har vi:

    - Hentet værktøjer fra MCP-serveren, `var tools = await GetMcpTools()`.
    - Defineret en bruger-prompt `userMessage`.
    - Konstrueret et options-objekt med model og værktøjer.
    - Lavet en forespørgsel mod LLM.

1. Ét sidste skridt, lad os se, om LLM mener, vi skal kalde en funktion:

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

    I den foregående kode har vi:

    - Gennemgået en liste af funktionskald.
    - For hvert værktøjskald, hentet navn og argumenter og kaldt værktøjet på MCP-serveren via MCP-klienten. Til sidst printer vi resultaterne.

Her er koden i sin helhed:

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
    // Udfør forespørgsler i naturligt sprog, der automatisk bruger MCP-værktøjer
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

I den foregående kode har vi:

- Brug simple naturlige sprogprompts til at interagere med MCP server-værktøjer.
- LangChain4j frameworket håndterer automatisk:
  - Konvertering af brugerprompts til værktøjskald når nødvendigt.
  - Kald til de passende MCP værktøjer baseret på LLM’s beslutning.
  - Styring af samtaleflowet mellem LLM og MCP server.
- `bot.chat()` metoden returnerer naturlige sprog-respons, som kan inkludere resultater fra MCP værktøjseksekveringer.
- Denne tilgang giver en sømløs brugeroplevelse, hvor brugerne ikke behøver at kende til den underliggende MCP-implementering.

Komplet kodeeksempel:

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

Her sker hovedparten af arbejdet. Vi kalder LLM med den initiale brugerprompt, behandler responsen for at se, om der skal kaldes nogle værktøjer. Hvis det er tilfældet, kalder vi de værktøjer og fortsætter samtalen med LLM, indtil der ikke er flere værktøjskald, og vi har et endeligt svar.

Vi vil lave flere kald til LLM, så lad os definere en funktion, der håndterer LLM-kald. Tilføj følgende funktion til din `main.rs` fil:

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

Funktionen tager LLM-klienten, en liste af beskeder (inklusive brugerprompt), værktøjerne fra MCP-serveren, og sender en forespørgsel til LLM, som returnerer svaret.
Responsen fra LLM vil indeholde et array af `choices`. Vi skal behandle resultatet for at se, om der er nogen `tool_calls`. Dette lader os vide, at LLM anmoder om, at et specifikt værktøj skal kaldes med argumenter. Tilføj følgende kode nederst i din `main.rs`-fil for at definere en funktion til at håndtere LLM-responsen:

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

    // Udskriv indhold, hvis tilgængeligt
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Håndter værktøjskald
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Tilføj assistentbesked

        // Udfør hvert værktøjskald
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Tilføj værktøjsresultat til beskeder
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Fortsæt samtale med værktøjsresultater
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

Hvis `tool_calls` er til stede, udtrækker den værktøjsinformationen, kalder MCP-serveren med værktøjsanmodningen og tilføjer resultaterne til samtalens beskeder. Den fortsætter derefter samtalen med LLM, og beskederne opdateres med assistentens svar og resultaterne fra værktøjskaldet.

For at udtrække værktøjskaldsinformation, som LLM returnerer til MCP-kald, tilføjer vi en anden hjælpefunktion til at udtrække alt, der er nødvendigt for at foretage kaldet. Tilføj følgende kode nederst i din `main.rs`-fil:

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

Med alle delene på plads kan vi nu håndtere den indledende brugerprompt og kalde LLM. Opdater din `main`-funktion til at inkludere følgende kode:

```rust
// LLM samtale med værktøjkald
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

Dette vil forespørge LLM med den indledende brugerprompt, der beder om summen af to tal, og det vil behandle responsen for dynamisk at håndtere værktøjskald.

Fremragende, du klarede det!

## Opgave

Tag koden fra øvelsen og byg serveren ud med flere værktøjer. Opret derefter en klient med en LLM, som i øvelsen, og test den med forskellige prompts for at sikre, at alle dine serverværktøjer kaldes dynamisk. Denne måde at bygge en klient på betyder, at slutbrugeren får en fremragende brugeroplevelse, da de kan bruge prompts i stedet for præcise klientkommandoer og være uvidende om eventuelle MCP-serveropkald.

## Løsning

[Løsning](/03-GettingStarted/03-llm-client/solution/README.md)

## Vigtige pointer

- Tilføjelse af en LLM til din klient giver en bedre måde for brugere at interagere med MCP-servere.
- Du skal konvertere MCP-serverens respons til noget, LLM kan forstå.

## Eksempler

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## Yderligere ressourcer

## Hvad er det næste

- Næste: [Forbrug af en server ved hjælp af Visual Studio Code](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, bedes du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritiske oplysninger anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for eventuelle misforståelser eller fejltolkninger, som måtte opstå som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
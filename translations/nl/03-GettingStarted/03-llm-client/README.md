# Een client maken met LLM

Tot nu toe heb je gezien hoe je een server en een client maakt. De client kon expliciet de server aanroepen om de beschikbare tools, resources en prompts te tonen. Dit is echter niet een heel praktische aanpak. Je gebruiker leeft in het agentische tijdperk en verwacht prompts te gebruiken en met een LLM te communiceren om dat te doen. Voor je gebruiker maakt het niet uit of je MCP gebruikt of niet om je mogelijkheden op te slaan, maar men verwacht wel natuurlijke taal te gebruiken voor interactie. Hoe lossen we dit op? De oplossing is het toevoegen van een LLM aan de client.

## Overzicht

In deze les focussen we op het toevoegen van een LLM aan je client en laten we zien hoe dit zorgt voor een veel betere ervaring voor je gebruiker.

## Leerdoelen

Aan het einde van deze les kun je:

- Een client maken met een LLM.
- Naadloos interactie hebben met een MCP-server via een LLM.
- Een betere eindgebruikerservaring bieden aan de clientzijde.

## Aanpak

Laten we proberen de aanpak te begrijpen die we moeten volgen. Een LLM toevoegen klinkt eenvoudig, maar gaan we dit echt zo doen?

Zo zal de client met de server communiceren:

1. Maak verbinding met de server.

1. Vraag mogelijkheden, prompts, resources en tools op en sla het schema ervan op.

1. Voeg een LLM toe en geef de opgeslagen mogelijkheden en schema’s door in een formaat dat de LLM begrijpt.

1. Verwerk een gebruikersprompt door deze samen met de door de client opgelijste tools aan de LLM te geven.

Geweldig, nu we op hoog niveau begrijpen hoe we dit kunnen doen, laten we dit eens uitproberen in onderstaande oefening.

## Oefening: Een client maken met een LLM

In deze oefening leren we een LLM toe te voegen aan onze client.

### Authenticatie via GitHub Personal Access Token

Een GitHub-token aanmaken is een eenvoudig proces. Zo doe je dat:

- Ga naar GitHub-instellingen – Klik op je profielfoto rechtsboven en kies Instellingen.
- Navigeer naar Developer Settings – Scroll naar beneden en klik op Developer Settings.
- Selecteer Personal Access Tokens – Klik op Fine-grained tokens en vervolgens op Generate new token.
- Configureer je token – Voeg een notitie toe ter referentie, stel een vervaldatum in en selecteer de benodigde scopes (toestemmingen). Voeg in dit geval de Models-permissie toe.
- Genereer en kopieer de token – Klik op Generate token en zorg dat je hem meteen kopieert, want je kunt hem later niet meer zien.

### -1- Verbinding maken met de server

Laten we eerst onze client maken:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Importeer zod voor schema-validatie

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

In bovenstaande code hebben we:

- De benodigde libraries geïmporteerd.
- Een klasse gemaakt met twee leden, `client` en `openai`, die ons respectievelijk helpen bij het beheren van een client en interactie met een LLM.
- Onze LLM instantie geconfigureerd om GitHub Models te gebruiken door `baseUrl` te zetten naar de inference API.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Maak serverparameters voor stdio-verbinding
server_params = StdioServerParameters(
    command="mcp",  # Uitvoerbaar bestand
    args=["run", "server.py"],  # Optionele opdrachtregelargumenten
    env=None,  # Optionele omgevingsvariabelen
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # Initialiseer de verbinding
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

In bovenstaande code hebben we:

- De benodigde libraries voor MCP geïmporteerd.
- Een client aangemaakt.

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

Eerst moet je de LangChain4j dependencies toevoegen aan je `pom.xml` bestand. Voeg deze dependencies toe om MCP-integratie en GitHub Models ondersteuning mogelijk te maken:

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

Maak vervolgens je Java clientklasse:

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
    
    public static void main(String[] args) throws Exception {        // Configureer de LLM om GitHub-modellen te gebruiken
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Maak MCP-transport aan voor verbinding met de server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // Maak MCP-client aan
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

In bovenstaande code hebben we:

- **LangChain4j dependencies toegevoegd**: Vereist voor MCP-integratie, officiële OpenAI-client en GitHub Models ondersteuning.
- **LangChain4j libraries geïmporteerd**: Voor MCP integratie en OpenAI chatmodel functionaliteit.
- **Een `ChatLanguageModel` gemaakt**: Geconfigureerd om GitHub Models te gebruiken met je GitHub-token.
- **HTTP-transport opgezet**: Met Server-Sent Events (SSE) om verbinding te maken met de MCP-server.
- **Een MCP-client gemaakt**: Die de communicatie met de server afhandelt.
- **LangChain4j's ingebouwde MCP-ondersteuning gebruikt**: Wat integratie tussen LLMs en MCP-servers vereenvoudigt.

#### Rust

Dit voorbeeld gaat ervan uit dat je een Rust-gebaseerde MCP-server hebt draaien. Als je die niet hebt, zie de [01-first-server](../01-first-server/README.md) les om de server aan te maken.

Als je je Rust MCP-server hebt, open dan een terminal en ga naar dezelfde directory als de server. Voer vervolgens de volgende opdracht uit om een nieuw LLM clientproject aan te maken:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Voeg de volgende dependencies toe aan je `Cargo.toml` bestand:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Er is geen officiële Rust library voor OpenAI, maar de `async-openai` crate is een [door de community onderhouden library](https://platform.openai.com/docs/libraries/rust#rust) die veel gebruikt wordt.

Open het bestand `src/main.rs` en vervang de inhoud door de volgende code:

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
    // Initiële bericht
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI-client instellen
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP-client instellen
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

    // TODO: Verkrijg MCP gereedschapslijst

    // TODO: LLM conversatie met gereedschapsoproepen

    Ok(())
}
```

Deze code zet een eenvoudige Rust applicatie op die verbinding maakt met een MCP-server en GitHub Models voor LLM-interacties.

> [!IMPORTANT]
> Zorg ervoor dat je de omgevingsvariabele `OPENAI_API_KEY` instelt met je GitHub-token voordat je de applicatie draait.

Geweldig, voor de volgende stap gaan we de mogelijkheden op de server opvragen.

### -2- Mogelijkheden van de server opvragen

Nu maken we verbinding met de server en vragen we de mogelijkheden op:

#### Typescript

Voeg in dezelfde klasse de volgende methoden toe:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // opsomming van gereedschappen
    const toolsResult = await this.client.listTools();
}
```

In bovenstaande code hebben we:

- Code toegevoegd om verbinding te maken met de server, `connectToServer`.
- Een `run` methode gemaakt die verantwoordelijk is voor de flow van de app. Tot nu toe lijst het alleen de tools, maar we voegen binnenkort meer toe.

#### Python

```python
# Lijst beschikbare bronnen
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Lijst beschikbare hulpmiddelen
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Dit hebben we toegevoegd:

- Resources en tools lijst opgehaald en afgedrukt. Voor tools vermelden we ook de `inputSchema` die we later gebruiken.

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

In bovenstaande code hebben we:

- De beschikbare tools op de MCP-server opgehaald.
- Voor elke tool de naam, beschrijving en het schema opgehaald. Dit laatste gebruiken we straks om tools aan te roepen.

#### Java

```java
// Maak een toolprovider die automatisch MCP-tools ontdekt
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// De MCP-toolprovider handelt automatisch af:
// - Beschikbare tools van de MCP-server weergeven
// - MCP-toolschema's converteren naar LangChain4j-formaat
// - Beheer van tooluitvoering en responsen
```

In bovenstaande code hebben we:

- Een `McpToolProvider` gemaakt die automatisch alle tools van de MCP-server ontdekt en registreert.
- De toolprovider handelt intern de conversie af van MCP tool schema’s naar het tool formaat van LangChain4j.
- Deze aanpak neemt het handmatig lijst en conversieproces weg.

#### Rust

Tools ophalen van de MCP-server gebeurt met de `list_tools` methode. Voeg in je `main` functie, nadat je de MCP client hebt opgezet, de volgende code toe:

```rust
// Verkrijg MCP-gereedschapslijst
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Servermogelijkheden omzetten naar LLM-tools

De volgende stap na het lijst maken van servermogelijkheden is deze omzetten naar een formaat dat de LLM begrijpt. Als dat gedaan is, kunnen we deze mogelijkheden als tools aan de LLM geven.

#### TypeScript

1. Voeg de volgende code toe om het antwoord van de MCP-server om te zetten naar een toolformaat dat de LLM kan gebruiken:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Maak een zod-schema op basis van het input_schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Stel expliciet het type in op "functie"
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

    De bovenstaande code neemt een antwoord van de MCP-server en zet dit om naar een tooldefinitie in een formaat dat de LLM begrijpt.

1. Laten we hierna de `run` methode aanpassen om servermogelijkheden te tonen:

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

    In bovenstaande code hebben we de `run` methode aangepast zodat hij over het resultaat loope en voor elke invoer `openAiToolAdapter` aanroept.

#### Python

1. Eerst maken we de volgende converterfunctie:

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

    In de functie `convert_to_llm_tools` hierboven nemen we een MCP-tool response en zetten deze om naar een formaat dat de LLM begrijpt.

1. Vervolgens passen we de clientcode aan om deze functie te gebruiken:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Hier voegen we een aanroep toe aan `convert_to_llm_tool` om het MCP tool response om te zetten naar iets dat we later aan de LLM kunnen doorgeven.

#### .NET

1. Laten we code toevoegen die de MCP toolresponse omzet naar iets dat de LLM begrijpt:

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

In bovenstaande code hebben we:

- Een functie `ConvertFrom` gemaakt die naam, beschrijving en input schema ontvangt.
- Functionaliteit gedefinieerd die een `FunctionDefinition` maakt die wordt doorgegeven aan een `ChatCompletionsDefinition`. Dit laatste begrijpt de LLM.

1. Zo kunnen we bestaande code aanpassen om deze functie te gebruiken:

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
// Maak een Bot-interface voor natuurlijke taalinteractie
public interface Bot {
    String chat(String prompt);
}

// Configureer de AI-service met LLM- en MCP-hulpmiddelen
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

In bovenstaande code hebben we:

- Een eenvoudige `Bot` interface gedefinieerd voor natuurlijke taal interacties.
- LangChain4j's `AiServices` gebruikt om de LLM automatisch te binden aan de MCP tool provider.
- Het framework regelt automatisch tool schema conversie en functie-aanroepen op de achtergrond.
- Deze aanpak elimineert handmatige tool conversie – LangChain4j regelt alle complexiteit van het omzetten van MCP tools naar een LLM-compatibel formaat.

#### Rust

Om het MCP toolresponse om te zetten naar een formaat dat de LLM begrijpt, voegen we een hulpfunctie toe die de toolslijst formatteert. Voeg de volgende code aan je `main.rs` bestand toe onder de `main` functie. Deze wordt aangeroepen bij verzoeken naar de LLM:

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

Geweldig, we zijn nu klaar om gebruikersaanvragen te verwerken, die pakken we bij de volgende stap aan.

### -4- Gebruikersprompt afhandelen

In dit deel van de code verwerken we gebruikersaanvragen.

#### TypeScript

1. Voeg een methode toe die gebruikt wordt om onze LLM aan te roepen:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Roep de tool van de server aan
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Doe iets met het resultaat
        // NOG TE DOEN

        }
    }
    ```

    In bovenstaande code:

    - Toegevoegd een methode `callTools`.
    - De methode neemt een LLM-respons en kijkt welke tools eventueel aangeroepen moeten worden:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // gereedschap aanroepen
        }
        ```

    - Roept een tool aan, als de LLM aangeeft dat dit moet:

        ```typescript
        // 2. Roep de tool van de server aan
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Doe iets met het resultaat
        // TODO
        ```

1. Pas de `run` methode aan zodat deze calls naar de LLM en `callTools` bevat:

    ```typescript

    // 1. Maak berichten die invoer zijn voor de LLM
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. Het aanroepen van de LLM
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Ga door de LLM-respons, controleer voor elke keuze of deze tool-aanroepen bevat
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Geweldig, hier is de volledige code:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Importeer zod voor schema validatie

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // mogelijk in de toekomst deze url moeten wijzigen: https://models.github.ai/inference
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
          // Maak een zod schema gebaseerd op het input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Stel expliciet het type in op "function"
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
    
    
          // 2. Roep de tool van de server aan
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Doe iets met het resultaat
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
    
        // 1. Ga door de LLM-respons, controleer voor elke keuze of er tool-aanroepen zijn
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

1. Voeg imports toe die nodig zijn om een LLM aan te roepen:

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Voeg daarna de functie toe die de LLM aanroept:

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
            # Optionele parameters
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

    In bovenstaande code hebben we:

    - Onze functies, gevonden op de MCP-server en geconverteerd, doorgegeven aan de LLM.
    - Vervolgens de LLM aangeroepen met die functies.
    - Daarna het resultaat bekeken om te zien welke functies we eventueel moeten aanroepen.
    - Tenslotte een lijst functies om aan te roepen doorgegeven.

1. Laatste stap, pas onze hoofdcode aan:

    ```python
    prompt = "Add 2 to 20"

    # vraag aan LLM welke hulpmiddelen, indien aanwezig, moeten worden gebruikt
    functions_to_call = call_llm(prompt, functions)

    # roep voorgestelde functies aan
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Dat was de laatste stap, in bovenstaande code:

    - Roept een MCP tool aan via `call_tool` met de functie die de LLM heeft gekozen op basis van onze prompt.
    - Drukt het resultaat van de toolcall van de MCP server af.

#### .NET

1. Hier is code voor het doen van een LLM-prompt aanvraag:

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

    In bovenstaande code hebben we:

    - Tools van de MCP server opgehaald, `var tools = await GetMcpTools()`.
    - Een gebruikersprompt gedefinieerd `userMessage`.
    - Een opties object gemaakt met model en tools.
    - Een verzoek naar de LLM gedaan.

1. Nog een laatste stap, kijken of de LLM denkt dat we een functie moeten aanroepen:

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

    In bovenstaande code hebben we:

    - Over een lijst van functie-aanroepen geloopt.
    - Voor elke tool-aanroep de naam en argumenten uitgepakt en de tool op de MCP server aangeroepen met de MCP client. Tenslotte de resultaten afgedrukt.

Hier is de complete code:

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
    // Voer verzoeken in natuurlijke taal uit die automatisch MCP-hulpmiddelen gebruiken
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

In bovenstaande code hebben we:

- Eenvoudige natuurlijke taal prompts gebruikt om met de MCP server tools te interacteren.
- Het LangChain4j framework handelt automatisch af:
  - Het omzetten van gebruikersprompts naar tool-aanroepen waar nodig.
  - Het aanroepen van de juiste MCP tools op basis van de beslissing van de LLM.
  - Het beheren van het gesprek tussen LLM en MCP server.
- De `bot.chat()` methode geeft natuurlijke taal antwoorden terug die resultaten kunnen bevatten van MCP tool uitvoeringen.
- Deze aanpak biedt een naadloze gebruikerservaring waar gebruikers niet hoeven te weten over de onderliggende MCP-implementatie.

Volledig codevoorbeeld:

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

Dit is het deel waar het grootste deel van het werk plaatsvindt. We roepen de LLM aan met de initiële gebruikersprompt, verwerken het antwoord om te zien of er tools aangeroepen moeten worden. Zo ja, dan roepen we die tools aan en zetten het gesprek met de LLM voort totdat er geen tool-aanroepen meer nodig zijn en we een definitief antwoord hebben.

We doen meerdere oproepen aan de LLM, laten we een functie definiëren die de LLM-aanroep afhandelt. Voeg de volgende functie toe aan je `main.rs` bestand:

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

Deze functie neemt de LLM client, een lijst berichten (inclusief de gebruikersprompt), tools van de MCP-server, en stuurt een verzoek naar de LLM terug, waarbij het antwoord wordt geretourneerd.
De reactie van de LLM bevat een array van `choices`. We moeten het resultaat verwerken om te zien of er `tool_calls` aanwezig zijn. Dit laat ons weten dat de LLM vraagt om een specifieke tool aan te roepen met argumenten. Voeg de volgende code onderaan je `main.rs` bestand toe om een functie te definiëren die de LLM-reactie afhandelt:

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

    // Inhoud afdrukken indien beschikbaar
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Omgaan met tool-aanroepen
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Assistentboodschap toevoegen

        // Voer elke tool-aanroep uit
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Voeg toolresultaat toe aan berichten
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Gesprek voortzetten met toolresultaten
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

Als er `tool_calls` aanwezig zijn, haalt het de toolinformatie op, roept het MCP-server aan met het toolverzoek, en voegt de resultaten toe aan de conversatieberichten. Daarna zet het de conversatie voort met de LLM en worden de berichten bijgewerkt met de reactie van de assistent en de resultaten van de toolaanroep.

Om toolaanroepinformatie te extraheren die de LLM teruggeeft voor MCP-aanroepen, voegen we een andere hulpfunctie toe om alles te halen wat nodig is om de aanroep te doen. Voeg de volgende code onderaan je `main.rs` bestand toe:

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

Met alle onderdelen op hun plaats kunnen we nu de initiële gebruikersprompt afhandelen en de LLM aanroepen. Werk je `main` functie bij met de volgende code:

```rust
// LLM-gesprek met gereedschapsoproepen
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

Dit zal de LLM raadplegen met de initiële gebruikersprompt die vraagt om de som van twee getallen, en het zal de reactie verwerken om dynamisch toolaanroepen af te handelen.

Geweldig, je hebt het gedaan!

## Opdracht

Neem de code uit de oefening en bouw de server uit met meer tools. Maak daarna een client met een LLM, zoals in de oefening, en test het met verschillende prompts om zeker te zijn dat al je servertolls dynamisch worden aangeroepen. Deze manier van een client bouwen betekent dat de eindgebruiker een geweldige gebruikerservaring zal hebben omdat ze prompts kunnen gebruiken in plaats van exacte clientcommando’s, en onbewust zijn van enige MCP-serveraanroepen.

## Oplossing

[Oplossing](/03-GettingStarted/03-llm-client/solution/README.md)

## Belangrijkste Leerpunten

- Het toevoegen van een LLM aan je client biedt een betere manier voor gebruikers om met MCP-servers te communiceren.
- Je moet de MCP-serverreactie converteren naar iets wat de LLM kan begrijpen.

## Voorbeelden

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## Extra Bronnen

## Wat Nu

- Volgende: [Een server gebruiken met Visual Studio Code](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:  
Dit document is vertaald met behulp van de AI-vertalingsdienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat automatische vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet als de gezaghebbende bron worden beschouwd. Voor belangrijke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
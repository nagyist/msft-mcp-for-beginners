# Kliendi loomine LLM-iga

Siiani nägite, kuidas luua server ja klient. Klient on suutnud serverit otseselt kutsuda, et loetleda selle tööriistu, ressursse ja käivitusi. Kuid see pole eriti praktiline lähenemine. Teie kasutaja elab agendi ajastul ja ootab, et kasutaks käivitusi ja suhtleks LLM-iga selleks. Teie kasutaja jaoks pole oluline, kas kasutate MCP-d oma võimete salvestamiseks, kuid nad ootavad loomuliku keelega suhtlemist. Kuidas me siis selle lahendame? Lahendus seisneb LLM-i lisamises kliendile.

## Ülevaade

Selles õppetükis keskendume LLM-i lisamisele kliendile ja näitame, kuidas see annab teie kasutajale palju parema kogemuse.

## Õpieesmärgid

Selle õppetüki lõpuks oskate:

- Luua kliendi, millel on LLM.
- Sujuvalt suhelda MCP serveriga, kasutades LLM-i.
- Pakkuda paremat lõppkasutaja kogemust kliendi poolel.

## Lähenemine

Püüame mõista, kuidas me peame toimima. LLM-i lisamine kõlab lihtsana, kuid kas me tõesti teeme seda nii?

Siin on, kuidas klient suhtleb serveriga:

1. Välistada ühendus serveriga.

1. Loetleda võimed, käivitused, ressursid ja tööriistad ning salvestada nende skeem.

1. Lisada LLM ja anda salvestatud võimed ja nende skeem vormingus, mida LLM mõistab.

1. Käsitleda kasutaja käivitust, edastades selle LLM-ile koos kliendi poolt loetletud tööriistadega.

Suurepärane, nüüd mõistame, kuidas seda kõrgtasemel teha, proovime seda allpool olevas harjutuses.

## Harjutus: Kliendi loomine LLM-iga

Selles harjutuses õpime lisama LLM-i meie kliendile.

### Autentimine GitHubi isikliku juurdepääsu tokeniga

GitHubi tokeni loomine on lihtne protsess. Siin on, kuidas seda teha:

- Minge GitHubi seadistustesse – klõpsake paremas ülanurgas oma profiilipildil ja valige Seaded.
- Liikuge arendaja seadistustesse – kerige alla ja klõpsake Arendaja seaded.
- Valige Isikliku juurdepääsu tokenid – klõpsake Täpsustatud tokenitel ja seejärel Genereeri uus token.
- Seadistage oma token – lisage märge viitamiseks, määrake aegumiskuupäev ja valige vajalikud õigused (luba). Antud juhul lisage kindlasti Models'i luba.
- Genereerige ja kopeerige token – klõpsake Genereeri token, ja kindlasti kopeerige see kohe, sest rohkem seda näha ei saa.

### -1- Ühendamine serveriga

Loome esmalt oma kliendi:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Impordi zod skeemi valideerimiseks

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

Eelnevas koodis me:

- Impordisime vajalikud teegid.
- Loodud klass koos kahe liikmega, `client` ja `openai`, mis aitavad meil hallata klienti ja suhelda LLM-iga.
- Konfigureerisime oma LLM-i instantsi kasutamaks GitHubi Mudeleid, määrates `baseUrl` suunama inference API-le.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Loo serveri parameetrid stdio ühenduseks
server_params = StdioServerParameters(
    command="mcp",  # Käivitatav fail
    args=["run", "server.py"],  # Valikulised käsurea argumendid
    env=None,  # Valikulised keskkonnamuutujad
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # Algata ühendus
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

Eelnevas koodis me:

- Impordisime MCP jaoks vajalikud teegid.
- Loodud kliendi.

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

Esmalt peate oma `pom.xml` faili lisama LangChain4j sõltuvused. Lisage need sõltuvused MCP integreerimise ja GitHubi mudelite toe lubamiseks:

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

Seejärel looge oma Java kliendiklass:

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
    
    public static void main(String[] args) throws Exception {        // Konfigureeri LLM kasutama GitHubi mudeleid
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Loo MCP transpordikiht serveriga ühendamiseks
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // Loo MCP klient
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

Eelnevas koodis me:

- **Lisasime LangChain4j sõltuvused**: vajalikud MCP integreerimiseks, OpenAI ametliku kliendi ja GitHubi mudelite toeks.
- **Impordisime LangChain4j teegid**: MCP integratsiooni ja OpenAI vestlusmudeli funktsioonide jaoks.
- **Loomine `ChatLanguageModel`**: konfigureeritud kasutama GitHubi mudelit koos teie GitHubi tokeniga.
- **Seadisime üles HTTP transporti**: kasutades Server-Sent Events (SSE), et ühendada MCP serveriga.
- **Lõi MCP kliendi**: mis haldab suhtlust serveriga.
- **Kasutasime LangChain4j sisseehitatud MCP tuge**: mis lihtsustab LLM-ide ja MCP serverite integratsiooni.

#### Rust

See näide eeldab, et teil töötab Rust-i baasil MCP server. Kui teil seda pole, vaadake [01-first-server](../01-first-server/README.md) õppetükki, et server luua.

Kui teil on Rust MCP server käivitunud, avage terminal ja liikuge samasse kausta, kus server. Käivitage järgmine käsk CLI abil, et luua uus LLM kliendi projekt:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Lisage järgmised sõltuvused oma `Cargo.toml` faili:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Puudub ametlik Rust'i OpenAI teek, kuid `async-openai` on [kogukonna hooldatav teek](https://platform.openai.com/docs/libraries/rust#rust), mida sageli kasutatakse.

Avage `src/main.rs` fail ja asendage selle sisu alljärgneva koodiga:

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
    // Algne sõnum
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // Määra OpenAI klient
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // Määra MCP klient
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

    // TODO: Hangi MCP tööriistade nimekiri

    // TODO: LLM vestlus tööriistakõnedega

    Ok(())
}
```

See kood seab üles põhilise Rust rakenduse, mis ühendub MCP serveri ja GitHubi mudelitega LLM-i suhtluseks.

> [!IMPORTANT]
> Veenduge, et enne rakenduse käivitamist määrate keskkonnamuutujaks `OPENAI_API_KEY` oma GitHubi tokeni.

Suurepärane, järgmises sammus loetleme serveri võimed.

### -2- Loetle serveri võimed

Nüüd ühendume serveriga ja küsime selle võimeid:

#### TypeScript

Lisage samasse klassi järgmised meetodid:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // tööriistade loetelu
    const toolsResult = await this.client.listTools();
}
```

Eelnevas koodis me:

- Lisatud kood ühenduseks serveriga, `connectToServer`.
- Loodud `run` meetod, mis vastutab meie rakenduse voo eest. Seni loetles see vaid tööriistu, kuid varsti lisame sellele rohkem.

#### Python

```python
# Loetle saadaolevad ressursid
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Loetle saadaolevad tööriistad
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Siin on, mida lisasime:

- Loetlesime ressursid ja tööriistad ning printisime need. Tööriistade juures loetlesime ka `inputSchema`, mida kasutame hiljem.

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

Eelnevas koodis me:

- Loetlesime MCP serveril saadaolevad tööriistad.
- Iga tööriista kohta loetlesime nime, kirjelduse ja selle skeemi. Viimane on midagi, mida me varsti tööriistade kutsumiseks kasutame.

#### Java

```java
// Loo tööriistade pakkuja, mis avastab automaatselt MCP tööriistad
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP tööriistade pakkuja haldab automaatselt:
// - MCP serverist saadaolevate tööriistade nimekirja koostamine
// - MCP tööriista skeemide teisendamine LangChain4j formaati
// - Tööriistade käivitamise ja vastuste haldamine
```

Eelnevas koodis me:

- Loodud `McpToolProvider`, mis automaatselt avastab ja registreerib kõik MCP serveri tööriistad.
- Tööriistade pakkuja käsitleb MCP tööriistade skeemide ja LangChain4j tööriistade formaadi sisemist teisendust.
- See lähenemine vabastab käsitsi tööriistade loetlemise ja teisendamise protsessist.

#### Rust

Tööriistade saamine MCP serverist toimub `list_tools` meetodiga. Oma `main` funktsioonis MCP kliendi seadistamise järel lisage järgnev kood:

```rust
// Hangi MCP tööriistade nimekiri
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Konverteeri serveri võimed LLM tööriistadeks

Järgmine samm serveri võimete loetelust on nende teisendamine LLM-ile arusaadavasse vormingusse. Kui me seda teeme, võime pakkuda neid võimeid oma LLM-ile tööriistadena.

#### TypeScript

1. Lisage järgmine kood MCP serverilt saadud vastuse teisendamiseks LLM-i kasutatava tööriista formaadiks:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Loo zod skeem sisendskeemi põhjal
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Määra eksplicitse tüübiks "function"
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

    Ülaltoodud kood võtab MCP serveri vastuse ja teisendab selle tööriistade definitsiooni vormingusse, mida LLM mõistab.

1. Uuendame järgmise sammuna `run` meetodit, et loetleda serveri võimed:

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

    Eelnevas koodis uuendasime `run` meetodit, mis läbib tulemust ja iga kirje puhul kutsub `openAiToolAdapter` funktsiooni.

#### Python

1. Esiteks loome järgmise teisendusfunktsiooni

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

    Ülaltoodud `convert_to_llm_tools` funktsioon võtab MCP tööriista vastuse ja teisendab selle formaadiks, mida LLM mõistab.

1. Seejärel uuendame kliendi koodi, et kasutada seda funktsiooni niimoodi:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Siin lisame kutsumise `convert_to_llm_tool` funktsioonile, mis teisendab MCP tööriistade vastuse milleski, mida saame hiljem LLM-ile edastada.

#### .NET

1. Lisame koodi MCP tööriista vastuse teisendamiseks vormiks, mida LLM mõistab

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

Eelnevas koodis me:

- Loodud funktsiooni `ConvertFrom`, mis võtab nime, kirjelduse ja sisendskeemi.
- Määratletud funktsionaalsus, mis loob `FunctionDefinition`, mida antakse edasi `ChatCompletionsDefinition`-ile. Viimane on någotav LLM-ile.

1. Vaatame, kuidas me võiksime mõnda olemasolevat koodi selleks funktsiooniks uuendada:

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
// Loo botiliides loomuliku keele suhtluseks
public interface Bot {
    String chat(String prompt);
}

// Konfigureeri tehisintellekti teenus LLM ja MCP tööriistadega
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

Eelnevas koodis me:

- Määratlenud lihtsa `Bot` liidese loomuliku keelega suhtlemiseks.
- Kasutanud LangChain4j `AiServices`-i, et automaatselt siduda LLM MCP tööriistade pakkujaga.
- Raamistik haldab automaatselt tööriistade skeemi teisendust ja funktsioonide kutsumist.
- See lähenemine kõrvaldab käsitsi tööriistade teisendamise - LangChain4j haldab kogu MCP tööriistade LLM-iga ühilduvaks vormindamise keerukuse.

#### Rust

MCP tööriista vastuse teisendamiseks LLM-i mõistetavasse vormingusse lisame abifunktsiooni, mis vormindab tööriistade loendi. Lisage järgmine kood oma `main.rs` faili `main` funktsiooni alla. Seda kutsutakse LLM päringute tegemisel:

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

Suurepärane, nüüd oleme valmis kasutajapäringute vastu võtmiseks, nii et asume sellele järgmisena.

### -4- Kasutaja päringu käsitlemine

Selles koodi osas käsitleme kasutaja päringuid.

#### TypeScript

1. Lisage meetod, mida kasutatakse meie LLM-i kutsumiseks:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Kutsu serveri tööriista
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Tee midagi tulemusega
        // TEE TEHTUD

        }
    }
    ```

    Eelnevas koodis me:

    - Lisatud meetod `callTools`.
    - Meetod võtab LLM-i vastuse ja kontrollib, milliseid tööriistu (kui üldse) on kutsutud:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // kutsu tööriist
        }
        ```

    - Kutsub tööriista, kui LLM näitab, et seda tuleb kutsuda:

        ```typescript
        // 2. Kutsu serveri tööriista
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Tee midagi tulemusega
        // TODO
        ```

1. Uuendage `run` meetod LLM kutsumiste ja `callTools` funktsiooni kutsumisega:

    ```typescript

    // 1. Loo sõnumid, mis on LLM-i sisendiks
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM-i kutsumine
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Läbi vaata LLM-i vastus, iga valiku puhul kontrolli, kas sellel on tööriista kutsed
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Suurepärane, loetleme kogu koodi:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Impordi zod skeemi valideerimiseks

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // võib tulevikus olla vaja muuta sellele URL-ile: https://models.github.ai/inference
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
          // Loo zod skeem sisend_skeemi põhjal
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Määra tüübi väärtus selgelt "function"
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
    
    
          // 2. Kutsu serveri tööriista
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Tee midagi tulemusega
          // TEE TEHTUD
    
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
    
        // 1. Läbi LLM vastus, iga valiku puhul kontrolli, kas seal on tööriista kutsed
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

1. Lisame vajalikud impordid LLM-i kutsumiseks

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Järgmisena lisame funktsiooni, mis kutsub LLM-i:

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
            # Valikulised parameetrid
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

    Eelnevas koodis me:

    - Andsime meie funktsioonid, mille MCP serverilt leidsime ja teisendasime, LLM-ile.
    - Seejärel kutsusime LLM-i nimetatud funktsioonidega.
    - Seejärel kontrollime tulemust, et näha, milliseid funktsioone kutsuda tuleks, kui üldse.
    - Lõpuks anname üleskutsete funktsioonide massiivi.

1. Lõpuks uuendame oma peamist koodi:

    ```python
    prompt = "Add 2 to 20"

    # küsi LLM-ilt, milliseid tööriistu kõik saavad, kui üldse
    functions_to_call = call_llm(prompt, functions)

    # kutsu välja soovitatud funktsioonid
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Siin, see oli viimane samm, kus me:

    - Kutsume MCP tööriista läbi `call_tool`, kasutades funktsiooni, mida LLM meie päringu põhjal kutsuda soovitas.
    - Trükime tööriista kutse tulemuse MCP serverisse.

#### .NET

1. Näitame koodi LLM päringu tegemiseks:

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

    Eelnevas koodis me:

    - Saime tööriistad MCP serverilt, `var tools = await GetMcpTools()`.
    - Määratlesime kasutajapäringu `userMessage`.
    - Konstrukteerisime optsi objektina mudel ja tööriistad.
    - Tegime päringu LLM-ile.

1. Viimane samm, vaatame, kas LLM arvas, et tuleks funktsiooni kutsuda:

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

    Eelnevas koodis me:

    - Läbisime funktsioonikutsumise listi.
    - Iga tööriista kutse puhul tõlgendame nime ja argumendid ning kutsume tööriista MCP serveril MCP kliendi kaudu. Lõpuks trükime tulemused.

Kood terviklikult:

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
    // Täida loomuliku keele päringud, mis automaatselt kasutavad MCP tööriistu
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

Eelnevas koodis me:

- Kasutasime lihtsaid loomuliku keele käivitusi MCP serveri tööriistadega suhtlemiseks.
- LangChain4j raamistik haldab automaatselt:
  - Kasutajapäringute teisendamist tööriistade kutsumiseks vajadusel.
  - Sobivate MCP tööriistade kutsumist LLM otsuse põhjal.
  - Vestluse juhtimist LLM-i ja MCP serveri vahel.
- `bot.chat()` meetod tagastab loomulikus keeles vastuseid, mis võivad sisaldada MCP tööriistade täitmise tulemusi.
- See lähenemine pakub sujuvat kasutajakogemust, kus kasutaja ei pea teadma MCP alusmehhanismi.

Täielik koodinäide:

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

Siin toimub töö suurem osa. Kutsume LLM-i algse kasutajapäringuga, töötleme vastuse, et näha, kas mõnda tööriista tuleb kutsuda. Kui tuleb, kutsume need tööriistad ja jätkame vestlust LLM-iga, kuni enam tööriistu kutsuda ei ole vaja ja meil on lõplik vastus.

Teeme mitu LLM kutsumist, seega määratleme funktsiooni, mis haldab LLM kutset. Lisage järgmine funktsioon oma `main.rs` faili:

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

See funktsioon võtab LLM kliendi, sõnumite loendi (sealhulgas kasutajapäring), MCP serveri tööriistad ja saadab LLM-ile päringu, tagastades vastuse.
LLM vastus sisaldab massiivi `choices`. Me peame töötlema tulemust, et näha, kas esinevad `tool_calls`. See annab teada, et LLM palub konkreetset tööriista kutsuda koos argumentidega. Lisa oma `main.rs` faili lõppu järgmine funktsioon LLM vastuse töötlemiseks:

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

    // Prindi sisu, kui see on saadaval
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Töötle tööriista kõnesid
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Lisa assistendi sõnum

        // Käivita iga tööriista kõne
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Lisa tööriista tulemus sõnumitesse
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Jätka vestlust tööriista tulemustega
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

Kui `tool_calls` on olemas, tõmmatakse välja tööriista info, kutsutakse MCP server tööriista päringuga ja tulemused lisatakse vestluse sõnumitesse. Seejärel jätkatakse vestlust LLM-iga ning sõnumid uuendatakse assistendi vastuse ja tööriista kutsumise tulemustega.

Selleks, et eraldada LLM-i poolt MCP kutsumiseks tagastatud tööriista kutsumise info, lisame veel ühe abifunktsiooni, mis eraldab kõik vajaliku kutsumiseks. Lisa oma `main.rs` faili lõppu järgmine kood:

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

Kõik vajalik paigas, saame nüüd töödelda esialgset kasutaja päringut ja kutsuda LLM-i. Uuenda oma `main` funktsiooni järgnevaga:

```rust
// LLM vestlus tööriistade kutsumisega
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

See küsib LLM-ilt esialgset kasutaja päringut kahe numbri summa kohta ning töötleb vastust, et dünaamiliselt hallata tööriista kutsumisi.

Suurepärane, sul see õnnestus!

## Ülesanne

Võta harjutusest kood ning arenda serverisse lisaks veel mõningaid tööriistu. Seejärel loo klient koos LLM-iga nagu harjutuses ning testi seda erinevate päringutega, et veenduda, kas kõik su serveri tööriistad kutsutakse dünaamiliselt. Selline kliendi loomise viis annab lõppkasutajale suurepärase kogemuse, sest nad saavad kasutada päringuid täpsete kliendi käskude asemel ning ei pea teadma, et MCP serverit kutsutakse.

## Lahendus

[Lahendus](/03-GettingStarted/03-llm-client/solution/README.md)

## Peamised õppetunnid

- LLM-i lisamine oma kliendile annab parema võimaluse kasutajatega MCP serveritega suhelda.
- Pead teisendama MCP serveri vastuse kujule, mida LLM mõistab.

## Näited

- [Java Kalkulaator](../samples/java/calculator/README.md)
- [.Net Kalkulaator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Kalkulaator](../samples/javascript/README.md)
- [TypeScript Kalkulaator](../samples/typescript/README.md)
- [Python Kalkulaator](../../../../03-GettingStarted/samples/python)
- [Rust Kalkulaator](../../../../03-GettingStarted/samples/rust)

## Lisamaterjalid

## Mis järgmiseks

- Järgmine: [Serveri kasutamine Visual Studio Code kaudu](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastutusest loobumine**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, olge teadlikud, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul on soovitatav kasutada professionaalset inimtõlget. Me ei vastuta käesoleva tõlke kasutamisest tingitud arusaamatuste või väärarusaamade eest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
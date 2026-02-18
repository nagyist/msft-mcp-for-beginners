# Ügyfél létrehozása LLM-mel

Eddig láttad, hogyan lehet szervert és ügyfelet létrehozni. Az ügyfél képes volt expliciten hívni a szervert, hogy listázza az eszközeit, erőforrásait és promptjait. Ez azonban nem túl praktikus megközelítés. A felhasználód az ügynöki korszakban él, és azt várja, hogy promtokat használjon és LLM-mel kommunikáljon. A felhasználód számára nem számít, hogy MCP-t használsz-e a képességeid tárolásához, de azt elvárja, hogy természetes nyelven kommunikálhasson. Hogyan oldjuk meg ezt? A megoldás az, hogy LLM-et adunk az ügyfélhez.

## Áttekintés

Ebben a leckében arra fókuszálunk, hogy egy LLM-et adjunk az ügyfélhez, és bemutatjuk, hogyan nyújt ez sokkal jobb élményt a felhasználónak.

## Tanulási célok

A lecke végére képes leszel:

- Ügyfelet létrehozni egy LLM-mel.
- Zökkenőmentesen kommunikálni MCP szerverrel LLM segítségével.
- Jobb végfelhasználói élményt biztosítani az ügyfél oldalon.

## Megközelítés

Próbáljuk meg megérteni, milyen megközelítést kell alkalmaznunk. Egy LLM hozzáadása egyszerűnek tűnik, de valóban így fogjuk csinálni?

Így fog az ügyfél kommunikálni a szerverrel:

1. Kapcsolat létesítése a szerverrel.

1. Képességek, promptok, erőforrások és eszközök listázása, majd ezek séma szerinti mentése.

1. LLM hozzáadása, valamint a mentett képességek és sémájuk átadása a LLM-nek olyan formátumban, amit az ért.

1. Felhasználói prompt kezelése azzal, hogy a promptot az LLM-nek adjuk át a kliens által listázott eszközökkel együtt.

Nagyszerű, most, hogy nagy vonalakban értjük, hogyan lehet ezt megvalósítani, próbáljuk ki a következő gyakorlatban.

## Gyakorlat: Ügyfél létrehozása LLM-mel

Ebben a gyakorlatban megtanuljuk, hogyan adjunk LLM-et az ügyfelünkhöz.

### Hitelesítés GitHub személyes hozzáférési tokennel

Egy GitHub token létrehozása egyszerű folyamat. Így csinálhatod:

- Menj a GitHub beállításokhoz – Kattints a profilképedre a jobb felső sarokban, majd válaszd a Beállításokat.
- Navigálj a Fejlesztői beállításokhoz – Görgess le és kattints a Fejlesztői beállításokra.
- Válaszd a Személyes hozzáférési tokeneket – Kattints a Finomhangolt tokenekre, majd az Új token létrehozására.
- Konfiguráld a tokened – Adj megjegyzést a tokenhez, állítsd be a lejárati időt, és válaszd ki a szükséges jogosultságokat. Ebben az esetben mindenképp add hozzá a Modellek engedélyt.
- Generáld és másold ki a tokent – Kattints a Token létrehozására, és azonnal másold ki, mert később nem fogod látni újra.

### -1- Kapcsolódás a szerverhez

Először hozzuk létre az ügyfelünket:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Zod importálása sémavizsgálathoz

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

A fenti kódban:

- Importáltuk a szükséges könyvtárakat.
- Létrehoztunk egy osztályt két taggal, `client` és `openai`, amelyek segítenek az ügyfél kezelésében, illetve az LLM-mel való interakcióban.
- Beállítottuk az LLM példányunkat, hogy GitHub Modelleket használjon az `baseUrl` értékének az inference API-ra mutatásával.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Szerver paraméterek létrehozása stdio kapcsolat számára
server_params = StdioServerParameters(
    command="mcp",  # Futtatható állomány
    args=["run", "server.py"],  # Opcionális parancssori argumentumok
    env=None,  # Opcionális környezeti változók
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # A kapcsolat inicializálása
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

A fenti kódban:

- Importáltuk az MCP-hez szükséges könyvtárakat.
- Létrehoztunk egy ügyfelet.

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

Először is hozzá kell adnod a LangChain4j függőségeket a `pom.xml` fájlodhoz. Add hozzá ezeket a függőségeket, hogy MCP integrációt és GitHub Modellek támogatást kapj:

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

Ezután hozd létre a Java kliens osztályodat:

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
    
    public static void main(String[] args) throws Exception {        // Állítsa be az LLM-et GitHub modellek használatára
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Hozzon létre MCP átvitelt a szerverhez való kapcsolódáshoz
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // Hozzon létre MCP klienst
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

A fenti kódban:

- **Hozzáadtuk a LangChain4j függőségeket**: Az MCP integráció, az OpenAI hivatalos kliens és a GitHub Modellek támogatásához szükségesek.
- **Importáltuk a LangChain4j könyvtárakat**: Az MCP integráció és az OpenAI chat modellhez.
- **Létrehoztunk egy `ChatLanguageModel`-t**: GitHub Modellekhez konfigurálva GitHub tokennel.
- **Beállítottuk az HTTP transportot**: Server-Sent Events (SSE) használatával az MCP szerverhez való kapcsolódáshoz.
- **Létrehoztunk egy MCP klienst**: Ami kezeli a szerverrel való kommunikációt.
- **Használtuk a LangChain4j beépített MCP támogatását**: Ami leegyszerűsíti az LLM-ek és MCP szerverek közötti integrációt.

#### Rust

Ez a példa feltételezi, hogy van egy Rust alapú MCP szervered futtatásra kész állapotban. Ha nincs, tekintsd át a [01-first-server](../01-first-server/README.md) leckét a szerver létrehozásához.

Miután megvan a Rust MCP szervered, nyiss egy terminált és navigálj abba a mappába, ahol a szerver található. Futtasd a következő parancsot egy új LLM kliens projekt létrehozásához:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Add hozzá a következő függőségeket a `Cargo.toml` fájlodhoz:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Nincs hivatalos Rust könyvtár az OpenAI-hoz, azonban az `async-openai` crate egy [közösség által karbantartott könyvtár](https://platform.openai.com/docs/libraries/rust#rust), amit gyakran használnak.

Nyisd meg a `src/main.rs` fájlt, és cseréld le a tartalmát a következő kódra:

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
    // Kezdeti üzenet
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI kliens beállítása
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP kliens beállítása
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

    // TODO: MCP eszközlistázás beszerzése

    // TODO: LLM beszélgetés eszközhívásokkal

    Ok(())
}
```

Ez a kód egy alap Rust alkalmazást állít be, amely kapcsolódik egy MCP szerverhez és GitHub Modellekhez az LLM interakcióhoz.

> [!IMPORTANT]
> Győződj meg róla, hogy beállítod az `OPENAI_API_KEY` környezeti változót GitHub tokeneddel, mielőtt futtatnád az alkalmazást.

Nagyszerű, a következő lépésként listázzuk a szerver képességeit.

### -2- Szerver képességek listázása

Most csatlakozunk a szerverhez és lekérdezzük a képességeit:

#### Typescript

Ugyanabban az osztályban add hozzá a következő metódusokat:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // eszközök felsorolása
    const toolsResult = await this.client.listTools();
}
```

A fenti kódban:

- Hozzáadtunk kapcsolódási kódot a szerverhez, `connectToServer`.
- Létrehoztunk egy `run` metódust, ami kezeli az alkalmazás folyamatát. Egyelőre csak az eszközöket listázza, de hamarosan többet fogunk adni hozzá.

#### Python

```python
# Elérhető erőforrások listázása
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Elérhető eszközök listázása
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Ezt adtuk hozzá:

- Lekérdeztük az erőforrásokat és eszközöket, majd kiírtuk őket. Az eszközöknél az `inputSchema`-t is listázzuk, amit később használunk.

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

A fenti kódban:

- Lekértük a MCP szerveren elérhető eszközöket.
- Minden eszközhöz listáztuk a nevét, leírását és sémáját, amit később fogunk használni az eszközök hívásához.

#### Java

```java
// Hozzon létre egy eszköz szolgáltatót, amely automatikusan felfedezi az MCP eszközöket
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// Az MCP eszköz szolgáltató automatikusan kezeli:
// - Az elérhető eszközök listázását az MCP szerverről
// - Az MCP eszköz sémák LangChain4j formátumba konvertálását
// - Az eszköz végrehajtásának és válaszainak kezelését
```

A fenti kódban:

- Létrehoztunk egy `McpToolProvider`-t, ami automatikusan felfedezi és regisztrálja az összes eszközt az MCP szerverről.
- Az eszköz szolgáltató belsőleg kezeli az MCP eszköz sémák és LangChain4j eszköz formátum közötti konverziót.
- Ez a megközelítés elrejti a kézi eszközlista és konverziós folyamatot.

#### Rust

Az MCP szerverről az eszközök lekérése a `list_tools` metódussal történik. A `main` függvényedben, miután beállítottad az MCP klienst, add hozzá a következő kódot:

```rust
// MCP eszköz lista lekérése
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Szerver képességek konvertálása LLM eszközökké

A következő lépés a szerver képességeinek olyan formátumba való konvertálása, amit az LLM ért. Ha ezt megcsináljuk, az LLM-nek eszközként tudjuk átadni ezeket a képességeket.

#### TypeScript

1. Add hozzá a következő kódot az MCP szerver válaszának olyan eszköz formátumba konvertálásához, amit az LLM használhat:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Hozzon létre egy zod sémát az input_schema alapján
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Állítsa be kifejezetten a típust "function"-re
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

    A fentebbi kód egy MCP szerver választ vesz, és azt eszköz definíciós formátumban alakítja át, amit az LLM érteni tud.

1. Frissítsük a `run` metódust, hogy listázza a szerver képességeit:

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

    A fenti kódban frissítettük a `run` metódust úgy, hogy átgörgeti az eredményt, és minden bejegyzéshez meghívja a `openAiToolAdapter`-t.

#### Python

1. Először készítsük el a következő konvertáló függvényt:

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

    A fenti `convert_to_llm_tools` függvény MCP eszközválaszt vesz, és olyanná alakítja át, amit az LLM érteni tud.

1. Ezután frissítsük a kliens kódját, hogy használja ezt a függvényt:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Itt meghívjuk a `convert_to_llm_tool`-t, hogy az MCP eszköz válaszát olyanná alakítsuk, amit később az LLM-nek átadhatunk.

#### .NET

1. Adjunk hozzá kódot, hogy az MCP eszköz válaszát olyanná alakítsuk, amit az LLM ért:

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

A fenti kódban:

- Létrehoztunk egy `ConvertFrom` nevű függvényt, ami nevet, leírást és bemeneti sémát vesz.
- Definiáltunk olyan funkcionalitást, ami egy `FunctionDefinition`-t hoz létre, amit egy `ChatCompletionsDefinition`-nek adunk át. Utóbbit az LLM érti.

1. Nézzük meg, hogyan frissíthetjük a meglévő kódot ennek a függvénynek a használatára:

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
// Hozzon létre egy Bot interfészt természetes nyelvű interakcióhoz
public interface Bot {
    String chat(String prompt);
}

// Konfigurálja az AI szolgáltatást LLM és MCP eszközökkel
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

A fenti kódban:

- Egyszerű `Bot` interfészt definiáltunk természetes nyelvű interakcióhoz.
- Használtuk a LangChain4j `AiServices` osztályát az LLM és az MCP eszköz szolgáltató automatikus összekapcsolásához.
- A keretrendszer automatikusan kezeli az eszköz sémák konverzióját és a függvényhívásokat a háttérben.
- Ez a megközelítés megszünteti a kézi eszköz konverzió szükségességét — a LangChain4j kezeli az MCP eszközök LLM-kompatibilis formátummá alakításának minden bonyolultságát.

#### Rust

Az MCP eszközválasz LLM által értett formátummá alakításához adunk egy segédfüggvényt, ami az eszközök listázását formázza. Add hozzá a következő kódot a `main.rs` fájlodba, a `main` függvény alá. Ezt akkor hívjuk meg, amikor az LLM-hez küldünk kérdéseket:

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

Nagyszerű, most, hogy készen állunk a felhasználói kérések kezelésére, lássuk azt.

### -4- Felhasználói prompt kezelés

Ebben a kódrészletben kezeljük a felhasználói kéréseket.

#### TypeScript

1. Adjunk hozzá egy metódust, amivel hívhatjuk az LLM-et:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Hívja meg a szerver eszközét
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Tegyen valamit az eredménnyel
        // TEENDŐ

        }
    }
    ```

    A fenti kódban:

    - Hozzáadtuk a `callTools` metódust.
    - A metódus kap egy LLM választ, és megnézi, hogy milyen eszközöket hívtak meg, ha egyáltalán:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // eszköz hívása
        }
        ```

    - Meghívja az eszközt, ha az LLM azt jelezte, hogy meg kell hívni:

        ```typescript
        // 2. Hívja meg a szerver eszközét
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Tegyen valamit az eredménnyel
        // TEENDŐ
        ```

1. Frissítsük a `run` metódust, hogy az LLM hívásokat és a `callTools` meghívását tartalmazza:

    ```typescript

    // 1. Üzenetek létrehozása, amelyek bemenetként szolgálnak az LLM-hez
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. Az LLM meghívása
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Végigmegyünk az LLM válaszán, minden választásnál ellenőrizzük, hogy vannak-e eszközhívások
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Nagyszerű, itt van a teljes kód:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Zod importálása sémavalidációhoz

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // előfordulhat, hogy a jövőben ezt az URL-t kell használni: https://models.github.ai/inference
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
          // Egy zod séma létrehozása az input_schema alapján
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Típusként kifejezetten "function" beállítása
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
    
    
          // 2. A szerver eszközének meghívása
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Valami végrehajtása az eredménnyel
          // Tennivaló
    
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
    
        // 1. Áttekinteni az LLM válaszát, minden választásnál ellenőrizni, hogy van-e eszközhívás
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

1. Adjunk hozzá néhány importot, amire szükség van az LLM híváshoz:

    ```python
    # nagy nyelvi modell
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Ezután hozzuk létre a függvényt, ami hívja az LLM-et:

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
            # Opcionális paraméterek
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

    A fenti kódban:

    - Átadtuk az MCP szerverről kapott, és átalakított függvényeinket az LLM-nek.
    - Meghívtuk az LLM-et ezekkel a függvényekkel.
    - Megvizsgáltuk az eredményt, hogy lássuk, kell-e függvényt hívnunk.
    - Végül egy függvények listáját adjuk át a hívásra.

1. Végül frissítsük a fő kódunkat:

    ```python
    prompt = "Add 2 to 20"

    # kérdezd meg az LLM-et, milyen eszközöket használjon, ha egyáltalán
    functions_to_call = call_llm(prompt, functions)

    # hívd meg a javasolt függvényeket
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Ez volt az utolsó lépés, a fenti kódban:

    - Meghívunk egy MCP eszközt `call_tool`-lal, azt a funkciót, amit az LLM szerint hívnunk kell a prompt alapján.
    - Kiíratjuk az eszköz hívás eredményét az MCP szerverre.

#### .NET

1. Mutatunk egy példát, hogyan lehet egy LLM prompt kérést végrehajtani:

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

    A fenti kódban:

    - Lekértük az eszközöket az MCP szerverről, `var tools = await GetMcpTools()`.
    - Definiáltunk egy felhasználói promptot `userMessage`.
    - Létrehoztunk egy opció objektumot, megadva modellt és eszközöket.
    - Küldtünk egy kérést az LLM-nek.

1. Egy utolsó lépésként nézzük meg, hogy az LLM szerint kell-e függvényt hívni:

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

    A fenti kódban:

    - Végigiteráltunk a függvényhívások listáján.
    - Minden eszközhívásnál kinyertük a nevet és az argumentumokat, majd meghívtuk az eszközt az MCP szerveren az MCP kliens segítségével. Végül kiírtuk az eredményeket.

Itt a teljes kód:

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
    // Végrehajtja a természetes nyelvű kéréseket, amelyek automatikusan használják az MCP eszközöket
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

A fenti kódban:

- Egyszerű, természetes nyelvű promptokat használtunk az MCP szerver eszközeinek hívásához.
- A LangChain4j keretrendszer automatikusan kezeli:
  - A felhasználói promptokat eszköz hívásokká alakítva szükség esetén
  - A megfelelő MCP eszközök hívását az LLM döntése alapján
  - A beszélgetési folyamat kezelését az LLM és az MCP szerver között
- A `bot.chat()` metódus visszaad természetes nyelvű válaszokat, amelyek tartalmazhatnak MCP eszközök eredményeit is.
- Ez a megközelítés zökkenőmentes felhasználói élményt nyújt, ahol a felhasználóknak nem kell érteniük az MCP implementációt.

Teljes kódpélda:

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

Itt történik a munka nagy része. Meghívjuk az LLM-et a kezdeti felhasználói prompttal, majd feldolgozzuk a választ, hogy lássuk, kell-e eszközt hívni. Ha kell, akkor meghívjuk az eszközöket, és folytatjuk a beszélgetést az LLM-mel, amíg már nincs több eszközhívás és megkapjuk a végleges választ.

Többször fogunk hívni az LLM-et, ezért definiáljunk egy függvényt az LLM hívás kezelésére. Add hozzá a következő függvényt a `main.rs` fájlodhoz:

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

Ez a függvény megkapja az LLM klienset, egy üzenetlistát (a felhasználói prompttal együtt), az MCP szerverről az eszközöket, és küld egy kérést az LLM-nek, majd visszaadja a választ.
Az LLM válasza egy `choices` tömböt fog tartalmazni. A válasz feldolgozásakor meg kell vizsgálnunk, hogy vannak-e `tool_calls`. Ez jelzi, hogy az LLM egy adott eszköz meghívását kéri argumentumokkal. Add hozzá a következő kódot a `main.rs` fájlod aljához, hogy definiálj egy függvényt az LLM válasz kezelésére:

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

    // Tartalom nyomtatása, ha elérhető
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Eszközhívások kezelése
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Segédüzenet hozzáadása

        // Minden eszközhívás végrehajtása
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Eszköz eredmény hozzáadása az üzenetekhez
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Beszélgetés folytatása eszközeredményekkel
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

Ha vannak `tool_calls`, akkor kinyeri az eszköz információkat, meghívja az MCP szervert az eszköz kérésével, és hozzáadja az eredményeket a beszélgetés üzeneteihez. Ezután folytatja a beszélgetést az LLM-mel, és az üzenetek frissülnek az asszisztens válaszával és az eszköz hívás eredményeivel.

Az MCP hívásokhoz az LLM által visszaadott eszköz hívási információk kinyeréséhez egy másik segédfüggvényt adunk hozzá, ami mindent kinyer a híváshoz. Add hozzá a következő kódot a `main.rs` fájlod aljához:

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

Minden darab a helyén, most már kezelhetjük a kezdeti felhasználói parancsot és hívhatjuk az LLM-et. Frissítsd a `main` függvényedet az alábbi kód hozzáadásával:

```rust
// LLM beszélgetés eszközhívásokkal
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

Ez az initial felhasználói prompt-tal lekérdezi az LLM-et, amely két szám összegét kéri, és feldolgozza a választ, hogy dinamikusan kezelje az eszköz hívásokat.

Nagyszerű, sikerült!

## Feladat

Vedd a gyakorlatból a kódot és építsd tovább a szervert több eszközzel. Ezután készíts egy klienset egy LLM-mel, mint a gyakorlatban, és teszteld különböző promptokkal, hogy biztos legyen, hogy a szerver összes eszköze dinamikusan hívásra kerül. Ez a kliens építési mód azt jelenti, hogy a végfelhasználónak nagyszerű felhasználói élménye lesz, mivel promptokat használhat parancsok helyett, és nem is fogja észrevenni, hogy MCP szerverhez történik a hívás.

## Megoldás

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## Fő tanulságok

- Egy LLM hozzáadása a klienshez jobb módot nyújt a felhasználóknak az MCP szerverekkel való interakcióra.
- Az MCP szerver válaszát át kell alakítani valami olyanná, amit az LLM megért.

## Minták

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## További források

## Mi következik

- Következő: [Szerver használata a Visual Studio Code segítségével](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár a pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget a fordítás használatából eredő félreértésekért vagy téves értelmezésekért.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
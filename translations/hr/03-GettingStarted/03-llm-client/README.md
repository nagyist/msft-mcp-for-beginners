# Kreiranje klijenta sa LLM-om

Do sada ste vidjeli kako stvoriti poslužitelj i klijenta. Klijent je mogao eksplicitno pozivati poslužitelj da navede njegove alate, resurse i upite. Međutim, to nije baš praktičan pristup. Vaš korisnik živi u doba agenata i očekuje da koristi upite i komunicira s LLM-om kako bi to učinio. Za vašeg korisnika nije važno koristite li MCP ili ne za pohranu vaših mogućnosti, ali očekuju da koriste prirodni jezik za interakciju. Kako to riješiti? Rješenje se sastoji u dodavanju LLM-a klijentu.

## Pregled

U ovoj lekciji fokusiramo se na dodavanje LLM-a vašem klijentu i pokazujemo kako to pruža mnogo bolje korisničko iskustvo.

## Ciljevi učenja

Do kraja ove lekcije moći ćete:

- Stvoriti klijenta s LLM-om.
- Bešavno komunicirati s MCP poslužiteljem koristeći LLM.
- Pružiti bolje korisničko iskustvo na strani klijenta.

## Pristup

Pokušajmo razumjeti pristup koji trebamo poduzeti. Dodavanje LLM-a zvuči jednostavno, ali hoćemo li to zaista napraviti?

Evo kako će klijent komunicirati s poslužiteljem:

1. Uspostavi vezu s poslužiteljem.

1. Navedite mogućnosti, upite, resurse i alate te spremite njihov shema.

1. Dodajte LLM i proslijedite spremljene mogućnosti i njihov shema u formatu koji LLM razumije.

1. Obradite korisnički upit prosljeđujući ga LLM-u zajedno s alatima koje je klijent naveo.

Odlično, sada kada razumijemo kako to učiniti na visokoj razini, isprobajmo u sljedećem zadatku.

## Vježba: Kreiranje klijenta s LLM-om

U ovoj vježbi naučit ćemo kako dodati LLM našem klijentu.

### Autentifikacija korištenjem GitHub tokena za pristup

Kreiranje GitHub tokena je jednostavan proces. Evo kako to možete napraviti:

- Idite na GitHub Postavke – Kliknite na svoju profilnu sliku u gornjem desnom kutu i odaberite Postavke.
- Navigirajte do Postavki programera – Skrolajte dolje i kliknite na Postavke programera.
- Odaberite Osobne Access Tokene – Kliknite na Fine-grained tokens zatim Generiraj novi token.
- Konfigurirajte svoj token – Dodajte napomenu za referencu, postavite datum isteka i odaberite potrebne ovlasti (dozvole). U ovom slučaju ne zaboravite dodati Models dozvolu.
- Generirajte i kopirajte token – Kliknite na Generiraj token i odmah ga kopirajte jer ga nećete moći ponovno vidjeti.

### -1- Povezivanje s poslužiteljem

Najprije stvorimo našeg klijenta:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Uvezite zod za validaciju sheme

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

U prethodnom kodu smo:

- Uvezli potrebne biblioteke
- Stvorili klasu s dva člana, `client` i `openai`, koji nam pomažu upravljati klijentom i komunicirati s LLM-om.
- Konfigurirali našu LLM instancu da koristi GitHub modele postavljanjem `baseUrl` na inferencijsku API adresu.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Stvori parametre poslužitelja za stdio vezu
server_params = StdioServerParameters(
    command="mcp",  # Izvršna datoteka
    args=["run", "server.py"],  # Neobavezni argumenti naredbenog retka
    env=None,  # Neobavezne varijable okoline
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # Inicijaliziraj vezu
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

U prethodnom kodu smo:

- Uvezli potrebne biblioteke za MCP
- Stvorili klijenta

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

Prvo, potrebno je dodati LangChain4j zavisnosti u vaš `pom.xml` datoteku. Dodajte ove zavisnosti za omogućavanje MCP integracije i podrške za GitHub modele:

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

Zatim stvorite svoju Java klijentsku klasu:

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
    
    public static void main(String[] args) throws Exception {        // Konfiguriraj LLM za korištenje GitHub modela
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Kreiraj MCP prijenos za povezivanje sa serverom
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // Kreiraj MCP klijent
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

U prethodnom kodu smo:

- **Dodali LangChain4j zavisnosti**: Potrebne za MCP integraciju, službeni OpenAI klijent i podršku za GitHub modele
- **Uvezli LangChain4j biblioteke**: Za MCP integraciju i funkcionalnost OpenAI chat modela
- **Stvorili `ChatLanguageModel`**: Konfiguriran da koristi GitHub modele s vašim GitHub tokenom
- **Postavili HTTP transport**: Korištenjem Server-Sent Events (SSE) za povezivanje s MCP poslužiteljem
- **Stvorili MCP klijenta**: Koji će upravljati komunikacijom s poslužiteljem
- **Koristili ugrađenu podršku za MCP u LangChain4j**: Koja pojednostavljuje integraciju između LLM-ova i MCP poslužitelja

#### Rust

Ovaj primjer pretpostavlja da imate Rust bazirani MCP poslužitelj koji radi. Ako ga nemate, pogledajte [01-first-server](../01-first-server/README.md) lekciju za kreiranje poslužitelja.

Kad imate svoj Rust MCP poslužitelj, otvorite terminal i idite u isti direktorij kao i poslužitelj. Zatim pokrenite sljedeću naredbu za stvaranje novog LLM klijentskog projekta:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Dodajte sljedeće zavisnosti u vašu `Cargo.toml` datoteku:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Nema službene Rust biblioteke za OpenAI, ali `async-openai` paket je [biblioteka održavana od zajednice](https://platform.openai.com/docs/libraries/rust#rust) koja se često koristi.

Otvorite `src/main.rs` datoteku i zamijenite njen sadržaj sa sljedećim kodom:

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
    // Početna poruka
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // Postavi OpenAI klijent
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // Postavi MCP klijent
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

    // TODO: Dohvati popis MCP alata

    // TODO: Razgovor LLM-a s pozivima alata

    Ok(())
}
```

Ovaj kod postavlja osnovnu Rust aplikaciju koja će se povezati na MCP poslužitelj i GitHub modele za interakcije s LLM-om.

> [!IMPORTANT]
> Obavezno postavite `OPENAI_API_KEY` varijablu okoline s vašim GitHub tokenom prije pokretanja aplikacije.

Odlično, za naš sljedeći korak, navedimo mogućnosti na poslužitelju.

### -2- Navođenje mogućnosti poslužitelja

Sada ćemo se povezati na poslužitelj i zatražiti njegove mogućnosti:

#### Typescript

U istoj klasi, dodajte sljedeće metode:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // popisivanje alata
    const toolsResult = await this.client.listTools();
}
```

U prethodnom kodu smo:

- Dodali kod za povezivanje s poslužiteljem, `connectToServer`.
- Stvorili `run` metodu odgovornu za rukovanje tijekom naše aplikacije. Do sada samo navodi alate, ali uskoro ćemo dodati i drugo.

#### Python

```python
# Nabrojite dostupne resurse
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Nabrojite dostupne alate
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Evo što smo dodali:

- Navođenje resursa i alata te ispis istih. Za alate također navodimo `inputSchema` koji ćemo koristiti kasnije.

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

U prethodnom kodu smo:

- Naveli dostupne alate na MCP poslužitelju
- Za svaki alat naveli ime, opis i njegov schemu. Ovo ćemo koristiti uskoro za pozivanje alata.

#### Java

```java
// Kreirajte pružatelja alata koji automatski otkriva MCP alate
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// Pružatelj MCP alata automatski upravlja:
// - Popisivanje dostupnih alata s MCP poslužitelja
// - Pretvaranje MCP šema alata u LangChain4j format
// - Upravljanje izvršavanjem alata i odgovorima
```

U prethodnom kodu smo:

- Stvorili `McpToolProvider` koji automatski otkriva i registrira sve alate s MCP poslužitelja
- Tool provider interno upravlja konverzijom između MCP tool schema i LangChain4j alata formata
- Ovaj pristup apstrahira ručno navođenje i konverziju alata

#### Rust

Dohvat alata s MCP poslužitelja se radi metodom `list_tools`. U vašoj `main` funkciji, nakon postavljanja MCP klijenta, dodajte sljedeći kod:

```rust
// Dohvati popis MCP alata
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Pretvorba mogućnosti poslužitelja u LLM alate

Sljedeći korak nakon navođenja mogućnosti poslužitelja jest konvertirati ih u format koji LLM razumije. Nakon što to učinimo, možemo ih ponuditi kao alate našem LLM-u.

#### TypeScript

1. Dodajte sljedeći kod za konverziju odgovora s MCP poslužitelja u format alata kojeg LLM može koristiti:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Kreirajte zod shemu na temelju input_schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Izričito postavite tip na "function"
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

    Gornji kod uzima odgovor MCP poslužitelja i konvertira ga u definiciju alata koju LLM može razumjeti.

1. Ažurirajmo zatim `run` metodu kako bismo naveli mogućnosti poslužitelja:

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

    U prethodnom kodu smo ažurirali `run` metodu da prolazi kroz rezultat i za svaki unos pozove `openAiToolAdapter`.

#### Python

1. Najprije, kreirajmo sljedeću funkciju za konverziju

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

    U funkciji `convert_to_llm_tools` uzimamo MCP alatni odgovor i konvertiramo ga u format koji LLM razumije.

1. Zatim ažurirajmo naš klijentski kod da koristi ovu funkciju ovako:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Ovdje dodajemo poziv `convert_to_llm_tool` za konvertiranje MCP alatnog odgovora u nešto što kasnije možemo proslijediti LLM-u.

#### .NET

1. Dodajmo kod za pretvorbu MCP alata odgovora u nešto što LLM može razumjeti

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

U prethodnom kodu smo:

- Stvorili funkciju `ConvertFrom` koja prihvaća ime, opis i input schemu.
- Definirali funkcionalnost koja stvara `FunctionDefinition` koja se prosljeđuje u `ChatCompletionsDefinition`. Ovo potonje LLM može razumjeti.

1. Pogledajmo kako ažurirati postojeći kod da koristi ovu funkciju:

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
// Napravite Bot sučelje za interakciju prirodnim jezikom
public interface Bot {
    String chat(String prompt);
}

// Konfigurirajte AI uslugu s LLM i MCP alatima
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

U prethodnom kodu smo:

- Definirali jednostavan `Bot` interface za interakciju prirodnim jezikom
- Koristili LangChain4j `AiServices` za automatsko povezivanje LLM-a s MCP tool providerom
- Framework automatski upravlja konverzijom tool schema i pozivanjem funkcija u pozadini
- Ovaj pristup eliminira ručnu konverziju alata - LangChain4j upravlja svim složenostima pretvaranja MCP alata u format kompatibilan s LLM-om

#### Rust

Za konverziju MCP alatnog odgovora u format koji LLM može razumjeti, dodati ćemo pomoćnu funkciju koja formatira popis alata. Dodajte sljedeći kod u vašu `main.rs` datoteku ispod `main` funkcije. Ovo će se pozivati kada šaljemo zahtjeve LLM-u:

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

Odlično, sada smo spremni za rukovanje korisničkim zahtjevima, pa to riješimo sljedeće.

### -4- Obrada korisničkog upita

U ovom dijelu koda ćemo obraditi korisničke zahtjeve.

#### TypeScript

1. Dodajte metodu koja će se koristiti za pozivanje našeg LLM-a:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Pozovite alat poslužitelja
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Napravite nešto s rezultatom
        // ZA URADITI

        }
    }
    ```

    U prethodnom kodu smo:

    - Dodali metodu `callTools`.
    - Metoda prima LLM odgovor i provjerava koji su alati pozvani, ako ih uopće ima:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // poziv alata
        }
        ```

    - Poziva alat ako LLM pokazuje da bi ga trebalo pozvati:

        ```typescript
        // 2. Pozovite alat servera
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Napravite nešto s rezultatom
        // ZA NAPRAVITI
        ```

1. Ažurirajte `run` metodu da uključi pozive LLM-u i pozivanje `callTools`:

    ```typescript

    // 1. Kreirajte poruke koje su ulaz za LLM
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. Pozivanje LLM-a
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Pregledajte odgovor LLM-a, za svaki izbor provjerite sadrži li pozive alata
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Odlično, evo kompletnog koda:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // Uvezi zod za validaciju sheme

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // možda će biti potrebno promijeniti u ovaj URL u budućnosti: https://models.github.ai/inference
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
          // Kreiraj zod shemu na temelju input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Izričito postavi tip na "function"
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
    
    
          // 2. Pozovi alat poslužitelja
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Uradi nešto s rezultatom
          // ZA NAPRAVITI
    
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
    
        // 1. Prođi kroz odgovor LLM-a, za svaki izbor provjeri ima li poziva alata
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

1. Dodajmo potrebne uvoze za pozivanje LLM-a

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Zatim, dodajmo funkciju koja će pozivati LLM:

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
            # Opcionalni parametri
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

    U prethodnom kodu smo:

    - Proslijedili naše funkcije, koje smo pronašli na MCP poslužitelju i konvertirali, LLM-u.
    - Zatim smo pozvali LLM s tim funkcijama.
    - Nakon toga pregledavamo rezultat da vidimo koje funkcije treba pozvati, ako uopće postoje.
    - Na kraju prosljeđujemo niz funkcija za pozivanje.

1. Završni korak, ažurirajmo glavni kod:

    ```python
    prompt = "Add 2 to 20"

    # pitaj LLM koje alate koristiti, ako ih ima
    functions_to_call = call_llm(prompt, functions)

    # pozovi predložene funkcije
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Eto, to je zadnji korak, u gornjem kodu:

    - Pozivamo MCP alat preko `call_tool` koristeći funkciju koju je LLM odabrao na temelju našeg upita.
    - Ispisujemo rezultat poziva alata MCP poslužitelju.

#### .NET

1. Prikažimo kod za poziv LLM upita:

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

    U prethodnom kodu smo:

    - Dohvatili alate s MCP poslužitelja, `var tools = await GetMcpTools()`.
    - Definirali korisnički upit `userMessage`.
    - Konstrukcija opcija navodeći model i alate.
    - Napravili upit prema LLM-u.

1. Još jedan korak, pogledajmo razmišljanje LLM-a o pozivanju funkcije:

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

    U prethodnom kodu smo:

    - Prošli kroz listu poziva funkcija.
    - Za svaki poziv alata dohvatili ime i argumente i pozvali MCP alat koristeći MCP klijenta. Na kraju ispisujemo rezultate.

Evo cijelog koda:

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
    // Izvršite zahtjeve na prirodnom jeziku koji automatski koriste MCP alate
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

U prethodnom kodu smo:

- Koristili jednostavne upite na prirodnom jeziku za interakciju s MCP alatima
- LangChain4j framework automatski upravlja:
  - Pretvaranjem korisničkih upita u pozive alata po potrebi
  - Pozivanjem odgovarajućih MCP alata prema odluci LLM-a
  - Upravljanjem tijekovima razgovora između LLM-a i MCP poslužitelja
- `bot.chat()` metoda vraća odgovore na prirodnom jeziku koji mogu uključivati rezultate izvršenja MCP alata
- Ovaj pristup pruža glatko korisničko iskustvo gdje korisnici ne moraju znati o pozadini MCP implementacije

Potpuni primjer koda:

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

Ovdje se odvija veći dio posla. Pozvat ćemo LLM s početnim korisničkim upitom, zatim procesirati odgovor da vidimo treba li zvati neke alate. Ako je potrebno, pozvat ćemo te alate i nastaviti razgovor s LLM-om dok više nema potrebe za pozivima alata i imamo konačni odgovor.

Obavit ćemo više poziva LLM-u, pa definirajmo funkciju koja će obraditi LLM poziv. Dodajte sljedeću funkciju u vaš `main.rs`:

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

Ova funkcija prima LLM klijenta, listu poruka (uključujući korisnički upit), alate s MCP poslužitelja i šalje zahtjev LLM-u, vraćajući odgovor.
Odgovor iz LLM-a sadržavat će niz `choices`. Morat ćemo obraditi rezultat da vidimo ima li prisutnih `tool_calls`. To nam govori da LLM traži da se pozove određeni alat s argumentima. Dodajte sljedeći kod na dno vaše datoteke `main.rs` kako biste definirali funkciju za rukovanje LLM odgovorom:

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

    // Ispiši sadržaj ako je dostupan
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Obradi pozive alata
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Dodaj poruku asistenta

        // Izvrši svaki poziv alata
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Dodaj rezultat alata u poruke
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Nastavi razgovor s rezultatima alata
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

Ako su prisutni `tool_calls`, funkcija izvlači informacije o alatu, poziva MCP server s zahtjevom za alat i dodaje rezultate u poruke razgovora. Zatim nastavlja razgovor s LLM-om, a poruke se ažuriraju s odgovorom asistenta i rezultatima poziva alata.

Da bismo izvukli informacije o pozivu alata koje LLM vraća za MCP pozive, dodati ćemo još jednu pomoćnu funkciju za izdvajanje svega što je potrebno za izvršenje poziva. Dodajte sljedeći kod na dno vaše datoteke `main.rs`:

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

Sada kada su svi dijelovi na mjestu, možemo rukovati početnim korisničkim upitom i pozvati LLM. Ažurirajte svoju funkciju `main` da uključi sljedeći kod:

```rust
// Razgovor LLM-a s pozivima alata
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

Ovo će poslati upit LLM-u s početnim korisničkim upitom tražeći zbroj dvaju brojeva te će obraditi odgovor za dinamičko upravljanje pozivima alata.

Odlično, uspjeli ste!

## Zadatak

Uzmite kod iz vježbe i izgradite server s još nekoliko alata. Zatim kreirajte klijenta s LLM-om, kao u vježbi, i testirajte ga s različitim upitima kako biste bili sigurni da se svi alati vašeg servera dinamički pozivaju. Ovaj način izgradnje klijenta znači da će krajnji korisnik imati izvrsno korisničko iskustvo jer može koristiti upite, umjesto točnih klijentskih naredbi, i bit će nesvjestan bilo kakvog MCP servera koji se poziva.

## Rješenje

[Rješenje](/03-GettingStarted/03-llm-client/solution/README.md)

## Ključni zaključci

- Dodavanje LLM-a vašem klijentu pruža bolji način za korisnike da komuniciraju s MCP Serverima.
- Potrebno je pretvoriti odgovor MCP Servera u nešto što LLM može razumjeti.

## Primjeri

- [Java Kalkulator](../samples/java/calculator/README.md)
- [.Net Kalkulator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Kalkulator](../samples/javascript/README.md)
- [TypeScript Kalkulator](../samples/typescript/README.md)
- [Python Kalkulator](../../../../03-GettingStarted/samples/python)
- [Rust Kalkulator](../../../../03-GettingStarted/samples/rust)

## Dodatni resursi

## Što slijedi

- Sljedeće: [Korištenje servera u Visual Studio Codeu](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Izjava o odricanju od odgovornosti**:
Ovaj je dokument preveden pomoću AI usluge za prijevod [Co-op Translator](https://github.com/Azure/co-op-translator). Iako nastojimo postići točnost, molimo imajte na umu da automatski prijevodi mogu sadržavati pogreške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporučuje se profesionalni ljudski prijevod. Ne snosimo odgovornost za bilo kakve nesporazume ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
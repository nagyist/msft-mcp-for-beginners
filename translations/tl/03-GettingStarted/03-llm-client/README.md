# Paglikha ng kliyente gamit ang LLM

Sa ngayon, nakikita mo na kung paano gumawa ng server at kliyente. Ang kliyente ay nakapagsagawa na ng tawag sa server nang hayagan upang ilista ang mga tool, resources, at prompts nito. Gayunpaman, hindi ito gaanong praktikal na paraan. Ang iyong user ay nabubuhay sa agentic era at inaasahan na gamitin ang mga prompt at makipag-ugnayan sa isang LLM upang gawin ito. Para sa iyong user, hindi mahalaga kung gagamit ka ng MCP o hindi para iimbak ang iyong mga kakayahan ngunit inaasahan nila na gamitin ang natural na wika para makipag-interact. Paano natin ito malulutas? Ang solusyon ay ang pagdaragdag ng LLM sa kliyente.

## Pangkalahatang Pagsilip

Sa araling ito, magpo-focus tayo sa pagdaragdag ng LLM sa iyong kliyente at ipapakita kung paano ito nagbibigay ng mas magandang karanasan para sa iyong user.

## Mga Layunin sa Pagkatuto

Sa pagtatapos ng araling ito, magagawa mo na:

- Gumawa ng kliyente na may LLM.
- Mag-interact nang maayos sa isang MCP server gamit ang LLM.
- Magbigay ng mas magandang karanasan sa end user sa bahagi ng kliyente.

## Pamamaraan

Subukan nating unawain ang pamamaraan na kailangan nating gawin. Ang pagdaragdag ng LLM ay tila simple, ngunit gagawin ba talaga natin ito?

Ganito makikipag-ugnayan ang kliyente sa server:

1. Magtatag ng koneksyon sa server.

1. Ililista ang mga kakayahan, prompts, resources at tools, at ise-save ang kanilang schema.

1. Magdaragdag ng LLM at ipapasa ang na-save na mga kakayahan at ang kanilang schema sa format na naiintindihan ng LLM.

1. Pamahalaan ang isang user prompt sa pamamagitan ng pagpapasa nito sa LLM kasabay ng mga tools na nilista ng kliyente.

Magaling, ngayon na naiintindihan natin kung paano ito gawin sa mataas na antas, subukan natin ito sa sumusunod na ehersisyo.

## Ehersisyo: Paglikha ng kliyente na may LLM

Sa ehersisyong ito, matututuhan natin kung paano magdagdag ng LLM sa ating kliyente.

### Pag-authenticate gamit ang GitHub Personal Access Token

Ang paggawa ng GitHub token ay diretso lang. Ganito ang paraan:

- Pumunta sa GitHub Settings – I-click ang iyong larawan sa profile sa kanang itaas na sulok at piliin ang Settings.
- Pumunta sa Developer Settings – Mag-scroll pababa at i-click ang Developer Settings.
- Piliin ang Personal Access Tokens – I-click ang Fine-grained tokens at pagkatapos ay Generate new token.
- I-configure ang Iyong Token – Magdagdag ng tala para reference, mag-set ng expiration date, at piliin ang kinakailangang mga scope (permissions). Sa kasong ito, siguraduhing idagdag ang Models permission.
- I-generate at Kopyahin ang Token – I-click ang Generate token, at siguraduhing kopyahin ito agad, dahil hindi mo na ito makikita muli.

### -1- Kumonekta sa server

Gawin muna natin ang ating kliyente:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // I-import ang zod para sa pag-validate ng schema

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

Sa naunang code ay:

- Inimport ang mga kinakailangang libraries
- Gumawa ng klase na may dalawang miyembro, `client` at `openai` na tutulong sa atin sa pamamahala ng kliyente at pag-interact sa LLM nang paisa-isa.
- I-setup ang LLM instance para gamitin ang GitHub Models sa pamamagitan ng pag-set ng `baseUrl` upang ituro sa inference API.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# Gumawa ng mga parameter ng server para sa stdio na koneksyon
server_params = StdioServerParameters(
    command="mcp",  # Maaaring patakbuhin
    args=["run", "server.py"],  # Opsyonal na mga argumento sa command line
    env=None,  # Opsyonal na mga variable sa kapaligiran
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # Simulan ang koneksyon
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

Sa naunang code ay:

- Inimport ang mga kinakailangang libraries para sa MCP
- Gumawa ng kliyente

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

Una, kailangan mong idagdag ang LangChain4j dependencies sa iyong `pom.xml` file. Idagdag ang mga ito upang paganahin ang MCP integration at suporta para sa GitHub Models:

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

Pagkatapos ay gumawa ng iyong Java client class:

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
    
    public static void main(String[] args) throws Exception {        // I-configure ang LLM upang gamitin ang mga Modelo ng GitHub
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // Gumawa ng MCP transport para kumonekta sa server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // Gumawa ng MCP client
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

Sa naunang code ay:

- **Idinagdag ang LangChain4j dependencies**: Kinakailangan para sa MCP integration, opisyal na OpenAI client, at suporta sa GitHub Models
- **Inimport ang LangChain4j libraries**: Para sa MCP integration at functionality ng OpenAI chat model
- **Gumawa ng `ChatLanguageModel`**: Naka-configure para gamitin ang GitHub Models gamit ang iyong GitHub token
- **Nag-setup ng HTTP transport**: Gamit ang Server-Sent Events (SSE) para kumonekta sa MCP server
- **Gumawa ng MCP client**: Na siyang hahawak ng komunikasyon sa server
- **Ginamit ang built-in MCP support ng LangChain4j**: Na nagpapadali ng integration sa pagitan ng LLMs at MCP servers

#### Rust

Ang halimbawang ito ay nagpapalagay na may tumatakbong Rust based MCP server ka na. Kung wala ka pang ganoon, balikan ang [01-first-server](../01-first-server/README.md) na aralin upang gumawa ng server.

Kapag mayroon ka nang Rust MCP server, buksan ang terminal at pumunta sa parehong direktoryo ng server. Patakbuhin ang sumusunod na utos upang gumawa ng bagong LLM client project:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

Idagdag ang mga sumusunod na dependencies sa iyong `Cargo.toml` file:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> Walang opisyal na Rust library para sa OpenAI, ngunit ang `async-openai` crate ay isang [community maintained library](https://platform.openai.com/docs/libraries/rust#rust) na karaniwang ginagamit.

Buksan ang `src/main.rs` na file at palitan ang laman nito ng sumusunod na code:

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
    // Paunang mensahe
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // I-set up ang OpenAI client
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // I-set up ang MCP client
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

    // TODO: Kumuha ng listahan ng MCP tool

    // TODO: Pag-uusap ng LLM kasama ang tawag sa tool

    Ok(())
}
```

Itinakda ng code na ito ang isang basic na Rust application na kokonekta sa MCP server at GitHub Models para sa pag-interak sa LLM.

> [!IMPORTANT]
> Siguraduhing i-set ang `OPENAI_API_KEY` environment variable gamit ang iyong GitHub token bago patakbuhin ang application.

Magaling, sa susunod nating hakbang, ililista natin ang mga kakayahan sa server.

### -2- Ilista ang mga kakayahan ng server

Ngayon ay kokonekta tayo sa server at hihingin ang mga kakayahan nito:

#### Typescript

Sa parehong klase, idagdag ang mga sumusunod na method:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // mga kasangkapang panglistahan
    const toolsResult = await this.client.listTools();
}
```

Sa naunang code ay:

- Idinagdag ang code para kumonekta sa server, `connectToServer`.
- Gumawa ng `run` method na responsable sa pag-handle ng daloy ng app. Sa ngayon, nililista lang nito ang mga tools ngunit magdagdag tayo ng iba pang hakbang.

#### Python

```python
# Ilan ang mga magagamit na mapagkukunan
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# Ilan ang mga magagamit na kasangkapan
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

Ito ang ating idinagdag:

- Paglilista ng resources at tools at ipinrint ang mga ito. Sa mga tools, nililista rin ang `inputSchema` na gagamitin natin mamaya.

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

Sa naunang code ay:

- Nilista ang mga tools na available sa MCP Server
- Para sa bawat tool, nilista ang pangalan, paglalarawan, at ang schema nito. Ito ay gagamitin natin para tawagan ang mga tool.

#### Java

```java
// Gumawa ng tagapagbigay ng tool na awtomatikong nakakahanap ng mga MCP tool
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// Ang tagapagbigay ng MCP tool ay awtomatikong humahawak ng:
// - Paglilista ng mga magagamit na tool mula sa MCP server
// - Pag-convert ng mga schema ng MCP tool sa format ng LangChain4j
// - Pamamahala ng pagpapatupad ng tool at mga tugon
```

Sa naunang code ay:

- Gumawa ng `McpToolProvider` na awtomatikong nagdi-discover at nagrerehistro ng lahat ng tools mula sa MCP server
- Pinangangasiwaan ng tool provider ang conversion sa pagitan ng MCP tool schemas at ng format ng tool sa LangChain4j internally
- Itong pamamaraan ay inaalis ang manu-manong proseso ng paglilista at conversion ng tools

#### Rust

Ang pagkuha ng mga tools mula sa MCP server ay ginagawa gamit ang `list_tools` method. Sa loob ng iyong `main` function, pagkatapos i-setup ang MCP client, idagdag ang sumusunod na code:

```rust
// Kunin ang listahan ng MCP tool
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- I-convert ang mga kakayahan ng server sa LLM tools

Susunod na hakbang pagkatapos mailista ang kakayahan ng server ay i-convert ang mga ito sa isang format na naiintindihan ng LLM. Kapag nagawa na natin ito, maaari nating ibigay ang mga kakayahang ito bilang mga tool sa ating LLM.

#### TypeScript

1. Idagdag ang sumusunod na code para i-convert ang response mula sa MCP Server sa isang tool format na magagamit ng LLM:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // Lumikha ng zod schema batay sa input_schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // Tahasang itakda ang uri sa "function"
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

    Kinukuha ng code sa itaas ang response mula sa MCP Server at kino-convert ito sa isang tool definition format na naiintindihan ng LLM.

1. I-update naman natin ang `run` method para ilista ang mga kakayahan ng server:

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

    Sa naunang code, na-update natin ang `run` method upang ipasa ang resulta at sa bawat entry ay tawagan ang `openAiToolAdapter`.

#### Python

1. Una, gumawa tayo ng sumusunod na converter function

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

    Sa function na `convert_to_llm_tools` ay kinukuha natin ang response ng MCP tool at kino-convert ito sa format na naiintindihan ng LLM.

1. Susunod, i-update natin ang client code para gamitin ang function na ito:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    Dito, nagdagdag tayo ng tawag sa `convert_to_llm_tool` para i-convert ang MCP tool response sa bagay na maaaring ipasa sa LLM.

#### .NET

1. Magdagdag tayo ng code para i-convert ang MCP tool response sa isang format na naiintindihan ng LLM

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

Sa naunang code ay:

- Gumawa ng function na `ConvertFrom` na tumatanggap ng name, description, at input schema.
- Nakadefine ang functionality na lumilikha ng FunctionDefinition na ipinapasa sa ChatCompletionsDefinition. Ito ang naiintindihan ng LLM.

1. Tingnan natin kung paano i-update ang ilang existing na code para mapakinabangan ang function sa itaas:

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
// Lumikha ng isang Bot na interface para sa pakikipag-ugnayan gamit ang natural na wika
public interface Bot {
    String chat(String prompt);
}

// I-configure ang serbisyo ng AI gamit ang LLM at MCP na mga kagamitan
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

Sa naunang code ay:

- Nagdefine ng simpleng `Bot` interface para sa natural language interactions
- Ginamit ang LangChain4j's `AiServices` para awtomatikong pagdugtungin ang LLM sa MCP tool provider
- Ang framework ay awtomatikong nagha-handle ng tool schema conversion at function calling sa likod ng mga eksena
- Inalis ng pamamaraang ito ang manu-manong conversion ng tools - ang LangChain4j ang humahawak ng lahat ng komplikasyon ng pag-convert ng MCP tools sa LLM-compatible na format

#### Rust

Para i-convert ang MCP tool response sa format na naiintindihan ng LLM, magdagdag tayo ng helper function na nagfo-format ng mga tool listing. Idagdag ang sumusunod na code sa `main.rs` file mo sa ilalim ng `main` function. Tatawagin ito kapag gumagawa ng request sa LLM:

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

Magaling, nakapaghanda na tayo para i-handle ang mga user requests, kaya gawin natin iyon ngayon.

### -4- Pamahalaan ang user prompt request

Sa bahagi ng code na ito, pamamahalaan natin ang mga user requests.

#### TypeScript

1. Magdagdag ng method na gagamitin para tawagan ang ating LLM:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. Tawagan ang kasangkapan ng server
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Gawin ang isang bagay gamit ang resulta
        // GAGAWIN

        }
    }
    ```

    Sa naunang code ay:

    - Nagdagdag ng method na `callTools`.
    - Tini-check ng method ang response ng LLM kung may mga tool na dapat tawagin:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // tawagan ang kasangkapan
        }
        ```

    - Tinatawag ang tool, kung sinasabi ng LLM na dapat itong tawagin:

        ```typescript
        // 2. Tawagin ang tool ng server
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. Gawin ang isang bagay gamit ang resulta
        // GAGAWIN
        ```

1. I-update ang `run` method para isama ang mga tawag sa LLM at pagtawag sa `callTools`:

    ```typescript

    // 1. Gumawa ng mga mensahe na input para sa LLM
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. Tawagin ang LLM
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. Suriin ang sagot ng LLM, para sa bawat pagpipilian, tingnan kung may mga tawag sa tool
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

Magaling, narito ang buong code:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // I-import ang zod para sa schema validation

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // maaaring kailanganin baguhin sa url na ito sa hinaharap: https://models.github.ai/inference
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
          // Gumawa ng zod schema base sa input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // Hayagang itakda ang uri bilang "function"
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
    
    
          // 2. Tawagan ang tool ng server
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. Gawin ang isang bagay sa resulta
          // GAGAWIN PA
    
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
    
        // 1. Suriin ang sagot ng LLM, para sa bawat pagpipilian, tingnan kung may tawag sa tool
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

1. Magdagdag muna tayo ng imports na kakailanganin para tawagin ang LLM

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. Susunod, idagdag ang function na tutawag sa LLM:

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
            # Mga opsyonal na parameter
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

    Sa naunang code ay:

    - Ipinasa ang mga functions na nakuha natin mula sa MCP server at na-convert sa LLM.
    - Tinawag ang LLM kasama ang mga functions.
    - Sinuri ang resulta upang makita kung anong function ang dapat tawagin, kung meron man.
    - Sa wakas, nagpapasa tayo ng array ng function na tatawagin.

1. Huling hakbang, i-update ang ating main na code:

    ```python
    prompt = "Add 2 to 20"

    # itanong sa LLM kung anong mga tool ang gagamitin, kung mayroon man
    functions_to_call = call_llm(prompt, functions)

    # tawagan ang mga mungkahing function
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    Dito, ang huling hakbang sa code ay:

    - Tinatawag ang MCP tool via `call_tool` gamit ang function na inisip ng LLM na dapat tawagin base sa prompt.
    - Ipiniprint ang resulta ng pagtawag sa tool mula sa MCP Server.

#### .NET

1. Ipakita natin ang code para sa LLM prompt request:

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

    Sa naunang code ay:

    - Kinuha ang mga tools mula sa MCP server, `var tools = await GetMcpTools()`.
    - Nagdefine ng user prompt na `userMessage`.
    - Gumawa ng options object na nagsasaad ng model at tools.
    - Gumawa ng request papunta sa LLM.

1. Isang huling hakbang, tingnan natin kung iniisip ng LLM na dapat tayong tumawag ng function:

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

    Sa naunang code ay:

    - Nag-loop sa listahan ng function calls.
    - Para sa bawat pagtawag ng tool, kinukuha ang pangalan at arguments at tinatawag ang tool sa MCP server gamit ang MCP client. Sa huli, piniprint ang resulta.

Narito ang buong code:

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
    // Isagawa ang mga kahilingan sa natural na wika na awtomatikong gumagamit ng mga kasangkapang MCP
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

Sa naunang code ay:

- Gumamit ng simpleng natural language prompts para makipag-ugnayan sa MCP server tools
- Awtonomong hinahandle ng LangChain4j framework ang:
  - Pag-convert ng user prompts sa tawag sa tool kung kinakailangan
  - Pagtawag sa tamang MCP tools base sa desisyon ng LLM
  - Pamamahala ng daloy ng usapan sa pagitan ng LLM at MCP server
- Ang `bot.chat()` method ay nagbabalik ng natural language responses na maaaring maglaman ng resulta mula sa pagpapatakbo ng MCP tools
- Binibigyan ng paraan na ito ang seamless na karanasan sa user kung saan hindi na nila kailangang malaman ang likod ng MCP implementation

Kumpletong halimbawa ng code:

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

Dito nagaganap ang karamihan ng trabaho. Tatawagin natin ang LLM gamit ang unang user prompt, tapos ipoproseso ang tugon para makita kung may mga tool na kailangang tawagin. Kung meron, tatawagin natin yung mga tool na iyon at ipagpapatuloy ang usapan sa LLM hanggang hindi na kailangan tawagan pa ng tool at makuha natin ang panghuling sagot.

Gagawa tayo ng maraming tawag sa LLM, kaya magdefine tayo ng function na hahawak sa tawag sa LLM. Idagdag ang sumusunod na function sa iyong `main.rs` file:

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

Ang function na ito ay tumatanggap ng LLM client, listahan ng mga mensahe (kasama ang user prompt), tools mula sa MCP server, at nagpapadala ng request sa LLM, na nagbabalik ng tugon.
Ang tugon mula sa LLM ay maglalaman ng isang array ng `choices`. Kailangan nating iproseso ang resulta upang makita kung may anumang `tool_calls` na naroroon. Ipinapakita nito na ang LLM ay humihiling na isang partikular na tool ang tawagin kasama ang mga argumento. Idagdag ang sumusunod na code sa dulo ng iyong `main.rs` na file upang tukuyin ang isang function para hawakan ang tugon ng LLM:

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

    // I-print ang nilalaman kung mayroon
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // Pamahalaan ang mga tawag sa tool
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // Magdagdag ng mensahe ng katulong

        // Isagawa ang bawat tawag sa tool
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // Idagdag ang resulta ng tool sa mga mensahe
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // Ipagpatuloy ang pag-uusap gamit ang mga resulta ng tool
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

Kung may mga `tool_calls`, kukunin nito ang impormasyon tungkol sa tool, tatawagin ang MCP server gamit ang tool request, at idaragdag ang mga resulta sa messages ng pag-uusap. Pagkatapos ay ipagpapatuloy nito ang pag-uusap kasama ang LLM at ang mga mensahe ay ia-update gamit ang tugon ng assistant at mga resulta ng tool call.

Upang kunin ang impormasyon ng tool call na ibinabalik ng LLM para sa mga tawag sa MCP, magdadagdag tayo ng isa pang helper function upang makuha ang lahat ng kailangan upang magawa ang tawag. Idagdag ang sumusunod na code sa dulo ng iyong `main.rs` na file:

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

Sa pagkakaroon ng lahat ng bahagi, maaari na nating hawakan ang initial user prompt at tawagin ang LLM. I-update ang iyong `main` na function upang isama ang sumusunod na code:

```rust
// Usapan ng LLM na may mga tawag sa kasangkapan
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

Ito ay magtatanong sa LLM gamit ang initial user prompt na humihiling ng sum ng dalawang numero, at ipoproseso nito ang tugon upang dinamiko na hawakan ang mga tool calls.

Magaling, nagawa mo ito!

## Assignment

Kunin ang code mula sa exercise at paunlarin ang server gamit ang ilan pang mga tool. Pagkatapos gumawa ng isang client na may LLM, gaya ng sa exercise, at subukan ito gamit ang iba't ibang mga prompt upang masiguro na ang lahat ng iyong server tools ay tinatawag nang dinamiko. Ang paraan ng paggawa ng client na ito ay nangangahulugan na magkakaroon ng mahusay na karanasan ang end user dahil maaari nilang gamitin ang mga prompt, sa halip na tiyak na mga utos ng client, at hindi nila direktang mapapansin ang anumang MCP server na tinatawag.

## Solution

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## Key Takeaways

- Ang pagdagdag ng LLM sa iyong client ay nagbibigay ng mas mahusay na paraan para makipag-ugnayan ang mga user sa MCP Servers.
- Kailangan mong i-convert ang tugon ng MCP Server sa isang bagay na maiintindihan ng LLM.

## Samples

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## Additional Resources

## What's Next

- Susunod: [Consuming a server using Visual Studio Code](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Paalala**:  
Ang dokumentong ito ay isinalin gamit ang serbisyong AI na pagsasalin na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kaming maging tumpak, mangyaring tandaan na ang mga awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o di-tumpak na impormasyon. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasaling-tao. Hindi kami mananagot sa anumang hindi pagkakaunawaan o maling interpretasyon na maaaring magmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# LLM உடன் ஒரு கிளையண்டை உருவாக்குதல்

இதுவரை, நீங்கள் எப்படி ஒரு சர்வர் மற்றும் கிளையண்டை உருவாக்குவது என்பதை பார்த்தீர்கள். கிளையண்ட் சர்வரை தெளிவாக அழைத்து அதன் கருவிகள், வளங்கள் மற்றும் ப்ராம்ட்களை பட்டியலிட முடிந்தது. இருப்பினும், இது மிகப் பயனுள்ள அணுகுமுறை அல்ல. உங்கள் பயனர் முகவரியாக் காலத்தில் வாழ்கிறார் மற்றும் இயற்கை மொழியைப் பயன்படுத்தி ப்ராம்ட்கள் மற்றும் LLM உடன் தொடர்பு கொள்வதை எதிர்பார்க்கிறார். உங்கள் பயனருக்கு, நீங்கள் MCP ஐ பயன்படுத்துகிறீர்களா என்பதை கவலைப்படாது, ஆனால் இயற்கை மொழியில் தொடர்பு கொள்வதையே அவர்கள் எதிர்பார்க்கிறார்கள். இப்போது இதை எப்படி தீர்ப்பது? தீர்வு என்பது கிளையண்டில் ஒரு LLM ஐச் சேர்ப்பதே.

## கண்ணோட்டம்

இந்த பாடத்தில், உங்கள் கிளையண்டில் LLM ஐச் சேர்ப்பதற்கு கவனம் செலுத்துவோம் மற்றும் இது உங்கள் பயனருக்கு எவ்வாறு சிறந்த அனுபவத்தை வழங்குகிறது என்பதை காண்போம்.

## கற்றல் நோக்கங்கள்

இந்த பாடத்தின் முடிவில், நீங்கள்:

- ஒரு LLM உடன் கிளையண்டை உருவாக்க முடியும்.
- MCP சர்வருடன் LLM ஐப் பயன்படுத்தி ஒருங்கிணைந்து செயல்பட முடியும்.
- கிளையண்ட் பக்கத்தில் ஒரு சிறந்த இறுதிப் பயனர் அனுபவத்தை வழங்க முடியும்.

## அணுகுமுறை

நாம் எடுக்க வேண்டிய அணுகுமுறையைப் புரிந்துகொள்ள முயன்றால். ஒரு LLM ஐச் சேர்ப்பது எளிதாய் தோன்றினாலும், நிச்சயமாக இதை செய்ய முடியுமா?

கிளையண்ட் சர்வருடன் தொடர்பு கொள்வது பின்வருமாறு இருக்கும்:

1. சர்வருடன் இணைக்கவும்.

1. திறன்கள், ப்ராம்ட்கள், வளங்கள் மற்றும் கருவிகளை பட்டியலிட்டு, அவற்றின் ஸ்கீமாவை சேமிக்கவும்.

1. ஒரு LLM ஐச் சேர்க்கவும் மற்றும் சேமித்த திறன்கள் மற்றும் அவற்றின் ஸ்கீமாவை LLM புரிந்து கொள்ளும் வடிவில் வழங்கவும்.

1. பயனர் ப்ராம்ட்டை LLM க்கு அனுப்பி, கிளையண்ட் பட்டியலிட்ட கருவிகளுடன் அதை பயன்படுத்தும்.

சிறந்தது, இப்போது இதை மேல்நிலை செயல்பாடாக எப்படி செய்யலாம் என்பதை புரிந்துகொண்டோம், கீழ்காணும் பயிற்சியில் இதை முயற்சிப்போம்.

## பயிற்சி: LLM உடன் ஒரு கிளையண்டை உருவாக்குதல்

இந்த பயிற்சியில், நாங்கள் LLM ஐ எங்கள் கிளையண்டில் சேர்க்கக் கற்றுக்கொள்ளப்போகிறோம்.

### GitHub Personal Access Token மூலம் அங்கீகரிப்பு

GitHub டோக்கன் உருவாக்குவது ஒரு நேர்த்தியான செயல்முறை. இதோ எப்படி செய்வது:

- GitHub அமைப்புகளுக்கு செல்லவும் – மேல் வலது மூலையில் உங்கள் ப்ரோஃபைல் படத்தை கிளிக் செய்து Settings ஐத் தேர்ந்தெடுக்கவும்.
- Developer Settings ஐ எட்டவும் – கீழே ஸ்க்ரோல் செய்து Developer Settings ஐத் தேர்ந்தெடுக்கவும்.
- Personal Access Tokens ஐ தேர்ந்தெடுக்கவும் – Fine-grained tokens மற்றும் பிறகு Generate new token கிளிக் செய்யவும்.
- உங்கள் டோக்கனை அமைக்கவும் – குறிப்பு சேர்க்கவும், கால அவகாசத்தை அமைக்கவும், தேவையான அனுமதிகளை (permissions) தேர்ந்தெடுக்கவும். இதில் Models அனுமதியையும் சேர்க்க உறுதி செய்யவும்.
- டோக்கனை உருவாக்கி நகலெடுக்கவும் – Generate token கிளிக் செய்து உடனடியாக நகலெடுக்கவும், மீண்டும் பார்க்க முடியாது.

### -1- சர்வருடன் இணைக்கவும்

முதல், எங்கள் கிளையண்டை உருவாக்குவோம்:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ஸ்கீமா சரிபார்ப்புக்கு zod ஐ இறக்குமதி செய்யவும்

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

மேலே உள்ள குறியியலில்:

- தேவையான நூலகங்களை இறக்குமதி செய்தோம்
- இரண்டு உறுப்பினர்கள் `client` மற்றும் `openai` கொண்ட ஒரு வகுப்பு உருவாக்கியுள்ளோம், இது பிரத்தியேகமாக ஒரு கிளையண்டுடன் மற்றும் LLM உடன் தொடர்பு கொள்ள உதவும்.
- GitHub Models ஐ பயன்படுத்த LLM உதாரணமானது `baseUrl` ஐ inference API க்கு அமைத்துள்ளோம்.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio இணைப்பிற்கான சேவையகம் அளவுருக்களை உருவாக்கு
server_params = StdioServerParameters(
    command="mcp",  # இயக்கக்கூடியது
    args=["run", "server.py"],  # விருப்பமான கட்டளை வரி argumentos
    env=None,  # விருப்பமான சுற்றுப்புற மாறிலிகள்
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # இணைப்பை துவக்கு
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

மேலே உள்ள குறியியலில்:

- MCP க்கான தேவையான நூலகங்களை இறக்குமதி செய்தோம்
- ஒரு கிளையண்டை உருவாக்கினோம்

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

முதலில், உங்கள் `pom.xml` கோப்பில் LangChain4j சார்புகளைச் சேர்க்க வேண்டும். MCP ஒருங்கிணைப்பு மற்றும் GitHub Models ஆதரவுக்கு இந்த சார்புகளைச் சேர்க்கவும்:

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

பிறகு உங்கள் ஜாவா கிளையண்ட் வகுப்பை உருவாக்குங்கள்:

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
    
    public static void main(String[] args) throws Exception {        // GitHub மாதிரிகளை பயன்படுத்த LLM ஐ கட்டமைக்கவும்
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // சேவையகத்துடன் இணைக்க MCP போக்குவரத்தை உருவாக்கவும்
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP கிளையன்டை உருவாக்கவும்
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

மேலே உள்ள குறியியலில்:

- **LangChain4j சார்புகளை** MCP ஒருங்கிணைப்பு, OpenAI அதிகாரப்பூர்வ கிளையண்ட், மற்றும் GitHub Models ஆதரவுக்கு சேர்த்துள்ளோம்
- **LangChain4j நூலகங்களை** MCP ஒருங்கிணைப்புக்கு மற்றும் OpenAI சந்தை மாதிரி செயல்பாட்டிற்கு இறக்குமதி செய்துள்ளோம்
- **`ChatLanguageModel` உருவாக்கியுள்ளோம்** உங்கள் GitHub டோக்கன் கொண்டு GitHub Models பயன்படுத்தி
- **HTTP பரிவஹனத்தை அமைத்துள்ளோம்** MCP சர்வருடன் சேர Server-Sent Events (SSE) பயன்படுத்தி
- **MCP கிளையண்டை உருவாக்கியுள்ளோம்** இது சர்வருடன் தொடர்பை கையாளும்
- **LangChain4j இன் இயற்கை MCP ஆதரவை** பயன்படுத்தியுள்ளோம், இது LLM மற்றும் MCP சர்வர்களுக்கு இடையேயான ஒருங்கிணைப்பை எளிதாக்குகிறது

#### Rust

இந்த எடுத்துக்காட்டு நீங்கள் ஒரு Rust அடிப்படையுடைய MCP சர்வர் இயக்கி இருந்தீர்கள் என கருதுகிறது. இல்லை என்றால், [01-first-server](../01-first-server/README.md) பாடத்துக்கு திரும்பி சர்வரை உருவாக்கவும்.

Rust MCP சர்வர் உள்ளது என்றால், ஒரு டெர்மினல் திறந்து சர்வர் இருப்பிடத்துக்கு செல்லவும். பிறகு கீழ்காணும் கட்டளையை இயக்கி ஒரு LLM கிளையண்ட் திட்டத்தை உருவாக்கவும்:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

`Cargo.toml` கோப்பில் பின்வரும் சார்புகளைச் சேர்க்கவும்:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI க்கான அதிகாரப்பூர்வ Rust நூலகம் இல்லை; இருப்பினும், `async-openai` கிரேட் என்பது பொதுமக்கள் பராமரிக்கும் நூலகம் ஆகும் [community maintained library](https://platform.openai.com/docs/libraries/rust#rust).

`src/main.rs` கோப்பை திறந்து உள்ளடக்கத்தை பின்வருமாறு மாற்றவும்:

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
    // ஆரம்ப செய்தி
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI கிளையண்டை அமைக்கவும்
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP கிளையண்டை அமைக்கவும்
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

    // TODO: MCP கருவி பட்டியலைப் பெறவும்

    // TODO: கருவி அழைப்புகளுடன் LLM உரையாடல்

    Ok(())
}
```

இந்த குறியியல் MCP சர்வர் மற்றும் GitHub Models உடன் LLM தொடர்பாக்கும் அடிப்படை Rust பயன்பாட்டை அமைக்கிறது.

> [!IMPORTANT]
> செயல்பாட்டை இயக்குவதற்கு முன் உங்கள் GitHub டோக்கன் கொண்டு `OPENAI_API_KEY` சுற்றுச்சூழல் மாறிலியை அமைக்க உறுதி செய்யவும்.

சிறந்தது, அடுத்த படி, சர்வர் திறன்களை பட்டியலிடுவோம்.

### -2- சர்வர் திறன்களை பட்டியலிடு

இப்போது சர்வருடன் இணைக்கப்பட்டு அதன் திறன்களை கேட்டுக்கொள்வோம்:

#### Typescript

அதே வகுப்பில், பின்வரும் முறைகளைச் சேர்க்கவும்:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // கருவிகளின் பட்டியலை காட்டுதல்
    const toolsResult = await this.client.listTools();
}
```

மேலே உள்ள குறியியலில்:

- `connectToServer` என்ற சர்வருடன் இணைக்கும் குறியீடு சேர்க்கப்பட்டுள்ளது.
- `run` என்ற முறை உருவாக்கப்பட்டுள்ளது, இது தற்பொழுது கருவிகளை மட்டுமே பட்டியலிடுகிறது, விரைவில் மேலும் சேர்க்கப்படும்.

#### Python

```python
# கிடைக்கும் வளங்களின் பட்டியல்
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# கிடைக்கும் கருவிகளின் பட்டியல்
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

நாம் சேர்த்தது:

- வளங்கள் மற்றும் கருவிகளை பட்டியலிட்டு அச்சிடியது. கருவிகள் க்கான `inputSchema` ஐ இது பயன்படுத்தி உள்ளது.

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

மேலே உள்ள குறியியலில்:

- MCP சர்வரில் கிடைக்கும் கருவிகள் பட்டியலிடப்பட்டன
- ஒவ்வொரு கருவிக்கும் பெயர், விளக்கம் மற்றும் அதன் ஸ்கீமா காட்டப்பட்டது. இதை விரைவில் கையாளவுள்ளோம்.

#### Java

```java
// MCP கருவிகளைக் தானாக கண்டுபிடிக்கும் ஒரு கருவி வழங்குநரை உருவாக்குங்கள்
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP கருவி வழங்குநர் தானாக கையாளும்:
// - MCP சர்வர் இருந்து கிடைக்கும் கருவிகளை பட்டியலிடுதல்
// - MCP கருவி சீரியங்களை LangChain4j வடிவத்திற்கு மாற்றுதல்
// - கருவி இயக்கம் மற்றும் பதில்களை நிர்வகித்தல்
```

மேலே உள்ள குறியியலில்:

- MCP சர்வரிலிருந்து அனைத்து கருவிகளையும் தானாக கண்டுபிடித்து பதிவு செய்யும் `McpToolProvider` உருவாக்கப்பட்டது
- கருவி வழங்கியவர் MCP கருவி ஸ்கீமாக்களையும் LangChain4j கருவி வடிவத்துடனும் மாற்றியமைக்கிறது
- இது கருவிகள் பட்டியலில் சேர்க்கும் மற்றும் மாற்றும் செயல்முறையை தானாக செய்கிறது

#### Rust

MCP சர்வரிலிருந்து கருவிகளை பெற `list_tools` முறை பயன்படுத்தப்படுகிறது. உங்கள் `main` செயல்பாட்டில் MCP கிளையண்டை அமைத்தபின் பின்வரும் குறியியலைச் சேர்க்கவும்:

```rust
// MCP கருவி பட்டியலை பெறுக
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- சர்வர் திறன்களை LLM கருவிகளுக்கு மாற்றுதல்

சர்வர் திறன்களை பட்டியலிட்ட பிறகு அடுத்த படி, அவைகளை LLM புரிந்து கொள்ளக்கூடிய வடிவுக்குத் தாருங்கள். இதுபோல் ஆவணங்களை LLM கருவிகளாக வழங்கலாம்.

#### TypeScript

1. MCP சர்வர் பதிலை LLM பயன்படுத்தக்கூடிய கருவிப் படியாக மாற்ற பின்வரும் குறியியலைச் சேர்க்கவும்:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // input_schema இன் அடிப்படையில் ஒரு zod ஸ்கீமாவை உருவாக்கவும்
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // வகையை "function" என்று தெளிவாக அமைக்கவும்
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

மேலே உள்ள குறியியல் MCP சர்வர் பதிலை எடுத்துக் கொண்டு LLM புரியும் கருவி வரையறையை உருவாக்குகிறது.

1. அடுத்து `run` முறையை இவ்வாறு புதுப்பிக்கவும்:

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

மேலே மேம்படுத்திய குறியியலில், `run` முறையில் முடிவுகளை வழிசெய்து ஒவ்வொரு நுழைவையும் `openAiToolAdapter` ஐ அழைக்கிறது.

#### Python

1. முதலில் பின்வருமாறு ஒரு மாற்றும் செயல்பாட்டை உருவாக்குவோம்

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

`convert_to_llm_tools` செயல்பாட்டில் MCP கருவி பதிலானது LLM புரிய கூடிய வடிவில் மாற்றப்படுகிறது.

1. அடுத்து, இதைப் பயன்படுத்தி கிளையண்ட் குறியியலை இவ்வாறு மேம்படுத்துவோம்:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

இங்கே, MCP கருவி பதிலை LLM க்கு அனுப்புவதற்கு `convert_to_llm_tool` அழைக்கும் குறியியலைச் சேர்த்துள்ளோம்.

#### .NET

1. MCP கருவி பதிலோ LLM புரியும் வடிவாக மாற்றியமைக்கும் குறியியலைச் சேர்ப்போம்

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

மேலே உள்ள குறியியலில்:

- `ConvertFrom` என்ற செயல்பாட்டை உருவாக்கியுள்ளோம், இதில் பெயர், விளக்கம் மற்றும் உள்ளீட்டு ஸ்கீமா ஆகியவை உள்ளன.
- ஒரு FunctionDefinition உருவாக்கி அதை ChatCompletionsDefinition க்கு அனுப்பும் செயல்பாடு உள்ளது, இது LLM புரிந்துகொள்ளும் வடிவம்.

1. இதை பயன்படுத்தி ஏற்கனவே உள்ள குறியியலை இவ்வாறு மேம்படுத்தலாம்:

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
// இயற்கை மொழி பரிமாற்றத்திற்கான ஒரு Bot இடைமுகத்தை உருவாக்குக
public interface Bot {
    String chat(String prompt);
}

// LLM மற்றும் MCP கருவிகளுடன் AI சேவையை கட்டமைக்கவும்
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

மேலே உள்ள குறியியலில்:

- இயற்கை மொழி தொடர்புக்கு ஒரு எளிய `Bot` இடைமுகம் வரையறுக்கப்பட்டுள்ளது
- LangChain4j இன் `AiServices` மூலம் LLM மற்றும் MCP கருவி வழங்கியவரை தானாக இணைத்துள்ளோம்
- கருவி ஸ்கீமா மாற்றமும் செயல்பாட்டுச் அழைப்பும் தானாக நடக்கின்றன
- இதன் மூலம் MCP கருவிகளை LLM உடன் பொருந்தக்கூடிய வடிவுக்கு மாற்றும் சிக்கல் LangChain4j மூலம் அகற்றப்படுகிறது

#### Rust

MCP கருவி பதிலை LLM புரிந்துகொள்ளும் வடிவமாக மாற்ற, கருவி பட்டியலை வடிவமைக்கும் உதவிச் செயல்பாட்டை `main.rs` ல் `main` செயல்பாட்டின் கீழ் சேர்க்கவும். இது LLM க்கு கோரிக்கை செய்யும் போது அழைக்கப்படும்:

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

சிறந்தது, பயனர் கோரிக்கைகளை கையாள இதுவரை அமைக்கவில்லை, அடுத்த படியாக அதை செய்யலாம்.

### -4- பயனர் ப்ராம்ட் கோரிக்கையை கையாளுதல்

இந்த பகுதி குறியியலில், பயனர் கோரிக்கைகளை கையாள்வோம்.

#### TypeScript

1. எங்கள் LLM ஐ அழைக்க பயன்படும் முறையைச் சேர்க்கவும்:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. சேவையகத்தின் கருவியை அழைக்கவும்
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. முடிவுடன் எதையாவது செய்க
        // செய்ய வேண்டியது

        }
    }
    ```

மேலே உள்ள குறியியலில்:

- `callTools` என்ற முறையைச் சேர்ந்துள்ளோம்.
- இந்த முறை LLM பதிலை எடுத்துக் கொண்டு எந்த கருவிகள் அழைக்கப்பட்டன என்று பார்க்கிறது:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // கருவியை அழைக்கவும்
        }
        ```

- LLM அழைக்க வேண்டும் என தெரிவினால் ஒரு கருவியை அழைக்கும்:

        ```typescript
        // 2. சர்வரின் கருவியை அழைக்கவும்
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. முடிவுடன் ஏதாவது செய்யவும்
        // செய்ய வேண்டியது
        ```

1. `run` முறையில் LLM அழைப்பੂக்கும் மற்றும் `callTools` அழைப்பூக்கும் சேர்க்கவும்:

    ```typescript

    // 1. LLMக்கு உள்ளீடு ஆகும் தகவல்களை உருவாக்கவும்
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM ஐ அழைக்கிறது
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM பதிலை ஒவ்வொரு தேர்வுக்கும் சென்றுச்சென்று, அது கருவி அழைப்புகளை கொண்டிருக்கிறதா என சரிபார்க்கவும்
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

நன்று, முழுக்க குறியியலை பட்டியலிடுகிறோம்:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // அமைப்பு சரிபார்ப்பிற்காக zod ஐ இறக்குமதி செய்க

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // வருங்காலத்தில் இந்த url ஆக மாற்றவேண்டும்: https://models.github.ai/inference
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
          // input_schema அடிப்படையில் ஒரு zod அமைப்பை உருவாக்குக
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // வகையை "function" என்று தெளிவுபடுத்தி அமைக்கவும்
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
    
    
          // 2. சர்வரின் கருவியை அழைக்கவும்
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. பெறுபடி மூலம் ஏதும் செய்யவும்
          // செய்யவேண்டியது
    
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
    
        // 1. LLM பதிலை வழியாகிச் செல்லவும், ஒவ்வொரு தேர்விலும் கருவி அழைப்புகள் உள்ளதா என்று சரிபார்க்கவும்
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

1. LLM அழைக்க தேவையான இறக்குமதிகளை சேர்க்கவும்

    ```python
    # எல்.எல்.எம்
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. அடுத்ததாக, LLM அழைக்கும் செயல்பாட்டைச் சேர்க்கவும்:

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
            # விருப்பமான அளவுருக்கள்
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

மேலே உள்ள குறியியலில்:

- MCP சர்வரில் கண்டுபிடித்த மற்றும் மாற்றிய செயல்பாடுகளை LLM க்கு வழங்கியுள்ளோம்.
- அதன் பிறகு இந்த செயல்பாடுகளுடன் LLM ஐ அழைத்துள்ளோம்.
- முடிவில் எதனை அழைக்கவேண்டும் என்பதைக் கண்டறிந்து, தேவையான செயல்பாடுகளை அழைக்கிறது.
- இறுதியில் அழைக்கவேண்டிய செயல்பாடுகளின் வரிசையை வழங்குகிறது.

1. கடைசி படி, நம்முடைய அடிப்படை குறியியலை மேம்படுத்துவோம்:

    ```python
    prompt = "Add 2 to 20"

    # LLM-க்கு எந்த கருவிகள் எல்லாம் இருக்கின்றன என்று கேளுங்கள், இருந்தால்
    functions_to_call = call_llm(prompt, functions)

    # பரிந்துரைக்கப்பட்ட செயல்பாடுகளை அழைக்கவும்
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

மேலே உள்ள குறியியலில்:

- LLM சொன்ன அடிப்படையில் `call_tool` மூலம் MCP கருவியை அழைக்கிறோம்.
- கருவி அழைப்பு முடிவுகளை MCP சர்வரில் அச்சிடுகிறோம்.

#### .NET

1. LLM ப்ராம்ட் கோரிக்கைக்கான குறியியலை காண்போம்:

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

மேலே:

- MCP சர்வரிலிருந்து கருவிகளை பெற்றோம் `var tools = await GetMcpTools()`.
- பயனர் ப்ராம்ட் `userMessage` உருவாக்கப்பட்டது.
- மாதிரி மற்றும் கருவிகளுடன் விருப்புகள் நிர்மாண்டப்பட்டன.
- LLM க்கு கோரிக்கை அனுப்பப்பட்டது.

1. கடைசியாக, LLM கூறின் ஏதேனும் செயல்பாட்டை அழைக்க தேவையிருக்கிறதா என்பதைப் பார்க்கலாம்:

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

மேலே:

- செயல்பாட்டுச் சுடுகாடுகளில் சுற்றி,
- ஒவ்வொரு கருவி அழைப்புக்கும் பெயர் மற்றும் அளவுருக்கள் பிரிக்கப்பட்டு MCP சர்வர் மூலம் அழைக்கப்பட்டது. முடிவுகள் அச்சிடப்பட்டன.

முழு குறியியல்:

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
    // MCP கருவிகளை தன்னியக்கமாகப் பயன்படுத்தும் இயற்கை மொழி கோரிக்கைகளை செயல்படுத்துங்கள்
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

மேலே:

- எளிய இயற்கை மொழி பதில்களுடன் MCP சர்வர் கருவிகளுடன் தொடர்பு கொண்டோம்
- LangChain4j கட்டமைப்பு தானாக:
  - தேவையான போது பயனர் ப்ராம்ட்களை கருவி அழைப்புகளாக மாற்றுகிறது
  - LLM தீர்மானப்படி பொருத்தமான MCP கருவிகளை அழைக்கிறது
  - LLM மற்றும் MCP சர்வருக்கு இடையேயான உரையாடலை நிர்வகிக்கிறது
- `bot.chat()` இயற்கை மொழி பதில்களை மடக்கி வழங்கும், அதில் MCP கருவி செயல்பாடுகளின் முடிவுகளும் இருக்கலாம்
- இதன் மூலம் பயனர் MCP உட்படுதலை பற்றிக் கவலைப்படாமலும் மென்மையான அனுபவம் கிடைக்கிறது

முழுமையான குறியியல் எடுத்துக்காட்டு:

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

இங்கு பெரும்பங்கு நடைபெறும். முதலில் பயனர் ப்ராம்ட் கொண்டு LLM ஐ அழைப்போம். பிறகு பதிலில் உள்ள கருவி அழைப்புகளை பரிசீலித்து, தேவையானவை இருந்தால் அவைகளை அழைக்கிறோம். இதைப் படிப்படியாக தொடர்ந்தே வா, எந்த கருவி அழைப்பும் தேவையில்லாத வரையில் LLM உடன் உரையாடலை நடத்துவோம்.

நாம் பல முறை LLM ஐ அழைப்பதாய் இருப்பதால் LLM அழைப்பை கையாளும் செயல்பாட்டை வரையறுக்கலாம். `main.rs` கோப்பில் பின்வரும் செயல்பாட்டைச் சேர்க்கவும்:

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

இந்த செயல்பாடு LLM கிளையண்டையும், செய்திகளின் பட்டியலையும் (பயனர் ப்ராம்டும் உட்பட), MCP சர்வர் கருவிகளையும் பெற்று, LLM ஐ கோரிக்கை செய்து பதிலை திருப்புகிறது.
LLM இலிருந்து வரும் பதில் `choices` என்ற வரிசையைக் கொண்டிருக்கும். எந்த `tool_calls` உள்ளதா என்பது பார்ப்பதற்காக முடிவை செயலாக்க வேண்டும். குறிப்பிட்ட கருவி உண்மையில் அழைக்கப்பட வேண்டுமா என LLM கேட்கும் என்பதை இதனால் தெரியும். LLM பதிலை கையாளும் ஒரு செயல்பாட்டை வரையறுக்க உங்கள் `main.rs` கோப்பின் கீழே பின்வரும் குறியீட்டை சேர்க்கவும்:

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

    // உள்ளடக்கம் கிடைத்தால் அச்சிடவும்
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // கருவி அழைப்புகளை கையாளவும்
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // உதவியாளர் செய்தியை சேர்க்கவும்

        // ஒவ்வொரு கருவி அழைப்பையும் செயல்படுத்தவும்
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // கருவி முடிவுகளை செய்திகள் பகுதியில் சேர்க்கவும்
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // கருவி முடிவுகளுடன் உரையாடலை தொடரவும்
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


`tool_calls` இருப்பின், அது கருவி தகவலை எடுத்து, MCP சேவையகத்துடன் கருவி வேண்டுதலை அழைக்கிறது மற்றும் உரையாடல் செய்திகளுக்கு முடிவுகளைச் சேர்க்கிறது. அதன் பிறகு LLM உடன் உரையாடலை தொடர்கிறது, செய்திகள் உதவியாளரின் பதிலும் கருவி அழைப்பு முடிவுகளும் இணைக்கப்படுகின்றன.

LLM MCP அழைப்புகளுக்காக திரும்ப அளிக்கும் கருவி அழைப்பு தகவலை எடுக்க மற்றொரு உதவி செயல்பாட்டையும் கீழே உள்ள `main.rs` கோப்பின் கீழ் சேர்க்கவுள்ளோம்:

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


அனைத்து பகுதிகளும் இடத்தில் இருப்பதால், ஆரம்ப பயனர் அறிவுறுத்தலையும் LLM அழைப்பையும் இப்போது கையாளலாம். `main` செயல்பாட்டை பின்வரும் குறியீட்டுடன் புதுப்பிக்கவும்:

```rust
// கருவி அழைப்புகளுடன் LLM உரையாடல்
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


இது ஆரம்ப பயனர் அறிவுறுத்தலை நாடி இரண்டு எண்களின் கூட்டுத்தொகையை கேட்டு LLM ஐ விசாரிக்கும், பதிலை செயலாக்கி கருவி அழைப்புக்களை தானாக கையாளும்.

அற்புதம், நீங்கள் முடித்தீர்கள்!

## பணிஇருப்பிடம்

உதவி குறியீட்டை எடுத்து மேலும் சில கருவிகளுடன் சேவையகத்தை உருவாக்கவும். பிறகு அதேபோல் ஒரு LLM உடன் கிளையண்டை உருவாக்கி, வேறு அறிவுறுத்தல்களுடன் பரிசோதித்து அனைத்து சேவையக கருவிகளும் தானாக அழைக்கப்படுவதைக் உறுதிசெய்யவும். இவ்வாறு கிளையண்டைப் build செய்வதால் இறுதிப் பயனர், துல்லியமான கிளையண்ட் கட்டளைகள் பதிலாக அறிவுறுத்தல்களை பயன்படுத்திச் சிறந்த பயனர் அனுபவத்தை பெறுவார் மற்றும் எந்த MCP சேவையகமும் அழைக்கப்படுவதை கவலைப்பட மாட்டார்.

## தீர்வு

[தீர்வு](/03-GettingStarted/03-llm-client/solution/README.md)

## முக்கியமானக் கருத்துக்கள்

- உங்கள் கிளையண்டில் LLM சேர்ப்பது MCP சேவையகங்களுடன் பயனர்களுக்குப் பயன்படுத்த எளிதாக இருக்கும்.
- MCP சேவையக பதிலை LLM புரிந்துகொள்ளக்கூடியவாறு மாற்ற வேண்டும்.

## மாதிரிகள்

- [Java கணக்கீட்டாளர்](../samples/java/calculator/README.md)
- [.Net கணக்கீட்டாளர்](../../../../03-GettingStarted/samples/csharp)
- [JavaScript கணக்கீட்டாளர்](../samples/javascript/README.md)
- [TypeScript கணக்கீட்டாளர்](../samples/typescript/README.md)
- [Python கணக்கீட்டாளர்](../../../../03-GettingStarted/samples/python)
- [Rust கணக்கீட்டாளர்](../../../../03-GettingStarted/samples/rust)

## கூடுதல் வளங்கள்

## அடுத்தது என்ன

- அடுத்து: [Visual Studio Code பயன்படுத்தி சேவையகத்தைப் பயன்படுத்துதல்](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**குறிப்புரை**:
இந்த ஆவணம் AI மொழி மாற்று சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழி மாற்றப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்கு முயற்சித்தாலும், தானாக உருவாக்கப்பட்ட மொழிபெயர்ப்புகளில் தவறுகள் அல்லது சிக்கல்கள் இருக்கலாம் என்பதை தயவுசெய்து கவனிக்கவும். இயல்பான மொழியில் உள்ள மூல ஆவணம் அதிகாரப்பூர்வமான ஆதாரமாகப் பாராய வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்முறை மனித மொழிபெயர்ப்பை பரிந்துரைக்கிறோம். இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்தவொரு தவறான புரிதல்கள் அல்லது தவறான விளக்கங்களுக்கு நாங்கள் பொறுப்பு ஏற்கவில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
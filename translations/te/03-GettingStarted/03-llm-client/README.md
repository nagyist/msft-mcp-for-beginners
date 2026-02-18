# LLMతో క్లయింట్ సృష్టించడం

ఇప్పటి వరకు, మీరు ఒక సర్వర్ మరియు క్లయింట్‌ను ఎలా సృష్టించాలో చూశారు. క్లయింట్ స్పష్టంగా సర్వర్‌ను కాల్ చేసి దాని టూల్స్, రిసోర్సెస్ మరియు ప్రాంప్ట్స్‌ను జాబితా చేయగలిగింది. అయితే, ఇది చాలా ప్రయోజనకరమైన పద్ధతి కాదు. మీ యూజర్ ఏజెంటిక్ యుగంలో ఉన్నారు మరియు ఇలాంటి సందర్భాల్లో ప్రాంప్ట్స్ ఉపయోగించి LLMతో కమ్యూనికేట్ చేయాలనుకుంటారు. మీ యూజర్ MCPని ఉపయోగించడం వల్ల ప్రతిభా నిల్వ చేసే 여부 పట్టించుకోలేదు కానీ స్వాభావిక భాషను ఉపయోగించి ఇంటరాక్ట్ చేయాలని ఆశిస్తూ ఉంటాడు. ఆ కాబట్టి దీనికి పరిష్కారం ఏమిటి? పరిష్కారం క్లయింట్‌కు LLMని జోడించడం.

## అవలోకనం

ఈ పాఠంలో మనం క్లయింట్‌కు LLMని జోడించడం పై దృష్టిసారించి, ఇది మీ యూజర్‌కు ఎలా మెరుగైన అనుభవం అందిస్తుందో చూపుతాము.

## అభ్యాస లక్ష్యాలు

ఈ పాఠం చివరికి, మీరు:

- LLMతో కూడిన క్లయింట్ సృష్టించగలరు.
- MCP సర్వర్‌తో LLM ఉపయోగించి సులభంగా ఇంటరాక్ట్ చేయగలరు.
- క్లయింట్ వైపు మెరుగైన ఎండ్-యూజర్ అనుభవం అందించగలరు.

## దృష్టికోణం

మనము తీసుకోవాల్సిన దృష్టికోణాన్ని అర్థం చేసుకోగలుగుదాం. LLM జోడించడం సులభం అనిపిస్తే, మనము దీన్ని ఎలా అమలు చేస్తాము?

క్లయింట్ సర్వర్‌తో ఎలా ఇంటరాక్ట్ చేస్తుందో చూపుతున్నది:

1. సర్వర్‌తో కనెక్షన్ స్థాపించండి.

1. సామర్థ్యాలు, ప్రాంప్ట్స్, వనరులు, మరియు టూల్స్ జాబితా చేయండి, వాటి స్కీమాను సేవ్ చేసుకోండి.

1. ఒక LLMని జోడించి సేవ్ చేసిన సామర్థ్యాలు మరియు వారి స్కీమాను LLM అర్థం చేసుకునే ఫార్మాట్‌లో అందించండి.

1. యూజర్ ప్రాంప్ట్‌ను తీసుకుని, క్లయింట్ జాబితా చేసిన టూల్స్‌తో పాటు LLMకు అందించండి.

బాగుంది, ఇప్పుడు మనం దీన్ని ఉన్నత స్థాయిలో ఎలా చేయాలో అర్థం చేసుకున్నాము, క్రిందని వ్యాయామంలో దీన్ని ప్రయత్నిద్దాం.

## వ్యాయామం: LLMతో కూడిన క్లయింట్ సృష్టించడం

ఈ వ్యాయామంలో మనము మా క్లయింట్‌కు LLMని ఎలా జోడించాలో నేర్చుకొంటాము.

### GitHub Personal Access Token ఉపయోగించి autentikēṣan

GitHub టోకెన్ సృష్టించడం ఒక సరళమైన ప్రక్రియ. దీన్ని ఎలా చేయాలో చూడండి:

- GitHub సెట్టింగ్స్‌కు వెళ్లండి – పై కుడి మూలలో ఉన్న మీ ప్రొఫైల్ చిత్రాన్ని క్లిక్ చేసి సెట్టింగ్స్ సెలెక్ట్ చేయండి.
- Developer Settings వైపు వెళ్లండి – క్రింద వరకు స్క్రోల్ చేసి Developer Settings క్లిక్ చేయండి.
- Personal Access Tokens ఎంచుకోండి – Fine-grained tokens ఎంచుకుని Generate new token క్లిక్ చేయండి.
- మీ టోకెన్‌ను కాంఫిగర్ చేయండి – సూచన కోసం ఒక నోట్ జోడించండి, గడువు తేదీ అమర్చండి మరియు అవసరమైన స్కోప్స్ (అనుమతులు) ఎంచుకోండి. ఈ సందర్భంలో Models అనుమతిని తప్పనిసరిగా జోడించండి.
- టోకెన్‌ను సృష్టించి కాపీ చేయండి – Generate token క్లిక్ చేసి వెంటనే దాన్ని కాపీ చేసుకోండి, ఎందుకంటే అది మరోసారి కనిపించదు.

### -1- సర్వర్‌కు కనెక్ట్ కావడం

ముందుగా మనం మా క్లయింట్‌ను సృష్టిద్దాం:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // స్కీమా ధృవీకరణ కోసం జడ్‌ను దిగుమతి చేసుకోండి

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

పైన ఉన్న కోడ్‌లో:

- అవసరమైన లైబ్రరీలు దిగుమతి చేసుకున్నాము
- `client` మరియు `openai` అనే రెండు సభ్యులతో కూడిన ఒక క్లాస్ సృష్టించాము, ఇది మాకు క్లయింట్‌ను నిర్వహించడంలో మరియు LLMతో ఇంటరాక్ట్ చేయడంలో సహాయం చేస్తుంది.
- GitHub మోడల్స్ ఉపయోగించడానికి `baseUrl`ని inference APIకై సెటప్ చేసి మన LLM ఇన్‌స్టన్స్‌ను కన్ఫిగర్ చేసుకున్నాము.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio కనెక్షన్ కోసం సర్వర్ పారామితులు సృష్టించండి
server_params = StdioServerParameters(
    command="mcp",  # కార్యాచరణ
    args=["run", "server.py"],  # ఐచ్ఛిక కమాండ్ లైన్ ఆర్గ్యుమెంట్లు
    env=None,  # ఐచ్ఛిక వాతావరణ వేరియబుల్స్
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # కనెక్షన్‌ను ప్రారంభించండి
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

పైన ఉన్న కోడ్‌లో:

- MCPకి అవసరమైన లైబ్రరీలు దిగుమతి చేసుకున్నాము
- ఒక క్లయింట్ సృష్టించాము

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

ముందుగా, మీరు మీ `pom.xml` ఫైల్‌లో LangChain4j డిపెండెన్సీలు జోడించాలి. MCP ఇంటిగ్రేషన్ మరియు GitHub మోడల్స్ సపోర్ట్ కోసం ఈ డిపెండెన్సీలు జోడించండి:

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

తర్వాత మీ Java క్లయింట్ క్లాస్ని సృష్టించండి:

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
    
    public static void main(String[] args) throws Exception {        // GitHub మోడల్స్ ఉపయోగించడానికి LLMను ఆకుపెట్టండి
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // సర్వర్‌కి కనెక్ట్ అయ్యేందుకు MCP ట్రాన్స్‌పోర్ట్ సృష్టించండి
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP క్లయింట్ సృష్టించండి
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

పైన ఉన్న కోడ్‌లో:

- **LangChain4j డిపెండెన్సీలు జోడించబడ్డవి**: MCP ఇంటిగ్రేషన్, OpenAI అధికారిక క్లయింట్, మరియు GitHub మోడల్స్ మొదలయినవి అవసరమైనవి
- **LangChain4j లైబ్రరీలు దిగుమతి చేసుకున్నాం**: MCP ఇంటిగ్రేషన్ మరియు OpenAI చాట్ మోడల్ ఫంక్షనాలిటీ కోసం
- **`ChatLanguageModel` సృష్టించాము**: GitHub మోడల్స్‌తో మీ GitHub టోకెన్తో కన్ఫిగర్ చేసినది
- **HTTP ట్రాన్స్‌పోర్ట్ సెటప్ చేసినాము**: MCP సర్వర్‌కు కనెక్ట్ కావడానికి Server-Sent Events (SSE) ఉపయోగిస్తూ
- **MCP క్లయింట్ సృష్టించాము**: ఇది సర్వర్‌తో కమ్యూనికేషన్ నిర్వహిస్తుంది
- **LangChain4j యొక్క MCP సపోర్ట్ ఉపయోగించాము**: ఇది LLMs మరియు MCP సర్వర్లు మధ్య ఇంటిగ్రేషన్ సులభతరం చేస్తుంది

#### Rust

ఈ ఉదాహరణలో మీరు Rust ఆధారిత MCP సర్వర్‌ను రన్ చేస్తున్నారని భావిస్తున్నాము. మీ దగ్గర ఒకటి లేకపోతే, [01-first-server](../01-first-server/README.md) పాఠం తెలీని సర్వర్ సృష్టించండి.

మీ Rust MCP సర్వర్ సిద్ధంగా ఉన్నప్పుడు, ఒక టెర్మినల్ తెరవండి మరియు సర్వర్ ఉన్న ఫోల్డర్‌కు వెళ్లండి. తరువాత క్రింది ఆజ్ఞను నడిపి కొత్త LLM క్లయింట్ ప్రాజెక్టును సృష్టించండి:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

మీ `Cargo.toml` ఫైల్‌లో క్రింది డిపెండెన్సీలను జోడించండి:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAIకి అధికారిక రస్ట్ లైబ్రరీ లేదు, కానీ `async-openai` క్రేట్ ఒక [సమాజం నిర్వహించే లైబ్రరీ](https://platform.openai.com/docs/libraries/rust#rust) మరియు దీనిని ప్రాచుర్యం పొందింది.

`src/main.rs` ఫైల్ తెరిచి దీని కంటెంట్ ను క్రింది కోడ్‌తో మార్చండి:

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
    // ప్రారంభ సందేశం
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI క్లయింట్ సెటప్ చేయండి
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP క్లయింట్ సెటప్ చేయండి
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

    // చేయాల్సింది: MCP టూల్ జాబితా పొందండి

    // చేయాల్సింది: టూల్ కాల్స్‌తో LLM సంభాషణ చేయండి

    Ok(())
}
```

ఈ కోడ్ ఒక ప్రాథమిక Rust అప్లికేషన్ సెట్ అప్ చేస్తుంది, ఇది MCP సర్వర్ మరియు GitHub మోడల్స్‌తో LLM ఇంటరాక్షన్ల కోసం కనెక్ట్ అవుతుంది.

> [!IMPORTANT]
> అప్లికేషన్ నడపడానికి ముందు `OPENAI_API_KEY` ఎన్విరాన్‌మెంట్변수를 మీ GitHub టోకెన్‌తో సెటప్ చేయడం మరచిపోకండి.

మంచిది, తదుపరి దశకు మనము సర్వర్‌పైని సామర్థ్యాలను జాబితా చేద్దాం.

### -2- సర్వర్ సామర్థ్యాలను జాబితా చేయడం

ఇప్పుడు మేము సర్వర్‌కు కనెక్ట్ అయి దాని సామర్థ్యాలను అడుగుతాము:

#### Typescript

అనే క్లాస్‌లో క్రింది మెథడ్స్ జోడించండి:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // పరికరాలను జాబితా చేయడం
    const toolsResult = await this.client.listTools();
}
```

పైన ఉన్న కోడ్‌లో:

- సర్వర్‌కు కనెక్ట్ అయ్యే కోడ్ `connectToServer` ను జోడించాము.
- `run` అనే ఒక మెథడ్ సృష్టించాము, ఇది మన యాప్ ఫ్లో నిర్వహిస్తుంది. ప్రస్తుతం ఇది కేవలం టూల్స్‌ను జాబితా చేస్తోంది కానీ త్వరలో మరిన్ని జోడించబోతున్నాము.

#### Python

```python
# అందుబాటులో ఉన్న వనరులను జాబితా చేయండి
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# అందుబాటులో ఉన్న పరికరాలను జాబితా చేయండి
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

ఇక్కడ మనం జోడించినవి:

- వనరులు మరియు టూల్స్ జాబితా చేసి ముద్రించాము. టూల్స్ కోసం `inputSchema` కూడా జాబితా చేశాము, ఇది తరవాత ఉపయోగిస్తాం.

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

పైన ఉన్న కోడ్‌లో:

- MCP సర్వర్‌లో అందుబాటులో ఉన్న టూల్స్‌ను జాబితా చేశాము
- ప్రతి టూల్ విషయంలో పేరు, వివరణ మరియు దాని స్కీమా జాబితా చేసాము. ఇది త్వరలో టూల్స్ కాల్ చేయడానికి ఉపయోగపడుతుంది.

#### Java

```java
// MCP టూల్స్‌ను ఆటోమేటిక్‌గా కనుగొనే ఒక టూల్ ప్రొవైడర్‌ను సృష్టించండి
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP టూల్ ప్రొవైడర్ ఆటోమేటిక్‌గా నిర్వహిస్తుంది:
// - MCP సర్వర్ నుండి అందుబాటులో ఉన్న టూల్స్ జాబితా చేయడం
// - MCP టూల్ స్కీమాలను LangChain4j ఫార్మాట్‌కు మార్చడం
// - టూల్ అమలు మరియు ప్రతిస్పందనలను నిర్వహించడం
```

పైన ఉన్న కోడ్‌లో:

- అన్ని MCP సర్వర్ టూల్స్‌ను ఆటోమేటిగ్గా కనుగొని రిజిస్టర్ చేసే `McpToolProvider` సృష్టించాము
- టూల్ ప్రొవైడర్ MCP టూల్ స్కీమాలతో LangChain4j టూల్ ఫార్మాట్ మార్చుకునే పని పూర్తిచేస్తుంది
- ఈ విధానం మానవీయంగా టూల్స్ జాబితా చేయడం మరియు మార్పిడి ప్రక్రియను దాచేస్తుంది

#### Rust

MCP సర్వర్ నుండి టూల్స్ తిరిగి పొందుటకు `list_tools` మెథడ్ ఉపయోగించబడుతుంది. మీ `main` ఫంక్షన్‌లో MCP క్లయింట్ సెట్ చేసిన తరువాత క్రింది కోడ్ జోడించండి:

```rust
// MCP టూల్ జాబితాను పొందండి
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- సర్వర్ సామర్థ్యాలను LLM టూల్స్‌గా మార్చడం

సర్వర్ సామర్థ్యాలను జాబితా చేసిన తర్వాత తర్వాతి దశ LLM అర్థం చేసుకునే ఫార్మాట్‌లో మార్చడం. ఒకసారి మేము దీన్ని చేస్తే, ఆ సామర్థ్యాలను ఏకంగా LLM కు టూల్స్‌గా అందించవచ్చు.

#### TypeScript

1. MCP సర్వర్ నుండి వచ్చిన ప్రతిస్పందనను LLM ఉపయోగించగల టూల్ ఫార్మాట్‌గా మార్చేందుకు క్రింది కోడ్ జోడించండి:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ఇన్పుట్_స్కీమా ఆధారంగా జోడ్ స్కీమా రూపొందించండి
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // టైప్ ను స్పష్టంగా "ఫంక్షన్" గా సెట్ చేయండి
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

    పై కోడ్ MCP సర్వర్ నుండి వచ్చిన ప్రతిస్పందనను LLM అర్థం చేసుకునే టూల్ డెఫినిషన్ ఫార్మాట్లోకి మార్చుతుంది.

1. ఇప్పుడေတာ့ `run` మెథడ్‌ను అప్డేట్ చేసి సర్వర్ సామర్థ్యాలను జాబితా చేద్దాం:

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

    పైన ఉన్న కోడ్‌లో, `run` మెథడ్‌లో ఫలితాన్ని మ్యాప్ చేసి ప్రతి ఎంట్రీకి `openAiToolAdapter` కాల్ చేశాము.

#### Python

1. ముందుగా, క్రింది కన్వర్టర్ ఫంక్షన్ సృష్టించుకుందాం

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

    `convert_to_llm_tools` అనే ఫంక్షన్‌లో MCP టూల్ ప్రతిస్పందనను LLM అర్థం చేసుకునే ఫార్మాట్‌కి మార్చుతాం.

1. తదనంతరం, మా క్లయింట్ కోడ్‌లో ఈ ఫంక్షన్ ఉపయోగించేందుకు ఇలా చేయండి:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    ఇక్కడ, MCP టూల్ ప్రతిస్పందనను LLMకు అందించదలచిన ఫార్మాట్‌గా మార్చేందుకు `convert_to_llm_tool`ను పిలుచుకుంటున్నాము.

#### .NET

1. MCP టూల్ ప్రతిస్పందనను LLM అర్థం చేసుకునే రూపంలోకి మార్చేందుకు క్రింది కోడ్ జోడించండి:

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

పైన ఉన్న కోడ్‌లో:

- `ConvertFrom` అనే ఫంక్షన్ సృష్టించాము, ఇది పేరు, వివరణ, ఇన్‌పుట్ స్కీమాను తీసుకుంటుంది.
- ఒక `FunctionDefinition` సృష్టించి, అది `ChatCompletionsDefinition`కు పంపబడుతుంది, ఇది LLMకు అర్థమయ్యేది.

1. ఇప్పుడు ఈ ఫంక్షన్ ఉపయోగించి కొంత మునుపటి కోడ్‌ను ఎలా మార్చాలో చూద్దాం:

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
// సహజ భాషా పరస్పర చర్య కోసం బాట్ ఇంటర్‌ఫేస్ సృష్టించండి
public interface Bot {
    String chat(String prompt);
}

// LLM మరియు MCP టూల్స్‌తో AI సేవను కాన్ఫిగర్ చేయండి
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

పైన ఉన్న కోడ్‌లో:

- సహజ భాషా ఇంటరాక్షన్ల కోసం ఒక సరళ `Bot` ఇంటర్‌ఫేస్ నిర్వచించాం
- MCP టూల్ ప్రొవైడర్‌తో LLM ఆటోమేటిగ్గా బైండ్ చేసే LangChain4j యొక్క `AiServices` ఉపయోగించాం
- ఫ్రేమ్‌వర్క్ స్వయంచాలకంగా టూల్ స్కీమా మార్పిడి మరియు ఫంక్షన్ కాలింగ్ నిర్వహిస్తుంది
- ఈ విధానం మానవీయ టూల్ మార్పిడి అవసరాన్ని తొలగిస్తుంది - LangChain4j MCP టూల్స్‌ను LLMకి అనుకూలమైన ఫార్మాట్‌గా మార్చడంలో అన్ని క్లిష్టతలను నిర్వహిస్తుంది

#### Rust

MCP టూల్ ప్రతిస్పందనను LLM అర్థం చేసుకునే ఫార్మాట్‌కు మార్చేందుకు సహాయక ఫంక్షన్ జోడించాలి, ఇది టూల్స్ జాబితాను ఫార్మాటింగ్ చేస్తుంది. క్రింద కోడ్‌ను `main.rs` లో `main` ఫంక్షన్ కింద జోడించండి. ఇది LLMకి అభ్యర్థనలు పంపేటప్పుడు పిలవబడుతుంది:

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

మంచిది, యూజర్ అభ్యర్థనలను నిర్వహించేందుకు మనము సిద్ధంగా ఉన్నాము, కాబట్టి తదుపరి దశ చేద్దాం.

### -4- యూజర్ ప్రాంప్ట్ అభ్యర్థనను నిర్వహించడం

ఈ భాగంలో మేము యూజర్ అభ్యర్థనలను పర్యవేక్షిస్తాము.

#### TypeScript

1. LLMను పిలవడానికి ఉపయోగించే మెథడ్ జోడించండి:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. సర్వర్ టూల్‌ను కాల్ చేయండి
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ఫలితంతో ఏదైనా చేయండి
        // చేయవలసింది

        }
    }
    ```

    పైన ఉన్న కోడ్‌లో:

    - `callTools` అనే మెథడ్ జోడించాము.
    - ఈ మెథడ్ LLM ప్రతిస్పందనను తీసుకుని, ఏ టూల్స్ కాల్ అయినాయో పరిశీలిస్తుంది:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // టూల్‌ను పిలవండి
        }
        ```

    - LLM సూచించినప్పుడు ఒక టూల్‌ని పిలుస్తుంది:

        ```typescript
        // 2. సర్వర్ టూల్‌ను కాల్ చేయండి
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ఫలితంతో ఏదో చేయండి
        // చేయాల్సింది
        ```

1. `run` మెథడ్‌ను అప్డేట్ చేసి LLM మరియు `callTools` కాలింగ్‌ని చేర్చండి:

    ```typescript

    // 1. LLM కోసం ఇన్‌పుట్ అయిన సందేశాలను సృష్టించండి
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM ని పిలవడం
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM ప్రతిస్పందనను అనుసంధానించండి, ప్రతి ఎంపిక కోసం, అది టూల్ కాల్స్ ఉన్నదో చూడండి
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

మంచిది, పూర్తి కోడ్ ఇక్కడ ఉంది:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // స్కీము ఆమోదం కోసం జాడ్ ని దిగుమతి చేసుకోండి

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // భవిష్యత్తులో ఈ URL కి మార్పు అవసరం కావచ్చు: https://models.github.ai/inference
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
          // ఇన్పుట్ స్కీము ఆధారంగా ఒక జాడ్ స్కీము సృష్టించండి
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // టైపు ను స్పష్టంగా "ఫంక్షన్" గా సెట్ చేయండి
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
    
    
          // 2. సర్వర్ యొక్క టూల్ ను పిలవండి
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. ఫలితంతో ఏదో చేయండి
          // చేయవలసింది
    
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
    
        // 1. LLM ప్రతిస్పందనలో ప్రతి ఎంచుకొన్నదాన్ని పరిశీలించి, అది టూల్ పిలుపులను కలిగుందో చూడండి
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

1. LLMని పిలవటానికి అవసరమైన కొన్ని దిగుమతులు జోడించండి

    ```python
    # ఎల్ఎల్ఎమ్
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. తదనంతరం, LLMను పిలవడానికి ఫంక్షన్ జోడించండి:

    ```python
    # ఎల్ఎల్‌ఎమ్

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
            # ఐచ్ఛిక పరామితులు
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

పైన ఉన్న కోడ్‌లో:

- మనము MCP సర్వర్‌పై కనిపించిన మరియు మార్చిన ఫంక్షన్లను LLMకు పంపాము.
- ఆపై, ఆ ఫంక్షన్లతో LLMని పిలిచాము.
- ఫలితాన్ని పరిశీలించి, ఏ ఫంక్షన్లను పిలవాలో చూసాము.
- చివరిగా పిలవవలసిన ఫంక్షన్ల జాబితాను ఇచ్చాము.

1. చివరి దశగా, మన ప్రధాన కోడ్‌ను అప్డేట్ చేద్దాం:

    ```python
    prompt = "Add 2 to 20"

    # ఎల్ఎల్ఎమ్‌కు ఏ టూల్స్ ఉన్నాయో, ఉంటే ఎవరికి వేసేందుకు అడగండి
    functions_to_call = call_llm(prompt, functions)

    # సూచించబడిన ఫంక్షన్లను కాల్ చేయండి
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

అది చివరి దశ, పైన కోడ్‌లో:

- LLM మన ప్రాంప్ట్ అనుసారం ఫంక్షన్ పిలవవలసిన భావించి `call_tool` ద్వారా MCP టూల్‌ను పిలుస్తోంది.
- టూల్ కాల్ ఫలితాన్ని MCP సర్వర్‌కు ముద్రిస్తున్నాము.

#### .NET

1. LLM ప్రాంప్ట్ అభ్యర్థన కోసం కొన్ని కోడ్ చూపిద్దాం:

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

పైన ఉన్న కోడ్‌లో:

- MCP సర్వర్ నుంచి టూల్స్‌ను తీసుకున్నాము, `var tools = await GetMcpTools()`.
- యూజర్ ప్రాంప్ట్ `userMessage` నిర్వచించాం.
- మోడల్ మరియు టూల్స్ ఎంచుకుని options ఆబ్జెక్ట్ సృష్టించాము.
- LLMకి అభ్యర్థన పంపాము.

1. చివరి దశగా, LLM ఫంక్షన్ కాల్ అవసరమైందా అని చూడండి:

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

పైన ఉన్న కోడ్‌లో:

- ఫంక్షన్ కాల్స్ జాబితా మీద లూప్ వేసాము.
- ప్రతి టూల్ కాల్ కోసం పేరు మరియు ఆర్గ్యుమెంట్స్ పార్స్ చేసి MCP క్లయింట్ ఉపయోగించి MCP సర్వర్上的 టూల్ పిలిచాం. చివరగా ఫలితాన్ని ప్రింట్ చేసాము.

పూర్తి కోడ్ ఇక్కడ ఉంది:

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
    // MCP టూల్స్ ను ఆటోమేటిక్‌గా ఉపయోగించి సహజ భాషా అభ్యర్థనలు అమలు చేయండి
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

పైన ఉన్న కోడ్‌లో:

- సహజ భాష ప్రాంప్ట్స్ ఉపయోగించి MCP సర్వర్ టూల్స్‌తో ఇంటరాక్ట్ చేశాము
- LangChain4j ఫ్రేమ్‌వర్క్ ఆటోమేటిగ్గా:

  - అవసరమైనప్పుడు యూజర్ ప్రాంప్ట్స్‌ను టూల్ కాల్స్‌గా మార్చడం
  - LLM నిర్ణయానికి అనుగుణంగా సరైన MCP టూల్స్ పిలవడం
  - LLM మరియు MCP సర్వర్ మధ్య సంభాషణను నిర్వహించడం

- `bot.chat()` మాధ్యమంగా సహజ భాషా ప్రతిస్పందనలు వస్తాయి, వాటిలో MCP టూల్‌లు అమలు చేసిన ఫలితాలు ఉంటాయి
- ఈ విధానం వినియోగదారులకు MCP అమలు గురించి తెలియకుండా సహజ అనుభవాన్ని అందిస్తుంది

పూర్తి కోడ్ ఉదాహరణ:

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

ఇక్కడ ప్రధానంగా పని జరుగుతుంది. మొదటి యూజర్ ప్రాంప్ట్‌తో LLMను పిలిచి, ఫలితాన్ని పరిశీలించి ఏ టూల్స్ పిలవాలన్నది నిర్ణయిస్తాము. అవసరమైతే ఆ టూల్స్‌ను పిలిచి, మరలా LLMతో సంభాషణ కొనసాగిస్తాం, ఇంకా టూల్స్ పిలవాల్సిన అవసరం లేకపోయేవరకు మరియు మనకు తుది ఫలితం వచ్చే వరకు.

మనం చాలాసార్లు LLMను పిలుస్తున్నాం, కాబట్టి LLM కాల్ నిర్వహించే ఒక ఫంక్షన్ నిర్వచించుదాం. క్రింద ఉన్న దాన్ని `main.rs` లో జోడించండి:

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

ఈ ఫంక్షన్ LLM క్లయింట్, సందేశాల జాబితా (యూజర్ ప్రాంప్ట్ సహా), MCP సర్వర్ నుంచి టూల్స్ తీసుకుని LLMకి అభ్యర్థన పంపి ఫలితాన్ని తిరిగి ఇస్తుంది.
LLM నుండి వచ్చిన ప్రతిస్పందనలో `choices` అనే శ్రేణి ఉంటుంది. ఎలాంటి `tool_calls` ఉన్నాయో లేదో పరీక్షించడానికి ఫలితాన్ని ప్రాసెస్ చేయాల్సి ఉంటుంది. దీని ద్వారా LLM ఒక నిర్దిష్ట టూల్ ఆహ్వానానికి వాదనలు సహితం పిలవాలనుకుంటున్నట్టు తెలిసిపోతుంది. LLM ప్రతిస్పందనను హ్యాండిల్ చేయడానికి దిగువ కోడ్‌ను మీ `main.rs` ఫైల్ చివర జతచేయండి:

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

    // విషయం అందుబాటులో ఉంటే ముద్రించండి
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // పరికర కాల్స్ ను నిర్వహించండి
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // సహాయక సందేశాన్ని జోడించండి

        // ప్రతి పరికర కాల్ ని అమలు చేయండి
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // పరికర ఫలితాన్ని సందేశాలకు జోడించండి
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // పరికర ఫలితాలతో సంభాషణ కొనసాగించండి
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
  
`tool_calls` ఉంటే, అది టూల్ సమాచారాన్ని తీసుకొని, MCP సర్వర్‌ను టూల్ అభ్యర్థనతో పిలుస్తుంది, మరియు ఫలితాలను సంభాషణ సందేశాలలో చేర్చేది. తర్వాత LLM తో సంభాషణ కొనసాగిస్తూ, అసిస్టెంట్ ప్రతిస్పందన మరియు టూల్ కాల్ ఫలితాలతో సందేశాలు నవీకరించబడతాయి.

MCP కాల్స్ కోసం LLM ఇచ్చే టూల్ కాల్ సమాచారాన్ని తీసుకోవడానికి మేము మరో హెల్పర్ ఫంక్షన్‌ను జతచేయబోతున్నాము, ఇది కాల్ కోసం కావలసిన అన్ని అంశాలను తీసుకుంటుంది. మీ `main.rs` ఫైల్ చివర ఈ క్రింది కోడ్‌ను జతచేయండి:

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
  
అన్ని భాగాలు సిద్ధంగా ఉన్నందున, ప్రారంభ యూజర్ ప్రాంప్ట్‌ను హ్యాండిల్ చేసి LLM ను పిలవచ్చు. మీ `main` ఫంక్షన్‌ను క్రింద ఇవ్వబడిన కోడ్‌తో నవీకరించండి:

```rust
// టూల్ కాల్లతో LLM సంభాషణ
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
  
దీని ద్వారా LLM ను ప్రారంభ యూజర్ ప్రాంప్ట్‌తో క్వెరీ చేసి, రెండు సంఖ్యల మొత్తం కోసం అడుగుతుంది, మరియు ప్రతిస్పందనను ప్రాసెస్ చేసి డైనమిక్‌గా టూల్ కాల్స్ ను హ్యాండిల్ చేస్తుంది.

చాలా బాగుంది, మీరు పూర్తి చేసారు!

## అసైన్‌మెంట్

వ్యాయామం నుండి కోడ్ తీసుకొని మరికొన్ని టూల్స్ తో సర్వర్ ను అభివృద్ధి చేయండి. తర్వాత వ్యాయామంలో లాగా LLM కలిగిన క్లయింట్‌ను సృష్టించి, వేర్వేరు ప్రాంప్ట్‌లతో పరీక్షించి మీ సర్వర్ టూల్స్ డైనమిక్‌గా పిలవబడుతున్నాయో చూడండి. ఈ విధంగా క్లయింట్ రూపకల్పన తో ఎండ్ యూజర్ కు మంచి అనుభవం వస్తుంది, ఎందుకంటే వారు ఖచ్చితమైన క్లయింట్ కమాండ్‌ ల కన్నా ప్రాంప్ట్ లతో పని చేస్తారు, మరియు MCP సర్వర్ పిలవబడుతున్నట్టు అప్రమత్తం కాకుండా ఉంటారు.

## పరిష్కారం

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## ముఖ్యమైన పాఠాలు

- మీ క్లయింట్‌లో LLM చేర్చడం MCP సర్వర్లతో వాడుకదారులు కలిసి పనిచేయడంలో మెరుగైన మార్గాన్ని అందిస్తుంది.
- MCP సర్వర్ నుండి వచ్చే ప్రతిస్పందనను LLM అర్ధం చేసుకునే రూపంలో మార్చుకోవాలి.

## నమూనాలు

- [Java కేలిక్యులేటర్](../samples/java/calculator/README.md)
- [.Net కేలిక్యులేటర్](../../../../03-GettingStarted/samples/csharp)
- [JavaScript కేలిక్యులేటర్](../samples/javascript/README.md)
- [TypeScript కేలిక్యులేటర్](../samples/typescript/README.md)
- [Python కేలిక్యులేటర్](../../../../03-GettingStarted/samples/python)
- [Rust కేలిక్యులేటర్](../../../../03-GettingStarted/samples/rust)

## అదనపు వనరులు

## తదుపరి ఎమిటి

- తరువాత: [Visual Studio Code ఉపయోగించి సర్వర్‌ను వినియోగించడం](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**సాట్**:
ఈ డాక్యుమెంట్‌ను AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ద్వారా అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నా, ఆటోమేటెడ్ అనువాదాల్లో పొరపాట్లు లేదా అసమర్ధతలు ఉండవచ్చు. ఆ ప్రామాణిక సమాచారం కోసం, సొంత భాషలో ఉన్న అసలు డాక్యుమెంట్‌ను ఆధారమైన వనరుగా పరిగణించాలి. అత్యవసరమైన సమాచారం కోసం, నిపుణుల చేత అనువాదం సిఫార్సు చేయబడుతుంది. ఈ అనువాదం వాడకంలో వచ్చే ఎటువంటి అసౌకర్యాలు లేదా తప్పుదారులు కోసం మేము బాధ్యులేమం.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
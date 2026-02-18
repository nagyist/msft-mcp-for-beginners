# LLM ဖြင့် client ဖန်တီးခြင်း

ယခုအချိန်ထိ၊ သင်သည် server နှင့် client ကို ဘယ်လိုဖန်တီးရမည်ကို မြင်တွေ့ခဲ့ပြီဖြစ်သည်။ Client သည် server ကို တိုက်ရိုက် ခေါ်ဆို၍ ၎င်း၏ tools, resources နှင့် prompts များကို စာရင်းပြုလုပ်နိုင်ခဲ့သည်။ သို့သော် ၎င်းသည် အလွန်အသုံးဝင်သောနည်းလမ်း မဟုတ်ပါ။ သင့်အသုံးပြုသူသည် agentic ခေတ်တွင် နေထိုင်ပြီး prompt များကို အသုံးပြု၍ LLM နှင့် ဆက်သွယ်ချင်ကြောင်းမျှော်လင့်သည်။ သင့်အသုံးပြုသူအတွက် MCP ကို သုံးရန်ရှိခြင်းမရှိခြင်း မကြာလှပါ၊ သင်သည် သဘာဝဘာသာစကားဖြင့် ဆက်သွယ်ခြင်းကို မျှော်လင့်သည်။ ဒါဆို ဤပြဿနာကို ဘယ်လို ဖြေရှင်းရမည်နည်း? ဖြေရှင်းနည်းမှာ client ထဲတွင် LLM တစ်ခု ထည့်သွင်းပေးခြင်း ဖြစ်သည်။

## အနှစ်ချုပ်

ဤသင်ခန်းစဉ်တွင် client တွင် LLM တစ်ခု ထည့်သွင်းပေးခြင်းအပေါ် အာရုံစိုက်ကာ သုံးစွဲသူအတွက် ပိုမိုကောင်းမွန်သောအတွေ့အကြုံကို မည်သို့ ပံ့ပိုးပေးနိုင်သည်ကို ပြသပေးမည်။

## သင်ယူရမည့် ရည်မှန်းချက်များ

ဤသင်ခန်းစဉ်အဆုံးတွင် သင်သည် -

- LLM တစ်ခုပါရှိသော client တစ်ခု ဖန်တီးနိုင်မည်။
- LLM ကိုအသုံးပြု၍ MCP server နှင့် ချိတ်ဆက်မည်။
- client ဘက်တွင် သုံးစွဲသူအတွက် ပိုမိုကောင်းမွန်သောအတွေ့အကြုံ ပံ့ပိုးပေးနိုင်မည်။

## နည်းလမ်း

လိုအပ်သော နည်းလမ်းကို နားလည်ကြည့်ကြပါစို့။ LLM တစ်ခု ထည့်ခြင်းသည် ရိုးရှင်းသော်လည်း သင်သည် အဖြစ်တည်ဆာပသလား?

client သည် server နှင့် အောက်ပါနည်းလမ်းဖြင့် ဆက်သွယ်ပါမည် -

1. server နှင့် ချိတ်ဆက်မှုတည်ဆောက်ခြင်း။

1. ဖြစ်နိုင်မှုများ၊ prompt များ၊ resource များနှင့် tool များကို စာရင်းပြုစုပြီး ၎င်း၏ schema ကို သိမ်းဆည်းခြင်း။

1. LLM တစ်ခု ထပ်ထည့်ပြီး သိမ်းထားသော ဖြစ်နိုင်မှုများနှင့် ၎င်း၏ schema ကို LLM ကနားလည်သည့် အချိန်ရောက်ဖော်စပ်မှုဖြင့် လွှဲပြောင်းပေးခြင်း။

1. user prompt ကို client listing တွင်ပါရှိသော tools များနှင့်အတူ LLM သို့ ပေးပို့ကာ ကိုင်တွယ်ခြင်း။

အဆင်ပြေပါပြီ၊ အဆင့်မြင့်အဆင့်တွင် ဘယ်လိုလုပ်ကြမယ်ဆိုတာ နားလည်သွားပြီ၊ အောက်တွင် လေ့ကျင့်ခန်းအား စမ်းသပ်ကြည့်ကြရအောင်။

## လေ့ကျင့်ခန်း - LLM ပါရှိသော client ဖန်တီးခြင်း

ဤ လေ့ကျင့်ခန်းတွင် client တွင် LLM တစ်ခု ထည့်သွင်းရွယ်သည်ကို သင်ယူပါမည်။

### GitHub Personal Access Token အသုံးပြုခြင်းမှ သက်သေပြုခြင်း

GitHub token ဖန်တီးခြင်းမှာ လွယ်ကူသည်။ အောက်ပါအတိုင်းလုပ်ဆောင်ရမည်။

- GitHub Settings သို့ သွားမည် - မျက်နှာပြင်အထက် ညာဘက်ထောင့်ရှိ သင့်ပရိုဖိုင်ပုံကို နှိပ်၍ Settings ရွေးချယ်ပါ။
- Developer Settings သို့ သွားရောက်မည် - အောက်သို့ ဆွဲချပြီး Developer Settings ကိုနှိပ်ပါ။
- Personal Access Tokens ကို ရွေးချယ်ပါ – Fine-grained tokens ကိုနှိပ်ပြီး Generate new token ကိုနှိပ်ပါ။
- သင့် token ကို ဖော်ပြရန်အတွက် မှတ်စုတစ်ခု ထည့်ပြီး သက်တမ်း သတ်မှတ်ခြင်း၊ လိုအပ်သော scopes (ခွင့်ပြုချက်များ) ကို ရွေးပါ။ ဤကိစ္စတွင် Models ခွင့်ပြုချက် ထည့်သွင်းရန် သေချာပါစေ။
- Generate token ကိုနှိပ်ပြီး ထုတ်ယူပါ၊ ၎င်းကို ချက်ချင်း ကူးယူထားခြင်း မရှိမဖြစ်လိုအပ်သည်၊ ပြန်မမြင်ရတော့ပဲဖြစ်နိုင်ပါသည်။

### -1- Server နှင့် ချိတ်ဆက်ခြင်း

client ကို ပထမဆုံး ဖန်တီးကြပါစို့။

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // schema စစ်ဆေးမှုအတွက် zod ကို आयातပါ။

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

အထက်ပါ ကုဒ်တွင် ကျွန်ုပ်တို့ -

- လိုအပ်သော libraries များကို တင်သွင်းထားသည်။
- `client` နှင့် `openai` ဆိုသော สมาชิกနှစ်ဦးပါရှိသော class တစ်ခု ဖန်တီးထားသည်၊ ၎င်းတို့သည် client ကို စီမံခန့်ခွဲခြင်းနှင့် LLM နှင့် ဆက်သွယ်ရန် ကူညီသည်။
- `baseUrl` ကို inference API သို့ညွှန်ပြပြီး GitHub Models အသုံးပြုရန် သတ်မှတ်ထားသော LLM instance ကို ပြင်ဆင်ထားသည်။

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio ချိတ်ဆက်မှုအတွက် ဆာဗာ ပါရာမီတာများ ဖန်တီးပါ
server_params = StdioServerParameters(
    command="mcp",  # အလုပ်လုပ်နိုင်သော
    args=["run", "server.py"],  # ရွေးချယ်သည့် command line အရာများ
    env=None,  # ရွေးချယ်သည့် ပတ်ဝန်းကျင်ဗားရီယာများ
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # ချိတ်ဆက်မှုကို စတင်တည်ဆောက်ပါ
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

အထက်ပါကုဒ်တွင် -

- MCP အတွက် လိုအပ်သော libraries များကို တင်သွင်းထားသည်။
- client တစ်ခု ဖန်တီးထားသည်။

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

ပထမဦးဆုံး LangChain4j မူလတန်းသတ်မှတ်ချက်များကို သင့် `pom.xml` ဖိုင်တွင် ထည့်သွင်းလိုက်ပါ။ အောက်ပါ dependencies များကို MCP ပေါင်းစည်းမှုနှင့် GitHub Models အား ထောက်ပံ့ရန် ထည့်ပါ။

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

ပြီးနောက် သင့် Java client class ကို ဖန်တီးပါ။

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
    
    public static void main(String[] args) throws Exception {        // LLM ကို GitHub မော်ဒယ်များအသုံးပြုရန် စီစဉ်ပါ
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // ဆာဗာနှင့် ချိတ်ဆက်ရန် MCP သယ်ယူပို့ဆောင်မှု ဖန်တီးပါ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP client ဖန်တီးပါ
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

အထက်ပါကုဒ်တွင် ကျွန်ုပ်တို့သည် -

- **LangChain4j dependencies များ ထည့်သွင်းထားသည်** - MCP ပေါင်းစည်းမှု၊ OpenAI client အာဏာပေးခြင်းနှင့် GitHub Models ပံ့ပိုးမှုအတွက်
- **LangChain4j libraries များကို တင်သွင်းထားသည်** - MCP ပေါင်းစည်းမှုနှင့် OpenAI စကားပြောပုံစံမော်ဒယ်အတွက်
- **`ChatLanguageModel` တစ်ခု ဖန်တီးထားသည်** - ကိုယ်ပိုင် GitHub token ဖြင့် GitHub Models အသုံးပြုရန် ပြင်ဆင်ထားသည်
- **HTTP မှတ်ပုံတင်မှုကို တပ်ဆင်ထားသည်** - Server-Sent Events (SSE) ဖြင့် MCP server နှင့် ချိတ်ဆက်ခြင်းအတွက်
- **MCP client ကို ဖန်တီးထားသည်** - server နှင့် ဆက်သွယ်ရန် ကူညီပေးမည်
- **LangChain4j ၏ MCP ပေါင်းစည်းမှုစနစ်ကို အသုံးပြုပြီး** - LLM များနှင့် MCP servers အကြား ပေါင်းစည်းမှုကို ပိုမိုလွယ်ကူစေသည်

#### Rust

ဤ ဥပမာမှာ Rust အခြေပြု MCP server တစ်ခု လက်ရှိ အသုံးပြုနေသည်ဟု ထင်မှတ်ထားသည်။ ရှိသည်မဟုတ်ပါက [01-first-server](../01-first-server/README.md) သင်ခန်းစာသို့ ပြန်လည် مراجعه ပြုလုပ်၍ server တစ်ခု ဖန်တီးပါ။

Rust MCP server ရပြီးပါက terminal တစ်ခု ဖွင့်၍ server နေရာတည်းထဲသို့ သွားလိုက်ပါ။ ပြီးနောက် အောက်ပါ command ဖြင့် သင်၏ LLM client ပရောဂျက်အသစ်ကို ပြုလုပ်ပါ။

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

`Cargo.toml` ဖိုင်ကို အောက်ပါ dependencies များဖြင့် ပြင်ဆင်ပါ။

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI အတွက် Rust အတည်ပြု စာကြည့်တိုက် တစ်ခု မရှိသော်လည်း `async-openai` crate သည် [အဖွဲ့အစည်းထိန်းသိမ်းသော စာကြည့်တိုက်တစ်ခု](https://platform.openai.com/docs/libraries/rust#rust) ဖြစ်ပြီး လူသုံးများသည်။

`src/main.rs` ဖိုင်ကို ဖွင့်၍ အောက်ပါကုတ်ဖြင့် အစားထိုးလိုက်ပါ။

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
    // စတင်သတင်းစောင်
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI ဖောက်သည်ကိုထားရှိပါ
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP ဖောက်သည်ကိုထားရှိပါ
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

    // TODO: MCP ကိရိယာစာရင်းရယူပါ

    // TODO: ကိရိယာခေါ်ဆိုမှုများနှင့် LLM စကားပြောပြုလုပ်ခြင်း

    Ok(())
}
```

ဤကုဒ်သည် MCP server နှင့် GitHub Models ပါရှိသည့် LLM တွဲဖက်မှုအတွက် အခြေခံ Rust အပလီကေးရှင်း တစ်ခုကို ပြင်ဆင်ထားသည်။

> [!IMPORTANT]
> အပလီကေးရှင်းကို စတင်မပြေးမီ `OPENAI_API_KEY` Environment Variable အား သင့် GitHub token ဖြင့် သတ်မှတ်ရန် သေချာပါစေ။

အဆင်ပြေပါပြီ၊ နောက်ဆက်တွဲ အဆင့်မှာ server ရှိ ဖြစ်နိုင်မှုများကို စာရင်းပြုစုကြရအောင်။

### -2- Server ဖြစ်နိုင်မှုများ စာရင်းပြုစုခြင်း

ယခု server နှင့် ချိတ်ဆက်၍ ၎င်း၏ ဖြစ်နိုင်မှုများကို မေးမြန်းကြမည်။

#### Typescript

၎င်း class ထဲတွင် အောက်ပါ method များ ထည့်သွင်းပါ -

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // ကိရိယာများစာရင်းပြုစုခြင်း
    const toolsResult = await this.client.listTools();
}
```

အထက်ပါကုဒ်တွင် -

- server နှင့် ချိတ်ဆက်ရန် `connectToServer` method ကို ထည့်သွင်းထားသည်။
- `run` method ကို ဖန်တီးထားပြီး ယခုအချိန်တွင် tool များ စာရင်းပြုစုထားသော်လည်း နောက်ပိုင်းတွင် ထပ်မံတိုးမြှင့်မည်။

#### Python

```python
# ရရှိနိုင်သော အရင်းအမြစ်များကို စာရင်းပြုစုပါ
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# ရရှိနိုင်သော ကိရိယာများကို စာရင်းပြုစုပါ
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

ကျွန်ုပ်တို့ ထည့်သွင်းထားသည် -

- resource များနှင့် tool များကို စာရင်းပြုစုထားခြင်း နှင့် မျက်နှာတစ်ရပ်တွင် ထုတ်ပြထားခြင်း။ tool များအတွက် `inputSchema` ကိုလည်း ဤနေရာတွင် ဖော်ပြထားပြီး နောက်ပိုင်းတွင် အသုံးပြုမည်။

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

ကျွန်ုပ်တို့ -

- MCP Server တွင်ရရှိနိုင်သည့် tool များကို စာရင်းပြုစုထားသည်။
- tool တစ်ခုချင်းအား name, description နှင့် schema ကို စာရင်းပြုစုထားသည်။ schema ကို tools ခေါ်ရာတွင် အသုံးပြုမည်။

#### Java

```java
// MCP ကိရိယာများကို အလိုအလျောက် ရှာဖွေသည့် ကိရိယာပံ့ပိုးသူတစ်ဦး ဖန်တီးပါ
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP ကိရိယာပံ့ပိုးသူသည် အလိုအလျောက် အောက်ပါအကြောင်းများကို ကိုင်တွယ်ပါသည်
// - MCP ဆာဗာမှ ရနိုင်သည့် ကိရိယာများစာရင်းပြုစုခြင်း
// - MCP ကိရိယာ စီမံကိန်းများကို LangChain4j ပုံစံသို့ ပြောင်းလဲခြင်း
// - ကိရိယာ ဆောင်ရွက်မှုနှင့် တုံ့ပြန်ချက်များ စီမံခန့်ခွဲခြင်း
```

ကျွန်ုပ်တို့ -

- MCP server မှ တစ်ဆင့် tools များကို အလိုအလျောက် ရှာဖွေမှတ်သားသည့် `McpToolProvider` တစ်ခု ဖန်တီးထားသည်။
- tool provider သည် MCP tool schema များနှင့် LangChain4j tool ပုံစံအကြား ပြောင်းလဲခြင်းကို အတွင်းပိုင်းမှာ ကိုင်တွယ်ထားသည်။
- ဤနည်းလမ်းသည် ကိုယ်တိုင်လက်မဲ့ tools စာရင်းပြုစုခြင်းနှင့် ပြောင်းလဲခြင်း လုပ်ငန်းစဉ်ကို ရှောင်ရှားစေသည်။

#### Rust

MCP server ထံမှ tools များကို ရယူရာတွင် `list_tools` method ကို အသုံးပြုသည်။ `main` function တွင် MCP client ပြင်ဆင်ပြီးနောက် အောက်ပါ ကုဒ်ကို ထည့်သွင်းပါ။

```rust
// MCP ကိရိယာစာရင်းကို ရယူပါ။
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- Server ဖြစ်နိုင်မှုများကို LLM tools သို့ ပြောင်းလဲခြင်း

server ၏ ဖြစ်နိုင်မှုများ စာရင်းပြုစုပြီးနောက် LLM နားလည်နိုင်သော ပုံစံသို့ ပြောင်းလဲရန်လိုသည်။ ပြုပြင်ပြီးပါက ၎င်းပုံစံကို LLM အတွက် tools အဖြစ်ပေးနိုင်မည်။

#### TypeScript

1. MCP Server မှ တုံ့ပြန်ချက်ကို LLM အသုံးပြုနိုင်သည့် tool ပုံစံသို့ ပြောင်းရေးရန် အောက်ပါကုဒ် ထည့်ပါ -

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // input_schema အပေါ်အခြေခံ၍ zod schema တစ်ခု ဖန်တီးပါ
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // type ကို "function" ဟု တိတိကျကျ သတ်မှတ်ပါ
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

    ကုဒ်အထက်မှာ MCP Server response ကို လက်ခံ၍ LLM နားလည်နိုင်သည့် tool definition ပုံစံသို့ ပြောင်းလဲပေးသည်။

1. ထို့နောက် `run` method ကို update လုပ်၍ server ဖြစ်နိုင်မှုများကို စာရင်းပြုစုပါ -

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

    အထက်ပါကုဒ်တွင် `run` method ကို update လုပ်ပြီး အထွေထွေရလဒ်ကို ပါ၀င်သည့် entry တစ်ခုချင်းစီအတွက် `openAiToolAdapter` ကို ခေါ်သည်။

#### Python

1. အရင်ဆုံး အောက်ပါ converter function တစ်ခု ဖန်တီးပါ။

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

    `convert_to_llm_tools` function တွင် MCP tools response ကို ယုံကြည်ပြီး LLM နားလည်နိုင်သည့် ပုံစံသို့ ပြောင်းလဲပေးသည်။

1. နောက်တစ်ဆင့် client ကုဒ်ကို အောက်ပါအတိုင်း update ပြုလုပ်၍ function ကို အသုံးချပါ။

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    ဒီနေရာတွင် MCP tools response ကို LLM ထံ ပေးပို့နိုင်အောင် `convert_to_llm_tool` function ကို ခေါ်သည်။

#### .NET

1. MCP tools response ကို LLM နားလည်နိုင်အောင် ပြောင်းရန် အောက်ပါကုဒ် ထည့်ပါ။

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

အထက်ပါ ကုဒ်မှာ -

- name, description, input schema များ အသုံးပြု၍ `ConvertFrom` function ဖြစ်ပေါ်စေသည်။
- FunctionDefinition ကို ဖန်တီးပြီး ChatCompletionsDefinition သို့ ပေးပို့သည်။ ဤတို့သည် LLM နားလည်နိုင်သည့် ပုံစံဖြစ်သည်။

1. အထက်ဖော်ပြထားသော function မှ အကျိုးရလဒ်များကို အသုံးချရန်အတွက် ရှိပြီးသား ကုဒ်ကို ပြုပြင်ခြင်း -

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
// သဘာဝဘာသာစကားအပြန်အလှန်ဆက်သွယ်မှုအတွက် Bot အင်တာဖေ့စ်တစ်ခုကို ဖန်တီးပါ
public interface Bot {
    String chat(String prompt);
}

// LLM နှင့် MCP ကိရိယာများဖြင့် AI ဝန်ဆောင်မှုကို သတ်မှတ်ပါ
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

အထက်ပါကုဒ်တွင် -

- သဘာဝဘာသာစကားအပြောအဆိုအတွက် ရိုးရှင်းသော `Bot` interface သတ်မှတ်ထားသည်
- LangChain4j ၏ `AiServices` ကို အသုံးပြုပြီး LLM နှင့် MCP tool provider ကို အလိုအလျောက် ချိတ်ဆက်ထားသည်
- ၎င်း framework သည် tool schema ပြောင်းပြန်ခြင်းနှင့် function ခေါ်ဆိုမှုကို အတွင်းပိုင်းမှ ကိုင်တွယ်ပေးသည်
- ဤနည်းလမ်းသည် လက်မဲ့ tool ပြောင်းပြန်မှုဖြုတ်ပစ်ပြီး LangChain4j သည် MCP tools ကို LLM ပေါ်လွင်စေရန် စွမ်းဆောင်ရည်များ စီမံခန့်ခွဲပေးသည်

#### Rust

MCP tools response ကို LLM နားလည်နိုင်အောင် format ပြန်ပြောင်းရန် helper function တစ်ခု ထည့်ပေးမည်။ `main.rs` ဖိုင်အတွင်း `main` function အောက်တွင် အောက်ပါကုဒ် ထည့်ပါ။ ၎င်းသည် LLM ၌ တောင်းဆိုမှု ပြုလုပ်ရာတွင် အသုံးပြုမည်။

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

အရမ်းကောင်းပြီ၊ user တောင်းဆိုချက်များကို ကိုင်တွယ်ရန် ပြင်ဆင်ထားသည်မဟုတ်သေးသဖြင့် ထို့နောက် ဆောင်ရွက်ကြရအောင်။

### -4- User prompt ကို ကိုင်တွယ်ခြင်း

ဤကုဒ်တွင် user တောင်းဆိုချက်များကို ကိုင်တွယ်မည်။

#### TypeScript

1. LLM ကို ခေါ်ဆိုရန် အသုံးပြုမည့် method တစ်ခု ဖြည့်သွင်းပါ။

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. ဆာဗာရဲ့ ကိရိယာကို ခေါ်ပါ
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ရလဒ်နဲ့ တူညီတဲ့အရာတစ်ခုခုလုပ်ပါ
        // ပြင်ဆင်ရန်

        }
    }
    ```

    အထက်ပါကုဒ်တွင် -

    - `callTools` method ကို ထည့်သွင်းထားသည်။
    - method သည် LLM response ကို ယူကာ ခေါ်ထားသော tool များရှိမရှိ စစ်ဆေးသည်။

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // ကိရိယာကို ခေါ်ဆောင်ပါ
        }
        ```

    - LLM သည် ခေါ်ရမည့် tool ရှိကြောင်း ပြသသောအခါ tool ကို ခေါ်ဆိုသည် -

        ```typescript
        // 2. ဆာဗာ၏ကိရိယာကိုခေါ်ပါ
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ရလဒ်နှင့်အတူတစ်ခုခုလုပ်ပါ
        // လုပ်ရန်တည်းဖြတ်ရန် (TODO)
        ```

1. `run` method ကို update လုပ်ပြီး LLM နှင့် `callTools` ကို ခေါ်သောအခါ ကိုထည့်ပါ။

    ```typescript

    // ၁။ LLM အတွက် အဝင်ဖြစ်သော စာတိုများ ဖန်တီးပါ
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // ၂။ LLM ကို ခေါ်ဆိုခြင်း
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // ၃။ LLM ပြန်ကြားချက်အား တစ်ခုချင်းစီ စစ်ဆေးပြီး tools ခေါ်ဆိုမှုရှိမရှိ စစ်ဆေးပါ
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

အရမ်းကောင်းပြီ၊ လက်လုံးကုဒ်အပြည့်အစုံကို ကြည့်ပါ။

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // စီမံချက်အတည်ပြုချက်အတွက် zod ကိုထည့်သွင်းပါ

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // အနာဂတ်တွင် ဒီ URL သို့ပြောင်းရန် လိုအပ်နိုင်သည်: https://models.github.ai/inference
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
          // input_schema အပေါ်မှာအခြေခံပြီး zod schema တစ်ခုရေးပါ
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // အမျိုးအစားကို "function" ဟုသတ်မှတ်ပေးပါ
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
    
    
          // 2။ ဆာဗာရဲ့ကိရိယာကိုခေါ်ပါ
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3။ ရလာဒ်နဲ့တစ်စိတ်တစ်ပိုင်းလုပ်ဆောင်ပါ
          // ပြုလုပ်ရန်
    
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
    
        // 1။ LLM ဖြေကြားချက်ကို ဖြတ်သန်းပြီး၊ ရွေးချယ်မှုတိုင်းအတွက် ကိရိယာခေါ်ဆိုမှုများရှိမရှိ စစ်ဆေးပါ
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

1. LLM ခေါ်ဆိုရန် လိုအပ်သော import များကို ထည့်ပါ။

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. LLM ကို ခေါ်မည့် function ကို ထည့်ပါ။

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
            # ရွေးချယ်စရာ ပါရာမီတာများ
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

    အထက်ပါကုဒ်တွင် -

    - MCP server တွင် တွေ့ရှိခဲ့သော functions များကို convert ပြုလုပ်ပြီး LLM ထံ ပေးသွင်းထားသည်။
    - function များဖြင့် LLM ကို ခေါ်ဆိုထားသည်။
    - ရလဒ်ကို စစ်ဆေးပြီး ခေါ်ရမည့် function ရှိမရှိ စစ်ဆေးသည်။
    - အဆုံးတွင် ခေါ်ရမည့် function များအား မှတ်စုထားသည်။

1. နောက်ဆုံးအဆင့် client ကုဒ်ကို update လုပ်ပါ။

    ```python
    prompt = "Add 2 to 20"

    # LLM ကို မည်သည့်ကိရိယာများရှိမရှိ မေးထားပါ
    functions_to_call = call_llm(prompt, functions)

    # အကြံပြုထားသောလုပ်ဆောင်ချက်များကို ခေါ်ပါ
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    အထက်ပါကုဒ်တွင် -

    - LLM မှ ရှာဖွေကြည့်ရွေးချယ်ထားသော function ဖြင့် MCP tool ကို `call_tool` ဆက်သွယ်ခေါ်ဆိုသည်။
    - MCP server မှ tool ခေါ်ဆိုမှုရလဒ်ကို ထုတ်ပြပါသည်။

#### .NET

1. LLM prompt request ဆွဲထုတ်ရန် ကုဒ်တစ်ခု ပြသပါ။

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

အထက်ပါကုဒ်တွင် -

- MCP server မှ tools များ ထုတ်ယူကြောင်း (`var tools = await GetMcpTools()`)
- user prompt ကို သတ်မှတ်ထားသည် (`userMessage`)
- model နှင့် tools ဖြည့်စွက်ထားသော options object ကို ဖန်တီးသည်
- LLM ထံ နှုတ်ခွန်းဆက် တောင်းဆိုထားသည်

1. နောက်ဆုံး အဆင့်တွင် LLM ၏ function ခေါ်ဆိုမှု အလားအလာ ရှိမရှိ စစ်ဆေးပါ။

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

အထက်ပါကုဒ်တွင် -

- function call များကို loop ဖြင့် စစ်ဆေးသည်။
- tool call များအတွက် name နှင့် argument များ ထုတ်ယူကာ MCP server တွင် tool ခေါ်ဆိုသည်။ နောက်ဆုံးတွင် ရလဒ်အား ပုံနှိပ်ပြသည်။

လုံးဝကုဒ် -

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
    // MCP ကိရိယာများကို ကိုယ့်အလိုအလျောက် အသုံးပြု၍ သဘာ၀ဘာသာစကား တောင်းဆိုချက်များအား အလုပ်အမှုဆောင်ပါ။
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

အထက်ပါကုဒ်တွင် -

- MCP server tools မျှသာ ရိုးရှင်းသော သဘာဝဘာသာစကား prompt များဖြင့် ဆက်သွယ်သည်
- LangChain4j framework သို့ လိုက်လျောညီထွေဖြစ်စွာ
  - user prompt များကို tool call များသို့ ပြောင်းလဲခြင်း
  - LLM ဆုံးဖြတ်ချက်အရ MCP tools ကိုခေါ်ဆိုခြင်း
  - LLM နှင့် MCP server အကြား ဆက်သွယ်မှုစဉ်ဆက်ကို စီမံခန့်ခွဲခြင်း
- `bot.chat()` method သည် MCP tools အလုပ်များရလဒ်ပါဝင်နိုင်သော သဘာဝဘာသာဖြေကြားချက်များကို ပြန်လည်ထုတ်ပေးသည်
- ဤနည်းလမ်းသည် အသုံးပြုသူများ MCP ၏ နောက်ကွယ်အကောင်အထည်ကို မသိဘဲ အသုံးပြုနိုင်သော ချောဆီမှုရှိစေသည်

ကုဒ်အပြည့်အစုံ -

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

ဤနေရာတွင် အလုပ်အများစု ကိုင်တွယ်သည်။ အစပိုင်း user prompt ဖြင့် LLM ကို ခေါ်ဆိုလာပြီး တုံ့ပြန်ချက်အား စစ်ဆေးကာ tool call မလိုအပ်သည်ထိ ဆက်လက်လုပ်ဆောင်မည်။ tool call လိုအပ်ပါက သတ်မှတ်ကိရိယာများကို ခေါ်ဆိုကာ LLM နှင့် ဆက်သွယ်မှုကို ဆက်လက်တိုးချဲ့မည်။

LLM အတွက် အခါအားလျော်စွာ ခေါ်ဆိုမှုများပြုလုပ်မည်ဖြစ်သောကြောင့် LLM call ကို ကိုင်တွယ်မည့် function ကို သတ်မှတ်ပါ။ ၎င်း function ကို `main.rs` ဖိုင်အတွင်း ထည့်ပါ။

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

ဤ function သည် LLM client, messages စာရင်း (user prompt ပိုင်းပါဝင်သည့်), MCP server မှ tools များကို လက်ခံကာ LLM အထံ တောင်းဆိုမှု ပေးပို့ပြီး တုံ့ပြန်ချက် ထုတ်ပေးသည်။
LLM ကနေ ရလာတဲ့ response မှာ `choices` ဆိုတဲ့ array တစ်ခု ပါမယ်။ တစ်ခုခု `tool_calls` ရှိမရှိ စစ်ဆေးဖို့အတွက် ရလာတဲ့ စာရင်းကို ပြန်လုပ်ဆောင်ဖို့ လိုပါတယ်။ ဒါကတော့ LLM က တိတိကျကျသော tool တစ်ခုကို argument တွေနဲ့ခေါ်တဲ့ function တစ်ခု လို့ သိနိုင်နိုင်ပါတယ်။ သင့် `main.rs` ဖိုင်အောက်ဆုံးမှာ ထည့်သွင်းဖို့ ဒီကုဒ်ကို ထည့်လိုက်ပါ။

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

    // အကြောင်းအရာရှိလျှင် ပရင့်ထုတ်ပါ
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // သုံးစွဲသူကိရိယာခေါ်ဆိုမှုများကို မီးမှိတ်ပါ
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // အကူအညီပေးသူ ဆန်းစစ်ချက်ထည့်ပါ

        // သုံးစွဲသူကိရိယာခေါ်ဆိုမှု တစ်ခုစီကို လုပ်ဆောင်ပါ
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // ကိရိယာရလဒ်များကို သတင်းစကားများထဲ ထည့်ပါ
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ကိရိယာရလဒ်များဖြင့် ဆက်သွယ်မှုကို ဆက်လက်ပါ
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


`tool_calls` တွေရှိရင်တော့၊ tool အချက်အလက်တွေကိုခွဲထုတ်ပြီး MCP server ကို tool request နဲ့ခေါ်ပါတယ်၊ ဒါနဲ့ ရလာတဲ့ ဆက်သွယ်ချက်တွေကို conversation messages ထဲသို့ ထည့်ပေးပါတယ်။ ထိုနောက် conversation ကို LLM တို့နဲ့ ဆက်လက်ဆောင်ရွက်သောကြောင့် assistant ရဲ့ တုံ့ပြန်ချက်နဲ့ tool call ရလဒ်တွေကို messages မှာ update လုပ်ပေးပါတယ်။

LLM က MCP calls များအတွက် ပြန်ပေးတဲ့ tool call အချက်အလက်တွေကို ခွဲထုတ်ဖို့ အတူတူ ဘယ်အရာတွေလိုအပ်တယ်ဆိုတာခွဲထုတ်ပေးမယ့် helper function တစ်ခုကို ထည့်လို့ရပါတယ်။ သင့် `main.rs` ဖိုင် အောက်ဆုံးမှာ အောက်ပါကုဒ်ကိုထည့်ပါ။

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


လုံးဝအပိုင်းတွေ ပြဿနာ အားလုံးအဆင်ပြေပြီးသွားပါက သုံးစွဲသူရဲ့ ဒါရိုက်ဖြင့် ရရှိတဲ့ user prompt ကို အစပျိုးပြီး LLM ကို ခေါ်ဆောင်ဖို့နဲ့ အောက်ပါကုဒ်များကို သင့် `main` function မှာ update လုပ်ပါ။

```rust
// ဝိသေသလက္ခဏာများနှင့် talk-to-tool စကားပြောဆိုခြင်း
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


ဒီအတိုင်းလုပ်ခြင်းဖြင့် LLM ကို သုံးစွဲသူရဲ့ ဗျူဟာ prompt ဖြင့် တိုက်ရိုက်မေးမြန်းပြီး နှစ်ခုသော ဂဏန်းများ၏ စုစုပေါင်းကို ရယူရန် တုံ့ပြန်မှုကို ချဉ်းကပ်နိုင်ပြီး၊ tool calls များကို dynamic အဖြစ် ပြုလုပ်ဆောင်ရွက်နိုင်ပါသည်။

အရမ်းကောင်းပြီ၊ သင် ပြီးသွားပါပြီ!

## တာဝန်ပေးစာ

အောက်ပါ အစီအရင်ခံစာမှ ကုဒ်ကို သင်ယူပြီး ဆာဗာကို တခြား tool များဖြင့် တိုးချဲ့ တည်ဆောက်ပါ။ ပြီးလျှင် LLM ပါ သုံးသော client တစ်ခုကို ဖန်တီးပြီး အမျိုးမျိုးသော prompt များဖြင့် စမ်းသပ်ပါ၊ ဒါကြောင့် သင်၏ ဆာဗာ tool များအား dynamic အားဖြင့် ခေါ်ယူခြင်း အပေါ် သေချာစေနိုင်မှာ ဖြစ်သည်။ Client တည်ဆောက်ပုံတွင် အသုံးပြုသူများသည် တိတိကျကျ client command မဟုတ်ပဲ prompt များဖြင့် အသုံးပြုနိုင်တဲ့ အတွက်၊ MCP server ကို တဖြည်းဖြည်း ခေါ်ယူနေရခြင်းကို မသိမဖြစ် အသုံးပြုနိုင်မည်ဖြစ်ပြီး ယခုနည်းဖြင့် အသုံးပြုသူအတွေ့အကြုံမှာ လုပ်ဆောင်မှု ကောင်းမွန်စေပါသည်။

## ဖြေရှင်းနည်း

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## အဓိက သင်ခန်းစာများ

- LLM ကို client အတွက် ထည့်သွင်းခြင်းသည် MCP Servers နှင့် အသုံးပြုသူများ၏ အပြန်အလှန် ဆက်သွယ်မှု အတွက် ပိုမိုကောင်းမွန်သော နည်းလမ်း ဖြစ်စေသည်။
- MCP Server ၏ တုံ့ပြန်မှုကို LLM မှာနားလည်နိုင်အောင် ပြောင်းလဲပေးရန် လိုအပ်သည်။

## နမူနာများ

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## အပိုဆောင်းရင်းမြစ်များ

## နောက်တစ်ဆင့်

- နောက်တစ်ဆင့်: [Visual Studio Code ကို အသုံးပြုပြီး server ကို စေ့စပ်အသုံးပြုခြင်း](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ကြေညာချက်**  
ဤစာတမ်းကို AI ဘာသာပြန်စနစ်ဖြစ်သော [Co-op Translator](https://github.com/Azure/co-op-translator) မှတဆင့်ဘာသာပြန်ထားသည်။ ကျွန်ုပ်တို့သည်တိကျမှန်ကန်မှုအတွက်ကြိုးပမ်းထားသော်လည်း၊ အလိုအလျောက်ဘာသာပြန်မှုတွင်အမှားများ သို့မဟုတ်မှားယွင်းချက်များပါရှိနိုင်ကြောင်း သတိပြုနားလည်ရေးရမည်။ မူရင်းစာတမ်းကို မူရင်းဘာသာဖြင့်သာအတည်ပြုရမည့်အချက်အလက်အနေဖြင့်ယူဆသင့်သည်။ အရေးကြီးသောအချက်အလက်များအတွက် လူကြီးမင်းသည် ပရော်ဖက်ရှင်နယ်လူသားဘာသာပြန်ကြီးကို အသုံးပြုရန်အကြံပြုပါသည်။ ဤဘာသာပြန်မှု အသုံးပြုခြင်းဖြင့် ဖြစ်ပေါ်သော မှားယွင်း နားမလည်မှုများအတွက် ကျွန်ုပ်တို့သည် တာဝန်ယူမထားပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
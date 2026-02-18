# LLM ഉപയോഗിച്ച് ഒരു ക്ലയന്റ് സൃഷ്ടിക്കല്‍

ഇതുവരെ, നിങ്ങൾ സെർവർ ഉം ക്ലയന്റ് ഉം എങ്ങനെ സൃഷ്ടിക്കാമെന്ന് കാണിട്ടുണ്ട്. ക്ലയന്റ് സെർവറെ സ്പഷ്ടമായി വിളിച്ച് അതിന്റെ ഉപകരണങ്ങൾ, വിഭവങ്ങൾ, പ്രോംപ്റ്റുകൾ എന്നിവയുടെ പട്ടിക ലഭിക്കുന്നതിന് കഴിയുകയും ചെയ്തു. എന്നിരുന്നുവെങ്കിലും, ഇത് വളരെ പ്രായോഗിക മാർഗമല്ല. നിങ്ങളുടെ ഉപയോക്താവ് ഏജന്റിക് കാലഘട്ടത്തിൽ ജീവിക്കുന്നു, അവർ പ്രോംപ്റ്റുകൾ ഉപയോഗിച്ച് LLM-നോടു സംവദിക്കാൻ പ്രതീക്ഷിക്കുന്നു. നിങ്ങളുടെ ഉപയോക്താവിന് MCP ഉപയോഗിക്കുന്നുവോ ഇല്ലയോ എന്ന് ശ്രദ്ധിക്കേണ്ടതില്ല, പക്ഷേ അവർ പ്രകൃത ഭാഷ ഉപയോഗിച്ച് സംവദിക്കാൻ ആഗ്രഹിക്കുന്നു. ആ então നിവാരണം എങ്ങനെ? പരിഹാരം ക്ലയന്റിൽ ഒരു LLM ചേർക്കുന്നതാണ്.

## അവലോകനം

ഈ પાઠത്തിൽ, ഞങ്ങൾ നിങ്ങളുടെ ക്ലയന്റിൽ ഒരു LLM ചേർക്കുന്നതിൽ കേന്ദ്രീകരിക്കുകയും ഇത് നിങ്ങളുടെ ഉപയോക്താവിന് എങ്ങനെ കൂടുതൽ മെച്ചപ്പെട്ട അനുഭവം നൽകുന്നുവെന്ന് കാണിക്കുകയും ചെയ്യും.

## പഠന ലക്ഷ്യങ്ങൾ

ഈ പാഠം പൂർത്തിയായി നിങ്ങൾക്ക് തരും:

- LLM ഉള്ള ക്ലയന്റ് സൃഷ്ടിക്കുക.
- LLM ഉപയോഗിച്ച് MCP സെർവർക്ക് സമഗ്രമായി സംവദിക്കുക.
- ക്ലയന്റ് വശത്ത് മികച്ച അവസാന ഉപയോക്തൃ അനുഭവം നൽകുക.

## സമീപനം

നമുക്ക് എടുക്കേണ്ട സമീപനം മനസ്സിലാക്കാം. LLM ചേർക്കുന്നത് ലളിതമാണ് തോന്നിയേക്കാം, പക്ഷേ നമുക്ക് അത് പ്രായോഗികമായി എങ്ങനെ നടത്താം?

ഇല്ലാ ക്ലയന്റ് സെർവറുമായി എങ്ങനെ സംവദിക്കും:

1. സെർവറുമായി കണക്ഷൻ സ്ഥാപിക്കുക.

1. കഴിവുകൾ, പ്രോംപ്റ്റുകൾ, വിഭവങ്ങൾ, ഉപകരണങ്ങൾ പട്ടികപ്പെടുത്തുക, അവയുടെ സ്കീമ സേവ് ചെയ്യുക.

1. LLM ചേർക്കുക, സേവ് ചെയ്ത കഴിവുകളും അവയുടെ സ്കീമയും LLM മനസ്സിലാക്കുന്ന ഫോർമാറ്റിൽ കടത്തുക.

1. ഉപയോക്തൃ പ്രോംപ്റ്റ് LLM-ന് കൈമാറുക, ക്ലയന്റ് പട്ടികപ്പെടുത്തിയ ഉപകരണങ്ങളോടൊപ്പം.

മികച്ചത്, നമുക്ക് ഇതിൽ ഹൈലെവൽ എങ്ങനെ ചെയ്യാമെന്ന് മനസ്സിലായി, താഴെയുള്ള സഹായഭാഗത്തിൽ പരീക്ഷിക്കാം.

## അഭ്യാസം: LLM ഉള്ള ക്ലയന്റ് സൃഷ്ടിക്കൽ

ഈ അഭ്യാസത്തിൽ, ഞങ്ങൾ എങ്ങനെ LLM ക്ലയന്റിൽ ചേർക്കാമെന്ന് പഠിക്കും.

### GitHub Personal Access Token ഉപയോഗിച്ച് പ്രാമാണീകരണം

GitHub ടോക്കൺ സൃഷ്ടിക്കൽ സരളമായ പ്രവൃത്തി ആണ്. ഇതാ എങ്ങനെ ചെയ്യാം:

- GitHub Settings-ലേക്ക് പോകുക - മുകളിൽ വലതുവശത്ത് നിങ്ങളുടെ പ്രൊഫൈൽ ചിത്രം ക്ലിക് ചെയ്ത് Settings തിരഞ്ഞെടുക്കുക.
- Developer Settings-ലേക്ക് പോകുക - താഴേക്ക് സ്ക്രോൾ ചെയ്ത് Developer Settings ക്ലിക് ചെയ്യുക.
- Personal Access Tokens തിരഞ്ഞെടുക്കുക - Fine-grained tokens ക്ലിക് ചെയ്ത് Generate new token തിരഞ്ഞെടുക്കുക.
- നിങ്ങളുടെ ടോക്കൺ കോൺഫിഗർ ചെയ്യുക - റഫറൻസിനായി ഒരു നോട്ടായി ചേർക്കുക, കാലാവധി സജ്ജമാക്കുക, ആവശ്യമായ സ്കോപ്പുകൾ (അനുഗ്രഹങ്ങൾ) തെരഞ്ഞെടുക്കുക. ഈ കേസിൽ Models അനുമതി നൽകുക.
- Generate and Copy Token - Generate token ക്ലിക് ചെയ്ത് ടോക്കൺ എപ്പോഴും കാണാൻ കഴിയാത്തതിനാൽ ഉടൻ കോപ്പി ചെയ്യുക.

### -1- സെർവറുമായി കണക്ട് ചെയ്യുക

ആദ്യമായി നമ്മുടെ ക്ലയന്റ് സൃഷ്ടിക്കാം:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // സ്കീമയുൽ പരിശോധനയ്ക്കായി zod ഇറക്കുമതി ചെയ്യുക

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

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- ആവശ്യമായ ലൈബ്രറികൾ ഇറക്കുമതി ചെയ്തു
- `client` ഉം `openai` ഉം എന്ന രണ്ട് അംഗങ്ങളുള്ള ക്ലാസ് സൃഷ്ടിച്ചു, ക്ലയന്റ് മാനേജും LLM-നോടുള്ള സംവദനവും സഹായിക്കും.
- LLM ഇൻസ്റ്റൻറ് GitHub മോഡലുകൾ ഉപയോഗിക്കാൻ `baseUrl` ഇൻഫറൻസ് API-യിലേക്ക് ചേർത്തു.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio കണക്ഷനിലേക്ക് സർവർ പാരാമീറ്ററുകൾ സൃഷ്ടിക്കുക
server_params = StdioServerParameters(
    command="mcp",  # എക്സിക്യുട്ടബിൾ
    args=["run", "server.py"],  # ഓപ്ഷണൽ കമാൻഡ് ലൈന arguments
    env=None,  # ഓപ്ഷണൽ പരിസര ച ഓച്ചുകൾ
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # കണക്ഷൻ ആരംഭിക്കൽ
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- MCP-നായി ആവശ്യമായ ലൈബ്രറികൾ ഇറക്കുമതി ചെയ്തു
- ഒരു ക്ലയന്റ് സൃഷ്ടിച്ചു

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

ആദ്യം, നിങ്ങളുടെ `pom.xml` ഫയലിൽ LangChain4j ഡിപ്പെൻഡൻസികൾ ചേർക്കണം. MCP ഇന്റഗ്രേഷൻ GitHub മോഡലുകൾ പിന്തുണയ്ക്കുന്നതിനുള്ള ഡിപ്പെൻഡൻസികൾ ചേർക്കുക:

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

അപ്പോൾ നിങ്ങളുടെ Java ക്ലയന്റ് ക്ലാസ് സൃഷ്ടിക്കുക:

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
    
    public static void main(String[] args) throws Exception {        // LLM-നെ GitHub മോഡലുകൾ ഉപയോഗിച്ചാണ് ക്രമീകരിക്കുക
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // സെർവറുമായി ബന്ധിപ്പിക്കാൻ MCP ട്രാൻസ്പോർട്ട് ഉണ്ടാക്കുക
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP ക്ലയന്റ് ഉണ്ടാക്കുക
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- **LangChain4j ഡിപ്പെൻഡൻസികൾ ചേർത്തു**: MCP ഇന്റഗ്രേഷൻ, OpenAI ഒഫീഷ്യൽ ക്ലയന്റ്, GitHub മോഡൾ പിന്തുണയ്ക്കാനായി
- **LangChain4j ലൈബ്രറികൾ ഇറക്കുമതി ചെയ്തു**: MCP ഇന്റഗ്രേഷനും OpenAI ചാറ്റ് മോഡ്യൂൾ പ്രവർത്തനത്തിനും
- **`ChatLanguageModel` സൃഷ്ടിച്ചു**: GitHub മోడലുകൾ ഉപയോഗിച്ച് GitHub ടോക്കൺ സജ്ജീകരിച്ചു
- **HTTP ട്രാൻസ്പോർട്ട് സെറ്റപ്പ് ചെയ്തു**: സെർവർ സെന്റ് ഇവന്റുകൾ (SSE) ഉപയോഗിച്ച് MCP സെർവറുമായി കണക്ട് ചെയ്യാൻ
- **MCP ക്ലയന്റ് സൃഷ്ടിച്ചു**: സെർവറുമായി ആശയവിനിമയം കൈകാര്യം ചെയ്യാനുള്ളത്
- **LangChain4j-യുടെ MCP പിന്തുണ ഉപയോഗിച്ചു**: LLM-കൾക്കും MCP സെർവർ മദ്ധ്യേ ഇന്റഗ്രേഷൻ ലളിതമാക്കുന്നു

#### Rust

ഈ ഉദാഹരണംğiniz Rust അടിസ്ഥാനമാക്കിയ MCP സെർവർ നടത്തുന്നുവെന്ന് കരുതുന്നു. നിങ്ങൾക്ക് ഒരു സെർവർ ഇല്ലെങ്കിൽ, [01-first-server](../01-first-server/README.md) പാഠത്തിലേക്ക് തിരിച്ചുപോകി സെർവർ സൃഷ്ടിക്കുക.

നിങ്ങളുടെ Rust MCP സെർവർ ഉണ്ടാകുമ്പോൾ, ടെർമിനൽ തുറന്ന് സെർവറിന്റെ നിർമ്മാണ ഡയറക്ടറിയിലേക്ക് പോവുക. തുടർന്ന് ഒരു പുതിയ LLM ക്ലയന്റ് പ്രോജക്റ്റ് സൃഷ്ടിക്കാൻ താഴെയുള്ള കമാൻഡ് ഉപയോഗിക്കുക:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

നിങ്ങളുടെ `Cargo.toml` ഫയലിൽ താഴെ കാണുന്ന ഡിപ്പെൻഡൻസികൾ ചേർക്കുക:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI-ക്കായി ഔദ്യോഗിക Rust ലൈബ്രറി ഇല്ലെങ്കിലും, `async-openai` ക്രേറ്റ് ഒരു [സമൂഹം സംരക്ഷിക്കുന്ന ലൈബ്രറിയാണ്](https://platform.openai.com/docs/libraries/rust#rust) സാധാരണയായി ഉപയോഗിക്കുന്നത്.

`src/main.rs` ഫയലിൽ താഴെയുള്ള കോഡുമായി പകരം മാറുക:

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
    // ആദ്യം സന്ദേശം
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI ക്ലയന്റ് സജ്ജമാക്കുക
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP ക്ലയന്റ് സജ്ജമാക്കുക
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

    // ചെയ്യേണ്ടത്: MCP ഉപകരണ പട്ടിക നേടുക

    // ചെയ്യേണ്ടത്: ടൂൾ കോളുകളോടും LLMസംവാദം

    Ok(())
}
```

ഈ കോഡ് ഒരു അടിസ്ഥാന Rust അപ്ലിക്കേഷൻ സജ്ജമാക്കുന്നു, ഇത് MCP സെർവറുമായും GitHub മോഡലുകളും LLM ആശയവിനിമയത്തിനും ബന്ധപ്പെട്ടിരിക്കുന്നു.

> [!IMPORTANT]
> ആപ്ലിക്കേഷൻ പ്രവർത്തിപ്പിക്കാൻ മുമ്പ് `OPENAI_API_KEY` പരിസ്ഥിതി ചാരിക GitHub ടോക്കണിൽ സജ്ജമാക്കുക.

മികച്ചത്, അടുത്ത ഘട്ടത്തിന് MCP സെർവറിലെ കഴിവുകൾ പട്ടികപ്പെടുത്താം.

### -2- സെർവർ കഴിവുകൾ പട്ടികപ്പെടുത്തുക

ഇപ്പോൾ സെർവറുമായി കണക്ട് ചെയ്ത് അതിന്റെ കഴിവുകൾ ചോദിക്കുന്നു:

#### Typescript

അതേ ക്ലാസ്സിൽ, താഴെ കാണുന്ന രീതിയിൽ മെത്തഡുകൾ ചേർക്കുക:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // ഉപകരണങ്ങൾ പട്ടികപ്പെടുത്തുന്നു
    const toolsResult = await this.client.listTools();
}
```

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- സെർവറുമായി കണക്ട് ചെയ്യുന്നതിനുള്ള കോഡ് `connectToServer` ചേർത്ത്.
- അപ്ലിക്കേഷൻ ഫ്ളോ കൈകാര്യം ചെയ്യുന്ന `run` മെത്തഡ് സൃഷ്ടിച്ചത്; ഇതുവരെ ഇത് ഉപകരണങ്ങൾ മാത്രം പട്ടികപ്പെടുത്തിയിട്ടുണ്ട്, തുടർന്ന് കൂടുതൽ ചേർക്കും.

#### Python

```python
# ലഭ്യമായ വിഭവങ്ങള്‍ പട്ടികവച്ച് കാണിക്കുക
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# ലഭ്യമായ ഉപകരണങ്ങള്‍ പട്ടികവച്ച് കാണിക്കുക
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

നിങ്ങൾ ചേർത്തത്:

- വിഭവങ്ങളും ഉപകരണങ്ങളും പട്ടികപ്പെടുത്തി, പ്രിന്റ് ചെയ്തു. ഉപകരണങ്ങൾക്ക് `inputSchema` പട്ടികപ്പെടുത്തി, പിന്നീട് ഉപയോഗിക്കും.

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

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- MCP സെർവറിലെ ലഭ്യമായ ഉപകരണങ്ങൾ പട്ടികപ്പെടുത്തി
- ഓരോ ഉപകരണത്തിനും പേര്, വിവരണം, സ്കീമ പട്ടികപ്പെടുത്തി. ഇത് ഉപകരണം വിളിക്കുമ്പോൾ ഉപയോഗിക്കും.

#### Java

```java
// MCP ടൂളുകൾ സ്വയം കണ്ടെത്തുന്ന ഒരു ടൂൾ പ്രൊവൈഡർ സൃഷ്ടിക്കുക
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP ടൂൾ പ്രൊവൈഡർ സ്വയം കൈകാര്യം ചെയ്യുന്നു:
// - MCP സെർവറിൽ നിന്ന് ലഭ്യമായ ടൂളുകളുടെ ലിസ്റ്റിംഗ്
// - MCP ടൂൾ സ്കീമകൾ LangChain4j ഫോർമാറ്റിലേക്കു പരിവർത്തനം ചെയ്യുക
// - ടൂൾ നിർവഹണവും പ്രതികരണങ്ങളും പരിപാലിക്കൽ
```

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- MCP സെർവറിൽ നിന്നുള്ള എല്ലാ ഉപകരണങ്ങളും സ്വയം കണ്ടെത്തി രജിസ്റ്റർ ചെയ്യുന്ന `McpToolProvider` സൃഷ്ടിച്ചു
- ടൂൾ പ്രൊവൈഡർ MCP ടൂൾ സ്കീമകൾ LangChain4j ഉപകരണ ഫോർമാറ്റിലേക്ക് മാറ്റം കൈകാര്യം ചെയ്തു
- ഈ സമീപനം കയ്യേറ്റ് പട്ടികപ്പെടുത്തലും മാറ്റങ്ങളും ഒഴിവാക്കി

#### Rust

MCP സെർവറിൽ നിന്ന് ഉപകരണങ്ങൾ കണ്ടെത്താൻ `list_tools` മെത്തഡ് ഉപയോഗിക്കുക. നിങ്ങളുടെ `main` ഫംഗ്ഷനിൽ MCP ക്ലയന്റ് സജ്ജീകരിച്ചതിനു ശേഷം താഴെയുള്ള കോഡ് ചേർക്കുക:

```rust
// MCP ടൂൾ ലിസ്റ്റിംഗ് നേടുക
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- സെർവർ കഴിവുകൾ LLM ഉപകരണങ്ങളായി പരിവർത്തനം ചെയ്യുക

സെർവർ കഴിവുകൾ പട്ടികപ്പെടുത്തിയശേഷം, അവ LLM മനസ്സിലാക്കുന്ന ഫോർമാറ്റിൽ പരിവർത്തനം ചെയ്യണം. അത് കഴിഞ്ഞാൽ, ഈ കഴിവുകൾ LLM ഉപകരണങ്ങളായി നൽകാം.

#### TypeScript

1. MCP സെർവറിന്റെ പ്രതിസന്ധി LLM ഉപയോഗിയ്ക്കാവുന്ന ടൂൾ ഫോർമാറ്റിലേക്ക് മാറ്റുവാൻ താഴെയുള്ള കോഡ് ചേർക്കുക:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ഇൻപুট്സ്‌കീമയുടെ അടിസ്ഥാനത്തിൽ ഒരു സോഡ് സ്കീമ സൃഷ്‌ടിക്കുക
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // ടൈപ്പ് "function" ആയി വ്യക്തമായി സജ്ജമാക്കുക
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

    മുകളിൽ കാണിച്ച കോഡ് MCP സെർവറിന്റെ പ്രതികരണം സ്വീകരിച്ച് LLM മനസ്സിലാക്കുന്ന ഉപകരണം നിർവചിക്കുന്ന ഫോർമാറ്റിലേക്ക് പരിവർത്തനം ചെയ്യുന്നു.

1. ശേഷം `run` മെത്തഡ് പുതുക്കാം, സെർവർ കഴിവുകൾ പട്ടികപ്പെടുത്താൻ:

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

    മുൻപത്തെ കോഡിൽ, `run` മെത്തഡ് ഫലം മാപ്പ് ചെയ്ത് ഓരോ എൻട്രിക്കും `openAiToolAdapter` വിളിക്കുന്നു.

#### Python

1. ആദ്യം ഒരു പരിണാമകരി ഫംഗ്ഷൻ ക്രിയേറ്റ് ചെയ്യാം:

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

    `convert_to_llm_tools` ഫങ്ക്ഷൻ MCP ടൂൾ പ്രതികരണം LLMക്ക് മനസ്സിലാകുന്ന ഫോർമാറ്റിലേക്ക് മാറ്റുന്നു.

1. തുടര്‍ന്ന് ക്ലയന്റ് കോഡ് ഈ ഫംഗ്ഷൻ ഉപയോഗിക്കാനായി അപ്‌ഡേറ്റ് ചെയ്യാം:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    ഇവിടെ MCP ടൂൾ പ്രതികരണം LLM-ന് നേരത്തെ നൽകി ഉപയോഗിക്കാനായി `convert_to_llm_tool` വിളിക്കുന്നു.

#### .NET

1. MCP ടൂൾ പ്രതികരണം LLM മനസ്സിലാക്കാവുന്ന ഫോർമാറ്റിലേക്ക് പരിവർത്തനം ചെയ്യുന്നതിനുള്ള കോഡ് ചേർക്കാം:

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

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- `ConvertFrom` എന്ന ഫംഗ്ഷൻ സൃഷ്ടിച്ചു, അതിൽ പേര്, വിവരണം, ഇൻപുട്ട് സ്കീമ സ്വീകരിക്കുന്നു.
- ഫംഗ്ഷൻ നിർവചിച്ചു, അത് `FunctionDefinition` സൃഷ്ടിച്ച് അത് `ChatCompletionsDefinition`-ക് പാസ്സ് ചെയ്യുന്നു, ഇത് LLMക്ക് മനസ്സിലാകുന്നു.

1. മുകളിലുള്ള ഫംഗ്ഷൻ ഉപയോഗിച്ച് നിലവിലുള്ള ചില കോഡ് തുടക്കത്തിൽ അപ്ഡേറ്റ് ചെയ്യാം:

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
// സ്വാഭാവിക ഭാഷ സംവാദത്തിനായി ബോട്ട് ഇന്റർഫേസ് സൃഷ്‌ടിക്കുക
public interface Bot {
    String chat(String prompt);
}

// LLM ഉം MCP ഉപകരണങ്ങളും ഉപയോഗിച്ച് AI സേവനം കോൺഫിഗർ ചെയ്യുക
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

മുൻകൂട്ടിയുള്ള കോഡിൽ ഞങ്ങൾ:

- സ്വഭാവഭാഷ സംവാദത്തിനുള്ള ലളിതമായ `Bot` ഇന്റർഫേസ് നിർവചിച്ചു
- LangChain4j ൽ നിന്നുള്ള `AiServices` ഉപയോഗിച്ച് LLM MCP ടൂൾ പ്രൊവൈഡറുമായി സ്വയം ബന്ധിപ്പിച്ചു
- ഫോർക്ക്വോർക്ക് ടൂൾ സ്കീമ പരിവർത്തനം, ഫംഗ്ഷൻ കോൾ എല്ലാം പിന്നിൽ കൈകാര്യംചെയ്യുന്നു
- ഇത് മാനുവൽ ടൂൾ മാറ്റം ഒഴിവാക്കി - LangChain4j MCP ഉപകരണങ്ങളെ LLM-സാമർത്ഥ്യമായ ഫോർക്ക്വാർമാറ്റിലേക്ക് മൂന്നുർത്തുന്നു

#### Rust

MCP ടൂൾ പ്രതികരണം LLM മനസ്സിലാക്കുന്ന ഫോർമാറ്റിലേക്ക് പരിവർത്തനം ചെയ്യാൻ സഹായകരമായ ഒരു ഫംഗ്ഷൻ ചേർക്കാം, ഇത് ടൂൾ പട്ടിക ഫോർമാറ്റ് ചെയ്യും. `main.rs` ഫയലിൽ `main` ഫംഗ്ഷനിലുള്ള താഴെ ചേർക്കുക. ഇത് LLM-ക് അഭ്യർത്ഥനകൾ ഫയൽ ചെയ്യുമ്പോൾ വിളക്കും:

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

മികച്ചത്, ഉപയോക്താവിന്റെ അഭ്യർത്ഥനകൾ കൈകാര്യം ചെയ്യാൻ സജ്ജമാകട്ടെ, അടുത്ത ഭാഗത്തേക്ക് പോവാം.

### -4- ഉപയോക്തൃ പ്രോംപ്റ്റ് അഭ്യർത്ഥനം കൈകാര്യം ചെയ്യുക

ഈ കോഡ് ഭാഗം ഉപയോക്തൃ അഭ്യർത്ഥനകൾ കൈകാര്യം ചെയ്യുന്നതാണ്.

#### TypeScript

1. LLM വിളിക്കാൻ ഉപയോഗിക്കുന്ന മെത്തഡ് ചേർക്കുക:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. സെർവറിന്റെ ടൂൾ വിളിക്കുക
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ഫലവുമായി എന്തെങ്കിലും ചെയ്യുക
        // TODO

        }
    }
    ```

    മുൻകൂട്ടിയുള്ള കോഡിൽ:

    - `callTools` എന്ന മെത്തഡ് ചേർത്തു.
    - ഈ മെത്തഡ് LLM പ്രതികരണം പരിശോധിച്ച് വിളിക്കേണ്ട ഉപകരണങ്ങൾ പരിശോധിക്കുന്നു:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // ഉപകരണം വിളിക്കുക
        }
        ```

    - LLM വിളിച്ചേക്കേണ്ടത് സൂചിപ്പിച്ചെങ്കിൽ ഉപകരണം വിളിക്കുന്നു:

        ```typescript
        // 2. സെർവറിന്റെ ഉപകരണം വിളിക്കുക
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ഫലത്തോടൊപ്പം എന്തെങ്കിലും ചെയ്യുക
        // ചെയ്യേണ്ടത്
        ```

1. `run` മെത്തഡ് LLM വിളിക്കലും `callTools` വിളിച്ചുല്ല ഭാഗവും ഉൾപ്പെടുത്തുന്നതായി അപ്ഡേറ്റ് ചെയ്യുക:

    ```typescript

    // 1. LLM ന് ഇൻപുട്ടായി വരുന്ന സന്ദേശങ്ങൾ സൃഷ്ടിക്കുക
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM വിളിക്കുന്നു
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM പ്രതികരണം പരിശോധിക്കുക, ഓരോ തിരഞ്ഞെടുപ്പിനും, അതിൽ ടൂൾ കോളുകളുണ്ടോ എന്ന് പരിശോധിക്കുക
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

മികച്ചത്, മുഴുവൻ കോഡ് ചുവടെ പട്ടികപ്പെടുത്താം:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // സ്കീമ സ്ഥിരീകരണത്തിനു വേണ്ടി zod നു ഇറക്കുമതി ചെയ്യുക

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // ഭാവിയിൽ ഈ url ന് മാറ്റം വരുത്തേണ്ടി വരാം: https://models.github.ai/inference
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
          // input_schema അടിസ്ഥാനമാക്കി ഒരു zod സ്കീമ സൃഷ്ടിക്കുക
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // ടൈപ് "function" എന്ന് വ്യക്തമായി സെറ്റ് ചെയ്യുക
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
    
    
          // 2. സെർവറിന്റെ ടൂൾ ഉപയോഗിക്കുക
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. ഫലം ഉപയോഗിച്ച് എന്തെങ്കിലും ചെയ്യുക
          // ചെയ്യേണ്ടതാണ്
    
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
    
        // 1. LLM പ്രതികരണം പരിശോധിക്കുക, ഓരോ തിരഞ്ഞെടുപ്പിനും ടൂൾ കോളുകൾ ഉണ്ടോ എന്ന് പരിശോധിക്കുക
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

1. LLM വിളിക്കാൻ ആവശ്യമായ ഇന്‍പുട്ടുകള്‍ ഇറക്കുമതി ചെയ്യാം

    ```python
    # എൽഎൽഎം
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. LLM വിളിക്കുന്ന ഫംഗ്ഷൻ ചേർക്കാം:

    ```python
    # എൽഎല്ല്എം

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
            # നിർദ്ദേശം നൽകാവുന്ന പാരാമീറ്ററുകൾ
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

    മുൻകൂട്ടിയുള്ള കോഡിൽ:

    - MCP സെർവറിൽ നിന്നുള്ള ടൂളുകൾ ഫംഗ്ഷനായി LLM-ന് നൽകി.
    - LLM-ന് ഫംഗ്ഷനുകൾ ഉപയോഗിച്ച് അഭ്യർത്ഥിച്ചു.
    - ഫലത്തിൽ കണ്ടുക, ഏത് ഫങ്ഷനുകൾ വിളിക്കേണ്ടതാണെന്ന് അന്വേഷണം നടത്തി.
    - വിളിക്കേണ്ട ഫംഗ്ഷനുകളുടെ ഒരു പട്ടിക സമർപ്പിച്ചു.

1. അന്തിമ ഘട്ടം, മെയിൻ കോഡ് അപ്ഡേറ്റ് ചെയ്യുക:

    ```python
    prompt = "Add 2 to 20"

    # ഏത് ടൂളുകളാണോ ഉണ്ടെങ്കിൽ LLM ലെ ചോദിക്കുക
    functions_to_call = call_llm(prompt, functions)

    # നിർദ്ദേശിച്ച ഫംഗ്ഷനുകൾ വിളിക്കുക
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    മുകളിൽ, MCP ടൂൾ `call_tool` മുഖേന LLM നിർദ്ദേശിച്ച ഫംഗ്ഷൻ ഉപയോഗിച്ച് വിളിക്കുന്നു.
    MCP സെർവറിൽ ടൂൾ കോൾ ഫലം പ്രിന്റ് ചെയ്യുന്നു.

#### .NET

1. LLM പ്രോംപ്റ്റ് അഭ്യർത്ഥനയ്ക്ക് ഉദാഹരണ കോഡ്:

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

    മുൻകൂട്ടിയുള്ള കോഡിൽ:

    - MCP സെർവറിൽ നിന്നുള്ള ടൂൾ കണ്ടെത്തുന്നു, `var tools = await GetMcpTools()`.
    - ഉപയോക്തൃ പ്രോംപ്റ്റ് `userMessage` നിർവചിച്ചു.
    - മോഡൽ, ടൂൾ എന്നിവ നേരത്തേ നിർബന്ധിച്ച ഓപ്ഷനുകൾ സജ്ജീകരിച്ചു.
    - LLM-ന് അഭ്യർത്ഥിച്ചു.

1. അവസാന ഘട്ടം, LLM ഫംഗ്ഷൻ കോൾ നിർദ്ദേശിക്കുന്നുണ്ടോ എന്ന് പരിശോധിക്കുക:

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

    മുൻകൂട്ടിയുള്ള കോഡിൽ:

    - ഫംഗ്ഷൻ കോൾ പട്ടികയിൽ ലൂപ്പ് ചെയ്തു.
    - ഓരോ ടൂൾ കോളിനും പേര്, ആർഗ്യുമെന്റുകൾ പാര്സ് ചെയ്ത് MCP സെർവറിൽ ടൂൾ കോൾ ചെയ്തു, ഫലം പ്രിന്റ് ചെയ്തു.

മുഴുവൻ കോഡ് ചുവടെ:

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
    // സ്വയം MCP ഉപകരണങ്ങൾ ഉപയോഗിച്ച് സ്വാഭാവിക ഭാഷാ അഭ്യർത്ഥനകൾ നടപ്പിലാക്കുക
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

മുൻകൂട്ടിയുള്ള കോഡിൽ:

- ലളിതമായ സ്വഭാവ ഭാഷ പ്രോംപ്റ്റുകൾ MCP സെർവർ ഉപകരണങ്ങളുമായി സംവദിക്കാൻ ഉപയോഗിച്ചു
- LangChain4j ഫ്രെയിംവർക്ക് സവിശേഷങ്ങൾ കൈകാര്യം ചെയ്തു:
  - ആവശ്യത്തിനനുസരിച്ച് ഉപയോക്തൃ പ്രോംപ്റ്റുകളിൽ നിന്ന് ഉപകരണ വിളികൾ സൃഷ്ടിക്കുന്നു
  - LLM തീരുമാന പ്രകാരം MCP യോജ്യമായ ടൂളുകൾ വിളിക്കുന്നു
  - LLM-ഉം MCP സെർവർ-ഉം തമ്മിലുള്ള സംവാദ ഫ്ലോ നിയന്ത്രിക്കുന്നു
- `bot.chat()` സ്വഭാവഭാഷ പ്രതികരണങ്ങൾ നൽകുന്നു, MCP ഉപകരണ ഫലങ്ങളും ഉൾപ്പെടുന്നു
- ഈ സമീപനം സുതാര്യമായി ഉപയോക്താവിന് അനുഭവം നൽകുന്നു, MCP യെക്കുറിച്ചറിയാതെ സുഖമാണ്

പൂർണ്ണ കോഡ് ഉദാഹരണം:

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

ഇവിടെ വർക്ക് ഭൂരിഭാഗവും നടക്കുന്നു. ആദ്യം ഉപയോക്തൃ പ്രോംപ്റ്റ് LLM-ന് വിളിച്ചശേഷം, പ്രതികരണം പരിശോധിച്ച് ഏത് ഉപകരണങ്ങൾ വിളിക്കണമെന്ന് നോക്കും. ആ ഉപകരണങ്ങൾ വിളിച്ച് LLM-നെ വീണ്ടും പോഷിപ്പിക്കും. കൂടുതൽ ഉപകരണ വിളികൾ ആവശ്യമില്ലാത്തപ്പോഴോടെ അവസാന പ്രതികരണം പ്രാപിക്കും.

LLM പലപ്പോഴും വിളിക്കുമെന്ന് ശേഷം, LLM വിളിച് കൈകാര്യം ചെയ്യുന്ന ഫംഗ്ഷൻ നിർവചിക്കാം. `main.rs` ഫയലിൽ താഴെ ചേർക്കുക:

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

ഈ ഫംഗ്ഷൻ LLM ക്ലയന്റ്, സന്ദേശങ്ങൾ (ഉപയോക്തൃ പ്രോംപ്റ്റ് ഉൾപ്പെടെ), MCP ടൂളുകൾ സ്വീകരിച്ച് LLM-ന് അഭ്യർത്ഥന അയയ്ക്കുകയും പ്രതികരണം മടക്കമായി നൽകുകയും ചെയ്യുന്നു.
LLM ന്റെ പ്രതികരണം `choices` എന്ന ഒരു അറേ അടങ്ങിയിരിക്കും. അതിൽ ഏതെങ്കിലും `tool_calls` ഉണ്ടോ എന്ന് പരിശോധിക്കാൻ നമുക്ക് ഫലം പ്രോസസ് ചെയ്യേണ്ടി വരും. LLM ഒരു പ്രത്യേക ടൂൾ ആഗ്യൂമെന്റുകളോടെ വിളിക്കണമെന്ന് ആവശ്യപ്പെടുന്നുണ്ടെന്ന് ഞങ്ങൾക്ക് ഈ വഴി അറിയാം. LLM പ്രതികരണത്തെ കൈകാര്യം ചെയ്യുന്നതിനായുള്ള ഫംഗ്ഷൻ നിർവചിക്കാൻ താഴെയുള്ള കോഡ് നിങ്ങളുടെ `main.rs` ഫയലിന്റെ അടിവസ്ത്രത്തിൽ ചേർക്കുക:

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

    // ഉള്ളത് ഉണ്ടെങ്കിൽ ഉള്ളടക്കം അച്ചടിക്കുക
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // ടൂൾ കോളുകൾ കൈകാര്യം ചെയ്യുക
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // സഹായി സന്ദേശം ചേർക്കുക

        // ഓരോ ടൂൾ കോൾ എക്സിക്യൂട്ട് ചെയ്യുക
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // ടൂൾ ഫലങ്ങൾ സന്ദേശങ്ങളിലേക്ക് ചേർക്കുക
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ടൂൾ ഫലങ്ങളോടൊപ്പം ആശയം തുടരുക
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

`tool_calls` ഉണ്ടെങ്കിൽ, അത് ടൂൾ വിവരങ്ങൾ പുറത്തെടുക്കുന്നു, ടൂൾ അഭ്യർത്ഥനയോടെ MCP സെർവറെ വിളിക്കുന്നു, തുടർന്ന് ഫലങ്ങൾ convo സന്ദേശങ്ങളിലേക്ക് ചേർക്കുന്നു. തുടർന്ന് LLM-സഹിതം സംഭാഷണം തുടരുന്നു, സന്ദേശങ്ങൾ അസിസ്റ്റന്റിൻറെ പ്രതികരണത്തോടും ടൂൾ കോൾ ഫലങ്ങളോടും അപ്ഡേറ്റ് ചെയ്യപ്പെടുന്നു.

LLM MCP കോൾസിനായി ഫലമായി നൽകുന്ന ടൂൾ കോൾ വിവരങ്ങൾ പുറത്തെടുക്കാൻ മറ്റൊരു സഹായ കരാർഘടകം ചേർക്കാം. താഴെയുള്ള കോഡ് നിങ്ങളുടെ `main.rs` ഫയലിന്റെ അടിവസ്ത്രത്തിൽ ചേർക്കുക:

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

എല്ലാ ഘടകങ്ങളും സജ്ജമാക്കിയാൽ, നാം ആദ്യം ഉപയോക്തൃ പ്രാമ്പ്റ്റും LLM-നും കൈകാര്യം ചെയ്യാന് സാധിക്കും. താഴെയുള്ള കോഡ് നിങ്ങളുടെ `main` ഫംഗ്ഷനിൽ അപ്ഡേറ്റ് ചെയ്യുക:

```rust
// ടൂൾ കോളുകൾ ഉൾപ്പെടെയുള്ള LLM സംഭാഷണം
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

ഇത് രണ്ടു സംഖ്യകളുടെ കൂട്ടം ചോദിക്കുന്ന ആദ്യ ഉപയോക്തൃ പ്രാമ്പ്റ്റിനോടൊപ്പം LLM-നെ ക്വറി ചെയ്യും, അതിന്റെ പ്രതികരണം പ്രോസസ്സ് ചെയ്ത് ടൂൾ കോൾസിനെ സർവുകളായ് കൈകാര്യം ചെയ്യും.

ശരിക്കും നന്നായി, നിങ്ങൾ പൂർത്തിയാക്കിയിരിക്കുന്നു!

## അസൈൻമെന്റ്

പരീക്ഷണ കോഡ് കൊണ്ടു സർവർ കൂടുതൽ ടൂളുകൾ ഉപയോഗിച്ച് സജ്ജമാക്കുക. തുടർന്ന് LLM ഉള്ള ക്ലയന്റ് സൃഷ്ടിച്ച് വ്യത്യസ്ത പ്രാമ്പ്റ്റുകളിലൂടെ ടെസ്റ്റ് ചെയ്യുക, നിങ്ങളുടെ സർവർ ടൂളുകൾ ഡൈനാമിക് ആയി വിളിക്കപ്പെടുന്നുണ്ടെന്ന് ഉറപ്പാക്കാൻ. ഈ രീതിയിൽ ക്ലയന്റ് നിർമാണമാക്കുന്നത്, എന്റുയസർക്ക് മനോഹരമായ അനുഭവം നൽകും; അവർ നിർദ്ദിഷ്ട ക്ലയന്റ് കമാൻഡുകൾക്കുപകരം പ്രാമ്പ്റ്റുകൾ ഉപയോഗിച്ച് MCP സെർവർ വിളിക്കപ്പെടുന്നത് തിരിച്ചറിയാതെ ഉപയോഗിക്കാനാകും.

## പരിഹാരം

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## പ്രധാന കാര്യങ്ങൾ

- നിങ്ങളുടെ ക്ലയന്റിൽ LLM ചേർക്കുന്നത് MCP സെർവർസുമായി യഥാർത്ഥ സൗകര്യകരമായ ഇടപാട് ഒരുക്കുന്നു.
- MCP സെർവർ പ്രതികരണം LLM മനസ്സിലാക്കുന്ന രീതിയിലേക്ക് മാറ്റേണ്ടിവരും.

## സാമ്പിളുകൾ

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## അധിക സ്രോതസ്സുകൾ

## അടുത്തത് എന്താണ്

- അടുത്തത്: [Visual Studio Code ഉപയോഗിച്ച് ഒരു സെർവർ ഉപഭോഗിക്കൽ](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അസൂയപ്പെടുത്തൽ**:  
ഈ രേഖ [Co-op Translator](https://github.com/Azure/co-op-translator) എന്ന AI വിവർത്തന സേവനം ഉപയോഗിച്ച് പരിഭാഷപ്പെട്ടതാണ്. ഞങ്ങൾ കൂടിയാണ് പരമമായ കാര്യക്ഷമതയ്ക്കായി ശ്രമിക്കുന്നത്, എങ്കിലും യന്ത്ര വിവർത്തനങ്ങളിൽ പിശകുകൾ അല്ലെങ്കിൽ അസാധുതകൾ ഉണ്ടാകാമെന്ന് ദയവായി തിരിച്ചറിയുക. मूल ഭാഷയിലുള്ള രേഖ തന്നെ അദ്ധിക്ഷിത ഉറവിടമായിരിക്കണം. നിർണ്ണായക വിവരങ്ങൾക്കായി പ്രൊഫഷണൽ മാനവ വിവർത്തനം ശുപാർശ ചെയ്യുന്നു. ഈ വിവർത്തനം ഉപയോഗിക്കുന്നതിനാൽ സൃഷ്ടിക്കുന്നവ കുഴപ്പങ്ങൾ അല്ലെങ്കിൽ തെറ്റിദ്ധാരണകൾക്കായി ഞങ്ങൾ જવાબവാഹികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
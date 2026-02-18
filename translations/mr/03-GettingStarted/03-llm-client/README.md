# LLM सह क्लायंट तयार करणे

आत्तापर्यंत, तुम्ही सर्व्हर आणि क्लायंट कसे तयार करायचे ते पाहिले आहे. क्लायंटने स्पष्टपणे सर्व्हरला कॉल करून त्याचे टूल्स, संसाधने आणि प्रॉम्प्ट्स यादीबद्ध करू शकले होते. तथापि, हे फारसे व्यावहारिक दृष्टिकोन नाही. तुमचा वापरकर्ता एजेन्टिक युगात राहतो आणि प्रॉम्प्ट्स वापरून LLM सोबत संवाद साधण्याची अपेक्षा करतो. तुमच्या वापरकर्त्यास, तुम्ही तुमच्या क्षमता साठवण्यासाठी MCP वापरत असाल की नाही याचा काही फरक पडत नाही, पण ते नैसर्गिक भाषेत संवाद साधण्याची अपेक्षा ठेवतात. तर आपण हे कसे सोडवू? याचे निराकरण म्हणजे क्लायंटमध्ये LLM जोडणे.

## आढावा

या धड्यात आपण LLM जोडण्यावर लक्ष केंद्रित करू, आणि दाखवू की हे तुमच्या वापरकर्त्यास किती चांगला अनुभव देते.

## शिक्षण उद्दिष्टे

या धड्याच्या शेवटी, तुम्ही सक्षम असाल:

- LLM सह क्लायंट तयार करणे.
- MCP सर्व्हरशी LLM वापरून सुरळीत संवाद साधणे.
- क्लायंट बाजूला चांगला अंतिम वापरकर्ता अनुभव प्रदान करणे.

## दृष्टिकोन

चला पाहूया की आपल्याला कोणता दृष्टिकोन घ्यावा लागेल. LLM जोडणे सोपे वाटते, पण आपण खरोखर याला कसे करणार आहोत?

क्लायंट सर्व्हरशी कसा संवाद करेल याचे वर्णन खालीलप्रमाणे:

1. सर्व्हरशी कनेक्शन स्थापित करा.

2. क्षमता, प्रॉम्प्ट्स, संसाधने आणि टूल्सची यादी करा, आणि त्यांचा स्कीमा जतन करा.

3. LLM जोडा आणि जतन केलेल्या क्षमता व त्यांच्या स्कीमांना LLM समजेल अशा स्वरूपात पास करा.

4. वापरकर्त्याचा प्रॉम्प्ट हाताळा, त्याला LLM कडे टाकून क्लायंटने यादी केलेले टूल्स द्या.

छान, आता आपल्याला उच्च पातळीवर कसे करायचे ते कळले, चला खालील व्यायामात प्रयत्न करू.

## व्यायाम: LLM सह क्लायंट तयार करणे

या व्यायामात, आपण आपल्या क्लायंटमध्ये LLM कसा जोडायचा ते शिकू.

### GitHub Personal Access Token वापरून प्रमाणीकरण

GitHub टोकन तयार करणे सोपे आहे. ते कसे करायचे खाली दिले आहे:

- GitHub सेटिंग्जवर जा – वरच्या उजव्या कोपर्‍यातील तुमच्या प्रोफाइल चित्रावर क्लिक करा आणि सेटिंग्ज निवडा.
- Developer Settings वर जा – स्क्रोल करा आणि Developer Settings क्लिक करा.
- Personal Access Tokens निवडा – Fine-grained tokens क्लिक करा आणि नंतर Generate new token बटणावर क्लिक करा.
- तुमचा टोकन कॉन्फिगर करा – संदर्भासाठी नोट जोडा, संपण्याची तारीख सेट करा, आणि आवश्यक परवाने (permissions) निवडा. या प्रकरणात Models permission जोडणे आवश्यक आहे.
- टोकन तयार करा व कॉपी करा – Generate token क्लिक करा आणि लगेच कॉपी करा, कारण नंतर ते पाहता येणार नाही.

### -1- सर्व्हरशी कनेक्ट करा

चला प्रथम आपला क्लायंट तयार करू:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // स्कीमा पडताळणीसाठी zod आयात करा

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

वरच्या कोडमध्ये आम्ही:

- आवश्यक लायब्ररी आयात केली आहेत
- दोन सदस्यांसह क्लास तयार केला आहे, `client` आणि `openai`, जे आम्हाला क्लायंट व्यवस्थापित करण्यासाठी आणि LLM सोबत संवाद साधण्यासाठी मदत करतील.
- आमच्या LLM इन्स्टन्सला GitHub Models वापरण्यासाठी `baseUrl` सेट करून कॉन्फिगर केले आहे जे inference API कडे निर्देशित करते.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio कनेक्शनसाठी सर्व्हर पॅरामीटर्स तयार करा
server_params = StdioServerParameters(
    command="mcp",  # कार्यान्वित करण्यायोग्य
    args=["run", "server.py"],  # ऐच्छिक कमांड लाइन आर्ग्युमेंट्स
    env=None,  # ऐच्छिक पर्यावरणातील व्हेरिएबल्स
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # कनेक्शन प्रारंभ करा
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

वरच्या कोडमध्ये आम्ही:

- MCP साठी आवश्यक लायब्ररी आयात केली आहेत
- क्लायंट तयार केला आहे

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

प्रथम, तुम्हाला LangChain4j अवलंबित्व (`pom.xml`) फाइलमध्ये जोडावे लागेल. MCP एकत्रीकरण आणि GitHub Models समर्थन सक्षम करण्यासाठी खालील अवलंबित्व जोडा:

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

मग तुमचा Java क्लायंट क्लास तयार करा:

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
    
    public static void main(String[] args) throws Exception {        // GitHub मॉडेल्स वापरण्यासाठी LLM कॉन्फिगर करा
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // सर्व्हरशी कनेक्ट होण्यासाठी MCP ट्रान्सपोर्ट तयार करा
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP क्लायंट तयार करा
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

वरच्या कोडमध्ये आम्ही:

- **LangChain4j अवलंबित्वे जोडली**: MCP एकत्रीकरण, OpenAI अधिकृत क्लायंट, आणि GitHub Models साठी आवश्यक
- **LangChain4j लायब्ररी आयात केल्या**: MCP integration आणि OpenAI चॅट मॉडेल कार्यासाठी
- **`ChatLanguageModel` तयार केला**: GitHub Models वापरून तुमच्या GitHub टोकन सह कॉन्फिगर केलेले
- **HTTP ट्रान्सपोर्ट सेट केला**: Server-Sent Events (SSE) वापरून MCP सर्व्हरशी कनेक्ट करण्यासाठी
- **MCP क्लायंट तयार केला**: जे सर्व्हरशी संवाद साधेल
- **LangChain4j चे अंतर्निर्मित MCP समर्थन वापरले**: जे LLMs आणि MCP सर्व्हर्समधील समाकलन सुलभ करते

#### Rust

हा उदाहरण मानतो की तुमच्याकडे Rust आधारित MCP सर्व्हर चालू आहे. जर नाही, तर [01-first-server](../01-first-server/README.md) धडे परत पहा आणि सर्व्हर तयार करा.

तुमचा Rust MCP सर्व्हर तयार केल्यानंतर, टर्मिनल उघडा आणि सर्व्हरच्या डायरेक्टरीत जा. नंतर खालील कमांड वापरून नवीन LLM क्लायंट प्रोजेक्ट तयार करा:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

तुमच्या `Cargo.toml` फाइलमध्ये खालील अवलंबित्व जोडा:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI साठी अधिकृत Rust लायब्ररी नाही, परंतु `async-openai` क्रेट हे [समुदायाद्वारे राखलेली लायब्ररी](https://platform.openai.com/docs/libraries/rust#rust) आहे जी सामान्यपणे वापरली जाते.

`src/main.rs` फाइल उघडा आणि त्याचे कंटेंट खालील कोडने बदला:

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
    // प्रारंभिक संदेश
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // ओपनएआय क्लायंट सेटअप करा
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // एमसीपी क्लायंट सेटअप करा
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

    // TODO: एमसीपी टूल यादी मिळवा

    // TODO: टूल कॉल्ससह LLM संभाषण

    Ok(())
}
```

हा कोड एक बेसिक Rust अ‍ॅप्लिकेशन सेट करतो जे MCP सर्व्हरशी आणि GitHub Models सह LLM संवादासाठी कनेक्ट होईल.

> [!IMPORTANT]
> अ‍ॅप्लिकेशन चालवण्यापूर्वी `OPENAI_API_KEY` पर्यावरण चलात तुमचा GitHub टोकन सेट करायला विसरू नका.

छान, पुढच्या टप्प्याला चला, आता सर्व्हरवरील क्षमता यादी करूया.

### -2- सर्व्हर क्षमता यादी करा

आता आपण सर्व्हरशी कनेक्ट होऊ आणि त्याच्या क्षमता मागवू:

#### Typescript

त्याच वर्गामध्ये खालील पद्धती जोडा:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // साधने यादी करीत आहे
    const toolsResult = await this.client.listTools();
}
```

वरील कोडमध्ये आम्ही:

- `connectToServer` नावाचे सर्व्हरशी कनेक्ट होण्याचे कोड जोडले आहे.
- `run` नावाचा एक मेथड तयार केला आहे जो आमच्या अ‍ॅप्लिकेशन फ्लोचा हाताळणी करतो. सध्या तो फक्त टूल्सची यादी करतो, परंतु आम्ही लवकरच अधिक जोडणार आहोत.

#### Python

```python
# उपलब्ध स्रोतांची यादी करा
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# उपलब्ध साधनांची यादी करा
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

खालील बदल केले आहेत:

- संसाधने आणि टूल्सची यादी केली आणि त्यांना प्रिंट केले. टूल्ससाठी `inputSchema` देखील यादीबद्ध केली आहे, जी नंतर वापरली जात आहे.

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

वरील कोडमध्ये आम्ही:

- MCP सर्व्हरवरील उपलब्ध टूल्सची यादी केली
- प्रत्येक टूलसाठी नाव, वर्णन आणि त्याचा स्कीमा यादीबद्ध केला आहे. हा स्कीमा आपल्याला लवकरच टूल कॉल करण्यात मदत करतो.

#### Java

```java
// असे टूल प्रदात्या तयार करा जे स्वयंचलितपणे MCP टूल्स शोधते
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP टूल प्रदाता स्वयंचलितपणे हाताळतो:
// - MCP सर्व्हरवरून उपलब्ध टूल्सची यादी करणे
// - MCP टूल स्कीमा LangChain4j स्वरूपात रुपांतरित करणे
// - टूल कार्यान्वयन आणि प्रतिसाद व्यवस्थापन करणे
```

वरील कोडमध्ये आम्ही:

- `McpToolProvider` तयार केला जो MCP सर्व्हरमधील सर्व टूल्स आपोआप शोधतो आणि नोंदवतो
- टूल प्रोव्हाइडर आंतरिकरित्या MCP टूल स्कीमा आणि LangChain4j च्या टूल स्वरूपामध्ये रूपांतर करते
- हा दृष्टिकोन मॅन्युअल टूल यादीकरण आणि रूपांतरणाची गरज नाकारतो

#### Rust

MCP सर्व्हरकडून टूल्स मिळविण्यासाठी `list_tools` पद्धत वापरली जाते. `main` फंक्शनमध्ये, MCP क्लायंट तयार केल्यानंतर खालील कोड जोडा:

```rust
// MCP उपकरण सूची प्राप्त करा
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- सर्व्हर क्षमता LLM टूल्समध्ये रूपांतरित करा

सर्व्हर क्षमता यादी करण्यानंतर पुढील टप्पा म्हणजे त्यांना LLM समजेल अशा स्वरूपात रूपांतरित करणे. हे केल्यावर, आपण या क्षमता LLM कडे टूल्स म्हणून देऊ शकतो.

#### TypeScript

1. MCP सर्व्हरच्या प्रतिसादाला LLM वापरू शकणाऱ्या टूल स्वरूपात रूपांतरित करण्यासाठी खालील कोड जोडा:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // इनपुट_स्कीमावर आधारित झोड स्कीमा तयार करा
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // प्रकार स्पष्टपणे "फंक्शन" म्हणून सेट करा
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

मागील कोडमध्ये MCP सर्व्हरचा प्रतिसाद घेतला आणि तो LLM समजू शकणाऱ्या टूल वर्णन स्वरूपात बदलला.

1. नंतर `run` मेथड अपडेट करा जेणेकरून सर्व्हर क्षमता यादी करता येतील:

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

वरील कोडमध्ये, आम्ही `run` मेथडमध्ये नियमितपणे संकलन केलेल्या निकालावर मॅप करून प्रत्येक आयटमसाठी `openAiToolAdapter` कॉल केला आहे.

#### Python

1. प्रथम, खालील रूपांतरण फंक्शन तयार करू:

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

या `convert_to_llm_tools` फंक्शनमध्ये MCP टूल प्रतिसाद घेतला आणि तो LLM समजू शकणाऱ्या स्वरूपात रूपांतरित केला.

1. नंतर, हा फंक्शन वापरून क्लायंट कोड अपडेट करा:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

येथे, आपण MCP टूल प्रतिसाद LLM कडे देण्यासाठी `convert_to_llm_tool` कॉल करत आहोत.

#### .NET

1. MCP टूल प्रतिसाद LLM समजेल अशा स्वरूपात रूपांतरित करण्यासाठी कोड जोडा:

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

वरील कोडमध्ये आम्ही:

- `ConvertFrom` नावाचे फंक्शन तयार केले जे नाव, वर्णन आणि इनपुट स्कीमा घेते.
- एक `FunctionDefinition` तयार करतो जे `ChatCompletionsDefinition` ला पास होते, जो LLM समजू शकणारा आहे.

1. आता आधीच्या कोडमध्ये हे फंक्शन कसे वापरायचे ते पाहू:

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
// नैसर्गिक भाषा संवादासाठी बोट इंटरफेस तयार करा
public interface Bot {
    String chat(String prompt);
}

// LLM आणि MCP साधनांसह AI सेवा कॉन्फिगर करा
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

वरील कोडमध्ये आम्ही:

- नॅचरल लँग्वेज संवादासाठी साधी `Bot` इंटरफेस परिभाषित केली आहे
- LangChain4j च्या `AiServices` वापरून LLM आणि MCP टूल प्रोव्हायडर आपोआप जुळवले आहे
- फ्रेमवर्क टूल स्कीमा रूपांतरण आणि फंक्शन कॉलिंग मागोमाग हाताळते
- हा दृष्टिकोन मॅन्युअल टूल रूपांतरण टाळतो — LangChain4j हे सर्व रूपांतरणाचा क्लिष्ट भाग हाताळतो

#### Rust

MCP टूल प्रतिसाद LLM समजू शकणाऱ्या स्वरूपात रूपांतरित करण्यासाठी, आपण एक हेल्पर फंक्शन जोडू जे टूल सूचीचे स्वरूपित करेल. `main.rs` फाइलमध्ये `main` फंक्शनखाली खालील कोड जोडा. हा LLM कडे विनंत्या करताना वापरला जाईल:

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

छान, आता आपण वापरकर्त्याच्या कोणत्याही विनंत्या हाताळण्यासाठी तयार आहोत, तर पुढे चालू ठेवूया.

### -4- वापरकर्ता प्रॉम्प्ट विनंती हाताळा

या कोडमध्ये आपण वापरकर्त्याच्या विनंत्या हाताळणार आहोत.

#### TypeScript

1. आपल्या LLM ला कॉल करण्यासाठी खालील मेथड जोडा:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. सर्व्हरच्या साधनाला कॉल करा
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. निकालासह काहीतरी करा
        // TODO

        }
    }
    ```

वरील कोडमध्ये:

- `callTools` नावाची मेथड जोडली आहे.
- ह्या मेथड मध्ये LLM प्रतिसाद तपासला जातो की कोणते टूल्स कॉल झाले आहेत का:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // टूल कॉल करा
        }
        ```

- जर LLM सूचित करत असेल तर टूल कॉल केली जाते:

        ```typescript
        // 2. सर्व्हरच्या टूलला कॉल करा
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. निकालासोबत काहीतरी करा
        // करायचे
        ```

1. `run` मेथड मध्ये अपडेट करा जेणेकरून LLM कॉल्स आणि `callTools` यांचा सतत वापर होईल:

    ```typescript

    // 1. LLM साठी इनपुट म्हणून मेसेजेस तयार करा
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM कॉल करणे
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM उत्तर पहा, प्रत्येक निवडीसाठी तपासा की त्यात टूल कॉल्स आहेत का
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

छान, पूर्ण कोड यादी बघूया:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // स्कीमा प्रमाणीकरणासाठी zod आयात करा

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // भविष्यात कदाचित या URL वर बदलण्याची गरज भासू शकते: https://models.github.ai/inference
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
          // इनपुट_स्कीमा आधारित zod स्कीमा तयार करा
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // प्रकार स्पष्टपणे "function" सेट करा
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
    
    
          // 2. सर्व्हरचे टूल कॉल करा
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. निकालासह काहीतरी करा
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
    
        // 1. LLM प्रतिसाद पाहा, प्रत्येक पर्यायासाठी तपासा की त्यात टूल कॉल आहेत का
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

1. LLM कॉल करण्यासाठी आवश्यक आयात जोडा:

    ```python
    # llm
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. नंतर LLM कॉल करणारी फंक्शन जोडा:

    ```python
    # एलएलएम

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
            # ऐच्छिक पॅरामीटर्स
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

वरील कोडमध्ये:

- MCP सर्व्हरवरून जे फंक्शन्स मिळाले आणि रूपांतरित केले, ते LLM कडे दिले.
- नंतर LLM कॉल केला.
- मग निकाल तपासून पाहतो कोणते फंक्शन्स कॉल करायला हवेत.
- शेवटी, कॉल करावयाच्या फंक्शन्सची यादी पास केली.

1. अंतिम टप्पा, मुख्य कोड अपडेट करा:

    ```python
    prompt = "Add 2 to 20"

    # सर्वांसाठी कोणती साधने वापरायची आहेत का, LLM ला विचारा
    functions_to_call = call_llm(prompt, functions)

    # सुचवलेल्या फंक्शन्सला कॉल करा
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

वरील कोडमध्ये:

- वापरकर्ता प्रॉम्प्टनुसार LLM निर्णयावर आधारित MCP टूल `call_tool` द्वारे कॉल केला आहे.
- टूल कॉलचा निकाल MCP सर्व्हरवर प्रिंट केला.

#### .NET

1. LLM प्रॉम्प्ट विनंतीसाठी कोड:

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

वरील कोडमध्ये:

- MCP सर्व्हरकडून टूल्स मिळवले (`var tools = await GetMcpTools()`).
- वापरकर्त्याचा प्रॉम्प्ट तयार केला (`userMessage`).
- मॉडेल व टूल्ससहित ऑप्शन्स ऑब्जेक्ट तयार केला.
- LLM कडे विनंती केली.

1. शेवटचा टप्पा, LLM ला विचारू की कोणते फंक्शन्स कॉल करावेत:

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

वरील कोडमध्ये:

- फंक्शन कॉल्सची सूची लूप केली.
- प्रत्येक टूल कॉलसाठी नाव व अर्ग्युमेंट्स पार्स करून MCP सर्व्हरवर टूल कॉल केला आणि निकाल प्रिंट केला.

पूर्ण कोड खालीलप्रमाणे आहे:

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
    // MCP साधने स्वयंचलितपणे वापरून नैसर्गिक भाषा विनंत्या कार्यान्वित करा
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

वरील कोडमध्ये:

- साध्या नैसर्गिक भाषेतील प्रॉम्प्ट्स वापरून MCP टूल्सशी संवाद साधला.
- LangChain4j फ्रेमवर्क आपोआप हाताळतो:
  - वापरकर्त्याचा प्रॉम्प्ट टूल कॉलमध्ये रूपांतर करणे
  - LLM निर्णयावर योग्य MCP टूल्स कॉल करणे
  - LLM व MCP सर्व्हर दरम्यान संभाषण प्रवाह व्यवस्थापित करणे
- `bot.chat()` मेथड नैसर्गिक भाषा उत्तरं परत करते ज्यात MCP टूल्सचे निकाल असू शकतात
- या दृष्टिकोनामुळे वापरकर्त्यांना अंतर्गत MCP रचना माहित न ठेवता सहज संवाद साधता येतो

पूर्ण कोड उदाहरण:

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

महा-कार्य येथे घडते. आपण सुरुवातीचा वापरकर्ता प्रॉम्प्ट LLM कडे पाठवू, मग प्रतिक्रिया पाहून कोणतेही टूल्स कॉल करायची गरज आहे का ते पाहू. असे असल्यास, ते टूल्स कॉल करू व LLM सोबत संभाषण चालू ठेवू जोपर्यंत अजून टूल कॉल्सची गरज नसते आणि अंतिम प्रतिसाद मिळतो.

आपण LLM कडे अनेक वेळा कॉल करणार आहोत, म्हणून LLM कॉल हाताळणारे फंक्शन तयार करूया. हे फंक्शन `main.rs` फाइलमध्ये जोडा:

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

हे फंक्शन LLM क्लायंट, संदेशांची यादी (वापरकर्ता प्रॉम्प्टसह), MCP सर्व्हरचे टूल्स घेतो, विनंती LLM कडे पाठवतो आणि प्रतिसाद परत करते.
LLM कडून आलेल्या प्रतिसादामध्ये `choices` नावाचा एक अनुक्रम असेल. आम्हाला निकाल प्रक्रिया करावी लागेल जेणेकरून कोणतेही `tool_calls` अस्तित्वात आहेत का ते पाहू शकू. यामुळे आम्हाला कळेल की LLM विशिष्ट साधनाला कॉल करण्यासाठी विनंती करत आहे ज्यासाठी तर्क दिले गेले आहेत. LLM प्रतिसाद हाताळण्यासाठीचा एक फंक्शन निश्चित करण्यासाठी आपल्या `main.rs` फाइलच्या खालील भागात पुढील कोड जोडा:

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

    // सामग्री उपलब्ध असल्यास मुद्रित करा
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // साधन कॉल हाताळा
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // सहाय्यक संदेश जोडा

        // प्रत्येक साधन कॉल अंमलात आणा
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // संदेशांमध्ये साधन निकाल जोडा
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // साधन निकालांसह संभाषण सुरू ठेवा
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

जर `tool_calls` उपस्थित असतील तर, तो साधनाची माहिती काढतो, MCP सर्वरला साधन विनंतीसह कॉल करतो, आणि निकाल संभाषण संदेशांमध्ये जोडतो. नंतर तो LLM सोबत संभाषण पुढे चालू ठेवतो आणि संदेश सहायकाच्या प्रतिसाद व साधन कॉल निकालांनी अपडेट होतात.

MCP कॉलसाठी LLM परत करत असलेल्या साधन कॉल माहिती काढण्यासाठी, आम्ही आणखी एक हेल्पर फंक्शन जोडू जे कॉलसाठी आवश्यक असलेल्या सर्व काही काढेल. आपल्या `main.rs` फाइलच्या खालील भागात पुढील कोड जोडा:

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

सर्व भाग व्यवस्थित असल्यावर, आपण प्रारंभिक वापरकर्ता प्रॉम्प्ट हाताळू आणि LLM कॉल करू शकतो. आपल्या `main` फंक्शनमध्ये पुढील कोड समाविष्ट करा:

```rust
// टूल कॉल्ससह LLM संभाषण
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

हे प्रारंभिक वापरकर्ता प्रॉम्प्टसह LLM चे क्वेरी करेल, जे दोन संख्यांचा बेरीज विचारते, आणि प्रतिसाद प्रक्रियेतून डायनामिकपणे साधन कॉल हाताळेल.

छान, तुम्ही ते केलं!

## कार्य

व्यायामातील कोड काढून आणखी काही साधनांसह सर्व्हर तयार करा. नंतर व्यायामाप्रमाणे LLM असलेला क्लायंट तयार करा आणि वेगवेगळ्या प्रॉम्प्टसह त्याचा चाचणी करा जेणेकरून तुमच्या सर्व्हरचे सर्व साधन डायनामिकपणे कॉल होतात याची खात्री करता येईल. अशा प्रकारे क्लायंट तयार करणे म्हणजे शेवटचा वापरकर्ता उत्तम वापरकर्ता अनुभव घेईल कारण ते प्रॉम्प्ट वापरू शकतील, अचूक क्लायंट आदेशाऐवजी, आणि कोणतेही MCP सर्व्हर कॉल होत असल्याचे त्यांना लक्षात येणार नाही.

## समाधान

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## मुख्य मुद्दे

- तुमच्या क्लायंटमध्ये LLM जोडल्याने वापरकर्त्यांसाठी MCP सर्व्हरशी संवाद साधण्याचा चांगला मार्ग उपलब्ध होतो.
- तुम्हाला MCP सर्व्हरच्या प्रतिसादाला LLM समजू शकणाऱ्या स्वरूपात रूपांतरित करणे आवश्यक आहे.

## नमुने

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## अतिरिक्त संसाधने

## पुढे काय

- पुढे: [Visual Studio Code वापरून सर्व्हरचा वापर](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**सूचना**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून भाषांतरित केला आहे. आम्ही अचूकतेसाठी प्रयत्नशील असलो तरी, कृपया ध्यानात ठेवा की स्वयंचलित भाषांतरांमध्ये चुका किंवा अचूकतेचे त्रुटी असू शकतात. मूळ दस्तऐवज त्याच्या स्थानिक भाषेत अधिकृत स्रोत मानला जावा. महत्त्वाच्या माहितीसाठी, व्यावसायिक मानवी भाषांतर सुचवले जाते. या भाषांतराच्या वापरामुळे झालेल्या कोणत्याही गैरसमजुतींबाबत किंवा चुकीच्या समजुतींबाबत आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
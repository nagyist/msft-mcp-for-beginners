# LLM के साथ क्लाइंट बनाना

अब तक, आपने देखा है कि एक सर्वर और एक क्लाइंट कैसे बनाया जाता है। क्लाइंट विशेष रूप से सर्वर को कॉल करके उसके टूल्स, संसाधन और प्रॉम्प्ट्स की सूची बना सकता है। हालांकि, यह बहुत व्यावहारिक तरीका नहीं है। आपका उपयोगकर्ता एजेंटिक युग में रहता है और प्रॉम्प्ट्स का उपयोग करने और एक LLM के साथ संवाद करने की उम्मीद करता है। आपके उपयोगकर्ता के लिए, वे इस बात की परवाह नहीं करते कि आप अपनी क्षमताएँ स्टोर करने के लिए MCP का उपयोग करते हैं या नहीं, लेकिन वे प्राकृतिक भाषा का उपयोग करके बातचीत करने की अपेक्षा करते हैं। तो हम इसे कैसे हल करें? समाधान यह है कि क्लाइंट में एक LLM जोड़ना।

## अवलोकन

इस पाठ में हम अपने क्लाइंट में एक LLM जोड़ने पर ध्यान केंद्रित करेंगे और यह दिखाएंगे कि यह आपके उपयोगकर्ता के लिए एक बेहतर अनुभव कैसे प्रदान करता है।

## सीखने के उद्देश्य

इस पाठ के अंत तक, आप सक्षम होंगे:

- LLM के साथ एक क्लाइंट बनाना।
- LLM का उपयोग करके MCP सर्वर के साथ सहज बातचीत करना।
- क्लाइंट पक्ष पर बेहतर एंड यूजर अनुभव प्रदान करना।

## दृष्टिकोण

आइए समझते हैं कि हमें किस दृष्टिकोण को अपनाना है। एक LLM जोड़ना सरल लगता है, लेकिन क्या हम वास्तव में ऐसा करेंगे?

यहाँ बताया गया है कि क्लाइंट सर्वर के साथ कैसे इंटरैक्ट करेगा:

1. सर्वर के साथ कनेक्शन स्थापित करें।

1. क्षमताओं, प्रॉम्प्ट्स, संसाधनों और टूल्स की सूची बनाएं, और उनकी स्कीमा को सेव करें।

1. एक LLM जोड़ें और सेव की गई क्षमताओं और उनकी स्कीमा को एक ऐसे फॉर्मेट में पास करें जिसे LLM समझता हो।

1. उपयोगकर्ता प्रॉम्प्ट को हैंडल करें, उसे LLM को पास करके और क्लाइंट द्वारा सूचीबद्ध टूल्स के साथ।

बहुत अच्छा, अब जब हम समझ चुके हैं कि उच्च स्तर पर यह कैसे किया जा सकता है, तो नीचे दिए गए अभ्यास में इसे आजमाएं।

## अभ्यास: LLM के साथ क्लाइंट बनाना

इस अभ्यास में, हम अपने क्लाइंट में एक LLM जोड़ना सीखेंगे।

### GitHub Personal Access Token का उपयोग करके प्रमाणीकरण

GitHub टोकन बनाना एक आसान प्रक्रिया है। आप इसे इस प्रकार कर सकते हैं:

- GitHub सेटिंग्स पर जाएं – टॉप राइट कॉर्नर में अपनी प्रोफाइल पिक्चर पर क्लिक करें और सेटिंग्स चुनें।
- Developer Settings पर जाएं – नीचे स्क्रॉल करें और Developer Settings पर क्लिक करें।
- Personal Access Tokens चुनें – Fine-grained tokens पर क्लिक करें और फिर Generate new token चुनें।
- अपना टोकन कॉन्फ़िगर करें – संदर्भ के लिए एक नोट जोड़ें, समाप्ति तिथि सेट करें, और आवश्यक स्कोप्स (अनुमतियाँ) चुनें। इस मामले में Models अनुमति जरूर जोड़ें।
- टोकन उत्पन्न करें और कॉपी करें – Generate token पर क्लिक करें, और तुरंत इसे कॉपी करना सुनिश्चित करें, क्योंकि इसे बाद में फिर से नहीं देखा जा सकता।

### -1- सर्वर से कनेक्ट करें

पहले अपने क्लाइंट को बनाते हैं:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // स्कीमा सत्यापन के लिए zod आयात करें

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

पिछले कोड में हमने:

- आवश्यक लाइब्रेरियां इम्पोर्ट कीं
- एक क्लास बनाई जिसमें दो सदस्य हैं, `client` और `openai`, जो हमें क्लाइंट मैनेज करने और LLM के साथ इंटरैक्ट करने में मदद करेंगे।
- अपने LLM इंस्टेंस को GitHub Models का उपयोग करने के लिए कॉन्फ़िगर किया, `baseUrl` को inference API की ओर सेट करके।

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio कनेक्शन के लिए सर्वर पैरामीटर बनाएं
server_params = StdioServerParameters(
    command="mcp",  # निष्पादन योग्य
    args=["run", "server.py"],  # वैकल्पिक कमांड लाइन तर्क
    env=None,  # वैकल्पिक पर्यावरण चर
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # कनेक्शन प्रारंभ करें
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

पिछले कोड में हमने:

- MCP के लिए आवश्यक लाइब्रेरियां इम्पोर्ट कीं
- एक क्लाइंट बनाया

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

पहले, आपको अपनी `pom.xml` फ़ाइल में LangChain4j निर्भरताएँ जोड़नी होंगी। MCP इंटीग्रेशन और GitHub Models सपोर्ट सक्षम करने के लिए ये निर्भरताएँ जोड़ें:

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

फिर अपनी Java क्लाइंट क्लास बनाएं:

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
    
    public static void main(String[] args) throws Exception {        // LLM को GitHub मॉडल्स का उपयोग करने के लिए कॉन्फ़िगर करें
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // सर्वर से कनेक्ट करने के लिए MCP ट्रांसपोर्ट बनाएं
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP क्लाइंट बनाएं
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

पिछले कोड में हमने:

- **LangChain4j निर्भरताएँ जोड़ीं**: MCP इंटीग्रेशन, OpenAI आधिकारिक क्लाइंट, और GitHub Models सपोर्ट के लिए आवश्यक
- **LangChain4j लाइब्रेरियां इम्पोर्ट कीं**: MCP इंटीग्रेशन और OpenAI चैट मॉडल कार्यक्षमता के लिए
- **`ChatLanguageModel` बनाई**: GitHub Models का उपयोग करने के लिए आपकी GitHub टोकन के साथ कॉन्फ़िगर किया
- **HTTP ट्रांसपोर्ट सेटअप किया**: Server-Sent Events (SSE) का उपयोग करके MCP सर्वर से कनेक्ट किया
- **MCP क्लाइंट बनाया**: जो सर्वर के साथ संचार संभालेगा
- **LangChain4j के बिल्ट-इन MCP सपोर्ट का उपयोग किया**: जो LLMs और MCP सर्वर के बीच इंटीग्रेशन को सरल बनाता है

#### Rust

यह उदाहरण मानता है कि आपके पास एक Rust आधारित MCP सर्वर चल रहा है। यदि आपके पास नहीं है, तो सर्वर बनाने के लिए [01-first-server](../01-first-server/README.md) पाठ देखें।

जब आपके पास Rust MCP सर्वर हो, तो एक टर्मिनल खोलें और सर्वर के समान डायरेक्टरी में जाएं। फिर एक नया LLM क्लाइंट प्रोजेक्ट बनाने के लिए निम्न आदेश चलाएं:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

अपने `Cargo.toml` फ़ाइल में निम्न निर्भरताएँ जोड़ें:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI के लिए एक आधिकारिक Rust लाइब्रेरी नहीं है, परंतु `async-openai` क्रेट एक [समुदाय द्वारा रखी गई लाइब्रेरी](https://platform.openai.com/docs/libraries/rust#rust) है जो आमतौर पर उपयोग की जाती है।

`src/main.rs` फ़ाइल खोलें और इसके कंटेंट को निम्न कोड से बदलें:

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

    // OpenAI क्लाइंट सेटअप करें
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP क्लाइंट सेटअप करें
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

    // TODO: MCP टूल सूची प्राप्त करें

    // TODO: टूल कॉल के साथ LLM बातचीत

    Ok(())
}
```

यह कोड एक बेसिक Rust एप्लिकेशन सेट करता है जो MCP सर्वर और GitHub Models के साथ LLM इंटरैक्शन के लिए कनेक्ट होगा।

> [!IMPORTANT]
> एप्लिकेशन चलाने से पहले अपने GitHub टोकन के साथ `OPENAI_API_KEY` पर्यावरण चर सेट करना सुनिश्चित करें।

बहुत बढ़िया, अगले कदम के लिए, चलिए सर्वर की क्षमताएँ सूचीबद्ध करते हैं।

### -2- सर्वर की क्षमताओं की सूची बनाएं

अब हम सर्वर से कनेक्ट करेंगे और इसकी क्षमताओं के बारे में पूछेंगे:

#### Typescript

उसी क्लास में निम्नलिखित मेथड जोड़ें:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // उपकरणों की सूची बनाना
    const toolsResult = await this.client.listTools();
}
```

पिछले कोड में हमने:

- सर्वर से कनेक्ट करने के लिए कोड जोड़ा, `connectToServer`।
- `run` मेथड बनाई जो हमारे ऐप के फ्लो को संभालती है। अभी तक यह सिर्फ टूल्स की सूची बनाती है, लेकिन हम जल्द ही इसमें और जोड़ेंगे।

#### Python

```python
# उपलब्ध संसाधनों की सूची बनाएं
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# उपलब्ध उपकरणों की सूची बनाएं
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

यहाँ हमने जोड़ा है:

- संसाधनों और टूल्स की सूची बनाई और उन्हें प्रिंट किया। टूल्स के लिए हमने `inputSchema` भी सूचीबद्ध किया जिसका बाद में उपयोग करेंगे।

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

पिछले कोड में हमने:

- MCP सर्वर पर उपलब्ध टूल्स की सूची बनाई
- हर टूल के लिए नाम, विवरण और इसकी स्कीमा सूचीबद्ध की। बाद में हम इसका उपयोग टूल कॉल करने के लिए करेंगे।

#### Java

```java
// एक टूल प्रदाता बनाएँ जो स्वचालित रूप से MCP टूल्स की खोज करता है
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP टूल प्रदाता स्वचालित रूप से संभालता है:
// - MCP सर्वर से उपलब्ध टूल्स की सूची बनाना
// - MCP टूल स्कीमाओं को LangChain4j फॉर्मेट में परिवर्तित करना
// - टूल निष्पादन और प्रतिक्रियाओं का प्रबंधन करना
```

पिछले कोड में हमने:

- एक `McpToolProvider` बनाया जो अपने आप MCP सर्वर से सभी टूल खोजता और रजिस्टर करता है
- टूल प्रोवाइडर MCP टूल स्कीमा और LangChain4j के टूल फॉर्मेट के बीच रूपांतरण को आंतरिक रूप से संभालता है
- यह दृष्टिकोण मैन्युअल टूल लिस्टिंग और रूपांतरण प्रक्रिया को छुपाता है

#### Rust

MCP सर्वर से टूल्स प्राप्त करना `list_tools` मेथड के उपयोग से किया जाता है। अपने `main` फ़ंक्शन में, MCP क्लाइंट सेटअप करने के बाद निम्न कोड जोड़ें:

```rust
// MCP टूल सूची प्राप्त करें
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- सर्वर क्षमताओं को LLM टूल्स में परिवर्तित करें

सर्वर की क्षमताओं की सूची बनाने के बाद अगला कदम उन्हें LLM के समझने योग्य फॉर्मेट में बदलना है। एक बार ऐसा कर लेने पर, हम इन क्षमताओं को अपने LLM को टूल्स के रूप में प्रदान कर सकते हैं।

#### TypeScript

1. MCC सर्वर की प्रतिक्रिया को LLM के उपयोग के लिहाज से टूल फॉर्मेट में बदलने के लिए निम्न कोड जोड़ें:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // इनपुट_schema के आधार पर एक zod स्कीमा बनाएं
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // प्रकार को स्पष्ट रूप से "function" सेट करें
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

    ऊपर वाला कोड MCP सर्वर से प्रतिक्रिया लेता है और उसे एक टूल परिभाषा में कन्वर्ट करता है जिसे LLM समझता है।

1. चलिए अगला काम `run` मेथड को अपडेट करें ताकि सर्वर क्षमताओं को सूचीबद्ध किया जा सके:

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

    पिछले कोड में, हमने `run` मेथड को अपडेट किया है जो रिजल्ट में मैप करता है और प्रत्येक एंट्री के लिए `openAiToolAdapter` कॉल करता है।

#### Python

1. पहले, निम्न कन्वर्टर फंक्शन बनाएं

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

    उपरोक्त `convert_to_llm_tools` फंक्शन में हम MCP टूल प्रतिक्रिया को एक ऐसे फॉर्मेट में परिवर्तित करते हैं जिसे LLM समझ सके।

1. फिर, अपने क्लाइंट कोड को इस फंक्शन का उपयोग करने के लिए अपडेट करें:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    यहाँ, हम `convert_to_llm_tool` कॉल जोड़ रहे हैं ताकि MCP टूल प्रतिक्रिया को LLM के लिए उपयुक्त फॉर्मेट में बदला जा सके।

#### .NET

1. MCP टूल प्रतिक्रिया को LLM के समझने योग्य फॉर्मेट में बदलने के लिए कोड जोड़ें:

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

पिछले कोड में हमने:

- `ConvertFrom` नामक फंक्शन बनाया जो नाम, विवरण और इनपुट स्कीमा लेता है।
- उस फंक्शन में एक `FunctionDefinition` बनती है जो `ChatCompletionsDefinition` में जाती है। यह वह फॉर्मेट है जिसे LLM समझता है।

1. आइए देखें कि पहले से मौजूद कोड को इस फंक्शन का लाभ लेने के लिए कैसे अपडेट किया जाए:

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
// प्राकृतिक भाषा इंटरैक्शन के लिए एक बोट इंटरफेस बनाएं
public interface Bot {
    String chat(String prompt);
}

// LLM और MCP टूल्स के साथ AI सेवा को कॉन्फ़िगर करें
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

पिछले कोड में हमने:

- प्राकृतिक भाषा इंटरैक्शन्स के लिए एक सरल `Bot` इंटरफेस बनाया
- LangChain4j के `AiServices` का उपयोग किया जो LLM को MCP टूल प्रोवाइडर के साथ स्वचालित रूप से जोड़ता है
- फ्रेमवर्क टूल स्कीमा रूपांतरण और फंक्शन कॉलिंग को अपने आप संभालता है
- इस दृष्टिकोण से मैन्युअल टूल रूपांतरण खत्म हो जाता है — LangChain4j सभी जटिलताओं को संभालता है

#### Rust

MCP टूल प्रतिक्रिया को LLM के समझने योग्य फॉर्मेट में बदलने के लिए, हम एक हेल्पर फंक्शन जोड़ेंगे जो टूल्स की लिस्टिंग को फॉर्मेट करता है। `main` फ़ंक्शन के नीचे अपनी `main.rs` फ़ाइल में निम्न कोड जोड़ें। यह LLM को अनुरोध करते समय कॉल किया जाएगा:

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

बहुत बढ़िया, अब हम किसी भी उपयोगकर्ता अनुरोध को संभालने के लिए तैयार हैं, तो चलिए अगला काम करते हैं।

### -4- उपयोगकर्ता प्रॉम्प्ट अनुरोध को हैंडल करें

इस कोड भाग में, हम उपयोगकर्ता अनुरोधों को हैंडल करेंगे।

#### TypeScript

1. एक ऐसा मेथड जोड़ें जो हमारे LLM को कॉल करने के लिए इस्तेमाल किया जाएगा:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. सर्वर के टूल को कॉल करें
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. परिणाम के साथ कुछ करें
        // टीओडीओ

        }
    }
    ```

    पिछले कोड में हमने:

    - `callTools` नामक मेथड जोड़ी।
    - यह मेथड LLM प्रतिक्रिया लेती है और देखती है कि कौन से टूल्स कॉल हुए हैं, यदि कोई हैं:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // टूल कॉल करें
        }
        ```

    - यदि LLM संकेत देता है कि कोई टूल कॉल करना चाहिए तो वह टूल कॉल करता है:

        ```typescript
        // 2. सर्वर के टूल को कॉल करें
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. परिणाम के साथ कुछ करें
        // करना है
        ```

1. `run` मेथड को अपडेट करें ताकि LLM कॉल और `callTools` को शामिल किया जा सके:

    ```typescript

    // 1. LLM के लिए इनपुट संदेश बनाएं
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM को कॉल करना
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM प्रतिक्रिया देखें, प्रत्येक विकल्प के लिए जांचें कि क्या उसमें टूल कॉल हैं
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

बहुत बढ़िया, पूरा कोड निम्नलिखित है:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // स्कीमा सत्यापन के लिए zod आयात करें

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // भविष्य में यह URL बदलने की आवश्यकता हो सकती है: https://models.github.ai/inference
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
          // input_schema के आधार पर एक zod स्कीमा बनाएं
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // स्पष्ट रूप से प्रकार को "function" सेट करें
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
    
    
          // 2. सर्वर के टूल को कॉल करें
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. परिणाम के साथ कुछ करें
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
    
        // 1. LLM प्रतिक्रिया के माध्यम से जाएं, प्रत्येक विकल्प के लिए, जांचें कि उसमें टूल कॉल हैं या नहीं
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

1. LLM कॉल करने के लिए आवश्यक कुछ इम्पोर्ट्स जोड़ें:

    ```python
    # एलएलएम
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. फिर, LLM कॉल करने वाला फंक्शन जोड़ें:

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
            # वैकल्पिक पैरामीटर
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

    पिछले कोड में हमने:

    - MCP सर्वर पर मिले और कन्वर्ट किए गए फंक्शन्स LLM को पास किए।
    - फिर LLM को उक्त फंक्शन्स के साथ कॉल किया।
    - परिणाम की जाँच की कि किन फंक्शन्स को कॉल करना चाहिए, यदि कोई हैं।
    - आखिर में कॉल करने के लिए फंक्शन्स की एक सूची पास की।

1. अंतिम कदम, हमारे मुख्य कोड को अपडेट करें:

    ```python
    prompt = "Add 2 to 20"

    # सभी उपकरणों के बारे में LLM से पूछें, यदि कोई हो
    functions_to_call = call_llm(prompt, functions)

    # सुझाए गए फ़ंक्शंस को कॉल करें
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    ऊपर दिए कोड में हम:

    - LLM के प्रॉम्प्ट के आधार पर MCP टूल को `call_tool` द्वारा कॉल करते हैं।
    - MCP सर्वर से टूल कॉल का परिणाम प्रिंट करते हैं।

#### .NET

1. LLM प्रॉम्प्ट अनुरोध करने का कोड देखें:

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

    पिछले कोड में हमने:

    - MCP सर्वर से टूल्स प्राप्त किए, `var tools = await GetMcpTools()`।
    - उपयोगकर्ता प्रॉम्प्ट `userMessage` परिभाषित किया।
    - मोडेल और टूल्स निर्दिष्ट करते हुए ऑप्शन्स ऑब्जेक्ट बनाया।
    - LLM की ओर अनुरोध भेजा।

1. अंतिम कदम, देखें कि क्या LLM सोचता है कि हमें कोई फंक्शन कॉल करनी चाहिए:

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

    पिछले कोड में हमने:

    - फंक्शन कॉल्स की सूची में लूप किया।
    - प्रत्येक टूल कॉल के लिए नाम और आर्गुमेंट्स पर्स किए और MCP क्लाइंट उपयोग करके MCP सर्वर पर टूल कॉल की। अंत में परिणाम प्रिंट किए।

पूरा कोड यहाँ है:

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
    // स्वचालित रूप से MCP टूल्स का उपयोग करने वाले प्राकृतिक भाषा अनुरोधों को निष्पादित करें
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

पिछले कोड में हमने:

- सरल प्राकृतिक भाषा प्रॉम्प्ट्स का उपयोग करके MCP सर्वर टूल्स के साथ इंटरैक्ट किया
- LangChain4j फ्रेमवर्क स्वचालित रूप से संभालता है:
  - जरूरत पड़ने पर उपयोगकर्ता प्रॉम्प्ट को टूल कॉल्स में कन्वर्ट करना
  - LLM के निर्णय के आधार पर उचित MCP टूल्स कॉल करना
  - LLM और MCP सर्वर के बीच बातचीत का प्रबंधन करना
- `bot.chat()` मेथड प्राकृतिक भाषा प्रतिक्रियाएं लौटाता है जिनमें MCP टूल निष्पादन के परिणाम शामिल हो सकते हैं
- यह दृष्टिकोण उपयोगकर्ता को MCP की अंतर्निहित कार्यप्रणाली के बारे में जानने की आवश्यकता नहीं होने देता है

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

यहाँ अधिकांश काम होता है। हम आरंभिक उपयोगकर्ता प्रॉम्प्ट के साथ LLM को कॉल करेंगे, फिर प्रतिक्रिया की जांच करेंगे कि क्या किसी टूल को कॉल करना आवश्यक है। यदि हाँ, तो हम उन टूल्स को कॉल करेंगे और तब तक LLM के साथ बातचीत जारी रखेंगे जब तक और टूल कॉल की आवश्यकता न हो और हमें अंतिम प्रतिक्रिया न मिल जाए।

हम कई बार LLM को कॉल करेंगे, इसलिए एक ऐसा फंक्शन परिभाषित करें जो LLM कॉल को संभाले। अपने `main.rs` फ़ाइल में निम्न फंक्शन जोड़ें:

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

यह फंक्शन LLM क्लाइंट, संदेशों (जिसमें उपयोगकर्ता प्रॉम्प्ट शामिल हैं), MCP सर्वर से टूल्स लेता है और LLM को अनुरोध भेजता है, और प्रतिक्रिया वापस करता है।
LLM से प्रतिक्रिया में `choices` का एक एरे होगा। हमें परिणाम को प्रोसेस करके देखना होगा कि क्या किसी भी `tool_calls` मौजूद हैं। इससे हमें पता चलता है कि LLM किसी विशेष टूल को दलीलों के साथ कॉल करने का अनुरोध कर रहा है। LLM प्रतिक्रिया को हैंडल करने के लिए अपने `main.rs` फ़ाइल के नीचे निम्न कोड जोड़ें:

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

    // सामग्री उपलब्ध होने पर प्रिंट करें
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // टूल कॉल संभालें
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // सहायक संदेश जोड़ें

        // प्रत्येक टूल कॉल निष्पादित करें
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // संदेशों में टूल परिणाम जोड़ें
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // टूल परिणामों के साथ बातचीत जारी रखें
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
  
यदि `tool_calls` मौजूद हैं, तो यह टूल जानकारी निकालता है, टूल अनुरोध के साथ MCP सर्वर को कॉल करता है, और परिणामों को वार्तालाप संदेशों में जोड़ता है। फिर यह LLM के साथ वार्तालाप जारी रखता है और संदेश सहायक की प्रतिक्रिया और टूल कॉल परिणामों के साथ अपडेट हो जाते हैं।

MCP कॉल के लिए LLM द्वारा लौटाई गई टूल कॉल जानकारी निकालने के लिए, हम एक और हेल्पर फ़ंक्शन जोड़ेंगे जो कॉल करने के लिए आवश्यक सभी चीज़ें निकाले। अपने `main.rs` फ़ाइल के नीचे निम्न कोड जोड़ें:

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
  
सभी हिस्सों को जोड़ने के बाद, अब हम प्रारंभिक उपयोगकर्ता प्रॉम्प्ट को हैंडल कर सकते हैं और LLM को कॉल कर सकते हैं। अपने `main` फ़ंक्शन को निम्न कोड के साथ अपडेट करें:

```rust
// टूल कॉल के साथ LLM बातचीत
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
  
यह प्रारंभिक उपयोगकर्ता प्रॉम्प्ट के साथ LLM से प्रश्न करेगा जो दो संख्याओं का योग पूछ रहा है, और प्रतिक्रिया को प्रोसेस करेगा ताकि टूल कॉल को डायनामिकली हैंडल किया जा सके।

शाबाश, आपने कर लिया!

## असाइनमेंट

अभ्यास से कोड लें और सर्वर को कुछ और टूल्स के साथ बनाएं। फिर LLM के साथ एक क्लाइंट बनाएं, जैसे अभ्यास में था, और इसे विभिन्न प्रॉम्प्ट्स के साथ टेस्ट करें ताकि यह सुनिश्चित किया जा सके कि आपके सभी सर्वर टूल्स डायनामिक रूप से कॉल हो रहे हैं। क्लाइंट बनाने का यह तरीका उपयोगकर्ता को बेहतर अनुभव देता है क्योंकि वे सटीक क्लाइंट कमांड के बजाय सरल प्रॉम्प्ट्स का उपयोग कर सकते हैं और बैकग्राउंड में MCP सर्वर कॉल होने से अनजान रहेंगे।

## समाधान

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## मुख्य बिंदु

- अपने क्लाइंट में LLM जोड़ने से उपयोगकर्ताओं को MCP सर्वरों के साथ इंटरैक्ट करने का बेहतर तरीका मिलता है।
- आपको MCP सर्वर प्रतिक्रिया को उस रूप में बदलना होगा जिसे LLM समझ सके।

## नमूने

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## अतिरिक्त संसाधन

## आगे क्या है

- अगला: [Visual Studio Code का उपयोग करके सर्वर से संपर्क करना](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। हम सटीकता के लिए प्रयासरत हैं, लेकिन कृपया ध्यान दें कि स्वचालित अनुवाद में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ को उसकी मातृभाषा में प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सलाह दी जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या व्याख्या के लिए हम जिम्मेदार नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
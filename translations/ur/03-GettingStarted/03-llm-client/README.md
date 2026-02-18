# LLM کے ساتھ کلائنٹ بنانا

اب تک، آپ نے دیکھا کہ سرور اور کلائنٹ کیسے بنائے جاتے ہیں۔ کلائنٹ واضح طور پر سرور کو کال کرکے اس کے ٹولز، وسائل اور پرامپٹس کی فہرست حاصل کر سکتا ہے۔ تاہم، یہ طریقہ کار زیادہ عملی نہیں ہے۔ آپ کا صارف agentic دور میں رہتا ہے اور توقع کرتا ہے کہ وہ پرامپٹس استعمال کرے گا اور ایک LLM کے ساتھ بات چیت کرے گا۔ آپ کے صارف کو اس بات کی کوئی پرواہ نہیں ہوتی کہ آپ MCP استعمال کرتے ہیں یا نہیں اپنی صلاحیتیں ذخیرہ کرنے کے لیے، لیکن وہ قدرتی زبان میں بات چیت کرنے کی توقع کرتا ہے۔ تو ہم اسے کیسے حل کریں؟ حل یہ ہے کہ کلائنٹ میں ایک LLM شامل کیا جائے۔

## جائزہ

اس سبق میں ہم اپنے کلائنٹ میں ایک LLM شامل کرنے پر توجہ دیں گے اور دکھائیں گے کہ یہ آپ کے صارف کے لیے بہتر تجربہ کیسے فراہم کرتا ہے۔

## سیکھنے کے مقاصد

اس سبق کے اختتام تک، آپ یہ کر سکیں گے:

- ایک LLM کے ساتھ کلائنٹ بنانا۔
- MCP سرور کے ساتھ بغیر کسی رکاوٹ کے LLM کے ذریعے بات چیت کرنا۔
- کلائنٹ کی طرف سے صارف کے لیے بہتر تجربہ فراہم کرنا۔

## طریقہ کار

آئیے سمجھتے ہیں کہ ہمیں کون سا طریقہ اپنانا ہے۔ LLM شامل کرنا آسان لگتا ہے، مگر کیا ہم واقعی ایسا کریں گے؟

کلائنٹ سرور کے ساتھ اس طرح تعامل کرے گا:

1. سرور سے کنکشن قائم کریں۔

1. صلاحیتوں، پرامپٹس، وسائل اور ٹولز کی فہرست بنائیں اور ان کا اسکیمہ محفوظ کریں۔

1. ایک LLM شامل کریں اور محفوظ شدہ صلاحیتوں اور ان کے اسکیمے کو ایسی شکل میں فراہم کریں جو LLM سمجھ سکے۔

1. صارف کے پرامپٹ کو منتقل کریں، LLM کو کلائنٹ کے ذریعے دستیاب ٹولز کے ساتھ پاس کریں۔

زبردست، اب ہم اعلیٰ سطح پر سمجھ گئے کہ یہ کیسے کرنا ہے، آئیں نیچے دیے گئے مشق میں اسے آزما کر دیکھتے ہیں۔

## مشق: ایک LLM کے ساتھ کلائنٹ بنانا

اس مشق میں، ہم اپنے کلائنٹ میں ایک LLM شامل کرنا سیکھیں گے۔

### GitHub ذاتی رسائی ٹوکن کے ذریعے توثیق

GitHub ٹوکن بنانا ایک سیدھا سا عمل ہے۔ آپ یہ کر سکتے ہیں:

- GitHub سیٹنگز پر جائیں – اوپر دائیں کونے میں اپنی پروفائل تصویر پر کلک کریں اور Settings منتخب کریں۔
- Developer Settings پر جائیں – نیچے اسکرول کریں اور Developer Settings پر کلک کریں۔
- Personal Access Tokens منتخب کریں – Fine-grained tokens پر کلک کریں اور پھر Generate new token پر کلک کریں۔
- اپنے ٹوکن کو ترتیب دیں – ریفرنس کے لیے ایک نوٹ شامل کریں، میعاد ختم ہونے کی تاریخ مقرر کریں، اور ضروری سکوپس (اجازتیں) منتخب کریں۔ اس کیس میں Models کی اجازت شامل کریں۔
- ٹوکن بنائیں اور نقل کریں – Generate token پر کلک کریں، اور فوراً اسے کاپی کر لیں کیونکہ آپ دوبارہ اسے نہیں دیکھ پائیں گے۔

### -1- سرور سے کنکٹ ہونا

آئیے پہلے اپنا کلائنٹ بناتے ہیں:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // اسکیمہ کی توثیق کے لیے zod درآمد کریں

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

پچھلے کوڈ میں ہم نے:

- ضروری لائبریریز امپورٹ کیں
- ایک کلاس بنائی جس کے دو ممبران ہیں، `client` اور `openai` جو ہمارے کلائنٹ کو منظم کرنے اور LLM کے ساتھ بات چیت کرنے میں مدد کریں گے۔
- اپنے LLM انسٹنس کو GitHub Models استعمال کرنے کے لیے `baseUrl` سیٹ کرکے انفرنس API کی طرف اشارہ کیا۔

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# اسٹڈی او کنکشن کے لئے سرور کے پیرامیٹرز بنائیں
server_params = StdioServerParameters(
    command="mcp",  # قابلِ عمل
    args=["run", "server.py"],  # اختیاری کمانڈ لائن دلائل
    env=None,  # اختیاری ماحول کے متغیرات
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # کنکشن کو ابتدائی شکل دیں
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

پچھلے کوڈ میں ہم نے:

- MCP کے لیے ضروری لائبریریز امپورٹ کیں
- ایک کلائنٹ بنایا

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

سب سے پہلے، آپ کو اپنے `pom.xml` فائل میں LangChain4j کی dependencies شامل کرنی ہوں گی۔ MCP انٹیگریشن اور GitHub Models سپورٹ فعال کرنے کے لیے یہ dependencies شامل کریں:

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

پھر اپنا جاوا کلائنٹ کلاس بنائیں:

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
    
    public static void main(String[] args) throws Exception {        // LLM کو GitHub ماڈلز استعمال کرنے کے لیے ترتیب دیں
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // سرور سے رابطہ کے لیے MCP ٹرانسپورٹ بنائیں
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP کلائنٹ بنائیں
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

پچھلے کوڈ میں ہم نے:

- **LangChain4j dependencies شامل کیں**: MCP انٹیگریشن، OpenAI آفیشل کلائنٹ، اور GitHub Models سپورٹ کے لیے ضروری
- **LangChain4j لائبریریز امپورٹ کیں**: MCP انٹیگریشن اور OpenAI چیٹ ماڈل کی فعالیت کے لیے
- **`ChatLanguageModel` بنایا**: اپنے GitHub ٹوکن کے ساتھ GitHub Models استعمال کرنے کے لیے ترتیب دیا
- **HTTP ٹرانسپورٹ قائم کیا**: MCP سرور سے کنیکٹ کرنے کے لیے Server-Sent Events (SSE) کا استعمال
- **MCP کلائنٹ بنایا**: جو سرور کے ساتھ بات چیت کرے گا
- **LangChain4j کے بلٹ ان MCP سپورٹ کا استعمال کیا**: جو LLMs اور MCP سرورز کے درمیان انٹیگریشن کو آسان بناتا ہے

#### Rust

یہ مثال فرض کرتی ہے کہ آپ کے پاس Rust پر مبنی MCP سرور چل رہا ہے۔ اگر نہیں ہے تو سرور بنانے کے لیے [01-first-server](../01-first-server/README.md) سبق دیکھیں۔

اپنے Rust MCP سرور کی ڈائریکٹری میں ترمینل کھولیں۔ پھر نیچے دیا گیا کمانڈ چلائیں تاکہ نیا LLM کلائنٹ پروجیکٹ بنایا جا سکے:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

اپنی `Cargo.toml` فائل میں درج ذیل dependencies شامل کریں:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI کے لیے کوئی سرکاری Rust لائبریری نہیں ہے، لیکن `async-openai` crate ایک [کمیونٹی کی برقرار رکھی ہوئی لائبریری](https://platform.openai.com/docs/libraries/rust#rust) ہے جو عام طور پر استعمال ہوتی ہے۔

`src/main.rs` فائل کھولیں اور اس کا موجودہ مواد درج ذیل کوڈ سے بدل دیں:

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
    // ابتدائی پیغام
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI کلائنٹ سیٹ کریں
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP کلائنٹ سیٹ کریں
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

    // کریں گے: MCP ٹول کی فہرست حاصل کریں

    // کریں گے: ٹول کالز کے ساتھ LLM گفتگو

    Ok(())
}
```

یہ کوڈ ایک بنیادی Rust ایپلیکیشن مرتب کرتا ہے جو MCP سرور اور LLM انٹریکشن کے لیے GitHub Models کے ساتھ کنکٹ ہوتی ہے۔

> [!IMPORTANT]
> اپلیکیشن چلانے سے پہلے `OPENAI_API_KEY` ماحولیاتی متغیر کو اپنے GitHub ٹوکن کے ساتھ سیٹ کرنا نہ بھولیں۔

زبردست، اگلے مرحلے کے لیے، چلیں سرور کی صلاحیتوں کی فہرست بناتے ہیں۔

### -2- سرور کی صلاحیتوں کی فہرست بنائیں

اب ہم سرور سے کنیکٹ ہوں گے اور اس کی صلاحیتوں کے بارے میں پوچھیں گے:

#### Typescript

اسی کلاس میں درج ذیل طریقے شامل کریں:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // اوزاروں کی فہرست بنانا
    const toolsResult = await this.client.listTools();
}
```

پچھلے کوڈ میں ہم نے:

- سرور سے کنیکٹ کرنے کا کوڈ `connectToServer` شامل کیا۔
- `run` میتھڈ بنایا جو ہمارے ایپ فلو کو ہینڈل کرے گا۔ ابھی یہ صرف ٹولز کی فہرست دیتا ہے لیکن ہم جلد ہی اس میں اضافہ کریں گے۔

#### Python

```python
# دستیاب وسائل کی فہرست بنائیں
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# دستیاب آلات کی فہرست بنائیں
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

یہ ہم نے شامل کیا:

- وسائل اور ٹولز کی فہرست بنائی اور پرنٹ کی۔ ٹولز کے لیے `inputSchema` بھی شامل کیا جو بعد میں استعمال ہوگا۔

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

پچھلے کوڈ میں ہم نے:

- MCP سرور پر دستیاب ٹولز کی فہرست بنائی
- ہر ٹول کے نام، وضاحت اور اس کا اسکیمہ حاصل کیا۔ یہ اسکیمہ ہم بعد میں ٹولز کو کال کرنے کے لیے استعمال کریں گے۔

#### Java

```java
// ایک ٹول فراہم کنندہ بنائیں جو خودکار طریقے سے MCP ٹولز کی دریافت کرے
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP ٹول فراہم کنندہ خودکار طریقے سے سنبھالتا ہے:
// - MCP سرور سے دستیاب ٹولز کی فہرست بنانا
// - MCP ٹول اسکیموں کو LangChain4j فارمیٹ میں تبدیل کرنا
// - ٹول کے نفاذ اور جوابات کا انتظام کرنا
```

پچھلے کوڈ میں ہم نے:

- ایک `McpToolProvider` بنایا جو خود بخود تمام ٹولز کو MCP سرور سے دریافت اور رجسٹر کرتا ہے
- ٹول فراہم کنندہ MCP ٹول اسکیمے کو LangChain4j کے ٹول فارمیٹ میں اندرونی طور پر تبدیل کرتا ہے
- یہ طریقہ کار دستی ٹول کی فہرست سازی اور تبدیلی کے عمل کو چھپا دیتا ہے

#### Rust

MCP سرور سے ٹولز حاصل کرنے کے لیے `list_tools` میتھڈ استعمال ہوتا ہے۔ `main` فنکشن میں MCP کلائنٹ سیٹ اپ کے بعد یہ کوڈ شامل کریں:

```rust
// ایم سی پی ٹول کی فہرست حاصل کریں
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- سرور کی صلاحیتوں کو LLM ٹولز میں تبدیل کریں

سرور کی صلاحیتوں کی فہرست بنانے کے بعد اگلا قدم انہیں ایسی شکل میں تبدیل کرنا ہے جو LLM سمجھ سکے۔ ایک بار ایسا کرنے کے بعد، ہم ان صلاحیتوں کو اپنے LLM کو ٹولز کے طور پر فراہم کر سکتے ہیں۔

#### TypeScript

1. MCP سرور کی جوابی فائل کو LLM کے قابل فہم ٹول فارمیٹ میں تبدیل کرنے کے لیے درج ذیل کوڈ شامل کریں:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ان پٹ_سکیما کی بنیاد پر زوڈ اسکیمہ بنائیں
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // قسم کو واضح طور پر "فنکشن" مقرر کریں
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

    مذکورہ کوڈ MCP سرور سے ملنے والے ردعمل کو ایسے ٹول تعریفی فارمیٹ میں تبدیل کرتا ہے جو LLM سمجھ سکتا ہے۔

1. اب `run` میتھڈ کو اپ ڈیٹ کرتے ہیں تاکہ وہ سرور کی صلاحیتوں کی فہرست دے:

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

    پچھلے کوڈ میں ہم نے `run` میتھڈ کو اپ ڈیٹ کیا تاکہ وہ نتائج کے ذریعے مِپ کرے اور ہر انٹری کے لیے `openAiToolAdapter` کال کرے۔

#### Python

1. سب سے پہلے، درج ذیل کنورٹر فنکشن بنائیں:

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

    `convert_to_llm_tools` فنکشن میں ہم MCP ٹول کا ردعمل LLM کے قابل فہم شکل میں تبدیل کرتے ہیں۔

1. پھر اپنے کلائنٹ کوڈ کو اس فنکشن سے استفادہ کرنے کے لیے اپ ڈیٹ کریں:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    یہاں ہم `convert_to_llm_tool` کو کال کر رہے ہیں تاکہ MCP ٹول کا ردعمل LLM کو دے سکیں۔

#### .NET

1. MCP ٹول کے ردعمل کو LLM کے قابل فہم شکل میں تبدیل کرنے کے لیے کوڈ شامل کریں:

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

پچھلے کوڈ میں ہم نے:

- `ConvertFrom` فنکشن بنایا جو نام، وضاحت اور ان پٹ اسکیمہ لیتا ہے۔
- ایک فنکشنلٹی پر مشتمل ہے جو `FunctionDefinition` بناتی ہے جسے `ChatCompletionsDefinition` میں پاس کیا جاتا ہے۔ یہ چیز LLM سمجھ سکتا ہے۔

1. اب دیکھیں کہ ہم موجودہ کوڈ کو اس فنکشن کے فائدے کے لیے کیسے اپ ڈیٹ کر سکتے ہیں:

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
// قدرتی زبان کے تعامل کے لیے ایک بوٹ انٹرفیس بنائیں
public interface Bot {
    String chat(String prompt);
}

// LLM اور MCP ٹولز کے ساتھ AI سروس کو تشکیل دیں
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

پچھلے کوڈ میں ہم نے:

- ایک سادہ `Bot` انٹرفیس بنایا جس سے قدرتی زبان میں بات چیت ہو سکے
- LangChain4j کی `AiServices` کا استعمال کیا جو خود بخود LLM کو MCP ٹول فراہم کنندہ کے ساتھ منسلک کرتا ہے
- فریم ورک خود بخود ٹول اسکیمہ تبدیل کرنے اور فنکشن کال کرنے کا کام کرتا ہے
- اس طریقہ کار سے دستی ٹول تبدیلی کی ضرورت ختم ہو جاتی ہے - LangChain4j تمام پیچیدگی کو ہینڈل کرتا ہے

#### Rust

MCP ٹول کے ردعمل کو LLM کی سمجھ میں آنے والے فارمیٹ میں تبدیل کرنے کے لیے ایک ہیلپر فنکشن شامل کریں جو ٹولز کی لسٹنگ کو فارمیٹ کرے۔ نیچے دی گئی کوڈ `main.rs` میں `main` فنکشن کے نیچے شامل کریں۔ یہ LLM سے درخواست کرتے وقت کال کیا جائے گا:

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

زبردست، اب ہم صارف کی درخواستیں ہینڈل کرنے کے لیے تیار ہیں، آئیں اب اس پر توجہ دیں۔

### -4- صارف کے پرامپٹ کی درخواست ہینڈل کرنا

اس حصے میں ہم صارف کی درخواستوں کو ہینڈل کریں گے۔

#### TypeScript

1. ایک ایسا میتھڈ شامل کریں جو ہمارے LLM کو کال کرے گا:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // ۲۔ سرور کے ٹول کو کال کریں
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // ۳۔ نتیجے کے ساتھ کچھ کریں
        // کرنا باقی ہے

        }
    }
    ```

    پچھلے کوڈ میں ہم نے:

    - `callTools` میتھڈ شامل کی۔
    - یہ میتھڈ LLM کے ردعمل کو لے کر چیک کرتا ہے کہ کون سے ٹولز کال ہوئے، اگر کوئی ہوں:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // آلے کو کال کریں
        }
        ```

    - اگر LLM کہے کہ کوئی ٹول کال ہونا چاہیے تو اسے کال کرتا ہے:

        ```typescript
        // 2۔ سرور کے آلے کو کال کریں
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3۔ نتیجے کے ساتھ کچھ کریں
        // کرنے کی ضرورت ہے
        ```

1. `run` میتھڈ کو اپ ڈیٹ کریں تاکہ LLM کو کال کرے اور `callTools` کو کال کرے:

    ```typescript

    // 1. میسجز بنائیں جو LLM کے لیے ان پٹ ہوں
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM کو کال کرنا
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM کے جواب کا جائزہ لیں، ہر انتخاب کے لیے چیک کریں کہ کیا اس میں ٹول کالز ہیں
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

زبردست، اب مکمل کوڈ درج کریں:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // اسکیمہ کی تصدیق کے لیے زوڈ امپورٹ کریں

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // مستقبل میں ممکن ہے کہ اس یو آر ایل کو تبدیل کرنے کی ضرورت ہو: https://models.github.ai/inference
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
          // ان پٹ_اسکیمہ کی بنیاد پر زوڈ اسکیمہ بنائیں
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // قسم کو واضح طور پر "فنکشن" پر سیٹ کریں
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
    
    
          // 2. سرور کے آلے کو کال کریں
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. نتیجہ کے ساتھ کچھ کریں
          // کرنے کے لیے
    
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
    
        // 1. ایل ایل ایم کے جواب کو دیکھیں، ہر انتخاب کے لیے چیک کریں کہ آیا اس میں ٹول کالز ہیں
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

1. LLM کو کال کرنے کے لیے ضروری امپورٹس شامل کریں:

    ```python
    # ایل ایل ایم
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. پھر وہ فنکشن شامل کریں جو LLM کو کال کرے گا:

    ```python
    # ایل ایل ایم

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
            # اختیاری پیرامیٹرز
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

    پچھلے کوڈ میں ہم نے:

    - وہ فنکشنز پاس کیے جو MCP سرور سے لی اور تبدیل کیے گئے۔
    - پھر LLM کو ان فنکشنز کے ساتھ کال کیا۔
    - پھر چیک کیا کہ کون سے فنکشنز کو کال کرنا ہے۔
    - آخر میں فنکشنز کی ایک فہرست کال کی۔

1. آخری قدم، اپنا مین کوڈ اپ ڈیٹ کریں:

    ```python
    prompt = "Add 2 to 20"

    # اگر کوئی ہو تو ٹولز کے بارے میں LLM سے پوچھیں
    functions_to_call = call_llm(prompt, functions)

    # تجویز کردہ افعال کو کال کریں
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    یہ آخری قدم تھا، اوپر والے کوڈ میں ہم:

    - MCP ٹول کو `call_tool` کے ذریعے کال کر رہے ہیں جو LLM نے ہمارے پرامپٹ کی بنیاد پر منتخب کیا۔
    - MCP سرور کے ٹول کال کا نتیجہ پرنٹ کر رہے ہیں۔

#### .NET

1. LLM پرامپٹ درخواست کرنے کے لیے کوڈ دکھاتے ہیں:

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

    پچھلے کوڈ میں ہم نے:

    - MCP سرور سے ٹولز حاصل کیے، `var tools = await GetMcpTools()`.
    - ایک صارف کا پرامپٹ بنایا `userMessage`.
    - ایک options آبجیکٹ بنایا جس میں ماڈل اور ٹولز شامل ہیں۔
    - LLM کو درخواست بھیجی۔

1. آخری قدم، دیکھیں کیا LLM سمجھتا ہے کہ فنکشن کال کرنی چاہیے:

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

    پچھلے کوڈ میں ہم نے:

    - فنکشن کالز کی فہرست پر لوپ لگایا۔
    - ہر ٹول کال کے لیے نام اور آرجیومنٹس نکالے اور MCP کلائنٹ کے ذریعے سرور پر ٹول کو کال کیا۔ آخر میں نتائج پرنٹ کیے۔

مکمل کوڈ درج ذیل ہے:

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
    // قدرتی زبان کی درخواستوں کو چلائیں جو خودکار طریقے سے MCP ٹولز استعمال کرتی ہیں
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

پچھلے کوڈ میں ہم نے:

- آسان قدرتی زبان کے پرامپٹس استعمال کر کے MCP سرور کے ٹولز کے ساتھ بات چیت کی
- LangChain4j فریم ورک خود بخود مندرجہ ذیل کام کرتا ہے:
  - صارف پرامپٹس کو جب ضرورت ہو ٹول کالز میں تبدیل کرتا ہے
  - LLM کے فیصلے کی بنیاد پر مناسب MCP ٹولز کال کرتا ہے
  - LLM اور MCP سرور کے درمیان گفتگو کے بہاؤ کو منظم کرتا ہے
- `bot.chat()` طریقہ قدرتی زبان میں جوابات لوٹاتا ہے جن میں MCP ٹولز کے نتائج شامل ہو سکتے ہیں
- اس طریقے سے صارف کو ایک ہموار تجربہ ملتا ہے جہاں انہیں MCP کی اندرونی پیچیدگیوں کا علم نہیں ہونا پڑتا

مکمل کوڈ مثال:

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

یہاں زیادہ تر کام ہوتا ہے۔ ہم صارف کے ابتدائی پرامپٹ کے ساتھ LLM کو کال کریں گے، پھر جواب کو پراسیس کریں گے تاکہ دیکھیں کیا کوئی ٹول کال کرنے کی ضرورت ہے۔ اگر ہے، تو ہم وہ ٹول کال کریں گے اور پھر LLM کے ساتھ گفتگو جاری رکھیں گے جب تک کہ مزید ٹول کالز کی ضرورت نہ ہو اور ہمیں آخری جواب مل جائے۔

ہم LLM کو متعدد بار کال کریں گے، اس لیے ایک فنکشن ڈیفائن کرتے ہیں جو LLM کال کو ہینڈل کرے گا۔ درج ذیل فنکشن اپنی `main.rs` فائل میں شامل کریں:

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

یہ فنکشن LLM کلائنٹ، پیغامات (جس میں صارف کا پرامپٹ شامل ہے)، MCP سرور کے ٹولز لیتا ہے اور LLM کو درخواست بھیج کر جواب لوٹاتا ہے۔
LLM کا جواب `choices` کی ایک صف پر مشتمل ہوگا۔ ہمیں نتیجہ اس طرح پروسیس کرنا ہوگا کہ کیا کوئی `tool_calls` موجود ہیں یا نہیں۔ اس سے ہمیں پتہ چلتا ہے کہ LLM کسی مخصوص ٹول کو دلائل کے ساتھ کال کرنے کی درخواست کر رہا ہے۔ LLM کے جواب کو ہینڈل کرنے کے لیے اپنا `main.rs` فائل کے نیچے مندرجہ ذیل کوڈ شامل کریں:

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

    // اگر دستیاب ہو تو مواد پرنٹ کریں
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // ٹول کالز کو سنبھالیں
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // معاون پیغام شامل کریں

        // ہر ٹول کال کو انجام دیں
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // میسجز میں ٹول کے نتائج شامل کریں
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ٹول کے نتائج کے ساتھ گفتگو جاری رکھیں
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

اگر `tool_calls` موجود ہوں، تو یہ ٹول کی معلومات نکالتا ہے، MCP سرور کو ٹول کی درخواست کے ساتھ کال کرتا ہے، اور نتائج کو مکالمے کے پیغامات میں شامل کرتا ہے۔ پھر یہ LLM کے ساتھ مکالمہ جاری رکھتا ہے اور پیغامات معاون کی جواب اور ٹول کال کے نتائج کے ساتھ اپڈیٹ ہوتے ہیں۔

MCP کالز کے لیے LLM کی طرف سے واپس کیے گئے ٹول کال کی معلومات نکالنے کے لیے، ہم ایک اور ہیلپر فنکشن شامل کریں گے جو کال کرنے کے لیے درکار تمام معلومات نکالے گا۔ اپنے `main.rs` فائل کے نیچے مندرجہ ذیل کوڈ شامل کریں:

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

اب جب تمام اجزاء موجود ہیں، ہم ابتدائی صارف کے پرامٹ کو ہینڈل کر کے LLM کو کال کر سکتے ہیں۔ اپنے `main` فنکشن کو مندرجہ ذیل کوڈ کے ساتھ اپڈیٹ کریں:

```rust
// آلات کی کالز کے ساتھ ایل ایل ایم گفتگو
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

یہ ابتدائی صارف کے پرامٹ کے ساتھ LLM کو سوال کرے گا کہ دو اعداد کا مجموعہ نکالے اور جواب کو پروسیس کرے گا تاکہ ڈائنامک طریقے سے ٹول کالز کی ہینڈلنگ ہو سکے۔

زبردست، آپ نے کر دکھایا!

## اسائنمنٹ

مشکل سوال سے کوڈ لے کر سرور میں مزید ٹولز شامل کریں۔ پھر مشابہت کی طرح ایک کلائنٹ بنائیں جس میں LLM ہو، اور مختلف پرامٹس کے ساتھ اسے ٹیسٹ کریں تاکہ یقینی بنایا جا سکے کہ آپ کے سرور کے تمام ٹولز ڈائنامک انداز میں کال ہوتے ہیں۔ اس طریقے سے کلائنٹ بنانے کا مطلب ہے کہ آخری صارف بہترین تجربہ حاصل کرے گا کیونکہ وہ مخصوص کلائنٹ کمانڈز کی بجائے صرف پرامٹس استعمال کرے گا اور MCP سرور کی کال کا احساس نہیں ہوگا۔

## حل

[حل](/03-GettingStarted/03-llm-client/solution/README.md)

## کلیدی نکات

- اپنے کلائنٹ میں LLM شامل کرنے سے صارفین کو MCP سرورز کے ساتھ بہتر رابطہ حاصل ہوتا ہے۔
- آپ کو MCP سرور کے جواب کو ایسی شکل میں تبدیل کرنا ہوگا جسے LLM سمجھ سکے۔

## نمونے

- [جاوا کیلکولیٹر](../samples/java/calculator/README.md)
- [.Net کیلکولیٹر](../../../../03-GettingStarted/samples/csharp)
- [جاوا اسکرپٹ کیلکولیٹر](../samples/javascript/README.md)
- [ٹائپ اسکرپٹ کیلکولیٹر](../samples/typescript/README.md)
- [پائتھن کیلکولیٹر](../../../../03-GettingStarted/samples/python)
- [رسٹ کیلکولیٹر](../../../../03-GettingStarted/samples/rust)

## اضافی وسائل

## آگے کیا ہے

- اگلا: [Visual Studio Code استعمال کرتے ہوئے سرور کو استعمال کرنا](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جب کہ ہم درستگی کے لیے بھرپور کوشش کرتے ہیں، براہ کرم یہ بات ذہن میں رکھیں کہ خودکار ترجموں میں غلطیاں یا نقائص ہو سکتے ہیں۔ اصل دستاویز اپنی مادری زبان میں ہی معتبر ماخذ سمجھی جانی چاہیے۔ اہم معلومات کے لیے پیشہ ورانہ انسانی ترجمہ تجویز کیا جاتا ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم پر عائد نہیں ہوتی۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
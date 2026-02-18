# ایجاد یک کلاینت با LLM

تا کنون، نحوه ایجاد سرور و کلاینت را مشاهده کرده‌اید. کلاینت توانسته است به‌صورت صریح سرور را برای فهرست کردن ابزارها، منابع و پرامپت‌ها فراخوانی کند. با این حال، این روش خیلی کاربردی نیست. کاربر شما در عصر عامل‌محور زندگی می‌کند و انتظار دارد از پرامپت‌ها استفاده کند و با LLM ارتباط برقرار کند. برای کاربر شما مهم نیست که قابلیت‌ها را با MCP ذخیره می‌کنید یا نه ولی آنها انتظار دارند با زبان طبیعی تعامل کنند. پس چگونه این مشکل را حل کنیم؟ راه حل افزودن یک LLM به کلاینت است.

## نمای کلی

در این درس تمرکز بر افزودن یک LLM به کلاینت شماست و نشان می‌دهیم چگونه این کار تجربه بسیار بهتری برای کاربر شما فراهم می‌کند.

## اهداف یادگیری

تا پایان این درس، قادر خواهید بود:

- ایجاد یک کلاینت با LLM.
- تعامل بدون درز با سرور MCP از طریق LLM.
- ارائه تجربه بهتر برای کاربر نهایی در سمت کلاینت.

## رویکرد

بیایید سعی کنیم رویکرد مورد نیاز را درک کنیم. افزودن یک LLM ساده به نظر می‌رسد، اما آیا واقعاً این کار را انجام خواهیم داد؟

نحوه تعامل کلاینت با سرور به شرح زیر است:

1. برقراری اتصال با سرور.

1. فهرست کردن قابلیت‌ها، پرامپت‌ها، منابع و ابزارها و ذخیره کردن طرح‌واره (schema) آن‌ها.

1. افزودن یک LLM و ارسال قابلیت‌های ذخیره شده همراه با طرح‌واره آنها به فرمتی که LLM می‌فهمد.

1. رسیدگی به پرامپت کاربر با ارسال آن به LLM همراه با ابزارهای فهرست شده توسط کلاینت.

خوب، حالا که در سطح کلان فهمیدیم چگونه می‌توانیم این کار را انجام دهیم، بیایید این را در تمرین زیر امتحان کنیم.

## تمرین: ایجاد یک کلاینت با LLM

در این تمرین، یاد می‌گیریم چگونه یک LLM به کلاینت خود اضافه کنیم.

### احراز هویت با استفاده از توکن دسترسی شخصی گیت‌هاب

ایجاد یک توکن گیت‌هاب فرآیندی ساده است. در اینجا چگونگی انجام آن آمده است:

- به تنظیمات گیت‌هاب بروید – روی تصویر پروفایل خود در گوشه بالا سمت راست کلیک کرده و Settings را انتخاب کنید.
- به تنظیمات توسعه‌دهنده بروید – صفحه را پایین بکشید و Developer Settings را کلیک کنید.
- توکن‌های دسترسی شخصی را انتخاب کنید – روی Fine-grained tokens کلیک کنید و سپس Generate new token را بزنید.
- توکن خود را پیکربندی کنید – یک یادداشت برای مرجع اضافه کنید، تاریخ انقضا تعیین کنید و مجوزهای لازم (Scopes) را انتخاب کنید. در این مورد مطمئن شوید که مجوز Models اضافه شده است.
- تولید و کپی توکن – روی Generate token کلیک کنید و مطمئن شوید که فوراً آن را کپی کنید، زیرا دیگر قابل مشاهده نخواهد بود.

### -1- اتصال به سرور

ابتدا کلاینت خود را ایجاد کنیم:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // وارد کردن zod برای اعتبارسنجی اسکیم

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

در کد بالا ما:

- کتابخانه‌های مورد نیاز را وارد کردیم.
- یک کلاس با دو عضو `client` و `openai` ایجاد کردیم که به ترتیب به ما کمک می‌کنند کلاینت را مدیریت کنیم و با LLM تعامل داشته باشیم.
- نمونه LLM خود را طوری پیکربندی کردیم که از GitHub Models استفاده کند و `baseUrl` را برای اشاره به API استنتاج تنظیم کردیم.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# ایجاد پارامترهای سرور برای اتصال stdio
server_params = StdioServerParameters(
    command="mcp",  # اجرایی
    args=["run", "server.py"],  # آرگومان‌های اختیاری خط فرمان
    env=None,  # متغیرهای محیطی اختیاری
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # مقداردهی اولیه اتصال
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

در کد بالا ما:

- کتابخانه‌های مورد نیاز برای MCP را وارد کردیم.
- یک کلاینت ایجاد کردیم.

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

ابتدا باید وابستگی‌های LangChain4j را به فایل `pom.xml` خود اضافه کنید. این وابستگی‌ها را اضافه کنید تا از ادغام MCP و پشتیبانی از GitHub Models برخوردار شوید:

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

سپس کلاس کلاینت جاوای خود را ایجاد کنید:

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
    
    public static void main(String[] args) throws Exception {        // پیکربندی مدل‌های GitHub برای استفاده توسط LLM
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // ایجاد انتقال MCP برای اتصال به سرور
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // ایجاد کلاینت MCP
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

در کد بالا ما:

- **وابستگی‌های LangChain4j را اضافه کرده‌ایم**: مورد نیاز برای ادغام MCP، کلاینت رسمی OpenAI و پشتیبانی GitHub Models
- **کتابخانه‌های LangChain4j را وارد کردیم**: برای ادغام MCP و عملکرد مدل چت OpenAI
- **یک `ChatLanguageModel` ساختیم**: پیکربندی شده برای استفاده از GitHub Models با توکن GitHub شما
- **انتقال HTTP را تنظیم کردیم**: با استفاده از Server-Sent Events (SSE) برای اتصال به سرور MCP
- **یک کلاینت MCP ایجاد کردیم**: که ارتباط با سرور را مدیریت می‌کند
- **از پشتیبانی داخلی MCP در LangChain4j استفاده کردیم**: که ادغام بین LLM‌ها و سرورهای MCP را ساده می‌کند

#### Rust

این مثال فرض می‌کند که شما یک سرور MCP مبتنی بر Rust دارید. اگر ندارید، به درس [01-first-server](../01-first-server/README.md) برای ایجاد سرور مراجعه کنید.

پس از داشتن سرور MCP Rust، یک ترمینال باز کنید و به همان دایرکتوری سرور بروید. سپس دستور زیر را اجرا کنید تا پروژه کلاینت LLM جدیدی بسازید:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

وابستگی‌های زیر را به فایل `Cargo.toml` خود اضافه کنید:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> کتابخانه رسمی Rust برای OpenAI وجود ندارد، اما crate `async-openai` یک کتابخانه [نگهداری شده توسط جامعه](https://platform.openai.com/docs/libraries/rust#rust) است که معمولاً استفاده می‌شود.

فایل `src/main.rs` را باز کنید و محتوای آن را با کد زیر جایگزین کنید:

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
    // پیام اولیه
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // راه‌اندازی کلاینت OpenAI
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // راه‌اندازی کلاینت MCP
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

    // کار در دست انجام: دریافت فهرست ابزار MCP

    // کار در دست انجام: مکالمه LLM با فراخوانی ابزار

    Ok(())
}
```

این کد یک برنامه پایه Rust تنظیم می‌کند که به سرور MCP و GitHub Models برای تعاملات LLM متصل می‌شود.

> [!IMPORTANT]
> قبل از اجرای برنامه مطمئن شوید متغیر محیطی `OPENAI_API_KEY` را با توکن GitHub خود تنظیم کرده‌اید.

خوب، در گام بعدی قابلیت‌های سرور را فهرست می‌کنیم.

### -2- فهرست کردن قابلیت‌های سرور

اکنون به سرور متصل می‌شویم و قابلیت‌های آن را دریافت می‌کنیم:

#### Typescript

در همان کلاس، روش‌های زیر را اضافه کنید:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // فهرست ابزارها
    const toolsResult = await this.client.listTools();
}
```

در کد بالا ما:

- کدی برای اتصال به سرور، `connectToServer` اضافه کردیم.
- متدی به نام `run` ایجاد کردیم که مسئول جریان برنامه است. تا کنون فقط ابزارها را فهرست می‌کند ولی به زودی بخش‌های بیشتری به آن اضافه خواهیم کرد.

#### Python

```python
# لیست منابع موجود
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# فهرست ابزارهای موجود
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

آنچه اضافه کردیم:

- منابع و ابزارها را فهرست کردیم و آنها را چاپ کردیم. برای ابزارها همچنین `inputSchema` را فهرست کردیم که بعداً استفاده می‌شود.

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

در کد بالا ما:

- ابزارهای موجود در سرور MCP را فهرست کردیم.
- برای هر ابزار نام، توضیح و طرح‌واره‌اش را فهرست کردیم. مورد آخر چیزی است که به زودی برای فراخوانی ابزارها استفاده خواهیم کرد.

#### Java

```java
// ایجاد یک ارائه‌دهنده ابزار که به‌طور خودکار ابزارهای MCP را کشف می‌کند
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// ارائه‌دهنده ابزار MCP به‌طور خودکار موارد زیر را مدیریت می‌کند:
// - فهرست کردن ابزارهای موجود از سرور MCP
// - تبدیل طرح‌های ابزار MCP به فرمت LangChain4j
// - مدیریت اجرای ابزار و پاسخ‌ها
```

در کد بالا ما:

- یک `McpToolProvider` ساختیم که تمام ابزارها را از سرور MCP به‌صورت خودکار کشف و ثبت می‌کند.
- ارائه‌دهنده ابزار تبدیل بین طرح‌واره ابزار MCP و فرمت ابزار LangChain4j را به‌صورت داخلی مدیریت می‌کند.
- این رویکرد فرایند فهرست کردن و تبدیل دستی ابزارها را حذف می‌کند.

#### Rust

دریافت ابزارها از سرور MCP با متد `list_tools` انجام می‌شود. در تابع `main` خود، پس از تنظیم کلاینت MCP، کد زیر را اضافه کنید:

```rust
// دریافت فهرست ابزار MCP
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- تبدیل قابلیت‌های سرور به ابزارهای LLM

گام بعدی پس از فهرست کردن قابلیت‌های سرور، تبدیل آنها به فرمتی است که LLM بتواند بفهمد. وقتی این کار را انجام دادیم، می‌توانیم این قابلیت‌ها را به عنوان ابزار به LLM خود بدهیم.

#### TypeScript

1. کد زیر را برای تبدیل پاسخ از سرور MCP به فرمتی که LLM می‌تواند استفاده کند، اضافه کنید:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ایجاد یک طرح zod بر اساس input_schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // نوع را به صورت صریح به "function" تنظیم کنید
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

    کد بالا پاسخ از سرور MCP را می‌گیرد و آن را به فرمت تعریف ابزار تبدیل می‌کند که LLM می‌تواند بفهمد.

1. حالا متد `run` را برای فهرست کردن قابلیت‌های سرور به‌روزرسانی کنیم:

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

    در کد بالا `run` را به روز کردیم تا روی نتیجه پیمایش کند و برای هر ورودی `openAiToolAdapter` را فراخوانی کند.

#### Python

1. ابتدا تابع تبدیل زیر را ایجاد کنیم:

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

    در تابع `convert_to_llm_tools` پاسخ ابزار MCP گرفته می‌شود و به فرمتی تبدیل می‌شود که LLM می‌فهمد.

1. سپس کد کلاینت خود را برای استفاده از این تابع به شکل زیر به‌روزرسانی کنیم:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    در اینجا فراخوانی `convert_to_llm_tool` را اضافه کردیم تا پاسخ ابزار MCP را به چیزی تبدیل کنیم که بعداً به LLM بدهیم.

#### .NET

1. کد تبدیل پاسخ ابزار MCP به چیزی که LLM می‌فهمد را اضافه کنیم:

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

در کد بالا:

- تابع `ConvertFrom` را ایجاد کردیم که نام، توضیح و طرح‌واره ورودی را می‌گیرد.
- عملکردی تعریف کردیم که یک `FunctionDefinition` می‌سازد که به `ChatCompletionsDefinition` داده می‌شود. مورد دوم چیزی است که LLM می‌تواند بفهمد.

1. حالا ببینیم چگونه می‌توانیم کد موجود را برای بهره‌برداری از این تابع به‌روزرسانی کنیم:

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
// ایجاد یک رابط ربات برای تعامل با زبان طبیعی
public interface Bot {
    String chat(String prompt);
}

// پیکربندی سرویس هوش مصنوعی با ابزارهای LLM و MCP
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

در کد بالا ما:

- یک رابط ساده `Bot` برای تعاملات زبان طبیعی تعریف کردیم.
- از `AiServices` LangChain4j استفاده کردیم تا به‌طور خودکار LLM را با ارائه‌دهنده ابزارهای MCP پیوند دهد.
- چارچوب به‌صورت خودکار تبدیل طرح‌واره ابزار و فراخوانی عملکرد را پشت صحنه مدیریت می‌کند.
- این رویکرد تبدیل دستی ابزار را حذف می‌کند - LangChain4j تمام پیچیدگی‌های تبدیل ابزارهای MCP به فرمت سازگار با LLM را مدیریت می‌کند.

#### Rust

برای تبدیل پاسخ ابزار MCP به فرمتی که LLM بتواند بفهمد، یک تابع کمکی اضافه می‌کنیم که فهرست ابزارها را فرمت می‌کند. کد زیر را در فایل `main.rs` خود زیر تابع `main` اضافه کنید. این تابع هنگام درخواست به LLM فراخوانی می‌شود:

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

عالی، حالا که آماده‌ایم درخواست‌های کاربر را مدیریت کنیم، به بخش بعدی برویم.

### -4- رسیدگی به درخواست پرامپت کاربر

در این بخش، درخواست‌های کاربر را مدیریت خواهیم کرد.

#### TypeScript

1. متدی اضافه کنید که برای فراخوانی LLM استفاده می‌شود:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // ۲. فراخوانی ابزار سرور
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // ۳. انجام کاری با نتیجه
        // انجام بده

        }
    }
    ```

    در کد بالا:

    - متدی به نام `callTools` اضافه کردیم.
    - این متد پاسخ LLM را می‌گیرد و بررسی می‌کند چه ابزارهایی باید فراخوانی شوند، اگر وجود داشته باشند:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // فراخوانی ابزار
        }
        ```

    - ابزار را فراخوانی می‌کند، اگر LLM مشخص کند باید فراخوانده شود:

        ```typescript
        // ۲. فراخوانی ابزار سرور
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // ۳. انجام کاری با نتیجه
        // انجام شد
        ```

1. متد `run` را به‌روزرسانی کنید تا فراخوانی به LLM و فراخوانی `callTools` را شامل شود:

    ```typescript

    // ۱. ایجاد پیام‌هایی که ورودی برای مدل زبان بزرگ (LLM) هستند
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // ۲. فراخوانی مدل زبان بزرگ (LLM)
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // ۳. بررسی پاسخ مدل زبان بزرگ (LLM)، برای هر گزینه، چک کنید که آیا فراخوانی ابزار دارد یا خیر
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

عالی، بیایید کد کامل را فهرست کنیم:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ایمپورت کردن zod برای اعتبارسنجی شِما

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // ممکن است در آینده نیاز باشد این آدرس به این تغییر کند: https://models.github.ai/inference
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
          // ایجاد یک شِما زود بر اساس input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // صراحتاً نوع را به "function" تنظیم کنید
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
    
    
          // ۲. فراخوانی ابزار سرور
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // ۳. انجام کاری با نتیجه
          // انجام دادنی
    
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
    
        // ۱. مرور پاسخ LLM، برای هر انتخاب، بررسی کنید آیا فراخوانی ابزار دارد
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

1. افزونه‌هایی که برای فراخوانی LLM نیاز داریم را اضافه کنیم:

    ```python
    # مدل زبان بزرگ
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. سپس تابعی که LLM را فرا می‌خواند اضافه کنیم:

    ```python
    # مدل زبان بزرگ

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
            # پارامترهای اختیاری
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

    در کد بالا:

    - توابعی که از سرور MCP یافتیم و تبدیل کردیم را به LLM فرستادیم.
    - سپس LLM را با این توابع فراخواندیم.
    - سپس نتیجه را بررسی کردیم تا ببینیم کدام توابع باید فراخوانی شوند، اگر وجود داشته باشند.
    - در نهایت آرایه‌ای از توابع برای فراخوانی می‌دهیم.

1. مرحله نهایی، کد اصلی را به‌روزرسانی کنیم:

    ```python
    prompt = "Add 2 to 20"

    # از LLM بپرس که چه ابزارهایی دارد، اگر اصلاً دارد
    functions_to_call = call_llm(prompt, functions)

    # توابع پیشنهادی را فراخوانی کن
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    این هم گام نهایی، در کد بالا ما:

    - ابزاری از MCP را از طریق `call_tool` فراخوانی کردیم با تابعی که LLM بر اساس پرامپت ما فکر می‌کرد باید فراخوانده شود.
    - نتیجه فراخوانی ابزار به سرور MCP را چاپ کردیم.

#### .NET

1. نمونه کدی برای درخواست پرامپت LLM نشان می‌دهیم:

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

    در کد بالا:

    - ابزارها را از سرور MCP دریافت کردیم، `var tools = await GetMcpTools()`.
    - پرامپت کاربر `userMessage` تعریف کردیم.
    - شیء گزینه‌ها را با تعیین مدل و ابزارها ساختیم.
    - درخواست به LLM ارسال شد.

1. آخرین مرحله، ببینیم آیا LLM فکر می‌کند باید تابعی فراخوانی شود:

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

    در کد بالا:

    - روی فهرست فراخوانی توابع حلقه زدیم.
    - برای هر فراخوانی ابزار، نام و آرگومان‌ها را استخراج کرده و ابزار را روی سرور MCP با استفاده از کلاینت MCP فراخوانی کردیم. در نهایت نتایج چاپ شد.

در اینجا کد کامل است:

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
    // اجرای درخواست‌های زبان طبیعی که به‌طور خودکار از ابزارهای MCP استفاده می‌کنند
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

در کد بالا:

- از پرامپت‌های ساده زبان طبیعی برای تعامل با ابزارهای سرور MCP استفاده کردیم.
- چارچوب LangChain4j به‌طور خودکار مدیریت می‌کند:
  - تبدیل پرامپت‌های کاربر به فراخوانی ابزار در صورت نیاز
  - فراخوانی ابزارهای مناسب MCP بر اساس تصمیم LLM
  - مدیریت جریان گفتگو بین LLM و سرور MCP
- متد `bot.chat()` پاسخ‌های زبان طبیعی را برمی‌گرداند که ممکن است شامل نتایج اجرای ابزارهای MCP باشد
- این رویکرد تجربه کاربری روانی فراهم می‌کند که کاربر نیازی ندارد در مورد پیاده‌سازی زیرساختی MCP بداند

کد کامل نمونه:

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

در اینجا بخش اعظم کار انجام می‌شود. ما ابتدا LLM را با پرامپت اولیه کاربر فرا می‌خوانیم، سپس پاسخ را پردازش می‌کنیم تا ببینیم آیا نیاز به فراخوانی ابزارها هست یا نه. اگر هست، آن ابزارها را فراخوانی می‌کنیم و گفتگو با LLM را ادامه می‌دهیم تا زمانی که فراخوانی ابزار بیشتری مورد نیاز نباشد و پاسخ نهایی دریافت شود.

چون چندین بار به LLM فراخوانی خواهیم داشت، یک تابع تعریف می‌کنیم که فراخوانی LLM را مدیریت کند. تابع زیر را به فایل `main.rs` خود اضافه کنید:

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

این تابع کلاینت LLM، فهرستی از پیام‌ها (از جمله پرامپت کاربر)، ابزارهای سرور MCP را می‌گیرد و درخواست به LLM ارسال می‌کند و پاسخ را برمی‌گرداند.
پاسخ از LLM شامل یک آرایه از `choices` خواهد بود. ما باید نتیجه را پردازش کنیم تا ببینیم آیا `tool_calls` وجود دارد یا خیر. این باعث می‌شود بدانیم LLM درخواست فراخوانی یک ابزار خاص با آرگومان‌ها را دارد. کد زیر را به انتهای فایل `main.rs` خود اضافه کنید تا تابعی برای رسیدگی به پاسخ LLM تعریف شود:

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

    // چاپ محتوا در صورت موجود بودن
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // مدیریت تماس‌های ابزار
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // افزودن پیام دستیار

        // اجرای هر تماس ابزار
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // افزودن نتیجه ابزار به پیام‌ها
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ادامه مکالمه با نتایج ابزار
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

اگر `tool_calls` وجود داشته باشد، اطلاعات ابزار را استخراج می‌کند، با درخواست ابزار به سرور MCP فراخوانی می‌زند و نتایج را به پیام‌های مکالمه اضافه می‌کند. سپس مکالمه را با LLM ادامه می‌دهد و پیام‌ها با پاسخ دستیار و نتایج فراخوانی ابزار به‌روزرسانی می‌شوند.

برای استخراج اطلاعات فراخوانی ابزار که LLM برای فراخوانی‌های MCP باز می‌گرداند، یک تابع کمکی دیگر اضافه خواهیم کرد تا همه موارد لازم برای انجام فراخوانی را استخراج کند. کد زیر را به انتهای فایل `main.rs` خود اضافه کنید:

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

با قرار دادن همه قطعات در جای خود، اکنون می‌توانیم پیام اولیه کاربر را پردازش کرده و LLM را فراخوانی کنیم. تابع `main` خود را به شکل زیر به‌روزرسانی نمایید:

```rust
// گفتگو با مدل زبان بزرگ همراه با فراخوانی ابزارها
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

این کار، LLM را با پیام اولیه کاربر که درخواست جمع دو عدد است پرس‌وجو می‌کند و پاسخ را به گونه‌ای پردازش می‌کند که بتواند به صورت پویا فراخوانی ابزارها را مدیریت نماید.

عالی است، شما موفق شدید!

## وظیفه

کد تمرین را بردارید و سرور را با ابزارهای بیشتری توسعه دهید. سپس یک کلاینت با یک LLM مثل تمرین ایجاد کنید و با پیام‌های مختلف آن را آزمایش کنید تا مطمئن شوید همه ابزارهای سرور به صورت پویا فراخوانی می‌شوند. این روش ساخت کلاینت باعث می‌شود تجربه کاربری نهایی عالی باشد چون کاربران می‌توانند به جای استفاده از دستورات دقیق کلاینت، با استفاده از پیام‌ها کار کنند و از فراخوانی هرگونه سرور MCP بی‌خبر باشند.

## راه‌حل

[راه‌حل](/03-GettingStarted/03-llm-client/solution/README.md)

## نکات کلیدی

- افزودن یک LLM به کلاینت باعث می‌شود کاربران راه بهتری برای تعامل با سرورهای MCP داشته باشند.
- شما باید پاسخ سرور MCP را به شکلی تبدیل کنید که LLM بتواند درک کند.

## نمونه‌ها

- [ماشین حساب جاوا](../samples/java/calculator/README.md)
- [ماشین حساب .Net](../../../../03-GettingStarted/samples/csharp)
- [ماشین حساب جاوااسکریپت](../samples/javascript/README.md)
- [ماشین حساب تایپ‌اسکریپت](../samples/typescript/README.md)
- [ماشین حساب پایتون](../../../../03-GettingStarted/samples/python)
- [ماشین حساب راست](../../../../03-GettingStarted/samples/rust)

## منابع اضافی

## مرحله بعد

- مرحله بعد: [مصرف یک سرور با استفاده از Visual Studio Code](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب‌مسئولیت**:  
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در پی دقت هستیم، لطفاً آگاه باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری آن باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، استفاده از ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوءتفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
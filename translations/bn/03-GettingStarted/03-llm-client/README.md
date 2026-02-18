# LLM দিয়ে ক্লায়েন্ট তৈরি করা

এখন পর্যন্ত, আপনি দেখেছেন কীভাবে একটি সার্ভার এবং একটি ক্লায়েন্ট তৈরি করতে হয়। ক্লায়েন্ট স্পষ্টভাবেই সার্ভারকে কল করে তার টুলস, রিসোর্সেস এবং প্রম্পটগুলো তালিকাভুক্ত করতে পেরেছে। তবে এটি খুবই ব্যবহারিক পদ্ধতি নয়। আপনার ব্যবহারকারী এজেন্টিক যুগে বসবাস করছে এবং প্রম্পট ব্যবহার করে একটি LLM-এর সাথে যোগাযোগ করার প্রত্যাশা রাখে। আপনার ব্যবহারকারীর জন্য, তারা MCP ব্যবহার করে আপনার ক্ষমতাগুলো সংরক্ষণ করছেন কিনা তা মানে রাখে না, তবে তারা প্রত্যাশা করে প্রাকৃতিক ভাষা ব্যবহার করে যোগাযোগ করতে। তাহলে আমরা কীভাবে এটি সমাধান করব? সমাধান হলো ক্লায়েন্টে একটি LLM যোগ করা।

## সংক্ষিপ্ত বিবরণ

এই পাঠে আমরা ফোকাস করব ক্লায়েন্টে একটি LLM যোগ করার ওপর এবং দেখাবো কীভাবে এটি আপনার ব্যবহারকারীর জন্য অনেক ভাল অভিজ্ঞতা প্রদান করে।

## শেখার উদ্দেশ্য

এই পাঠের শেষে, আপনি সক্ষম হবেন:

- একটি LLM সহ ক্লায়েন্ট তৈরি করতে।
- একটি MCP সার্ভারের সাথে নিরবচ্ছিন্নভাবে LLM ব্যবহার করে যোগাযোগ করতে।
- ক্লায়েন্ট সাইডে উন্নত ব্যবহারকারী অভিজ্ঞতা প্রদান করতে।

## পদ্ধতি

আমরা যা করতে হবে তা বোঝার চেষ্টা করি। একটি LLM যোগ করা সহজ শোনালেও, আমরা কি সত্যিই এটি করব?

ক্লায়েন্ট কীভাবে সার্ভারের সাথে যোগাযোগ করবে তা নিম্নরূপ:

1. সার্ভারের সাথে সংযোগ স্থাপন করা।

1. কর্মক্ষমতা, প্রম্পট, রিসোর্সেস এবং টুলস তালিকাভুক্ত করা এবং তাদের স্কিমা সংরক্ষণ করা।

1. একটি LLM যোগ করা এবং সংরক্ষিত ক্ষমতা ও তাদের স্কিমা LLM বোঝার ফরম্যাটে প্রদান করা।

1. একটি ব্যবহারকারীর প্রম্পট হ্যান্ডল করা এবং এটি LLM-এ পাঠানো, একই সাথে ক্লায়েন্ট থেকে তালিকাভুক্ত টুলস যুক্ত করা।

দারুণ, এখন আমরা উচ্চ পর্যায়ে বুঝে গেছি কীভাবে এটি করা যায়, নিচের অনুশীলনে এটি চেষ্টা করি।

## অনুশীলন: একটি LLM সহ ক্লায়েন্ট তৈরি করা

এই অনুশীলনে, আমরা শিখব কীভাবে আমাদের ক্লায়েন্টে একটি LLM যোগ করতে হয়।

### গিটহাব পার্সোনাল অ্যাক্সেস টোকেন দিয়ে প্রমাণীকরণ

একটি গিটহাব টোকেন তৈরি করা সোজা প্রক্রিয়া। এখানে কীভাবে করবেন:

- GitHub সেটিংসে যান – উপরের ডান কোণায় আপনার প্রোফাইল ছবিতে ক্লিক করুন এবং Settings নির্বাচন করুন।
- Developer Settings-এ যান – স্ক্রল করে নিচে Developer Settings-এ ক্লিক করুন।
- Personal Access Tokens নির্বাচন করুন – Fine-grained tokens-এ ক্লিক করুন, তারপর Generate new token নির্বাচন করুন।
- আপনার টোকেন কনফিগার করুন – রেফারেন্সের জন্য একটি নোট যোগ করুন, মেয়াদ সেট করুন এবং প্রয়োজনীয় স্কোপ (অনুমতি) নির্বাচন করুন। এই ক্ষেত্রে Models অনুমতি যোগ করা নিশ্চিত করুন।
- টোকেন জেনারেট করুন এবং কপি করুন – Generate token এ ক্লিক করুন এবং অবিলম্বে কপি করুন, কারণ আপনি তা আর দেখতে পারবেন না।

### -1- সার্ভারের সাথে সংযোগ স্থাপন

প্রথমে চলুন আমাদের ক্লায়েন্ট তৈরি করি:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // স্কিমা বৈধতার জন্য zod আমদানি করুন

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

উপরের কোডে আমরা:

- প্রয়োজনীয় লাইব্রেরি ইমপোর্ট করেছি
- একটি ক্লাস তৈরি করেছি দুইটি সদস্যসহ, `client` এবং `openai`, যা আমাদের ক্লায়েন্ট পরিচালনা করতে এবং LLM-র সাথে যোগাযোগ করতে সাহায্য করবে।
- LLM ইনস্ট্যান্স কনফিগার করেছি GitHub Models ব্যবহার করতে `baseUrl` সেট করে inference API-এর ঠিকানায়।

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio সংযোগের জন্য সার্ভার প্যারামিটার তৈরি করুন
server_params = StdioServerParameters(
    command="mcp",  # কার্যকরযোগ্য
    args=["run", "server.py"],  # ঐচ্ছিক কমান্ড লাইন আর্গুমেন্ট
    env=None,  # ঐচ্ছিক পরিবেশ পরিবর্তনশীলগুলি
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # সংযোগ শুরু করুন
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

উপরের কোডে আমরা:

- MCP-এর জন্য প্রয়োজনীয় লাইব্রেরি ইমপোর্ট করেছি
- একটি ক্লায়েন্ট তৈরি করেছি

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

প্রথমে, আপনাকে আপনার `pom.xml` ফাইলে LangChain4j নির্ভরতাগুলো যোগ করতে হবে। MCP ইন্টিগ্রেশন এবং GitHub Models সাপোর্ট সক্রিয় করতে নিম্নলিখিত নির্ভরতাগুলো যোগ করুন:

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

তারপর আপনার Java ক্লায়েন্ট ক্লাস তৈরি করুন:

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
    
    public static void main(String[] args) throws Exception {        // LLM-কে GitHub মডেল ব্যবহার করার জন্য কনফিগার করুন
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // সার্ভারের সাথে সংযোগ করার জন্য MCP ট্রান্সপোর্ট তৈরি করুন
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP ক্লায়েন্ট তৈরি করুন
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

উপরের কোডে আমরা:

- **LangChain4j নির্ভরতাগুলো যোগ করেছি**: MCP ইন্টিগ্রেশন, OpenAI অফিসিয়াল ক্লায়েন্ট, এবং GitHub Models সাপোর্টের জন্য
- **LangChain4j লাইব্রেরি ইমপোর্ট করেছি**: MCP ইন্টিগ্রেশন এবং OpenAI চ্যাট মডেল ফাংশনালিটির জন্য
- **একটি `ChatLanguageModel` তৈরি করেছি**: GitHub Models ব্যবহার করার জন্য আপনার গিটহাব টোকেন দিয়ে কনফিগার করা
- **HTTP ট্রান্সপোর্ট সেটআপ করেছি**: Server-Sent Events (SSE) ব্যবহার করে MCP সার্ভারের সাথে সংযোগের জন্য
- **একটি MCP ক্লায়েন্ট তৈরি করেছি**: যা সার্ভারের সাথে যোগাযোগ পরিচালনা করবে
- **LangChain4j-এর MCP সাপোর্ট ব্যবহার করেছি**: যা LLM আর MCP সার্ভারের ইন্টিগ্রেশন সহজ করে

#### Rust

এই উদাহরণটি ধরে নেয় যে আপনার কাছে একটি Rust ভিত্তিক MCP সার্ভার চলছে। যদি না থাকে, তাহলে [01-first-server](../01-first-server/README.md) লেসনে ফিরে গিয়ে সার্ভার তৈরি করুন।

আপনার Rust MCP সার্ভার থাকা অবস্থায়, টার্মিনাল খুলুন এবং সার্ভারের ডিরেক্টরিতে যান। তারপর নিম্নলিখিত কমান্ড দিয়ে একটি নতুন LLM ক্লায়েন্ট প্রজেক্ট তৈরি করুন:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

আপনার `Cargo.toml` ফাইলে নিম্নলিখিত নির্ভরতাগুলো যোগ করুন:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI-এর জন্য কোনো অফিসিয়াল Rust লাইব্রেরি নেই, তবে `async-openai` crate একটি [কমিউনিটি রক্ষণাবেক্ষিত লাইব্রেরি](https://platform.openai.com/docs/libraries/rust#rust) যা সাধারণত ব্যবহৃত হয়।

`src/main.rs` ফাইলটি খুলুন এবং নিচের কোড দিয়ে এর কন্টেন্ট প্রতিস্থাপন করুন:

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
    // প্রাথমিক বার্তা
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI ক্লায়েন্ট সেটআপ করুন
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP ক্লায়েন্ট সেটআপ করুন
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

    // TODO: MCP টুল তালিকা পান

    // TODO: টুল কল সহ LLM কথোপকথন

    Ok(())
}
```

এই কোড একটি বেসিক Rust অ্যাপ্লিকেশন সেটআপ করে যা MCP সার্ভার এবং GitHub Models-এর সাথে LLM ইন্টারঅ্যাকশনের জন্য সংযোগ স্থাপন করবে।

> [!IMPORTANT]
> অ্যাপ্লিকেশন চালানোর আগে অবশ্যই `OPENAI_API_KEY` পরিবেশ পরিবর্তনীতে আপনার গিটহাব টোকেন সেট করুন।

দারুণ, এরপরের ধাপে চলুন সার্ভারের ক্ষমতাগুলো তালিকাভুক্ত করি।

### -2- সার্ভারের ক্ষমতা তালিকাভুক্ত করা

এখন আমরা সার্ভারের সাথে সংযোগ স্থাপন করব এবং তার ক্ষমতাগুলো জিজ্ঞেস করব:

#### TypeScript

একই ক্লাসে, নিচের মেথডগুলো যুক্ত করুন:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // সরঞ্জাম তালিকা করা
    const toolsResult = await this.client.listTools();
}
```

উপরে আমরা:

- সার্ভারের সাথে সংযোগ করার কোড যোগ করেছি, `connectToServer`.
- একটি `run` মেথড তৈরি করেছি যা আমাদের অ্যাপ ফ্লো হ্যান্ডল করবে। এখন পর্যন্ত এটি শুধুমাত্র টুলস তালিকাভুক্ত করে, তবে আমরা শীঘ্রই আরও যুক্ত করব।

#### Python

```python
# উপলব্ধ সম্পদগুলির তালিকা দিন
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# উপলব্ধ সরঞ্জামগুলির তালিকা দিন
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

আমরা যা যুক্ত করেছি:

- রিসোর্স এবং টুলস তালিকাভুক্ত করা এবং প্রিন্ট করা। টুলসের জন্য আমরা `inputSchema` ও তালিকাভুক্ত করেছি যা পরে ব্যবহার করব।

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

উপরে আমরা:

- MCP সার্ভারে উপলব্ধ টুলস তালিকাভুক্ত করেছি
- প্রতিটি টুলের নাম, বিবরণ এবং তার স্কিমা তালিকাভুক্ত করেছি। পরবর্তী সময়ে আমরা এটি ব্যবহার করব টুলস কল করার জন্য।

#### Java

```java
// একটি টুল প্রদানকারী তৈরি করুন যা স্বয়ংক্রিয়ভাবে MCP টুলগুলো আবিষ্কার করে
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP টুল প্রদানকারী স্বয়ংক্রিয়ভাবে পরিচালনা করে:
// - MCP সার্ভার থেকে উপলব্ধ টুল তালিকা
// - MCP টুল স্কিমাগুলো LangChain4j ফরম্যাটে রূপান্তর করা
// - টুলের কার্যকরীতা এবং প্রতিক্রিয়া পরিচালনা করা
```

উপরে আমরা:

- একটি `McpToolProvider` তৈরি করেছি যা MCP সার্ভারের সব টুল স্বয়ংক্রিয়ভাবে আবিষ্কার ও রেজিস্টার করে
- টুল প্রোভাইডার MCP টুল স্কিমা থেকে LangChain4j টুল ফরম্যাটে রূপান্তর অভ্যন্তরীণভাবে হ্যান্ডল করে
- এই পদ্ধতি ম্যানুয়াল টুল তালিকা ও রূপান্তরের প্রয়োজনীয়তা দূর করে

#### Rust

MCP সার্ভারের টুলগুলো আনার জন্য `list_tools` মেথড ব্যবহার হয়। আপনার `main` ফাংশনে, MCP ক্লায়েন্ট সেটআপ করার পর নিম্নলিখিত কোড যোগ করুন:

```rust
// এমসিপি টুল তালিকা পান
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- সার্ভারের ক্ষমতাগুলোকে LLM টুলসে রূপান্তর করা

সার্ভারের ক্ষমতাগুলো তালিকাভুক্ত করার পরের ধাপ হলো সেগুলোকে এমন ফরম্যাটে রূপান্তর করা যা LLM বুঝতে পারে। এরপর আমরা LLM-কে এই টুলস হিসেবে দিতে পারবো।

#### TypeScript

1. MCP সার্ভারের রেসপন্সকে LLM ব্যবহারযোগ্য টুল ফরম্যাটে রূপান্তর করতে নিচের কোড যোগ করুন:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ইনপুট_স্কিমার উপর ভিত্তি করে একটি জড স্কিমা তৈরি করুন
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // স্পষ্টভাবে টাইপ "function" সেট করুন
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

    উপরের কোড MCP সার্ভারের রেসপন্স নিয়ে সেটিকে LLM বোঝার মত টুল ডেফিনিশনে রূপান্তর করে।

1. এবার `run` মেথড আপডেট করি সার্ভারের ক্ষমতাগুলো তালিকাভুক্ত করতে:

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

    উপরে আমরা `run` মেথড আপডেট করেছি, যেখানে ফলাফল ম্যাপ করে প্রতিটি এন্ট্রির জন্য `openAiToolAdapter` কল করা হয়।

#### Python

1. প্রথমে নিচের রূপান্তরকারী ফাংশন তৈরি করি:

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

    ফাংশন `convert_to_llm_tools` MCP টুল রেসপন্স নেয় এবং LLM বুঝতে পারার মত একটি ফরম্যাটে রূপান্তর করে।

1. পরবর্তীতে আমাদের ক্লায়েন্ট কোড আপডেট করি যা এই ফাংশন ব্যবহার করবে:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    এখানে আমরা MCP টুল রেসপন্সকে LLM-এ ফিড করার মত রূপান্তর করতে `convert_to_llm_tool` কল যুক্ত করেছি।

#### .NET

1. MCP টুল রেসপন্সকে LLM বুঝতে পারার মত রূপান্তর করার জন্য কোড যোগ করি:

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

উপরে আমরা:

- একটি `ConvertFrom` ফাংশন তৈরি করেছি যা নাম, বিবরণ এবং ইনপুট স্কিমা নেয়।
- ফাংশন যা একটি FunctionDefinition তৈরি করে যা ChatCompletionsDefinition-এ পাঠানো যায়। এটি একটা ফরম্যাট যা LLM বুঝতে পারে।

1. পরবর্তী ধাপে কিছু বিদ্যমান কোড আপডেট করি যাতে এই ফাংশনের সুবিধা নেয়া যায়:

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
// প্রাকৃতিক ভাষার ইন্টারঅ্যাকশনের জন্য একটি বট ইন্টারফেস তৈরি করুন
public interface Bot {
    String chat(String prompt);
}

// LLM এবং MCP টুল ব্যবহার করে AI পরিষেবা কনফিগার করুন
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

উপরে আমরা:

- একটি সরল `Bot` ইন্টারফেস ডিফাইন করেছি প্রাকৃতিক ভাষার যোগাযোগের জন্য
- LangChain4j-এর `AiServices` ব্যবহার করেছি যা LLM কে MCP টুল প্রোভাইডারের সাথে স্বয়ংক্রিয়ভাবে বেঁধে দেয়
- ফ্রেমওয়ার্ক স্বয়ংক্রিয়ভাবে টুল স্কিমা রূপান্তর এবং ফাংশন কলিং হ্যান্ডল করে
- এই পদ্ধতি ম্যানুয়াল টুল রূপান্তর দূর করেছে – LangChain4j MCP টুলগুলোকে LLM-সামঞ্জস্যপূর্ণ ফরম্যাটে রূপান্তর করার জটিলতা সামলে নেয়

#### Rust

MCP টুল রেসপন্সকে LLM বুঝতে পৰা ফরম্যাটে রূপান্তর করার জন্য একটি হেল্পার ফাংশন যোগ করব যা টুল লিস্টিং ফরম্যাট করে। আপনার `main.rs` ফাইলে `main` ফাংশনের নিচে নিম্নলিখিত কোড যোগ করুন। এটি LLM-এ রিকোয়েস্ট করার সময় কল করা হবে:

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

দারুণ, এখন আমরা ব্যবহারকারীর রিকোয়েস্ট হ্যান্ডল করার জন্য প্রস্তুত, তাই চলুন সেটি করি।

### -4- ব্যবহারকারীর প্রম্পট রিকোয়েস্ট হ্যান্ডল করা

এই অংশে, আমরা ব্যবহারকারীর রিকোয়েস্ট হ্যান্ডল করব।

#### TypeScript

1. আমাদের LLM কল করার জন্য একটি মেথড যোগ করি:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // ২। সার্ভারের সরঞ্জাম কল করুন
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // ৩। ফলাফলের সাথে কিছু করুন
        // টুডু

        }
    }
    ```

উপরে আমরা:

- একটি মেথড `callTools` যোগ করেছি।
- মেথডটি একটি LLM রেসপন্স নেয় এবং পরীক্ষা করে কোন টুলো গুলো কল হয়েছে, যদি থাকে:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // টুল কল করুন
        }
        ```

- যদি LLM নির্দেশ দেয় যে কোন টুল কল করা উচিত, তাহলে টুল কল করা হয়:

        ```typescript
        // ২. সার্ভারের টুল কল করুন
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // ৩. ফলাফলের সাথে কিছু করুন
        // টুডু
        ```

1. `run` মেথড আপডেট করি যাতে LLM কল এবং `callTools` অন্তর্ভুক্ত থাকে:

    ```typescript

    // 1. LLM-এর ইনপুট হিসেবে মেসেজ তৈরি করুন
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM কল করা হচ্ছে
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM এর প্রতিক্রিয়া পরীক্ষা করুন, প্রতিটি বিকল্পের জন্য দেখুন এতে টুল কল আছে কিনা
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

দারুণ, সম্পূর্ণ কোড একসাথে দেখি:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // স্কিমা যাচাইয়ের জন্য zod আমদানি করুন

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // ভবিষ্যতে এই URL-এ পরিবর্তন করতে হতে পারে: https://models.github.ai/inference
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
          // ইনপুট_স্কিমা অনুযায়ী একটি zod স্কিমা তৈরি করুন
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // স্পষ্টভাবে টাইপ "function" সেট করুন
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
    
    
          // ২. সার্ভারের টুল কল করুন
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // ৩. ফলাফলের সাথে কিছু কাজ করুন
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
    
        // ১. LLM প্রতিক্রিয়া অনুসরণ করুন, প্রতিটি পছন্দের জন্য পরীক্ষা করুন এটি টুল কল আছে কিনা
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

1. LLM কল করার জন্য কিছু প্রয়োজনীয় ইমপোর্ট যোগ করি:

    ```python
    # এলএলএম
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. এরপর LLM কলের ফাংশন যোগ করি:

    ```python
    # এলএলএম

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
            # ঐচ্ছিক প্যারামিটারসমূহ
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

উপরের কোডে:

- আমরা MCP সার্ভারে পাওয়া ফাংশনগুলো LLM-এ পাস করেছি।
- তারপর LLM কে ওই ফাংশনগুলোসহ কল করেছি।
- তারপর রেজাল্ট ইনস্পেক্ট করেছি কোন ফাংশনগুলো কল করা উচিত।
- শেষে একটি ফাংশনের অ্যারে পাস করছি কল করার জন্য।

1. শেষ ধাপে, আমাদের মেইন কোড আপডেট করি:

    ```python
    prompt = "Add 2 to 20"

    # যদি থাকে তবে LLM-কে সব টুল কত জানতে বলুন
    functions_to_call = call_llm(prompt, functions)

    # প্রস্তাবিত ফাংশনগুলি কল করুন
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

উপরের কোডে আমরা:

- LLM-এর প্রম্পটের ভিত্তিতে MCP টুল কল করার জন্য `call_tool` ব্যবহার করেছি।
- MCP সার্ভারের টুল কলের ফলাফল প্রিন্ট করেছি।

#### .NET

1. LLM প্রম্পট রিকোয়েস্টের কোড দেখানো হল:

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

উপরে আমরা:

- MCP সার্ভার থেকে টুলস এনেছি, `var tools = await GetMcpTools()`.
- একটি ব্যবহারকারী প্রম্পট `userMessage` ডিফাইন করেছি।
- মডেল এবং টুলস সহ একটি options অবজেক্ট কনস্ট্রাক্ট করেছি।
- LLM এর প্রতি অনুরোধ করেছি।

1. শেষ ধাপে, দেখি LLM কি ফাংশন কল করা উচিত বলে মনে করে:

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

উপরে আমরা:

- ফাংশন কলের একটি তালিকা লুপ করেছি।
- প্রতিটি টুল কলের জন্য নাম ও আর্গুমেন্ট পার্স করে MCP ক্লায়েন্ট দিয়ে কল করেছি এবং ফলাফল প্রিন্ট করেছি।

সম্পূর্ণ কোড এখানে:

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
    // স্বয়ংক্রিয়ভাবে MCP সরঞ্জাম ব্যবহার করে প্রাকৃতিক ভাষার অনুরোধগুলি কার্যকর করুন
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

উপরের কোডে আমরা:

- সরল প্রাকৃতিক ভাষার প্রম্পট ব্যবহার করে MCP সার্ভারের টুলসের সাথে যোগাযোগ করেছি
- LangChain4j ফ্রেমওয়ার্ক স্বয়ংক্রিয়ভাবে হ্যান্ডল করে:
  - প্রয়োজনমতো ব্যবহারকারীর প্রম্পট থেকে টুল কলে রূপান্তর
  - LLM-এর সিদ্ধান্ত অনুযায়ী সঠিক MCP টুলগুলো কল করা
  - LLM ও MCP সার্ভারের মধ্যে কথোপকথনের প্রবাহ পরিচালনা
- `bot.chat()` মেথড প্রাকৃতিক ভাষার প্রতিক্রিয়া প্রদান করে যা MCP টুল এক্সিকিউশনের ফলাফল থাকতে পারে
- এই পদ্ধতি একটি ব্যবহারকারী-বান্ধব অভিজ্ঞতা দেয় যেখানে ব্যবহারকারীদের MCP বাস্তবায়ন সম্পর্কে জানার প্রয়োজন হয় না

সম্পূর্ণ কোড উদাহরণ:

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

এখানেই মূল কাজ ঘটে। আমরা LLM কল করব প্রাথমিক ব্যবহারকারীর প্রম্পট দিয়ে, তারপর রেসপন্স প্রক্রিয়া করব দেখে যদি কোন টুল কল করার দরকার হয় কী না। যদি থাকে, সে টুলগুলো কল করব এবং LLM-এর সাথে কথোপকথন চালিয়ে যাব যতক্ষণ না আর টুল কলের প্রয়োজন হয় না এবং আমাদের একটি চূড়ান্ত রেসপন্স থাকে।

আমরা LLM কে একাধিকবার কল করব, তাই একটি ফাংশন ডিফাইন করি যা LLM কল হ্যান্ডল করবে। নিচের ফাংশন আপনার `main.rs` ফাইলে যোগ করুন:

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

এই ফাংশনটি LLM ক্লায়েন্ট, মেসেজের তালিকা (ব্যবহারকারীর প্রম্পট সহ), MCP সার্ভারের টুলস নেয় এবং LLM-এ রিকোয়েস্ট পাঠায়, ফলাফল ফেরত দেয়।
LLM থেকে প্রাপ্ত উত্তর একটি `choices` এর অ্যারে থাকবে। আমাদের ফলাফল প্রসেস করতে হবে দেখে নিতে যদি কোন `tool_calls` উপস্থিত থাকে। এটি আমাদের জানাবে LLM বিশেষ কোন টুল কল করার অনুরোধ করছে কী না, সাথে আর্গুমেন্ট সহ। নিচের কোডটি আপনার `main.rs` ফাইলের শেষ অংশে যোগ করুন একটি ফাংশন ডিফাইন করতে যা LLM এর উত্তর হ্যান্ডেল করবে:

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

    // উপাদান প্রিন্ট করুন যদি উপলব্ধ থাকে
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // টুল কলগুলি পরিচালনা করুন
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // সহকারী বার্তা যুক্ত করুন

        // প্রতিটি টুল কল কার্যকর করুন
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // টুল ফলাফল বার্তাগুলিতে যুক্ত করুন
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // টুল ফলাফলের সঙ্গে কথোপকথন চালিয়ে যান
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


যদি `tool_calls` উপস্থিত থাকে, এটি টুলের তথ্য বের করে, MCP সার্ভারকে টুল রিকোয়েস্ট পাঠায়, এবং রেজাল্টগুলো সংলাপের মেসেজে যোগ করে। তারপর তা LLM এর সাথে সংলাপ অব্যাহত রাখে এবং মেসেজগুলো আপডেট হয় সহকারী (assistant) এর উত্তর ও টুল কল রেজাল্টসহ।

LLM MCP কলের জন্য যেসব টুল কল তথ্য প্রদান করে তার সমস্ত প্রয়োজনীয় তথ্য বের করতে আমরা আরেকটি সাহায্যকারী ফাংশন যোগ করব। নিচের কোডটি আপনার `main.rs` ফাইলের শেষে যোগ করুন:

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


সব কাঠামো ঠিকঠাক বসানোর পর, আমরা এখনিতেই ব্যবহারকারীর প্রাথমিক প্রম্পট হ্যান্ডেল করতে এবং LLM কল করতে পারব। আপনার `main` ফাংশন আপডেট করুন নিম্নলিখিত কোডসহ:

```rust
// টুল কল সহ এলএলএম কথোপকথন
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


এটি ব্যবহারকারীর প্রাথমিক প্রম্পট সহ LLM কে প্রশ্ন করবে দুইটি সংখ্যার যোগফল সম্পর্কে, আর তা প্রসেস করবে রেসপন্স যাতে ডায়নামিকভাবে টুল কলগুলো হ্যান্ডেল করা যায়।

শাবাশ, আপনি সফল হয়েছেন!

## অ্যাসাইনমেন্ট

এক্সারসাইজ থেকে কোড নিয়ে আরও কিছু টুলসহ সার্ভার তৈরি করুন। তারপর একটি ক্লায়েন্ট তৈরি করুন LLM সহ, যেমন এক্সারসাইজে ছিল, এবং বিভিন্ন প্রম্পট দিয়ে পরীক্ষা করুন যাতে আপনার সব সার্ভার টুল ডায়নামিকভাবে কল হচ্ছে কিনা নিশ্চিত করতে পারেন। এই পদ্ধতিতে ক্লায়েন্ট তৈরি করলে শেষ ব্যবহারকারী একটি অসাধারণ ব্যবহারকারীর অভিজ্ঞতা পাবে কারণ তারা ডিরেক্ট ক্লায়েন্ট কমান্ডের পরিবর্তে প্রম্পট ব্যবহার করতে পারবে এবং MCP সার্ভার কলের কোনো ঝামেলা বুঝতে পারবে না।

## সমাধান

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## প্রধান শিক্ষণীয় বিষয়

- আপনার ক্লায়েন্টে LLM যোগ করা ব্যবহারকারীদের MCP সার্ভারের সাথে ইন্টারঅ্যাক্ট করার উন্নত উপায় দেয়।
- MCP সার্ভারের উত্তর এমন কিছুতে রূপান্তর করতে হবে যা LLM বুঝতে পারে।

## নমুনা

- [জাভা ক্যালকুলেটর](../samples/java/calculator/README.md)
- [.নেট ক্যালকুলেটর](../../../../03-GettingStarted/samples/csharp)
- [জাভাস্ক্রিপ্ট ক্যালকুলেটর](../samples/javascript/README.md)
- [টাইপস্ক্রিপ্ট ক্যালকুলেটর](../samples/typescript/README.md)
- [পাইথন ক্যালকুলেটর](../../../../03-GettingStarted/samples/python)
- [রাস্ট ক্যালকুলেটর](../../../../03-GettingStarted/samples/rust)

## অতিরিক্ত সম্পদ

## পরবর্তী ধাপ

- পরবর্তী: [ভিজ্যুয়াল স্টুডিও কোড ব্যবহার করে সার্ভার ব্যবহার](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকারোক্তি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। আমরা যথাসাধ্য সঠিকতার জন্য কাজ করি, তবে অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ভুল বা নির্ভুলতার ঘাটতি থাকতে পারে। মূল নথিটি তার স্থানীয় ভাষায় প্রামাণিক উৎস হিসেবে বিবেচনা করা উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদের পরামর্শ দেওয়া হয়। এই অনুবাদের ব্যবহারের ফলে কোনো ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়ী নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# 使用 LLM 创建客户端

到目前为止，您已经了解了如何创建服务器和客户端。客户端能够显式调用服务器以列出其工具、资源和提示。然而，这种方法并不太实用。您的用户处于智能代理时代，期望使用提示并与 LLM 进行交流。对您的用户来说，他们不在意您是否使用 MCP 来存储您的能力，但他们确实期望使用自然语言进行交互。那么，我们该如何解决这个问题？解决方案是向客户端添加一个 LLM。

## 概述

本课重点是向您的客户端添加一个 LLM，并展示这如何为您的用户提供更好的体验。

## 学习目标

在本课结束时，您将能够：

- 创建包含 LLM 的客户端。
- 使用 LLM 无缝地与 MCP 服务器进行交互。
- 在客户端提供更好的终端用户体验。

## 方法

让我们尝试理解我们需要采取的方法。添加 LLM 听起来很简单，但我们实际上会这样做吗？

客户端与服务器的交互方式如下：

1. 与服务器建立连接。

1. 列出能力、提示、资源和工具，并保存它们的架构。

1. 添加 LLM，并以 LLM 理解的格式传入已保存的能力及其架构。

1. 通过传递用户提示及客户端列出的工具来处理用户提示。

很好，现在我们理解了如何在高层次上实现它，让我们在下面的练习中尝试一下。

## 练习：使用 LLM 创建客户端

在本练习中，我们将学习如何向客户端添加 LLM。

### 使用 GitHub 个人访问令牌进行身份验证

创建 GitHub 令牌是一个简单的过程。方法如下：

- 进入 GitHub 设置 – 点击右上角的头像，选择设置。
- 导航到开发者设置 – 向下滚动并点击“开发者设置”。
- 选择个人访问令牌 – 点击细粒度令牌（Fine-grained tokens），然后生成新的令牌。
- 配置您的令牌 – 添加备注以备参考，设置过期日期，并选择必要的权限范围。在此情况下，确保添加“Models”权限。
- 生成并复制令牌 – 点击生成令牌，生成后请确保立即复制，因为之后无法再次查看。

### -1- 连接到服务器

让我们先创建客户端：

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // 导入 zod 进行模式验证

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

在上述代码中我们：

- 导入了所需库
- 创建了一个包含两个成员 `client` 和 `openai` 的类，分别帮助我们管理客户端和与 LLM 交互。
- 配置了我们的 LLM 实例使用 GitHub Models，设置 `baseUrl` 指向推理 API。

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# 为stdio连接创建服务器参数
server_params = StdioServerParameters(
    command="mcp",  # 可执行文件
    args=["run", "server.py"],  # 可选的命令行参数
    env=None,  # 可选的环境变量
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # 初始化连接
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

在上述代码中我们：

- 导入了 MCP 所需库
- 创建了一个客户端

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

首先，您需要将 LangChain4j 依赖项添加到您的 `pom.xml` 文件中。添加这些依赖以启用 MCP 集成和支持 GitHub Models：

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

然后创建您的 Java 客户端类：

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
    
    public static void main(String[] args) throws Exception {        // 配置LLM以使用GitHub模型
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // 创建用于连接服务器的MCP传输
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // 创建MCP客户端
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

在上述代码中我们：

- **添加了 LangChain4j 依赖**：用于 MCP 集成、OpenAI 官方客户端以及 GitHub Models 支持
- **导入了 LangChain4j 库**：用于 MCP 集成和 OpenAI 聊天模型功能
- **创建了 `ChatLanguageModel`**：配置为使用您的 GitHub 令牌连接 GitHub Models
- **设置了 HTTP 传输**：使用服务器推送事件（SSE）连接至 MCP 服务器
- **创建了 MCP 客户端**：处理与服务器的通信
- **使用了 LangChain4j 内置的 MCP 支持**：简化了 LLM 与 MCP 服务器间的集成

#### Rust

该示例假设您已有基于 Rust 的 MCP 服务器运行。如果您没有，请回到 [01-first-server](../01-first-server/README.md) 课程创建服务器。

拥有 Rust MCP 服务器后，打开终端并进入与服务器相同的目录，然后运行以下命令创建新的 LLM 客户端项目：

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

在您的 `Cargo.toml` 文件中添加以下依赖：

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> 目前没有官方的 Rust OpenAI 库，但 `async-openai` crate 是一个 [社区维护的库](https://platform.openai.com/docs/libraries/rust#rust)，被广泛使用。

打开 `src/main.rs` 文件，将其内容替换为以下代码：

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
    // 初始消息
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // 设置 OpenAI 客户端
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // 设置 MCP 客户端
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

    // 待办事项：获取 MCP 工具列表

    // 待办事项：使用工具调用进行大型语言模型对话

    Ok(())
}
```

此代码设置了一个基本的 Rust 应用，它将连接 MCP 服务器和 GitHub Models 以进行 LLM 交互。

> [!IMPORTANT]
> 运行应用前，请确保设置环境变量 `OPENAI_API_KEY` 为您的 GitHub 令牌。

很好，下一步，让我们列出服务器上的能力。

### -2- 列出服务器能力

现在我们将连接到服务器并请求它的能力：

#### Typescript

在同一个类中，添加以下方法：

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // 列出工具
    const toolsResult = await this.client.listTools();
}
```

在上述代码中我们：

- 添加了连接服务器的代码 `connectToServer`。
- 创建了一个负责处理应用流程的 `run` 方法。到目前为止，它只列出了工具，但稍后我们将添加更多功能。

#### Python

```python
# 列出可用资源
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# 列出可用工具
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

这里是我们添加的内容：

- 列出了资源和工具并打印它们。对于工具，我们还列出了稍后要用的 `inputSchema`。

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

在上述代码中我们：

- 列出了 MCP 服务器上的可用工具
- 对每个工具，列出了名称、描述及其架构。后者将在稍后调用工具时使用。

#### Java

```java
// 创建一个自动发现MCP工具的工具提供者
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP工具提供者自动处理：
// - 从MCP服务器列出可用工具
// - 将MCP工具模式转换为LangChain4j格式
// - 管理工具的执行和响应
```

在上述代码中我们：

- 创建了一个 `McpToolProvider`，自动发现并注册 MCP 服务器上的所有工具
- 工具提供者内部处理了 MCP 工具架构与 LangChain4j 工具格式之间的转换
- 这种方法抽象了手动工具列出和转换的过程

#### Rust

从 MCP 服务器检索工具使用 `list_tools` 方法。在您的 `main` 函数中，设置 MCP 客户端之后，添加以下代码：

```rust
// 获取MCP工具列表
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- 将服务器能力转换为 LLM 工具

列出服务器能力后，下一步是将它们转换为 LLM 理解的格式。转换完成后，我们可以将这些能力作为工具提供给 LLM。

#### TypeScript

1. 添加以下代码，将 MCP 服务器响应转换为 LLM 可用的工具格式：

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // 根据 input_schema 创建一个 zod 模式
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // 显式将类型设置为“function”
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

    上述代码将 MCP 服务器的响应转换为 LLM 能理解的工具定义格式。

1. 接着更新 `run` 方法以列出服务器能力：

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

    在上述代码中，我们更新了 `run` 方法，对结果进行映射，并对每一项调用了 `openAiToolAdapter`。

#### Python

1. 首先，创建以下转换函数：

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

    在函数 `convert_to_llm_tools` 中，我们接收 MCP 工具响应并将其转换为 LLM 能理解的格式。

1. 接下来，更新客户端代码以利用该函数，如下所示：

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    这里，我们添加了对 `convert_to_llm_tool` 的调用，将 MCP 工具响应转换为能够传递给 LLM 的数据。

#### .NET

1. 添加代码将 MCP 工具响应转换为 LLM 可理解的格式：

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

在上述代码中我们：

- 创建了函数 `ConvertFrom`，接受名称、描述和输入架构。
- 定义了能创建 `FunctionDefinition` 的功能，随后传递给 `ChatCompletionsDefinition`，后者是 LLM 可理解的格式。

1. 看看如何更新现有代码以使用此函数：

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
// 创建一个用于自然语言交互的机器人接口
public interface Bot {
    String chat(String prompt);
}

// 使用LLM和MCP工具配置AI服务
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

在上述代码中我们：

- 定义了一个用于自然语言交互的简单 `Bot` 接口
- 使用 LangChain4j 的 `AiServices` 来自动绑定 LLM 与 MCP 工具提供者
- 框架自动处理工具架构转换及函数调用细节
- 此方法消除了手动转换工具过程 —— LangChain4j 处理所有 MCP 工具到 LLM 兼容格式的复杂性

#### Rust

为将 MCP 工具响应转换为 LLM 理解的格式，我们将添加辅助函数来格式化工具列表。将以下代码添加到 `main.rs` 文件的 `main` 函数下方。调用 LLM 时会使用此函数：

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

很好，我们已准备好处理用户请求，接下来开始实现。

### -4- 处理用户提示请求

此部分代码将处理用户请求。

#### TypeScript

1. 添加一个方法用于调用 LLM：

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. 调用服务器的工具
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. 对结果做一些处理
        // 待办事项

        }
    }
    ```

    在上述代码中我们：

    - 添加了方法 `callTools`。
    - 该方法接收 LLM 响应并检查是否调用了工具：

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // 调用工具
        }
        ```

    - 若 LLM 表示应调用，调用工具：

        ```typescript
        // 2. 调用服务器的工具
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. 对结果做一些处理
        // 待办事项
        ```

1. 更新 `run` 方法以调用 LLM 和 `callTools`：

    ```typescript

    // 1. 创建作为LLM输入的消息
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. 调用LLM
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. 遍历LLM响应，对每个选项检查是否包含工具调用
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

很好，让我们完整列出代码：

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // 导入 zod 进行模式验证

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // 以后可能需要更改为此网址：https://models.github.ai/inference
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
          // 根据 input_schema 创建一个 zod 模式
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // 明确设置类型为 "function"
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
    
    
          // 2. 调用服务器的工具
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. 对结果进行处理
          // 待办事项
    
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
    
        // 1. 浏览 LLM 响应，对于每个选择，检查是否有工具调用
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

1. 添加调用 LLM 需要的导入：

    ```python
    # 大型语言模型
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. 接着添加调用 LLM 的函数：

    ```python
    # 大语言模型

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
            # 可选参数
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

    在上述代码中我们：

    - 将从 MCP 服务器获取并转换的函数传递给 LLM。
    - 调用带有这些函数的 LLM。
    - 检查结果以确定是否有函数要调用。
    - 最终传递一个函数调用数组。

1. 最后，更新主流程代码：

    ```python
    prompt = "Add 2 to 20"

    # 询问大型语言模型是否需要使用任何工具
    functions_to_call = call_llm(prompt, functions)

    # 调用建议的函数
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    以上就是最终步骤，我们：

    - 使用 `call_tool` 调用 MCP 工具，调用名称由 LLM 基于提示确定。
    - 打印调用 MCP 服务器工具的结果。

#### .NET

1. 展示进行 LLM 提示请求的代码：

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

    在上述代码中我们：

    - 从 MCP 服务器获取工具，`var tools = await GetMcpTools()`。
    - 定义用户提示 `userMessage`。
    - 构造包含模型和工具的选项对象。
    - 向 LLM 发送请求。

1. 最后一步，看看是否应调用函数：

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

    在上述代码中我们：

    - 遍历函数调用列表。
    - 对每次调用，解析名称及参数，通过 MCP 客户端调用 MCP 服务器上的工具，并打印结果。

完整代码如下：

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
    // 执行自动使用MCP工具的自然语言请求
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

在上述代码中我们：

- 使用简单的自然语言提示与 MCP 服务器工具交互
- LangChain4j 框架自动处理：
  - 根据需要将用户提示转化为工具调用
  - 基于 LLM 决策调用合适的 MCP 工具
  - 管理 LLM 与 MCP 服务器之间的对话流程
- `bot.chat()` 方法返回可能包含 MCP 工具执行结果的自然语言响应
- 此方法提供无缝用户体验，用户无需了解底层 MCP 实现细节

完整代码示例：

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

这里是主要工作部分。我们首先调用 LLM 发送用户初始提示，随后处理响应，看是否需要调用工具。如果需要，将调用那些工具，并持续与 LLM 交互，直到不再需要调用工具并得到最终响应。

我们将进行多次 LLM 调用，因此定义一个函数用于处理所有 LLM 调用。将以下函数添加到 `main.rs` 文件中：

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

该函数接收 LLM 客户端、一系列消息（包括用户提示）、来自 MCP 服务器的工具，并向 LLM 发送请求，返回其响应。
LLM 的响应将包含一个 `choices` 数组。我们需要处理结果以查看是否存在任何 `tool_calls`。这让我们知道 LLM 正在请求调用带有参数的特定工具。将以下代码添加到你的 `main.rs` 文件底部，用于定义处理 LLM 响应的函数：

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

    // 如果有内容则打印
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // 处理工具调用
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // 添加助手消息

        // 执行每个工具调用
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // 将工具结果添加到消息中
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // 使用工具结果继续对话
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

如果存在 `tool_calls`，它会提取工具信息，调用 MCP 服务器发送工具请求，并将结果添加到对话消息中。然后它继续与 LLM 进行对话，消息会根据助手的响应和工具调用结果进行更新。

为了提取 LLM 返回的用于 MCP 调用的工具调用信息，我们将添加另一个辅助函数来提取调用所需的所有内容。将以下代码添加到你的 `main.rs` 文件底部：

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

所有部分到位后，我们现在可以处理初始用户提示并调用 LLM。更新你的 `main` 函数，包含以下代码：

```rust
// LLM 与工具调用的对话
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

这会使用初始用户提示询问两个数字的和来查询 LLM，并处理响应以动态处理工具调用。

太棒了，你完成了！

## 任务

使用练习中的代码构建服务器，增加更多工具。然后创建一个带有 LLM 的客户端，如练习所示，并使用不同的提示测试，确保你的所有服务器工具都能被动态调用。这种构建客户端的方式意味着最终用户将获得极佳的用户体验，因为他们可以使用提示，而不是精确的客户端命令，并且对任何被调用的 MCP 服务器毫无察觉。

## 解决方案

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## 关键要点

- 在客户端添加 LLM 为用户提供了与 MCP 服务器交互的更好方式。
- 你需要将 MCP 服务器的响应转换为 LLM 能理解的内容。

## 示例

- [Java 计算器](../samples/java/calculator/README.md)
- [.Net 计算器](../../../../03-GettingStarted/samples/csharp)
- [JavaScript 计算器](../samples/javascript/README.md)
- [TypeScript 计算器](../samples/typescript/README.md)
- [Python 计算器](../../../../03-GettingStarted/samples/python)
- [Rust 计算器](../../../../03-GettingStarted/samples/rust)

## 额外资源

## 下一步

- 下一步：[使用 Visual Studio Code 调用服务器](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：  
本文件通过 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译而成。尽管我们努力确保准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版本的文件应被视为权威来源。对于重要信息，建议采用专业人工翻译。对于因使用本翻译而产生的任何误解或错误理解，我们不承担任何责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
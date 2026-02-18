# LLMを使ったクライアントの作成

これまでに、サーバーとクライアントの作成方法を見てきました。クライアントは明示的にサーバーを呼び出して、そのツール、リソース、プロンプトの一覧を取得できました。しかし、これはあまり実用的な方法ではありません。ユーザーはエージェンシックな時代に生きており、プロンプトを使ってLLMとコミュニケーションを取りたいと期待しています。ユーザーにとって、能力をMCPに格納しているかどうかは重要ではありませんが、自然言語でインタラクションすることを望みます。では、どうやってこれを解決するのでしょうか？解決策はクライアントにLLMを追加することです。

## 概要

このレッスンでは、クライアントにLLMを追加し、ユーザーにとってより良い体験を提供する方法に焦点を当てます。

## 学習目標

このレッスンを終える頃には、以下ができるようになります：

- LLMを使ったクライアントの作成。
- LLMを用いてMCPサーバーとシームレスにやり取りを行う。
- クライアント側でより良いエンドユーザー体験を提供する。

## アプローチ

必要なアプローチを理解してみましょう。LLMの追加は単純そうに見えますが、実際にどう行うのでしょうか？

クライアントがサーバーとやり取りする流れは以下のようになります：

1. サーバーとの接続を確立する。

1. 能力、プロンプト、リソース、ツールを一覧表示し、それらのスキーマを保存する。

1. LLMを追加して、保存した能力とそのスキーマをLLMが理解できる形式で渡す。

1. ユーザーのプロンプトを受け取り、LLMに渡し、クライアントが一覧したツールも一緒に渡して処理する。

素晴らしい、これで大まかな流れが理解できました。次の演習でこれを実際に試してみましょう。

## 演習：LLMを使ったクライアントの作成

この演習では、クライアントにLLMを追加する方法を学びます。

### GitHub Personal Access Token を使った認証

GitHubトークンの作成は簡単なプロセスです。以下の手順で行えます：

- GitHubの設定へ — 右上のプロフィール写真をクリックし、[Settings]を選択。
- [Developer Settings]へ移動 — 下にスクロールして[Developer Settings]をクリック。
- [Personal Access Tokens] を選択 — [Fine-grained tokens]をクリックし、次に[Generate new token]をクリック。
- トークンの設定 — 参照用のメモを追加し、有効期限を設定し、必要なスコープ（権限）を選択します。この場合、必ずModels権限を追加してください。
- トークンの生成とコピー — [Generate token]をクリックし、トークンが表示されたらすぐにコピーしてください。二度と見ることはできません。

### -1- サーバーへ接続

まずクライアントを作成しましょう：

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // スキーマ検証のためにzodをインポートする

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

上記コードでは：

- 必要なライブラリをインポートしました
- クライアント管理用の `client` と LLMとやり取りするための `openai` の二つのメンバーを持つクラスを作成しました。
- `baseUrl` を推論API（inference API）に設定し、GitHub Modelsを使うようにLLMのインスタンスを設定しました。

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio接続のためのサーバーパラメータを作成する
server_params = StdioServerParameters(
    command="mcp",  # 実行可能ファイル
    args=["run", "server.py"],  # オプションのコマンドライン引数
    env=None,  # オプションの環境変数
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # 接続を初期化する
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

上記コードでは：

- MCP用に必要なライブラリをインポートしました
- クライアントを作成しました

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

まず、`pom.xml`ファイルにLangChain4jの依存関係を追加する必要があります。MCP統合とGitHub Modelsのサポートのために以下の依存関係を追加してください：

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

次に、Javaクライアントクラスを作成します：

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
    
    public static void main(String[] args) throws Exception {        // LLMをGitHubモデル使用に設定する
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // サーバー接続用のMCPトランスポートを作成する
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCPクライアントを作成する
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

上記コードでは：

- **LangChain4jの依存関係を追加**：MCP統合、OpenAI公式クライアント、GitHub Models対応に必要です
- **LangChain4jライブラリをインポート**：MCP統合とOpenAIチャットモデル機能のため
- **`ChatLanguageModel`を作成**：GitHubトークンでGitHub Modelsを使うよう設定
- **HTTPトランスポートを設定**：Server-Sent Events(SSE)を使ってMCPサーバーに接続
- **MCPクライアントを作成**：サーバーとの通信を担当
- **LangChain4jのMCPサポートを利用**：LLMとMCPサーバーの統合を簡単に実現

#### Rust

この例はRustベースのMCPサーバーが稼働していることを前提としています。もしまだ持っていなければ、[01-first-server](../01-first-server/README.md) のレッスンを参照してサーバーを作成してください。

Rust MCPサーバーが起動している状態で、ターミナルを開きサーバーと同じディレクトリに移動して、以下のコマンドを実行し、新しいLLMクライアントプロジェクトを作成します：

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

`Cargo.toml`ファイルに以下の依存関係を追加します：

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAIの公式Rustライブラリはありませんが、`async-openai`クレートは[コミュニティでメンテナンスされているライブラリ](https://platform.openai.com/docs/libraries/rust#rust)で一般的に使われています。

`src/main.rs`ファイルの内容を以下のコードに置き換えます：

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
    // 初期メッセージ
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAIクライアントの設定
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCPクライアントの設定
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

    // TODO: MCPツールリストを取得

    // TODO: ツール呼び出しを伴うLLM会話

    Ok(())
}
```

このコードは基本的なRustアプリケーションを設定し、MCPサーバーおよびGitHub Modelsと接続してLLM連携を行うものです。

> [!IMPORTANT]
> アプリケーションを実行する前に、必ず環境変数 `OPENAI_API_KEY` にGitHubトークンを設定してください。

素晴らしい、次のステップとしてサーバーの機能一覧を取得しましょう。

### -2- サーバーの機能一覧を取得

次に、サーバーへ接続し、能力一覧を取得します：

#### TypeScript

同じクラスに以下のメソッドを追加します：

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // ツールのリスト表示
    const toolsResult = await this.client.listTools();
}
```

上記コードでは：

- サーバー接続用の `connectToServer` メソッドを追加しました。
- アプリの処理を担当する `run` メソッドを作成。今のところツール一覧のみの表示ですが、追って機能追加します。

#### Python

```python
# 利用可能なリソースを一覧表示
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# 利用可能なツールを一覧表示
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

追加した内容：

- リソースとツールの一覧を取得し表示しました。ツールについては後で使う `inputSchema` も一覧に含めています。

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

上記コードでは：

- MCPサーバーの利用可能ツールを一覧表示しました。
- 各ツールの名前、説明、スキーマを取得しています。スキーマは後でツール呼び出しに使います。

#### Java

```java
// MCPツールを自動的に検出するツールプロバイダーを作成する
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCPツールプロバイダーは自動的に以下を処理します:
// - MCPサーバーから利用可能なツールのリストを取得
// - MCPツールのスキーマをLangChain4j形式に変換
// - ツールの実行とレスポンスの管理
```

上記コードでは：

- MCPサーバーから自動的にツールを検出・登録する `McpToolProvider` を作成しました。
- ツールプロバイダーはMCPツールのスキーマとLangChain4jのツール形式の変換を内部で処理しています。
- 手動でツールを一覧し変換する処理を抽象化しています。

#### Rust

MCPサーバーからツールを取得するには、`list_tools`メソッドを使います。`main`関数の中でMCPクライアントをセットアップした後に、以下のコードを追加してください：

```rust
// MCPツールのリストを取得する
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- サーバーの機能をLLMツールに変換

次のステップは、サーバー能力一覧をLLMが理解するフォーマットに変換することです。変換できたら、LLMにはこれらの機能をツールとして提供できます。

#### TypeScript

1. MCPサーバーからの応答をLLMが使えるツール形式に変換するコードを追加します：

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // 入力スキーマに基づいてzodスキーマを作成する
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // 型を明示的に "function" に設定する
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

    上記コードはMCPサーバーの応答を受け取り、LLMが理解できるツール定義形式に変換しています。

1. つづいて `run` メソッドを更新し、サーバーの機能を一覧表示しましょう：

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

    上記コードで `run` メソッドを更新し、結果をマップし各エントリーに `openAiToolAdapter` を呼び出しています。

#### Python

1. まずコンバーター関数を作成します：

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

    上記の `convert_to_llm_tools` 関数では、MCPツールの応答をLLMが理解できる形式に変換しています。

1. 次にクライアントコードを更新して、この関数を活用しましょう：

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    ここではMCPツール応答をLLMに渡すために、`convert_to_llm_tool` を呼び出して変換しています。

#### .NET

1. MCPツール応答をLLM向けに変換するコードを追加します：

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

上記コードでは：

- 名前、説明、入力スキーマを受け取る `ConvertFrom` 関数を作成しました。
- `FunctionDefinition` を生成し、それを `ChatCompletionsDefinition` に渡す処理を定義しました。後者はLLMが理解できるものです。

1. 既存コードをこの関数に合わせて更新する方法を見てみましょう：

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
// 自然言語での対話のためのBotインターフェースを作成する
public interface Bot {
    String chat(String prompt);
}

// LLMとMCPツールを用いてAIサービスを設定する
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

上記コードでは：

- 自然言語インタラクション用のシンプルな `Bot` インターフェースを定義しました。
- LangChain4jの `AiServices` を使い、LLMとMCPツールプロバイダーを自動的にバインドしました。
- フレームワークが内部でツールスキーマの変換と関数呼び出しを自動処理します。
- これにより手動のツール変換が不要となり、LangChain4jがMCPツールのLLM対応形式への変換の複雑さをすべて扱います。

#### Rust

MCPツール応答をLLMが理解できる形式に変換するため、ツール一覧をフォーマットするヘルパー関数を追加します。`main.rs`ファイルの `main` 関数の下に以下のコードを追加してください。LLMへのリクエスト時に呼び出されます：

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

素晴らしい、これでユーザーリクエストを処理する準備ができました。次に進みましょう。

### -4- ユーザープロンプトの処理

この部分のコードではユーザーからのリクエストを処理します。

#### TypeScript

1. LLM呼び出しに使うメソッドを追加します：

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. サーバーのツールを呼び出す
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. 結果を使って何かをする
        // TODO

        }
    }
    ```

    上記で：

    - `callTools` メソッドを追加。
    - メソッドはLLMのレスポンスを受けて、どのツールが呼び出されたかを確認します：

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // ツールを呼び出す
        }
        ```

    - LLMが呼び出すべきツールを示していれば、そのツールを呼び出します：

        ```typescript
        // 2. サーバーのツールを呼び出す
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. 結果で何かを行う
        // TODO
        ```

1. `run` メソッドを更新して、LLMの呼び出しと `callTools` を行います：

    ```typescript

    // 1. LLMへの入力となるメッセージを作成する
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLMを呼び出す
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLMの応答を確認し、各選択肢にツール呼び出しがあるかどうかをチェックする
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

さあ、コード全体を見てみましょう：

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // スキーマ検証のためにzodをインポートする

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // 将来的にこのURLに変更する必要があるかもしれません：https://models.github.ai/inference
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
          // input_schemaに基づいてzodスキーマを作成する
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // 型を明示的に「function」に設定する
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
    
    
          // 2. サーバーのツールを呼び出す
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. 結果を使って何かを行う
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
    
        // 1. LLMの応答を順に処理し、各選択肢にツールコールがあるか確認する
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

1. まずLLM呼び出しに必要なインポートを追加します：

    ```python
    # 大規模言語モデル
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. 次にLLMを呼び出す関数を追加します：

    ```python
    # 大規模言語モデル

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
            # オプションのパラメータ
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

    上記コードでは：

    - MCPサーバーで見つけた関数群をLLMに渡しています。
    - その関数を使ってLLMを呼び出しています。
    - 結果から、呼び出すべき関数があるかどうかを調べます。
    - 呼び出す関数の配列を渡しています。

1. 最後にメインコードを更新しましょう：

    ```python
    prompt = "Add 2 to 20"

    # もしあれば、どのツールをすべて使用するかLLMに尋ねる
    functions_to_call = call_llm(prompt, functions)

    # 推奨される関数を呼び出す
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    これが最終ステップです。上記コードでは：

    - プロンプトに基づきLLMが呼び出すべきと判断した関数を使ってMCPツールを呼び出しています。
    - MCPサーバーでのツール呼び出し結果を出力しています。

#### .NET

1. LLMプロンプトリクエストのコード例を見てみましょう：

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

    ここでは：

    - MCPサーバーからツールを取得しています（`var tools = await GetMcpTools()`）。
    - ユーザープロンプト `userMessage` を定義。
    - モデルとツールを指定したオプションオブジェクトを作成。
    - LLMにリクエストを送っています。

1. 最後に、LLMが関数呼び出しを必要と判断したかを確認しましょう：

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

    上記コードでは：

    - 関数呼び出しリストをループ処理。
    - 各ツール呼び出しについて名前と引数を解析し、MCPクライアントでツールを呼び出し、結果を表示しています。

以下が全体コードです：

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
    // MCPツールを自動的に使用する自然言語のリクエストを実行する
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

上記コードでは：

- 簡単な自然言語プロンプトでMCPサーバーツールとインタラクションしています
- LangChain4jフレームワークは以下を自動化しています：
  - 必要なときにユーザープロンプトをツール呼び出しへ変換
  - LLMの判断に基づき適切なMCPツールを呼び出し
  - LLMとMCPサーバー間の会話の流れを管理
- `bot.chat()` メソッドはMCPツール実行結果を含む自然言語の応答を返します
- この方法により、ユーザーはMCPの実装詳細を知らずにシームレスな体験が得られます

完全なコード例：

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

ここがメインの処理部分です。最初のユーザープロンプトでLLMを呼び出し、応答を解析して呼び出すべきツールがあれば実行し、再度LLMと対話を続けて、ツール呼び出しが不要な最終の応答が得られるまで繰り返します。

複数回LLMに問い合わせるため、それを処理する関数を定義しましょう。`main.rs`ファイルに以下の関数を追加します：

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

この関数は、LLMクライアント、メッセージ一覧（ユーザープロンプト含む）、MCPサーバーのツールを受け取り、LLMへリクエストを送り応答を返します。
LLMからの応答には `choices` の配列が含まれています。結果に `tool_calls` が存在するかどうかを処理する必要があります。これにより、LLMが特定のツールを引数付きで呼び出す要求をしていることがわかります。LLMの応答を処理する関数を定義するために、`main.rs` ファイルの一番下に次のコードを追加してください：

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

    // 利用可能な場合は内容を印刷する
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // ツールの呼び出しを処理する
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // アシスタントメッセージを追加する

        // 各ツール呼び出しを実行する
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // ツールの結果をメッセージに追加する
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ツールの結果で会話を続ける
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

`tool_calls` が存在する場合、ツールの情報を抽出し、ツールリクエストを持ってMCPサーバーを呼び出し、その結果を会話のメッセージに追加します。その後、LLMとの会話を続行し、アシスタントの応答とツールコールの結果でメッセージが更新されます。

MCPコールのためにLLMが返すツールコール情報を抽出するために、呼び出しに必要なすべてを抽出するもう一つのヘルパー関数を追加します。`main.rs` ファイルの一番下に以下のコードを追加してください：

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

すべてのパーツが揃ったので、最初のユーザープロンプトを処理しLLMを呼び出すことができます。`main` 関数を以下のコードで更新してください：

```rust
// ツール呼び出しを伴うLLMとの会話
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

これにより、初期ユーザープロンプトで2つの数の合計を求めるようLLMへ問い合わせ、応答を処理して動的にツールコールを扱います。

素晴らしい、できましたね！

## 課題

演習のコードを使って、さらに多くのツールを備えたサーバーを構築してください。その後、演習のようにLLMを使ったクライアントを作成し、さまざまなプロンプトでテストして、サーバーツールがすべて動的に呼び出されることを確認してください。このクライアント構築方法により、最終ユーザーは正確なクライアントコマンドではなくプロンプトを使い、MCPサーバーが呼び出されていることを意識せずに優れたユーザー体験を得られます。

## 解答例

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## 重要なポイント

- クライアントにLLMを加えることで、ユーザーがMCPサーバーとより良く対話できる方法を提供します。
- MCPサーバーの応答をLLMが理解できる形に変換する必要があります。

## サンプル

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## 追加リソース

## 次にやること

- 次へ: [Visual Studio Codeを用いたサーバーの利用](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：  
本書類はAI翻訳サービス[Co-op Translator](https://github.com/Azure/co-op-translator)を使用して翻訳されました。正確性には努めておりますが、自動翻訳には誤りや不正確な部分が含まれる可能性があります。原文の言語で記載された文書が正式な情報源とみなされます。重要な情報については、専門の人間による翻訳を推奨いたします。本翻訳の使用により生じたいかなる誤解や誤訳に対しても当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
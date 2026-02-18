# LLM ਨਾਲ ਇੱਕ ਕਲਾਇਂਟ ਬਣਾਉਣਾ

ਅਜੇ ਤੱਕ, ਤੁਸੀਂ ਵੇਖਿਆ ਹੈ ਕਿ ਇੱਕ ਸਰਵਰ ਅਤੇ ਇੱਕ ਕਲਾਇਂਟ ਕਿਵੇਂ ਬਣਾਇਆ ਜਾਂਦਾ ਹੈ। ਕਲਾਇਂਟ ਸਰਵਰ ਨੂੰ ਸਪੱਸ਼ਟ ਤੌਰ 'ਤੇ ਕਾਲ ਕਰ ਸਕਦਾ ਹੈ ਜਿਸ ਨਾਲ ਉਹਦੇ ਟੂਲ, ਸਰੋਤ ਅਤੇ ਪ੍ਰਾਂਪਟ ਦੀ ਸੂਚੀ ਮਿਲ ਸਕਦੀ ਹੈ। ਪਰ, ਇਹ ਬਹੁਤ ਪ੍ਰਯੋਗਿਕ ਤਰੀਕਾ ਨਹੀਂ ਹੈ। ਤੁਹਾਡਾ ਯੂਜ਼ਰ ਏਜੇਂਟਿਕ ਯੁੱਗ ਵਿਚ ਰਹਿੰਦਾ ਹੈ ਅਤੇ ਉਮੀਦ ਕਰਦਾ ਹੈ ਕਿ ਉਹ ਪ੍ਰਾਂਪਟਜ਼ ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਅਤੇ ਇੱਕ LLM ਨਾਲ ਸੰਚਾਰ ਕਰਕੇ ਆਪਣੇ ਕੰਮ ਕਰੇ। ਤੁਹਾਡੇ ਯੂਜ਼ਰ ਲਈ ਇਹ ਕੋਈ ਫਰਕ ਨਹੀਂ ਪੈਂਦਾ ਕਿ ਤੁਸੀਂ MCP ਵਰਤਦੇ ਹੋ ਜਾਂ ਨਹੀਂ, ਪਰ ਉਹ ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਦੇ ਨਾਲ ਗੱਲਬਾਤ ਕਰਨ ਦੀ ਉਮੀਦ ਰੱਖਦੇ ਹਨ। ਤਾਂ ਇਸ ਨੂੰ ਕਿਵੇਂ ਹੱਲ ਕੀਤਾ ਜਾਵੇ? ਹੱਲ ਇਹ ਹੈ ਕਿ ਇੱਕ LLM ਨੂੰ ਕਲਾਇਂਟ ਵਿੱਚ ਸ਼ਾਮਲ ਕੀਤਾ ਜਾਵੇ।

## ਝਲਕ

ਇਸ ਪਾਠ ਵਿੱਚ ਅਸੀਂ ਇੱਕ LLM ਨੂੰ ਆਪਣੇ ਕਲਾਇਂਟ ਵਿੱਚ ਸ਼ਾਮਿਲ ਕਰਨ ਤੇ ਧਿਆਨ ਕੇਂਦ੍ਰਿਤ ਕਰਾਂਗੇ ਅਤੇ ਦਿਖਾਵਾਂਗੇ ਕਿ ਇਹ ਤੁਹਾਡੇ ਯੂਜ਼ਰ ਲਈ ਬਹੁਤ ਵਧੀਆ ਅਨੁਭਵ ਕਿਵੇਂ ਪ੍ਰਦਾਨ ਕਰਦਾ ਹੈ।

## ਸਿੱਖਣ ਦੇ ਲਕੜ

ਇਸ ਪਾਠ ਦੇ ਅੰਤ ਵਿੱਚ, ਤੁਸੀਂ ਸਮਰੱਥ ਹੋਵੋਗੇ:

- ਇੱਕ LLM ਨਾਲ ਕਲਾਇਂਟ ਬਣਾਉਣਾ।
- ਇੱਕ MCP ਸਰਵਰ ਨਾਲ LLM ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਸਾਥੀ ਸੰਚਾਰ ਕਰਨਾ।
- ਕਲਾਇਂਟ ਪਾਸੇ ਇੱਕ ਵਧੀਆ ਅੰਤਿਮ ਯੂਜ਼ਰ ਅਨੁਭਵ ਪ੍ਰਦਾਨ ਕਰਨਾ।

## ਤਰੀਕਾ

ਆਓ ਸਮਝੀਏ ਕਿ ਸਾਨੂੰ ਕਿਹੜਾ ਤਰੀਕਾ ਅਪਣਾਉਣਾ ਹੈ। ਇੱਕ LLM ਸ਼ਾਮਲ ਕਰਨਾ ਸੌਖਾ ਲੱਗਦਾ ਹੈ, ਪਰ ਕੀ ਅਸੀਂ ਵਾਕਈ ਇਹ ਕਰਾਂਗੇ?

ਇਸ ਤਰ੍ਹਾਂ ਕਲਾਇਂਟ ਸਰਵਰ ਨਾਲ ਗੱਲਬਾਤ ਕਰੇਗਾ:

1. ਸਰਵਰ ਨਾਲ ਕਨੈਕਸ਼ਨ ਸਥਾਪਿਤ ਕਰੋ।

1. ਸਮਰੱਥਾਵਾਂ, ਪ੍ਰਾਂਪਟ, ਸਰੋਤ ਅਤੇ ਟੂਲਾਂ ਦੀ ਸੂਚੀ ਬਣਾਓ ਅਤੇ ਉਨ੍ਹਾਂ ਦੇ ਸਕੀਮਾ ਨੂੰ ਸੁਰੱਖਿਅਤ ਕਰੋ।

1. ਇੱਕ LLM ਸ਼ਾਮਲ ਕਰੋ ਅਤੇ ਸੰਭਾਲੇ ਹੋਏ ਸਮਰੱਥਾਵਾਂ ਅਤੇ ਉਨ੍ਹਾਂ ਦਾ ਸਕੀਮਾ LLM ਵੱਲੋਂ ਸਮਝ ਕੇ ਹੋਏ ਕਿਸੇ ਫਾਰਮੈਟ ਵਿੱਚ ਭੇਜੋ।

1. ਇਕ ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਨੂੰ ਸੰਭਾਲੋ ਜਿਸ ਵਿੱਚ ਇਹ ਪ੍ਰਾਂਪਟ LLM ਨੂੰ ਭੇਜੋ ਨਾਲ ਹੀ ਕਲਾਇਂਟ ਵੱਲੋਂ ਲਿਸਟ ਕੀਤੇ ਗਏ ਟੂਲ ਵੀ ਜੋੜੋ।

ਵਧੀਆ, ਹੁਣ ਸਾਨੂੰ ਉੱਚ ਪੱਧਰ ਤੇ ਸਮਝ ਆ ਗਿਆ ਹੈ ਕਿ ਅਸੀਂ ਇਹ ਕਿਵੇਂ ਕਰ ਸਕਦੇ ਹਾਂ, ਆਓ ਹੇਠਾਂ ਦਿੱਤੇ ਅਭਿਆਸ ਵਿੱਚ ਇਸਨੂੰ ਅਜਮਾਈਏ।

## ਅਭਿਆਸ: LLM ਨਾਲ ਇੱਕ ਕਲਾਇਂਟ ਬਣਾਉਣਾ

ਇਸ ਅਭਿਆਸ ਵਿੱਚ, ਅਸੀਂ ਸਿੱਖਾਂਗੇ ਕਿ ਆਪਣੇ ਕਲਾਇਂਟ ਵਿੱਚ ਇੱਕ LLM ਕਿਵੇਂ ਸ਼ਾਮਿਲ ਕਰਨਾ ਹੈ।

### GitHub ਨਿੱਜੀ ਪ੍ਰਵੇਸ਼ ਟੋਕਨ ਦੀ ਵਰਤੋਂ ਨਾਲ ਪ੍ਰਮਾਣੀਕਰਨ

GitHub ਟੋਕਨ ਬਣਾਉਣਾ ਇੱਕ ਸੌਖਾ ਪ੍ਰਕਿਰਿਆ ਹੈ। ਤੁਸੀਂ ਇਹ ਤਰੀਕਾ ਵਰਤ ਸਕਦੇ ਹੋ:

- GitHub ਸੈਟਿੰਗਜ਼ ਤੇ ਜਾਓ – ਸਿਰਲੇਖ ਕੋਨੇ ਉੱਤੇ ਆਪਣੀ ਪ੍ਰੋਫਾਈਲ ਤਸਵੀਰ ਤੇ ਕਲਿੱਕ ਕਰੋ ਅਤੇ ਸੈਟਿੰਗਜ਼ ਚੁਣੋ।
- ਡਿਵੈਲਪਰ ਸੈਟਿੰਗਜ਼ ਤੇ ਜਾਓ – ਹੇਠਾਂ ਸਕ੍ਰੋਲ ਕਰੋ ਅਤੇ ਡਿਵੈਲਪਰ ਸੈਟਿੰਗਜ਼ ਤੇ ਕਲਿੱਕ ਕਰੋ।
- ਨਿੱਜੀ ਪ੍ਰਵੇਸ਼ ਟੋਕਨ ਚੁਣੋ – ਫਾਈਨ-ਗਰੇਨਡ ਟੋਕਨ ਤੇ ਕਲਿੱਕ ਕਰੋ ਅਤੇ ਫਿਰ ਨਵਾਂ ਟੋਕਨ ਬਣਾਓ।
- ਆਪਣਾ ਟੋਕਨ ਕਨਫਿਗਰ ਕਰੋ – ਇੱਕ ਨੋਟ ਜੋੜੋ, ਸਮਾਪਤੀ ਤਾਰੀਖ ਸੈੱਟ ਕਰੋ, ਅਤੇ ਜਰੂਰੀ ਸਕੋਪ (ਅਧਿਕਾਰ) ਚੁਣੋ। ਇਸ ਮਾਮਲੇ ਵਿਚ ਮਾਡਲਾਂ ਦਾ ਅਧਿਕਾਰ ਜੋੜਨਾ ਯਕੀਨੀ ਬਣਾਓ।
- ਟੋਕਨ ਬਣਾਓ ਅਤੇ ਕਾਪੀ ਕਰੋ – ਜਨਰੇਟ ਟੋਕਨ 'ਤੇ ਕਲਿੱਕ ਕਰੋ ਅਤੇ ਇਸਨੂੰ ਤੁਰੰਤ ਕਾਪੀ ਕਰੋ ਕਿਉਂਕਿ ਤੁਸੀਂ ਇਕ ਵਾਰ ਹੀ ਇਸਨੂੰ ਵੇਖ ਸਕੋਗੇ।

### -1- ਸਰਵਰ ਨਾਲ ਜੁੜੋ

ਆਓ ਪਹਿਲਾਂ ਆਪਣਾ ਕਲਾਇਂਟ ਬਣਾਈਏ:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ਸਕੀਮਾ ਵੈਰੀਫਿਕੇਸ਼ਨ ਲਈ zod ਆਮਦ ਕਰੋ

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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਲੋੜੀਂਦੀਆਂ ਲਾਇਬ੍ਰੇਰੀਆਂ ਆਯਾਤ ਕੀਤੀਆਂ ਹਨ।
- ਇੱਕ ਕਲਾਸ ਬਣਾਈ ਹੈ ਜਿਸ ਵਿੱਚ ਦੋ ਮੈਂਬਰ ਹਨ, `client` ਅਤੇ `openai`, ਜੋ ਸਾਡੇ ਕਲਾਇਂਟ ਨੂੰ ਸੰਭਾਲਣ ਅਤੇ LLM ਨਾਲ ਸੰਚਾਰ ਕਰਨ ਵਿੱਚ ਮਦਦ ਕਰਨਗੇ।
- ਸਾਡੇ LLM ਉਦਾਹਰਨ ਨੂੰ GitHub ਮਾਡਲਜ਼ ਵੇਖਦਾ ਹੈ ਜਿਸ ਲਈ `baseUrl` ਨੂੰ ਇੰਫਰੈਂਸ API ਵੱਲ ਸੈਟ ਕੀਤਾ ਗਿਆ।

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio ਕਨੈਕਸ਼ਨ ਲਈ ਸਰਵਰ ਪੈਰਾਮੀਟਰ ਬਣਾਓ
server_params = StdioServerParameters(
    command="mcp",  # ਚਲਾਉਣ ਯੋਗ
    args=["run", "server.py"],  # ਵਿਕਲਪਿਕ ਕਮਾਂਡ ਲਾਈਨ ਤਰਕ
    env=None,  # ਵਿਕਲਪਿਕ ਵਾਤਾਵਰਣ ਵੈਰੀਏਬਲ
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # ਕਨੈਕਸ਼ਨ ਸ਼ੁਰੂ ਕਰੋ
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- MCP ਲਈ ਲੋੜੀਂਦੀਆਂ ਲਾਇਬ੍ਰੇਰੀਆਂ ਆਯਾਤ ਕੀਤੀਆਂ ਹਨ।
- ਇੱਕ ਕਲਾਇਂਟ ਬਣਾਇਆ ਹੈ।

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

ਸਭ ਤੋਂ ਪਹਿਲਾਂ, ਤੁਹਾਨੂੰ ਆਪਣੇ `pom.xml` ਫਾਈਲ ਵਿੱਚ LangChain4j ਨਿਰਭਰਤਾਵਾਂ ਸ਼ਾਮਿਲ ਕਰਨੀ ਪੈਣਗੀਆਂ। ਇਹ ਨਿਰਭਰਤਾਵਾਂ MCP ਇੰਟੀਗ੍ਰੇਸ਼ਨ ਅਤੇ GitHub ਮਾਡਲਜ਼ ਸਹਾਇਤਾ ਲਈ ਹਨ:

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

ਫਿਰ ਆਪਣੀ ਜਾਵਾ ਕਲਾਇਂਟ ਕਲਾਸ ਬਣਾਓ:

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
    
    public static void main(String[] args) throws Exception {        // LLM ਨੂੰ GitHub ਮਾਡਲਾਂ ਨਾਲ ਵਰਤਣ ਲਈ ਸੰਰਚਿਤ ਕਰੋ
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // ਸਰਵਰ ਨਾਲ ਜੁੜਨ ਲਈ MCP ਟ੍ਰਾਂਸਪੋਰਟ ਬਣਾਓ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP ਕਲਾਇੰਟ ਬਣਾਓ
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- **LangChain4j ਨਿਰਭਰਤਾਵਾਂ ਜੋੜੀਆਂ**: MCP ਇੰਟੀਗ੍ਰੇਸ਼ਨ, OpenAI ਆਧਿਕਾਰਕ ਕਲਾਇਂਟ ਅਤੇ GitHub ਮਾਡਲਜ਼ ਸਹਾਇਤਾ ਲਈ।
- **LangChain4j ਲਾਇਬ੍ਰੇਰੀਆਂ ਆਯਾਤ ਕੀਤੀਆਂ**: MCP ਇੰਟੀਗ੍ਰੇਸ਼ਨ ਅਤੇ OpenAI ਚੈਟ ਮਾਡਲ ਫੰਕਸ਼ਨਲਿਟੀ ਲਈ।
- **`ChatLanguageModel` ਬਣਾਈ**: GitHub ਮਾਡਲਾਂ ਦੀ ਵਰਤੋਂ ਲਈ ਤੁਹਾਡੇ GitHub ਟੋਕਨ ਦੇ ਨਾਲ ਸੰਰਚਿਤ।
- **HTTP ਟրանսਪੋਰਟ ਸੈੱਟ ਕੀਤਾ**: ਸੈਰਵਰ-ਸੈਂਟ ਈਵੈਂਟਸ (SSE) ਦੀ ਵਰਤੋਂ ਕਰਕੇ MCP ਸਰਵਰ ਨਾਲ ਕਨੈਕਸ਼ਨ ਬਣਾਉਣਾ।
- **MCP ਕਲਾਇਂਟ ਬਣਾਇਆ**: ਜੋ ਸਰਵਰ ਨਾਲ ਸੰਚਾਰ ਦਾ ਪ੍ਰਬੰਧ ਕਰਦਾ ਹੈ।
- **LangChain4j ਦੀ MCP ਸਹਾਇਤਾ ਵਰਤੀ**: ਜੋ LLM ਅਤੇ MCP ਸਰਵਰਾਂ ਵਿਚਕਾਰ ਇੰਟੀਗ੍ਰੇਸ਼ਨ ਨੂੰ ਸਰਲ ਬਨਾਉਂਦੀ ਹੈ।

#### Rust

ਇਹ ਉਦਾਹਰਨ ਇਸ ਮੰਨ ਕੇ ਚਲਦੀ ਹੈ ਕਿ ਤੁਹਾਡੇ ਕੋਲ ਇਕ ਰੱਸਟ ਆਧਾਰਤ MCP ਸਰਵਰ ਚੱਲ ਰਿਹਾ ਹੈ। ਜੇਕਰ ਤੁਹਾਡੇ ਕੋਲ ਇਕ ਨਹੀਂ ਹੈ, ਤਾਂ ਸਰਵਰ ਬਣਾਉਣ ਲਈ [01-first-server](../01-first-server/README.md) ਪਾਠ ਨੂੰ ਵੇਖੋ।

ਜਿਵੇਂ ਹੀ ਤੁਹਾਡੇ ਕੋਲ ਰੱਸਟ MCP ਸਰਵਰ ਹੈ, ਟਰਮੀਨਲ ਖੋਲ੍ਹੋ ਅਤੇ ਸਰਵਰ ਵਾਲੇ ਫੋਲਡਰ ਵਿੱਚ ਜਾਓ। ਫਿਰ ਨਵੇਂ LLM ਕਲਾਇਂਟ ਪ੍ਰੋਜੈਕਟ ਬਣਾਉਣ ਲਈ ਹੇਠਾਂ ਦਿਤਾ ਕਮਾਂਡ ਚਲਾਓ:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

ਆਪਣੇ `Cargo.toml` ਫਾਈਲ ਵਿੱਚ ਹੇਠਾਂ ਦਿੱਤੀਆਂ ਨਿਰਭਰਤਾਵਾਂ ਸ਼ਾਮਲ ਕਰੋ:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAI ਲਈ ਕੋਈ ਅਧਿਕਾਰਿਤ ਰੱਸਟ ਲਾਇਬ੍ਰੇਰੀ ਨਹੀਂ ਹੈ, ਪਰ `async-openai` ਕ੍ਰੇਟ ਇੱਕ [ਕੌਮਿਊਨਿਟੀ ਰੱਖਿਆ ਲਾਇਬ੍ਰੇਰੀ](https://platform.openai.com/docs/libraries/rust#rust) ਹੈ ਜੋ ਆਮ ਤੌਰ ਤੇ ਵਰਤੀ ਜਾਂਦੀ ਹੈ।

`src/main.rs` ਫਾਈਲ ਖੋਲ੍ਹੋ ਅਤੇ ਇਸਦਾ ਸਮੱਗਰੀ ਹਟਾ ਕੇ ਹੇਠਾਂ ਦਿੱਤਾ ਕੋਡ ਪੇਸਟ ਕਰੋ:

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
    // ਸ਼ੁਰੂਆਤੀ ਸੁਨੇਹਾ
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI ਕਲਾਇੰਟ ਸੈਟਅਪ ਕਰੋ
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP ਕਲਾਇੰਟ ਸੈਟਅਪ ਕਰੋ
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

    // TODO: MCP ਟੂਲ ਸੂਚੀ ਪ੍ਰਾਪਤ ਕਰੋ

    // TODO: ਟੂਲ ਕਾਲਾਂ ਨਾਲ LLM ਗੱਲਬਾਤ

    Ok(())
}
```

ਇਹ ਕੋਡ ਇੱਕ ਮੂਲ ਰੱਸਟ ਐਪਲੀਕੇਸ਼ਨ ਸੈੱਟ ਕਰਦਾ ਹੈ ਜੋ MCP ਸਰਵਰ ਅਤੇ GitHub ਮਾਡਲਜ਼ ਨਾਲ LLM ਇੰਟਰਐਕਸ਼ਨਸ ਲਈ ਕਨੈਕਟ ਹੋਵੇਗਾ।

> [!IMPORTANT]
> ਐਪਲੀਕੇਸ਼ਨ ਚਲਾਉਣ ਤੋਂ ਪਹਿਲਾਂ ਆਪਣੇ GitHub ਟੋਕਨ ਨਾਲ `OPENAI_API_KEY` ਐਨਵਾਇਰਨਮੈਂਟ ਵੈਰੀਏਬਲ ਸੈੱਟ ਕਰਨਾ ਯਕੀਨੀ ਬਣਾਓ।

ਅਗਲੇ ਕਦਮ ਲਈ, ਆਓ ਸਰਵਰ ਦੀਆਂ ਸਮਰੱਥਾਵਾਂ ਦੀ ਸੂਚੀ ਬਣਾਈਏ।

### -2- ਸਰਵਰ ਸਮਰੱਥਾਵਾਂ ਦੀ ਸੂਚੀ ਬਣਾਓ

ਹੁਣ ਅਸੀਂ ਸਰਵਰ ਨਾਲ ਜੁੜ ਕੇ ਇਸ ਦੀਆਂ ਸਮਰੱਥਾਵਾਂ ਦੇ ਬਾਰੇ ਪੁੱਛਾਂਗੇ:

#### TypeScript

ਉਸੇ ਕਲਾਸ ਵਿੱਚ, ਹੇਠਾਂ ਦਿੱਤੀਆਂ ਮੈਥਡਜ਼ ਸ਼ਾਮਲ ਕਰੋ:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // ਸੰਦ ਸੂਚੀਬੱਧ ਕਰਨਾ
    const toolsResult = await this.client.listTools();
}
```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਸਰਵਰ ਨਾਲ ਜੁੜਨ ਲਈ ਕੋਡ ਜੋੜਿਆ, `connectToServer`.
- ਇੱਕ `run` ਮੈਥਡ ਬਣਾਈ ਜੋ ਐਪਲੀਕੇਸ਼ਨ ਦੇ ਫਲੋ ਨੂੰ ਸੰਭਾਲਦਾ ਹੈ। ਹੁਣ ਤੱਕ ਇਹ ਸਿਰਫ ਟੂਲਾਂ ਦੀ ਸੂਚੀ ਬਨਾਉਂਦਾ ਹੈ ਪਰ ਅਸੀਂ ਬਾਅਦ ਵਿੱਚ ਹੋਰ ਜੋੜਾਂਗੇ।

#### Python

```python
# ਉਪਲਬਧ ਸਾਧਨਾਂ ਦੀ ਸੂਚੀ ਬਣਾਓ
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# ਉਪਲਬਧ ਤੰਤਰ ਦੀ ਸੂਚੀ ਬਣਾਓ
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

ਇੱਥੇ ਅਸੀਂ ਜੋ ਜੋੜਿਆ:

- ਸਰੋਤਾਂ ਅਤੇ ਟੂਲਾਂ ਨੂੰ ਲਿਸਟ ਕੀਤਾ ਅਤੇ ਪ੍ਰਿੰਟ ਕੀਤਾ। ਟੂਲਾਂ ਲਈ `inputSchema` ਵੀ ਲਿਸਟ ਕੀਤਾ ਜੋ ਅਸੀਂ ਅੱਗੇ ਵਰਤਾਂਗੇ।

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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- MCP ਸਰਵਰ 'ਤੇ ਉਪਲਬਧ ਟੂਲਾਂ ਦੀ ਸੂਚੀ ਬਣਾਈ।
- ਹਰ ਟੂਲ ਲਈ ਨਾਮ, ਵੇਰਵਾ ਅਤੇ ਉਸਦਾ ਸਕੀਮਾ ਲਿਸਟ ਕੀਤਾ। ਇਹ ਅਸੀਂ ਜਲਦੀ ਟੂਲ ਕਾਲ ਕਰਨ ਲਈ ਵਰਤਾਂਗੇ।

#### Java

```java
// ਇੱਕ ਸੰਦ ਪ੍ਰਦਾਤਾ ਬਣਾਓ ਜੋ ਸਵੈਚਾਲਿਤ ਤੌਰ 'ਤੇ MCP ਸੰਦਾਂ ਦੀ ਖੋਜ ਕਰਦਾ ਹੈ
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP ਸੰਦ ਪ੍ਰਦਾਤਾ ਸਵੈਚਾਲਿਤ ਤੌਰ 'ਤੇ ਇਹ ਸਾਰੇ ਕੰਮ ਕਰਦਾ ਹੈ:
// - MCP ਸਰਵਰ ਤੋਂ ਉਪਲਬਧ ਸੰਦਾਂ ਦੀ ਸੂਚੀ ਬਣਾਉਣਾ
// - MCP ਸੰਦਾਂ ਦੇ ਸਕੀਮਾ ਨੂੰ LangChain4j ਫਾਰਮੈਟ ਵਿੱਚ ਬਦਲਣਾ
// - ਸੰਦਾਂ ਦੇ ਚਲਾਉਣ ਅਤੇ ਜਵਾਬਾਂ ਦਾ ਪ੍ਰਬੰਧ ਕਰਨਾ
```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਇੱਕ `McpToolProvider` ਬਣਾਇਆ ਜੋ MCP ਸਰਵਰ ਤੋਂ ਸਾਰੇ ਟੂਲ ਖੋਜ ਕੇ ਰਜਿਸਟਰ ਕਰਦਾ ਹੈ।
- ਟੂਲ ਪ੍ਰਦਾਤਾ MCP ਟੂਲ ਸਕੀਮਾਂ ਅਤੇ LangChain4j ਦੇ ਟੂਲ ਫਾਰਮੈਟ ਵਿਚਕਾਰ ਕਨਵਰਸ਼ਨ ਨੂੰ ਅੰਦਰੂਨੀ ਤੌਰ 'ਤੇ ਸੰਭਾਲਦਾ ਹੈ।
- ਇਹ ਤਰੀਕਾ ਮੈਨੂਅਲ ਟੂਲ ਲਿਸਟਿੰਗ ਅਤੇ ਕਨਵਰਸ਼ਨ ਦੀ ਲੋੜ ਨੂੰ ਹਟਾ ਦਿੰਦਾ ਹੈ।

#### Rust

MCP ਸਰਵਰ ਤੋਂ ਟੂਲ ਪ੍ਰਾਪਤ ਕਰਨ ਲਈ `list_tools` ਮੈਥਡ ਵਰਤੀ ਜਾਂਦੀ ਹੈ। ਆਪਣੇ `main` ਫੰਕਸ਼ਨ ਵਿੱਚ MCP ਕਲਾਇਂਟ ਸੈੱਟਅਪ ਤੋਂ ਬਾਅਦ ਹੇਠਾਂ ਦਿੱਤਾ ਕੋਡ ਜੋੜੋ:

```rust
// MCP ਟੂਲ ਸੂਚੀ ਪ੍ਰਾਪਤ ਕਰੋ
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- ਸਰਵਰ ਸਮਰੱਥਾਵਾਂ ਨੂੰ LLM ਟੂਲਾਂ ਵਿੱਚ ਬਦਲੋ

ਅਗਲਾ ਕਦਮ ਸਰਵਰ ਦੀਆਂ ਸਮਰੱਥਾਵਾਂ ਨੂੰ ਉਸ ਫਾਰਮੈਟ ਵਿੱਚ ਬਦਲਣਾ ਹੈ ਜੋ LLM ਨੂੰ ਸਮਝ ਆਵੇ। ਇਸ ਤੋਂ ਬਾਅਦ ਅਸੀਂ ਇਹ ਸਮਰੱਥਾਵਾਂ ਆਪਣੇ LLM ਨੂੰ ਟੂਲਾਂ ਵਜੋਂ ਦੇ ਸਕਦੇ ਹਾਂ।

#### TypeScript

1. MCP ਸਰਵਰ ਤੋਂ ਪ੍ਰਾਪਤ ਜਵਾਬ ਨੂੰ LLM ਲਈ ਸੂਝਵਾਨ ਟੂਲ ਫਾਰਮੈਟ ਵਿੱਚ ਬਦਲਣ ਲਈ ਹੇਠਾਂ ਦਿੱਤਾ ਕੋਡ ਜੋੜੋ:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ਇਨਪੁਟ_ਸਕੀਮਾ ਦੇ ਆਧਾਰ 'ਤੇ ਇੱਕ ਜੋਡ ਸਕੀਮਾ ਬਣਾਓ
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // ਕਿਸ਼ੋਰੀ ਤੌਰ 'ਤੇ ਕਿਸਮ "ਫੰਕਸ਼ਨ" ਸੈਟ ਕਰੋ
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

    ਉਪਰੋਕਤ ਕੋਡ MCP ਸਰਵਰ ਤੋਂ ਪ੍ਰਾਪਤ ਜਵਾਬ ਲੈ ਕੇ ਇਸਨੂੰ LLM ਲਈ ਸਮਝ ਆਉਣ ਵਾਲੇ ਟੂਲ ਪਰਿਭਾਸ਼ਾ ਵਿੱਚ ਬਦਲਦਾ ਹੈ।

1. ਹੁਣ `run` ਮੈਥਡ ਨੂੰ ਅੱਪਡੇਟ ਕਰੀਏ ਤਾਂ ਜੋ ਸਰਵਰ ਸਮਰੱਥਾਵਾਂ ਦੀ ਸੂਚੀ ਪ੍ਰਦਾਨ ਕਰੇ:

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

    ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ, ਅਸੀਂ `run` ਮੈਥਡ ਨੂੰ ਅਪਡੇਟ ਕੀਤਾ ਹੈ ਜੋ ਪ੍ਰਤੀਕ੍ਰਿਆ 'ਚੋਂ ਹਰ ਐਂਟਰੀ ਨੂੰ `openAiToolAdapter` ਕਾਲ ਕਰਦਾ ਹੈ।

#### Python

1. ਪਹਿਲਾਂ, ਆਓ ਹੇਠਾਂ ਦਿੱਤਾ ਕਨਵਰਟਰ ਫੰਕਸ਼ਨ ਬਣਾਈਏ

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

    ਉਪਰ `convert_to_llm_tools` ਫੰਕਸ਼ਨ ਵਿੱਚ ਅਸੀਂ MCP ਟੂਲ ਪ੍ਰਤੀਕ੍ਰਿਆ ਨੂੰ LLM ਲਈ ਸਮਝ ਆਉਣ ਵਾਲੇ ਫਾਰਮੈਟ ਵਿੱਚ ਬਦਲਦੇ ਹਾਂ।

1. ਫਿਰ, ਆਪਣਾ ਕਲਾਇਂਟ ਕੋਡ ਇਸ ਫੰਕਸ਼ਨ ਦੀ ਵਰਤੋਂ ਕਰਨ ਲਈ ਅਪਡੇਟ ਕਰੋ:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    ਇੱਥੇ ਅਸੀਂ `convert_to_llm_tool` ਕਾਲ ਕਰਦੇ ਹਾਂ ਜੋ MCP ਟੂਲ ਪ੍ਰਤੀਕ੍ਰਿਆ ਨੂੰ LLM ਫੀਡ ਕਰਨ ਯੋਗ ਬਣਾਉਂਦਾ ਹੈ।

#### .NET

1. MCP ਟੂਲ ਪ੍ਰਤੀਕ੍ਰਿਆ ਨੂੰ LLM ਲਈ ਸਮਝਣਯੋਗ ਬਣਾਉਣ ਲਈ ਕੋਡ ਜੋੜੋ:

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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਇੱਕ `ConvertFrom` ਫੰਕਸ਼ਨ ਬਣਾਇਆ ਜੋ ਨਾਮ, ਵੇਰਵਾ ਅਤੇ ਇਨਪੁੱਟ ਸਕੀਮਾ ਲੈਂਦਾ ਹੈ।
- ਫੰਕਸ਼ਨਜ ਮਿਆਦ ਦੀ ਪਰਿਭਾਸ਼ਾ ਜੋ ChatCompletionsDefinition ਨੂੰ ਪਾਸ ਕੀਤੀ ਜਾਂਦੀ ਹੈ, ਜਿਸਨੂੰ LLM ਸਮਝਦਾ ਹੈ।

1. ਮੁਜੂਦਾ ਕੋਡ ਨੂੰ ਇਸ ਫੰਕਸ਼ਨ ਦੀ ਵਰਤੋਂ ਕਰਨ ਲਈ ਕਿਵੇਂ ਅਪਡੇਟ ਕਰੀਏ:

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
// ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਅੰਤਰਕ੍ਰਿਆ ਲਈ ਬੋਟ ਇੰਟਰਫੇਸ ਬਣਾਓ
public interface Bot {
    String chat(String prompt);
}

// LLM ਅਤੇ MCP ਸੰਦਾਂ ਨਾਲ AI ਸੇਵਾ ਨੂੰ ਸੰਰਚਿਤ ਕਰੋ
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਅੰਤਰਕ੍ਰਿਆ ਲਈ ਇੱਕ ਸਧਾਰਨ `Bot` ਇੰਟਰਫੇਸ ਬਣਾਇਆ।
- LangChain4j ਦੀ `AiServices` ਵਰਤੀ ਜੋ ਆਟੋਮੈਟੀਕ ਤੌਰ 'ਤੇ LLM ਨੂੰ MCP ਟੂਲ ਪ੍ਰਦਾਤਾ ਨਾਲ ਜੁੜਾ ਦਿੰਦੀ ਹੈ।
- ਫ੍ਰੇਮਵਰਕ ਮੈਨੂਅਲ ਟੂਲ ਕਨਵਰਸ਼ਨ ਅਤੇ ਫੰਕਸ਼ਨ ਕਾਲਿੰਗ ਦਾ ਪ੍ਰਬੰਧ ਪਿਛੋਕੜ ਵਿੱਚ ਕਰਦਾ ਹੈ।
- ਇਸ ਤਰੀਕੇ ਨਾਲ MCP ਟੂਲਾਂ ਨੂੰ LLM-ਅਨੁਕੂਲ ਫਾਰਮੈਟ ਵਿੱਚ ਬਦਲਣ ਦੀ ਖਤਮ ਹੁੰਦੀ ਹੈ।

#### Rust

MCP ਟੂਲ ਪ੍ਰਤੀਕ੍ਰਿਆ ਨੂੰ LLM ਲਈ ਸਮਝਣਯੋਗ ਬਣਾਉਣ ਲਈ ਸਹਾਇਕ ਫੰਕਸ਼ਨ ਜੋੜਾਂਗੇ ਜੋ ਟੂਲਾਂ ਦੀ ਲਿਸਟਿੰਗ ਨੂੰ ਫਾਰਮੈਟ ਕਰੇਗਾ। `main.rs` ਫਾਇਲ ਵਿਚ `main` ਫੰਕਸ਼ਨ ਦੇ ਥੱਲੇ ਹੇਠਾਂ ਦਿੱਤਾ ਕੋਡ ਸ਼ਾਮਿਲ ਕਰੋ। ਇਹ LLM ਨੂੰ ਬੇਨਤੀ ਕਰਨ ਵੇਲੇ ਕਾਲ ਕੀਤਾ ਜਾਵੇਗਾ:

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

ਵਧੀਆ, ਹੁਣ ਅਸੀਂ ਯੂਜ਼ਰ ਦੀਆਂ ਮੰਗਾਂ ਨੂੰ ਸੰਭਾਲਨ ਲਈ ਤਿਆਰ ਹਾਂ।

### -4- ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਅਰਜ਼ੀ ਸੰਭਾਲੋ

ਇਸ ਕੋਡ ਹਿੱਸੇ ਵਿੱਚ, ਅਸੀਂ ਯੂਜ਼ਰ ਦੀਆਂ ਮੰਗਾਂ ਨੂੰ ਸੰਭਾਲਾਂਗੇ।

#### TypeScript

1. ਇੱਕ ਮੈਥਡ ਜੋ LLM ਨੂੰ ਕਾਲ ਕਰਨ ਵਿੱਚ ਵਰਤੀ ਜਾਵੇਗੀ ਜੋੜੋ:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. ਸਰਵਰ ਦੇ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰੋ
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ਨਤੀਜੇ ਨਾਲ ਕੁਝ ਕਰੋ
        // ਕਰਨਾ ਬਾਕੀ ਹੈ

        }
    }
    ```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਇੱਕ ਮੈਥਡ `callTools` ਜੋੜੀ।
- ਇਹ ਮੈਥਡ LLM ਦੀ ਪ੍ਰਤੀਕ੍ਰਿਆ ਲੈਂਦੀ ਹੈ ਅਤੇ ਜਾਂਚਦੀ ਹੈ ਕਿ ਕਿਸੇ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਦੀ ਲੋੜ ਹੈ ਜਾਂ ਨਹੀਂ:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // ਟੂਲ ਨੂੰ ਕਾਲ ਕਰੋ
        }
        ```

- ਜੇ LLM ਦੱਸਦਾ ਹੈ ਕਿ ਕਿਸੇ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨਾ ਹੈ ਤਾਂ ਕਾਲ ਕੀਤੀ ਜਾਂਦੀ ਹੈ:

        ```typescript
        // 2. ਸਰਵਰ ਦੇ ਟੂਲ ਨੂੰ ਕੌਲ ਕਰੋ
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ਨਤੀਜੇ ਨਾਲ ਕੁਝ ਕਰੋ
        // ਕਰਨ ਲਈ
        ```

1. `run` ਮੈਥਡ ਨੂੰ ਅਪਡੇਟ ਕਰੋ ਤਾਂ ਜੋ LLM ਕਾਲ ਅਤੇ `callTools` ਨੂੰ ਸ਼ਾਮਿਲ ਕਰੇ:

    ```typescript

    // 1. ਸੁਨੇਹੇ ਬਣਾਓ ਜੋ LLM ਲਈ ਇਨਪੁੱਟ ਹਨ
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM ਨੂੰ ਕਾਲ ਕਰਨਾ
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM ਜਵਾਬ ਵਿੱਚੋਂ ਗੁਜ਼ਰੋ, ਹਰ ਵਿਕਲਪ ਲਈ, ਚੈੱਕ ਕਰੋ ਕਿ ਕੀ ਇਸ ਵਿੱਚ ਟੂਲ ਕਾਲ ਹੁੰਦੇ ਹਨ
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

ਵਧੀਆ, ਅਸੀਂ ਪੂਰਾ ਕੋਡ ਹੇਠਾਂ ਦਿੱਤਾ ਹੈ:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ਸਕੀਮਾ ਵੈਧਤਾ ਲਈ ਜ਼ੋਡ ਆਯਾਤ ਕਰੋ

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // ਭਵਿੱਖ ਵਿੱਚ ਇਹ URL ਬਦਲਣ ਦੀ ਲੋੜ ਹੋ ਸਕਦੀ ਹੈ: https://models.github.ai/inference
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
          // ਇਨਪੁੱਟ_ਸਕੀਮਾ ਦੇ ਆਧਾਰ 'ਤੇ ਇੱਕ ਜ਼ੋਡ ਸਕੀਮਾ ਬਣਾਓ
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // ਕਿਸਮ ਨੂੰ "ਫੰਕਸ਼ਨ" ਵਜੋਂ ਸਪਸ਼ਟ ਤੌਰ 'ਤੇ ਸੈੱਟ ਕਰੋ
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
    
    
          // 2. ਸਰਵਰ ਦੇ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰੋ
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. ਨਤੀਜਾ ਨਾਲ ਕੁਝ ਕਰੋ
          // ਕਰਨ ਦੀ ਲੋੜ ਹੈ
    
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
    
        // 1. LLM ਜਵਾਬ ਵਿੱਚੋਂ ਹਰ ਚੋਣ ਨੂੰ ਵੇਖੋ, ਚੈੱਕ ਕਰੋ ਕਿ ਕੀ ਇਸ ਵਿੱਚ ਟੂਲ ਕਾਲ ਹਨ
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

1. LLM ਕਾਲ ਕਰਨ ਲਈ ਕੁਝ ਲੋੜੀਂਦੇ ਇੰਪੋਰਟ ਜੋੜੋ

    ```python
    # ਐਲ ਐਲ ਐਮ
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. ਹੁਣ ਉਹ ਫੰਕਸ਼ਨ ਜੋ LLM ਨੂੰ ਕਾਲ ਕਰੇਗੀ ਜੋੜੋ:

    ```python
    # ਏਲਐਲਐਮ

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
            # ਵਿਕਲਪੀ ਪੈਰਾਮੀਟਰ
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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- MCP ਸਰਵਰ ਤੋਂ ਮਿਲੇ ਫੰਕਸ਼ਨਾਂ ਨੂੰ LLM ਨੂੰ ਦਿੱਤਾ।
- ਫਿਰ LLM ਨੂੰ ਉਹਨਾਂ ਫੰਕਸ਼ਨਾਂ ਨਾਲ ਕਾਲ ਕੀਤਾ।
- ਨਤੀਜਾ ਚੈੱਕ ਕੀਤਾ ਕਿ ਕਿਸੇ ਫੰਕਸ਼ਨ ਨੂੰ ਕਾਲ ਕਰਨ ਦੀ ਲੋੜ ਹੈ ਜਾਂ ਨਹੀਂ।
- ਆਖ਼ਰੀ ਵਿੱਚ ਕਾਲ ਕਰਨ ਲਈ ਫੰਕਸ਼ਨਾਂ ਦੀ ਲਿਸਟ ਦਿੱਤੀ।

1. ਅੰਤ ਵਿੱਚ ਆਪਣਾ ਮੁੱਖ ਕੋਡ ਅਪਡੇਟ ਕਰੋ:

    ```python
    prompt = "Add 2 to 20"

    # LLM ਨੂੰ ਪੁੱਛੋ ਕਿ ਸਾਰੇ ਟੂਲ ਕਿਹੜੇ ਹਨ, ਜੇ ਕੋਈ ਹੋਣ
    functions_to_call = call_llm(prompt, functions)

    # ਸੁਝਾਏ ਗਏ ਫੰਕਸ਼ਨ ਨੂੰ ਕਾਲ ਕਰੋ
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- LLM ਦੇ ਪ੍ਰਾਂਪਟ ਅਨੁਸਾਰ MCP ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਕੇ ਨਤੀਜੇ ਪ੍ਰਿੰਟ ਕੀਤੇ।

#### .NET

1. LLM ਪ੍ਰਾਂਪਟ ਲਈ ਇਕ ਛੋਟਾ ਕੋਡ ਦਿਖਾਈਏ:

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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- MCP ਸਰਵਰ ਤੋਂ ਟੂਲ ਲਏ, `var tools = await GetMcpTools()`।
- ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ `userMessage` ਬਣਾਇਆ।
- ਮਾਡਲ ਅਤੇ ਟੂਲ ਦੱਸ ਕੇ ਵਿਕਲਪ ਬਣਾਇਆ।
- LLM ਵੱਲ ਬੇਨਤੀ ਭੇਜੀ।

1. ਆਖ਼ਰੀ ਕਦਮ, ਦੇਖੀਏ LLM ਕੀ ਸੋਚਦਾ ਹੈ ਕਿ ਕਿਸੇ ਫੰਕਸ਼ਨ ਨੂੰ ਕਾਲ ਕਰਨਾ ਚਾਹੀਦਾ ਹੈ ਜਾਂ ਨਹੀਂ:

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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- ਫੰਕਸ਼ਨਾਂ ਦੀ ਸੂਚੀ ਵਿੱਚ ਲੂਪ ਮਾਰਿਆ।
- ਹਰ ਟੂਲ ਕਾਲ ਲਈ ਨਾਮ ਅਤੇ ਆਰਗੁਮੈਂਟ ਪਰਸਤ ਕੀਤਾ ਅਤੇ MCP ਸਰਵਰ ਤੇ ਟੂਲ ਕਾਲ ਕੀਤਾ।
- ਫਲ ਪ੍ਰਿੰਟ ਕੀਤਾ।

ਪੂਰਾ ਕੋਡ ਇੱਥੇ ਹੈ:

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
    // ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਬੇਨਤੀਆਂ ਨੂੰ ਚਲਾਓ ਜੋ ਆਪਮੈ MCP ਟੂਲਜ਼ ਨੂੰ ਵਰਤਦੀਆਂ ਹਨ
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

ਉਪਰ ਦਿੱਤੇ ਕੋਡ ਵਿੱਚ ਅਸੀਂ:

- MCP ਸਰਵਰ ਟੂਲਾਂ ਨਾਲ ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਪ੍ਰਾਂਪਟ ਦੁਆਰਾ ਸੰਚਾਰ ਕੀਤਾ।
- LangChain4j ਫ੍ਰੇਮਵਰਕ ਆਟੋਮੈਟਿਕ ਤੌਰ 'ਤੇ ਸੰਭਾਲਦਾ ਹੈ:
  - ਜਦੋਂ ਲੋੜ ਹੋਵੇ ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਤੋਂ ਟੂਲ ਕਾਲ ਵਿੱਚ ਬਦਲਣਾ
  - LLM ਫੈਸਲੇ ਅਨੁਸਾਰ MCP ਟੂਲ ਕਾਲ ਕਰਨਾ
  - LLM ਅਤੇ MCP ਸਰਵਰ ਵਿਚਾਲੇ ਗੱਲਬਾਤ ਦਾ ਪ੍ਰਬੰਧ
- `bot.chat()` ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਵਿੱਚ ਨਤੀਜਾ ਦਿੰਦਾ ਹੈ ਜਿਸ ਵਿੱਚ MCP ਟੂਲਜ਼ ਕਾਰਜਕੇਸ਼ਨ ਦੇ ਨਤੀਜੇ ਹੋ ਸਕਦੇ ਹਨ
- ਇਹ ਤਰੀਕਾ ਇੱਕ ਨਰਮਾਲ ਯੂਜ਼ਰ ਅਨੁਭਵ ਦਿੰਦਾ ਹੈ ਜਿਸ ਵਿੱਚ ਯੂਜ਼ਰ MCP ਤੋਂ ਵਿਗਿਆਨਿਕ ਜਾਣਕਾਰੀ ਜਾਣਨ ਦੀ ਲੋੜ ਨਹੀਂ।

ਪੂਰਾ ਕੋਡ ਉਦਾਹਰਨ:

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

ਇੱਥੇ ਮੁੱਖ ਭਾਗ ਹੁੰਦਾ ਹੈ। ਅਸੀਂ ਸ਼ੁਰੂਆਤੀ ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਨਾਲ LLM ਨੂੰ ਕਾਲ ਕਰਾਂਗੇ, ਫਿਰ ਪ੍ਰਤੀਕ੍ਰਿਆ ਨੂੰ ਪ੍ਰੋਸੈਸ ਕਰ ਕੇ ਦੇਖਾਂਗੇ ਕਿ ਕਿਸੇ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਦੀ ਲੋੜ ਹੈ ਜਾਂ ਨਹੀਂ। ਜੇ ਹਾਂ, ਤਾਂ ਉਹਨਾਂ ਟੂਲਾਂ ਨੂੰ ਕਾਲ ਕਰਾਂਗੇ ਅਤੇ ਗੱਲਬਾਤ ਨੂੰ LLM ਨਾਲ ਜਾਰੀ ਰੱਖਾਂਗੇ ਜਦ ਤਕ ਹੋਰ ਟੂਲ ਕਾਲਾਂ ਦੀ ਲੋੜ ਨਾਂ ਹੋਵੇ ਅਤੇ ਅਸੀਂ ਅੰਤਿਮ ਨਤੀਜਾ ਪ੍ਰਾਪਤ ਨਾ ਕਰ ਲਵਾਂ।

ਅਸੀਂ LLM ਨੂੰ ਕਈ ਵਾਰੀ ਕਾਲ ਕਰਾਂਗੇ, ਇਸ ਲਈ ਇੱਕ ਫੰਕਸ਼ਨ ਬਣਾਈਏ ਜੋ LLM ਕਾਲ ਨੂੰ ਸੰਭਾਲੇ। ਤੁਹਾਡੇ `main.rs` ਫਾਈਲ ਵਿੱਚ ਹੇਠਾਂ ਦਿੱਤਾ ਫੰਕਸ਼ਨ ਸ਼ਾਮਿਲ ਕਰੋ:

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

ਇਹ ਫੰਕਸ਼ਨ LLM ਕਲਾਇਂਟ, ਮੇਸਜ਼ ਦੀ ਲਿਸਟ (ਜਿਸ ਵਿੱਚ ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਵੀ ਸ਼ਾਮਲ ਹੈ), MCP ਸਰਵਰ ਤੋਂ ਟੂਲਾਂ ਨੂੰ ਲੈਂਦਾ ਹੈ ਅਤੇ LLM ਨੂੰ ਬੇਨਤੀ ਭੇਜਦਾ ਹੈ, ਫਿਰ ਪ੍ਰਤੀਕ੍ਰਿਆ ਵਾਪਸ ਕਰਦਾ ਹੈ।
LLM ਤੋਂ ਪ੍ਰਾਪਤ ਜਵਾਬ ਵਿੱਚ `choices` ਦਾ ਇੱਕ ਐਰੇ ਹੋਵੇਗਾ। ਸਾਨੂੰ ਨਤੀਜੇ ਨੂੰ ਪ੍ਰਕਿਰਿਆ ਕਰਨ ਦੀ ਲੋੜ ਹੈ ਤਾਂ ਕਿ ਦੇਖਿਆ ਜਾ ਸਕੇ ਕਿ ਕੋਈ `tool_calls` ਮੌਜੂਦ ਹਨ ਕਿ ਨਹੀਂ। ਇਹ ਸਾਨੂੰ ਦੱਸਦਾ ਹੈ ਕਿ LLM ਕਿਸੇ ਵਿਸ਼ੇਸ਼ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਦੀ ਮੰਗ ਕਰ ਰਿਹਾ ਹੈ ਜਿਸਦੇ ਨਾਲ ਤਰਕ ਦਿੱਤੇ ਜਾਣਗੇ। LLM ਜਵਾਬ ਨੂੰ ਸੰਭਾਲਣ ਲਈ ਇੱਕ ਫੰਕਸ਼ਨ ਨੂੰ ਪਰਿਭਾਸ਼ਿਤ ਕਰਨ ਲਈ ਆਪਣੀ `main.rs` ਫਾਈਲ ਦੇ ਨੀਵੇਂ ਹਿੱਸੇ ਵਿੱਚ ਹੇਠਾਂ ਦਿੱਤੇ ਕੋਡ ਨੂੰ ਜੋੜੋ:

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

    // ਸਮੱਗਰੀ ਉਪਲਬਧ ਹੋਣ 'ਤੇ ਪ੍ਰਿੰਟ ਕਰੋ
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // ਟੂਲ ਕਾਲਾਂ ਨੂੰ ਹੈਂਡਲ ਕਰੋ
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // ਸਹਾਇਕ ਸੁਨੇਹਾ ਜੋੜੋ

        // ਹਰ ਇੱਕ ਟੂਲ ਕਾਲ ਨੂੰ ਚਲਾਓ
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // ਟੂਲ ਨਤੀਜੇ ਨੂੰ ਸੁਨੇਹੇ ਵਿੱਚ ਸ਼ਾਮਿਲ ਕਰੋ
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ਟੂਲ ਨਤੀਜਿਆਂ ਨਾਲ ਗੱਲਬਾਤ ਜਾਰੀ ਰੱਖੋ
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

ਜੇ `tool_calls` ਮੌਜੂਦ ਹਨ, ਤਾਂ ਇਹ ਟੂਲ ਜਾਣਕਾਰੀ ਨੂੰ ਨਿਕਾਲਦਾ ਹੈ, MCP ਸਰਵਰ ਨੂੰ ਟੂਲ ਬੇਨਤੀ ਦੇ ਨਾਲ ਕਾਲ ਕਰਦਾ ਹੈ, ਅਤੇ ਨਤੀਜੇ ਗੱਲਬਾਤ ਦੇ ਸੁਨੇਹਿਆਂ ਵਿੱਚ ਜੋੜਦਾ ਹੈ। ਫਿਰ ਇਹ LLM ਨਾਲ ਗੱਲਬਾਤ ਜਾਰੀ ਰੱਖਦਾ ਹੈ ਅਤੇ ਸੁਨੇਹੇ ਸਹਾਇਕ ਦੇ ਜਵਾਬ ਅਤੇ ਟੂਲ ਕਾਲ ਨਤੀਜਿਆਂ ਨਾਲ ਅਪਡੇਟ ਕੀਤੇ ਜਾਂਦੇ ਹਨ।

MCP ਕਾਲਾਂ ਲਈ LLM ਵਲੋਂ ਵਾਪਸ ਕੀਤੀ ਟੂਲ ਕਾਲ ਜਾਣਕਾਰੀ ਨੂੰ ਨਿਕਾਲਣ ਲਈ, ਅਸੀਂ ਇੱਕ ਹੋਰ ਸਹਾਇਕ ਫੰਕਸ਼ਨ ਜੋੜਾਂਗੇ ਜੋ ਕਾਲ ਕਰਨ ਲਈ ਸਾਰੀਆਂ ਲੋੜੀਂਦੀਆਂ ਚੀਜ਼ਾਂ ਨੂੰ ਕੱਢਦਾ ਹੈ। ਆਪਣੀ `main.rs` ਫਾਈਲ ਦੇ ਨੀਵੇਂ ਹਿੱਸੇ ਵਿੱਚ ਹੇਠਾਂ ਦਿੱਤਾ ਕੋਡ ਸ਼ਾਮਲ ਕਰੋ:

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

ਸਭ ਟੁਕੜੇ ਆਪਣੀ ਜਗ੍ਹਾ ਤੇ ਹਨ, ਹੁਣ ਅਸੀਂ ਮੁੱਢਲੇ ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਨੂੰ ਸੰਭਾਲ ਸਕਦੇ ਹਾਂ ਅਤੇ LLM ਨੂੰ ਕਾਲ ਕਰ ਸਕਦੇ ਹਾਂ। ਆਪਣੇ `main` ਫੰਕਸ਼ਨ ਨੂੰ ਹੇਠਾਂ ਦਿੱਤੇ ਕੋਡ ਨਾਲ ਅੱਪਡੇਟ ਕਰੋ:

```rust
// ਟੂਲ ਕਾਲਾਂ ਨਾਲ LLM ਗੱਲਬਾਤ
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

ਇਹ ਸ਼ੁਰੂਆਤੀ ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ ਦੇ ਨਾਲ LLM ਨੂੰ ਕੁਐਰੀ ਕਰੇਗਾ ਜੋ ਦੋ ਨੰਬਰਾਂ ਦੇ ਜੋੜ ਲਈ ਪੁੱਛਦਾ ਹੈ, ਅਤੇ ਜਵਾਬ ਨੂੰ ਪ੍ਰਕਿਰਿਆ ਕਰੇਗਾ ਤਾਂ ਕਿ ਟੂਲ ਕਾਲਾਂ ਨੂੰ ਗਤੀਸ਼ੀਲ ਢੰਗ ਨਾਲ ਸੰਭਾਲਿਆ ਜਾ ਸਕੇ।

ਵਧੀਆ, ਤੁਸੀਂ ਕਰ ਲਿਆ!

## ਅਸਾਈਨਮੈਂਟ

ਐਕਸਰਸਾਈਜ਼ ਤੋਂ ਕੋਡ ਲੈ ਕੇ ਸਰਵਰ ਵਿੱਚ ਹੋਰ ਟੂਲਜ਼ ਸ਼ਾਮਲ ਕਰੋ। ਫਿਰ LLM ਨਾਲ ਇੱਕ ਕਲਾਇੰਟ ਬਣਾਓ, ਐਕਸਰਸਾਈਜ਼ ਵਾਂਗ, ਅਤੇ ਵੱਖ-ਵੱਖ ਪ੍ਰਾਂਪਟ ਨਾਲ ਇਸਦੀ ਜਾਂਚ ਕਰੋ ਤਾਂ ਕਿ ਤੁਹਾਡੇ ਸਾਰੇ ਸਰਵਰ ਟੂਲਜ਼ ਗਤੀਸ਼ੀਲ ਤੌਰ 'ਤੇ ਕਾਲ ਹੋਣ। ਇਸ ਤਰ੍ਹਾਂ ਕਲਾਇੰਟ ਬਣਾਉਣ ਨਾਲ ਅੰਤਿਮ ਯੂਜ਼ਰ ਨੂੰ ਵਧੀਆ ਅਨੁਭਵ ਮਿਲਦਾ ਹੈ ਕਿਉਂਕਿ ਉਹ ਪ੍ਰਾਂਪਟ ਵਰਤ ਸਕਦੇ ਹਨ ਨਾ ਕਿ ਸਟੀਕ ਕਲਾਇੰਟ ਹੁਕਮਾਂ ਦਾ, ਅਤੇ MCP ਸਰਵਰ ਕਾਲ ਹੋ ਰਹੀ ਹੋਣ ਦੇ ਬਾਵਜੂਦ ਅਣਜਾਣ ਰਹਿੰਦਾ ਹੈ।

## ਸਮਾਧਾਨ

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## ਮੁੱਖ ਨੁਕਤੇ

- ਆਪਣੇ ਕਲਾਇੰਟ ਵਿੱਚ LLM ਸ਼ਾਮਲ ਕਰਨਾ MCP ਸਰਵਰਜ਼ ਨਾਲ ਯੂਜ਼ਰਜ਼ ਦੇ ਵਧੀਆ ਇੰਟਰੈਕਸ਼ਨ ਦਾ ਤਰੀਕਾ ਦੇਂਦਾ ਹੈ।
- ਤੁਹਾਨੂੰ MCP ਸਰਵਰ ਜਵਾਬ ਨੂੰ ਐਸਾ ਬਦਲਣਾ ਪੈਂਦਾ ਹੈ ਜੋ LLM ਸਮਝ ਸਕੇ।

## ਨਮੂਨੇ

- [Java Calculator](../samples/java/calculator/README.md)
- [.Net Calculator](../../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](../samples/javascript/README.md)
- [TypeScript Calculator](../samples/typescript/README.md)
- [Python Calculator](../../../../03-GettingStarted/samples/python)
- [Rust Calculator](../../../../03-GettingStarted/samples/rust)

## ਵਾਧੂ ਸਾਂਸਾਧਨ

## ਅੱਗੇ ਕੀ

- ਅਗਲਾ: [Visual Studio Code ਨੂੰ ਵਰਤ ਕੇ ਸਰਵਰ ਦੀ ਖਪਤ](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰਨ**:  
ਇਹ ਦਸਤਾਵੇਜ਼ AI ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਅਨੁਵਾਦ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸ਼ੁੱਧਤਾ ਲਈ ਯਤਨ ਕਰਦੇ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਇਹ ਜਾਣਕਾਰੀ ਰੱਖੋ ਕਿ ਸੁਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਤੀਤਵ ਹੋ ਸਕਦੇ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੇ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਿਤ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜ਼ਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਿਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਪ੍ਰਯੋਗ ਤੋਂ ਉਪਜੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆ ਲਈ ਜ਼ਿੰਮੇਵਾਰ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
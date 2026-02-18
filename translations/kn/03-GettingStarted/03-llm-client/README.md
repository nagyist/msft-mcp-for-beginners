# LLM ನೊಂದಿಗೆ ಕ್ಲೈಂಟ್ ರಚನೆ

ಇನ್ನಷ್ಟು ನೀವು ಸರ್ವರ್ ಮತ್ತು ಕ್ಲೈಂಟ್ ಅನ್ನು ರಚಿಸುವ ವಿಧಾನವನ್ನು ನೋಡಿದ್ದೀರಿ. ಕ್ಲೈಂಟ್ ಸ್ಪಷ್ಟವಾಗಿ ಸರ್ವರ್ ಅನ್ನು ಕರೆಸಿ ಅದರ ಸಾಧನಗಳು, ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಪ್ರಾಂಪ್ಟ್ಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಬಹುದು. ಆದಾಗ್ಯೂ, ಇದು ತುಂಬಾ ಪ್ರಾಯೋಗಿಕ ಮೋಡ್ ಅಲ್ಲ. ನಿಮ್ಮ ಬಳಕೆದಾರರು ಏಜೆಂಟಿಕ್ ಯುಗದಲ್ಲಿ ವಾಸಿಸುತ್ತಿದ್ದಾರೆ ಮತ್ತು ಪ್ರಾಂಪ್ಟ್‌ಗಳನ್ನು ಬಳಸಲು ಮತ್ತು LLM ಜೊತೆ ಸಂವಹನ ಮಾಡಲು ನಿರೀಕ್ಷಿಸುತ್ತಾರೆ. ನಿಮ್ಮ ಬಳಕೆದಾರರು ನಿಮ್ಮ ಸಾಮರ್ಥ್ಯಗಳನ್ನು MCP ಗೆ ಸಂಗ್ರಹಿಸಬೇಕಾ ಎಂದು ತಿಳಿದುಕೊಳ್ಳುವುದಿಲ್ಲ, ಆದರೆ ಅವರು ಸ್ವಾಭಾವಿಕ ಭಾಷೆಯನ್ನು ಬಳಸುವುದು ನಿರೀಕ್ಷಿಸುತ್ತಾರೆ. ಹಾಗಾಗಿ ಇದನ್ನು ನಾವು ಹೇಗೆ ಪರಿಹರಿಸಬಹುದು? ಪರಿಹಾರವು ಕ್ಲೈಂಟ್‌ಗೆ LLM ಅನ್ನು ಸೇರಿಸುವುದಾಗಿದೆ.

## ಅವಲೋಕನ

ಈ ಪಾಠದಲ್ಲಿ ನಾವು ನಿಮ್ಮ ಕ್ಲೈಂಟ್‌ಗಾಗಿ LLM ಅನ್ನು ಸೇರಿಸುವುದರಲ್ಲಿ ಗಮನ ಹರಿಸುತ್ತೇವೆ ಮತ್ತು ಇದು ನಿಮ್ಮ ಬಳಕೆದಾರರಿಗೆ ಉತ್ತಮ ಅನುಭವವನ್ನು ಹೇಗೆ ಒದಗಿಸುತ್ತದೆ ಎಂಬುದನ್ನು ತೋರಿಸುತ್ತೇವೆ.

## ಕಲಿಕಾ ಉದ್ದೇಶಗಳು

ಈ ಪಾಠದ ಅಂತ್ಯಕ್ಕೆ, ನೀವು ಈ ಕಾರ್ಯಗಳನ್ನು ನಿರ್ವಹಿಸಲು সক্ষমರಾಗಿರುತ್ತೀರಿ:

- LLM ಹೊಂದಿರುವ ಕ್ಲೈಂಟ್ ರಚಿಸಲು.
- LLM ಬಳಸಿ MCP ಸರ್ವರ್ ಜೊತೆ ನಿರಂತರವಾಗಿ ಸಂವಹನ ಮಾಡಲು.
- ಕ್ಲೈಂಟ್ ಬದಿಯ ಮೇಲೆ ಉತ್ತಮ ಅಂತಿಮ ಬಳಕೆದಾರ ಅನುಭವ ಒದಗಿಸಲು.

## ವಿಧಾನ

ನಾವು ತೆಗೆದುಕೊಳ್ಳಬೇಕಾದ ವಿಧಾನವನ್ನು ವಿವರಿಸೋಣ. LLM ಸೇರಿಸುವುದು ಸರಳವಾಗಲಿರುವುದಿಲ್ಲ, ಆದ್ರೆ ನಾವುವೇ ಇದನ್ನು ನಿಜವಾಗಿಯೂ ಮಾಡಬೇಕಾಗುತ್ತದೆಯೇ?

ಕ್ಲೈಂಟ್ ಸರ್ವರ್ ಜೊತೆ ಹೇಗೆ ಸಂವಹನ ಮಾಡುವುದೋ ಇಲ್ಲಿದೆ:

1. ಸರ್ವರ್ ಜೊತೆಗೆ ಸಂಪರ್ಕವನ್ನು ಸ್ಥಾಪಿಸಲಾಗುತ್ತದೆ.

1. ಸಾಮರ್ಥ್ಯಗಳು, ಪ್ರಾಂಪ್ಟ್‌ಗಳು, ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿ, ಅವುಗಳ ಯೋಜನೆಯನ್ನು ಉಳಿಸಿ.

1. LLM ಅನ್ನು ಸೇರಿಸಿ ಮತ್ತು ಉಳಿಸಿದ ಸಾಮರ್ಥ್ಯಗಳು ಹಾಗೂ ಅವುಗಳ ಯೋಜನೆಯನ್ನು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ಸ್ವರೂಪದಲ್ಲಿ ಪಾಸ್ ಮಾಡಿ.

1. ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ ಅನ್ನು LLM ಗೆ ಮತ್ತು ಕ್ಲೈಂಟ್ ಪಟ್ಟ ಮಾಡಿದ ಸಾಧನಗಳೊಡನೆ ಒಟ್ಟಿಗೆ ಪಾಸ್ ಮಾಡಿ ನಿರ್ವಹಿಸಿ.

ಚೆನ್ನಾಗಿದ್ದು, ನಾವು ಎತ್ತರ ಮಟ್ಟದಲ್ಲಿ ಇದನ್ನು ಹೇಗೆ ಮಾಡಬಹುದು ಎಂದು ಅರ್ಥಮಾಡಿಕೊಂಡಿದ್ದೇವೆ, ಕೆಳಗಿನ ಅಭ್ಯಾಸದಲ್ಲಿ ಇದನ್ನು ಪ್ರಯತ್ನಿಸೋಣ.

## ಅಭ್ಯಾಸ: LLM ಹೊಂದಿರುವ ಕ್ಲೈಂಟ್ ರಚನೆ

ಈ ಅಭ್ಯಾಸದಲ್ಲಿ, ನಾವು ಕ್ಲೈಂಟ್‌ಗೆ LLM ಸೇರಿಸುವುದನ್ನು ಕಲಿಯುತ್ತೇವೆ.

### GitHub ವೈಯಕ್ತಿಕ ಪ್ರವೇಶ ಟೋಕನ್ ಬಳಸಿ ಪ್ರಾಮಾಣೀಕರಣ

GitHub ಟೋಕನ್ ರಚಿಸುವುದು ಸರಳ ಪ್ರಕ್ರಿಯೆಯಾಗಿದೆ. ಇದು ಹೇಗೆ ಮಾಡಲು ಸಾಧ್ಯ:

- GitHub ಸೆಟ್ಟಿಂಗ್ಸ್ ಗೆ ಹೋಗಿ – ಮೇಲಿನ ಬಲವನ್ನು ನಿಮ್ಮ ಪ್ರೊಫೈಲ್ ಚಿತ್ರವನ್ನು ಕ್ಲಿಕ್ ಮಾಡಿ ಮತ್ತು ಸೆಟ್ಟಿಂಗ್ಸ್ ಆಯ್ಕೆಮಾಡಿ.
- ಡೆವಲಪರ್ ಸೆಟ್ಟಿಂಗ್ಸ್ ಗೆ ಸಾಗಿಸಿ – ಕೆಳಗೆ ಸ್ಕ್ರೋಲ್ ಮಾಡಿ ಮತ್ತು ಡೆವಲಪರ್ ಸೆಟ್ಟಿಂಗ್ಸ್ ಕ್ಲಿಕ್ ಮಾಡಿ.
- ವೈಯಕ್ತಿಕ ಪ್ರವೇಶ ಟೋಕನ್‌ಗಳನ್ನು ಆರಿಸಿ – ಫೈನ್-ಗ್ರೇನಡೇಟು ಟೋಕನ್ಗಳನ್ನು ಕ್ಲಿಕ್ ಮಾಡಿ ನಂತರ ಹೊಸ ಟೋಕನ್ ರಚಿಸಿ.
- ನಿಮ್ಮ ಟೋಕನ್ ಅನ್ನು ಕಾನ್ಫಿಗರ್ ಮಾಡಿ – ಸ್ಟೋರ್ ಮಾಡಲು ಸೂಚನೆ ನೀಡಿ, ಅವಧಿ ನಿಗದಿ ಮಾಡಿ, ಮತ್ತು ಅಗತ್ಯವಿರುವ ಅನುಮತಿಗಳನ್ನು (ಪರ್ಮಿಷನ್‌ಗಳನ್ನು) ಆರಿಸಿ. ಈ ಸಂದರ್ಭದಲ್ಲಿ, Models ಅನುಮತಿ ಸೇರಿಸಬೇಕು.
- ಟೋಕನ್ ರಚಿಸಿ ಮತ್ತು ನಕಲಿಸಿ – ಜನರೇಟ್ ಟೋಕನ್ ಕ್ಲಿಕ್ ಮಾಡಿ, ತಕ್ಷಣ ನಕಲಿಸಿ, ಏಕೆಂದರೆ ಮತ್ತೆ ನೋಡಲಾರಿರಿ.

### -1- ಸರ್ವರ್‌ಕ್ಕೆ ಸಂಪರ್ಕ

ಮೊದಲು ನಮ್ಮ ಕ್ಲೈಂಟ್ ಅನ್ನು ರಚಿಸೋಣ:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ಸ್ಕೀಮಾ ಮಾನ್ಯತೆಗಾಗಿ zod ಅನ್ನು ಆಮದು ಮಾಡಿ

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

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

- ಅಗತ್ಯವಾದ ಲೈಬ್ರರಿಗಳನ್ನು ಆಮದುಮಾಡಿದ್ದೇವೆ
- `client` ಮತ್ತು `openai` ಎಂಬ ಎರಡು ಸದಸ್ಯರೊಂದಿಗಿನ ಕ್ಲಾಸ್ ರಚಿಸಿದ್ದು, ಇದು ಕ್ಲೈಂಟ್ ಮತ್ತು LLM ಸಂವಹನ ನಿರ್ವಹಿಸಲು ಸಹಾಯಮಾಡುತ್ತದೆ.
- LLM ಉದಾಹರಣೆಯನ್ನು GitHub Models ಬಳಸಿ `baseUrl` ಅನ್ನು ನಿಯತಗೊಳಿಸಿದೆ.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# stdio ಸಂಪರ್ಕಕ್ಕಾಗಿ ಸರ್ವರ್ ಪರಿಮಾಣಗಳನ್ನು ರಚಿಸಿ
server_params = StdioServerParameters(
    command="mcp",  # ಕೈಗೊಳ್ಳಬಹುದಾದ
    args=["run", "server.py"],  # ಐಚ್ಚಿಕ ಕಮಾಂಡ್ ಲೈನ್ ಆರ್ಗ್ಯುಮೆಂಟ್ಸ್
    env=None,  # ಐಚ್ಚಿಕ ಪರಿಸರ ಚರಗಳು
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # ಸಂಪರ್ಕವನ್ನು ಆರಂಭಿಸಿ
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

- MCPಗೆ ಅಗತ್ಯವಾದ ಲೈಬ್ರರಿಗಳನ್ನು ಆಮದುಮಾಡಿದ್ದೇವೆ
- ಕ್ಲೈಂಟ್ ಅನ್ನು ರಚಿಸಿದ್ದೇವೆ

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

ಮೊದಲು, ನಿಮ್ಮ `pom.xml` ಕಡತದಲ್ಲಿ LangChain4j ಅವಲಂಬನೆಗಳನ್ನು ಸೇರಿಸಬೇಕು. MCP ಇಂಟಿಗ್ರೇಶನ್ ಮತ್ತು GitHub Models ಬೆಂಬಲಕ್ಕಾಗಿ ಈ ಅವಲಂಬನೆಗಳನ್ನು ಸೇರಿಸಿ:

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

ನಂತರ ನಿಮ್ಮ ಜಾವಾ ಕ್ಲೈಂಟ್ ಕ್ಲಾಸ್ ರಚಿಸಿರಿ:

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
    
    public static void main(String[] args) throws Exception {        // GitHub ಮಾದರಿಗಳನ್ನು ಬಳಸಲು LLM ಅನ್ನು ಸಂರಚಿಸಿ
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // ಸರ್ವರ್‌ಗೆ ಸಂಪರ್ಕಿಸಲು MCP ಸಂಚರಣೆಯನ್ನು ರಚಿಸಿ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // MCP ಕ್ಲೈಂಟ್ ರಚಿಸಿ
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

- **LangChain4j ಅವಲಂಬನೆಗಳನ್ನು ಸೇರಿಸಿದ್ದೇವೆ**: MCP ಮತ್ತು GitHub Models ಬೆಂಬಲಕ್ಕಾಗಿ ಅಗತ್ಯ
- **LangChain4j ಗ್ರಂಥಾಲಯಗಳನ್ನು ಆಮದುಮಾಡಿದ್ದೇವೆ**: MCP ಇಂಟಿಗ್ರೇಶನ್ ಮತ್ತು OpenAI ಚಾಟ್ ಮಾದರಿಗಾಗಿ
- **`ChatLanguageModel` ರಚಿಸಿದ್ದೇವೆ**: ನಿಮ್ಮ GitHub ಟೋಕನ್ ಬಳಸಿ GitHub Models ಬಳಸಲು ಕಾನ್ಫಿಗರ್ ಮಾಡಿ
- **ಹೆಚ್‌ಟಿಟಿಪಿ ಟ್ರಾನ್ಸ್‌ಪೋರ್ಟ್ ಸೆಟ್ ಮಾಡಿದೆವು**: Server-Sent Events (SSE) ಮೂಲಕ MCP ಸರ್ವರ್‌ಗೆ ಸಂಪರ್ಕಿಸಲು
- **MCP ಕ್ಲೈಂಟನ್ನು ರಚಿಸಿದೆವು**: ಸರ್ವರ್ ಜೊತೆಗೆ ಸಂವಹನ ನಿರ್ವಹಿಸಲು
- **LangChain4j ಇದರಲ್ಲಿ MCP ಬೆಂಬಲವನ್ನು ಬಳಸಿದೆ**: ಇದು LLM ಮತ್ತು MCP ಸರ್ವರ್ ನಡುವೆ ಸಂಯೋಜನೆಯನ್ನು ಸುಲಭಗೊಳಿಸುತ್ತದೆ

#### Rust

ಈ ಉದಾಹರಣೆ ನೀವು Rust ಆಧಾರಿತ MCP ಸರ್ವರ್‌ವನ್ನು ಹೊಂದಿರುವ ಕುರಿತು ಅನ್ವಯಿಸುತ್ತದೆ. ಇಲ್ಲದಿದ್ದರೆ, ಸರ್ವರ್ ರಚಿಸಲು [01-first-server](../01-first-server/README.md) ಪಾಠವನ್ನು ನೋಡಿ.

ನಿಮ್ಮ Rust MCP ಸರ್ವರ್ ಅನ್ನು ಹೊಂದಿರುವ ನಂತರ, ಟರ್ಮಿನಲ್ ತೆರೆಯಿರಿ ಮತ್ತು ಸರ್ವರ್ ಅನುಸ್ಥಿತಿಗಿರುವ ಡೈರೆಕ್ಟರಿಯಲ್ಲಿ ನಯವಾಡಿ. ನಂತರ ಹೊಸ LLM ಕ್ಲೈಂಟ್ ಪ್ರಾಜೆಕ್ಟ್ ರಚಿಸಲು ಕೆಳಗಿನ ಆಜ್ಞೆಯನ್ನು ನಿರ್ವಹಿಸಿ:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

ನಿಮ್ಮ `Cargo.toml` ಕಡತಕ್ಕೆ ಕೆಳಗಿನ ಅವಲಂಬನೆಗಳನ್ನು ಸೇರಿಸಿ:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> OpenAIಗಾಗಿ ಅಧಿಕೃತ Rust ಗ್ರಂಥಾಲಯ ಇಲ್ಲದಿರುವುದರಿಂದ, `async-openai` ಕ್ರೇಟು ಒಂದು [ಸಮುದಾಯ ನಿರ್ವಹಿತ ಗ್ರಂಥಾಲಯ](https://platform.openai.com/docs/libraries/rust#rust) ಆಗಿದೆ ಮತ್ತು ಜನಪ್ರಿಯವಾಗಿದೆ.

`src/main.rs` ಕಡತವನ್ನು ತೆರೆಯಿರಿ ಮತ್ತು ಅದರ ವಿಷಯವನ್ನು ಕೆಳಗಿನ ಕೋಡ್‌ನೊಂದಿಗೆ ಬದಲಾಯಿಸಿ:

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
    // ಪ್ರಾಥಮಿಕ ಸಂದೇಶ
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // OpenAI ಗ್ರಾಹಕವನ್ನು ಹೊಂದಿಸಿ
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // MCP ಗ್ರಾಹಕವನ್ನು ಹೊಂದಿಸಿ
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

    // TODO: MCP ಉಪಕರಣದ ಪಟ್ಟಿ ಪಡೆಯಿರಿ

    // TODO: ಉಪಕರಣ ಕರೆಗಳೊಂದಿಗೆ LLM ಸಂಭಾಷಣೆ

    Ok(())
}
```

ಈ ಕೋಡ್ ಮೂಲಭೂತ Rust ಅಪ್ಲಿಕೇಶನ್ ಅನ್ನು ಸೆಟ್‌ಅಪ್ ಮಾಡುತ್ತದೆ, ಇದು MCP ಸರ್ವರ್ ಮತ್ತು GitHub Models ಜೊತೆಗೆ LLM ಸಂವಹನಕ್ಕಾಗಿ ಸಂಪರ್ಕಗೊಳ್ಳುತ್ತದೆ.

> [!IMPORTANT]
> ಅಪ್ಲಿಕೇಶನ್ ಚಾಲನೆಯಾಗುವ ಮೊದಲು ನಿಮ್ಮ GitHub ಟೋಕನ್ ಅನ್ನು `OPENAI_API_KEY` ಪರಿಸರ ವ್ಯತ್ಯಯವಾಗಿ ನಿಗದಿಪಡಿಸಿ.

ಚೆನ್ನಾಗಿದೆ, ಮುಂದಿನ ಹಂತಕ್ಕೆ ನಾವು ಸರ್ವರ್ ಸಾಮರ್ಥ್ಯಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡೋಣ.

### -2- ಸರ್ವರ್ ಸಾಮರ್ಥ್ಯಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡುವುದು

ಈಗ ನಾವು ಸರ್ವರ್‌ಗೆ ಸಂಪರ್ಕ ಮಾಡಿ ಅದರ ಸಾಮರ್ಥ್ಯಗಳನ್ನು ಕೇಳುತ್ತೇವೆ:

#### Typescript

ಈ ಅದೇ ಕ್ಲಾಸ್‌ನಲ್ಲಿ ಕೆಳಗಿನ ವಿಧಾನಗಳನ್ನು ಸೇರಿಸಿ:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡುವುದು
    const toolsResult = await this.client.listTools();
}
```

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

- ಸರ್ವರ್‌ಕ್ಕೆ ಸಂಪರ್ಕಿಸುವ `connectToServer` ಕೋಡ್ ಸೇರಿಸಿದ್ದೇವೆ.
- ನಮ್ಮ ಅನುಕ್ರಮಣಿಕೆಯನ್ನು ನಿರ್ವಹಿಸುವ `run` ವಿಧಾನ ರಚಿಸಿದೆವು. ಇದು ಈಗಾಗಲೇ ಸಾಧನಗಳನ್ನು ಮಾತ್ರ ಪಟ್ಟಿ ಮಾಡುತ್ತದೆ ಆದರೆ ಇದಕ್ಕೆ ಇನ್ನೂ ಮುಂದುವರಿವು ಸೇರ್ಪಡೆ ಮಾಡುತ್ತೇವೆ.

#### Python

```python
# ಲಭ್ಯವಿರುವ ಸಂಪನ್ಮೂಲಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿ
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# ಲಭ್ಯವಿರುವ ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿ
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

ನಾವು ಸೇರಿಸಿದವು:

- ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿಸಿ ಮತ್ತು ಮುದ್ರಿಸಿವೆ. ಸಾಧನಗಳಿಗಾಗಿ ನಾವು ನಂತರ ಬಳಸುವ `inputSchema` ಕೂಡ ಪಟ್ಟಿ ಮಾಡಲಾಗಿದೆ.

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

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

- MCP ಸರ್ವರ್‌ನಲ್ಲಿ ಲಭ್ಯವಿರುವ ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿದ್ದು
- ಪ್ರತಿ ಸಾಧನದ ಹೆಸರು, ವಿವರಣೆ ಮತ್ತು ಯೋಜನೆವನ್ನು ಪಟ್ಟಿ ಮಾಡಿದ್ದೇವೆ. ಈ ಭಾಗವನ್ನು ನಾವು ಸ್ವಲ್ಪ ಹೊತ್ತಿನಲ್ಲಿ ಉಪಯೋಗಿಸಲಿದ್ದೇವೆ.

#### Java

```java
// ಸ್ವಯಂಚಾಲಿತವಾಗಿ MCP ಪರಿಕರಗಳನ್ನು ಕಂಡುಹಿಡಿಯುವ ಪರಿಕರ ಪೂರೈಕೆದಾರರನ್ನು ರಚಿಸಿ
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// MCP ಪರಿಕರ ಪೂರೈಕೆದಾರರು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ನಿರ್ವಹಿಸುತ್ತದೆ:
// - MCP ಸರ್ವರ್‌ನಿಂದ ಲಭ್ಯವಿರುವ ಪರಿಕರಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡುವುದು
// - MCP ಪರಿಕರ ಸ್ಕೀಮಾಗಳನ್ನು LangChain4j ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸುವುದು
// - ಪರಿಕರ ನಿಯೋಜನೆ ಮತ್ತು ಪ್ರತಿಕ್ರಿಯೆಗಳ ನಿರ್ವಹಣೆ ಮಾಡುವುದು
```

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

- MCP ಸರ್ವರ್‌ನಿಂದ ಎಲ್ಲಾ ಸಾಧನಗಳನ್ನು ಸ್ವಯಂಕ್ರಿಯವಾಗಿ ಹುಡುಕಾಟ, ನೋಂದಣಿ ಮಾಡಿಕೊಳ್ಳುವ `McpToolProvider` ರಚಿಸಿದ್ದೇವೆ
- ಸಾಧನದ ಯೋಜನೆ ಶೈಲಿ MCP ಸಾಧನ ಮತ್ತು LangChain4j ಸಾಧನ ಮಾದರಿಯನ್ನು ಒಳಗೊಳ್ಳುವ ಪರಿವರ್ತನೆಯನ್ನು `tool provider` ಅರ್ಹತೆಯಿಂದ ನಿಭಾಯಿಸುತ್ತದೆ
- ಈ ವಿಧಾನವು ಕೈಯಿಂದ ಸಾಧನ ಪಟ್ಟಿ ಹಾಗೂ ಪರಿವರ್ತನೆ ಪ್ರಕ್ರಿಯೆಯನ್ನು ದೂರವಾಗಿಸುತ್ತದೆ

#### Rust

MCP ಸರ್ವರ್‌ನಿಂದ ಸಾಧನಗಳನ್ನು ಪಡೆಯಲು `list_tools` ವಿಧಾನವನ್ನು ಬಳಸಿ. ನಿಮ್ಮ `main` ಫಂಕ್ಷನ್ನಲ್ಲಿಅ MCP ಕ್ಲೈಂಟ್ ಅನ್ನು ಸೆಟ್ ಮಾಡಿದ ನಂತರ ಈ ಕೆಳಗಿನ ಕೋಡ್ ಸೇರಿಸಿ:

```rust
// MCP ಉಪಕರಣ ಪಟ್ಟಿ ಪಡೆಯಿರಿ
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- ಸರ್ವರ್ ಸಾಮರ್ಥ್ಯಗಳನ್ನು LLM ಸಾಧನಗಳಿಗೆ ಪರಿವರ್ತಿಸುವುದು

ಸರ್ವರ್ ಸಾಮರ್ಥ್ಯಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿದ ನಂತರ ಮುಂದಿನ ಹಂತ ಅವುಗಳನ್ನು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸುವುದು. ಪರಿವರ್ತಿಸಿದ ನಂತರ, ನಮ್ಮ LLM ಗೆ ಈ ಸಾಮರ್ಥ್ಯಗಳನ್ನು ಸಾಧನಗಳಾಗಿ ಒದಗಿಸಬಹುದು.

#### TypeScript

1. MCP ಸರ್ವರ್ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು LLM ಕೈಗೆಟಕುವ ಸಾಧನ ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸಲು ಕೆಳಗಿನ ಕೋಡ್ ಸೇರಿಸಿ:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // ಇನ್‌ಪುಟ್_ಸ್ಕೀಮಾಧಾರಿತ ಜೋಡ್ ಸ್ಕೀಮಾ ರಚಿಸಿ
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // ಪ್ರಕಾರವನ್ನು ಸ್ಪಷ್ಟವಾಗಿ "function" ಆಗಿ ನಿಗದಿಪಡಿಸಿ
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

    ಮೇಲಿನ ಕೋಡ್ MCP ಸರ್ವರ್‌ನ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ತೆಗೆದು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ಸಾಧನ ವ್ಯಾಖ್ಯಾನ ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸುತ್ತದೆ.

1. ನಂತರ, `run` ವಿಧಾನವನ್ನು ನವీకರಿಸೋಣ, ಸರ್ವರ್ ಸಾಮರ್ಥ್ಯಗಳನ್ನು ಪಟ್ಟಿಗೊಳಿಸಲು:

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

    ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು `run` ವಿಧಾನವನ್ನು ನವೀಕರಿಸಿದ್ದು, ಫಲಿತಾಂಶದ ಮೂಲಕ ಮ್ಯಾಪ್ ಮಾಡಿ ಪ್ರತಿ ಎಂಟ್ರಿಗೆ `openAiToolAdapter` ಕರೆಮಾಡುತ್ತೇವೆ.

#### Python

1. ಮೊದಲು ಕೆಳಗಿನ ಪರಿವರ್ತಕ ವಿಧಾನವನ್ನು ರಚಿಸೋಣ:

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

    ಮೇಲಿನ `convert_to_llm_tools` ಫಂಕ್ಷನಿನಲ್ಲಿ ನಾವು MCP ಸಾಧನ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸುತ್ತೇವೆ.

1. ನಂತರ, ಈ ಫಂಕ್ಷನನ್ನು ಉಪಯೋಗಿಸಲು ನಮ್ಮ ಕ್ಲೈಂಟ್ ಕೋಡ್ ನವೀಕರಿಸೋಣ:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

    ಇಲ್ಲಿ MCP ಸಾಧನ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು LLM ಗೆ ತೆರೆಯುವ ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸಲು `convert_to_llm_tool` ಕರೆ ಸೇರಿಸಲಾಗಿದೆ.

#### .NET

1. MCP ಸಾಧನ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸುವ ಕೋಡ್ ಸೇರಿಸೋಣ:

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

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ:

- ಹೆಸರು, ವಿವರಣೆ ಮತ್ತು ಇನ್‌ಪುಟ್ ಸ್ಕೆಮಾ ತೆಗೆದುಕೊಳ್ಳುವ `ConvertFrom` ವಿದ್ಯಮಾನವನ್ನು ರಚಿಸಿದ್ದೇವೆ.
- ಇದು ChatCompletionsDefinition ಗೆ ವರ್ಗಾಯಿಸುವ FunctionDefinition ರಚಿಸಲು ಕಾರ್ಯಗತಗೊಳಿಸುತ್ತದೆ. ನಂತರದ ಭಾಗವು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳಬಹುದು.

1. ಈ ಕಾರ್ಯಾಚರಣೆಯನ್ನು ಉಪಯೋಗಿಸಿ ಕೆಲವು ಸಿದ್ದ ಕೋಡ್ ಅನ್ನು ನವೀಕರಿಸುವುದನ್ನು ನೋಡೋಣ:

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
// ಸಹಜ ಭಾಷಾ ಸಂವಾದಕ್ಕಾಗಿ ಬಾಟ್ ಇಂಟರ್‌ಫೇಸ್ ಅನ್ನು ರಚಿಸಿ
public interface Bot {
    String chat(String prompt);
}

// LLM ಮತ್ತು MCP ಸಾಧನಗಳೊಂದಿಗೆ AI ಸೇವೆಯನ್ನು ಕಾಂಫಿಗರ್ ಮಾಡಿ
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ:

- ನೈಸರ್ಗಿಕ ಭಾಷೆಯಲ್ಲಿ ಸಂವಹನಕ್ಕಾಗಿ ಸರಳ `Bot` ಇಂಟರ್ಫೇಸ್ ವ್ಯಾಖ್ಯಾನಿಸಲಾಗಿದೆ
- LangChain4j ನ `AiServices` ಉಪಯೋಗಿಸಿ LLM ಮತ್ತು MCP ಸಾಧನ ಪ್ರೊವೈಡರ್ ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಸಂಯೋಜಿಸಲಾಗಿದೆ
- ಸಾಧನ ಸ್ಕೆಮಾ ಪರಿವರ್ತನೆ ಮತ್ತು ಫಂಕ್ಷನ್ ಆಗಲೆ ಕರೆಮಾಡುವಿಕೆಯನ್ನು ಫ್ರೇಮ್ವರ್ಕ್ ಹಿಂದೆ ನಿರ್ವಹಿಸುತ್ತದೆ
- ಕೈಯಿಂದ ಪರಿವರ್ತನೆಯ ಅಗತ್ಯ ಇಲ್ಲ - MCP ಸಾಧನಗಳನ್ನು LLM-ಗೆ ಹೊಂದಾಣಿಕೆಯ ರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸುವ ಸಂಕೀರ್ಣತೆ LangChain4j ನಿಂದ ನಿರ್ವಹಣೆಯಾಗುತ್ತದೆ

#### Rust

MCP ಸಾಧನ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ಸ್ವರೂಪಕ್ಕೆ ಪರಿವರ್ತಿಸಲು `main` ಫಂಕ್ಷನಿನ ಕೆಳಗೆ ಸಹಾಯಕ ಫಂಕ್ಷನನ್ನು ಸೇರಿಸೋಣ; ಇದು ಸಾಧನ ಪಟ್ಟಿ ರೂಪಗೊಳಿಸುತ್ತದೆ ಮತ್ತು LLM ಕೇಳುವಾಗ ಬಳಕೆ ಮಾಡಲಾಗುತ್ತದೆ:

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

ಚೆನ್ನಾಗಿದೆ, ಬಳಕೆದಾರ ವಿನಂತಿಗಳನ್ನು ನಿರ್ವಹಿಸಲು ನಾವು ಸಿದ್ಧ, ಹಾಗಾಗಿ ಮುಂದುವರೀತು.

### -4- ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ ವಿನಂತಿ ನಿರ್ವಹಣೆ

ಈ ಭಾಗದಲ್ಲಿ ನಾವು ಬಳಕೆದಾರ ವಿನಂತಿಗಳನ್ನು ನಿರ್ವಹಿಸುವೆವು.

#### TypeScript

1. LLM ಅನ್ನು ಕರೆ ಮಾಡಲು ಬಳಕೆಯಾಗುವ ವಿಧಾನವನ್ನು ಸೇರಿಸೋಣ:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. ಸರ್ವರ್‌ನ ಉಪಕರಣವನ್ನು ಕರೆಯಿರಿ
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ಫಲಿತಾಂಶದೊಂದಿಗೆ ಏದುದೋ ಮಾಡಿ
        // ಮಾಡಬೇಕಿದೆ

        }
    }
    ```

    ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ:

    - `callTools` ವಿಧಾನ ಸೇರಿಸಲಾಗಿದೆ.
    - ಇದು LLM ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ಪಡೆದು ಯಾವ ಸಾಧನಗಳನ್ನು ಕರೆ ಮಾಡಬೇಕೆಂದು ಪರಿಶೀಲಿಸುತ್ತದೆ:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // ಸಾಧನವನ್ನು ಕರೆಮಾಡಿ
        }
        ```

    - LLM ಸೂಚಿಸಿದಂತೆ ಸಾಧನವನ್ನು ಕರೆ ಮಾಡುತ್ತದೆ:

        ```typescript
        // 2. ಸರ್ವರ್‌ನ ಸಾಧನವನ್ನು ಕರೆ ಮಾಡಿ
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. ಫಲಿತಾಂಶದೊಂದಿಗೆ ಏನಾದರೂ ಮಾಡಿ
        // ಮಾಡಬೇಕಿದೆ
        ```

1. `run` ವಿಧಾನವನ್ನು ನವೀಕರಿಸಿ LLM ಕರೆ ಮತ್ತು `callTools` ಸೇರ್ಪಡೆ ಮಾಡುವುದನ್ನು ಮಾಡಿ:

    ```typescript

    // 1. LLM ಗೆ ಇನ್‌ಪುಟ್ ಆಗುವ ಸಂದೇಶಗಳನ್ನು ರಚಿಸಿ
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. LLM ಅನ್ನು ಕರೆಮಾಡುವುದು
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. LLM ಉತ್ತರವನ್ನು ಪರಿಶೀಲಿಸಿ, ಪ್ರತಿ ಆಯ್ಕೆಗೆ ಟೂಲ್ ಕಾಲ್‌ಗಳಿವೆ ಎಂದು ನೋಡಿರಿ
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

ಶುಭಂ, ಸಂಪೂರ್ಣ ಕೋಡ್ ಪಟ್ಟಿ ಇಲ್ಲಿದೆ:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ಸ್ಕೀಮಾ ಮಾನ್ಯತೆಗಾಗಿ ಜೋಡ್ ಅನ್ನು ಆಮದುಮಾಡಿ

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // ಭವಿಷ್ಯದಲ್ಲಿ ಈ URL ಅನ್ನು ಬದಲಾಗಿಸಬೇಕಾದ ಅಗತ್ಯವ থাকতে পারে: https://models.github.ai/inference
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
          // ಇನ್‌ಪುಟ್_ಸ್ಕೀಮಾ ಆಧರಿಸಿ ಜೋಡ್ ಸ್ಕೀಮಾ ರಚಿಸಿ
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // ಸ್ಪಷ್ಟವಾಗಿ "function" ರೂಪವನ್ನು ಸೆಟ್ ಮಾಡಿ
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
    
    
          // 2. ಸರ್ವರ್‌ರ ಸಾಧನವನ್ನು ಕರೆ ಮಾಡಿ
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. ಫಲಿತಾಂಶದೊಂದಿಗೆ ಏನೋ ಮಾಡಿ
          // ಮಾಡಬೇಕಾದದ್ದು (TODO)
    
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
    
        // 1. LLM ಪ್ರತಿಕ್ರಿಯೆಯ ಮೂಲಕ ಹೋಗಿ, ಪ್ರತಿ ಆಯ್ಕೆಗೆ, ಅದು ಸಾಧನ ಕರೆಗಳನ್ನು ಹೊಂದಿದೆಯೇ ಎಂದು ಪರಿಶೀಲಿಸಿ
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

1. LLMನ್ನು ಕರೆ ಮಾಡಲು ಬೇಕಾಗುವ ಕೆಲವು ಆಮದುಗಳನ್ನು ಸೇರಿಸೋಣ

    ```python
    # ಎಲ್‌ಎಲ್‌ಎಮ್
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. ನಂತರ LLM ಅನ್ನು ಕರೆ ಮಾಡುವ ಕಾರ್ಯವನ್ನು ಸೇರಿಸೋಣ:

    ```python
    # ಎಲ್‌ಎಲ್‌ಎಂ

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
            # ಐಚ್ಛಿಕ ಪರಿಮಾಣಗಳು
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

    ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು:

    - MCP ಸರ್ವರ್‌ನಲ್ಲಿ ಕಂಡುಹಿಡಿದ ಮತ್ತು ಪರಿವರ್ತಿಸಿದ 우리의 ಕಾರ್ಯಗಳನ್ನು LLMಗೆ ನೀಡಿದ್ದೇವೆ.
    - ನಂತರ LLM ಅನ್ನು ಆ ಕಾರ್ಯಗಳೊಂದಿಗೆ ಕರೆಮಾಡಿದ್ದೇವೆ.
    - ಫಲಿತಾಂಶವನ್ನು ಪರಿಶೀಲಿಸಿ ಯಾವ ಕಾರ್ಯಗಳನ್ನು ಕರೆ ಮಾಡಬೇಕು ಎಂಬುದನ್ನು ತಿಳಿದುಕೊಳ್ಳುತ್ತೇವೆ.
    - ಕೊನೆಗೆ ಕರೆ ಮಾಡಬೇಕಾದ ಕಾರ್ಯಗಳ ಸರಣಿಯನ್ನು ಪಾಸ್ ಮಾಡುತ್ತೇವೆ.

1. ಕೊನೆಯ ಹಂತ, ನಮ್ಮ ಮುಖ್ಯ ಕೋಡ್ ನವೀಕರಿಸೋಣ:

    ```python
    prompt = "Add 2 to 20"

    # ಎಲ್ಲಕ್ಕೆ ಯಾವ ಉಪಕರಣಗಳನ್ನು ಬೇಕು ಎಂದು LLM ಅನ್ನು ಕೇಳಿ, ಇದ್ದರೆ
    functions_to_call = call_llm(prompt, functions)

    # ಸೂಚಿಸಲಾದ ಕಾರ್ಯಗಳನ್ನು ಕರೆಸಿರಿ
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

    ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ:

    - LLMನ ಪ್ರಾಂಪ್ಟ್ ಆಧಾರಿತವಾಗಿ MCP ಸಾಧನವನ್ನು `call_tool` ಮೂಲಕ ಕರೆಮಾಡುತ್ತಿದೆ.
    - ಸರ್ವರ್‌ ನೋಡಿ ಸಾಧನವನ್ನು ಕರೆ ಮಾಡಿದ ಫಲಿತಾಂಶವನ್ನು ಮುದ್ರಿಸುತ್ತಿದೆ.

#### .NET

1. LLM ಪ್ರಾಂಪ್ಟ್ ವಿನಂತಿಗಾಗಿ ಕೆಲವು ಕೋಡ್ ತೋರಿಸೋಣ:

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

    ಮೇಲೆ:

    - MCP ಸರ್ವರ್‌ನಿಂದ ಸಾಧನಗಳನ್ನು ಪಡೆಯಲಾಗಿದೆ `var tools = await GetMcpTools()`.
    - ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ `userMessage` ನಿರ್ಮಿಸಲಾಗಿದೆ.
    - ಮಾದರಿ ಮತ್ತು ಸಾಧನಗಳನ್ನು ಸೂಚಿಸುವ ಆಯ್ಕೆಗಳು ರಚಿಸಲಾದವು.
    - LLM ಗೆ ವಿನಂತಿ ಕಳಿಸಲಾಗಿದೆ.

1. ಕೊನೆ ಹಂತ, LLM ಯಾವ ಕಾರ್ಯವನ್ನು ಕರೆ ಮಾಡಬೇಕೆಂದು ತಿಳಿದುಕೊಳ್ಳೋಣ:

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

    ಮೇಲೆ:

    - ಕಾರ್ಯಗಳ ಕರೆಯುವಿಕೆಯನ್ನು ಲೂಪ್ ಮಾಡಲಾಗಿದೆ.
    - ಪ್ರತಿ ಸಾಧನ ಕಾರ್‌ಗೆ ಹೆಸರು ಮತ್ತು ನಿರ್ದಿಷ್ಟತೆ ಸರಾನ್ವಯಿಸಿ MCP ಗ್ರಾಹಕ ಬಳಸಿ ಸಾಧನ ಕರೆಮಾಡಿದೆವು. ಕೊನೆಯಲ್ಲಿ ಫಲಿತಾಂಶ ಮುದ್ರಿಸಲಾಗಿದೆ.

ಪೂರ್ಣ ಕೋಡ್ ಇಲ್ಲಿದೆ:

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
    // MCP ಸಾಧನಗಳನ್ನು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಬಳಸಿ ನೈಸರ್ಗಿಕ ಭಾಷಾ ವಿನಂತಿಗಳನ್ನು ಕಾರ್ಯಗತಗೊಳಿಸಿ
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

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ:

- ಸರಳ ನೈಸರ್ಗಿಕ ಭಾಷಾ ಪ್ರಾಂಪ್ಟ್‌ಗಳನ್ನು ಬಳಸಿ MCP ಸಾಧನಗಳೊಂದಿಗೆ ಸಂವಹನ ಮಾಡಲಾಗಿದೆ
- LangChain4j ಫ್ರೇಮ್ವರ್ಕ್ ಸ್ವಯಂಚಾಲಿತವಾಗಿ ನಿರ್ವಹಿಸುತ್ತದೆ:
  - ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್‌ಗಳು ಅಗತ್ಯವಿದ್ದರೆ ಸಾಧನ ಕರೆಗಳಿಗೆ ಪರಿವರ್ತಿಸುವುದು
  - LLM ನಿರ್ಧಾರ ಆಧಾರಿತ MCP ಸಾಧನ ಕರೆದಿತ್ತಲು
  - LLM ಮತ್ತು MCP ಸರ್ವರ್ ನಡುವೆ ಸಂಭಾಷಣೆಯನ್ನು ನಿರ್ವಹಿಸುವುದು
- `bot.chat()` ಪದ್ಧತಿ ನೈಸರ್ಗಿಕ ಭಾಷೆ ಉತ್ತರಗಳನ್ನು ಹಿಂದಿರುಗಿಸುತ್ತದೆ, ಅದರಲ್ಲಿ MCP ಸಾಧನ ಫಲಿತಾಂಶಗಳೂ ಇರಬಹುದು
- ಇದರಿಂದ ಬಳಕೆದಾರರಿಗೆ MCP ಅಡಿಯಲ್ಲಿ ನಡೆಯುವ ವ್ಯವಸ್ಥೆಯ ಅರಿವಿಲ್ಲದೆ ಸಹಜ ಅನುಭವ ಸಿಗುತ್ತದೆ

ಪೂರ್ಣ ಕೋಡ್ ಉದಾಹರಣೆ:

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

ಇಲ್ಲಿ ಅಧಿಕವಾದ ಕೆಲಸ ಸಂಭವಿಸುತ್ತದೆ. ನಾವು ಪ್ರಾರಂಭಿಕ ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ ನೊಂದಿಗೆ LLM ಅನ್ನು ಕರೆಮಾಡುತ್ತೇವೆ, ನಂತರ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ಪರಿಶೀಲಿಸಿ ಯಾವುದೇ ಸಾಧನ ಕರೆ ಬೇಕಾ ಎಂದು ನೋಡುತ್ತೇವೆ. ಈ ಸಲ ಬೇಕಾದಾಗ ನಾವು ಆ ಸಾಧನಗಳನ್ನು ಕರೆ ಮಾಡಿ, LLM ಜೊತೆ ಸಂಭಾಷಣೆಯನ್ನು ನಿರಂತರಗೊಳಿಸುತ್ತೇವೆ, ಸಾಧನ ಕರೆಗಳ ಅಗತ್ಯ ಇಲ್ಲದಾಗ ಮತ್ತು ಅಂತಿಮ ಪ್ರತಿಕ್ರಿಯೆ ಬರುವಾಗ ನಿಲ್ಲಿಸುತ್ತೇವೆ.

ನಾವು ಅನೇಕರಿಗೂ LLM ಕರೆ ಮಾಡುತ್ತೇವೆ, ಆದ್ದರಿಂದ LLM ಕರೆ ನಡೆಸುವ ಫಂಕ್ಷನನ್ನು ವ್ಯಾಖ್ಯಾನಿಸೋಣ. ನಿಮ್ಮ `main.rs` ಕಡತಕ್ಕೆ ಕೆಳಗಿನ ಫಂಕ್ಷನನ್ನು ಸೇರಿಸಿ:

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

ಈ ಫಂಕ್ಷನ್ LLM ಗ್ರಾಹಕ, ಸಂದೇಶಗಳ ಪಟ್ಟಿಯನ್ನು (ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ ಸೇರಿದಂತೆ), MCP ಸರ್ವರ್‌ನ ಸಾಧನಗಳನ್ನು ತೆಗೆದು LLMಗೆ ವಿನಂತಿ ಕಳುಹಿಸುತ್ತದೆ ಮತ್ತು ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ಹಿಂತಿರುಗಿಸುತ್ತದೆ.
LLM ನ ಪ್ರತಿಕ್ರಿಯೆಯಲ್ಲಿ `choices` ಎಂಬ ಸರಣಿಯನ್ನು ಒಳಗೊಂಡಿರುತ್ತದೆ. ಯಾವುದೇ `tool_calls` ಇರುವುದನ್ನು ನೋಡಲು ಫಲಿತಾಂಶವನ್ನು ಪ್ರಕ್ರಿಯೆ ಮಾಡಲು ನಾವು ಅಗತ್ಯವಿದೆ. LLM ನಿರ್ದಿಷ್ಟ ಉಪಕರಣ ಮಾತ್ರವನ್ನು ಕರೆಯಬೇಕೆಂದು ಬೋಲುತ್ತಿದ್ದೆನೆಂಬುದನ್ನು ತಿಳಿಸಲು ಇದು ಸಹಾಯ ಮಾಡುತ್ತದೆ ಮತ್ತು ಆರ್ಗ್ಯೂಮೆಂಟ್‌ಗಳೊಂದಿಗೆ ಕರೆ ಮಾಡಬೇಕು. LLM ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ಹ್ಯಾಂಡಲ್ ಮಾಡುವ ಫಂಕ್ಷನ್ ಅನ್ನು ನಿರ್ದೇಶಿಸಲು ನಿಮ್ಮ `main.rs` ಕಡತದ ಕೆಳಭಾಗದಲ್ಲಿ ಕೆಳಗಿನ ಕೋಡ್ ಅನ್ನು ಸೇರಿಸಿ:

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

    // ಲಭ್ಯವಿದ್ದರೆ ವಿಷಯವನ್ನು ಮುದ್ರಿಸಿ
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // ಟೂಲ್ ಕರೆಗಳನ್ನು ನಡೆಸಿ
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // ಸಹಾಯಕ ಸಂದೇಶವನ್ನು ಸೇರಿಸಿ

        // ಪ್ರತಿ ಟೂಲ್ ಕರೆ ಅನ್ನು ನಡಿಸಿ
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // ಸಂದೇಶಗಳಿಗೆ ಟೂಲ್ ಫಲಿತಾಂಶವನ್ನು ಸೇರಿಸಿ
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // ಟೂಲ್ ಫಲಿತಾಂಶಗಳೊಂದಿಗೆ ಸಂವಾದವನ್ನು ಮುಂದುವರಿಸಿ
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

`tool_calls` ಇದ್ದರೆ, ಅದು ಉಪಕರಣ సమాచಾರವನ್ನು ಪ್ರವೇಶಿಸುತ್ತದೆ, MCP ಸರ್ವರ್‌ಗೆ ಉಪಕರಣ ವಿನಂತಿಯನ್ನು ಕರೆದೊಯ್ಯುತ್ತದೆ ಮತ್ತು ಫಲಿತಾಂಶಗಳನ್ನು ಸಂವಾದ ಸಂದೇಶಗಳಿಗೆ ಸೇರಿಸುತ್ತದೆ. ನಂತರ LLM ಜೊತೆಗೆ ಸಂವಾದವನ್ನು ಮುಂದುವರೆಸುತ್ತದೆ ಮತ್ತು ಸಂದೇಶಗಳು ಸಹಾಯಕರ ಪ್ರತಿಕ್ರಿಯೆ ಮತ್ತು ಉಪಕರಣ ಕರೆ ಫಲಿತಾಂಶಗಳೊಂದಿಗೆ ನವೀಕರಿಸಲಾಗುತ್ತವೆ.

LLM MCP ಕರೆಗಳಿಗೆ ನೀಡುವ ಉಪಕರಣ ಕರೆ ಮಾಹಿತಿಯನ್ನು ಹೊರತೆಗೆಯಲು, ನಾವು ಮತ್ತೊಬ್ಬ ಸಹಾಯಕ ಫಂಕ್ಷನ್ ಅನ್ನು ಸೇರಿಸಲಿದ್ದೇವೆ, ಅದು ಕರೆ ಮಾಡಲು ಬೇಕಾಗುವ ಎಲ್ಲವನ್ನೂ ತೆಗೆಯುತ್ತದೆ. ನಿಮ್ಮ `main.rs` ಕಡತದ ಕೆಳಭಾಗದಲ್ಲಿ ಕೆಳಗಿನ ಕೋಡ್ ಅನ್ನು ಸೇರಿಸಿ:

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

ಎಲ್ಲಾ ಘಟಕಗಳು ಸಿದ್ಧವಾಗಿರುವಾಗ, ಪ್ರಾಥಮಿಕ ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ ಅನ್ನು ಹ್ಯಾಂಡಲ್ ಮಾಡಿ LLM ಅನ್ನು ಕರೆ ಮಾಡಬಹುದು. ನಿಮ್ಮ `main` ಫಂಕ್ಷನ್ ಅನ್ನು ಕೆಳಗಿನ ಕೋಡ್ ಅನ್ನು ಸೇರಿಸಲು ನವೀಕರಿಸಿ:

```rust
// ಟೂಲ್ ಕರೆಗಳನ್ನು ಒಳಗೊಂಡ LLM ಸಂಭಾಷಣೆ
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

ಇದು ಪ್ರಾಥಮಿಕ ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್ ಅನ್ನು LLM ಗೆ ಕೇಳುತ್ತದೆ, ಎರಡು ಸಂಖ್ಯೆಗಳ ಮೊತ್ತವನ್ನು ವಿನಂತಿಸಿ, ಮತ್ತು ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ಪ್ರಕ್ರಿಯೆ ಮಾಡಿ ಉಪಕರಣ ಕರೆಗಳನ್ನು ಡೈನಾಮಿಕ್‌ವಾಗಿ ಹ್ಯಾಂಡಲ್ ಮಾಡುತ್ತದೆ.

ಚೆನ್ನಾಗಿದೆ, ನೀವು ಇದನ್ನು ಮಾಡಿ!

## ಹಾಜರಾತು

ಅಭ್ಯಾಸದಿಂದ ಕೋಡ್ ತೆಗೆದು ಸರ್ವರ್‌ಗೆ ಇನ್ನಷ್ಟು ಉಪಕರಣಗಳನ್ನು ಸೇರಿಸಿ. ನಂತರ LLM ಇರುವ ಕ್ಲೈಯೆಂಟ್ ಅನ್ನು ಸೃಷ್ಟಿಸಿ, ಅಭ್ಯಾಸದಲ್ಲಿದ್ದಂತೆ, ವಿಭಿನ್ನ ಪ್ರಾಂಪ್ಟ್‌ಗಳೊಂದಿಗೆ ಪರೀಕ್ಷಿಸಿ ಎಲ್ಲ MCP ಸರ್ವರ್ ಉಪಕರಣಗಳು ಡೈನಾಮಿಕ್ ಆಗಿ ಕರೆಯಲ್ಪಡುವುದನ್ನು ಖಚಿತಪಡಿಸಿಕೊಳ್ಳಿ. ಈ ರೀತಿಯ ಕ್ಲೈಯೆಂಟ್ ನಿರ್ಮಾಣದ ಮೂಲಕ ಅಂತಿಮ ಬಳಕೆದಾರರಿಗೆ ಉತ್ತಮ ಬಳಕೆದಾರ ಅನುಭವ ಸಿಗುತ್ತದೆ ಏಕೆಂದರೆ ಅವರು ನಿಖರ ಕ್ಲೈಯೆಂಟ್ ಆಜ್ಞೆಗಳ ಬದಲು ಪ್ರಾಂಪ್ಟ್‌ಗಳನ್ನು ಬಳಸಬಲ್ಲರು ಮತ್ತು ಯಾವುದೇ MCP ಸರ್ವರ್ ಕರೆಯಲ್ಪಡುವುದನ್ನು ತಿಳಿಯದು.

## ಪರಿಹಾರ

[Solution](/03-GettingStarted/03-llm-client/solution/README.md)

## ಮುಖ್ಯ ಅಂಶಗಳು

- ನಿಮ್ಮ ಕ್ಲೈಯೆಂಟ್‌ಗೆ LLM ಸೇರಿಸುವುದು ಬಳಕೆದಾರರಿಗೆ MCP ಸರ್ವರ್‌ಗಳೊಂದಿಗೆ ಉತ್ತಮ ಸಂವಹನ ಮಾರ್ಗವನ್ನು ಒದಗಿಸುತ್ತದೆ.
- MCP ಸರ್ವರ್ ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು LLM ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವ ರೀತಿಗೆ ಪರಿವರ್ತಿಸುವ ಅಗತ್ಯವಿದೆ.

## ನಮೂನೆಗಳು

- [ಜೆವಾ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್](../samples/java/calculator/README.md)
- [.NET ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್](../../../../03-GettingStarted/samples/csharp)
- [ಜೆಎಸ್ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್](../samples/javascript/README.md)
- [ಟೈಪ್‌ಸ್ಕ್ರಿಪ್ಟ್ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್](../samples/typescript/README.md)
- [ಪೈಥಾನ್ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್](../../../../03-GettingStarted/samples/python)
- [ರಸ್ಟ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್](../../../../03-GettingStarted/samples/rust)

## ಹೆಚ್ಚುವರಿ ಸಂಪನ್ಮೂಲಗಳು

## ಮುಂದೇನು

- ಮುಂದಿನ ವಿಭಾಗ: [ವಿಜುವಲ್ ಸ್ಟುಡಿಯೋ ಕೋಡ್ ಬಳಸಿ ಸರ್ವರ್ ಅನ್ನು ಉಪಯೋಗಿಸುವುದು](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಅಸ್ವೀಕಾರಿಕೆ**:
ಈ ದಸ್ತಾವೇಜನ್ನು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ನಿಖರತೆಯ ಮೇಲಿನ ಪ್ರಯತ್ನ ಮಾಡುತ್ತೇವೆ ಆದರೆ ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ತಪ್ಪುಗಳು ಅಥವಾ ಅಸತ್ಯತೆಗಳು ಇರಬಹುದು ಎಂಬುದನ್ನು ದಯವಿಟ್ಟು ಗಮನಿಸಿ. ಮೂಲ ದಸ್ತಾವೇಜು ಅದರ ಮೂಲ ಭಾಷೆಯಲ್ಲಿ ಅಧಿಕೃತ ಮೂಲ angesehen ಮಾಡಲಾಗಬೇಕು. ಪ್ರಮುಖ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವರ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡಲಾಗುತ್ತದೆ. ಈ ಅನುವಾದದ ಬಳಕೆಯಿಂದ ಉಂಟಾಗುವ ಯಾವುದೇ ತಪ್ಪು ಗ್ರಹಿಕೆಗಳು ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನಗಳಿಗೆ ನಾವು ಜವಾಬ್ದಾರಿಯಲ್ಲ.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
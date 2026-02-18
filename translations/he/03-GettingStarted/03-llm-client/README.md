# יצירת לקוח עם LLM

עד כה, ראית כיצד ליצור שרת ולקוח. הלקוח היה מסוגל לקרוא לשרת במפורש כדי לרשום את הכלים, המשאבים וההנחיות שלו. עם זאת, זו גישה לא מאוד פרקטית. המשתמש שלך חי בעידן הסוכנות ומצפה להשתמש בהנחיות ולתקשר עם LLM כדי לעשות זאת. עבור המשתמש שלך, זה לא חשוב אם אתה משתמש ב-MCP או לא לאחסן את היכולות שלך, אבל הם כן מצפים להשתמש בשפה טבעית לצורך האינטראקציה. אז איך פותרים את זה? הפתרון הוא להוסיף LLM ללקוח.

## סקירה כללית

בשיעור זה נתמקד בהוספת LLM ללקוח שלך ונראה כיצד זה מספק חוויית משתמש טובה הרבה יותר.

## מטרות הלמידה

בסיום שיעור זה תוכל:

- ליצור לקוח עם LLM.
- לתקשר בצורה חלקה עם שרת MCP באמצעות LLM.
- לספק חוויית משתמש טובה יותר בצד הלקוח.

## גישה

בוא ננסה להבין את הגישה שעלינו לנקוט. הוספת LLM נשמעת פשוטה, אבל האם באמת נעשה זאת?

כך הלקוח יתקשר עם השרת:

1. יצירת חיבור עם השרת.

1. לרשום יכולות, הנחיות, משאבים וכלים, ולשמור את הסכמה שלהם.

1. להוסיף LLM ולמסור את היכולות שנשמרו ואת הסכמה שלהם בפורמט ש-LLM מבין.

1. לטפל בהנחיית משתמש על ידי העברתה ל-LLM יחד עם הכלים שרשום הלקוח.

מעולה, עכשיו כשאנחנו מבינים כיצד לעשות זאת ברמה גבוהה, ננסה זאת בתרגיל למטה.

## תרגיל: יצירת לקוח עם LLM

בתרגיל זה נלמד להוסיף LLM ללקוח שלנו.

### אימות באמצעות אסימון גישה אישי של GitHub

יצירת אסימון GitHub היא תהליך פשוט. כך תוכל לעשות זאת:

- עבור ל-Settings של GitHub – לחץ על תמונת הפרופיל שלך בפינה הימנית העליונה ובחר Settings.
- נווט ל-Developer Settings – גלול מטה ולחץ על Developer Settings.
- בחר Personal Access Tokens – לחץ על Fine-grained tokens ואז Generate new token.
- הגדר את האסימון שלך – הוסף הערה להתייחסות, הגדר תאריך תפוגה, ובחר את הרשאות הנדרשות. במקרה זה ודא להוסיף את ההרשאה Models.
- צור והעתק את האסימון – לחץ Generate token, וודא להעתיק אותו מיידית, כיוון שלא תוכל לראות אותו שוב.

### -1- התחבר לשרת

בוא ניצור קודם את הלקוח שלנו:

#### TypeScript

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ייבא zod לאימות סכימה

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

בקוד הקודם עשינו:

- ייבוא הספריות הנדרשות
- יצירת מחלקה עם שני חברים, `client` ו-`openai` שיעזרו לנו לנהל לקוח ולקיים אינטראקציה עם LLM בהתאמה.
- הגדרת מופע ה-LLM שלנו לשימוש ב-Models של GitHub על ידי הגדרת `baseUrl` לכתובת API ההסקה.

#### Python

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# צור פרמטרים לשרת עבור חיבור stdio
server_params = StdioServerParameters(
    command="mcp",  # קובץ הרצה
    args=["run", "server.py"],  # ארגומנטים אופציונליים לשורת הפקודה
    env=None,  # משתני סביבה אופציונליים
)


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write
        ) as session:
            # אתחל את החיבור
            await session.initialize()


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())

```

בקוד הקודם עשינו:

- ייבוא הספריות הנדרשות ל-MCP
- יצירת לקוח

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

ראשית, תצטרך להוסיף את התלויות של LangChain4j אל הקובץ `pom.xml` שלך. הוסף תלויות אלו כדי לאפשר אינטגרציה עם MCP ותמיכה ב-Models של GitHub:

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

לאחר מכן צור את מחלקת הלקוח בג'אווה:

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
    
    public static void main(String[] args) throws Exception {        // הגדר את ה-LLM לשימוש במודלים של GitHub
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .isGitHubModels(true)
                .apiKey(System.getenv("GITHUB_TOKEN"))
                .timeout(Duration.ofSeconds(60))
                .modelName("gpt-4.1-nano")
                .build();

        // צור תחבורה MCP להתחברות לשרת
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .timeout(Duration.ofSeconds(60))
                .logRequests(true)
                .logResponses(true)
                .build();

        // צור לקוח MCP
        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();
    }
}
```

בקוד הקודם עשינו:

- **הוספת תלויות LangChain4j**: דרושות לאינטגרציה עם MCP, לקוח רשמי של OpenAI, ותמיכה ב-Models של GitHub
- **ייבוא ספריות LangChain4j**: לאינטגרציה עם MCP ולפונקציונליות של מודל הצ'אט OpenAI
- **יצירת `ChatLanguageModel`**: מוגדר לשימוש ב-Models של GitHub עם אסימון GitHub שלך
- **הגדרת תחבורה HTTP**: בשימוש ב-Server-Sent Events (SSE) להתחברות לשרת MCP
- **יצירת לקוח MCP**: שיטפל בתקשורת עם השרת
- **שימוש בתמיכה מובנית של LangChain4j ב-MCP**: שמפשטת את האינטגרציה בין LLM לשרתי MCP

#### Rust

דוגמה זו מניחה שיש לך שרת MCP מבוסס Rust פעיל. אם אין לך אחד, עיין בשיעור [01-first-server](../01-first-server/README.md) ליצירת השרת.

כאשר יש לך את שרת Rust MCP, פתח טרמינל ונווט לאותה תיקייה שבה נמצא השרת. לאחר מכן הרץ את הפקודה הבאה ליצירת פרויקט לקוח LLM חדש:

```bash
mkdir calculator-llmclient
cd calculator-llmclient
cargo init
```

הוסף את התלויות הבאות לקובץ `Cargo.toml` שלך:

```toml
[dependencies]
async-openai = { version = "0.29.0", features = ["byot"] }
rmcp = { version = "0.5.0", features = ["client", "transport-child-process"] }
serde_json = "1.0.141"
tokio = { version = "1.46.1", features = ["rt-multi-thread"] }
```

> [!NOTE]
> אין ספריית Rust רשמית ל-OpenAI, אולם `async-openai` הוא [ספרייה מתוחזקת על ידי הקהילה](https://platform.openai.com/docs/libraries/rust#rust) שהינה נפוצה.

פתח את הקובץ `src/main.rs` והחלף את תוכנו בקוד הבא:

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
    // הודעה ראשונית
    let mut messages = vec![json!({"role": "user", "content": "What is the sum of 3 and 2?"})];

    // הקמת לקוח OpenAI
    let api_key = std::env::var("OPENAI_API_KEY")?;
    let openai_client = Client::with_config(
        OpenAIConfig::new()
            .with_api_base("https://models.github.ai/inference/chat")
            .with_api_key(api_key),
    );

    // הקמת לקוח MCP
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

    // TODO: לקבלת רשימת כלים של MCP

    // TODO: שיחה עם LLM עם קריאות לכלים

    Ok(())
}
```

קוד זה מגדיר אפליקציית Rust בסיסית אשר תתחבר לשרת MCP ול-Models של GitHub לאינטראקציות LLM.

> [!IMPORTANT]
> וודא להגדיר את משתנה הסביבה `OPENAI_API_KEY` עם אסימון GitHub שלך לפני הפעלת האפליקציה.

מעולה, לשלב הבא, בוא נציג את היכולות של השרת.

### -2- רשום יכולות השרת

כעת נתחבר לשרת ונבקש את היכולות שלו:

#### Typescript

באותה מחלקה, הוסף את הפונקציות הבאות:

```typescript
async connectToServer(transport: Transport) {
     await this.client.connect(transport);
     this.run();
     console.error("MCPClient started on stdin/stdout");
}

async run() {
    console.log("Asking server for available tools");

    // רישום כלים
    const toolsResult = await this.client.listTools();
}
```

בקוד הקודם עשינו:

- הוספנו קוד לחיבור לשרת, `connectToServer`.
- יצרנו את המתודה `run` שאחראית על זרימת האפליקציה שלנו. עד כה היא רק רושמת את הכלים אבל נוסיף לה עוד בקרוב.

#### Python

```python
# רשום משאבים זמינים
resources = await session.list_resources()
print("LISTING RESOURCES")
for resource in resources:
    print("Resource: ", resource)

# רשום כלים זמינים
tools = await session.list_tools()
print("LISTING TOOLS")
for tool in tools.tools:
    print("Tool: ", tool.name)
    print("Tool", tool.inputSchema["properties"])
```

הנה מה שהוספנו:

- רשימת משאבים וכלים והדפסתם. עבור כלים גם רשמנו `inputSchema` שנשתמש בו אחר כך.

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

בקוד הקודם עשינו:

- רישום הכלים הזמינים על שרת MCP
- עבור כל כלי, רשמנו שם, תיאור והסכמה שלו. האחרון הוא משהו שנשתמש בו לקריאת הכלים בקרוב.

#### Java

```java
// צור ספק כלים שמגלה אוטומטית כלים של MCP
ToolProvider toolProvider = McpToolProvider.builder()
        .mcpClients(List.of(mcpClient))
        .build();

// ספק הכלים של MCP מטפל אוטומטית ב:
// - רישום כלים זמינים משרת MCP
// - המרת סכמות כלים של MCP לפורמט LangChain4j
// - ניהול ביצוע הכלים והתגובות
```

בקוד הקודם עשינו:

- יצירת `McpToolProvider` שמגלה ומרשם אוטומטית את כל הכלים משרת MCP
- ספק הכלים מטפל בהמרה בין סכמות הכלים של MCP לפורמט הכלים של LangChain4j באופן פנימי
- גישה זו מסתירה את הרישום וההמרה הידנית של כלים

#### Rust

הוצאת כלים משרת MCP נעשית באמצעות השיטה `list_tools`. בפונקציה `main` שלך, לאחר כינוס לקוח MCP, הוסף את הקוד הבא:

```rust
// קבל רשימת כלי MCP
let tools = mcp_client.list_tools(Default::default()).await?;
```

### -3- המרת יכולות השרת לכלי LLM

השלב הבא לאחר רישום יכולות השרת הוא המרתן לפורמט שה-LLM מבין. ברגע שנעשה זאת, נוכל לספק את היכולות האלה ככלים ל-LLM שלנו.

#### TypeScript

1. הוסף את הקוד הבא להמרת התגובה משרת MCP לפורמט כלי שה-LLM יכול להשתמש בו:

    ```typescript
    openAiToolAdapter(tool: {
        name: string;
        description?: string;
        input_schema: any;
        }) {
        // צור סכמת זוד בהתבסס על הקלט_schema
        const schema = z.object(tool.input_schema);
    
        return {
            type: "function" as const, // קבע במפורש את הסוג ל"פונקציה"
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

קוד זה מקבל תגובה משרת MCP והופך אותה להגדרת כלי שה-LLM יכול להבין.

1. נעדכן את המתודה `run` כדי לרשום את יכולות השרת:

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

בקוד הקודם עדכנו את `run` כדי למפות דרך התוצאה ולכל רשומה לקרוא ל-`openAiToolAdapter`.

#### Python

1. קודם כל, ניצור את פונקציית ההמרה הבאה:

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

בפונקציה `convert_to_llm_tools` לעיל, אנו לוקחים תגובה של כלי MCP וממירים אותה לפורמט שה-LLM יכול להבין.

1. לאחר מכן, נעדכן את קוד הלקוח שלנו להשתמש בפונקציה כך:

    ```python
    functions = []
    for tool in tools.tools:
        print("Tool: ", tool.name)
        print("Tool", tool.inputSchema["properties"])
        functions.append(convert_to_llm_tool(tool))
    ```

כאן אנו מוסיפים קריאה ל-`convert_to_llm_tool` על מנת להמיר תגובת כלי MCP למשהו שנוכל להזין ל-LLM מאוחר יותר.

#### .NET

1. נוסיף קוד להמרת תגובת כלי MCP למשהו שה-LLM יכול להבין:

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

בקוד הקודם עשינו:

- יצירת פונקציה `ConvertFrom` שלוקחת שם, תיאור וסכמת קלט.
- הגדרת פונקציונליות שיוצרת `FunctionDefinition` שמועבר ל-`ChatCompletionsDefinition`. האחרון הוא פורמט שה-LLM יכול להבין.

1. נעדכן קוד קיים לנצל את הפונקציה שלמעלה:

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
// צור ממשק בוט לאינטראקציה בשפה טבעית
public interface Bot {
    String chat(String prompt);
}

// הגדר את שירות ה-AI עם כלים LLM ו-MCP
Bot bot = AiServices.builder(Bot.class)
        .chatLanguageModel(model)
        .toolProvider(toolProvider)
        .build();
```

בקוד הקודם עשינו:

- הגדרת ממשק פשוט `Bot` לאינטראקציות בשפה טבעית
- שימוש ב-`AiServices` של LangChain4j לקשר אוטומטי בין ה-LLM לספק הכלים של MCP
- המסגרת מנהלת אוטומטית המרת סכמות כלים וקריאת פונקציות מאחורי הקלעים
- גישה זו מבטלת את הצורך בהמרת כלים ידנית - LangChain4j מטפל בכל המורכבות של המרת כלים של MCP לפורמט שה-LLM תומך בו

#### Rust

להמרת תגובת כלי MCP לפורמט שה-LLM יכול להבין, נוסיף פונקציית עזר שמפרמטת את רשימת הכלים. הוסף את הקוד הבא לקובץ `main.rs` מתחת לפונקציה `main`. פונקציה זו תקרא בעת ביצוע בקשות ל-LLM:

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

מעולה, אנחנו מוכנים לטפל בבקשות משתמש, אז נתקדם לשלב הבא.

### -4- טיפול בבקשת הנחיית משתמש

בחלק זה של הקוד נטפל בבקשות משתמש.

#### TypeScript

1. הוסף מתודה שתשמש לקריאת ה-LLM שלנו:

    ```typescript
    async callTools(
        tool_calls: OpenAI.Chat.Completions.ChatCompletionMessageToolCall[],
        toolResults: any[]
    ) {
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);


        // 2. קריאה לכלי של השרת
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. עשה משהו עם התוצאה
        // לעשות

        }
    }
    ```

בקוד הקודם עשינו:

- הוספת מתודה `callTools`.
- המתודה מקבלת תגובת LLM ובודקת אילו כלים נקראו, אם בכלל:

        ```typescript
        for (const tool_call of tool_calls) {
        const toolName = tool_call.function.name;
        const args = tool_call.function.arguments;

        console.log(`Calling tool ${toolName} with args ${JSON.stringify(args)}`);

        // קריאת כלי
        }
        ```

- קורא לכלי, אם ה-LLM מציין שיש לקרוא לו:

        ```typescript
        // 2. לקרוא לכלי של השרת
        const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
        });

        console.log("Tool result: ", toolResult);

        // 3. לעשות משהו עם התוצאה
        // יש לעשות
        ```

1. עדכן את המתודה `run` שיכלול קריאות ל-LLM וקריאה ל-`callTools`:

    ```typescript

    // 1. צור הודעות שהן קלט עבור מודל השפה הגדול (LLM)
    const prompt = "What is the sum of 2 and 3?"

    const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
            {
                role: "user",
                content: prompt,
            },
        ];

    console.log("Querying LLM: ", messages[0].content);

    // 2. קריאה למודל השפה הגדול (LLM)
    let response = this.openai.chat.completions.create({
        model: "gpt-4.1-mini",
        max_tokens: 1000,
        messages,
        tools: tools,
    });    

    let results: any[] = [];

    // 3. עבר על תגובת ה-LLM, עבור כל אפשרות, בדוק אם יש לה קריאות לכלים
    (await response).choices.map(async (choice: { message: any; }) => {
        const message = choice.message;
        if (message.tool_calls) {
            console.log("Making tool call")
            await this.callTools(message.tool_calls, results);
        }
    });
    ```

מעולה, הנה הקוד במלואו:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
import { Transport } from "@modelcontextprotocol/sdk/shared/transport.js";
import OpenAI from "openai";
import { z } from "zod"; // ייבא את zod לאימות סכימה

class MyClient {
    private openai: OpenAI;
    private client: Client;
    constructor(){
        this.openai = new OpenAI({
            baseURL: "https://models.inference.ai.azure.com", // ייתכן שיהיה צורך לשנות לכתובת URL זו בעתיד: https://models.github.ai/inference
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
          // צור סכימת zod מבוססת על input_schema
          const schema = z.object(tool.input_schema);
      
          return {
            type: "function" as const, // הגדר במפורש את סוג ל-"function"
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
    
    
          // 2. קרא לכלי של השרת
          const toolResult = await this.client.callTool({
            name: toolName,
            arguments: JSON.parse(args),
          });
    
          console.log("Tool result: ", toolResult);
    
          // 3. עשה משהו עם התוצאה
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
    
        // 1. עבור על התגובה של ה-LLM, עבור כל אפשרות, בדוק אם יש קריאות לכלים
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

1. נוסיף כמה ייבואים הנדרשים לקריאת LLM:

    ```python
    # למ"מ
    import os
    from azure.ai.inference import ChatCompletionsClient
    from azure.ai.inference.models import SystemMessage, UserMessage
    from azure.core.credentials import AzureKeyCredential
    import json
    ```

1. לאחר מכן נוסיף את הפונקציה שקוראת ל-LLM:

    ```python
    # llm

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
            # פרמטרים אופציונליים
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

בקוד הקודם עשינו:

- העברנו ל-LLM את הפונקציות שמצאנו בשרת MCP והמרנו
- לאחר מכן קראנו ל-LLM עם אותן פונקציות
- אחר כך בדקנו את התוצאה כדי לראות אילו פונקציות יש לקרוא, אם בכלל
- לבסוף, העברנו מערך פונקציות לקריאה

1. שלב סופי, נעדכן את הקוד הראשי שלנו:

    ```python
    prompt = "Add 2 to 20"

    # תשאל את ה-LLM אילו כלים זמינים, אם בכלל
    functions_to_call = call_llm(prompt, functions)

    # קרא לפונקציות המוצעות
    for f in functions_to_call:
        result = await session.call_tool(f["name"], arguments=f["args"])
        print("TOOLS result: ", result.content)
    ```

הנה, זה השלב הסופי, בקוד הנ"ל אנו:

- קוראים לכלי MCP באמצעות `call_tool` תוך שימוש בפונקציה שה-LLM חשב שצריך לקרוא לה בהתבסס על ההנחיה שלנו
- מדפיסים את תוצאת קריאת הכלי לשרת MCP

#### .NET

1. נראה קוד שמבצע בקשה להנחיית LLM:

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

בקוד הקודם עשינו:

- משכנו כלים משרת MCP, `var tools = await GetMcpTools()`.
- הגדרת הנחיית משתמש `userMessage`.
- יצירת אובייקט אפשרויות כולל דגם וכלים.
- ביצוע בקשה אל ה-LLM.

1. שלב אחרון, נראה אם ה-LLM חושב שצריך לקרוא פונקציה:

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

בקוד הקודם עשינו:

- לולאה על רשימת קריאות פונקציות.
- עבור כל קריאת כלי, מנתחים שם וארגומנטים וקוראים לכלי דרך לקוח MCP. בסוף מדפיסים את התוצאות.

הנה הקוד במלואו:

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
    // בצע בקשות בשפה טבעית שמשתמשות בכלי MCP באופן אוטומטי
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

בקוד הקודם עשינו:

- שימוש בהנחיות שפה טבעית פשוטות לאינטראקציה עם כלים משרת MCP
- מסגרת LangChain4j מנהלת אוטומטית:
  - המרת הנחיות משתמש לקריאות לכלי לפי הצורך
  - קריאת הכלים המתאימים בשרת MCP לפי החלטת ה-LLM
  - ניהול מהלך שיחה בין ה-LLM לשרת MCP
- המתודה `bot.chat()` מחזירה תגובות בשפה טבעית שעשויות לכלול תוצאות מהפעלת כלים של MCP
- גישה זו מספקת חוויית משתמש חלקה שבה המשתמשים אינם צריכים לדעת על יישום MCP מתחת לפני השטח

דוגמת קוד מלאה:

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

כאן מתרחש רוב העבודה. נקרא ל-LLM עם הנחיית המשתמש ההתחלתית, ואז נעבד את התגובה כדי לבדוק האם יש צורך לקרוא לכלים. אם כן, נקרא לכלים אלה ונמשיך בשיחה עם ה-LLM עד שלא יידרשו עוד קריאות כלים וקיבלנו תגובה סופית.

נבצע קריאות מרובות ל-LLM, לכן נגדיר פונקציה שתטפל בקריאת ה-LLM. הוסף את הפונקציה הבאה ל`main.rs` שלך:

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

פונקציה זו מקבלת את לקוח ה-LLM, רשימת הודעות (כולל הנחיית המשתמש), כלים משרת MCP, ושולחת בקשה ל-LLM ומחזירה את התגובה.
התשובה מ-LLM תכיל מערך של `choices`. נצטרך לעבד את התוצאה כדי לראות אם קיימים `tool_calls`. זה מאפשר לנו לדעת ש-LLM מבקש שיקראו לכלי ספציפי עם ארגומנטים. הוסף את הקוד הבא לתחתית הקובץ `main.rs` שלך כדי להגדיר פונקציה שתטפל בתשובת ה-LLM:

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

    // הדפס את התוכן אם זמין
    if let Some(content) = message.get("content").and_then(|c| c.as_str()) {
        println!("🤖 {}", content);
    }

    // טיפול בקריאות כלי
    if let Some(tool_calls) = message.get("tool_calls").and_then(|tc| tc.as_array()) {
        messages.push(message.clone()); // הוסף הודעת עוזר

        // ביצוע כל קריאות הכלי
        for tool_call in tool_calls {
            let (tool_id, name, args) = extract_tool_call_info(tool_call)?;
            println!("⚡ Calling tool: {}", name);

            let result = mcp_client
                .call_tool(CallToolRequestParam {
                    name: name.into(),
                    arguments: serde_json::from_str::<Value>(&args)?.as_object().cloned(),
                })
                .await?;

            // הוסף את תוצאות הכלי להודעות
            messages.push(json!({
                "role": "tool",
                "tool_call_id": tool_id,
                "content": serde_json::to_string_pretty(&result)?
            }));
        }

        // המשך שיחה עם תוצאות הכלי
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
  
אם קיימים `tool_calls`, הפונקציה תחלץ את מידע הכלי, תקרא לשרת MCP עם בקשת הכלי, ותוסיף את התוצאות להודעות השיחה. לאחר מכן ממשיכה השיחה עם ה-LLM וההודעות מתעדכנות עם תגובת העוזר ותוצאות קריאת הכלי.

כדי לחלץ מידע על קריאות כלי שה-LLM מחזיר עבור קריאות ל-MCP, נוסיף פונקציית עזר נוספת שתחלץ את כל הנדרש לביצוע הקריאה. הוסף את הקוד הבא לתחתית הקובץ `main.rs` שלך:

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
  
עם כל הרכיבים במקומם, נוכל כעת לטפל בהנחיית המשתמש ההתחלתית ולקרוא ל-LLM. עדכן את פונקציית `main` שלך כך שתכלול את הקוד הבא:

```rust
// שיחה עם מודל שפה גדול הכוללת קריאות לכלים
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
  
זה ישאל את ה-LLM עם הנחיית המשתמש ההתחלתית לבקשת סכום של שני מספרים, ויעבד את התגובה לטיפול דינמי בקריאות לכלים.

נהדר, הצלחת!

## מטלה

קח את הקוד מהתרגיל ובנה את השרת עם עוד כלים. לאחר מכן צור לקוח עם LLM, כמו בתרגיל, ובדוק אותו עם הנחיות שונות כדי לוודא שכל כלי השרת שלך נקראים דינמית. דרך בניית הלקוח הזו מאפשרת למשתמשים חווית משתמש מעולה כי הם יכולים להשתמש בהנחיות במקום פקודות מדויקות ללקוח, ויהיו בלתי מודעים לקריאות כלשהן לשרת MCP.

## פתרון

[פתרון](/03-GettingStarted/03-llm-client/solution/README.md)

## נקודות מפתח

- הוספת LLM ללקוח שלך מספקת דרך טובה יותר למשתמשים לתקשר עם שרתי MCP.  
- יש להמיר את תגובת שרת MCP למשהו שה-LLM יכול להבין.

## דוגמאות

- [מחשבון ג'אווה](../samples/java/calculator/README.md)  
- [מחשבון .Net](../../../../03-GettingStarted/samples/csharp)  
- [מחשבון JavaScript](../samples/javascript/README.md)  
- [מחשבון TypeScript](../samples/typescript/README.md)  
- [מחשבון Python](../../../../03-GettingStarted/samples/python)  
- [מחשבון Rust](../../../../03-GettingStarted/samples/rust)

## משאבים נוספים

## מה הלאה

- הבא: [צריכת שרת באמצעות Visual Studio Code](../04-vscode/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:  
מסמך זה תורגם באמצעות שירות התרגום המבוסס בינה מלאכותית [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לזכור כי תרגומים ממוחשבים עלולים להכיל טעויות או אי-דיוקים. יש להתייחס למסמך המקורי בשפת המקור כמקור הסמכותי. למידע קריטי מומלץ להסתייע בתרגום מקצועי על ידי אדם. אנו לא נושאים באחריות לכל אי-הבנה או פרשנות שגויה הנובעת משימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# 進階伺服器使用

MCP SDK 中暴露了兩種不同類型的伺服器，你常用的伺服器與低階伺服器。通常，你會用常規伺服器來添加功能。但在某些情況下，你會依賴低階伺服器，例如：

- 更佳的架構。用常規伺服器與低階伺服器都能建立乾淨的架構，但可以說用低階伺服器稍微更容易一些。
- 功能可用性。有些進階功能只能透過低階伺服器使用。後續章節你將看到這點，我們會新增抽樣與引導。

## 常規伺服器與低階伺服器

以下是用常規伺服器建立 MCP 伺服器的樣子

**Python**

```python
mcp = FastMCP("Demo")

# 新增一個加法工具
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b
```

**TypeScript**

```typescript
const server = new McpServer({
  name: "demo-server",
  version: "1.0.0"
});

// 新增一個加法工具
server.registerTool("add",
  {
    title: "Addition Tool",
    description: "Add two numbers",
    inputSchema: { a: z.number(), b: z.number() }
  },
  async ({ a, b }) => ({
    content: [{ type: "text", text: String(a + b) }]
  })
);
```

重點是你明確地新增每個你想讓伺服器擁有的工具、資源或提示。這沒什麼問題。

### 低階伺服器方式

不過用低階伺服器方式時，你需要用不一樣的思維：不是註冊每個工具，而是針對每種類型的功能（工具、資源或提示）建立兩個處理器。舉例來說，工具只有兩個函式：

- 列出所有工具。一個函式負責處理所有列出工具的嘗試。
- 處理呼叫所有工具。這裡也只有一個函式負責處理對工具的呼叫。

聽起來工作量可能較少，對吧？所以不是註冊工具，而是確保工具在列出所有工具時會被列出，並且在收到呼叫工具的請求時會被呼叫。

我們來看看現在的程式碼長什麼樣：

**Python**

```python
@server.list_tools()
async def handle_list_tools() -> list[types.Tool]:
    """List available tools."""
    return [
        types.Tool(
            name="add",
            description="Add two numbers",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number", "description": "nubmer to add"}, 
                    "b": {"type": "number", "description": "nubmer to add"}
                },
                "required": ["query"],
            },
        )
    ]
```

**TypeScript**

```typescript
server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // 返回已註冊工具的清單
  return {
    tools: [{
        name="add",
        description="Add two numbers",
        inputSchema={
            "type": "object",
            "properties": {
                "a": {"type": "number", "description": "nubmer to add"}, 
                "b": {"type": "number", "description": "nubmer to add"}
            },
            "required": ["query"],
        }
    }]
  };
});
```

這裡我們有一個回傳功能清單的函式。工具清單裡的每一筆現在都有 `name`、`description` 和 `inputSchema` 這些欄位以符合回傳型別。這讓我們能將工具與功能定義放到別處。我們現在可以在 tools 資料夾建立所有工具，功能同理，所以你的專案組織可以突然變成這樣：

```text
app
--| tools
----| add
----| substract
--| resources
----| products
----| schemas
--| prompts
----| product-description
```

太好了，我們的架構可以變得非常整齊。

呼叫工具呢？也是同樣概念嗎，一個處理器呼叫任意工具？沒錯，這是相關範例程式：

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools 是一個以工具名稱為鍵的字典
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

**TypeScript**

```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
    const { params: { name } } = request;
    let tool = tools.find(t => t.name === name);
    if(!tool) {
        return {
            error: {
                code: "tool_not_found",
                message: `Tool ${name} not found.`
            }
       };
    }
    
    // 參數：request.params.arguments
    // 待辦 呼叫工具，

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

從上面程式碼看到，我們需要解析出要呼叫哪個工具，以及用哪些參數，然後執行呼叫該工具。

## 用驗證改善此方式

目前為止你已看到如何用這每種類型兩個處理器替代所有新增工具、資源與提示的註冊。還需要做什麼呢？我們應該加入某種驗證機制，確保呼叫工具時的參數是正確的。每個執行環境都有自己的解決方案，例如 Python 用 Pydantic，TypeScript 用 Zod。其想法是：

- 移動建立功能（工具、資源或提示）的邏輯到它對應的資料夾。
- 新增一種方式驗證進來請求，例如呼叫工具時的參數。

### 建立功能

要建立功能，我們需要為該功能建立檔案，並確保它具備該功能必備欄位。工具、資源與提示的欄位會略有不同。

**Python**

```python
# schema.py
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float

# add.py

from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: 加入 Pydantic，以便我們可以建立 AddInputModel 並驗證參數

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

你可看到我們做以下幾件事：

- 在 *schema.py* 建立 Pydantic `AddInputModel`，欄位為 `a` 和 `b`。
- 嘗試解析傳入請求為 `AddInputModel`，如果參數不匹配會拋錯：

   ```python
   # add.py
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

你可以決定是把解析邏輯放在工具呼叫本體，或是放在處理器函式。

**TypeScript**

```typescript
// 服務器.ts
server.setRequestHandler(CallToolRequestSchema, async (request) => {
    const { params: { name } } = request;
    let tool = tools.find(t => t.name === name);
    if (!tool) {
       return {
        error: {
            code: "tool_not_found",
            message: `Tool ${name} not found.`
        }
       };
    }
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);

       // @ts-忽略
       const result = await tool.callback(input);

       return {
          content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
      };
    } catch (error) {
       return {
          error: {
             code: "invalid_arguments",
             message: `Invalid arguments for tool ${name}: ${error instanceof Error ? error.message : String(error)}`
          }
    };
   }

});

// 架構.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// 新增.ts
import { Tool } from "./tool.js";
import { MathInputSchema } from "./schema.js";
import { zodToJsonSchema } from "zod-to-json-schema";

export default {
    name: "add",
    rawSchema: MathInputSchema,
    inputSchema: zodToJsonSchema(MathInputSchema),
    callback: async ({ a, b }) => {
        return {
            content: [{ type: "text", text: String(a + b) }]
        };
    }
} as Tool;
```

- 在處理全部工具呼叫的處理器中，我們嘗試將進來的請求解析到該工具定義的模式：

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    如果成功，就繼續呼叫實際工具：

    ```typescript
    const result = await tool.callback(input);
    ```

可見這方式能形成良好架構，所有東西都有位置，*server.ts* 是個非常小的檔案，只負責接線請求處理器，每項功能則放在其對應的資料夾如 tools/、resources/ 或 prompts/。

很好，我們接著來建構這部分。

## 練習：建立低階伺服器

本練習將做如下幾件事：

1. 建立低階伺服器，處理工具列出與工具呼叫。
2. 實作一個你能持續擴充的架構。
3. 加入驗證，確保工具呼叫被適當驗證。

### -1- 建立架構

首要是我們先建立架構，方便往後擴充，長得像這樣：

**Python**

```text
server.py
--| tools
----| __init__.py
----| add.py
----| schema.py
client.py
```

**TypeScript**

```text
server.ts
--| tools
----| add.ts
----| schema.ts
client.ts
```

現在我們建立的架構確保能輕鬆在 tools 資料夾新增工具。你也可以依此流程為資源和提示新增子資料夾。

### -2- 建立工具

接著看看建立工具長什麼樣。首先，要在 *tool* 子目錄建立，如下：

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # 待辦事項：加入 Pydantic，使我們可以建立 AddInputModel 並驗證參數

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

這裡可看到我們如何定義名稱、說明、用 Pydantic 定義輸入結構，還有當工具被呼叫時執行的處理器。最後，我們公開 `tool_add` 字典，包含所有這些屬性。

還有 *schema.py* 用來定義這個工具用的輸入結構：

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

我們還需填寫 *__init__.py*，讓 tools 目錄被視為模組，並暴露其內部模組，如下：

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

隨著工具越來越多，我們將持續往此檔案新增。

**TypeScript**

```typescript
import { Tool } from "./tool.js";
import { MathInputSchema } from "./schema.js";
import { zodToJsonSchema } from "zod-to-json-schema";

export default {
    name: "add",
    rawSchema: MathInputSchema,
    inputSchema: zodToJsonSchema(MathInputSchema),
    callback: async ({ a, b }) => {
        return {
            content: [{ type: "text", text: String(a + b) }]
        };
    }
} as Tool;
```

這裡建立一個字典包含屬性：

- name，工具名稱。
- rawSchema，Zod schema，用於驗證呼叫該工具的請求。
- inputSchema，該結構由處理器使用。
- callback，用來執行該工具。

還有 `Tool`，用來將此字典轉為 mcp 伺服器處理器能接受的型別，如下：

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

還有 *schema.ts* 放工具輸入結構，現有一個 schema，但隨工具新增會持續新增：

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

太好了，我們接著處理列出工具。

### -3- 處理工具列出

接著要處理列出工具，需設定請求處理器。要加的程式碼在伺服器檔案：

**Python**

```python
# 為簡潔起見，省略代碼
from tools import tools

@server.list_tools()
async def handle_list_tools() -> list[types.Tool]:
    tool_list = []
    print(tools)

    for tool in tools.values():
        tool_list.append(
            types.Tool(
                name=tool["name"],
                description=tool["description"],
                inputSchema=pydantic_to_json(tool["input_schema"]),
            )
        )
    return tool_list
```

這裡加上裝飾器 `@server.list_tools` 與實作函式 `handle_list_tools`。函式要回傳工具清單。注意每個工具需包含 `name`、`description` 與 `inputSchema`。

**TypeScript**

為了設置列出工具的請求處理器，我們調用伺服器的 `setRequestHandler`，帶入相符我們需求的 schema，例 : `ListToolsRequestSchema`。

```typescript
// index.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// server.ts
// 為簡潔起見省略代碼
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // 返回已註冊工具的列表
  return {
    tools: tools
  };
});
```

很好，我們完成列出工具這一塊，接著看看怎麼呼叫工具。

### -4- 處理呼叫工具

呼叫工具時，我們要設另個請求處理器，針對請求指明要呼叫哪個功能及其參數。

**Python**

我們使用裝飾器 `@server.call_tool`，透過函式 `handle_call_tool` 實作。裡頭要解析工具名稱、引數，並確保參數對該工具有效。我們可以在此函式或工具本體做參數驗證。

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools 是一個以工具名稱作為鍵的字典
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # 調用該工具
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

流程說明：

- 我們有工具名稱在輸入參數 `name` 中，參數則在 `arguments` 字典格式。

- 使用 `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` 呼叫工具。參數驗證在 `handler` 屬性指向的函式中進行，失敗會拋異常。

此處，我們完整了解如何用低階伺服器列舉與呼叫工具。

[完整範例](./code/README.md) 參考此處

## 作業

擴充你已有的程式碼，加上多個工具、資源與提示，反思你會發現只需新增檔案在 tools 目錄，不須改其他地方。

*未提供解答*

## 總結

本章我們看到低階伺服器方式運作，以及它如何幫助我們建立可持續擴充的良好架構。我們也討論驗證，並介紹如何使用驗證函式庫建立輸入驗證結構。

## 接下來

- 下一章：[簡易認證](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：  
本文件由人工智能翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們致力於確保準確性，請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於重要資訊，建議聘請專業人工翻譯。我們概不對因使用此翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# 進階伺服器使用

MCP SDK 中公開了兩種不同類型的伺服器，分別是你的普通伺服器和低階伺服器。通常你會使用普通伺服器來新增功能。但在某些情況下，你會想依賴低階伺服器，例如：

- 更佳的架構。使用普通伺服器和低階伺服器都能建立乾淨的架構，但有人認為使用低階伺服器會稍微更簡單。
- 功能可用性。某些進階功能只能透過低階伺服器使用。你會在後面的章節看到我們加入抽樣和誘導的部分。

## 普通伺服器 vs 低階伺服器

以下是使用普通伺服器建立 MCP Server 的範例

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

重點在於你明確地新增每個你希望伺服器擁有的工具、資源或提示。這樣做沒錯。

### 低階伺服器方式

使用低階伺服器時，你需要換個角度思考，具體來說不是註冊每一個工具，而是對每種功能類型（工具、資源或提示）建立兩個處理器。例如對工具來說，只有兩個函式：

- 列出所有工具。這個函式負責所有列出工具的請求。
- 處理呼叫所有工具。這裡只有一個函式負責處理呼叫工具。

聽起來好像工作量比較少對吧？所以不需要註冊工具，只要確保在列出工具時包含該工具，且當有呼叫工具請求時它能被呼叫。

我們來看看程式碼現在的樣貌：

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
  // 返回已註冊工具的列表
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

這裡我們有一個函式回傳功能列表。每個工具清單裡都包含 `name`、`description` 和 `inputSchema` 等欄位以符合回傳型別。這使我們可以把工具和功能定義放在其他地方。我們現在可以在一個 tools 資料夾中建立所有工具，其他功能也是如此，專案結構可以變成如下：

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

太棒了，我們的架構可以設計得相當乾淨。

呼叫工具也是同理嗎？只用一個處理器呼叫任何工具？沒錯，下面是相關程式碼：

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
    // TODO 呼叫工具，

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

從以上程式碼可以看到，我們需要解析要呼叫的工具以及它所帶的參數，接著執行呼叫該工具。

## 使用驗證改進方法

到目前為止，你已經看過如何用每種功能類型的兩個處理器替代所有註冊工具、資源和提示的方式。那還需要做什麼呢？我們應該添加某種驗證機制，以確保工具被呼叫時帶上正確的參數。各個執行環境有自己的解決方案，例如 Python 使用 Pydantic，而 TypeScript 使用 Zod。核心概念如下：

- 把建立功能（工具、資源或提示）的邏輯搬到專屬資料夾。
- 添加方法驗證前來呼叫工具的請求。

### 建立功能

要建立功能，我們需要為該功能建立一個檔案，並確保它擁有該功能必須的欄位，不同工具、資源和提示間欄位略有差異。

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

    # TODO: 添加 Pydantic，這樣我們可以創建 AddInputModel 並驗證參數

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

這裡示範了：

- 透過 Pydantic 建立架構 `AddInputModel`，欄位有 `a` 和 `b`，位於 *schema.py*。
- 嘗試解析輸入請求為 `AddInputModel` 類型，若參數不符會引發錯誤：

   ```python
   # add.py
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

你可以選擇把這段解析邏輯放在工具呼叫本身或在處理器函式中。

**TypeScript**

```typescript
// 伺服器.ts
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

- 在處理所有工具呼叫的 handler 會先嘗試將輸入請求解析成對應工具定義的 schema：

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    若成功，接著呼叫工具：

    ```typescript
    const result = await tool.callback(input);
    ```

正如你所見，這個方法創造出優秀的架構，所有東西各就各位，*server.ts* 只是一個很小的檔案，只負責串接請求處理函式，而各功能分別放在對應資料夾內，如 tools/、resources/ 或 prompts/。

很好，我們接著來實作看看。

## 練習：建立低階伺服器

本練習將進行以下步驟：

1. 建立一個低階伺服器，處理列出工具和呼叫工具。
1. 實作一個可擴充的架構。
1. 添加驗證以確保工具呼叫參數正確。

### -1- 建立架構

首先要解決的是架構問題，讓我們能隨著功能擴充而擴展，架構如下：

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

現在我們建立了一個架構，可以輕鬆在 tools 資料夾中新增工具。你也可以依此架構為 resources 和 prompts 新增子目錄。

### -2- 建立工具

接著看看建立工具的樣子。工具得放在它的 *tool* 子目錄中，像這樣：

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # 待辦事項：加入 Pydantic，讓我們可以建立 AddInputModel 並驗證參數

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

這裡示範如何定義名稱、描述、使用 Pydantic 建立輸入 schema，以及定義工具被呼叫時會觸發的 handler。最後暴露 `tool_add`，這是一個字典包含以上屬性。

還有 *schema.py* 用來定義工具的輸入 schema：

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

我們也得建立 *__init__.py* 確保 tools 目錄被當成模組。此外要暴露此模組中的內容，如下：

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

隨著新增更多工具，可以一直在此檔案添加。

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

這裡定義了由以下屬性組成的字典：

- name，工具名稱。
- rawSchema，Zod schema，用途為驗證呼叫此工具的輸入。
- inputSchema，handler 會使用此 schema。
- callback，用來呼叫工具。

還有 `Tool` 類型用來把該字典轉換成 mcp server handler 可接受的型別，看起來如下：

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

然後 *schema.ts* 儲存所有工具的輸入 schema，目前只有一個，但未來可以依功能新增：

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

很好，接下來我們處理工具列出的部分。

### -3- 處理工具列出

要處理工具列出，我們需要設定一個請求處理器。下面是在 server 檔案加上的內容：

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

這裡用裝飾器 `@server.list_tools` 和實作函式 `handle_list_tools`。該函式要回傳工具清單，注意每個工具必須有名稱、描述和輸入 schema。

**TypeScript**

要設定工具列出的請求處理器，需要在伺服器呼叫 `setRequestHandler`，並傳入對應的 schema，這裡是 `ListToolsRequestSchema`。

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

太好了，工具列出功能完成，接著我們看怎麼呼叫工具。

### -4- 處理呼叫工具

呼叫工具需要另一個請求處理器，關注點是指定要呼叫的功能和其參數。

**Python**

使用裝飾器 `@server.call_tool` 並以 `handle_call_tool` 函式實作。函式中需解析工具名稱、參數，並確保參數符合規範。你可以在此函式驗證參數或放至實際工具中驗證。

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools 是以工具名稱作為鍵的字典
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

說明如下：

- 工具名稱存在輸入參數 `name`，參數在 `arguments` 字典中。
- 工具透過 `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` 呼叫。參數驗證發生在 `handler` 函式，失敗會拋出例外。

這樣，我們完整了解如何用低階伺服器列出和呼叫工具。

參考 [完整範例](./code/README.md)

## 作業

擴充你手上的程式碼，新增多個工具、資源和提示，並觀察你實際上只需要在 tools 目錄下新增檔案，而不必修改其他位置。

*未提供解答*

## 總結

這章節我們了解低階伺服器方式的運作，以及如何用這種架構持續擴充。我們也討論驗證問題，並展示如何使用驗證函式庫建立輸入驗證 schema。

## 接下來

- 下一步：[簡易認證](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議採用專業人工翻譯。我們對因使用本翻譯而產生的任何誤解或誤釋不承擔任何責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
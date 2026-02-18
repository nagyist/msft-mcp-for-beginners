# 進階伺服器使用

MCP SDK 裡面有兩種不同的伺服器類型，分別是你一般使用的伺服器，以及低階伺服器。通常你會用一般伺服器來新增功能，但在某些情況下，你會想用低階伺服器，例如：

- 更好的架構。用一般伺服器加低階伺服器當然也能打造出乾淨的架構，但可以說用低階伺服器會稍微容易一些。
- 功能可用性。有些進階功能只能搭配低階伺服器才能使用。你在後面的章節會看到，我們加入抽樣與誘導（elicitation）功能時就會用到。

## 一般伺服器 與 低階伺服器

下面是用一般伺服器建立 MCP Server 的範例

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

重點是你要明確新增想給伺服器的工具、資源或提示。這樣做沒有問題。

### 低階伺服器做法

但用低階伺服器時，需要以不同的思維來做處理，也就是說，不是註冊每個工具，而是為每種功能類型（工具、資源或提示）建立兩個處理器。比如對工具而言，只有兩個函式：

- 列出所有工具。會有一個函式負責處理所有列出工具的請求。
- 呼叫工具。這裡也只有一個函式負責呼叫某個工具。

聽起來工作量可能比較少，對吧？所以不需要註冊工具，只要確保列出工具時會列出它，且呼叫工具時能找到它即可。

我們來看一下目前的程式碼長怎麼樣：

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

這裡有一個函式回傳工具清單。tools清單中的每個項目都有 `name`、`description` 跟 `inputSchema` 等欄位，以符合回傳型別。這讓我們可以把工具與功能定義放到別處。我們現在可以在專案的 tools 資料夾裡建立所有工具，其他功能也是一樣，所以專案結構突然看起來像這樣：

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

很好，我們的架構看起來相當乾淨。

呼叫工具呢？是不是也是一樣的概念，只有一個處理器負責呼叫任意工具？沒錯，下面是對應的程式碼：

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

從上面程式碼可以看到，我們需要從請求中解析出要呼叫的工具名稱及參數，然後呼叫該工具。

## 用驗證來改進做法

目前你看到所有新增工具、資源與提示的註冊都可用這兩個處理器取代。接下來該做什麼？我們應該加入某種驗證機制，確保呼叫工具時參數是正確的。每個執行時環境各有解決方案，例如 Python 用 Pydantic，TypeScript 用 Zod。思路如下：

- 把建立功能（工具、資源、提示）的邏輯移到該功能專用的資料夾。
- 加入驗證來檢查輸入請求是否合乎規定，比如呼叫工具的參數格式要正確。

### 建立功能

創建一個功能時，需要為該功能建立一個檔案，並確定它擁有該功能必備的欄位。不同工具、資源、提示所需欄位會略有差異。

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

    # 待辦事項：加入 Pydantic，這樣我們就可以創建一個 AddInputModel 並驗證參數

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

- 在 *schema.py* 裡利用 Pydantic 建立 `AddInputModel` schema，擁有欄位 `a` 和 `b`。
- 嘗試將輸入請求解析成 `AddInputModel` 型別，如果參數不符合會造成錯誤：

   ```python
   # add.py
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

你可以選擇把這解析邏輯放在工具呼叫裡，或是放在處理器函式。

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

       // @ts-ignore
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

- 在處理所有工具呼叫的處理器中，嘗試用工具定義的 schema 來解析輸入請求：

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    如果成功，接著呼叫實際工具：

    ```typescript
    const result = await tool.callback(input);
    ```

如你所見，這種做法具備良好的架構：所有東西各有其位置，*server.ts* 是一個非常簡潔的檔案，只負責串接各種請求處理器，而各個功能放在它們各自的資料夾，例如 tools/、resources/ 或 prompts/。

很棒，我們接著來實作看看。

## 練習：創建低階伺服器

這個練習裡，我們要做：

1. 建立一個低階伺服器，負責工具的列出與呼叫。
2. 實作一個可擴充的架構。
3. 加入驗證，確保工具呼叫被正確驗證。

### -1- 建立架構

首先要打造一個架構，幫助我們隨著功能增加而容易擴充，大致長這樣：

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

這樣我們就建立了一個架構，可以輕鬆在 tools 資料夾新增新工具。你也可以同理新增 resources 和 prompts 子目錄。

### -2- 建立工具

接著來看看建立工具的範例。先要在 *tool* 子資料夾下建立一個工具，像這樣：

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # 使用 Pydantic 模型驗證輸入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # 待辦：加入 Pydantic，以便我們可以建立 AddInputModel 並驗證引數

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

我們看到這裡定義了名稱、說明、用 Pydantic 定義輸入 schema，以及呼叫工具時會呼叫的 handler。最後會公開 `tool_add` 這個字典，裡面包含這些屬性。

還有 *schema.py* 來定義工具使用的輸入 schema：

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

還要填寫 *__init__.py*，確保 tools 資料夾被視為模組，並把裡面的模組公開，寫法如下：

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

隨著加入更多工具，我們可以持續往這個檔案新增。

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

這裡我們建立一個字典，包含這些屬性：

- name：工具名稱。
- rawSchema：Zod schema，用於驗證來呼叫該工具的請求。
- inputSchema：處理器用的 schema。
- callback：用來呼叫工具的函式。

還有 `Tool`，用來將這個字典轉成 MCP 伺服器處理器可接受的型別，長這樣：

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

還有 *schema.ts*，裡面放每個工具使用的輸入 schema，目前只有一個，但可隨著工具增加而加入更多：

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

不錯，我們接下來來處理工具的列出請求。

### -3- 處理列出工具

為了處理列出工具需求，我們要設計一個請求處理器。要加到伺服器檔案的內容如下：

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

我們在函式上加上 `@server.list_tools` 裝飾器，並實作 `handle_list_tools` 函式，回傳工具清單。注意每個工具必須有 name、description 跟 inputSchema 欄位。

**TypeScript**

列出工具的請求處理器設置方式是，在伺服器呼叫 `setRequestHandler`，並帶入符合需求的 schema，這裡是 `ListToolsRequestSchema`。

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
// 為簡潔省略的代碼
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // 返回已註冊工具的列表
  return {
    tools: tools
  };
});
```

太好了，這樣我們完成列出工具的部分，接下來看看怎麼呼叫工具。

### -4- 處理工具呼叫

為了呼叫工具，我們要建立另一個請求處理器，專門處理指定要呼叫哪個功能及參數的請求。

**Python**

用裝飾器 `@server.call_tool` 並實作 `handle_call_tool` 函式。在函式中要解析出要呼叫的工具名稱、其參數，並確保參數對該工具是有效的。我們可以在這個函式做驗證，或者留給實際的工具函式驗證。

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
        # 調用該工具
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

流程說明：

- 工具名稱已包含在輸入參數 `name` 中，參數則是 `arguments` 字典。

- 用 `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` 呼叫工具。驗證參數在 `handler` 屬性的函式中進行，若失敗會拋出錯誤。

這樣，我們就完整了解用低階伺服器來列出與呼叫工具的流程。

另一個完整範例，[點這裡查看](./code/README.md)

## 作業

在給定的程式碼基礎上，新增多種工具、資源與提示，並觀察你只需在 tools 目錄新增檔案，其他地方不用動。

*不提供解答*

## 總結

本章節中，我們介紹了低階伺服器的做法，以及如何打造能持續擴充的乾淨架構。也講解了驗證的部分，並示範使用驗證程式庫創建輸入 schema。

## 接下來

- 下一章節: [簡易認證](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：  
本文件乃使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們致力於確保準確性，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件之母語版本應視為權威資料來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用此翻譯而產生的任何誤解或誤釋負責。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# 高级服务器使用

MCP SDK 中暴露了两种不同类型的服务器，普通服务器和低级服务器。通常情况下，你会使用普通服务器来添加功能。但在某些情况下，你会依赖低级服务器，例如：

- 更好的架构。使用普通服务器和低级服务器都可以创建干净的架构，但可以说使用低级服务器稍微更容易一些。
- 功能可用性。一些高级功能只能在低级服务器中使用。你将在后续章节中看到，我们会添加采样和引导功能。

## 普通服务器 vs 低级服务器

使用普通服务器创建 MCP Server 的示例：

**Python**

```python
mcp = FastMCP("Demo")

# 添加一个加法工具
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

// 添加一个加法工具
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

重点是你需要显式添加你想让服务器拥有的每个工具、资源或提示。这样也没有问题。

### 低级服务器方式

但是，当你使用低级服务器方式时，你需要以不同的思维方式来看待，具体来说，不是注册每个工具，而是为每种功能类型（工具、资源或提示）创建两个处理程序。例如，工具只有两个函数：

- 列出所有工具。一个函数负责所有尝试列出工具的操作。
- 处理调用所有工具。这里同样，只有一个函数处理调用某个工具的请求。

听起来可能少一些工作，对吧？所以，不是注册工具，而是确保工具在列出所有工具时会被列出，并且当有调用工具的请求时会被调用。

我们看一下代码现在是什么样子：

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
  // 返回已注册工具的列表
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

这里我们有一个返回功能列表的函数。工具列表里的每项现在都有 `name`、`description` 和 `inputSchema` 字段，以符合返回类型。这让我们可以把工具和功能定义放到其他地方。我们现在可以在 tools 文件夹里创建所有工具，资源和提示同理，这样你的项目结构就可以突然变成这样：

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

很好，我们的架构可以非常干净。

调用工具呢，是不是也是一个处理程序调用任意工具？是的，完全正确，代码如下：

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools 是一个以工具名称作为键的字典
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
    
    // 参数：request.params.arguments
    // 待办：调用工具，

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

从以上代码可见，我们需要解析被调用的工具以及传递的参数，然后执行调用。

## 使用验证改善方法

到目前为止，你已看到如何用每个功能类型两个处理程序代替添加工具、资源和提示的所有注册。接下来我们还需要做什么？我们应该添加某种形式的验证，确保调用工具时参数正确。每种运行时有自己的解决方案，例如 Python 使用 Pydantic，TypeScript 使用 Zod。思路如下：

- 将创建功能（工具、资源或提示）的逻辑移动到其专用文件夹。
- 添加一个方式验证进入请求，例如调用工具时验证请求。

### 创建一个功能

要创建一个功能，你需要创建该功能的文件，并确保它有该功能所必需的字段，不同的工具、资源和提示字段略有不同。

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
        # 使用 Pydantic 模型验证输入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # 待办事项：添加 Pydantic，以便我们可以创建 AddInputModel 并验证参数

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

这里展示如何做如下操作：

- 用 Pydantic 创建一个模式 `AddInputModel`，带有字段 `a` 和 `b`，保存在 *schema.py* 文件中。
- 试图将传入请求解析成 `AddInputModel` 类型，如果参数不匹配则会抛出错误：

   ```python
   # add.py
    try:
        # 使用 Pydantic 模型验证输入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

你可以选择把解析逻辑放在工具调用内部，或者放在处理函数中。

**TypeScript**

```typescript
// 服务器.ts
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

// 模式.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// 添加.ts
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

- 在处理所有工具调用的处理程序中，尝试将传入请求解析成对应工具定义的模式：

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

如果解析成功，则继续调用实际工具：

    ```typescript
    const result = await tool.callback(input);
    ```

如你所见，这种方法创建了很棒的架构，因为所有东西都各司其职，*server.ts* 是一个非常小的文件，只负责连接请求处理器，每个功能各自放在对应文件夹，如 tools/、resources/ 或 prompts/。

不错，让我们尝试下一步构建。

## 练习：创建一个低级服务器

在本练习中，我们将做以下内容：

1. 创建一个低级服务器，处理工具列出和调用。
1. 实现一个可以扩展的架构。
1. 添加验证，确保工具调用被正确验证。

### -1- 创建架构

首先，我们需要一个帮助我们随着功能增加而扩展的架构，示例如下：

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

我们现在建立了一个架构，确保我们可以轻松地在 tools 文件夹内添加新工具。你也可以按需为资源和提示添加子目录。

### -2- 创建一个工具

下面看看创建工具的过程。首先，需要在其 *tool* 子目录中创建：

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # 使用 Pydantic 模型验证输入
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # 待办事项：添加 Pydantic，以便我们可以创建 AddInputModel 并验证参数

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

这里展示了如何定义名称、描述，使用 Pydantic 定义一个输入模式，还有当工具被调用时会执行的处理函数。最后，我们暴露了 `tool_add`，这是一个包含所有这些属性的字典。

还有一个 *schema.py*，用于定义工具使用的输入模式：

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

我们还需要填充 *__init__.py*，确保 tools 目录作为模块，同时需要像下面这样导出里面的模块：

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

随着我们添加更多工具，可以继续向这个文件中添加。

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

这里我们创建了一个包含属性的字典：

- name，工具名称。
- rawSchema，Zod 模式，用于验证调用工具的请求。
- inputSchema，处理函数使用的模式。
- callback，用来调用这个工具。

还有 `Tool` 类型，用来将这个字典转换为 mcp 服务器处理器能接受的类型，示例如下：

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

以及 *schema.ts*，用于为各工具存储输入模式，目前只有一个模式，后续增加工具时可以继续添加：

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

很好，接下来处理工具列表的功能。

### -3- 处理工具列表

要处理列出工具请求，我们需要为此设置请求处理器。添加到服务器文件中的代码如下：

**Python**

```python
# 代码为简洁起见已省略
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

我们添加了装饰器 `@server.list_tools` 以及其实现函数 `handle_list_tools`。在函数中需要返回工具列表。注意每个工具需要包含名称、描述和 inputSchema。

**TypeScript**

要设置列出工具请求的处理器，需要在服务器上调用 `setRequestHandler`，并使用我们定义的模式，比如 `ListToolsRequestSchema`。

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
// 代码省略以简洁起见
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // 返回已注册工具的列表
  return {
    tools: tools
  };
});
```

很好，现在已经解决了工具列表部分，接下来看看怎么调用工具。

### -4- 处理调用工具

调用工具时，我们需要设置另一个请求处理器，处理指定调用哪个功能及用哪些参数的请求。

**Python**

我们使用 `@server.call_tool` 装饰器，并实现一个函数如 `handle_call_tool`。函数内部需要解析工具名称、参数，并确保参数对该工具有效。参数验证可以在该函数里做，也可以在具体工具里做。

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools 是一个以工具名称为键的字典
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # 调用该工具
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

说明如下：

- 工具名已作为输入参数 `name`，参数作为 `arguments` 字典传入。

- 工具调用用 `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`，参数验证发生在指向函数的 `handler` 属性，如果验证失败将抛出异常。

这样，我们就全面理解了如何用低级服务器进行工具的列出和调用。

查看[完整示例](./code/README.md)

## 作业

用多个工具、资源和提示扩展你已有的代码，并反思你会发现你只需要在 tools 目录添加文件，而无需在其他地方添加。

*不提供解决方案*

## 总结

本章我们了解了低级服务器方式是如何工作的，及它如何帮助构建干净且易扩展的架构。我们也讨论了验证，展示了如何用验证库创建模式实现输入验证。

## 下一步

- 下一节: [简单认证](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：  
本文件由人工智能翻译服务[Co-op Translator](https://github.com/Azure/co-op-translator)进行翻译。虽然我们力求准确，但请注意自动翻译可能包含错误或不准确之处。应以原始语言的文档为权威来源。对于关键信息，建议采用专业人工翻译。因使用本翻译而产生的任何误解或误读，我们不承担任何责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
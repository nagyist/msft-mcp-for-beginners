# Advanced server usage

Dem get two kain servers wey dey inside the MCP SDK, your normal server and the low-level server. Normally, you go dey use the regular server to add features enter am. But for some cases, you go wan rely on the low-level server like:

- Better architecture. E possible to create clean architecture with both the regular server and low-level server but e fit be say e easy small better with low-level server.
- Feature availability. Some advanced features fit only use low-level server. You go see dis one for later chapters as we dey add sampling and elicitation.

## Regular server vs low-level server

Na how to create MCP Server be dis with regular server

**Python**

```python
mcp = FastMCP("Demo")

# Add wan add tool
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

// Add wan plus tool
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

The koko be say you go explicitly add every tool, resource or prompt wey you want the server to get. Nothing wrong with that.

### Low-level server approach

But if you use low-level server approach, you go need reason am different. Instead of registering every tool, you go create two handlers per feature type (tools, resources or prompts). So for example tools get only two functions like this:

- Listing all tools. One function go dey in charge of all attempts to list tools.
- Handle calling all tools. Here too, one function go dey handle calls to tool.

E sound like less work abi? So instead of registering tool, I go only need make sure say tool dey listed when I list all tools and say e go call am when request dey come to call tool.

Make we look how the code be now:

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
  // Return di list of tools wey dem don register
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

Here we get function wey dey return list of features. Each tools list entry get fields like `name`, `description` and `inputSchema` to fit the return type. Dis one make us fit put our tools and feature definition for another place. We fit create all our tools for tools folder and same for all your features so your project fit organize like dis:

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

That one good, our architecture fit clean well well.

How about calling tools, na the same idea here, one handler to call any tool? Yes, na im be dat, here be the code:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools na dictionary wey get tool names as keys
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
    
    // args: request.params.arguments
    // TODO make call to di tool,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

As you fit see from code wey dey above, we need parse out the tool whey we wan call, and the arguments wey e go get, then we proceed to call the tool.

## Improving the approach with validation

So far, you don see how all your registrations to add tools, resources and prompts fit replace with these two handlers per feature type. Wetin else we need do? We suppose add some kind validation to make sure say tool dey call with correct arguments. Each runtime get their own way of doing am, for example Python dey use Pydantic and TypeScript dey use Zod. The plan be say we go do dis:

- Move the logic to create feature (tool, resource or prompt) to the folder wey e suppose dey.
- Add way to validate incoming request wey ask to call tool for example.

### Create a feature

To create a feature, we go need create file for that feature and make sure say e get the fields wey the feature need. The fields fit differ small between tools, resources and prompts.

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
        # Check input correct wit Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: add Pydantic, make we fit create AddInputModel and check args correct

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Here you fit see how we dey do dis:

- Create schema using Pydantic `AddInputModel` with fields `a` and `b` for file *schema.py*.
- Try parse the incoming request to be like `AddInputModel`, if parameter no match e go crash:

   ```python
   # add.py
    try:
        # Check if di input dey correct wit Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

You fit decide whether put this parsing logic inside the tool call itself or for the handler function.

**TypeScript**

```typescript
// server.ts
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

// schema.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// add.ts
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

- For handler wey dey handle all tool calls, now we try parse the incoming request using the tool defined schema:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    if e work, then we go call the real tool:

    ```typescript
    const result = await tool.callback(input);
    ```

As you fit see, dis approach fit create better architecture because everything get im place, the *server.ts* na small small file wey just wire up the request handlers and each feature dey their correct folder i.e tools/, resources/ or /prompts.

Good, make we try build this one now.

## Exercise: Creating a low-level server

For this exercise, we go do these ones:

1. Create low-level server wey go handle listing of tools and calling tools.
2. Implement architecture wey you fit build upon.
3. Add validation to make sure your tool calls dey properly validated.

### -1- Create an architecture

The first thing we go do na create architecture wey go help us scale as we add more features, this na how e be:

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

Now we don set up architecture wey make am easy to add new tools for tools folder. Feel free to follow dis one to add subdirectories for resources and prompts.

### -2- Creating a tool

Make we see how to create tool go be. First, e need dey inside its *tool* subdirectory like dis:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Check di input wit Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: add Pydantic, make we fit create AddInputModel and check args right

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Wetin we see here na how we define name, description, input schema using Pydantic and handler wey go run when tool dey call. Last last, we expose `tool_add` wey be dictionary wey hold all these properties.

Another one na *schema.py* wey dem use define input schema wey our tool dey use:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

We also need to add to *__init__.py* to make sure the tools directory dey treated as module. Plus, we need expose the modules inside am like dis:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

We fit continue add to this file as we add more tools.

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

Here we create dictionary wey get properties:

- name, na tool name be dis.
- rawSchema, na Zod schema, e go validate incoming requests to call tool.
- inputSchema, handler go use dis schema.
- callback, dis one go call the tool.

Dem get `Tool` wey dey convert this dictionary to type wey mcp server handler fit accept and e be like dis:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

And dem get *schema.ts* where dem dey store input schemas for each tool, e look like dis with just one schema so far but as we add tools, we go add more entries:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Good, make we continue with handling the tool listing next.

### -3- Handle tool listing

Next, to handle list tools, we go set request handler for am. This na wetin we go add for the server file:

**Python**

```python
# code dem no put for the sake of shortness
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

Here' we add decorator `@server.list_tools` and function `handle_list_tools`. Inside here, we go produce tools list. Note say every tool need get name, description and inputSchema.

**TypeScript**

To set request handler for listing tools, we go call `setRequestHandler` on the server with schema wey fit wetin we dey try do, for this case na `ListToolsRequestSchema`.

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
// code wey dem comot to make am short
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Bring back di list of tools wey dem don register
  return {
    tools: tools
  };
});
```

Good, now we don solve the listing tools part, make we see how we go dey call tools next.

### -4- Handle calling a tool

To call tool, we go set another request handler, this time e go focus on request wey specify which feature to call and with which arguments.

**Python**

Make we use decorator `@server.call_tool` and implement am with function like `handle_call_tool`. Inside function we go parse tool name, arguments and make sure arguments correct for the tool. We fit validate arguments here or for the real tool downstream.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools na dictionary wey get tool names as keys
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # make you run the tool
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Dis na wetin dey happen:

- Our tool name already dey as input parameter `name` which dey true for our arguments wey dey form `arguments` dictionary.

- Tool dey call with `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. The validation for arguments dey happen inside `handler` property wey point to function, if e fail e go raise exception.

There, now we get full understanding of listing and calling tools using low-level server.

See the [full example](./code/README.md) here

## Assignment

Extend the code wey dem give you with plenty tools, resources and prompts and notice how you only need add files for tools directory and no where else.

*No solution given*

## Summary

For this chapter, we don see how low-level server approach dey work and how e fit help us create clean architecture we fit keep build on top. We talk about validation and you see how to use validation libraries to create schemas for input validation.

## What's Next

- Next: [Simple Authentication](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:  
Dis document na translate wey AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator) do. Even though we dey try make am correct, abeg make you sabi say automated translation fit get mistake or no too correct. Di original document wey dey im own language na di correct one wey you go trust pass. If na very important matter, make person wey sabi human translation do am. We no go take any blame if person no understand well or if dem use dis translation do anyhow.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
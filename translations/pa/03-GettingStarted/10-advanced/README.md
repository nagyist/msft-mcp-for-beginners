# ਉੱਨਤ ਸਰਵਰ ਵਰਤੋਂ

MCP SDK ਵਿੱਚ ਦੋ ਵੱਖ-ਵੱਖ ਕਿਸਮਾਂ ਦੇ ਸਰਵਰ ਪ੍ਰਦਰਸ਼ਿਤ ਕੀਤੇ ਗਏ ਹਨ, ਤੁਹਾਡਾ ਆਮ ਸਰਵਰ ਅਤੇ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ। ਆਮ ਤੌਰ 'ਤੇ, ਤੁਸੀਂ ਇਸ ਵਿਚ ਫੀਚਰ ਸ਼ਾਮਲ ਕਰਨ ਲਈ ਆਮ ਸਰਵਰ ਦੀ ਵਰਤੋਂ ਕਰੋਗੇ। ਕੁਝ ਮਾਮਲਿਆਂ ਵਿੱਚ, ਤੁਸੀਂ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ 'ਤੇ ਨਿਰਭਰ ਕਰਨਾ ਚਾਹੁੰਦੇ ਹੋ ਜਿਵੇਂ ਕਿ:

- ਬਿਹਤਰ ਢਾਂਚਾ। ਆਮ ਸਰਵਰ ਅਤੇ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਦੋਹਾਂ ਨਾਲ ਇੱਕ ਸਾਫ਼ ਢਾਂਚਾ ਬਣਾਉਣਾ ਸੰਭਵ ਹੈ ਪਰ ਇਹ ਕਹਿ ਸਕਦੇ ਹਾਂ ਕਿ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਨਾਲ ਇਹ ਥੋੜ੍ਹਾ ਆਸਾਨ ਹੁੰਦਾ ਹੈ।
- ਫੀਚਰ ਉਪਲੱਬਧਤਾ। ਕੁਝ ਉੱਨਤ ਫੀਚਰ ਸਿਰਫ਼ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਨਾਲ ਹੀ ਵਰਤੇ ਜਾ ਸਕਦੇ ਹਨ। ਤੁਸੀਂ ਬਾਦ ਦੇ ਅਧਿਆਇ ਵਿੱਚ ਇਸ ਨੂੰ ਵੇਖੋਗੇ ਜਦੋਂ ਅਸੀਂ ਸੈਂਪਲਿੰਗ ਅਤੇ ਪ੍ਰਭਾਵ ਪ੍ਰਕ੍ਰਿਆ ਸ਼ਾਮਲ ਕਰਾਂਗੇ।

## ਆਮ ਸਰਵਰ ਬਨਾਮ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ

ਇਹ ਹੈ ਕਿ MCP ਸਰਵਰ ਬਣਾਉਣ ਦਾ ਸਧਾਰਣ ਸਰਵਰ ਨਾਲ ਕਿਵੇਂ ਦਿਖਾਈ ਦਿੰਦਾ ਹੈ

**Python**

```python
mcp = FastMCP("Demo")

# ਇੱਕ ਜੋੜਨ ਵਾਲਾ ਸੰਦ ਸ਼ਾਮਲ ਕਰੋ
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

// ਇੱਕ ਜੁੜਾਈ ਦਾ ਸੰਦ ਸ਼ਾਮਿਲ ਕਰੋ
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

ਮੁੱਦਾ ਇਹ ਹੈ ਕਿ ਤੁਸੀਂ ਖੁਦਹੀ ਹਰੇਕ ਟੂਲ, ਸਰੋਤ ਜਾਂ ਪ੍ਰੌਮਪਟ ਜੁੜਦੇ ਹੋ ਜੋ ਤੁਸੀਂ ਚਾਹੁੰਦੇ ਹੋ ਕਿ ਸਰਵਰ ਕੋਲ ਹੋਵੇ। ਇਸ ਵਿੱਚ ਕੋਈ ਗਲਤ ਨਹੀਂ।

### ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਪদ্ধਤੀ

ਹਾਲਾਂਕਿ, ਜਦੋਂ ਤੁਸੀਂ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਪদ্ধਤੀ ਵਰਤਦੇ ਹੋ ਤਾਂ ਤੁਹਾਨੂੰ ਇਹ ਬਿਨਾਂ soਚਣਾ ਪੈਂਦਾ ਹੈ ਕਿ ਤੁਸੀਂ ਹਰੇਕ ਟੂਲ ਨੂੰ ਰਜਿਸਟਰ ਕਰਨ ਦੀ ਬਜਾਏ, ਤੁਹਾਨੂੰ ਹਰ ਫੀਚਰ ਕਿਸਮ (ਟੂਲ, ਸਰੋਤ ਜਾਂ ਪ੍ਰੌਮਪਟ) ਲਈ ਦੋ ਹੈਂਡਲਰ ਬਣਾਉਣੇ ਪੈਂਦੇ ਹਨ। ਉਦਾਹਰਣ ਵਜੋਂ ਟੂਲਜ਼ ਲਈ ਸਿਰਫ਼ ਇਹ ਦੋ ਫੰਕਸ਼ਨ ਹੋਣਗੇ:

- ਸਾਰੇ ਟੂਲਜ਼ ਦੀ ਸੂਚੀ ਬਣਾ ਰਹੇ ਹਨ। ਇੱਕ ਫੰਕਸ਼ਨ ਸਾਰੇ ਟੂਲਜ਼ ਦੀ ਸੂਚੀ ਬਣਾਉਣ ਲਈ ਜ਼ਿੰਮੇਵਾਰ ਹੋਵੇਗਾ।
- ਸਾਰੇ ਟੂਲਜ਼ ਨੂੰ ਕਾਲ ਕਰਨ ਦਾ ਹੈਂਡਲ ਕਰਨ ਵਾਲਾ। ਇੱਥੇ ਵੀ ਸਿਰਫ਼ ਇੱਕ ਫੰਕਸ਼ਨ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਦੀ ਸੰਭਾਲ ਕਰਦਾ ਹੈ।

ਇਹ ਸ਼ਾਇਦ ਘੱਟ ਕੰਮ ਵਰਗਾ ਲੱਗਦਾ ਹੈ? ਇਸ ਲਈ ਟੂਲ ਨੂੰ ਰਜਿਸਟਰ ਕਰਨ ਦੀ ਬਜਾਏ, ਮੈਨੂੰ ਸਿਰਫ ਇਹ ਯਕੀਨੀ ਬਣਾਉਣਾ ਹੈ ਕਿ ਜਦੋਂ ਮੈਂ ਸਾਰੇ ਟੂਲਸ ਦੀ ਸੂਚੀ ਦਿਖਾਂ ਤਾਂ ਟੂਲ ਲਿਸਟ ਵਿੱਚ ਹੁੰਦਾ ਹੈ ਅਤੇ ਜਦੋਂ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਲਈ ਬੇਨਤੀ ਆਉਂਦੀ ਹੈ ਤਾਂ ਉਨ੍ਹਾਂ ਨੂੰ ਕਾਲ ਕੀਤਾ ਜਾਂਦਾ ਹੈ।

ਆਓ ਵੇਖੀਏ ਹੁਣ ਕੋਡ ਕਿਵੇਂ ਦਿਖਦਾ ਹੈ:

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
  // ਰਜਿਸਟਰਡ ਟੂਲਜ਼ ਦੀ ਸੂਚੀ ਵਾਪਸ ਕਰੋ
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

ਇਥੇ ਸਾਡੇ ਕੋਲ ਹੁਣ ਇੱਕ ਫੰਕਸ਼ਨ ਹੈ ਜੋ ਫੀਚਰਾਂ ਦੀ ਸੂਚੀ ਵਾਪਸ ਕਰਦਾ ਹੈ। ਟੂਲਜ਼ ਲਿਸਟ ਵਿੱਚ ਹਰ ਐਨਟਰੀ ਲਈ ਹੁਣ `name`, `description` ਅਤੇ `inputSchema` ਵਰਗੇ ਖੇਤਰ ਹੁੰਦੇ ਹਨ ਜੋ ਵਾਪਸੀ ਦੇ ਕਿਸਮ ਨਾਲ ਮੇਲ ਖਾਂਦੇ ਹਨ। ਇਹ ਸਾਨੂੰ ਸਾਡੇ ਟੂਲਜ਼ ਅਤੇ ਫੀਚਰ ਪਰਿਭਾਸ਼ਾ ਨੂੰ ਕਿਤੇ ਹੋਰ ਰੱਖਣ ਦੀ ਆਗਿਆ ਦਿੰਦਾ ਹੈ। ਅਸੀਂ ਹੁਣ ਸਾਰੇ ਟੂਲਜ਼ ਨੂੰ tools ਫੋਲਡਰ ਵਿੱਚ ਬਣਾਉਣ ਲਈ ਤਿਆਰ ਹਾਂ ਅਤੇ ਤੁਹਾਡੇ ਸਾਰੇ ਫੀਚਰਾਂ ਲਈ ਵੀ ਇਹੀ ਹੈ ਤਾਂ ਤੁਹਾਡਾ ਪ੍ਰੋਜੈਕਟ ਅਚਾਨਕ ਇਸ ਤਰ੍ਹਾਂ ਇਕਥਾ ਹੋ ਸਕਦਾ ਹੈ:

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

ਬਹੁਤ ਵਧੀਆ, ਸਾਡਾ ਢਾਂਚਾ ਕਾਫੀ ਸਾਫ਼ ਬਣ ਸਕਦਾ ਹੈ।

ਟੂਲਜ਼ ਨੂੰ ਕਾਲ ਕਰਨ ਦਾ ਕੀ ਹਾਲ ਹੈ, ਕੀ ਇਹੋ ਜਿਹਾ ਵਿਚਾਰ ਹੈ ਕਿ ਇੱਕ ਹੈਂਡਲਰ ਹੈ ਜੋ ਕਿਸੇ ਵੀ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਦਾ ਹੈ? ਹਾਂ, ਬਿਲਕੁਲ, ਇਹ ਉਸਦਾ ਕੋਡ ਹੈ:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ਇੱਕ ਡਿਕਸ਼ਨਰੀ ਹੈ ਜਿਸ ਵਿੱਚ ਟੂਲ ਦੇ ਨਾਮ ਕੁੰਜੀਆਂ ਵਜੋਂ ਹਨ
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
    
    // ਆਰਗਸ: request.params.arguments
    // TODO ਟੂਲ ਨੂੰ ਕਾਲ ਕਰੋ,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

ਤੁਹਾਨੂੰ ਉਪਰੋਕਤ ਕੋਡ ਤੋਂ ਪਤਾ ਲੱਗਦਾ ਹੈ ਕਿ ਸਾਨੂੰ ਕਾਲ ਕਰਣ ਲਈ ਟੂਲ ਅਤੇ ਅਰਗਿਊਮੈਂਟ ਨਿਕਾਲਣੇ ਪੈਂਦੇ ਹਨ, ਫਿਰ ਅਸੀਂ ਟੂਲ ਕਾਲ ਕਰਨ ਜਾ ਰਹੇ ਹਾਂ।

## ਪੁਸ਼ਟੀਕਰਨ ਨਾਲ ਪਧਤੀ ਸੁਧਾਰਨਾ

ਹੁਣ ਤੱਕ, ਤੁਸੀਂ ਵੇਖਿਆ ਕਿ ਸਾਰੇ ਟੂਲ, ਸਰੋਤ ਅਤੇ ਪ੍ਰੌਮਪਟ ਜੋੜਨ ਲਈ ਤੁਹਾਡੀਆਂ ਸਾਰੀਆਂ ਰਜਿਸਟ੍ਰੇਸ਼ਨਾਂ ਨੂੰ ਇਹ ਦੋ ਹੈਂਡਲਰ ਪ੍ਰਤੀ ਫੀਚਰ ਕਿਸਮ ਨਾਲ ਬਦਲਿਆ ਜਾ ਸਕਦਾ ਹੈ। ਹੋਰ ਕੀ ਕਰਨ ਦੀ ਲੋੜ ਹੈ? ਸਾਨੂੰ ਕੁਝ ਤਰ੍ਹਾਂ ਦੀ ਪੁਸ਼ਟੀਕਰਨ ਸ਼ਾਮਲ ਕਰਨੀ ਚਾਹੀਦੀ ਹੈ ਜਿਸ ਨਾਲ ਇਹ ਪੱਕਾ ਹੋਵੇ ਕਿ ਟੂਲ ਸਹੀ ਅਰਗਿਊਮੈਂਟਸ ਨਾਲ ਕਾਲ ਕੀਤਾ ਜਾ ਰਿਹਾ ਹੈ। ਹਰ ਰਨਟਾਈਮ ਲਈ ਇਸਦਾ ਆਪਣਾ ਹੱਲ ਹੈ, ਉਦਾਹਰਣ ਵਜੋਂ Python ਵਿੱਚ Pydantic ਅਤੇ TypeScript ਵਿੱਚ Zod ਵਰਤਿਆ ਜਾਂਦਾ ਹੈ। ਵਿਚਾਰ ਇਹ ਹੈ ਕਿ ਅਸੀਂ ਇਨ੍ਹਾਂ ਗੱਲਾਂ ਨੂੰ ਕਰੀਏ:

- ਕਿਸੇ ਫੀਚਰ (ਟੂਲ, ਸਰੋਤ ਜਾਂ ਪ੍ਰੌਮਪਟ) ਦੀ ਬਣਤਰ ਨੂੰ ਉਸਦੇ ਸਮਰਪਿਤ ਫੋਲਡਰ ਵਿੱਚ ਸ਼ਿਫਟ ਕਰੋ।
- ਆ ਰਹੀ ਬੇਨਤੀ ਦਾ ਪੁਸ਼ਟੀਕਰਨ ਕਰੋ ਜਿਵੇਂ ਕਿ ਊਦਾਹਰਣ ਵਜੋਂ ਟੂਲ ਕਾਲ ਕਰਨ ਲਈ ਬੇਨਤੀ।

### ਇੱਕ ਫੀਚਰ ਬਣਾਉਣਾ

ਫੀਚਰ ਬਣਾਉਣ ਲਈ, ਸਾਨੂੰ ਉਸ ਫੀਚਰ ਲਈ ਇੱਕ ਫਾਈਲ ਬਣਾਉਣੀ ਪਵੇਗੀ ਤੇ ਇਹ ਯਕੀਨੀ ਬਣਾਉਣਾ ਪਵੇਗਾ ਕਿ ਇਸ ਵਿੱਚ ਹਰੇਕ ਜਰੂਰੀ ਖੇਤਰ ਮੌਜੂਦ ਹਨ। ਇਹ ਖੇਤਰ ਟੂਲਜ਼, ਸਰੋਤ ਅਤੇ ਪ੍ਰੌਮਪਟ ਵਿੱਚ ਥੋੜ੍ਹਾ ਫਰਕ ਹੁੰਦਾ ਹੈ।

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
        # ਇਨਪੁੱਟ ਨੂੰ ਪਾਈਡੈਂਟਿਕ ਮਾਡਲ ਦੀ ਵਰਤੋਂ ਨਾਲ ਵੈਧਤਾ ਕਰੋ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: ਪਾਈਡੈਂਟਿਕ ਜੋੜੋ, ਤਾਂ ਜੋ ਅਸੀਂ AddInputModel ਬਣਾ ਸਕੀਏ ਅਤੇ args ਦੀ ਚੈੱਕਿੰਗ ਕਰ ਸਕੀਏ

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ਇਥੇ ਤੁਸੀਂ ਦੇਖ ਸਕਦੇ ਹੋ ਕਿ ਅਸੀਂ ਕੀ ਕਰਦੇ ਹਾਂ:

- Pydantic ਦੇ ਨਾਲ `AddInputModel` ਸਕੀਮਾ ਬਣਾਉਂਦੇ ਹਾਂ ਜੋ ਫਾਈਲ *schema.py* ਵਿੱਚ `a` ਅਤੇ `b` ਖੇਤਰ ਰੱਖਦਾ ਹੈ।
- ਆ ਰਹੀ ਬੇਨਤੀ ਨੂੰ `AddInputModel` ਟਾਈਪ ਵਿੱਚ ਪਾਰਸ ਕਰਨ ਦੀ ਕੋਸ਼ਿਸ਼ ਕਰਦੇ ਹਾਂ, ਜੇ ਪੈਰਾਮੀਟਰਾਂ ਵਿੱਚ ਮਿਸ਼ਮੇਲ ਹੈ ਤਾਂ ਇਹ ਕ੍ਰੈਸ਼ ਕਰ ਜਾਵੇਗਾ:

   ```python
   # add.py
    try:
        # ਪਾਈਡੈਂਟਿਕ ਮਾਡਲ ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਇਨਪੁੱਟ ਦੀ ਪੁਸ਼ਟੀ ਕਰੋ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

ਤੁਸੀਂ ਇਹ ਧਾਰਣਾ ਕਰ ਸਕਦੇ ਹੋ ਕਿ ਇਹ ਪਾਰਸਿੰਗ ਲਾਜਿਕ ਟੂਲ ਕਾਲ ਖੁਦ ਜਾਂ ਹੈਂਡਲਰ ਫੰਕਸ਼ਨ ਵਿੱਚ ਰੱਖੀ ਜਾਵੇ।

**TypeScript**

```typescript
// ਸਰਵਰ.ts
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

// ਸਕੀਮਾ.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// ਜੋੜੋ.ts
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

- ਸਾਰੇ ਟੂਲ ਕਾਲ ਹੈਂਡਲਰ ਵਿੱਚ, ਹੁਣ ਅਸੀਂ ਆ ਰਹੀ ਬੇਨਤੀ ਨੂੰ ਉਸ ਟੂਲ ਦੇ ਪਰਿਭਾਸ਼ਿਤ ਸਕੀਮਾ ਵਿੱਚ ਪਾਰਸ ਕਰਨ ਦੀ ਕੋਸ਼ਿਸ਼ ਕਰਦੇ ਹਾਂ:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ਜੇ ਇਹ ਸਫਲ ਹੁੰਦਾ ਹੈ ਤਾਂ ਅਸੀਂ ਹਕੀਕਤ ਵਿੱਚ ਟੂਲ ਕਾਲ ਕਰਦੇ ਹਾਂ:

    ```typescript
    const result = await tool.callback(input);
    ```

ਤੁਸੀਂ ਵੇਖ ਸਕਦੇ ਹੋ ਕਿ ਇਹ ਪਧਤੀ ਇੱਕ ਵਧੀਆ ਢਾਂਚਾ ਬਣਾਉਂਦੀ ਹੈ ਕਿਉਂਕਿ ਹਰ ਚੀਜ਼ ਦੀ ਆਪਣੀ ਜਗ੍ਹਾ ਹੁੰਦੀ ਹੈ, *server.ts* ਇੱਕ ਬਹੁਤ ਛੋਟੀ ਫਾਈਲ ਹੈ ਜੋ ਸਿਰਫ ਬੇਨਤੀ ਹੈਂਡਲਰਾਂ ਨੂੰ ਵਾਇਰ ਕਰਦਾ ਹੈ ਅਤੇ ਹਰ ਫੀਚਰ ਆਪਣੇ ਅਨੁਕੂਲ ਫੋਲਡਰ ਵਿੱਚ ਹੁੰਦਾ ਹੈ ਜਿਵੇਂ tools/, resources/ ਜਾਂ /prompts।

ਵਧੀਆ, ਆਓ ਹੁਣ ਇਸਨੂੰ ਬਣਾਉਣ ਦੀ ਕੋਸ਼ਿਸ਼ ਕਰੀਏ।

## ਅਭਿਆਸ: ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਬਣਾਉਣਾ

ਇਸ ਅਭਿਆਸ ਵਿੱਚ, ਅਸੀਂ ਇਹ ਕਰਨਗੇ:

1. ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਬਣਾਉਣਾ ਜੋ ਟੂਲਜ਼ ਦੀ ਲਿਸਟ ਅਤੇ ਟੂਲਜ਼ ਨੂੰ ਕਾਲ ਕਰਨ ਨੂੰ ਸੰਭਾਲੇ।
1. ਇੱਕ ਐਸਾ ਢਾਂਚਾ ਲਾਗੂ ਕਰਨਾ ਜਿਸ 'ਤੇ ਤੁਸੀਂ ਅੱਗੇ ਬਣਾਉ ਸਕੋ।
1. ਪੁਸ਼ਟੀਕਰਨ ਸ਼ਾਮਲ ਕਰਨਾ ਤਾਂ ਜੋ ਤੁਹਾਡੇ ਟੂਲ ਕਾਲ ਸਹੀ ਤਰ੍ਹਾਂ ਪੁਸ਼ਟੀ ਹੋ ਜਾਣ।

### -1- ਇੱਕ ਢਾਂਚਾ ਬਣਾਉਣਾ

ਸਭ ਤੋਂ ਪਹਿਲੀ ਗੱਲ ਜੋ ਸਾਨੂੰ ਚਿੰਤਾ ਕਰਨੀ ਹੈ ਉਹ ਐਸਾ ਢਾਂਚਾ ਹੈ ਜੋ ਸਾਨੂੰ ਵਧਦੇ ਫੀਚਰਾਂ ਨੂੰ ਜੋੜਨ ਵਿੱਚ ਸਹਾਇਤਾ ਕਰੇ, ਇਹ ਇਸ ਤਰ੍ਹਾਂ ਦਿਖਦਾ ਹੈ:

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

ਹੁਣ ਅਸੀਂ ਇੱਕ ਐਸਾ ਢਾਂਚਾ ਤਿਆਰ ਕੀਤਾ ਹੈ ਜੋ ਇਹ ਯਕੀਨੀ ਬਣਾਉਂਦਾ ਹੈ ਕਿ ਅਸੀਂ ਆਸਾਨੀ ਨਾਲ ਟੂਲਜ਼ ਜੋੜ ਸਕਦੇ ਹਾਂ tools ਫੋਲਡਰ ਵਿੱਚ। ਇਹੇ ਤਰ੍ਹਾਂ ਤੁਸੀਂ resources ਅਤੇ prompts ਲਈ ਵੀ ਸਬਡਾਇਰੈਕਟਰੀਜ਼ ਜੋੜ ਸਕਦੇ ਹੋ।

### -2- ਇੱਕ ਟੂਲ ਬਣਾਉਣਾ

ਆਓ ਦੇਖੀਏ ਟੂਲ ਬਣਾਉਣਾ ਕਰਮਵਾਰ ਕਿਵੇਂ ਹੁੰਦਾ ਹੈ। ਸਭ ਤੋਂ ਪਹਿਲਾਂ, ਇਹ ਆਪਣੇ *tool* ਸਬਡਾਇਰੈਕਟਰੀ ਵਿੱਚ ਬਣਾਇਆ ਜਾਵੇਗਾ ਜਿਵੇਂ ਕਿ:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # ਪਾਇਡੈਂਟਿਕ ਮਾਡਲ ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਇਨਪੁੱਟ ਦੀ ਪੁਸ਼ਟੀ ਕਰੋ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # ਕਰਨ ਵਾਲਾ ਕੰਮ: ਪਾਇਡੈਂਟਿਕ ਸ਼ਾਮਲ ਕਰੋ, ਤਾਂ ਜੋ ਅਸੀਂ AddInputModel ਬਣਾ ਸਕੀਏ ਅਤੇ args ਦੀ ਪੁਸ਼ਟੀ ਕਰ ਸਕੀਏ

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ਇਥੇ ਅਸੀਂ ਵੇਖ ਰਹੇ ਹਾਂ ਕਿ ਕਿਵੇਂ ਨਾਮ, ਵੇਰਵਾ, Pydantic ਵਰਤ ਕੇ ਇਨਪੁਟ ਸਕੀਮਾ ਅਤੇ ਇੱਕ ਹੈਂਡਲਰ ਬਣਾਇਆ ਜਾਂਦਾ ਹੈ ਜੋ ਟੂਲ ਕਾਲ ਹੋਣ 'ਤੇ ਚੱਲੇਗਾ। ਆਖਰ ਵਿੱਚ, ਅਸੀਂ `tool_add` ਜੋ ਕਿ ਇੱਕ ਡਿਕਸ਼ਨਰੀ ਹੈ ਜੋ ਇਹਨਾਂ ਸਾਰੀਆਂ ਗੁਣਾਂ ਨੂੰ ਰੱਖਦੀ ਹੈ, ਨੂੰ ਇੱਕਸਪੋਜ਼ ਕਰਦੇ ਹਾਂ।

ਇੱਕ *schema.py* ਵੀ ਹੈ ਜੋ ਸਾਡੇ ਟੂਲ ਲਈ ਇਨਪੁਟ ਸਕੀਮਾ ਪਰਿਭਾਸ਼ਿਤ ਕਰਦਾ ਹੈ:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

ਸਾਨੂੰ *__init__.py* ਨੂੰ ਭੀ ਭਰਨਾ ਪਵੇਗਾ ਤਾਂ ਜੋ tools ਡਾਇਰੈਕਟਰੀ ਨੂੰ ਮੋਡੀਊਲ ਵਜੋਂ ਮੰਨਿਆ ਜਾਵੇ। ਇਲਾਵਾ, ਸਾਨੂੰ ਇਸ ਦੇ ਅੰਦਰ ਮੌਜੂਦ ਮੋਡੀਊਲਾਂ ਨੂੰ ਐਸਿਈਐਸ ਕਰਨਾ ਪਵੇਗਾ ਜਿਵੇਂ:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

ਅਸੀਂ ਨਵੇਂ ਟੂਲ ਸ਼ਾਮਲ ਕਰਦੇ ਰਹਿਣ ਦੇ ਨਾਲ ਇਸ ਫਾਈਲ ਨੂੰ ਉਦਾਹਰਣ ਵਜੋਂ ਭਰਦੇ ਰਹਿ ਸਕਦੇ ਹਾਂ।

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

ਇਥੇ ਅਸੀਂ ਪ੍ਰਾਪਰਟੀਜ਼ ਵਾਲੀ ਡਿਕਸ਼ਨਰੀ ਬਣਾਉਂਦੇ ਹਾਂ:

- name, ਇਹ ਟੂਲ ਦਾ ਨਾਮ ਹੈ।
- rawSchema, ਇਹ ਜ਼ੋਡ ਸਕੀਮਾ ਹੈ, ਜੋ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਵਾਲੀਆਂ ਆਗਮਨ ਬੇਨਤੀਆਂ ਦੀ ਪੁਸ਼ਟੀ ਕਰਨ ਲਈ ਵਰਤੀ ਜਾਵੇਗੀ।
- inputSchema, ਇਹ ਸਕੀਮਾ ਹੈ ਜੋ ਹੈਂਡਲਰ ਦੁਆਰਾ ਵਰਤੀ ਜਾਵੇਗੀ।
- callback, ਇਹ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਲਈ ਵਰਤੀ ਜਾਂਦੀ ਹੈ।

ਇੱਕ `Tool` ਵੀ ਹੈ ਜੋ ਇਸ ਡਿਕਸ਼ਨਰੀ ਨੂੰ ਐਸਾ ਟਾਈਪ ਵਿੱਚ ਬਦਲਦਾ ਹੈ ਜਿਸਨੂੰ MCP ਸਰਵਰ ਹੈਂਡਲਰ ਸੁਵੀਕਾਰ ਕਰ ਸਕਦਾ ਹੈ, ਅਤੇ ਇਹ ਇਸ ਤਰ੍ਹਾਂ ਦਿਖਦਾ ਹੈ:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

ਅਤੇ *schema.ts* ਹੈ ਜਿੱਥੇ ਅਸੀਂ ਹਰ ਟੂਲ ਲਈ ਇਨਪੁਟ ਸਕੀਮਾ ਸਟੋਰ ਕਰਦੇ ਹਾਂ ਜੋ ਹੁਣ ਇੱਕ ਸਕੀਮਾ ਹੈ ਪਰ ਜਿਵੇਂ ਅਸੀਂ ਹੋਰ ਟੂਲਜ਼ ਸ਼ਾਮਲ ਕਰਾਂਗੇ ਵਧਾਕੇ ਜੋੜ ਸਕਦੇ ਹਾਂ:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

ਵਧੀਆ, ਆਓ ਹੁਣ ਟੂਲ ਲਿਸਟਿੰਗ ਨੂੰ ਸੰਭਾਲਣ ਲਈ ਅੱਗੇ ਵਧੀਏ।

### -3- ਟੂਲ ਲਿਸਟਿੰਗ ਸੰਭਾਲਣਾ

ਹੁਣ ਟੂਲ ਲਿਸਟ ਕਰਨ ਲਈ ਇੱਕ ਬੇਨਤੀ ਹੈਂਡਲਰ ਸੈਟ ਕਰਨਾ ਹੈ। ਸਾਨੂੰ ਆਪਣੇ ਸਰਵਰ ਫਾਈਲ ਵਿੱਚ ਇਹ ਸ਼ਾਮਲ ਕਰਨਾ ਪਵੇਗਾ:

**Python**

```python
# ਸੰਕੁਚਿਤਤਾ ਲਈ ਕੋਡ ਛੱਡ ਦਿੱਤਾ ਗਿਆ
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

ਇੱਥੇ ਅਸੀਂ ਡੈਕੋਰੇਟਰ `@server.list_tools` ਜੋੜਦੇ ਹਾਂ ਅਤੇ ਹੈਂਡਲ ਕਰਨ ਵਾਲੀ ਫੰਕਸ਼ਨ `handle_list_tools` ਨੂੰ ਲਾਗੂ ਕਰਦੇ ਹਾਂ। ਇਸ ਫੰਕਸ਼ਨ ਵਿੱਚ, ਸਾਨੂੰ ਟੂਲਜ਼ ਦੀ ਸੂਚੀ ਤਿਆਰ ਕਰਨੀ ਹੈ। ਧਿਆਨ ਦਿਓ ਕਿ ਹਰ ਟੂਲ ਨੂੰ ਨਾਮ, ਵਰਣਨ ਅਤੇ inputSchema ਚਾਹੀਦਾ ਹੈ।

**TypeScript**

ਟੂਲ ਲਿਸਟਿੰਗ ਲਈ ਬੇਨਤੀ ਹੈਂਡਲਰ ਸੈਟ ਕਰਨ ਲਈ ਸਾਨੂੰ ਸਰਵਰ 'ਤੇ `setRequestHandler` ਕਾਲ ਕਰਨਾ ਹੈ ਜਿਸਨੂੰ ਉਹ ਸਕੀਮਾ ਮਿਲੇ ਜੋ ਅਸੀਂ ਕਰਨਾ ਚਾਹੁੰਦੇ ਹਾਂ, ਇਸ ਮਾਮਲੇ ਵਿੱਚ `ListToolsRequestSchema`।

```typescript
// ਇੰਡੈਕਸ.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// ਸਰਵਰ.ts
// ਸਾਰੰਾਸ਼ ਲਈ ਕੋਡ ਹਟਾਇਆ ਗਿਆ
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // ਦਰਜ ਕੀਤੇ ਗਏ ਟੂਲਾਂ ਦੀ ਸੂਚੀ ਵਾਪਸ ਕਰੋ
  return {
    tools: tools
  };
});
```

ਵਧੀਆ, ਹੁਣ ਅਸੀਂ ਟੂਲ ਲਿਸਟਿੰਗ ਹੱਲ ਕਰ ਲਈ ਹੈ, ਆਓ ਵੇਖੀਏ ਕਿ ਅਸੀਂ ਟੂਲਜ਼ ਕਿਵੇਂ ਕਾਲ ਕਰ ਸਕਦੇ ਹਾਂ।

### -4- ਟੂਲ ਕਾਲ ਕਰਨ ਦੀ ਸੰਭਾਲ

ਕਿਸੇ ਟੂਲ ਨੂੰ ਕਾਲ ਕਰਨ ਲਈ ਸਾਨੂੰ ਇੱਕ ਹੋਰ ਬੇਨਤੀ ਹੈਂਡਲਰ ਸੈਟ ਕਰਨ ਦੀ ਲੋੜ ਹੈ, ਜੇਹੜਾ ਕਿ ਇਹ ਹੱਲ ਕਰਦਾ ਹੈ ਕਿ ਕਿਹੜੇ ਫੀਚਰ ਨੂੰ ਕਾਲ ਕਰਨਾ ਹੈ ਅਤੇ ਕਿਹੜੇ ਅਰਗਿਊਮੈਂਟਸ ਦੇ ਨਾਲ।

**Python**

ਆਓ ਡੈਕੋਰੇਟਰ `@server.call_tool` ਵਰਤਦੇ ਹਾਂ ਅਤੇ ਇਸਨੂੰ `handle_call_tool` ਨਾਂ ਦੀ ਫੰਕਸ਼ਨ ਨਾਲ ਲਾਗੂ ਕਰੀਏ। ਇਸ ਫੰਕਸ਼ਨ ਵਿੱਚ ਸਾਨੂੰ ਟੂਲ ਦਾ ਨਾਮ, ਇਸਦਾ ਆਰਗਿਊਮੈਂਟ ਪਾਰਸ ਕਰਨਾ ਅਤੇ ਇਹ ਯਕੀਨੀ ਬਣਾਉਣਾ ਜਰੂਰੀ ਹੈ ਕਿ ਆਰਗਿਊਮੈਂਟਸ ਸਹੀ ਹਨ। ਅਸੀਂ ਇਹ ਪੁਸ਼ਟੀ ਇਸ ਫੰਕਸ਼ਨ ਵਿੱਚ ਵੀ ਕਰ ਸਕਦੇ ਹਨ ਜਾਂ ਅਸਲ ਟੂਲ ਵਿੱਚ ਵੀ ਜਾ ਸਕਦੀ ਹੈ।

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ਇੱਕ ਸ਼ਬਦਕੋਸ਼ ਹੈ ਜਿਸ ਵਿੱਚ ਟੂਲ ਦੇ ਨਾਮ ਕੁੰਜੀਆਂ ਵਜੋਂ ਹਨ
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # ਟੂਲ ਨੂੰ ਕਾਲ ਕਰੋ
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

ਇੱਥੇ ਕੀ ਹੁੰਦਾ ਹੈ:

- ਸਾਡੇ ਕੋਲ ਪਹਿਲਾਂ ਹੀ ਇਨਪੁੱਟ ਪੈਰਾਮੀਟਰ `name` ਹੈ ਜੋ ਟੂਲ ਦੇ ਨਾਮ ਨੂੰ ਦਰਸਾਉਂਦਾ ਹੈ ਅਤੇ `arguments` ਡਿਕਸ਼ਨਰੀ ਦੇ ਰੂਪ ਵਿੱਚ ਸਾਡੇ ਕੋਲ ਆਰਗਿਊਮੈਂਟਸ ਹਨ।
- ਟੂਲ ਨੂੰ ਕਾਲ ਕੀਤਾ ਜਾਂਦਾ ਹੈ `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` ਨਾਲ। ਆਰਗਿਊਮੈਂਟਸ ਦੀ ਪੁਸ਼ਟੀ `handler` ਗੁਣ ਵਿੱਚ ਹੁੰਦੀ ਹੈ ਜੋ ਕਿਸੇ ਫੰਕਸ਼ਨ ਨੂੰ ਸੰਕੇਤ ਕਰਦਾ ਹੈ, ਜੇ ਇਹ ਫੇਲ ਹੋ ਜਾਂਦਾ ਹੈ ਤਾਂ ਇਹ ਇੱਕ ਐਕਸਪਸ਼ਨ ਉਠਾਉਂਦਾ ਹੈ।

ਹੁਣ ਸਾਡੇ ਕੋਲ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਵਰਤ ਕੇ ਟੂਲਜ਼ ਦੀ ਲਿਸਟਿੰਗ ਅਤੇ ਕਾਲ ਦਾ ਪੂਰਾ ਸਮਝ ਹੈ।

ਇੱਥੇ [ਪੂਰਾ ਉਦਾਹਰਣ](./code/README.md) ਵੇਖੋ

## ਕਰਤੱਬ

ਤੁਹਾਨੂੰ ਦਿੱਤਾ ਗਿਆ ਕੋਡ ਵਧਾ ਕੇ ਕਈ ਟੂਲਜ਼, ਸਰੋਤ ਅਤੇ ਪ੍ਰੌਮਪਟ ਸ਼ਾਮਲ ਕਰੋ ਅਤੇ ਦੇਖੋ ਕਿ ਤੁਸੀਂ ਸਿਰਫ਼ tools ਡਾਇਰੈਕਟਰੀ ਵਿੱਚ ਫਾਈਲਾਂ ਜੋੜਨ ਦੀ ਲੋੜ ਮਹਿਸੂਸ ਕਰਦੇ ਹੋ ਨਾ ਕਿ ਕਿਸੇ ਹੋਰ ਥਾਂ।

*ਕੋਈ ਹੱਲ ਨਹੀਂ ਦਿੱਤਾ ਗਿਆ*

## ਸਾਰांश

ਇਸ ਅਧਿਆਇ ਵਿੱਚ, ਅਸੀਂ ਦੇਖਿਆ ਕਿ ਨੀਵਾਂ-ਸਤਰ ਸਰਵਰ ਪਹੁੰਚ ਕਿਵੇਂ ਕੰਮ ਕਰਦੀ ਹੈ ਅਤੇ ਇਹ ਕਿ ਇਹ ਸਾਡੀ ਮਦਦ ਕਰ ਸਕਦੀ ਹੈ ਇੱਕ ਵਧੀਆ ਢਾਂਚਾ ਬਣਾਉਣ ਵਿੱਚ ਜਿਸ ਤੇ ਅਸੀਂ ਅਗਲੇ ਵਿਕਾਸ ਕਰ ਸਕੀਏ। ਅਸੀਂ ਪੁਸ਼ਟੀਕਰਨ ਵੀ ਚਰਚਾ ਕੀਤੀ ਅਤੇ ਤੁਹਾਨੂੰ ਦਿਖਾਇਆ ਕਿ ਕਿਵੇਂ ਪੁਸ਼ਟੀਕਰਨ ਲਾਇਬਰੇਰੀਆਂ ਨਾਲ ਇਨਪੁੱਟ ਦੀ ਪੁਸ਼ਟੀ ਲਈ ਸਕੀਮਾ ਬਣਾਏ ਜਾ ਸਕਦੇ ਹਨ।

## ਅਗਲਾ ਕੀ ਹੈ

- ਅਗਲਾ: [ਸਧਾਰਣ ਪ੍ਰਮਾਣਿਕਤਾ](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਜ਼ਿੰਮੇਵਾਰੀ ਤਿਆਗਣਾ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸ਼ੁੱਧਤਾ ਲਈ ਕੋਸ਼ਿਸ਼ ਕਰਦੇ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸੁਚਾਲਿਤ ਅਨੁਵਾਦ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਣਸੁਚਿਤਤਾਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਿਤ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਇਸ ਅਨੁਵਾਦ ਦੀ ਵਰਤੋਂ ਤੋਂ ਉਤਪੰਨ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਵਖਿਆਵਾਂ ਲਈ ਅਸੀਂ ਜਿੰਮੇਵਾਰ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# ಅತ್ಯಾಧುನಿಕ ಸರ್ವರ್ ಬಳಕೆ

MCP SDK ನಲ್ಲಿ ಎರಡು ವಿಭಿನ್ನ ಪ್ರಕಾರದ ಸರ್ವರ್‌ಗಳು ಬಹಿರಂಗಪಡಿಸಲ್ಪಟ್ಟಿವೆ, ನಿಮ್ಮ ಸಾಮಾನ್ಯ ಸರ್ವರ್ ಮತ್ತು ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್‌. ಸಾಮಾನ್ಯವಾಗಿ, ನೀವು ಅದರ ವೈಶಿಷ್ಟ್ಯಗಳನ್ನು ಸೇರಿಸಲು ಸಾಮಾನ್ಯ ಸರ್ವರ್ ಅನ್ನು ಬಳಕೆ ಮಾಡುತ್ತೀರಿ. ಕೆಲವು ಸಂದರ್ಭಗಳಲ್ಲಿ, ನೀವು ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ಮೇಲೆ ಅವಲಂಬಿತವಾಗಿರಬೇಕಾಗುತ್ತದೆ ಉದಾಹರಣೆಗೆ:

- ಉತ್ತಮ ಸ್ಥಾಪನೆ. ಸಾಮಾನ್ಯ ಸರ್ವರ್ ಮತ್ತು ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ಎರಡೂ ಬಳಸಿಕೊಂಡು ಸ್ವಚ್ಛ ಸ್ಥಾಪನೆಯನ್ನು ರಚಿಸುವುದಕ್ಕೆ ಸಾಧ್ಯವಿದೆ ಆದರೆ ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ಬಳಕೆ ಮಾಡುವುದು ಸ್ವಲ್ಪ ಸುಲಭವೆಂದು ವಾದಿಸಬಹುದು.
- ವೈಶಿಷ್ಟ್ಯ ಲಭ್ಯತೆ. ಕೆಲವು ಅತ್ಯಾಧುನಿಕ ವೈಶಿಷ್ಟ್ಯಗಳನ್ನು ಕೇವಲ ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್‌ನೊಂದಿಗೆ ಮಾತ್ರ ಬಳಸಬಹುದು. ನೀವು ಇದನ್ನು ಮುಂದಿನ ಅಧ್ಯಾಯಗಳಲ್ಲಿ ಸೆಂಪ್ಲಿಂಗ್ ಮತ್ತು ಎಲಿಸಿಟೇಶನ್ ಸೇರಿಸುವಾಗ ನೋಡುತ್ತೀರಿ.

## ಸಾಮಾನ್ಯ ಸರ್ವರ್ ಮತ್ತು ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ನಡುವೆ

ಇಲ್ಲಿ MCP ಸರ್ವರ್ ರಚನೆಯು ಸಾಮಾನ್ಯ ಸರ್ವರ್ ಬಳಕೆ ಮಾಡುತ್ತಿರುವಂತೆ ಕಾಣುತ್ತದೆ

**Python**

```python
mcp = FastMCP("Demo")

# ಒಂದು ಸೇರಿಸುವ ಸಾಧನವನ್ನು ಸೇರಿಸಿ
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

// ಒಂದು ಜೊತೆಗೆ ಸೇರಿಸುವ ಸಾಧನವನ್ನು ಸೇರಿಸಿ
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

ಮುಖ್ಯವಾಗಿ ನೀವು ಸರ್ವರ್‌ಗೆ ಹೊಂದಿಸಲು ಬಯಸುವ ಪ್ರತಿಯೊಂದು ಟೂಲನ್ನು, ಸಂಪನ್ಮೂಲವನ್ನು ಅಥವಾ ಪ್ರಾಂಪ್ಟ್ ಅನ್ನು ಸ್ಪಷ್ಟವಾಗಿ ಸೇರಿಸುತ್ತೀರಿ. ಇದರಲ್ಲಿ ಯಾವುದೇ ತಪ್ಪು ಇಲ್ಲ.

### ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ಬಳಕೆ ವಿಧಾನ

ಆದರೆ, ನೀವು ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ವಿಧಾನವನ್ನು ಬಳಸುವಾಗ, ಪ್ರತಿ ವೈಶಿಷ್ಟ್ಯ ಪ್ರಕಾರಕ್ಕೆ (ಟೂಲ್ಸ್, ಸಂಪನ್ಮೂಲಗಳು ಅಥವಾ ಪ್ರಾಂಪ್ಟ್‌ಗಳು) ಎರಡು ಹ್ಯಾಂಡ್ಲರ್‌ಗಳನ್ನು ರಚಿಸುವುದನ್ನು ಬಗ್ಗೆ ವಿಭಿನ್ನವಾಗಿ ಯೋಚಿಸಬೇಕು. ಉದಾಹರಣೆಗೆ ಟೂಲ್ಸ್ ಬಳಕೆಯಾದರೆ ಕೇವಲ ಎರಡು ಫಂಕ್ಷನ್‌ಗಳು ಇರುತ್ತವೆ:

- ಎಲ್ಲಾ ಟೂಲ್ಸ್ ಅನ್ನು ಪಟ್ಟಿ ಮಾಡುವುದು. ಒಂದು ಫಂಕ್ಷನ್ ಎಲ್ಲಾ ಪ್ರಯತ್ನಗಳನ್ನು ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡಲು ಜವಾಬ್ದಾರಿಯಾಗಿರುತ್ತದೆ.
- ಎಲ್ಲಾ ಟೂಲ್ಸ್ ಕರೆಯುವುದನ್ನು ಸಂಭಾಳಿಸುವುದು. ಇಲ್ಲಿ ಹಾಂಡ್ಲಿಂಗ್ ಮಾಡುವುದು ಒಬ್ಬ ಫಂಕ್ಷನ್ ಮಾತ್ರ.

ಇದು ಕಡಿಮೆ ಕೆಲಸವಲ್ಲವೇ? ಟೂಲ್ ನೋಂದಣಿ ಮಾಡಿಸುವ ಬದಲು, ನಾನು ಕೇವಲ ಪಟ್ಟಿ ಮಾಡಲು ಎಲ್ಲಾ ಟೂಲ್ಸ್ ಅನ್ನು ಪಟ್ಟಿ ಮಾಡಬೇಕು ಮತ್ತು ಟೂಲ್ ಕರೆ ಮಾಡುವ ವಿನಂತಿ ಬಂದಾಗ ಅದನ್ನು ಕರೆಯಬೇಕು ಎಂಬುದನ್ನು ಖಚಿತಪಡಿಸಿಕೊಳ್ಳಬೇಕಾಗುತ್ತದೆ.

ಈಗ ಕოდის ರೂಪ ಹೇಗಿದೆ ನೋಡೋಣ:

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
  // ನೋಂದಾಯಿಸಿದ ಸಾಧನಗಳ ಪಟ್ಟಿಯನ್ನು ಹಿಂತಿರುಗಿಸಿ
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

ಇಲ್ಲಿ ಈಗ ನಮಗೆ ವೈಶಿಷ್ಟ್ಯಗಳ ಪಟ್ಟಿ ನೀಡುವ ಒಂದು ಫಂಕ್ಷನ್ ಇದೆ. ಟೂಲ್ಸ್ ಪಟ್ಟಿಯಲ್ಲಿ ಪ್ರತಿಯೊಂದು ಸ್ಥಳದಲ್ಲಿ `name`, `description` ಮತ್ತು `inputSchema` ಎಂಬ ಕ್ಷೇತ್ರಗಳನ್ನು ಹೊಂದಿರುತ್ತದೆ, ಇದು ರಿಟರ್ನ್ ಪ್ರಕಾರವನ್ನು ಅನುಸರಿಸುತ್ತದೆ. ಇದು ನಮ್ಮ ಟೂಲ್ಸ್ ಮತ್ತು ವೈಶಿಷ್ಟ್ಯ ವ್ಯಾಖ್ಯಾನವನ್ನು ಬೇರೆಡೆ ಇರಿಸಲು ಸಾಧ್ಯವಾಗುತ್ತದೆ. ಈಗ ನಾವು ಎಲ್ಲಾ ಟೂಲ್ಸ್ ಅನ್ನು tools ಫೋಲ್ಡರ್‌ನಲ್ಲಿ ರಚಿಸಬಹುದು ಮತ್ತು ಇದೇ ರೀತಿ ನಿಮ್ಮ ಎಲ್ಲಾ ವೈಶಿಷ್ಟ್ಯಗಳಿಗೂ ಅನ್ವಯಿಸುತ್ತದೆ ಆದ್ದರಿಂದ ನಿಮ್ಮ ಪ್ರಾಜೆಕ್ಟ್ ಈ ರೀತಿಯಾಗಿ ಸುಸಂಸ್ಕೃತವಾಗಬಹುದು:

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

ಅದು ಚೆನ್ನಾಗಿದೆ, ನಮ್ಮ ಸ್ಥಾಪನೆಯನ್ನು ಸ್ವಚ್ಛವಾಗಿ ನೋಡಲು ಮಾಡಬಹುದು.

ಟೂಲ್ಸ್ ಕರೆಯುವುದು ಹೇಗೆ? ಆಲೋಚನೆ ಒಂದೇನಾ, ಯಾವುದು ಟೂಲ್ ಆಗಿದ್ದರೂ ಅದನ್ನು ಕರೆ ಮಾಡುವ ಒಂದೇ ಹ್ಯಾಂಡ್ಲರ್ ಇಲ್ಲವೇ? ಹೌದು, ನಿಖರವಾಗಿ, ಕೆಳಗಿನ ಕೋಡ್ ಅದಕ್ಕೆ:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ಎಂಬುದು ಸಾಧನದ ಹೆಸರುಗಳನ್ನು കീಗಳಾಗಿ ಹೊಂದಿರುವ ಡಿಕ್ಷನರಿ ಆಗಿದೆ
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
    // TODO ಸಾಧನವನ್ನು ಕರೆಸಿರಿ,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

ಮೇಲಿನ ಕೋಡ್‌ನಲ್ಲಿ ನಾವು ಟೂಲ್ ಅನ್ನು ಕರೆಯಲು ಯಾವುದು ಎನ್ನುವುದನ್ನು ಹಾಗೂ ಯಾವ ಪರಿಮಾಣಗಳೊಂದಿಗೆ ಕರೆಯಬೇಕೆಂಬುದನ್ನು ವಿಶ್ಲೇಷಿಸಲು ಬೇಕಾಗುತ್ತದೆ ಮತ್ತು ನಂತರ ಟೂಲ್ ಅನ್ನು ಕರೆದೊಯ್ಯಬೇಕು.

## ದೃಢೀಕರಣದಿಂದ ವಿಧಾನವನ್ನು ಉತ್ತಮಗೊಳಿಸುವುದು

ಈವರೆಗೆ ನೀವು ಟೂಲ್ಸ್, ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಪ್ರಾಂಪ್ಟ್‌ಗಳನ್ನು ಸೇರಿಸಲು ನೀವು ಮಾಡಿದ ಎಲ್ಲಾ ನೋಂದಣಿಗಳನ್ನು ಪ್ರತಿ ವೈಶಿಷ್ಟ್ಯ ಪ್ರಕಾರಕ್ಕೆ ಎರಡು ಹ್ಯಾಂಡ್ಲರ್‌ಗಳಿಂದ ಬದಲಿ ಮಾಡಬಹುದಾಗಿದೆ. ಇನ್ನೇನು ಮಾಡಬೇಕಾಗಿದೆ? ಟೂಲು ಸರಿಯಾದ ಪರಿಮಾಣಗಳೊಂದಿಗೆ ಕರೆದಲ್ಪಡಲಿದೆಯೇ ಎಂದು ಖಚಿತಪಡಿಸಿಕೊಳ್ಳಲು ಕೆಲವು ರೀತಿಯ ದೃಢೀಕರಣವನ್ನು ಸೇರಿಸಬೇಕು. ಪ್ರತಿ ರನ್‌ಟೈಮ್ ಇದಕ್ಕೆ ತನ್ನದೇ ಪರಿಹಾರವಿದೆ, ಉದಾಹರಣೆಗೆ Python Pydantic ಅನ್ನು ಮತ್ತು TypeScript Zod ಅನ್ನು ಬಳಸುತ್ತದೆ. ಯೋಚನೆ ಇಂತಿದೆ:

- ವೈಶಿಷ್ಟ್ಯ (ಟೂಲ್, ಸಂಪನ್ಮೂಲ ಅಥವಾ ಪ್ರಾಂಪ್ಟ್) ರಚಿಸುವ ತರ್ಕವನ್ನು ಅದರ ತನ್ನ ಫೋಲ್ಡರ್‌ಗೆ ಸ್ಥಳಾಂತರಿಸುವುದು.
- ಉದಾಹರಣೆಗೆ ಟೂಲ್ ಕರೆ ಮಾಡಲು ಬಂದಿರುವ ವಿನಂತಿಯನ್ನು ಪರಿಶೀಲಿಸುವುದು.

### ವೈಶಿಷ್ಟ್ಯವನ್ನು ರಚಿಸುವುದು

ವೈಶಿಷ್ಟ್ಯವನ್ನು ರಚಿಸಲು, ನಾವು ಆ ವೈಶಿಷ್ಟ್ಯದ ಫೈಲ್ ಅನ್ನು ರಚಿಸಿ ಮತ್ತು ಆ ವೈಶಿಷ್ಟ್ಯಕ್ಕೆ ಬೇಕಾದ ಕಡ್ಡಾಯ ಕ್ಷೇತ್ರಗಳನ್ನು ಹೊಂದಿಸಬೇಕು. ಟೂಲ್ಸ್, ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಪ್ರಾಂಪ್ಟ್‌ಗಳಲ್ಲಿ ಕೆಲ ಕ್ಷೇತ್ರಗಳಲ್ಲಿ ಭಿನ್ನತೆ ಇದೆ.

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
        # ಪೈಡ್ಯಾಂಟಿಕ್ ಮಾದರಿಯನ್ನು ಬಳಸಿ ಇನ್‌ಪುಟ್ ಮಾನ್ಯಗೊಳಿಸಿ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: ಪೈಡ್ಯಾಂಟಿಕ್ ಸೇರಿಸಿ, այնպես ನಾವು AddInputModel ರಚಿಸಿ, ಮತ್ತು ಆರ್ಗ್ಸ್ ಅನ್ನು ಮಾನ್ಯಗೊಳಿಸಬಹುದು

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ಇಲ್ಲಿ ನೀವು ಈ ಕೆಳಗಿನಂತೆ ಮಾಡುತ್ತೀರಾ:

- Pydantic ಬಳಸಿ `AddInputModel` ಎಂಬ ಸ್ಕೀಮಾವನ್ನು `a` ಮತ್ತು `b` ಎಂಬ ಕ್ಷೇತ್ರಗಳೊಂದಿಗೆ *schema.py* ಫೈಲ್‌ನಲ್ಲಿ ರಚಿಸುವುದು.
- ಬರುತ್ತಿರುವ ವಿನಂತಿಯನ್ನು `AddInputModel` ಪ್ರಕಾರಕ್ಕೆ ವಿಶ್ಲೇಷಿಸಲು ಪ್ರಯತ್ನಿಸುವುದು, ಕ್ಷೇತ್ರಗಳು ತಪ್ಪಿದ್ದು ಇದ್ದರೆ ಇದು ಕ್ರ್ಯಾಷ್ ಆಗುವುದು:

   ```python
   # add.py
    try:
        # ಪೈಡಾಂಟಿಕ್ ಮಾದರಿ ಬಳಸಿ ಇನ್ಪುಟ್ ಮಾನ್ಯಗೊಳಿಸಿ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

ನೀವು ಈ ವಿಶ್ಲೇಷಣಾ ತರ್ಕವನ್ನು ಟೂಲ್ ಕರೆವುದರಲ್ಲಿ ಸೇರಿಸಬಹುದು ಅಥವಾ ಹ್ಯಾಂಡ್ಲರ್ ಫಂಕ್ಷನ್‌ನಲ್ಲಿ ಇರಿಸಬಹುದು.

**TypeScript**

```typescript
// ಸರ್ವರ್.ts
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

// ವೃತ್ತಾಂತ.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// ಸೇರಿಸು.ts
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

- ಎಲ್ಲಾ ಟೂಲ್ಸ್ ಕರೆಗಳನ್ನು ಹ್ಯಾಂಡಲ್ ಮಾಡುವ ಹ್ಯಾಂಡ್ಲರ್‌ನಲ್ಲಿ ನಾವು ಈಗ ಬರುತ್ತಿರುವ ವಿನಂತಿಯನ್ನು ಟೂಲ್‌ನ ವ್ಯಾಖ್ಯಾನಿಸಿದ ಸ್ಕೀಮಾಗೆ ವಿಶ್ಲೇಷಿಸಲು ಪ್ರಯತ್ನಿಸುತ್ತೇವೆ:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ಅದು ಸರಿ ಇದ್ದಾರೆಂದರೆ, ನಾವು ನಿಜವಾದ ಟೂಲ್ ಅನ್ನು ಕರೆ ಮಾಡುತ್ತೇವೆ:

    ```typescript
    const result = await tool.callback(input);
    ```

ಈ ವಿಧಾನದಿಂದ ಒಂದೊಂದು ವಿಷಯಕ್ಕೂ ತನ್ನ ಸಂರಚನೆಯು ಸಿಗುತ್ತದೆ, *server.ts* ಒಂದು ಸಣ್ಣ ಫೈಲ್ ಆಗಿದ್ದು ಕೇವಲ ವಿನಂತಿ ಹ್ಯಾಂಡ್ಲರ್‌ಗಳನ್ನು ಸಂಪರ್ಕಿಸುವುದಕ್ಕಾಗಿ ಮತ್ತು ಪ್ರತಿ ವೈಶಿಷ್ಟ್ಯ ತನ್ನ ಸ್ವಂತ ಫೋಲ್ಡರ್‌ನಲ್ಲಿ ಇರುತ್ತದೆ ಉದಾಹರಣೆಗೆ tools/, resources/ ಅಥವಾ prompts/.

ಒಳ್ಳೆಯದು, ಇದನ್ನು ಮುಂದುವರೆಸೋಣ.

## ಅಭ್ಯಾಸ: ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ರಚನೆ

ಈ ಅಭ್ಯಾಸದಲ್ಲಿ ನಾವು ಇದನ್ನು ಮಾಡುತ್ತೇವೆ:

1. ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡುವುದು ಮತ್ತು ಟೂಲ್ಸ್ ಕರೆ ಮಾಡುವುದನ್ನು ಹ್ಯಾಂಡಲ್ ಮಾಡುವ ನಿಷ್ಪ್ರಭಾತ ಸರ್ವರ್ ರಚನೆ.
2. ನೀವು ನಿರ್ಮಿಸಬಹುದಾದ ಸ್ಥಾಪನೆಯನ್ನು ಅನುಷ್ಠಾನಗೊಳಿಸುವುದು.
3. ನಿಮ್ಮ ಟೂಲ್ ಕರೆಗಳನ್ನು ಸರಿಯಾಗಿ ದೃಢೀಕರಿಸಲು ದೃಢೀಕರಣ ಸೇರಿಸುವುದು.

### -1- ಸ್ಥಾಪನೆ ರಚನೆ

ನಾವು ಮೊದಲು ಸೌಲಭ್ಯ ಮತ್ತು ಹೆಚ್ಚು ವೈಶಿಷ್ಟ್ಯಗಳನ್ನು ಸೇರಿಸುತ್ತಾ ಸ್ಕೇಲ್ ಆಗುವ ಸ್ಥಾಪನೆಯನ್ನು ನಿರ್ವಹಿಸುವುದು ಮುಖ್ಯ, ಇದು ಹೀಗಿದೆ:

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

ಈಗ ನಾವು ಸ್ಥಾಪನೆಯನ್ನು ರಚಿಸಿದ್ದೇವೆ ಇದು tools ಫೋಲ್ಡರ್‌ನಲ್ಲಿ ಹೊಸ ಟೂಲ್ಸ್‌ಗಳನ್ನು ಸುಲಭವಾಗಿ ಸೇರಿಸಲು ಸಾಧ್ಯವಿದೆ. ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಪ್ರಾಂಪ್ಟ್‌ಗಳಿಗೆ ಉಪಡೈರೆಕ್ಟರಿಗಳನ್ನು ಸೇರಿಸಲು ಅದನ್ನು ಅನುಸರಿಸಿ.

### -2- ಟೂಲ್ ರಚನೆ

ಟೂಲ್ ರಚಿಸುವುದು ಹೇಗೆ ಎಂಬುದನ್ನು ನೋಡೋಣ. ಮೊದಲು, ಅದನ್ನು ಅದರ *tool* ಉಪಡೈರೆಕ್ಟರಿಯಲ್ಲಿ ರಚಿಸಬೇಕು:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic ಮಾದರಿಯನ್ನು ಬಳಸಿ ಇನ್ಪುಟ್ ಮಾನ್ಯಗೊಳಿಸಿ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ಅನ್ನು ಸೇರಿಸಿ, ಹೀಗಾಗಿ ನಾವು AddInputModel ಅನ್ನು ರಚಿಸಿ ಆರ್ಗ್ಸ್ ಅನ್ನು ಮಾನ್ಯಗೊಳಿಸಬಹುದು

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ಇಲ್ಲಿ ನೀವು ನೋಡಬಹುದು ನಾವು ಹೆಸರು, ವಿವರಣೆ, Pydantic ಬಳಸಿ ಇನ್‌ಪುಟ್ ಸ್ಕೀಮಾ ಮತ್ತು ಕರೆ ಮಾಡಿದಾಗ invoke ಆಗುವ ಹ್ಯಾಂಡ್ಲರ್ ಅನ್ನು ವ್ಯಾಖ್ಯಾನಿಸುತ್ತೇವೆ. ಕೊನೆಯಲ್ಲಿ, ಈ ಎಲ್ಲಾ ಗುಣಲಕ್ಷಣಗಳನ್ನೊಳಗೊಂಡ ಡಿಕ್ಷನರಿ `tool_add` ಅನ್ನು ಬಹಿರಂಗಪಡಿಸುತ್ತೇವೆ.

ಹೆಚ್ಚಾಗಿ *schema.py* ಇದೆ, ಇದು ನಮ್ಮ ಟೂಲ್ ಬಳಸುವ ಇನ್‌ಪುಟ್ ಸ್ಕೀಮಾ ನಿರ್ಧರಿಸಲು ಉಪಯೋಗಿಸುವುದು:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

ಮತ್ತು ನಾವು *__init__.py* ನಲ್ಲೂ ಟೂಲ್ಸ್ ಡೈಮುಖಿಯನ್ನು ಒಂದು ಮೋಡ್ಯೂಲ್ ಎಂದು ಪರಿಗಣಿಸಲು ಭರ್ತಿ ಮಾಡಬೇಕು. ಜೊತೆಗೆ ಅವುಗಳನ್ನು ಈ ರೀತಿಯಲ್ಲಿ ಬಹಿರಂಗ ಮಾಡಬೇಕು:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

ಹೆಚ್ಚಿನ ಟೂಲ್ಸ್ ಸೇರಿಸುವಾಗ ನಾವು ಈ ಫೈಲ್ನಲ್ಲಿ ಸೇರಿಸಬಹುದು.

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

ಇಲ್ಲಿ ನಾವು ಗುಣಲಕ್ಷಣಗಳಿಂದ ಡಿಕ್ಷನರಿ ರಚಿಸುತ್ತೇವೆ:

- name, ಟೂಲಿನ ಹೆಸರು.
- rawSchema, ಇದು Zod ಸ್ಕೀಮಾ, ಟೂಲ್ ಕರೆಯಲು ಬರುವ ವಿನಂತಿಗಳನ್ನು ಪರಿಶೀಲಿಸಲು ಉಪಯೋಗಿಸಲಾಗುತ್ತದೆ.
- inputSchema, ಹ್ಯಾಂಡ್ಲರ್ ಬಳಸುವ ಸ್ಕೀಮಾ.
- callback, ಟೂಲನ್ನು invoke ಮಾಡಲು ಉಪಯೋಗಿಸುವದು.

ಮತ್ತು `Tool` ಎಂಬುದು ಇದೆ ಅದು ಈ ಡಿಕ್ಷನರಿ ಅನ್ನು MCP ಸರ್ವರ್ ಹ್ಯಾಂಡ್ಲರ್ ಸ್ವೀಕರಿಸುವ ಪ್ರಕಾರಕ್ಕೆ ಪರಿವರ್ತಿಸುತ್ತದೆ, ಇದು ಹೀಗಿದೆ:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

ಆಮೇಲೆ *schema.ts* ಇದೆ ಅಲ್ಲಿ ಪ್ರತಿ ಟೂಲಿನ ಇನ್‌ಪುಟ್ ಸ್ಕೀಮಾಗಳನ್ನು ಸಂಗ್ರಹಿಸಲಾಗುತ್ತದೆ, ಪ್ರಸ್ತುತ ಒಂದೇ ಸ್ಕೀಮಾ ಇದೆ ಆದರೆ ಹೆಚ್ಚು ಟೂಲ್ಸ್ ಸೇರಿಸುವಂತೆ entries ಹೆಚ್ಚಿಸಬಹುದು:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

ಚೆನ್ನಾಗಿದೆ, ಈಗ ನಾವು ನಮ್ಮ ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡೋದರ ಹ್ಯಾಂಡ್ಲರ್ ಅನ್ನು ತಯಾರಿಸೋಣ.

### -3- ಟೂಲ್ ಪಟ್ಟಿ ಮಾಡುವ ಹ್ಯಾಂಡ್ಲರ್

ಪ್ರತಿ ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡುವ ವಿನಂತಿಯನ್ನು ಹ್ಯಾಂಡಲ್ ಮಾಡಲು ನಾವು ಸರ್ವರ್ ಫೈಲ್‌ನಲ್ಲಿ ಈ ಕೆಳಗಿನ ವೈಶಿಷ್ಟ್ಯಗಳನ್ನು ಸೇರಿಸಬೇಕು:

**Python**

```python
# ಸರಳತೆಗಾಗಿ ಕೋಡ್ ಅನ್ನು ಕಳೆಯಲಾಗಿದೆ
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

ಇಲ್ಲಿ ನಾವು `@server.list_tools` ಡೆಕೋರೇಟರ್ ಹಾಗೂ ಜಾರಿಗೊಳಿಸುವ  `handle_list_tools` ಫಂಕ್ಷನ್ ಅನ್ನು ಸೇರಿಸಲಿದ್ದೇವೆ. ಇದರಲ್ಲಿ, ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಉತ್ಪಾದಿಸಲಾಗುತ್ತದೆ. ಪ್ರತಿ ಟೂಲಿನ ಹೆಸರು, ವಿವರಣೆ ಮತ್ತು inputSchema ಇರಬೇಕಿದೆ.

**TypeScript**

ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡಲು ವಿನಂತಿ ಹ್ಯಾಂಡ್ಲರ್ ಸರಿಯಾದ Schema ಹೊಂದಿರಬೇಕಾದ್ದರಿಂದ ನಾವು ಸರ್ವರ್ ನಲ್ಲಿ `setRequestHandler` ಅನ್ನು ಉಪಯೋಗಿಸುತ್ತೇವೆ, ಈ ಸಂದರ್ಭದಲ್ಲಿ `ListToolsRequestSchema`.

```typescript
// ಇಂಡೆಕ್ಸ್.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// ಸರ್ವರ್.ts
// ಸಂಕ್ಷಿಪ್ತತೆಗೆ ಕೋಡ್ ತೆಗೆಯಲಾಗಿದೆ
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // ನೋಂದಾಯಿಸಲಾದ ಸಾಧನಗಳ ಪಟ್ಟಿ ಮರುಕಳಿಸಿ
  return {
    tools: tools
  };
});
```

ಚೆನ್ನಾಗಿದೆ, ಈಗ ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡೋದು ಸಾಧ್ಯವಾಯಿತು, ಮುಂದಿನ ಹಂತದಲ್ಲಿ ಟೂಲ್ಸ್ ಕರೆ ಮಾಡುವುದನ್ನು ನೋಡೋಣ.

### -4- ಟೂಲ್ ಕರೆ ಮಾಡುವ ಹ್ಯಾಂಡ್ಲರ್

ಟೂಲನ್ನು ಕರೆ ಮಾಡಲು, ನಿರ್ದಿಷ್ಟ ವೈಶಿಷ್ಟ್ಯ ಏನು ಮತ್ತು ಯಾವ ಪರಿಮಾಣಗಳಿಗೆ ಕರೆ ಮಾಡಬೇಕು ಎನ್ನುವ ವಿನಂತಿಯನ್ನು ಹ್ಯಾಂಡಲ್ ಮಾಡುವ ಇನ್ನೊಂದು ವಿನಂತಿ ಹ್ಯಾಂಡ್ಲರ್ ಬೇಕು.

**Python**

`@server.call_tool` ಡೆಕೋರೇಟರ್ ಬಳಸಿ `handle_call_tool` ಫಂಕ್ಷನ್ ರಚಿಸೋಣ. ಈ ಫಂಕ್ಷನಿನೊಳಗೆ, ಟುಲ್ ಹೆಸರು, ಅದರ ಅರ್ಗುಮೆಂಟ್ ಮತ್ತು ಆರ್ಗ್ಯುಮೆಂಟ್ ಸರಿಯಾದ್ದೇ ಎಂದು ಪರಿಶೀಲಿಸುವುದು ಮಾಡಬೇಕು. ನೀವು ಈ ಪರಿಶೀಲನೆಯನ್ನು ಈ ಫಂಕ್ಷನಿನಲ್ಲೇ ಅಥವಾ ಪರವಾನಗಿ ನೀಡಿದ ಟೂಲ್‌ನಲ್ಲಿ ಮಾಡಬಹುದು.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ಎಂಬುದು ಸಾಧನ ಹೆಸರುಗಳನ್ನು ಕೀಗಳಾಗಿ ಹೊಂದಿರುವ ನಿಘಂಟು
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # ಸಾಧನವನ್ನು ಕರೆಮಾಡಿ
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

ಇಲ್ಲಿ ನಡೆಯುವುದು:

- ಟೂಲಿನ ಹೆಸರು ಈಗಲೇ ಇನ್‌ಪುಟ್ ಪ್ಯಾರಾಮೀಟರ್ `name` ಆಗಿದೆ ಮತ್ತು `arguments` ಡಿಕ್ಷನರಿಯ ರೂಪದಲ್ಲಿ ಅರ್ಗ್ಯುಮೆಂಟ್ಸ್ ಆಗಿವೆ.

- ಟೂಲ್ ಕರೆಯುವಾಗ ನಾವು `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` ಎಂದು ಕರೆಯುವೆವು. ಆರ್ಗ್ಯುಮೆಂಟ್ ಪರಿಶೀಲನೆ `handler` ಗುಣಲಕ್ಷಣದಲ್ಲಿ ಫಂಕ್ಷನಿಗೆ ಸೂಚಿಸುತ್ತದೆ, ವಿಫಲವಾದರೆ_exception_ ಉಂಟಾಗುತ್ತದೆ.

ಈಗ ನಾವು ಸಂಪೂರ್ಣವಾಗಿ ಲೋ ಲೆವೆಲ್ ಸರ್ವರ್ ಬಳಸಿ ಟೂಲ್ಸ್ ಪಟ್ಟಿ ಮಾಡುವುದು ಮತ್ತು ಕರೆಯುವುದು ಹೇಗೆ ಎನ್ನುವದನ್ನು ಅರಿತಿದ್ದೇವೆ.

[ನೀವು ಸಂಪೂರ್ಣ ಉದಾಹರಣೆಯನ್ನು ಇಲ್ಲಿ ನೋಡಬಹುದು](./code/README.md)

## ನಿಯೋಜನೆ

ನೀIALOGವಿದ ಟೂಲ್ಸ್, ಸಂಪನ್ಮೂಲಗಳು ಮತ್ತು ಪ್ರಾಂಪ್ಟ್‌ಗಳನ್ನು ಬಳಸಿ ಕೊಡ್ ವಿಸ್ತರಿಸಿ ಮತ್ತು ನೀವು ಗಮನಿಸುವುದೇನಂದರೆ ನೀವು ಕೇವಲ tools ಡೈರೆಕ್ಟರಿಯಲ್ಲಿ ಫೈಲ್‌ಗಳನ್ನು ಸೇರಿಸುವುದು ಮಾತ್ರ ಬೇಕಾಗುತ್ತದೆ, ಬೇರೆ ಎಲ್ಲಿಗೆ ಬೇಕಾಗುವುದಿಲ್ಲ ಎನ್ನುವ ವಿಚಾರದಲ್ಲಿ ಟೀಟಲು ಮಾಡಿಸಿ.

*ಯಾವುದೇ ಪರಿಹಾರ ನೀಡಿಲ್ಲ*

## ಸಾರಾಂಶ

ಈ ಅಧ್ಯಾಯದಲ್ಲಿ, ನಾವು ಹೇಗೆ ಲೋ-ಲೆವೆಲ್ ಸರ್ವರ್ ವಿಧಾನ ಕಾರ್ಯನಿರ್ವಹಿಸಬಹುದೆನ್ನುವುದನ್ನು ಮತ್ತು ಇದರಿಂದ ಸುಂದರ ಸ್ಥಾಪನೆಯನ್ನು ಸೃಷ್ಟಿಸುವುದನ್ನು ನೋಡಿದೆವು. ನಾವು ದೃಢೀಕರಣವನ್ನು ಚರ್ಚಿಸಿ ಮತ್ತು ಇನ್‌ಪುಟ್ ದೃಢೀಕರಣಕ್ಕಾಗಿ ಸ್ಕೀಮಾಗಳನ್ನು ರಚಿಸಲು ಪರಿಶೀಲನಾ ಲೈಬ್ರರಿಗಳನ್ನು ಬಳಸುವ ವಿಧಾನದ ಬಗ್ಗೆ ತಿಳಿದಿದೆವು.

## ಮುಂದಿನ ಹಂತ

- ಮುಂದಿನದು: [ಸರಳ ಪ್ರಮಾಣೀಕರಣ](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಮನವೊಮ್ಮೆ ಸೂಚನೆ**:  
ಈ ದಾಖಲೆವನ್ನು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ಯಾಕಾಗಲೂ ಶುದ್ಧತೆಗೆ ಪ್ರಯತ್ನಿಸುವುದಾದರೂ, ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ತಪ್ಪುಗಳು ಅಥವಾ ಅಸಮರ್ಪಕತೆಗಳಿರಬಹುದು ಎಂದು ದಯವಿಟ್ಟು ಗಮನಿಸಿ. ಮೂಲ ಭಾಷೆಯಲ್ಲಿನ ಮೂಲ ದಾಖಲೆ ಪ್ರಾಮಾಣಿಕ ಮೂಲ ಎಂದು ಪರಿಗಣಿಸಬೇಕು. ಮಹತ್ವದ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡು ತದೆ. ಈ ಅನುವಾದ ಬಳಕೆಯಿಂದ ಉಂಟಾಗುವ ಯಾವುದೇ ಅರ್ಥಮರ್ಚನೆ ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನದ ಹೊಣೆಗಾರಿಕೆ ನಮ್ಮದಾಗಿರದು.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
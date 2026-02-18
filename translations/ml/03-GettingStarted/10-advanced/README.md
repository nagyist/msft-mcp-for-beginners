# അഡ്വാൻസ്ഡ് സർവർ ഉപയോഗം

MCP SDK-യിൽ രണ്ട് വ്യത്യസ്ത തരത്തിലുള്ള സർവർസുകൾ ഉണ്ട്: നിങ്ങളുടെ സാധാരണ സർവർയും ലോ-ലെവൽ സർവർയും. സാധാരണയായി, നിങ്ങൾക്ക് സാധാരണ സർവർ ഉപയോഗിച്ച് അതിൽ ഫീച്ചറുകൾ ചേർക്കും. ചില കേസുകളിൽ നിങ്ങളാകുന്നത് ലോ-ലെവൽ സർവറിനെ ആശ്രയിക്കണം, ഉദാഹരണത്തിന്:

- മെച്ചപ്പെട്ട ആർക്കിടെക്ചർ. സാധാരണ സർവർയും ലോ-ലെവൽ സർവറും ഉപയോഗിച്ച് ക്ലീൻ ആർക്കിടെക്ചർ സൃഷ്‌ടിക്കുന്നതിനു സാധിക്കുന്നത് എന്നാൽ ലോ-ലെവൽ സർവറുമായി അത് കുറച്ച് എളുപ്പത്തിൽ സാധ്യമാകുന്നതായി പറയാം.
- ഫീച്ചർ ലഭ്യത. ചില അഡ്വാൻസ്ഡ് ഫീച്ചറുകൾക്കു മാത്രമേ ലോ-ലെവൽ സർവർ ഉപയോഗിക്കാവു. വേഴ്സംല ൽ ഞങ്ങൾ سیم്പ്‌ളിങ്‌, elicitation എന്നിവ ചേർക്കുമ്പോളാണ് ഇത് കാണാൻ സാധിക്കുന്നത്.

## സാധാരണ സർവർ vs ലോ-ലെവൽ സർവർ

സാധാരണ സർവറോടെ MCP സർവർ സൃഷ്‌ടിക്കുന്നത് ഇങ്ങനെയാണ്

**Python**

```python
mcp = FastMCP("Demo")

# ഒരു കൂട്ടിച്ചേർക്കൽ ഉപകരണം ചേർക്കുക
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

// ഒരു കൂട്ടിച്ചേർക്കൽ ഉപകരണം ചേർക്കുക
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

പോയിന്റ് എന്തെന്നാൽ, നിങ്ങൾ സർവറിന് വേണ്ട എല്ലാ ടൂൾസും, റിസോഴ്‌സുകളും അല്ലെങ്കിൽ പ്രോംപ്റ്റുകളും വ്യക്തമായി ചേർക്കുകയാണ് ചെയ്യുന്നത്. ഇതിൽ യാതൊരു തെറ്റുമില്ല.  

### ലോ-ലെവൽ സർവർ സമീപനം

എങ്കിലും, ലോ-ലെവൽ സർവർ സമീപനം ഉപയോഗിക്കുമ്പോൾ നിങ്ങൾക്ക് വേറിട്ട രീതിയിൽ ചിന്തിക്കേണ്ടതാണ്, അഥവാ ഓരോ ടൂൾ രജിസ്റ്റർ ചെയ്യുന്ന പകരം ഓരോ ഫീച്ചർ ടൈപ്പിനും (ടൂളുകൾ, റിസോഴ്‌സുകൾ അല്ലെങ്കിൽ പ്രോംപ്റ്റുകൾ) രണ്ട് ഹാൻഡ്ലർമാരെ സൃഷ്‌ടിക്കണം. ഉദാഹരണത്തിന് ടൂളുകൾക്കായി രണ്ട് ഫംഗ്ഷനുകൾ മാത്രമേ ഉണ്ടാകൂ:

- എല്ലാ ടൂളുകളും ലിസ്റ്റ് ചെയ്യുക. ടൂളുകൾ ലിസ്റ്റ് ചെയ്യാനുള്ള എല്ലാ ശ്രമങ്ങൾക്കും ഒരു ഫംഗ്ഷൻ ഉത്തരവാദിത്വം വഹിക്കും.
- എല്ലാ ടൂളുകളെയും വിളിക്കുന്നത് കൈകാര്യം ചെയ്യുക. ഇവിടെ കൂടി ടൂൾ വിളിക്കാൻ ഒരു ഫംഗ്ഷൻ പക്ഷെ മാത്രം കൈകാര്യം ചെയ്യും.

കുറച്ചേ കുറവ് ജോലി ചെയ്യേണ്ടതായിരിക്കാം എന്നു തോന്നുന്നുണ്ടല്ലോ? അതായത്, ഒരു ടൂൾ രജിസ്റ്റർ ചെയ്യുന്നതിന് പകരം, ഞാൻ ടൂളുകൾ ലിസ്റ്റ് ചെയ്യുമ്പോൾ ടൂളുകൾ ലിസ്റ്റിൽ ഉൾപ്പെടുത്തും, ടൂൾ വിളിക്കാനുള്ള ആവശ്യകത വന്നാൽ അത് വിളിക്കുന്നുണ്ടെന്ന് ഉറപ്പുവരുത്തുക മാത്രമെ വേണം. 

ഇപ്പോൾയുടെ കോഡ് ഇങ്ങനെയാണ്:

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
  // രജിസ്റ്റർ ചെയ്ത ടൂളുകളുടെ പട്ടിക തിരികെ നൽക്കുക
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

ഇവിടെ ഫീച്ചറുകളുടെ പട്ടിക തിരിച്ചും നൽകുന്ന ഫംഗ്ഷൻ ഉണ്ട്. ഇതിലെ ടൂളുകളുടെ പട്ടികയിൽ ഓരോ എൻട്രിക്കും `name`, `description`, `inputSchema` പോലുള്ള ഫീൽഡുകൾ ഉണ്ട് റിട്ടേൺ ടൈപ്പിൽ ഒപ്പമുണ്ടാകാൻ. ഇതിലൂടെ ഞങ്ങൾക്ക് ടൂളുകളും ഫീച്ചർ ഡിഫിനിഷനും മറ്റിടത്ത് ഒപ്പം സൂക്ഷിക്കാം. നമ്മൾ എല്ലാ ടൂളുകളും tools ഫോൾഡറിൽ സൃഷ്‌ടിക്കാം, അതുപോലെ റിസോഴ്‌സുകളും പ്രോംപ്റ്റുകളും വേണ്ട സ്ഥലങ്ങളിൽ സൂക്ഷിക്കാം, അതുവഴി പ്രോജക്ട് അങ്ങനെ സംഘടിപ്പിക്കാം:

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

ശരിയും, നമ്മുടെ ആർക്കിടെക്ചർ സുതാര്യമായി ഒരുക്കാവുന്നതാണ്.

ടൂളുകൾ വിളിക്കുന്നത് എങ്ങനെയാകും? തലമൂനേടേ കഴിഞ്‌ന്യൂ ഒരാളെ ഓരെരെ മൊഴി വേണ്ടി വാലി ഒരു ടൂൾ വിളിക്കുക എന്നതാണോ? അതാണ്, ഇതിനുള്ള കോഡ് താഴെ:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools എന്നത് ടൂളുകളുടെ പേരുകൾ കീസായി ഉള്ള ഒരു ഡിക്ഷണറിയാണ്
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
    // TODO ടൂൾ کال്പ് ചെയ്യുക,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

അപ്പൊൾ ഇവിടെയുള്ള കോഡിൽ നിങ്ങൾക്കു കാണാം ടൂൾ ആരെന്ന്, എങ്ങനെ(arguments) എന്നത് പാര്‍സുചെയ്ത് ടൂൾ വിളിക്കണം.

## പരിശോധനയോടെയുള്ള മുന്നേറ്റം

ഇതുവരെ നിങ്ങൾ കണ്ടത്, ടൂൾസ്, റിസോഴ്‌സുകൾ, പ്രോംപ്റ്റുകൾ എന്നിവ ചേർക്കുന്നതിന്റെ പകരമായി ഓരോ ഫീച്ചർ ടൈപ്പിനും ഇവയായി രണ്ട് ഹാൻഡ്ലർമാരെ ഉപയോഗിക്കുന്നത് എങ്ങനെയെന്ന്. ഇനി എന്തു വേണം? ടൂൾ ശരിയായ argument-കളോടാണ് വിളിക്കപ്പെടുന്നതെന്ന് ഉറപ്പുവരുത്താൻ ചില form validation ചേർക്കണം. ഓരോ റൺടൈംക്കും വ്യത്യസ്തമായത് ഉണ്ട്, ഉദാഹരണത്തിന് Python-ൽ Pydantic ഉപയോഗിക്കുന്നു, TypeScript-ൽ Zod. ഇതിന്റെ ആശയം:

- ഫീച്ചർ സൃഷ്‌ട്ടിക്കാനുള്ള ലജിക് ആ ഫീച്ചറിനും സമർപ്പിച്ച ഫോൾഡറിലേക്കു മാറ്റുക.
- ഒരു റിക്വസ്റ്റ് വന്നപ്പോൾ ഉദാഹരണത്തിന് ടൂൾ വിളിക്കുമ്പോൾ അതില്‍ аргументുകള്‍ ശരിയാണ് എന്നു പരിശോധിക്കാൻ സംവിധാനം ചേർക്കുക.

### ഫീച്ചർ സൃഷ്‌ടിക്കൽ

ഒരു ഫീച്ചർ സൃഷ്ടിക്കാൻ, ആ ഫീച്ചറിന് വേണ്ട ഫയൽ സൃഷ്ടിച്ച് അതിൽ അവശ്യ ഫീൽഡുകൾ വേണം. ടൂളുകൾ, റിസോഴ്‌സുകൾ, പ്രോംപ്റ്റുകൾക്ക് ആവശ്യമായ ഫീൽഡുകൾ ഒന്ന് തവണ വ്യത്യസ്ഥമാണു.

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
        # Pydantic മോഡൽ ഉപയോഗിച്ച് ഇൻപുട്ട് പരിശോധന നടത്തുക
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ചേർക്കുക, അതിലൂടെ നാം AddInputModel സൃഷ്ടിച്ച് എർഗ്യുമെന്റുകൾ പരിശോധിക്കാമാകും

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ഇവിടെ ഇങ്ങനെ ചെയ്യുന്നു:

- Pydantic ഉപയോഗിച്ച് `AddInputModel` എന്ന സ്കീമ സൃഷ്‌ടിക്കുന്നു *schema.py* ൽ, ഫീൽഡുകൾ `a` , `b`.
- വ്യത്യസ്ത പാരാമീറ്ററുകൾ ഉണ്ടെങ്കിൽ ഇതു പരാജയപ്പെടും, അതിന് parsing ശ്രമിക്കുന്നു:

   ```python
   # add.py
    try:
        # Pydantic മോഡൽ ഉപയോഗിച്ച് ഇൻപുട്ട് പരിശോധന നടത്തുക
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

ഈ parsing ലജിക് ടൂൾ കോളിലോ ഹാൻഡ്ലർ ഫംഗ്ഷനിലോ വേണമെങ്കിൽ നിർത്താവുന്നതാണ്.

**TypeScript**

```typescript
// സെർവർ.ts
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

       // @ts-നിര്‍വ്വചനം
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

// സ്കീമ.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// ചേർക്കുക.ts
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

- എല്ലാ ടൂൾ കോളുകളെയും കൈകാര്യം ചെയ്യുന്ന ഹാൻഡ്ലറിൽ, ടൂൾ നിർവചിച്ച സ്കീമയിലേക്ക് റിക്വസ്റ്റ് പാർസ് ചെയ്യും:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

- അതുവഴി സാധുവാണെങ്കിൽ ടൂൾ കോളിംഗ് നടക്കും:

    ```typescript
    const result = await tool.callback(input);
    ```

ഈ സമീപനം നല്ല ആർക്കിടെക്ചർ സൃഷ്‌ടിക്കുന്നു, എല്ലാമൂലവും സ്വന്തം തിടുക്കിയിടത്താണ്. *server.ts* വളരെ ചെറിയ ഫയൽ മാത്രമാണ്, അത് മാത്രം റിക്വസ്റ്റ് ഹാൻഡ്ലർമാരെ ഇവിടെ കാത്തിരിക്കുന്നു, ഓരോ ഫീച്ചറും സ്വന്തം ഫോൾഡറിലുണ്ട് (tools/, resources/, prompts/).

അതെ, ഇനി ഇത് നിർമ്മിക്കാൻ ശ്രദ്ധിക്കാം. 

## അഭ്യാസം: ലോ-ലെവൽ സർവർ സൃഷ്‌ടിക്കൽ

ഈ അഭ്യാസത്തിൽ, താഴെ പറയുന്നതുപോലെ ചെയ്യും:

1. ടൂളുകൾ ലിസ്റ്റ് ചെയ്യുന്നതിനും ടൂൾസ് വിളിക്കുന്നതിനും ലോ-ലെവൽ സർവർ ഹാൻഡ്ലർ സൃഷ്‌ടിക്കുക.
1. മെച്ചപ്പെട്ട ആർക്കിടെക്ചർ നടപ്പിലാക്കുക.
1. നിങ്ങളുടെ ടൂൾ കോളുകൾ ശരിയായി പരിശോധനയിലൂടെ കടക്കാൻ വാലിഡേഷൻ ചേർക്കുക.

### -1- ആർക്കിടെക്ചർ സൃഷ്‌ടിക്കൽ

ആദ്യം നമ്മൾ ശ്രദ്ധിക്കേണ്ടത് ഒരു ആർക്കിടെക്ചർ ആണ്, കൂടുതൽ ഫീച്ചറുകൾ ചേർക്കുമ്പോൾ സ്കേൽ ചെയ്യാൻ സഹായിക്കുന്നതാണ്, ഇങ്ങനെ ആകും:

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

ഇപ്പോൾ ഒരു ആർക്കിടെക്ചർ സജ്ജീകരിച്ചിട്ടുണ്ട് tools ഫോൾഡറിൽ ടൂളുകൾ എളുപ്പത്തിൽ ചേർക്കാനാകും. റിസോഴ്‌സുകൾക്കും പ്രോംപ്റ്റുകൾക്കും സബ്ഡ ഇരക്ടറികൾ ചേർക്കുന്നത് സ്വീകാര്യമാണ്.

### -2- ടൂൾ സൃഷ്‌ടിക്കൽ

ടൂൾ സൃഷ്‌ടിക്കുന്നത് ഇതിനുപിന്നീട് നോക്കാം. ആദ്യം ടൂൾ താഴെ കാണുന്ന പോലെ അതിന്റെ tools സബ്ഡിരക്ടറിയിൽ സൃഷ്‌ടിക്കണം:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic മോഡൽ ഉപയോഗിച്ച് ഇൻപുട്ട് സ്ഥിരീകരിക്കുക
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ചേർക്കുക, ώστε AddInputModel രൂപപ്പെടുത്തുകയും ആർഗ്യുമെന്റുകൾ സ്ഥിരീകരിക്കാനും കഴിയും

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ഇവിടെ നാം ടൂളിന്റെ പേര്, വിവരണം, Pydantic ഉപയോഗിച്ച് ഇൻപുട്ട് സ്കീമയും തുടര്‍ന്ന് ടൂൾ വിളിക്കുമ്പോൾ ബന്ധിപ്പിക്കുന്ന ഹാൻഡ്ലർ ഉൾപ്പെടുത്തുന്നു. ഒടുക്കം, `tool_add` എന്ന ഡിക്ഷനറി എല്ലാം പിൻവലിക്കുന്നു.

*schema.py* എന്ന ഫയൽ ടൂളിന് ആവശ്യമായ ഇൻപുട്ട് സ്കീമ നിർവചിക്കാൻ ഉപയോഗിക്കുന്നു:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

നാം *__init__.py* ഫയൽ റിക്വയർ ചെയ്യണം tools ഡയറക്ടറിയെ മോഡ്യൂളായി പരിഗണിക്കാൻ. കൂടാതെ അതിലെ മോഡ്യൂളുകളും പുറത്തു തുറക്കണം:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

ഇവിടെ കൂടുതൽ ടൂളുകൾ ചേർക്കുമ്പോൾ ഈ ഫയൽ കൂടി അപ്ഡേറ്റ് ചെയ്യാം.

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

ഇവിടെ നാം ഈ ഡിക്ഷനറിയിൽ ഉൾപ്പെടുത്തി:

- name: ടൂളിന്റെ പേര്.
- rawSchema: Zod സ്കീമ, റിക്വസ്റ്റ് സാധുത പരിശോദിക്കാൻ.
- inputSchema: ഹാൻഡ്ലർ ഉപയോഗിക്കുന്നത്.
- callback: ടൂൾ ഇൻവോക്കേഷൻ.

`Tool` ടൈപ്പ് ഈ ഡിക്ഷനറി MCP സർവർ ഹാൻഡ്ലർക്ക് സ്വീകരിക്കാവുന്ന രീതിയിലേക്ക് മാറ്റുന്നു:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

ഇവിടെ *schema.ts* ടൂളുകൾക്കുള്ള ഇൻപുട്ട് സ്കീമ സോഴ്‌സ് ആണ്. ഇപ്പോൾ ഒറ്റ സ്കീമ മാത്രമാണ്, ടൂളുകൾ കൂട്ടുമ്പോൾ അധികം ഉണ്ടാകും:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

ശരിയാണ്, ഇപ്പോൾ ടൂളുകൾ ലിസ്റ്റ് ചെയ്യുന്നത് കൈകാര്യം ചെയ്യാം.

### -3- ടൂൾ ലിസ്റ്റിങ് കൈകാര്യം ചെയ്യൽ

ടൂളുകൾ ലിസ്റ്റ് ചെയ്യാൻ, അതിന് വേണ്ട റിക്വസ്റ്റ് ഹാൻഡ്ലർ സജ്ജമാക്കണം. ഇതാണ് സർവർ ഫയലിലേക്ക് ചേർക്കേണ്ടത്:

**Python**

```python
# സംക്ഷിപ്തതക്ക് കോഡ് ഒഴിവാക്കിയിരിക്കുന്നു
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

`@server.list_tools` ഡെക്കറേറ്റർയും `handle_list_tools` ഫംഗ്ഷനും ഇവിടെ ചേർക്കുന്നു. `handle_list_tools` ടൂളുകളുടെ പട്ടിക തിരികെ നൽകണം. ഓരോ ടූലിനും name, description, inputSchema ഫീൽഡുകൾ വേണം.   

**TypeScript**

ടൂൾ ലിസ്റ്റിങ്ങിനായി റിക്വസ്റ്റ് ഹാൻഡ്ലർ സജ്ജീകരിക്കാൻ, `setRequestHandler` സർവറിൽ വിളിക്കുമ്പോൾ `ListToolsRequestSchema` പോലുള്ള സ്കീമ നൽകണം:

```typescript
// ഇൻഡക്സ്സ്.ടിഎസ്
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// സർവർ.ടിഎസ്
// ചെറിയതാക്കാൻ കോഡ് ഒഴിവാക്കിയിട്ടുണ്ട്
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // രജിസ്റ്റർ ചെയ്തിട്ടുള്ള ഉപകരണങ്ങളുടെ പട്ടിക തിരിച്ച് നൽകുക
  return {
    tools: tools
  };
});
```

ശരി, ടൂൾ ലിസ്റ്റിങ്ങിൽ പ്രശ്‌നം പരിഹരിച്ചിട്ടുണ്ട്. അടുത്തതായി ടൂളുകൾ വിളിക്കുന്നത് എങ്ങനെയാണെന്ന് നോക്കാം.

### -4- ടൂൾ കോളിംഗ് കൈകാര്യം ചെയ്യൽ

ടൂൾ വിളിക്കാൻ മറ്റൊരു റിക്വസ്റ്റ് ഹാൻഡ്ലർ സജ്ജീകരിക്കണം, ഇത് വിളിക്കാനുള്ള ഫീച്ചർ പറ്റിയ വിവരം (എന്താണ് ടൂൾ നമ്പർ) നൽകിയിരിക്കണം, കൂടാതെ ആർക്ക്യുമെന്റുകളും സർവ്വീസിന് ലഭിക്കണം.

**Python**

`@server.call_tool` ഡെക്കറേറ്റർ ഉപയോഗിച്ച് `handle_call_tool` ഫംഗ്ഷൻ നടപ്പാക്കാം. ഈ ഫംഗ്ഷനിൽ ടൂൾ നാമവും ആർക്ക്യുമെന്റുകളും പാഴ്സുചെയ്യാം, അവ ശരിയാണോ എന്ന് ഉറപ്പാക്കും. വാലിഡേഷൻ ഇങ്ങോ ഈ ഫംഗ്ഷനിൽ, അല്ലെങ്കിൽ ടൂളിന്റെ ലോയറിൽ നടക്കാം.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools എന്നത് ടൂൾ നാമങ്ങളുള്ള കീകളുള്ള ഒരു നിഘണ്ടുവാണ്
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # ടൂൾ വിളിക്കുക
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

കീഴെ നടന്നത്:

- ടൂൾ നാമം `name` എന്ന ഇൻപുട്ട് പാരാമീറ്ററിൽ നേരത്തെ ലഭ്യമാണ്, ആർക്ക്യുമെന്റുകൾ `arguments` ഡിക്ഷണറിയിൽ.
- ടൂൾ വിളിക്കുന്നത് `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` വഴി. വാലിഡേഷൻ `handler` ഫംഗ്ഷനിൽ. പരാജയം ഉണ്ടെങ്കിൽ എക്‌സപ്ഷൻ ഉയരും.

ഇവിടെ, ലോ-ലെവൽ സർവർ ഉപയോഗിച്ച് ടൂളുകൾ ലിസ്റ്റ് ചെയ്യാനും വിളിക്കാനും പൂർണ്ണമായ ധാരണ ലഭിച്ചു.

[ഫുൾ എക്സാമ്പിൾ](./code/README.md) ഇവിടെ കാണാൻ കഴിയും

## അസൈൻമെന്റ്

നൽകിയിരിക്കുന്ന കോഡ് വിപുലീകരിച്ചു ടൂളുകൾ, റിസോഴ്‌സുകൾ, പ്രോംപ്റ്റുകളെ അധികമാക്കുക. നിങ്ങൾ കാണുന്നത്, ടൂളുകൾ ഡയറക്ടറിയിൽ മാത്രമേ ഫയലുകൾ ചേർക്കേണ്ടതാവു, മറ്റെവിടെയും ഇല്ല.

*ഉത്തരം നൽകിയിട്ടില്ല*

## സംഗ്രഹം

ഈ അധ്യായത്തിൽ, ലോ-ലെവൽ സർവർ സമീപനം എങ്ങനെയാണെന്ന് കണ്ടു, അത് നല്ല ആർക്കിടെക്ചർ സൃഷ്ടിക്കാൻ എങ്ങനെ സഹായിക്കുന്നു എന്നറിയാമായി. കൂടാതെ വാലിഡേഷൻ ഉണ്ടാക്കുന്നത് എങ്ങനെ എന്നും പഠിച്ചു, ഇൻപുട്ട് സ്കീമകൾ സൃഷ്‌ടിക്കാൻ വാലിഡേഷൻ ലൈബ്രറികൾ ഉപയോഗിക്കാനുള്ള മാർഗ്ഗനിർദ്ദേശം ലഭിച്ചു.

## പിന്നെ എന്താണ്

- അടുത്തത്: [സിമ്പിൾ ഓതന്റിക്കേഷൻ](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ഡിസ്ക്ലെയ്‌മർ**:  
ഈ രേഖ [Co-op Translator](https://github.com/Azure/co-op-translator) എന്ന എഐ ഭാഷান্তര സേവനം ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ സ്ഥിരീകരണത്തിന് ശ്രമിച്ചെങ്കിലും, സ്വയംമാറ്റിയ പരിഭാഷയിൽ പിഴവുകളോ അകുറ്റലുകളോ ഉണ്ടായിരിക്കാമെന്ന് ദയവായി ഗൃഹീകരിക്കുക. ഒറിജിനൽ ഡോക്യുമെന്റ് സമ്പൂർണ്ണമായും അവിന്റെ യഥാർത്ഥ ഭാഷയിലെ പ്രാപ്തമായ ഉറവിടം ആയി കരുതപ്പെടണം. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ ആയ മനുഷ്യഭാഷാന്തരം നിർദ്ദേശിക്കുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ചത് മൂലം ഉണ്ടാകുന്ന ഏതെന്തെങ്കിലും തെറ്റിദ്ധാരണകൾക്ക് ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
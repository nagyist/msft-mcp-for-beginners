# மேம்பட்ட சர்வர் பயன்பாடு

MCP SDK இல் இரண்டு விதமான சர்வர்கள் உள்ளன, உங்கள் சாதாரண சர்வர் மற்றும் குறைந்த நிலை சர்வர். சாதாரணமாக, நீங்கள் சேர்க்க அம்சங்களைச் சேர்க்க சாதாரண சர்வரைப் பயன்படுத்துவீர்கள். சில பாத்திரங்களில், நீங்கள் குறைந்த நிலை சர்வரை சார்ந்திருப்பீர்கள், உதாரணமாக:

- சிறந்த கட்டமைப்பு. சாதாரண சர்வரும் குறைந்த நிலை சர்வரும் இரண்டும் ஒரு சுத்தமான கட்டமைப்பை உருவாக்குவதாக முடியும், ஆனால் குறைந்த நிலை சர்வருடன் அது சுலபமாக இருக்கக் கூடும் என்று வாதிடலாம்.
- அம்சக் கிடைக்கும் தன்மை. சில மேம்பட்ட அம்சங்கள் குறைந்த நிலை சர்வருடன் மட்டுமே பயன்படுத்தலாம். இதைப் பின்பு அத்தியாயங்களில், டேட்டா சேகரிப்பு மற்றும் எலிசிடேஷன் சேர்க்கும் போது காண்பீர்கள்.

## சாதாரண சர்வர் மற்றும் குறைந்த நிலை சர்வர்

சாதாரண சர்வர் மூலம் MCP சர்வர் உருவாக்குவது இதுபோன்று இருக்கும்

**Python**

```python
mcp = FastMCP("Demo")

# ஒரு கூட்டல் கருவியைச் சேர்க்கவும்
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

// ஒரு கூட்டும் கருவியைச் சேர்க்கவும்
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

நீங்கள் சர்வரில் சேர்க்க விரும்பும் ஒவ்வொரு கருவி, வளம் அல்லது புரம்ப்டை நீங்கள் தெளிவாகச் சேர்க்க வேண்டும் என்பது தான் கருத்து. இது தவறு இல்லை.

### குறைந்த நிலை சர்வர் அணுகுமுறை

ஆனால், குறைந்த நிலை சர்வர் அணுகுமுறையைப் பயன்படுத்தும்போது, ஒவ்வொரு அம்ச வகைக்கும் (கருவிகள், வளங்கள் அல்லது புரம்ப்ட்கள்) இரண்டு ஹேண்ட்லர்களை உருவாக்க வேண்டும். உதாரணமாக கருவிகள் இரண்டு செயல்பாடுகளையே கொண்டிருக்க வேண்டும்:

- அனைத்து கருவிகளையும் பட்டியலிடுதல். ஒரு செயல்பாடு அனைத்து கருவிகளின் பட்டியலை உருவாக்கப் பொறுப்பு.
- அனைத்து கருவிகளையும் அழைக்கும் செயல்பாடு. இங்கே ஒரு செயல்பாடு கருவி அழைப்புகளை கையாளும்.

இது சாத்தியமாக குறைவான வேலை என்று தோன்றுகிறதா? அதாவது ஒரு கருவியை பதிவுசெய்யும் பதிலாக, நான் கருவி பட்டியலில் இருப்பதை உறுதி செய்ய வேண்டும் மற்றும் அது அழைக்கும்போது அது அழைக்கப்பட வேண்டும்.

இப்போது குறியீடு எப்படி இருக்கும் என்பதைக் காணலாம்:

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
  // பதிவு செய்த கருவிகளின் பட்டியலை திரும்ப அனுப்பு
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

இங்கே ஒரு செயல்பாடு அம்சங்களின் பட்டியலை எடுத்து வழங்குகிறது. கருவி பட்டியலில் ஒவ்வொரு உருப்படியும் `name`, `description` மற்றும் `inputSchema` போன்ற புலங்களைக் கொண்டிருக்க வேண்டும். இது கருவிகள் மற்றும் அம்ச வரைவுகளை வேறு இடத்தில் வைக்க உதவுகிறது. நாம் அனைத்து கருவிகளையும் ஒரு tools என்ற கோப்புறையிலே உருவாக்கலாம், அதேபோல் உங்கள் அனைத்து அம்சங்களும் சேர்ந்து உங்களின் திட்டம் இதுபோன்ற அமைப்பினோடு ஒழுங்குபடுத்தப்படும்:

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

சிறந்தது, எங்கள் கட்டமைப்பு மிகவும் சுத்தமாக இருக்கலாம்.

கருவி அழைப்பது அவ்வாறு தான் அல்லவா? ஒரு ஹேண்ட்லர் ஒரே கருவியையும் அழைக்கும், எந்த கருவி என்றாலும்? ஆம், சரி, இதோ அதற்கான குறியீடு:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools என்பது கருவி பெயர்கள் முக்கியத்துவமாகக் கொண்ட ஒரு அகராதி ஆகும்
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
    // செய்ய வேண்டியது கருவியை அழைக்கவும்,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

மேலுள்ள குறியீட்டை பார்த்தால், நாம் அழைக்க வேண்டிய கருவியை மற்றும் வாதங்களை பிரிக்க வேண்டும், பிறகு கருவியை அழைக்கும்.

## சரிபார்ப்பு கொண்டு அணுகுமுறையை மேம்படுத்துதல்

இன்னும், நீங்கள் எப்படி கருவிகள், வளங்கள், புரம்ப்ட்களைச் சேர்க்கும் பதிவு முறைகளை இந்த இரண்டு ஹேண்ட்லர்களால் மாற்ற முடியும் என்பதை பார்த்துள்ளீர்கள். பிறகு என்ன செய்ய வேண்டும்? கருவி சரியான வாதங்களுடன் அழைக்கப்படுகிறதா என்பதை உறுதிசெய்ய ஒரு வகை சரிபார்ப்பைச் சேர்க்க வேண்டும். ஒவ்வொரு ரன்டைமிலும் தனித்துவமாக தீர்வு உண்டு, உதாரணமாக Python Pydantic பயன்படுத்துகிறது, TypeScript Zod பயன்படுத்துகிறது. யோசனை இதுவாகும்:

- ஒரு அம்சத்தை உருவாக்கும் (கருவி, வளம் அல்லது புரம்ப்ட்) இயல்பை அதன் தனி கோப்புறைக்கு மாற்றுதல்.
- ஒரு கோரிக்கையை சரிபார்க்க ஒரு முறையைச் சேர்ப்பது, உதாரணமாக ஒரு கருவியை அழைக்க கோரிக்கை வந்தால் அது சரியானதா என.

### ஒரு அம்சத்தை உருவாக்குதல்

ஒரு அம்சத்தை உருவாக்க, அந்த அம்சத்திற்கான கோப்பை உருவாக்கி அந்த அம்சத்திற்கான அவசியமான புலங்களை உள்ளிட வேண்டும். புலங்கள் கருவிகள், வளங்கள் மற்றும் புரம்ப்டுகள் மாதிரி கொஞ்சம் வித்தியாசமாக இருக்கும்.

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
        # Pydantic மாடல் பயன்படுத்தி உள்ளீட்டைக் சரிபார்க்கவும்
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # செய்திக்குறிப்பு: Pydantic ஐ சேர்க்கவும், அதனால் நாம AddInputModel ஐ உருவாக்கி பாராமீட்டர்களை சரிபார்க்க முடியும்

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

இங்கே செய்வது:

- Pydantic `AddInputModel` ஆன ஸ்கீமாவை *schema.py* என்ற கோப்பில் புலங்கள் `a` மற்றும் `b` உடன் உருவாக்குதல்.
- வந்த கோரிக்கையை `AddInputModel` வகையாகப் பார் செய்து பார்க்கிறது, இணக்கமில்லாமல் இருந்தால் இது பிழை ஏற்படும்:

   ```python
   # add.py
    try:
        # Pydantic மாதிரியை பயன்படுத்தி உள்ளீட்டை சரிபார்க்கவும்
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

இந்த பார் செய்வதை கருவி அழைப்பில் நேரடியாகச் செய்யலாம் அல்லது ஹேண்ட்லர் செயல்பாட்டில் செய்யலாம்.

**TypeScript**

```typescript
// சர்வர்.ts
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

       // @ts-புறக்கணி
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

// திட்டம்.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// சேர்க்க.ts
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

- எல்லா கருவி அழைப்புகளையும் கையாளும் ஹேண்ட்லரில், வந்த கோரிக்கையை கருவியின் வரையறுக்கப்பட்ட ஸ்கீமாக மாற்ற முயற்சிக்கின்றோம்:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    அது இயன்றால் உண்மையான கருவியை அழைக்க போகின்றோம்:

    ```typescript
    const result = await tool.callback(input);
    ```

இவ்வாறு கட்டமைப்பு சிறப்பாக இருக்கும், *server.ts* என்பது ஒரு மிகச் சிறிய கோப்பை மட்டுமே கொண்டுள்ளது மற்றும் ஒவ்வொரு அம்சமும் தக்க கோப்புறைகளிலேயே உள்ளது, உதாரணமாக tools/, resources/ அல்லது /prompts/.

சரி, அடுத்ததாக இதை உருவாக்க முயற்சிப்போம்.

## பயிற்சி: குறைந்த நிலை சர்வர் உருவாக்குதல்

இந்த பயிற்சியில், நாம் பின்வருமாறு செய்வோம்:

1. கருவிகள் பட்டியலை கையாளும் மற்றும் கருவிகளை அழைக்கும் குறைந்த நிலை சர்வரை உருவாக்கல்.
2. நீங்கள் மேம்படுத்தக்கூடிய கட்டமைப்பை செயல்படுத்தல்.
3. உங்கள் கருவி அழைப்புகள் சரிபார்க்கப்படுவதை உறுதிசெய்ய சரிபார்ப்பைச் சேர்த்தல்.

### -1- கட்டமைப்பை உருவாக்குதல்

முதலில், மிக அதிக அம்சங்களை உள்ளடக்கும்போது அளவுரு பெற உதவும் கட்டமைப்பை உருவாக்க வேண்டும், இதுபோல இருக்கும்:

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

இப்போது நாங்கள் ஒரு கட்டமைப்பை அமைத்துள்ளோம், இதில் புதிய கருவிகளை tools என்ற கோப்புறையில் எளிதில் சேர்க்கலாம். வளங்களுக்கும் புரம்ப்டுகளுக்கும் துணை கோப்புறைகளைச் சேர்க்கலாம் இத்த கருவியில் பின்பற்றலாம்.

### -2- கருவி உருவாக்குதல்

அடுத்து, ஒரு கருவி உருவாக்குவது எப்படி என்பதைப் பார்க்கலாம். முதலில், அது *tool* துணை கோப்புறையில் உருவாக்கப்பட வேண்டும்:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic மாதிரியை பயன்படுத்தி உள்ளீட்டை சரிபார்க்கவும்
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # செய்யவேண்டியது: Pydantic ஐச் சேர்க்கவும், ஆகையால் நாம் AddInputModel ஐ உருவாக்கி args ஐ சரிபார்க்கலாம்

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

இங்கே நாம் பெயர், விளக்கம், Pydantic பயன்படுத்தி உள்ளீட்டு ஸ்கீமா, மேலும் இந்த கருவி அழைக்கப்படும் போது இயக்கப்படும் ஹேண்ட்லரை வரைவாகக் காண்கிறோம். கடைசியில், `tool_add` என்ற அகராதியை வெளியிடுகிறோம், இது அனைத்து பண்புகளையும் கொண்டுள்ளது.

மேலும், *schema.py* உள்ளது, இது கருவியின் உள்ளீட்டு ஸ்கீமாவை வரையறுக்க உதவுகிறது:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

*__init__.py* கோப்பில் இதனை நமக்கு மட்யூலை என சிகிச்சை செய்ய வேண்டும். மேலும, அதில் உள்ள மட்யூலை வெளியிட இதுபோன்று செய்கிறோம்:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

பிறகு மேல் இன்னும் கருவிகளைச் சேர்க்க இந்த கோப்பை தொடர்ந்தே மேம்படுத்தலாம்.

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

இங்கே பண்புகளைக் கொண்ட ஒரு அகராதி உருவாக்குகிறோம்:

- name, இது கருவியின் பெயர்.
- rawSchema, இது Zod ஸ்கீமா, கருவி அழைக்கும் கோரிக்கைகளை சரிபார்க்க பயன்படுத்தப்படும்.
- inputSchema, இந்த ஸ்கீமா ஹேண்ட்லர் பயன்படுத்தும்.
- callback, இது கருவியை இயக்கும்.

`Tool` உள்ளது, இது அகராதியை MCP சர்வர் ஹேண்ட்லர் ஏற்கக்கூடிய வகையாக மாற்றுகிறது:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

மேலும், *schema.ts* உள்ளது, இங்கு அனைத்து கருவிகளுக்குமான உள்ளீட்டு ஸ்கீமாக்கள் சேகரிக்கப்பட்டுள்ளன, தற்போதைக்கு ஒரே ஸ்கீமாவைக் கொண்டது, புதிய கருவிகளுடன் சேர்க்கலாம்:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

சரி, அடுத்து கருவிகள் பட்டியலை கையாள கடைசியாக செல்கின்றோம்.

### -3- கருவிகள் பட்டியலை கையாளுதல்

பட்டியலை கையாள கோரிக்கை ஹேண்ட்லரை அமைக்க வேண்டும். இதோ அதை செர்‌வர் கோப்பில் சேர்க்க தேவையானது:

**Python**

```python
# குறுகிய வடிவத்துக்காக குறியீடு நீக்கப்பட்டது
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

இங்கே `@server.list_tools` என்ற ஆலங்காரம் மற்றும் `handle_list_tools` என்ற செயல்பாட்டைச் சேர்க்கிறோம். இதில் பட்டியல் உருவாக்க வேண்டும். ஒவ்வொரு கருவிக்கும் பெயர், விளக்கம் மற்றும் inputSchema தேவை.

**TypeScript**

கருவிகள் பட்டியலுக்கான கோரிக்கை ஹேண்ட்லரை தொகுக்கும் போது, சரியான ஸ்கீமாவுடன் `setRequestHandler` ஐ அழைக்க வேண்டும், இக்கேஸில் `ListToolsRequestSchema` ஆகும்:

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
// எளிதாக்க பிரதேசமாக குறியீடு வெளியேற்றப்பட்டது
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // பதிவு செய்யப்பட்ட கருவிகளின் பட்டியலை திருப்பவும்
  return {
    tools: tools
  };
});
```

சரி, இப்போது கருவிகள் பட்டியலிடப்படுவதைக் கையாள முடிந்தது, அடுத்து கருவிகளை அழைக்க எப்படி என்பதை பார்க்கலாம்.

### -4- கருவி அழைப்பை கையாளுதல்

கருவியை அழைக்க மற்றொரு கோரிக்கை ஹேண்ட்லரை அமைக்க வேண்டும், இது எது அம்சம் அழைக்கப்படுகிறது மற்றும் அதை அழைக்கும் வாதங்கள் என்ன என்பதை கவனிக்கும்.

**Python**

`@server.call_tool` ஆலங்காரத்துடன் `handle_call_tool` என்பதை உருவாக்கலாம். இதில், கருவியின் பெயர், வாதங்களை பிரித்து, அவை சரியானவையா எனச் சரிபார்க்க வேண்டும். வாதங்களை இங்கே அல்லது கருவியின் உள்ளே சரிபார்க்கலாம்.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools என்பது கருவி பெயர்கள் முக்கியமாக உள்ள ஒரு அகராதி ஆகும்
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # கருவியை அழைக்கவும்
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

இதோ விவரம்:

- கருவி பெயர் உருபடி பெயராக `name` உள்ளது; வாதங்கள் `arguments` அகராதியில் உள்ளது.
- கருவி `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` ஆக அழைக்கப்படுகிறது. வாதங்கள் ஹேண்ட்லர் சொத்து செயல்பாட்டில் சரிபார்க்கப்படுகிறது, தவறாயின் மிகுதிப்பிழை ஏற்படும்.

இனி குறைந்த நிலை சர்வரில் கருவிகளை பட்டியலிடும் மற்றும் அழைக்கும் முறையை முழுமையாக புரிந்துகொண்டோம்.

[முழு எடுத்துக்காட்டு](./code/README.md) இங்கே

## பணியை ஒப்படைப்பு

உங்களுக்கு கொடுக்கப்பட்ட குறியீட்டைப் பெருக்கி பல கருவிகள், வளங்கள் மற்றும் புரம்ப்ட்களைச் சேர்க்கவும் மற்றும் நீங்கள் ஒரே tools கோப்புறையில் கோப்புகள் சேர்க்கவேண்டியதாக மட்டுமே கவனித்தீர்களா என்பதைப்பாருங்கள்.

*பதிலளிப்பு இல்லை*

## சுருக்கம்

இந்த அத்தியாயத்தில், குறைந்த நிலை சர்வர் அணுகுமுறையைக் காண்ந்து அது எப்படிப் பணி புரியும் மற்றும் நாம் கட்டமைப்பை எளிதில் உருவாக்க முடியும் என்பதையும் கற்றோம். மேலும், சரிபார்ப்பை பற்றி விவாதித்து அதற்கான நூலகங்களைப் பயன்படுத்தி உள்ளீட்டு சரிபார்ப்பை செய்யும் முறையைப் பார்த்தோம்.

## அடுத்து என்ன

- அடுத்தது: [எளிய அங்கீகாரம்](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**குறிப்பு**:  
இந்த ஆவணம் AI மொழிப்பெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) மூலம் மொழிபெயர்க்கப்பட்டது. நாங்கள் துல்லியமான மொழிபெயர்ப்பு கொடுக்க முயலினாலும், தானியங்கி மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்க வாய்ப்பு உள்ளது என்பதை தயவு செய்து கவனிக்கவும். அசல் ஆவணம் அதன் சொந்த மொழியிலேயே அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்முறை மனித மொழிபெயர்ப்பை பரிந்துரைக்கின்றோம். இந்த மொழிபெயர்ப்பை பயன்படுத்தியதிலிருந்து ஏற்பட்ட எந்த வகை தவறுபாடுகளுக்கும் நாங்கள் பொறுப்பற்றவர்கள்.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
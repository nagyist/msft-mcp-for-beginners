# အဆင့်မြင့် ဆာဗာ အသုံးပြုခြင်း

MCP SDK တွင် ဖော်ပြထားသည့် ဆာဗာအမျိုးအစားနှစ်မျိုးရှိပြီး၊ သင်၏ ပုံမှန် ဆာဗာနှင့် နိမ့်ဆုံးအဆင့် ဆာဗာဖြစ်သည်။ ပုံမှန်အားဖြင့်၊ သင်သည် ပုံမှန် ဆာဗာကို အသုံးပြု၍ အင်္ဂါရပ်များ ထည့်သွင်းသည်။ သို့သော် ထူးခြားသောအခါများတွင် နိမ့်ဆုံးအဆင့် ဆာဗာကို မူတည်လိုနိုင်သည်၊ ဥပမာ-

- ပိုမိုကောင်းမွန်သော ဖန်တီးမှု။ ပုံမှန် ဆာဗာနှင့် နိမ့်ဆုံးအဆင့် ဆာဗာနှစ်ခုလုံးဖြင့် သန့်ရှင်းသော ဖန်တီးမှုတစ်ခု ဖန်တီးနိုင်သော်လည်း နိမ့်ဆုံးအဆင့် ဆာဗာဖြင့် ပိုမိုလွယ်ကူနိုင်ကြောင်း တွက်ချက်နိုင်သည်။
- အင်္ဂါရပ်များ ရရှိနိုင်မှု။ တချို့အဆင့်မြင့် အင်္ဂါရပ်များကို နိမ့်ဆုံးအဆင့် ဆာဗာဖြင့်သာ အသုံးပြုနိုင်သည်။ ယင်းကို နောက်ပိုင်းခန်းတွင် ပြသသွားမည် ဒါမှမဟုတ် sampling နှင့် elicitation ထည့်သွင်းသည်။

## ပုံမှန် ဆာဗာ vs နိမ့်ဆုံးအဆင့် ဆာဗာ

ပုံမှန် ဆာဗာဖြင့် MCP ဆာဗာတစ်ခု ဖန်တီးခြင်းကဒီလိုပုံစံဖြစ်သည် -

**Python**

```python
mcp = FastMCP("Demo")

# ပေါင်းသည့်ကိရိယာတစ်ခုထည့်ပါ
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

// ပေါင်းထည့် နည်းပညာတစ်ခုကို ထည့်ပါ
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

အဓိကအချက်ကတော့ ဆာဗာမှာ ပါရမည့် ကိရိယာ၊ အရင်းမြစ် သို့မဟုတ် prompt အသီးသီးကို သတ်မှတ်ပေးရပါသည်။ မည်သည့်အရာမှ ရိုးရိုး မဟုတ်ပါ။

### နိမ့်ဆုံးအဆင့် ဆာဗာနည်းလမ်း

သို့သော် နိမ့်ဆုံးအဆင့် ဆာဗာနည်းလမ်းကိုအသုံးပြုသောအခါ သင်သည် အခြားသဘောထားဖြင့်စဉ့်စားရမည်၊ မည်သည့်ကိရိယာမှမှတ်ပုံတင်ခြင်း မလုပ်ပဲ၊ အမျိုးအစားတစ်ခုစီ (ကိရိယာများ၊ အရင်းမြစ်များ သို့မဟုတ် prompt များ) အတွက် handler နှစ်ခုကို ဖန်တီးရမည်ဖြစ်သည်။ ဥပမာ နိမ့်ဆုံးအဆင့် ဆာဗာတွင် ကိရိယာများအတွက် ဖော်ပြပါအတိုင်း function နှစ်ခုသာရှိသည်-

- ကိရိယာများအားလုံးကို စာရင်းပြုစုခြင်း။ function တစ်ခုက ကိရိယာများအားလုံးရဲ့ စာရင်းကို စီမံသည်။
- ကိရိယာအားလုံးကို ခေါ်ဆိုခြင်းကို ကူညီဆောင်ရွက်ခြင်း။ ဒီမှာလည်း၊ function တစ်ခုတည်း သာ ကိရိယာခေါ်ဆိုမှုများကို ချက်ချင်း ဆောင်ရွက်သည်။

ဒါက လုပ်ရတာ သာမန်ထက် သာမနည်း လွယ်ကူတာမျိုး တစ်ခုလား? ကိရိယာ တစ်ခုစီအား မှတ်ပုံတင်ခြင်း လုပ်ရန်မလိုပဲ ကိရိယာအားလုံး စာရင်းထဲမှာ ပါနေဖို့နဲ့ ကိရိယာခေါ်ဆိုမယ့် အင်္ဂါရပ်လက္ခဏာရှိတဲ့ အခါမှာ ခေါ်ဆိုဖို့ တာဝန်ယူရုံပဲ ဖြစ်ပါတယ်။

ယခု ကုဒ်ကို ကြည့်ပါ-

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
  // မှတ်ပုံတင်ထားသော စက်ပစ္စည်းများ စာရင်းကို ပြန်ပေးပါ။
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

ယခုတွင် အင်္ဂါရပ်စာရင်း တစ်ခု ပြန်လည်ပေးသော function တစ်ခုရှိသည်။ ကိရိယာစာရင်းတွင် `name`, `description`, နှင့် `inputSchema` ကဲ့သို့သော field များပါဝင်ပြီး မည်သည့် return type ကို သဟဇာတဖြစ်စေရန်ဖြစ်သည်။ ဒီကိရိယာနှင့် အင်္ဂါရပ် သတ်မှတ်ချက်များကို တခြားနေရာတွင်ထားနိုင်သည်။ ကိရိယာအားလုံးကို tools ဖိုလ်ဒါထဲတွင် ဖန်တီးနိုင်ပြီး မိမိ project ကို အောက်ပါအတိုင်း စီမံခန့်ခွဲသွားနိုင်သည်-

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

အဲဒါကောင်းတယ်၊ ကျွန်ုပ်တို့ရဲ့ ဖန်တီးမှုကို သန့်ရှင်းလှပစေမယ်။

ကိရိယာခေါ်ဆိုခြင်းကော၊ အဲ့ဒါကလည်း တစ်ချက် handler တစ်ခုသာ ကိရိယာတစ်ခုခုကို ခေါ်ဆိုမှု ဖော်ဆောင်မှာလား? ဟုတ်ပါတယ်၊ ဒါက function ကုဒ်ဖြစ်ပါတယ်-

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools သည် tool အမည်များကို key အဖြစ် ရှိသော dictionary တစ်ခုဖြစ်သည်
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
    // TODO ကိရိယာကို ခေါ်ပါ,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

အထက်ပါ ကုဒ်တွင် ကိရိယာကို ဘယ်မှာဘယ်လို arguments အနေနဲ့ ခေါ်မယ်ဆိုတာ ပြောင်းပြန်ခွဲခြမ်းဖော်ထုတ်ရုံဖြစ်ပြီး၊ ကြိုတင် arguments ပေါ်မူတည်ပြီး ခေါ်ဆိုမှုကို ဆက်လက်လုပ်ဆောင်ရမည် ဖြစ်သည်။

## အသုံးပြုမှုပုံစံတိုးတက်စေခြင်း (validation နဲ့)

ယနေ့အထိ သင်သည် ကိရိယာ၊ အရင်းမြစ် နှင့် prompt များအား အသစ်ထည့်သွင်းရာတွင် မည်သည့် feature အမျိုးအစားအတွက် handler နှစ်ခု သာ အသုံးပြုနိုင်ကြောင်း ကြည့်မြင်ပြီးပါပြီ။ နောက်တစ်ခုတော့ validation အဖြစ် အချက်မှာ ရှိသင့်သည်။ လက်ရှိ runtime တစ်ခုချင်းစီမှာ မတူညီတဲ့ validation ဖြေရှင်းနည်းများ ရှိကြပြီး၊ Python သည် Pydantic ကို အသုံးပြုကာ၊ TypeScript သည် Zod ကို အသုံးပြုသည်။ 

အယူအဆမှာ အောက်ပါအတိုင်း ဖြစ်သည်-

- အင်္ဂါရပ်တစ်ခု (ကိရိယာ၊ အရင်းမြစ် သို့မဟုတ် prompt) ဖန်တီးရာ လုပ်ဆောင်ချက်ကို သီးသန့် ဖိုလ်ဒါတစ်ခုတွင်ထားသည်။
- လာရောက်သော တောင်းဆိုမှုတစ်ခုကို ကိရိယာခေါ်ဆိုမှုအနေဖြင့် စစ်ဆေးရန် နည်းလမ်း တစ်ခု ထည့်သွင်းသည်။

### အင်္ဂါရပ်တစ်ခု ဖန်တီးခြင်း

အင်္ဂါရပ်တစ်ခု ဖန်တီးရန်အတွက် file တစ်ခု ပြုလုပ်ပြီး အင်္ဂါရပ်ဟူသော အစိတ်အပိုင်းများမှာ တစ်ခုချင်းစီ မလိုအပ်နိုင်သော field များ ပါရှိကြောင်း အာမခံရမည်။ tools, resources နှင့် prompts တွင် မတူညီသော fields များ ရှိနိုင်သည်။

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
        # Pydantic မော်ဒယ်ကို အသုံးပြု၍ အချက်အလက်များကို စစ်ဆေးပါ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ကို ထပ်ထည့်ရမည်၊ ဒါမှ addInputModel ကို ဖန်တီးပြီး args များကို စစ်ဆေးနိုင်မည်ဖြစ်သည်။

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ဒီမှာ ကျွန်ုပ်တို့သည် အောက်ပါအတိုင်း လုပ်ဆောင်သည်။

- Pydantic အသုံးပြုပြီး schema `AddInputModel` ကို `a` နှင့် `b` field များဖြင့် *schema.py* ဖိုင်ထဲ သတ်မှတ်သည်။
- လာတဲ့ တောင်းဆိုမှုကို `AddInputModel` အမျိုးအစားဖြင့် ပြောင်းလိုက်ပြီး parameter မတွေ့လျှင် သိပ်သည်းလွှတ်မှုရပါမည် -

   ```python
   # add.py
    try:
        # Pydantic ပုံစံကို အသုံးပြု၍ နောက်ခံထားသော အချက်အလက်များကို စစ်ဆေးပါ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

ဒီ parsing ဝိုင်းလျားမှုကို ကိရိယာခေါ်ခုနှစ်ခြင်း သို့မဟုတ် handler function ထဲတွင် ထည့်နိုင်သည်။

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

- ကိရိယာခေါ်နိုင်ရန် စီစစ်နေသော handler တွင် လာတဲ့ တောင်းဆိုမှုကို tool ရဲ့ သတ်မှတ်ထားသော schema ပြောင်းဖို့ ကြိုးစားသည်-

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    အောင်မြင်လျှင် ကိရိယာကို ခေါ်ဆို ဆောင်ရွက်သည်-

    ```typescript
    const result = await tool.callback(input);
    ```

ဒီနည်းလမ်းသည် ဖန်တီးမှုကို စနစ်တကျ ပြုလုပ်နိုင်စေပြီး *server.ts* ဖိုင်သည် request handler များကို ကျဉ်းမြောင်း စီမံရာဖြစ်သည်။ အင်္ဂါရပ်တစ်ခုချင်းစီကို tools/, resources/ သို့မဟုတ် prompts/ ဖိုင်တွဲများတွင် ထားရှိသည်။

အရမ်းကောင်းတာပဲ၊ နောက်တစ်ခုကို တည်ဆောက်ကြပါစို့။

## လက်တွေ့လုပ်ငန်း: နိမ့်ဆုံးအဆင့် ဆာဗာ ဖန်တီးခြင်း

ဒီ လေ့ကျင့်မှုတွင် လုပ်ဆောင်ရမည့်အချက်များမှာ -

1. ကိရိယာစာရင်းပြုစုခြင်းနှင့် ကိရိယာခေါ်ဆိုခြင်းအား ကိုင်တွယ်နိုင်သော နိမ့်ဆုံးအဆင့် ဆာဗာဖန်တီးခြင်း။
2. တည်ဆောက်မှုတစ်ခု အကောင်းတစ်ခု အဖြစ် အကောင်အထည်ဖော်ခြင်း။
3. ကိရိယာခေါ်ဆိုမှုများ အတိအကျစစ်ဆေးသည့် validation ထည့်သွင်းခြင်း။

### -1- တည်ဆောက်မှု ဖန်တီးခြင်း

ဤအဆင့်တွင် feature များ တိုးခြင်းအတွက် ရည်ရွယ်ချက်ရှိတဲ့ တည်ဆောက်မှုဖြစ်သည်-

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

ယခု tools ဖိုလ်ဒါတွင် ကိရိယာအသစ်များ ထည့်သွင်းနိုင်ရန် တည်ဆောက်မှု စတင်ပြီးဖြစ်သည်။ resources နှင့် prompts ဖိုလ်ဒါများအတွက် မိမိစိတ်ကြိုက် directory လည်း ထည့်နိုင်ပါသည်။

### -2- ကိရိယာတစ်ခု ဖန်တီးခြင်း

နောက်တစ်ခုက ကိရိယာတစ်ခု ဖန်တီးခြင်း ဖြစ်သည်။ ပထမဆုံးအနေဖြင့် *tool* ဖိုလ်ဒါအတွင်း ဖိုင်ချိန်ကြည့်ရမည်ဖြစ်သည်-

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic မော်ဒယ်ကို အသုံးပြု၍ အဝင်ကို စစ်ဆေးပါ
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ကို ထည့်သွင်းပါ၊ ဒါမှသာ AddInputModel ချည်း နှင့် args များကို စစ်ဆေးနိုင်ပါမည်

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ဒီမှာ ကျွန်ုပ်တို့သည် name, description, Pydantic ကိုအသုံးပြုပြီး input schema နှင့် tool ကို ခေါ်ဆိုချိန်မှာ ဖော်ပြထားသော handler တစ်ခု ဖန်တီးခဲ့သည်။ နောက်ဆုံးတွင် `tool_add` ဟု ဆိုသော property များ ပါသော dictionary ကို ပြသထားသည်။

*schema.py* တွင် ကိရိယာအတွက် အသုံးပြုမည့် input schema သတ်မှတ်ထားသည်။

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

*__init__.py* ဖိုင်ကိုပါ ဖြည့်စွက်၍ tools directory ကို module အဖြစ် သတ်မှတ်မှု လုပ်ရမည်၊ ထို့အပြင် modules များကိုပြသရန်：

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

နောက်ထပ် tools များ ထည့်သွင်းရန် ဒီဖိုင်ထဲ အမြဲထပ်မံထည့်နိုင်ပါသည်။

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

ဒီမှာ သက်ဆိုင်ရာ property များပါအုံးသည်-

- name, ကိရိယာအမည်။
- rawSchema, Zod schema ဖြစ်ပြီး ကိရိယာခေါ်ဆိုမှုများ စစ်ဆေးရန် အသုံးပြုသည်။
- inputSchema, handler အတွက် အသုံးပြုသည့် schema။
- callback, ကိရိယာကို ခေါ်ဆိုရန် အသုံးပြုသည်။

`Tool` ဟူသောမျိုးက ဒီ dictionary ကို mcp server handler သံဃာတစ်ခုလက်ခံနိုင်သော type အဖြစ် ပြောင်းပေးသည်။

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

*schema.ts* မှာ ကိရိယာတစ်ခုချင်း input schema များ ကို ထည့်သွင်းသော ဖိုင် ဖြစ်ပြီး လက်ရှိတွင် schema တစ်ခုတည်းရှိသော်လည်း ကိရိယာများ ပေါင်းထပ်လာသည်နှင့် အပိုင်းများ ထပ်မံထည့်သွင်းနိုင်သည်။

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

ကောင်းပါပြီ၊ tools စာရင်း ပြုစုခြင်းကို ဆက်လုပ်ကြည့်ရအောင်။

### -3- ကိရိယာစာရင်း ကို ကိုင်တွယ်ခြင်း

tools စာရင်းကို ကိုင်တွယ်ရန် တောင်းဆိုချက် handler တစ်ခု ဖြည့်စွက်ရမည်။ ဆာဗာဖိုင်တွင် ထည့်သွင်းရန်လိုအပ်သော အချက်များ----

**Python**

```python
# အတိုချုပ်ရေးသားခြင်းအတွက် ကုဒ် ဖျက်ထားသည်
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

ဒီမှာ `@server.list_tools` decorator နှင့် `handle_list_tools` function ရှိသည်။ function မှ tools လုပ်ငန်းစဉ် အားလုံးရဲ့ စာရင်းကို ပြန်ပေးရမည်ဖြစ်ပြီး name, description နှင့် inputSchema သတ်မှတ်ချက်များ မဖြစ်မနေ လိုအပ်သည်။

**TypeScript**

tools စာရင်း ဖော်ပြရန် request handler ကို ဖန်တီးရာတွင် server ပေါ်မှ `setRequestHandler` ကိုခေါ်ပြီး မည်သည့် schema နှင့် ကိုက်ညီမည်ဆိုသည်မှာ `ListToolsRequestSchema` ဖြစ်သည်။ 

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
// ရုပ်သိမ်းခြင်းအတွက် ကုဒ်
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // စာရင်းသွင်းထားသော ကိရိယာများစာရင်းကို ပြန်ပေးပါ
  return {
    tools: tools
  };
});
```

ထိပ်တန်း tools စာရင်း ပြုစုမှု ပြီးစီးသွားပါပြီ၊ နောက်တစ်ဆင့် tools ခေါ်ဆိုဖို့ ကိုကြည့်ပါ။

### -4- ကိရိယာခေါ်ဆိုခြင်း ကို ကိုင်တွယ်ခြင်း

ကိရိယာကို ခေါ်ဆိုရန် ဆုံးဖြတ်ချက်နှင့် arguments များအား ဦးတည်မှု ရှိသော request handler တစ်ခု အသစ် ဖန်တီးရမည်။

**Python**

`@server.call_tool` decorator သုံးပြီး `handle_call_tool` function ဖြင့် ဆောင်ရွက်မည်။ function အတွင်းကိရိယာအမည်၊ အားပေး parameter များ ခွဲခြားထုတ်၍ ကိရိယာအတွက် သင့်လျော်မှု စစ်ဆေးရမည်။ သို့ဖြစ်၍ validation ကို function အတွင်း ထိန်းချုပ်နိုင်သော်လည်း တိုက်ရိုက် ကိရိယာအတွင်း ဆောင်ရွက်နိုင်ပါသည်။

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools သည် ကိရိယာအမည်များကို key များအဖြစ် အသုံးပြုထားသော dictionary တစ်ခုဖြစ်သည်
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # ကိရိယာအား ဖိတ်ခေါ်ပါ
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

အောက်ပါအချက်များ ဖြစ်ပေါ်သည်-

- ကိရိယာအမည်မှာ input parameter `name` အဖြစ် ရှိပြီး၊ argument များသည် `arguments` dictionary အတိုင်း ဖြစ်သည်။
- `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` ဖြင့် ကိရိယာကို ခေါ်ဆိုသည်။ arguments validation ကို handler ၌ ဖော်ပြထားသော function ဝင်ရောက် စစ်ဆေးသည်။ ထိုစစ်ဆေးမှု မအောင်မြင်မှန်းလျှင် exception ဖွဲ့စည်းမည်။

ဒီလိုနဲ့ နိမ့်ဆုံးအဆင့် ဆာဗာ အသုံးပြုပြီး ကိရိယာစာရင်း ပြုစုခြင်းနှင့် ခေါ်ဆိုခြင်းကို ကောင်းစွာ နားလည်ထားပါပြီ။

[full example](./code/README.md) ကို ဒီမှာ ကြည့်ရှုပါ

## တာဝန်

သင်လက်ခံထားသော ကုဒ်အပေါ် tools, resources နှင့် prompt များ ထပ်မံထည့်၍ tools directory သာတွင်သာ ဖိုင် ထည့်ရမည်ဖြစ်ကြောင်း သိရှိအောင် သက်သေပြပါ။

*ဖြေရှင်းချက် မပါရှိပါ*

## တိုတောင်းချက်

ဒီခန်းတွင် နိမ့်ဆုံးအဆင့် ဆာဗာနည်းလမ်း ပြုလုပ်ပုံနှင့် ရိုးရာ ဒီဇိုင်းတည့်တည့်ကို ဖန်တီးနိုင်သည့်နည်းလမ်းများ အသိပညာ ကြည့်ရှုခဲ့ပြီး၊ validation နှင့် validation libraries အသုံးပြုပြီး input schema များ ဖန်တီးနည်း ကိုလည်း သင်ကြားခဲ့ပါသည်။

## နောက်ဆုံးမှာ

- နောက်တစ်ခန်း: [Simple Authentication](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**စာသားငြင်းဆိုချက်**  
ဤစာတမ်းကို AI ဘာသာပြန်ခြင်းဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) ကိုအသုံးပြုပြီး ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းပါသော်လည်း အလိုအလျောက်ဘာသာပြန်ခြင်းတွင် အမှားများ သို့မဟုတ် မှားယွင်းမှုများ ဖြစ်ပေါ်နိုင်ကြောင်း သတိပြုပါရန် သတိပေးအပ်ပါသည်။ မူရင်းစာတမ်းကို မိမိတို့ဘာသာစကားဖြင့်သာ ကိုးကားစဉ်းစားသင့်ပါသည်။ အရေးကြီးသောအချက်အလက်များအတွက် လူ့ဘာသာပြန်ပညာရှင်မှ ပြန်ဆိုခြင်းကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက် အသုံးပြုရာမှ ဖြစ်ပေါ်နိုင်သည့် နားလည်မှုမှားယွင်းမှုများအတွက် ကျွန်ုပ်တို့အနေနှင့် တာဝန်မရှိပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# جدید سرور کے استعمال

MCP SDK میں دو مختلف قسم کے سرورز مہیا کیے گئے ہیں، آپ کا معمول کا سرور اور ایک لو لیول سرور۔ عام طور پر، آپ عام سرور کو فیچرز شامل کرنے کے لیے استعمال کرتے ہیں۔ کچھ صورتوں میں، آپ لو لیول سرور پر انحصار کرنا چاہیں گے جیسے کہ:

- بہتر آرکیٹیکچر۔ صاف ستھری آرکیٹیکچر بنانا ممکن ہے دونوں عام سرور اور لو لیول سرور کے ساتھ، لیکن کہا جا سکتا ہے کہ لو لیول سرور کے ساتھ یہ تھوڑا آسان ہے۔
- فیچر کی دستیابی۔ بعض جدید فیچرز صرف لو لیول سرور کے ساتھ استعمال کیے جا سکتے ہیں۔ آپ اسے اگلے ابواب میں دیکھیں گے جب ہم سیمپلنگ اور ایلیسیٹیشن شامل کریں گے۔

## عام سرور بمقابلہ لو لیول سرور

یہاں MCP سرور بنانے کا طریقہ عام سرور کے ساتھ دکھایا گیا ہے

**Python**

```python
mcp = FastMCP("Demo")

# ایک جمع کرنے کا آلہ شامل کریں
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

// ایک جمع کرنے کا آلہ شامل کریں
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

بات یہ ہے کہ آپ وضاحتاً ہر ٹول، ریسورس یا پرامپٹ شامل کرتے ہیں جو آپ چاہتے ہیں کہ سرور کے پاس ہو۔ اس میں کوئی غلطی نہیں ہے۔  

### لو لیول سرور کا طریقہ کار

تاہم، جب آپ لو لیول سرور طریقہ کار استعمال کرتے ہیں تو آپ کو اس بارے میں مختلف سوچنا ہوتا ہے، یعنی کہ ہر ٹول کو رجسٹر کرنے کی بجائے آپ فیچر کی قسم (ٹولز، ریسورسز یا پرامپٹس) کے لیے دو ہینڈلرز بناتے ہیں۔ مثال کے طور پر ٹولز کے لیے صرف دو فنکشن ہوتے ہیں:

- تمام ٹولز کی فہرست دینا۔ ایک فنکشن تمام کوششوں کے لیے ذمہ دار ہوگا کہ ٹولز کی فہرست دی جائے۔
- تمام ٹولز کو کال کرنا۔ یہاں بھی، ایک ہی فنکشن ہوتا ہے جو ٹول کی کالز کو ہینڈل کرتا ہے۔

یہ شاید کم کام لگتا ہے، صحیح ہے؟ تو ٹول کو رجسٹر کرنے کے بجائے، مجھے بس یہ یقینی بنانا ہے کہ جب میں تمام ٹولز کی فہرست دوں تو وہ فہرست میں ہو اور جب بھی کال کی درخواست آئے تو صحیح ٹول کو کال کیا جائے۔ 

آئیے دیکھتے ہیں کہ کوڈ اب کیسا نظر آتا ہے:

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
  // رجسٹرڈ شدہ آلات کی فہرست واپس کریں
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

یہاں اب ہمارے پاس ایک فنکشن ہے جو فیچرز کی فہرست واپس کرتا ہے۔ ٹولز کی فہرست کے ہر انٹری میں اب `name`, `description` اور `inputSchema` جیسی فیلڈز ہیں تاکہ ریٹرن ٹائپ کے مطابق ہوں۔ اس سے ہم اپنے ٹولز اور فیچر کی تعریف کہیں اور رکھ سکتے ہیں۔ ہم اب تمام اپنے ٹولز کو tools فولڈر میں بنا سکتے ہیں اور یہی بات تمام فیچرز کے لیے بھی لاگو ہوتی ہے، اس طرح آپ کا پروجیکٹ اس طرح منظم ہو سکتا ہے:

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

یہ بہت اچھا ہے، ہماری آرکیٹیکچر کافی صاف ستھری بن سکتا ہے۔

ٹولز کو کال کرنے کا کیا؟ کیا وہی خیال ہے، ایک ہینڈلر جو کسی بھی ٹول کو کال کرے؟ جی ہاں، بالکل، یہاں اس کا کوڈ ہے:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ایک لغت ہے جس میں آلے کے نام بطور کلید ہیں
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
    
    // دلائل: request.params.arguments
    // کرنے کے لئے ٹول کو کال کریں،

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

جیسا کہ آپ اوپر کے کوڈ سے دیکھ سکتے ہیں، ہمیں ٹول کا نام نکالنا پڑتا ہے اور کون سے آرگیومنٹس کے ساتھ کال کرنا ہے اور پھر کال کا عمل کرنا پڑتا ہے۔

## توثیق (Validation) کے ساتھ طریقہ کار کو بہتر بنانا

اب تک آپ نے دیکھا کہ ٹولز، ریسورسز اور پرامپٹس شامل کرنے کے لیے آپ کے تمام رجسٹریشنز کو ہر فیچر کی قسم کے لیے یہ دو ہینڈلرز تبدیل کر سکتے ہیں۔ مزید کیا کرنا ہے؟ ہم کو کچھ قسم کی توثیق شامل کرنی چاہیے تاکہ یہ یقین ہو کہ ٹول صحیح آرگیومنٹس کے ساتھ کال ہو رہا ہے۔ ہر رن ٹائم اس کا اپنا حل رکھتا ہے، مثلا Python پیڈانٹک استعمال کرتا ہے اور TypeScript Zod استعمال کرتا ہے۔ خیال یہ ہے کہ ہم درج ذیل کریں:

- فیچر بنانے کی منطق (ٹول، ریسورس یا پرامپٹ) اس کے مخصوص فولڈر میں منتقل کریں۔
- ایک طریقہ شامل کریں تاکہ آنے والی درخواست کو جیسے کہ کسی ٹول کو کال کرنے کی درخواست کو درست طور پر جانچا جا سکے۔

### فیچر بنانا

فیچر بنانے کے لیے ہمیں اس فیچر کی فائل بنانی ہوگی اور اس میں ضروری فیلڈز کی موجودگی یقینی بنانی ہوگی۔ یہ فیلڈز ٹولز، ریسورسز اور پرامپٹس کے لحاظ سے مختلف ہو سکتے ہیں۔

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
        # Pydantic ماڈل کا استعمال کرتے ہوئے ان پٹ کی تصدیق کریں
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic شامل کریں، تاکہ ہم AddInputModel بنا سکیں اور دلائل کی تصدیق کر سکیں۔

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

یہاں آپ دیکھ سکتے ہیں کہ ہم درج ذیل کرتے ہیں:

- پیڈانٹک کے `AddInputModel` کے ساتھ اسکیمہ بنائیں جس میں فیلڈز `a` اور `b` ہیں، فائل *schema.py* میں۔
- آنے والی درخواست کو `AddInputModel` کی قسم میں تبدیل کرنے کی کوشش کریں، اگر پیرامیٹرز میں تضاد ہو تو یہ کرش کر جائے گا:

   ```python
   # add.py
    try:
        # پائیڈینٹک ماڈل کا استعمال کرتے ہوئے ان پٹ کی توثیق کریں
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

آپ فیصلہ کر سکتے ہیں کہ یہ پارسنگ لاجک ٹول کال کے اندر ہو یا ہینڈلر فنکشن میں۔

**TypeScript**

```typescript
// سرور.ts
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

       // @ts-نظر انداز کریں
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

// سکیمہ.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// شامل کریں.ts
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

- تمام ٹول کالز کے ہینڈلر میں، ہم اب کوشش کرتے ہیں کہ آنے والی درخواست کو ٹول کے متعین کردہ اسکیمہ میں پارس کریں:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    اگر یہ کامیاب ہو جائے تو ہم اصل ٹول کو کال کرتے ہیں:

    ```typescript
    const result = await tool.callback(input);
    ```

جیسا کہ آپ دیکھ سکتے ہیں، یہ طریقہ ایک عمدہ آرکیٹیکچر تخلیق کرتا ہے کیونکہ ہر چیز کی اپنی جگہ ہے، *server.ts* ایک بہت چھوٹی فائل ہے جو صرف درخواست ہینڈلرز کو جوڑتی ہے اور ہر فیچر اپنے متعلقہ فولڈر میں ہے مثلاً tools/، resources/ یا prompts/۔

بہت اچھا، آئیے اگلا حصہ بنائیں۔

## مشق: لو لیول سرور بنانا

اس مشق میں، ہم یہ کریں گے:

1. لو لیول سرور بنائیں جو ٹولز کی فہرست دکھانے اور ٹولز کو کال کرنے کا انتظام کرے۔
1. ایک ایسی آرکیٹیکچر نافذ کریں جس پر آپ مزید تعمیر کر سکیں۔
1. توثیق شامل کریں تاکہ آپ کے ٹول کالز درست طریقے سے جانچے جائیں۔

### -1- آرکیٹیکچر بنانا

سب سے پہلے ہمیں ایک ایسی آرکیٹیکچر چاہیے جو ہمیں اس وقت سکیل کرنے میں مدد دے جب ہم مزید فیچرز شامل کریں، یہ کچھ اس طرح نظر آتی ہے:

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

اب ہم نے ایک ایسی آرکیٹیکچر بنائی ہے جو یہ یقینی بناتی ہے کہ ہم آسانی سے tools فولڈر میں نئے ٹولز شامل کر سکیں۔ آپ ریسورسز اور پرامپٹس کے لیے بھی سب ڈائریکٹریز بنا سکتے ہیں۔

### -2- ایک ٹول بنانا

آئیے دیکھتے ہیں کہ ٹول بنانے کا طریقہ کیا ہوتا ہے۔ سب سے پہلے، اسے اپنے *tool* ذیلی فولڈر میں بنائیں گے، اس طرح:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # انپٹ کی تصدیق کے لیے Pydantic ماڈل استعمال کریں
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic شامل کریں، تاکہ ہم AddInputModel بنا سکیں اور args کی تصدیق کر سکیں

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

یہاں ہم دیکھتے ہیں کہ ہم نام، وضاحت، پیڈانٹک استعمال کرتے ہوئے ان پٹ اسکیمہ اور ہینڈلر کو تعریف کرتے ہیں جو اس ٹول کے کال ہونے پر چلایا جائے گا۔ آخر میں، ہم `tool_add` کو ظاہر کرتے ہیں جو ایک ڈکشنری ہے جس میں یہ خصوصیات موجود ہیں۔

*schema.py* بھی ہے جو ان پٹ اسکیمہ کی تعریف کرنے کے لیے استعمال ہوتا ہے:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

ہمیں *__init__.py* بھی بھرنا ہوگا تاکہ tools ڈائریکٹری کو ماڈیول سمجھا جائے۔ مزید برآں، ہمیں اس میں موجود ماڈیولز کو اس طرح ظاہر کرنا ہو گا:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

ہم اس فائل میں مزید ٹولز شامل کرتے رہ سکتے ہیں۔

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

یہاں ہم ایک ڈکشنری بناتے ہیں جس میں خصوصیات شامل ہیں:

- name، یہ ٹول کا نام ہے۔
- rawSchema، یہ Zod اسکیمہ ہے، یہ ٹول کو کال کرنے والی آنے والی درخواستوں کی توثیق کے لیے استعمال ہوگا۔
- inputSchema، یہ اسکیمہ ہینڈلر کے لیے استعمال ہوتا ہے۔
- callback، یہ ٹول کو کال کرنے کے لیے استعمال ہوتا ہے۔

ایک `Tool` بھی ہے جو اس ڈکشنری کو ایک ایسی قسم میں تبدیل کرتا ہے جسے mcp سرور ہینڈلر قبول کر سکتا ہے، اور یہ اس طرح نظر آتا ہے:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

اور *schema.ts* ہے جہاں ہم ہر ٹول کے ان پٹ اسکیمے رکھتے ہیں، موجودہ طور پر صرف ایک اسکیمہ ہے لیکن جیسے جیسے ہم مزید ٹولز شامل کریں گے ہم مزید اندراجات کرسکتے ہیں:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

بہت اچھا، اب ہم ٹولز کی فہرست سنبھالنے کو آگے بڑھاتے ہیں۔

### -3- ٹولز کی فہرست سنبھالنا

اب، ٹولز کی فہرست سنبھالنے کے لیے ہمیں درخواست ہینڈلر قائم کرنا ہوگا۔ یہ وہ چیز ہے جو ہمیں اپنے سرور کی فائل میں شامل کرنی ہوگی:

**Python**

```python
# مختصر کرنے کے لیے کوڈ کو چھوڑ دیا گیا ہے
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

یہاں ہم `@server.list_tools` ڈیکوریٹر اور `handle_list_tools` فنکشن شامل کرتے ہیں۔ اس فنکشن میں ہمیں ٹولز کی فہرست تیار کرنی ہے۔ دھیان دیں کہ ہر ٹول میں نام، وضاحت اور inputSchema ہونا چاہیے۔   

**TypeScript**

ٹولز کی فہرست دینے کے لیے درخواست ہینڈلر قائم کرنے کیلئے ہمیں سرور پر `setRequestHandler` کال کرنا ہوگا اور اسکیمہ دی جائے جو ہم کرنا چاہتے ہیں، اس صورت میں `ListToolsRequestSchema`۔

```typescript
// انڈیکس.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// سرور.ts
// کوڈ کو اختصار کے لیے حذف کیا گیا
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // رجسٹرڈ ٹولز کی فہرست واپس کریں
  return {
    tools: tools
  };
});
```

زبردست، اب ہم نے ٹولز کی فہرست کا مسئلہ حل کر لیا، اگلا دیکھتے ہیں کہ ٹولز کو کال کیسے کیا جا سکتا ہے۔

### -4- ٹول کال کرنا سنبھالنا

ٹول کال کرنے کے لیے، ہمیں ایک اور درخواست ہینڈلر قائم کرنا ہوگا، جو فیچر کا نام اور آرگیومنٹس کو لے کر درخواست کو سنبھالے۔

**Python**

`@server.call_tool` ڈیکوریٹر استعمال کریں اور اسے `handle_call_tool` فنکشن سے نافذ کریں۔ اس فنکشن کے اندر، ہمیں ٹول کا نام، اس کے آرگیومنٹس نکالنا ہوں گے اور یقینی بنانا ہوگا کہ آرگیومنٹس درست ہیں۔ آپ آرگیومنٹس کی توثیق اس فنکشن میں یا اصل ٹول میں کر سکتے ہیں۔

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ایک لغت ہے جس میں ٹول کے نام بطور چابیاں ہوتے ہیں
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # آلے کو چلائیں
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

یہاں کیا ہوتا ہے:

- ہمارا ٹول نام پہلے سے ہی ان پٹ پیرامیٹر `name` کے طور پر موجود ہے، جبکہ آرگیومنٹس `arguments` ڈکشنری کی صورت میں ہیں۔

- ٹول کو `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` کے ساتھ کال کیا جاتا ہے۔ آرگیومنٹس کی توثیق `handler` پراپرٹی میں ہوتی ہے جو ایک فنکشن کی طرف اشارہ کرتی ہے، اگر وہ ناکام ہو جائے تو استثنا اٹھائے گا۔ 

تو، اب ہمیں لو لیول سرور کے ذریعے ٹولز کی فہرست دینے اور کال کرنے کا مکمل علم ہو گیا ہے۔

پورا [نمونہ](./code/README.md) یہاں دیکھیں

## اسائنمنٹ

آپ کو دیا گیا کوڈ مزید ٹولز، ریسورسز اور پرامپٹس کے ساتھ بڑھائیں اور غور کریں کہ آپ کو صرف tools ڈائریکٹری میں فائلیں شامل کرنی پڑتی ہیں اور کہیں اور نہیں۔

*کوئی حل فراہم نہیں کیا گیا*

## خلاصہ

اس باب میں، ہم نے دیکھا کہ لو لیول سرور کا طریقہ کار کیسے کام کرتا ہے اور یہ کیسے ایک صاف ستھری آرکیٹیکچر بنانے میں مدد دیتا ہے جس پر آپ مزید تعمیر کر سکتے ہیں۔ ہم نے توثیق پر بھی بات کی اور آپ کو دکھایا گیا کہ ان پٹ کی توثیق کے لیے اسکیمے کیسے بنائے جاتے ہیں۔

## آگے کیا ہے

- اگلا: [سادہ توثیق](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**اعلانِ دستبرداری**:  
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ اگرچہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات کا ادراک رکھیں کہ خودکار تراجم میں غلطیاں یا نقائص ہوسکتے ہیں۔ اصل دستاویز اس کی ملکی زبان میں ہی مستند ماخذ سمجھی جانی چاہیے۔ اہم معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ ہم اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کے ذمہ دار نہیں ہیں۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
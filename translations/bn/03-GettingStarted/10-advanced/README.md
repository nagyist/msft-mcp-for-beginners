# উন্নত সার্ভার ব্যবহার

MCP SDK তে দুটি ধরণের সার্ভার রয়েছে, আপনার সাধারণ সার্ভার এবং লো-লেভেল সার্ভার। সাধারণত, আপনি ফিচার যুক্ত করার জন্য সাধারণ সার্ভার ব্যবহার করবেন। তবে কিছু ক্ষেত্রে, আপনাকে লো-লেভেল সার্ভারের উপর নির্ভর করতে হতে পারে যেমন:

- উন্নত আর্কিটেকচার। সাধারণ সার্ভার এবং লো-লেভেল সার্ভার উভয় দিয়ে পরিষ্কার আর্কিটেকচার তৈরি করা সম্ভব, তবে যুক্তি করা যায় যে লো-লেভেল সার্ভার দিয়ে এটি একটু সহজ।
- ফিচার উপলব্ধতা। কিছু উন্নত ফিচার কেবল লো-লেভেল সার্ভার দিয়ে ব্যবহার করা যায়। পরবর্তী অধ্যায়ে আপনি এটি দেখতে পাবেন যেমন স্যাম্পলিং এবং এলিসিটেশন যুক্ত করার সময়।

## সাধারণ সার্ভার বনাম লো-লেভেল সার্ভার

সাধারণ সার্ভার দিয়ে একটি MCP সার্ভার তৈরি করার উদাহরণ হলো:

**Python**

```python
mcp = FastMCP("Demo")

# একটি যোগ করার টুল যোগ করুন
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

// একটি যোগ করার টুল যোগ করুন
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

বিষয়টি হলো আপনি স্পষ্টভাবে প্রতিটি টুল, রিসোর্স বা প্রম্পট যোগ করেন যা সার্ভার এ থাকবে। এতে কোন সমস্যা নেই।  

### লো-লেভেল সার্ভার পদ্ধতি

তবে, যখন আপনি লো-লেভেল সার্ভার পদ্ধতি ব্যবহার করেন, তখন আপনাকে ভিন্ন ভাবে চিন্তা করতে হয়, অর্থাৎ প্রতিটি টুল নিবন্ধন করার বদলে প্রতিটি ফিচার টাইপের (টুল, রিসোর্স বা প্রম্পট) জন্য দুইটি হ্যান্ডলার তৈরি করবেন। উদাহরণস্বরূপ টুলগুলোর ক্ষেত্রে কেবল দুটি ফাংশন থাকবে, উদাহরণস্বরূপ:

- সব টুলের তালিকা দেওয়া। একটি ফাংশন সমস্ত টুল তালিকা দেওয়ার চেষ্টা পরিচালনা করবে।
- সমস্ত টুল কল পরিচালনা করা। এখানে কেবল একটি ফাংশন টুলে কল পরিচালনা করবে।

এটি কম কাজের মত শোনাচ্ছে, তাই না? তাই টুল নিবন্ধন করার বদলে, আমি কেবল নিশ্চিত করব যখন আমি সব টুলের তালিকা দিচ্ছি তখন টুলটি তালিকাভুক্ত আছে এবং যখন একটা টুল কল করার অনুরোধ আসবে তখন তাকে কল করা হয়। 

চলুন দেখি এখন কোড কেমন দেখায়:

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
  // নিবন্ধিত টুলগুলির তালিকা প্রদান করুন
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

এখানে একটি ফাংশন আছে যা ফিচারগুলোর তালিকা প্রদান করে। টুল তালিকার প্রতিটি এন্ট্রিতে এখন `name`, `description` এবং `inputSchema` মত ফিল্ড থাকে যাতে রিটার্ন টাইপের সাথে মিলে। এর ফলে আমরা আমাদের টুল এবং ফিচার ডেফিনিশন অন্য কোথাও রাখতে পারি। এখন আমরা আমাদের সব টুল tools ফোল্ডারে রাখতে পারি, আর একইভাবে সব ফিচারও রাখতে পারি, ফলে আপনার প্রকল্প হঠাৎ করে এমনভাবে সংগঠিত হতে পারে:

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

এটি চমৎকার, আমাদের আর্কিটেকচার খুব পরিষ্কার দেখাতে পারে।

টুল কল করার কথা কী, এটা কি একই ধারণা, একটি হ্যান্ডলার যে কোনো টুল কল করবে? হ্যাঁ, ঠিক তাই, এটি করার জন্য কোড হলো:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools হলো একটি ডিকশনারি যেখানে টুলের নামগুলি কী হিসেবে আছে
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
    
    // আর্গুমেন্ট: request.params.arguments
    // TODO টুলটি কল করুন,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

উপরের কোড থেকে দেখা যায়, আমাদের টুলটি কল করতে এবং কোন আর্গুমেন্ট দিয়ে কল করতে হবে তা পার্স করতে হবে, তারপর টুল কল করতে হবে।

## ভ্যালিডেশন দিয়ে পদ্ধতি উন্নত করা

এখন পর্যন্ত, আপনি দেখেছেন কিভাবে টুল, রিসোর্স এবং প্রম্পট যোগ করার সমস্ত নিবন্ধন এই দুটি হ্যান্ডলার দ্বারা প্রতিস্থাপিত হতে পারে ফিচার টাইপ অনুসারে। আর কি করতে হবে? অবশ্যই, টুল সঠিক আর্গুমেন্টের সাথে কল হচ্ছে কিনা তা নিশ্চিত করার জন্য নির্দিষ্ট ভ্যালিডেশন যোগ করা উচিত। প্রতিটি রানটাইমের জন্য এর নিজস্ব সমাধান আছে, যেমন Python এ Pydantic ব্যবহার হয় এবং TypeScript এ Zod। ধারণাটি হলো:

- একটি ফিচার (টুল, রিসোর্স বা প্রম্পট) তৈরি করার লজিক dedicated ফোল্ডারে স্থানান্তর করা।
- একটি উপায় যোগ করা যাতে ইনকামিং রিকোয়েস্ট যাচাই করা যায়, যেমন একটি টুল কল করার অনুরোধ।

### একটি ফিচার তৈরি করা

একটি ফিচার তৈরি করতে, ঐ ফিচারের জন্য একটি ফাইল তৈরি করতে হবে এবং নিশ্চিত করতে হবে এতে ঐ ফিচারের জন্য প্রয়োজনীয় ফিল্ড গুলো আছে কি না। টুল, রিসোর্স এবং প্রম্পটের মধ্যে কোন ফিল্ড গুলো আলাদা:

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
        # Pydantic মডেল ব্যবহার করে ইনপুট যাচাই করুন
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic যোগ করুন, যাতে আমরা একটি AddInputModel তৈরি করতে এবং আর্গুমেন্ট যাচাই করতে পারি

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

এখানে দেখা যাচ্ছে আমরা নিম্নোক্ত কাজ করি:

- Pydantic `AddInputModel` ব্যবহার করে একটি স্কিমা তৈরি করেছি *schema.py* ফাইলে, যেখানে ফিল্ড `a` এবং `b` রয়েছে।
- ইনকামিং রিকোয়েস্টকে `AddInputModel` টাইপে পার্স করার চেষ্টা করি, যদি প্যারামিটার মেলাতে সমস্যা হয় তাহলে এটি ক্র্যাশ করবে:

   ```python
   # add.py
    try:
        # Pydantic মডেল ব্যবহার করে ইনপুট যাচাই করুন
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

আপনি চাইলে এই পার্সিং লজিক সরাসরি টুল কলেই রাখতে পারেন বা হ্যান্ডলার ফাংশনে রাখতে পারেন।

**TypeScript**

```typescript
// সার্ভার.ts
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

// স্কিমা.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// যোগ করুন.ts
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

- সকল টুল কল পরিচালনার হ্যান্ডলার ফাংশনে আমরা এখন ইনকামিং রিকোয়েস্ট টুলের নির্ধারিত স্কিমায় পার্স করার চেষ্টা করি:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    যদি সফল হয় তবে আসল টুল কল করা হয়:

    ```typescript
    const result = await tool.callback(input);
    ```

দেখতে পাচ্ছেন, এই পদ্ধতিতে খুব সুন্দর আর্কিটেকচার তৈরি হয় কারণ সবকিছু তার ঠিক জায়গায় থাকে, *server.ts* একটি খুব ছোট ফাইল যা শুধুমাত্র রিকোয়েস্ট হ্যান্ডলার যুক্ত করে এবং প্রতিটি ফিচার তাদের নিজস্ব ফোল্ডারে থাকে যেমন tools/, resources/ বা prompts/।

দারুণ, এখন এটি তৈরি করার চেষ্টা করি।

## অনুশীলন: একটি লো-লেভেল সার্ভার তৈরি করা

এই অনুশীলনে আমরা নিম্নোক্ত কাজ করব:

1. টুল তালিকা দেখানো এবং টুল কল করার জন্য একটি লো-লেভেল সার্ভার তৈরি করা।
2. একটি এমন আর্কিটেকচার বাস্তবায়ন করা যা আপনি ভবিষ্যতে বাড়াতে পারবেন।
3. নিশ্চিত করতে ভ্যালিডেশন যোগ করা যে আপনার টুল কল সঠিকভাবে যাচাই হয়।

### -1- একটি আর্কিটেকচার তৈরি করা

প্রথমেই আমাদের এমন একটি আর্কিটেকচার তৈরি করতে হবে যা নতুন ফিচার যোগ করার সময় সহজে স্কেল হতে পারে, এরকম:

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

এখন আমরা এমন একটি আর্কিটেকচার সেটআপ করেছি যা সহজেই tools ফোল্ডারে নতুন টুল যুক্ত করার সুযোগ দেয়। আপনি চাইলে resources এবং prompts এর জন্য সুবডিরেক্টরি যোগ করতে পারেন।

### -2- একটি টুল তৈরি করা

পরবর্তী, একটি টুল তৈরি করার কীভাবে দেখুন। প্রথমত, এটি তার *tool* সুবডিরেক্টরিতে তৈরি করতে হবে, যেমন:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic মডেল ব্যবহার করে ইনপুট যাচাই করুন
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic যোগ করুন, যাতে আমরা একটি AddInputModel তৈরি করতে এবং আর্গুমেন্টগুলি যাচাই করতে পারি

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

এখানে দেখা যাচ্ছে আমরা কিভাবে নাম, বর্ণনা, Pydantic ব্যবহার করে ইনপুট স্কিমা এবং একটি হ্যান্ডলার নিয়ে ডিফাইন করি যেটি যখন টুল কল হবে তখন চালানো হবে। অবশেষে, আমরা `tool_add` প্রকাশ করি যা একটি ডিকশনারি holding এই সব প্রপার্টি।

এছাড়াও *schema.py* থাকে যা আমাদের টুলের ইনপুট স্কিমা ডিফাইন করতে ব্যবহৃত হয়:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

আমাদের *__init__.py* ফাইলেও কিছু প্রবেশ করাতে হবে যাতে tools ডিরেক্টরিকে একটি মডিউল হিসেবে বিবেচনা করা হয়। এছাড়া এর ভিতরের মডিউল গুলো এইভাবে এক্সপোজ করতে হবে:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

যত বেশি টুল যোগ করি তত বেশি এখানে যোগ করতে পারি।

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

এখানে আমরা একটি ডিকশনারি তৈরি করেছি যার প্রপার্টিসমূহ:

- name, এটি টুলের নাম।
- rawSchema, এটি Zod স্কিমা, যা ইনকামিং রিকোয়েস্ট যাচাই করতে ব্যবহৃত হবে।
- inputSchema, এই স্কিমাটি হ্যান্ডলার দ্বারা ব্যবহৃত হবে।
- callback, এটি টুলটি কল করতে ব্যবহৃত হয়।

আরো আছে `Tool` যা এই ডিকশনারিকে MCP সার্ভারের হ্যান্ডলার গ্রহণযোগ্য টাইপে রূপান্তর করে এবং এরকম দেখায়:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

এবং আছে *schema.ts* যেখানে প্রতিটি টুলের ইনপুট স্কিমা রাখা হয়, বর্তমানে কেবল একটি স্কিমা আছে কিন্তু পরবর্তীতে আমরা আরো টুল যুক্ত করলে আরো এন্ট্রি যোগ করব:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

চমৎকার, এখন টুলের তালিকা দেখানোর অংশটি হ্যান্ডল করা যাক।

### -3- টুল তালিকা হ্যান্ডল করা

পরবর্তী, টুলের তালিকা দেখানোর জন্য একটি রিকোয়েস্ট হ্যান্ডলার সেটআপ করতে হবে। সার্ভার ফাইলে এটা এভাবে যোগ করতে হবে:

**Python**

```python
# সংক্ষিপ্ততার জন্য কোড বাদ দেওয়া হয়েছে
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

এখানে `@server.list_tools` ডেকোরেটর এবং `handle_list_tools` ফাংশন যোগ করা হয়েছে। এখানে একটি টুলের তালিকা তৈরি করতে হবে। লক্ষ্য করুন প্রতিটি টুলের নাম, বর্ণনা এবং inputSchema থাকা দরকার।   

**TypeScript**

টুল তালিকা দেখানোর রিকোয়েস্ট হ্যান্ডলার সেটআপ করতে আমরা সার্ভারে `setRequestHandler` কল করি উপযুক্ত স্কিমা যেমন `ListToolsRequestSchema` দিয়ে:

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
// সংক্ষেপের জন্য কোডটি বাদ দেওয়া হয়েছে
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // নিবন্ধিত সরঞ্জামগুলির তালিকা ফেরত দিন
  return {
    tools: tools
  };
});
```

দারুণ, এখন টুল তালিকা দেখানোর কাজ সমাধান হয়েছে, পরবর্তীতে টুল কল করার পদ্ধতি দেখি।

### -4- টুল কল হ্যান্ডল করা

একটি টুল কল করার জন্য আরেকটি রিকোয়েস্ট হ্যান্ডলার সেটআপ করতে হবে, যা নির্দিষ্ট করবে কোন ফিচার কল করতে হবে এবং কোন আর্গুমেন্ট দিয়ে।

**Python**

চলুন `@server.call_tool` ডেকোরেটর ব্যবহার করি এবং `handle_call_tool` ফাংশন ইমপ্লিমেন্ট করি। ফাংশনের ভিতরে আমাদের টুল নাম, আর্গুমেন্ট পার্স করতে হবে এবং নিশ্চিত করতে হবে যে আর্গুমেন্টগুলো টুলের জন্য বৈধ কিনা। আমরা এই ভ্যালিডেশন এই ফাংশনে বা টুলের ভিতরেও করতে পারি।

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools একটি ডিকশনারি যেখানে টুল নামগুলি কী হিসেবে রয়েছে
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # টুলটি আহ্বান করুন
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

এখানে ঘটে:

- আমাদের টুল নাম `name` ইনপুট প্যারামিটার হিসাবে দেওয়া আছে, এবং আর্গুমেন্ট `arguments` ডিকশনারি আকারে।

- টুল কল করা হয় `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` দিয়ে। আর্গুমেন্ট ভ্যালিডেশন হয় `handler` এ যা একটি ফাংশনে নির্দেশ করে, তা যদি ব্যর্থ হয় একটি এক্সেপশন ছুড়ে দেয়।

এতদূর, আমরা একটি লো-লেভেল সার্ভার ব্যবহার করে টুল তালিকা দেখানো এবং কল করা সম্পূর্ণ বুঝে গেছি।

সম্পূর্ণ উদাহরণ দেখুন [এখানে](./code/README.md)

## অ্যাসাইনমেন্ট

আপনার দেওয়া কোডে আরও টুল, রিসোর্স এবং প্রম্পট যুক্ত করুন এবং লক্ষ্য করুন যে কেবল tools ডিরেক্টরিতে ফাইল যোগ করলেই হয় এবং অন্য কোথাও লাগেনা।

*কোন সমাধান দেওয়া হয়নি*

## সারাংশ

এই অধ্যায়ে, আমরা দেখেছি কিভাবে লো-লেভেল সার্ভার পদ্ধতি কাজ করে এবং কিভাবে এটি একটি সুন্দর আর্কিটেকচার তৈরি করতে সাহায্য করে যা আমরা পরবর্তীতে বাড়াতে পারি। আমরা ভ্যালিডেশনকেও আলোচনা করেছি এবং দেখেছি কিভাবে ভ্যালিডেশন লাইব্রেরি ব্যবহার করে ইনপুট যাচাইয়ের জন্য স্কিমা তৈরি করা যায়।

## পরবর্তী

- পরবর্তী: [সহজ অথেনটিকেশন](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**বিজ্ঞপ্তি**:  
এই নথিটি AI অনুবাদ সেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনুবাদ করা হয়েছে। আমরা যথাসম্ভব সঠিকতার চেষ্টা করি, তবে দয়া করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ভুল বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার নিজস্ব ভাষায়ই কর্তৃত্বপূর্ণ উৎস হিসেবে গণ্য হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদের পরামর্শ দেওয়া হয়। এই অনুবাদের ব্যবহার থেকে সৃষ্ট কোনো ভুল বোঝাবুঝি বা অপব্যাখ্যার জন্য আমরা দায়বদ্ধ থাকব না।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
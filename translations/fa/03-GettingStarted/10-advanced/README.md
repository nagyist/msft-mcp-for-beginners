# استفاده پیشرفته از سرور

دو نوع مختلف سرور در MCP SDK وجود دارد، سرور عادی و سرور سطح پایین. معمولا شما از سرور عادی برای اضافه کردن امکانات استفاده می‌کنید. اما در برخی موارد، می‌خواهید به سرور سطح پایین متکی باشید مانند:

- معماری بهتر. ایجاد یک معماری تمیز با هر دو سرور عادی و سرور سطح پایین ممکن است، اما می‌توان بحث کرد که کمی آسان‌تر با سرور سطح پایین است.
- دسترسی به امکانات. برخی ویژگی‌های پیشرفته فقط با سرور سطح پایین قابل استفاده هستند. این موضوع را در فصل‌های بعد خواهید دید، هنگام افزودن نمونه‌برداری و استخراج.

## سرور عادی در مقابل سرور سطح پایین

این تصویری از نحوه ایجاد یک سرور MCP با سرور عادی است

**Python**

```python
mcp = FastMCP("Demo")

# افزودن یک ابزار جمع کردن
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

// افزودن ابزار جمع
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

نکته این است که شما هر ابزار، منبع یا پرامپت مورد نظر برای سرور را به صورت صریح اضافه می‌کنید. هیچ ایرادی در این نیست.

### رویکرد سرور سطح پایین

اما وقتی از رویکرد سرور سطح پایین استفاده می‌کنید باید به صورت متفاوتی فکر کنید، یعنی به جای ثبت هر ابزار، شما دو هندلر برای هر نوع ویژگی (ابزارها، منابع یا پرامپت‌ها) می‌سازید. مثلا ابزارها فقط دو تابع دارند:

- فهرست کردن همه ابزارها. یک تابع مسئول تمام تلاش‌ها برای فهرست کردن ابزارها است.
- هندل کردن تماس با همه ابزارها. همچنین فقط یک تابع تماس با ابزار را مدیریت می‌کند.

این احتمالاً به معنای کار کمتر است، درست است؟ پس به جای ثبت یک ابزار، فقط باید مطمئن شوم که ابزار هنگام فهرست کردن همه ابزارها فهرست شده و هنگام درخواست تماس به ابزار، فراخوانی می‌شود.

بیایید ببینیم کد حالا چگونه است:

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
  // بازگرداندن لیست ابزارهای ثبت‌شده
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

اینجا ما یک تابع داریم که لیستی از ویژگی‌ها را برمی‌گرداند. هر ورودی در فهرست ابزارها اکنون فیلدهایی مانند `name`، `description` و `inputSchema` دارد تا با نوع برگشتی مطابقت داشته باشد. این امکان را می‌دهد تا ابزارها و تعریف ویژگی‌مان را در جای دیگری قرار دهیم. حالا می‌توانیم همه ابزارها را در یک پوشه ابزارها بسازیم و همینطور همه ویژگی‌ها تا پروژه به شکل زیر سازماندهی شود:

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

عالی است، معماری ما می‌تواند خیلی تمیز به نظر برسد.

تماس با ابزارها چطور است، آیا همان ایده است، یک هندلر برای تماس با هر ابزار؟ بله دقیقاً، کد آن چنین است:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # ابزارها یک دیکشنری هستند که نام ابزارها به عنوان کلیدها استفاده شده‌اند
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
    
    // آرگومان‌ها: request.params.arguments
    // TODO ابزار را فراخوانی کنید،

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

همانطور که در کد بالا می‌بینید، باید ابزاری که باید صدا زده شود را تشخیص دهیم، همراه با آرگومان‌ها، سپس ابزار را فراخوانی کنیم.

## بهبود رویکرد با اعتبارسنجی

تا اینجا دیدید چگونه همه ثبت‌ها برای افزودن ابزارها، منابع و پرامپت‌ها را می‌توان با این دو هندلر برای هر نوع ویژگی جایگزین کرد. حالا چه کار دیگری باید انجام دهیم؟ باید یک نوع اعتبارسنجی اضافه کنیم تا اطمینان یابیم ابزار با آرگومان‌های درست فراخوانی می‌شود. هر runtime راهکار خودش را دارد، برای مثال Python از Pydantic و TypeScript از Zod استفاده می‌کند. ایده این است که ما موارد زیر را انجام دهیم:

- منطق ایجاد یک ویژگی (ابزار، منبع یا پرامپت) را به پوشه اختصاصی آن منتقل کنیم.
- روشی اضافه کنیم تا درخواست‌های ورودی که مثلا می‌خواهند ابزاری را فراخوانی کنند اعتبارسنجی شوند.

### ایجاد یک ویژگی

برای ایجاد یک ویژگی، باید یک فایل برای آن ویژگی بسازیم و اطمینان پیدا کنیم که فیلدهای اجباری آن ویژگی وجود دارد. فیلدها کمی بین ابزارها، منابع و پرامپت‌ها متفاوتند.

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
        # ورودی را با استفاده از مدل Pydantic اعتبارسنجی کنید
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: اضافه کردن Pydantic تا بتوانیم یک AddInputModel بسازیم و آرگومان‌ها را اعتبارسنجی کنیم

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

در اینجا می‌بینید که ما چگونه موارد زیر را انجام می‌دهیم:

- ساخت یک اسکما با استفاده از Pydantic به نام `AddInputModel` با فیلدهای `a` و `b` در فایل *schema.py*.
- تلاش برای تبدیل درخواست ورودی به نوع `AddInputModel`، اگر پارامترها همخوانی نداشته باشند، این باعث خطا می‌شود:

   ```python
   # add.py
    try:
        # اعتبارسنجی ورودی با استفاده از مدل پایدانتیک
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

شما می‌توانید تصمیم بگیرید که این منطق تبدیل را در خود فراخوانی ابزار قرار دهید یا در تابع هندلر.

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

// طرح‌واره.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// اضافه کن.ts
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

- در هندلری که با همه تماس‌های ابزارها سروکار دارد، حالا تلاش می‌کنیم درخواست ورودی را به اسکمای تعریف‌شده برای ابزار تبدیل کنیم:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    اگر این کار موفقیت‌آمیز بود، به فراخوانی ابزار ادامه می‌دهیم:

    ```typescript
    const result = await tool.callback(input);
    ```

همانطور که می‌بینید، این رویکرد معماری عالی ایجاد می‌کند چون هر چیز جای خودش را دارد. *server.ts* یک فایل بسیار کوچک است که فقط هندلرهای درخواست را وصل می‌کند و هر ویژگی در پوشه اختصاصی خودش مانند tools/، resources/ یا /prompts قرار دارد.

عالی است، بیایید این را بسازیم.

## تمرین: ایجاد یک سرور سطح پایین

در این تمرین، کارهای زیر را انجام می‌دهیم:

1. ایجاد یک سرور سطح پایین که فهرست ابزارها و تماس با ابزارها را هندل کند.
1. پیاده‌سازی معماری که بتوانید روی آن بسازید.
1. اضافه کردن اعتبارسنجی برای اطمینان از اینکه تماس‌های ابزار شما به درستی اعتبارسنجی می‌شوند.

### -1- ایجاد معماری

اولین چیزی که باید به آن بپردازیم معماری است که به ما کمک می‌کند هنگام افزودن امکانات بیشتر، مقیاس‌پذیری داشته باشیم. بدین شکل است:

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

اکنون معماری‌ای راه‌اندازی کرده‌ایم که اطمینان حاصل می‌کند می‌توانیم ابزارهای جدید را به راحتی در پوشه ابزارها اضافه کنیم. احساس راحتی کنید که برای منابع و پرامپت‌ها هم زیرپوشه بسازید.

### -2- ایجاد یک ابزار

بیایید ببینیم ایجاد یک ابزار چگونه است. اول، باید در زیرپوشه *tool* ساخته شود به شکل زیر:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # اعتبارسنجی ورودی با استفاده از مدل پایدنتیک
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # کارهای انجام‌نشده: افزودن پایدنتیک، تا بتوانیم یک AddInputModel ایجاد کنیم و آرگومان‌ها را اعتبارسنجی کنیم

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

اینجا می‌بینیم که چگونه نام، توضیح، یک اسکمای ورودی با Pydantic و یک هندلری که هنگام فراخوانی این ابزار اجرا می‌شود تعریف شده است. در نهایت، `tool_add` که یک دیکشنری شامل تمام این ویژگی‌هاست، ارائه می‌شود.

همچنین *schema.py* وجود دارد که برای تعریف اسکمای ورودی ابزار استفاده می‌شود:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

همچنین باید *__init__.py* را پر کنیم تا اطمینان حاصل شود که دایرکتوری ابزارها به عنوان یک ماژول در نظر گرفته می‌شود. علاوه بر این باید ماژول‌های درون آن را مثل زیر صادر کنیم:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

می‌توانیم هنگام افزودن ابزارهای بیشتر این فایل را ادامه دهیم.

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

اینجا یک دیکشنری با ویژگی‌های زیر ساخته‌ایم:

- name، نام ابزار است.
- rawSchema، اسکمای Zod است که برای اعتبارسنجی درخواست‌های ورودی استفاده می‌شود.
- inputSchema، این اسکما توسط هندلر استفاده می‌شود.
- callback، برای صدا زدن ابزار استفاده می‌شود.

همچنین `Tool` وجود دارد که این دیکشنری را به تایپی تبدیل می‌کند که هندلر سرور mcp می‌تواند بپذیرد و به این صورت است:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

و *schema.ts* جایی است که اسکماهای ورودی برای هر ابزار ذخیره شده‌اند، که الان فقط یک اسکما دارد ولی با افزودن ابزارهای جدید می‌توان موارد بیشتری اضافه کرد:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

عالی است، حالا برویم سراغ هندل کردن فهرست ابزارها.

### -3- هندل کردن فهرست ابزارها

برای هندل کردن فهرست ابزارها، باید یک هندلر درخواست تنظیم کنیم. این چیزی است که باید به فایل سرور اضافه کنیم:

**Python**

```python
# کد برای اختصار حذف شد
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

در اینجا دکوراتور `@server.list_tools` و تابع پیاده‌سازی‌شده `handle_list_tools` را اضافه می‌کنیم. در دومی باید یک لیست از ابزارها تولید کنیم. توجه کنید که هر ابزار باید نام، توضیح و inputSchema داشته باشد.

**TypeScript**

برای راه‌اندازی هندلر درخواست برای فهرست ابزارها، باید `setRequestHandler` را روی سرور با اسکمای مناسب صدا بزنیم، در این حالت `ListToolsRequestSchema`.

```typescript
// ایندکس.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// سرور.ts
// کد برای اختصار حذف شد
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // بازگرداندن فهرست ابزارهای ثبت شده
  return {
    tools: tools
  };
});
```

عالی، حالا که بخش فهرست ابزارها حل شد، بیایید ببینیم چطور می‌توانیم ابزارها را فراخوانی کنیم.

### -4- هندل کردن فراخوانی ابزار

برای فراخوانی ابزار، باید هندلر درخواست دیگری تنظیم کنیم، این بار تمرکز روی درخواست‌هایی که مشخص می‌کنند کدام ویژگی فراخوانی شود و با چه آرگومان‌هایی.

**Python**

از دکوراتور `@server.call_tool` استفاده می‌کنیم و آن را با تابعی مثل `handle_call_tool` پیاده‌سازی می‌کنیم. در این تابع باید نام ابزار، آرگومان‌های آن را جدا کنیم و اطمینان حاصل کنیم آرگومان‌ها برای ابزار مورد نظر معتبر هستند. می‌توانیم اعتبارسنجی آرگومان‌ها را در این تابع یا در خود ابزار انجام دهیم.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # ابزارها یک دیکشنری با نام‌های ابزار به‌عنوان کلیدها هستند
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # فراخوانی ابزار
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

کارهایی که انجام می‌شود:

- نام ابزار به صورت ورودی `name` حاضر است که برای آرگومان‌ها در قالب دیکشنری `arguments` است.

- ابزار با `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` فراخوانی می‌شود. اعتبارسنجی آرگومان‌ها در ویژگی `handler` که به یک تابع اشاره می‌کند، انجام می‌شود و اگر ناموفق باشد استثناء صادر می‌شود.

اینجا، اکنون فهم کامل از فهرست کردن و فراخوانی ابزارها با استفاده از سرور سطح پایین داریم.

نسخه [کامل نمونه](./code/README.md) را اینجا ببینید

## تکلیف

کدی که داده شده را با تعدادی ابزار، منابع و پرامپت گسترش دهید و در نظر بگیرید چگونه فقط با اضافه کردن فایل در دایرکتوری ابزارها نیاز دارید و جای دیگری لازم نیست.

*هیچ راه حلی ارائه نشده*

## خلاصه

در این فصل دیدیم چگونه رویکرد سرور سطح پایین کار می‌کند و چگونه می‌تواند به ما کمک کند معماری تمیزی بسازیم که بتوانیم روی آن بسازیم. همچنین درباره اعتبارسنجی صحبت کردیم و نحوه کار با کتابخانه‌های اعتبارسنجی برای ایجاد اسکماهای ورودی را دیدید.

## مرحله بعد

- مرحله بعد: [احراز هویت ساده](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان بومی خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، توصیه به استفاده از ترجمه حرفه‌ای انسانی می‌شود. ما در قبال هرگونه سوءتفاهم یا تفسیر نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
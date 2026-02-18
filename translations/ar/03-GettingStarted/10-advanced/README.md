# الاستخدام المتقدم للخادم

هناك نوعان مختلفان من الخوادم المعروضة في MCP SDK، خادمك العادي وخادم منخفض المستوى. عادة، ستستخدم الخادم العادي لإضافة ميزات إليه. ولكن في بعض الحالات، ترغب في الاعتماد على الخادم منخفض المستوى مثل:

- بنية أفضل. من الممكن إنشاء بنية نظيفة باستخدام كل من الخادم العادي وخادم منخفض المستوى، ولكن يمكن القول إنها أسهل قليلاً مع خادم منخفض المستوى.
- توفر الميزات. يمكن استخدام بعض الميزات المتقدمة فقط مع الخادم منخفض المستوى. سترى هذا في الفصول القادمة حيث نضيف التمثيل والاقتراح.

## الخادم العادي مقابل الخادم منخفض المستوى

هذا ما يبدو عليه إنشاء خادم MCP باستخدام الخادم العادي

**Python**

```python
mcp = FastMCP("Demo")

# إضافة أداة جمع
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

// أضف أداة جمع
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

النقطة هي أنك تضيف صراحةً كل أداة، مورد أو مطالبة تريد أن يكون لدى الخادم. لا يوجد خطأ في ذلك.

### نهج الخادم منخفض المستوى

ومع ذلك، عند استخدام نهج الخادم منخفض المستوى، تحتاج إلى التفكير بطريقة مختلفة، وهي بدلاً من تسجيل كل أداة، تنشئ معالجين لكل نوع ميزة (أدوات، موارد أو مطالبات). فمثلاً بالنسبة للأدوات، هناك وظيفتان فقط كما يلي:

- سرد كل الأدوات. وظيفة واحدة ستكون مسؤولة عن كل المحاولات لسرد الأدوات.
- معالجة استدعاء كل الأدوات. هنا أيضاً، هناك وظيفة واحدة فقط تتعامل مع استدعاءات الأداة.

يبدو هذا أقل عملاً على الأرجح، أليس كذلك؟ لذا بدلاً من تسجيل أداة، فقط أحتاج إلى التأكد من أن الأداة مُدرجة عند سرد كل الأدوات وأنه يتم استدعاؤها عند وجود طلب وارد لاستدعاء أداة.

لنتطلع كيف يبدو الكود الآن:

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
  // أرجع قائمة الأدوات المسجلة
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

لدينا الآن وظيفة تُرجع قائمة بالميزات. كل إدخال في قائمة الأدوات يحتوي الآن على حقول مثل `name` و`description` و`inputSchema` ليتوافق مع نوع الإرجاع. هذا يمكّننا من وضع أدواتنا وتعريف الميزة في مكان آخر. يمكننا الآن إنشاء كل أدواتنا في مجلد tools ونفس الشيء للميزات الأخرى بحيث يمكن لمشروعك أن يُنظم فجأة هكذا:

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

هذا عظيم، يمكن جعل بنية التطبيق تبدو نظيفة جداً.

ماذا عن استدعاء الأدوات، هل هي نفس الفكرة إذًا، معالج واحد لاستدعاء أي أداة؟ نعم، بالضبط، هذا هو الكود لذلك:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # أدوات هو قاموس يحتوي على أسماء الأدوات كمفاتيح
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
    
    // الوسيطات: request.params.arguments
    // TODO استدعاء الأداة،

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

كما ترى من الكود أعلاه، نحتاج إلى تحليل الأداة المطلوب استدعاؤها، وبما هي الوسائط، ثم نتابع استدعاء الأداة.

## تحسين النهج بالتحقق من الصحة

حتى الآن، رأيت كيف يمكن استبدال جميع تسجيلاتك لإضافة الأدوات، الموارد والمطالبات بهذين المعالجين لكل نوع ميزة. ماذا نحتاج أن نفعل أيضا؟ يجب أن نضيف شكلاً من أشكال التحقق من الصحة لضمان استدعاء الأداة بالوسائط الصحيحة. كل بيئة تشغيل لها حلها الخاص، مثلاً يستخدم بايثون Pydantic وTypeScript يستخدم Zod. الفكرة هي أننا سنفعل التالي:

- نقل منطق إنشاء ميزة (أداة، مورد أو مطالبة) إلى مجلدها المخصص.
- إضافة طريقة للتحقق من صحة طلب وارد يطلب مثلاً استدعاء أداة.

### إنشاء ميزة

لإنشاء ميزة، سنحتاج إلى إنشاء ملف لتلك الميزة والتأكد من أنه يحتوي على الحقول الإلزامية المطلوبة لتلك الميزة. تختلف الحقول قليلاً بين الأدوات، الموارد والمطالبات.

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
        # تحقق من صحة الإدخال باستخدام نموذج Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: أضف Pydantic، حتى نتمكن من إنشاء AddInputModel والتحقق من الوسائط

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

هنا ترى كيف نفعل التالي:

- إنشاء مخطط باستخدام Pydantic `AddInputModel` بالحقول `a` و `b` في ملف *schema.py*.
- محاولة تحليل الطلب الوارد ليكون من نوع `AddInputModel`، إذا كان هناك عدم تطابق في المعلمات فهذا سيؤدي إلى تعطل:

   ```python
   # add.py
    try:
        # التحقق من صحة الإدخال باستخدام نموذج Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

يمكنك اختيار وضع منطق التحليل هذا في استدعاء الأداة نفسه أو في وظيفة المعالج.

**TypeScript**

```typescript
// سيرفر.ts
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

       // @ts-تجاهل
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

// مخطط.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// إضافة.ts
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

- في المعالج الذي يتعامل مع كل استدعاءات الأدوات، نحاول الآن تحليل الطلب الوارد إلى المخطط المعرفة للأداة:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    إذا نجح ذلك نتابع استدعاء الأداة الفعلية:

    ```typescript
    const result = await tool.callback(input);
    ```

كما ترى، هذا النهج يخلق بنية رائعة حيث لكل شيء مكانه، ملف *server.ts* هو ملف صغير جداً يربط فقط معالجات الطلبات ولكل ميزة مجلدها الخاص مثل tools/ ، resources/ أو /prompts.

عظيم، دعنا نحاول بناء ذلك بعد ذلك.

## تمرين: إنشاء خادم منخفض المستوى

في هذا التمرين، سنفعل التالي:

1. إنشاء خادم منخفض المستوى يتعامل مع سرد الأدوات واستدعاء الأدوات.
1. تنفيذ بنية يمكنك البناء عليها.
1. إضافة تحقق من صحة لضمان التحقق بشكل صحيح من استدعاءات الأداة.

### -1- إنشاء بنية

أول شيء نحتاج لمعالجته هو بنية تساعدنا على التوسع كلما أضفنا المزيد من الميزات، هذا ما تبدو عليه:

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

الآن أعددنا بنية تضمن أننا يمكننا بسهولة إضافة أدوات جديدة في مجلد tools. لا تتردد في اتباع ذلك لإضافة مجلدات فرعية للموارد والمطالبات.

### -2- إنشاء أداة

لنرَ كيف يبدو إنشاء أداة بعد ذلك. أولاً، يجب إنشاؤها في دليل فرعي *tool* هكذا:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # التحقق من صحة الإدخال باستخدام نموذج Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: إضافة Pydantic، حتى نتمكن من إنشاء AddInputModel والتحقق من صحة الوسائط

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ما نراه هنا هو كيف نعرف الاسم، الوصف، مخطط الإدخال باستخدام Pydantic ومعالج سيتم استدعاؤه بمجرد استدعاء هذه الأداة. وأخيرًا، نعرض `tool_add` وهي قاموس يحتوي على كل هذه الخصائص.

هناك أيضًا *schema.py* يُستخدم لتعريف مخطط الإدخال المستخدم بأداتنا:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

نحتاج أيضاً إلى تعبئة *__init__.py* لضمان اعتبار مجلد الأدوات كوحدة. بالإضافة إلى ذلك، نحتاج إلى عرض الوحدات الموجودة بداخله هكذا:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

يمكننا الاستمرار في الإضافة إلى هذا الملف مع زيادة الأدوات.

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

هنا ننشئ قاموس يتألف من الخصائص:

- name، هذا هو اسم الأداة.
- rawSchema، هذا هو مخطط Zod، سيتم استخدامه للتحقق من صحة الطلبات الواردة لاستدعاء هذه الأداة.
- inputSchema، سيتم استخدام هذا المخطط بواسطة المعالج.
- callback، يُستخدم لاستدعاء الأداة.

هناك أيضاً `Tool` التي تُستخدم لتحويل هذا القاموس إلى نوع يمكن لمعالج خادم mcp قبوله ويبدو هكذا:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

وهناك *schema.ts* حيث نخزن مخططات الإدخال لكل أداة والتي تبدو هكذا مع وجود مخطط واحد فقط حالياً، ولكن مع إضافة أدوات يمكن إضافة المزيد من الإدخالات:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

عظيم، دعنا نتابع لمعالجة سرد أدواتنا التالي.

### -3- معالجة سرد الأدوات

بعد ذلك، لمعالجة سرد أدواتنا، نحتاج إلى إعداد معالج طلب لذلك. هذا ما نحتاج إضافته إلى ملف الخادم:

**Python**

```python
# تم حذف الكود للاختصار
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

هنا نضيف الزخرفة `@server.list_tools` والوظيفة المنفذة `handle_list_tools`. في الأخيرة، نحتاج إلى إنتاج قائمة بالأدوات. لاحظ كيف يجب أن يحتوي كل أداة على اسم، وصف وinputSchema.

**TypeScript**

لإعداد معالج الطلبات لسرد الأدوات، نحتاج إلى استدعاء `setRequestHandler` على الخادم مع مخطط يتناسب مع ما نحاول فعله، في هذه الحالة `ListToolsRequestSchema`.

```typescript
// ملف الفهرس.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// ملف الخادم.ts
// تم حذف الكود للاختصار
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // إعادة قائمة الأدوات المسجلة
  return {
    tools: tools
  };
});
```

عظيم، الآن حللنا جزء سرد الأدوات، لننظر كيف يمكننا استدعاء الأدوات بعد ذلك.

### -4- معالجة استدعاء أداة

لاستدعاء أداة، نحتاج إلى إعداد معالج طلب آخر، هذه المرة يركز على التعامل مع طلب يحدد أي ميزة لاستدعاؤها وبما هي الوسائط.

**Python**

لنستخدم الزخرفة `@server.call_tool` وننفذها بوظيفية مثل `handle_call_tool`. داخل تلك الوظيفة، نحتاج إلى تحليل اسم الأداة، وسيطها والتأكد من صحة الوسائط الخاصة بالأداة المعنية. يمكننا إما التحقق من صحة الوسائط في هذه الوظيفة أو في الأداة الفعلية لاحقًا.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # الأدوات هي قاموس بأسماء الأدوات كمفاتيح
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # استدعاء الأداة
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

هذا ما يحدث:

- اسم أداتنا موجود بالفعل كمعامل إدخال `name` وهذا ينطبق على وسائطنا في شكل قاموس `arguments`.

- تم استدعاء الأداة بـ `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. تحقق صحة الوسائط يتم في خاصية `handler` التي تشير إلى دالة، إذا فشل ذلك سترتفع استثناء.

ها نحن ذا، الآن لدينا فهم كامل لسرد واستدعاء الأدوات باستخدام خادم منخفض المستوى.

شاهد [المثال الكامل](./code/README.md) هنا

## المهمة

قم بتوسيع الكود الذي حصلت عليه بعدد من الأدوات، الموارد والمطالبات وانعكس كيف تلاحظ أنك تحتاج فقط إلى إضافة ملفات في دليل الأدوات وليس في مكان آخر.

*لا يوجد حل مقدم*

## الملخص

في هذا الفصل، رأينا كيف يعمل نهج الخادم منخفض المستوى وكيف يمكن أن يساعدنا في إنشاء بنية جميلة يمكننا مواصلة البناء عليها. ناقشنا أيضاً التحقق من الصحة وعرضنا لك كيف تعمل مع مكتبات التحقق لإنشاء مخططات للتحقق من صحة الإدخال.

## ما هو التالي

- التالي: [مصادقة بسيطة](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**إخلاء المسؤولية**:  
تمت ترجمة هذا المستند باستخدام خدمة الترجمة الآلية [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للحرص على الدقة، يُرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية هو المصدر الرسمي والمعتمد. بالنسبة للمعلومات الحرجة، يُنصح بالاعتماد على الترجمة المهنية البشرية. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# การใช้งานเซิร์ฟเวอร์ขั้นสูง

มีเซิร์ฟเวอร์สองประเภทที่เปิดให้ใช้งานใน MCP SDK คือ เซิร์ฟเวอร์ปกติและเซิร์ฟเวอร์ระดับต่ำ โดยปกติแล้วคุณจะใช้เซิร์ฟเวอร์ปกติเพื่อเพิ่มฟีเจอร์ต่างๆ สำหรับบางกรณี คุณอาจต้องพึ่งพาเซิร์ฟเวอร์ระดับต่ำ เช่น:

- สถาปัตยกรรมที่ดีกว่า สามารถสร้างสถาปัตยกรรมที่สะอาดทั้งกับเซิร์ฟเวอร์ปกติและเซิร์ฟเวอร์ระดับต่ำได้ แต่สามารถโต้แย้งได้ว่าง่ายกว่าบ้างกับเซิร์ฟเวอร์ระดับต่ำ
- ความพร้อมใช้งานของฟีเจอร์ บางฟีเจอร์ขั้นสูงสามารถใช้ได้เฉพาะกับเซิร์ฟเวอร์ระดับต่ำเท่านั้น คุณจะได้เห็นในบทต่อไปเมื่อเราเพิ่มการสุ่มตัวอย่างและการกระตุ้น

## เซิร์ฟเวอร์ปกติกับเซิร์ฟเวอร์ระดับต่ำ

นี่คือลักษณะการสร้าง MCP Server ด้วยเซิร์ฟเวอร์ปกติ

**Python**

```python
mcp = FastMCP("Demo")

# เพิ่มเครื่องมือสำหรับการบวก
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

// เพิ่มเครื่องมือบวก
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

ประเด็นคือคุณต้องเพิ่มแต่ละเครื่องมือ แหล่งข้อมูล หรือคำสั่งที่คุณต้องการให้เซิร์ฟเวอร์มีอย่างชัดเจน ไม่มีอะไรผิดกับวิธีนี้  

### แนวทางเซิร์ฟเวอร์ระดับต่ำ

อย่างไรก็ตาม เมื่อใช้แนวทางเซิร์ฟเวอร์ระดับต่ำ คุณต้องคิดแตกต่างออกไป โดยแทนที่จะลงทะเบียนเครื่องมือแต่ละอย่าง คุณจะสร้างตัวจัดการสองตัวต่อประเภทฟีเจอร์ (เครื่องมือ แหล่งข้อมูล หรือคำสั่ง) เช่น สำหรับเครื่องมือจะมีเพียงสองฟังก์ชันเท่านั้นดังนี้:

- แสดงรายการเครื่องมือทั้งหมด ฟังก์ชันหนึ่งจะรับผิดชอบสำหรับการพยายามทั้งหมดที่จะเรียกรายการเครื่องมือ
- จัดการการเรียกใช้เครื่องมือทั้งหมด ในที่นี้ก็จะมีเพียงฟังก์ชันเดียวจัดการการเรียกใช้เครื่องมือ

ฟังดูเหมือนจะลดความยุ่งยากใช่ไหม? ดังนั้นแทนที่จะลงทะเบียนเครื่องมือ ฉันแค่ต้องแน่ใจว่าเครื่องมือนั้นแสดงเมื่อฉันเรียกรายการเครื่องมือ และเรียกใช้งานเมื่อมีคำขอเข้ามาเรียกเครื่องมือ

ลองดูโค้ดตอนนี้:

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
  // ส่งคืนรายการเครื่องมือที่ลงทะเบียนแล้ว
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

ตอนนี้เรามีฟังก์ชันที่ส่งคืนรายการของฟีเจอร์ รายการแต่ละตัวในรายการเครื่องมือจะมีฟิลด์เช่น `name`, `description` และ `inputSchema` เพื่อปฏิบัติตามประเภทการส่งคืน ซึ่งช่วยให้เราสามารถเก็บเครื่องมือและการกำหนดฟีเจอร์ไว้ที่อื่นได้ เราสามารถสร้างเครื่องมือทั้งหมดในโฟลเดอร์ tools และทำเช่นเดียวกันกับฟีเจอร์ทั้งหมด ดังนั้นโปรเจกต์ของคุณจึงสามารถจัดระเบียบได้ดังนี้:

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

ยอดเยี่ยม สถาปัตยกรรมของเราดูสะอาดตาขึ้นมาก

แล้วเรื่องการเรียกเครื่องมือล่ะ ไอเดียเดียวกันใช่ไหม เอาตัวจัดการเดียวเรียกเครื่องมือไหนก็ได้? ใช่เลย นี่คือโค้ดสำหรับการนั้น:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools เป็นพจนานุกรมที่มีชื่อเครื่องมือเป็นคีย์
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
    // TODO เรียกใช้เครื่องมือ,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

จากโค้ดด้านบน คุณจะเห็นว่าเราต้องแยกวิเคราะห์เครื่องมือที่ต้องการเรียก และด้วยอาร์กิวเมนต์อะไร แล้วจึงดำเนินการเรียกใช้เครื่องมือ

## การพัฒนาวิธีการด้วยการตรวจสอบความถูกต้อง

จนถึงตอนนี้ คุณได้เห็นว่าการลงทะเบียนเครื่องมือ แหล่งข้อมูล และคำสั่งทั้งหมดสามารถแทนที่ด้วยตัวจัดการสองตัวนี้ต่อประเภทฟีเจอร์ได้แล้ว แล้วต้องทำอะไรอีกไหม? เราควรเพิ่มการตรวจสอบความถูกต้องเพื่อให้แน่ใจว่าเครื่องมือถูกเรียกด้วยอาร์กิวเมนต์ที่ถูกต้อง แต่ละรันไทม์มีวิธีแก้ปัญหาของตัวเอง เช่น Python ใช้ Pydantic และ TypeScript ใช้ Zod แนวคิดคือเราทำสิ่งต่อไปนี้:

- ย้ายตรรกะสำหรับการสร้างฟีเจอร์ (เครื่องมือ แหล่งข้อมูล หรือคำสั่ง) ไปยังโฟลเดอร์เฉพาะของฟีเจอร์นั้น
- เพิ่มวิธีตรวจสอบคำขอเข้ามา เช่น การเรียกใช้เครื่องมือ

### การสร้างฟีเจอร์

เพื่อสร้างฟีเจอร์ เราต้องสร้างไฟล์สำหรับฟีเจอร์นั้นและต้องแน่ใจว่ามีฟิลด์บังคับตามที่ฟีเจอร์นั้นต้องการ ฟิลด์จะแตกต่างกันเล็กน้อยระหว่างเครื่องมือ แหล่งข้อมูล และคำสั่ง

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
        # ตรวจสอบความถูกต้องของอินพุตโดยใช้โมเดล Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: เพิ่ม Pydantic เพื่อให้เราสามารถสร้าง AddInputModel และตรวจสอบความถูกต้องของ args ได้

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ที่นี่คุณจะเห็นว่าทำสิ่งต่อไปนี้:

- สร้างสกีมาโดยใช้ Pydantic `AddInputModel` โดยมีฟิลด์ `a` และ `b` ในไฟล์ *schema.py*
- พยายามแยกวิเคราะห์คำขอที่เข้ามาให้เป็นชนิด `AddInputModel` หากพารามิเตอร์ไม่ตรงกันจะทำให้เกิดข้อผิดพลาด:

   ```python
   # add.py
    try:
        # ตรวจสอบความถูกต้องของข้อมูลนำเข้าด้วยโมเดล Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

คุณสามารถเลือกได้ว่าจะวางตรรกะการแยกวิเคราะห์นี้ไว้ในตัวเรียกใช้เครื่องมือเองหรือในฟังก์ชันตัวจัดการ

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

- ในตัวจัดการที่ดูแลการเรียกใช้เครื่องมือทั้งหมด ตอนนี้เราลองแยกวิเคราะห์คำขอที่เข้ามาเข้าสู่สกีมาที่เครื่องมือกำหนดไว้:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    หากทำงานได้ดี เราจะดำเนินการเรียกเครื่องมือจริง:

    ```typescript
    const result = await tool.callback(input);
    ```

อย่างที่คุณเห็น แนวทางนี้สร้างสถาปัตยกรรมที่ดี เพราะทุกอย่างมีที่ของมัน *server.ts* เป็นไฟล์ขนาดเล็กที่เชื่อมโยงตัวจัดการคำขอ และแต่ละฟีเจอร์แยกเป็นโฟลเดอร์ของตัวเอง เช่น tools/, resources/ หรือ /prompts

ดีมาก มาเริ่มสร้างสิ่งนี้กันต่อ

## แบบฝึกหัด: การสร้างเซิร์ฟเวอร์ระดับต่ำ

ในแบบฝึกหัดนี้ เราจะทำดังนี้:

1. สร้างเซิร์ฟเวอร์ระดับต่ำที่จัดการการแสดงรายการเครื่องมือและการเรียกใช้เครื่องมือ
1. นำเสนอสถาปัตยกรรมที่คุณสามารถพัฒนาต่อได้
1. เพิ่มการตรวจสอบความถูกต้องเพื่อให้แน่ใจว่าการเรียกเครื่องมือของคุณได้รับการตรวจสอบอย่างถูกต้อง

### -1- สร้างสถาปัตยกรรม

สิ่งแรกที่เราต้องจัดการคือสถาปัตยกรรมที่ช่วยให้เราขยายตัวได้เมื่อเพิ่มฟีเจอร์ นี่คือสิ่งที่ดูเหมือน:

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

ตอนนี้เราได้ตั้งค่าสถาปัตยกรรมที่ช่วยให้เพิ่มเครื่องมือใหม่ได้ง่ายในโฟลเดอร์ tools คุณสามารถเพิ่มไดเรกทอรีย่อยสำหรับ resources และ prompts ได้ตามต้องการ

### -2- การสร้างเครื่องมือ

มาดูว่าการสร้างเครื่องมือเป็นอย่างไร ก่อนอื่นต้องสร้างในไดเรกทอรีย่อย *tool* ดังนี้:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # ตรวจสอบข้อมูลนำเข้าด้วยโมเดล Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: เพิ่ม Pydantic เพื่อที่เราจะได้สร้าง AddInputModel และตรวจสอบ args

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ที่นี่เราจะเห็นการกำหนดชื่อ คำอธิบาย สกีมาอินพุตโดยใช้ Pydantic และตัวจัดการที่จะถูกเรียกเมื่อเครื่องมือนี้ถูกเรียก สุดท้ายเราเปิดเผย `tool_add` ซึ่งเป็นพจนานุกรมที่เก็บคุณสมบัติเหล่านี้ทั้งหมด

ยังมี *schema.py* ที่ใช้กำหนดสกีมาอินพุตของเครื่องมือ:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

เรายังต้องเติม *__init__.py* เพื่อให้ไดเรกทอ tools ถูกพิจารณาเป็นโมดูล นอกจากนี้ต้องเปิดเผยโมดูลภายในไฟล์นี้ดังนี้:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

เราสามารถเพิ่มได้เรื่อยๆ เมื่อเพิ่มเครื่องมือใหม่

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

ที่นี่เราสร้างพจนานุกรมที่ประกอบด้วยคุณสมบัติ:

- name ชื่อของเครื่องมือ
- rawSchema คือสกีมา Zod ใช้ตรวจสอบคำขอที่เข้ามาเรียกเครื่องมือนี้
- inputSchema สกีมานี้จะใช้โดยตัวจัดการ
- callback ใช้เรียกเครื่องมือจริง

นอกจากนี้ยังมี `Tool` ที่ใช้แปลงพจนานุกรมนี้เป็นชนิดที่ตัวจัดการ MCP Server รับได้ ดังนี้:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

และมี *schema.ts* ที่เก็บสกีมาของอินพุตสำหรับแต่ละเครื่องมือซึ่งดูเหมือนดังนี้ โดยมีสกีมาเดียวตอนนี้ แต่เราสามารถเพิ่มรายการได้เมื่อเพิ่มเครื่องมือ:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

เยี่ยม เรามาดูจัดการการแสดงรายการเครื่องมือต่อ

### -3- จัดการการแสดงรายการเครื่องมือ

ถัดไป เพื่อจัดการแสดงรายการเครื่องมือ เราต้องตั้งค่าตัวจัดการคำขอสำหรับเรื่องนี้ ดังนี้เพิ่มในไฟล์เซิร์ฟเวอร์ของเรา:

**Python**

```python
# โค้ดถูกตัดออกเพื่อความกระชับ
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

ที่นี่เราเพิ่มตัวตกแต่ง `@server.list_tools` และฟังก์ชัน `handle_list_tools` ในฟังก์ชันนี้เราต้องคืนรายการเครื่องมือ โปรดสังเกตว่าแต่ละเครื่องมือต้องมี name, description และ inputSchema  

**TypeScript**

เพื่อกำหนดตัวจัดการคำขอสำหรับการแสดงรายการเครื่องมือ เราต้องเรียกใช้ `setRequestHandler` บนเซิร์ฟเวอร์พร้อมสกีมาที่เหมาะสมกับสิ่งที่เราพยายามทำ ในกรณีนี้คือ `ListToolsRequestSchema`

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
// โค้ดถูกละเว้นเพื่อความกระชับ
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // คืนค่ารายการของเครื่องมือที่ลงทะเบียนแล้ว
  return {
    tools: tools
  };
});
```

เยี่ยม ตอนนี้เราแก้ปัญหาการแสดงรายการเครื่องมือแล้ว มาดูวิธีเรียกใช้เครื่องมือต่อกัน

### -4- จัดการการเรียกใช้เครื่องมือ

เพื่อเรียกใช้เครื่องมือ เราต้องตั้งค่าตัวจัดการคำขออีกตัวหนึ่ง ตอนนี้เน้นจัดการคำขอที่ระบุฟีเจอร์ที่จะเรียกและอาร์กิวเมนต์ที่ใช้

**Python**

ลองใช้ตัวตกแต่ง `@server.call_tool` และดำเนินการด้วยฟังก์ชัน `handle_call_tool` ภายในฟังก์ชันนี้เราต้องแยกวิเคราะห์ชื่อเครื่องมือ อาร์กิวเมนต์ และตรวจสอบว่าอาร์กิวเมนต์ถูกต้องสำหรับเครื่องมือหรือไม่ เราสามารถตรวจสอบความถูกต้องของอาร์กิวเมนต์ในฟังก์ชันนี้หรือในเครื่องมือจริงได้

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools เป็นพจนานุกรมที่มีชื่อเครื่องมือเป็นคีย์
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # เรียกใช้เครื่องมือ
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

สิ่งที่เกิดขึ้นคือ:

- ชื่อเครื่องมือมีอยู่แล้วในพารามิเตอร์อินพุต `name` และอาร์กิวเมนต์ในรูปแบบพจนานุกรม `arguments`

- เครื่องมือถูกเรียกใช้ด้วย `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` การตรวจสอบความถูกต้องของอาร์กิวเมนต์เกิดขึ้นในคุณสมบัติ `handler` ซึ่งชี้ไปยังฟังก์ชัน ถ้าไม่ถูกต้องจะเกิดข้อผิดพลาด

ตอนนี้เรามีความเข้าใจครบถ้วนเกี่ยวกับการแสดงรายการและเรียกใช้เครื่องมือโดยใช้เซิร์ฟเวอร์ระดับต่ำ

ดู [ตัวอย่างเต็ม](./code/README.md) ที่นี่

## งานที่ได้รับมอบหมาย

ขยายโค้ดที่คุณได้รับด้วยเครื่องมือ แหล่งข้อมูล และคำสั่งจำนวนหนึ่ง และสะท้อนให้เห็นว่าคุณสังเกตเห็นเพียงแค่ต้องเพิ่มไฟล์ในไดเรกทอ tools เท่านั้นและไม่ต้องทำที่อื่น

*ไม่มีคำตอบให้*

## สรุป

ในบทนี้ เราได้เห็นว่าแนวทางเซิร์ฟเวอร์ระดับต่ำทำงานอย่างไร และช่วยให้เราสร้างสถาปัตยกรรมที่ดีขึ้นได้อย่างไร เราได้พูดคุยเรื่องการตรวจสอบความถูกต้อง และคุณถูกสอนให้ทำงานกับไลบรารีตรวจสอบความถูกต้องเพื่อสร้างสกีมาในการตรวจสอบอินพุต

## ถัดไป

- ถัดไป: [การตรวจสอบสิทธิ์อย่างง่าย](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ข้อจำกัดความรับผิดชอบ**:  
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) แม้เราจะพยายามอย่างดีที่สุดเพื่อความถูกต้อง กรุณาทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาดั้งเดิมถือเป็นแหล่งข้อมูลมาตรฐาน สำหรับข้อมูลที่สำคัญ ขอแนะนำให้ใช้บริการแปลโดยนักแปลมืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดใด ๆ ที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# Sử dụng máy chủ nâng cao

Có hai loại máy chủ khác nhau được cung cấp trong MCP SDK, máy chủ thông thường và máy chủ cấp thấp. Thông thường, bạn sẽ sử dụng máy chủ thông thường để thêm các tính năng vào đó. Tuy nhiên, trong một số trường hợp, bạn muốn dựa vào máy chủ cấp thấp như:

- Kiến trúc tốt hơn. Có thể tạo một kiến trúc sạch với cả máy chủ thông thường và máy chủ cấp thấp nhưng có thể tranh luận rằng nó dễ dàng hơn một chút với máy chủ cấp thấp.
- Tính năng khả dụng. Một số tính năng nâng cao chỉ có thể sử dụng với máy chủ cấp thấp. Bạn sẽ thấy điều này trong các chương sau khi chúng ta thêm sampling và elicitation.

## Máy chủ thông thường so với máy chủ cấp thấp

Đây là cách tạo một MCP Server với máy chủ thông thường

**Python**

```python
mcp = FastMCP("Demo")

# Thêm một công cụ cộng
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

// Thêm một công cụ cộng thêm
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

Ý chính là bạn thêm rõ ràng từng công cụ, tài nguyên hoặc prompt mà bạn muốn máy chủ có. Không có gì sai với cách đó.

### Phương pháp máy chủ cấp thấp

Tuy nhiên, khi bạn sử dụng phương pháp máy chủ cấp thấp, bạn cần suy nghĩ khác đi, cụ thể thay vì đăng ký từng công cụ, bạn thay vào đó tạo hai hàm xử lý cho mỗi loại tính năng (công cụ, tài nguyên hoặc prompt). Ví dụ công cụ thì chỉ có hai hàm như sau:

- Liệt kê tất cả công cụ. Một hàm sẽ chịu trách nhiệm cho tất cả các lần cố gắng liệt kê công cụ.
- Xử lý gọi tất cả các công cụ. Ở đây cũng chỉ có một hàm xử lý các cuộc gọi tới công cụ.

Nghe có vẻ ít công việc hơn đúng không? Vậy thay vì đăng ký một công cụ, tôi chỉ cần đảm bảo công cụ được liệt kê khi tôi liệt kê tất cả các công cụ và được gọi khi có yêu cầu gọi công cụ đến.

Hãy xem đoạn mã bây giờ trông như thế nào:

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
  // Trả về danh sách các công cụ đã đăng ký
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

Ở đây bây giờ chúng ta có một hàm trả về danh sách các tính năng. Mỗi mục trong danh sách công cụ có các trường như `name`, `description` và `inputSchema` để tuân theo kiểu trả về. Điều này cho phép chúng ta đặt công cụ và định nghĩa tính năng ở nơi khác. Giờ chúng ta có thể tạo tất cả các công cụ trong thư mục tools và tương tự với tất cả các tính năng của bạn để dự án của bạn đột nhiên có thể được tổ chức như sau:

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

Thật tuyệt, kiến trúc của chúng ta có thể được làm cho rõ ràng khá tốt.

Còn việc gọi công cụ thì sao, có phải cũng cùng ý tưởng, một trình xử lý gọi công cụ, bất kỳ công cụ nào? Đúng vậy, chính xác, đây là đoạn mã cho việc đó:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools là một từ điển với tên công cụ làm khóa
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
    // TODO gọi công cụ,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Như bạn có thể thấy từ đoạn mã trên, chúng ta cần phân tích công cụ để gọi, và với những tham số gì, sau đó chúng ta cần tiến hành gọi công cụ đó.

## Cải thiện phương pháp với xác thực

Cho đến nay, bạn đã thấy cách tất cả các đăng ký công cụ, tài nguyên và prompt có thể được thay thế bằng hai trình xử lý cho mỗi loại tính năng. Vậy còn cần làm gì nữa? Chúng ta nên thêm một hình thức xác thực để đảm bảo rằng công cụ được gọi với các tham số đúng. Mỗi runtime có giải pháp riêng của mình cho việc này, ví dụ Python sử dụng Pydantic và TypeScript sử dụng Zod. Ý tưởng là chúng ta làm như sau:

- Di chuyển logic tạo tính năng (công cụ, tài nguyên hoặc prompt) vào thư mục riêng của nó.
- Thêm cách để xác thực một yêu cầu đến, ví dụ như gọi một công cụ.

### Tạo một tính năng

Để tạo một tính năng, chúng ta cần tạo một file cho tính năng đó và đảm bảo nó có các trường bắt buộc dành cho tính năng đó. Các trường này hơi khác nhau giữa công cụ, tài nguyên và prompt.

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
        # Xác thực đầu vào sử dụng mô hình Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: thêm Pydantic, để chúng ta có thể tạo AddInputModel và xác thực các đối số

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ở đây bạn có thể thấy cách chúng ta làm như sau:

- Tạo một schema bằng Pydantic `AddInputModel` với các trường `a` và `b` trong file *schema.py*.
- Cố gắng phân tích yêu cầu đến thành kiểu `AddInputModel`, nếu có sự không khớp tham số thì sẽ gây lỗi:

   ```python
   # add.py
    try:
        # Xác thực đầu vào sử dụng mô hình Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Bạn có thể chọn đặt logic phân tích này trong chính cuộc gọi công cụ hoặc trong hàm xử lý.

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

- Trong trình xử lý xử lý tất cả các cuộc gọi công cụ, giờ chúng ta cố gắng phân tích yêu cầu đến thành schema được định nghĩa của công cụ:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    nếu điều đó thành công thì ta tiến hành gọi công cụ thực tế:

    ```typescript
    const result = await tool.callback(input);
    ```

Như bạn thấy, phương pháp này tạo ra một kiến trúc tuyệt vời vì mọi thứ đều có chỗ của nó, file *server.ts* rất nhỏ chỉ để liên kết các trình xử lý yêu cầu và mỗi tính năng nằm trong thư mục tương ứng của chúng, ví dụ tools/, resources/ hoặc prompts/.

Tuyệt vời, hãy thử xây dựng điều này tiếp theo.

## Bài tập: Tạo một máy chủ cấp thấp

Trong bài tập này, chúng ta sẽ làm như sau:

1. Tạo một máy chủ cấp thấp xử lý liệt kê công cụ và gọi công cụ.
2. Triển khai kiến trúc bạn có thể xây dựng tiếp.
3. Thêm xác thực để đảm bảo các cuộc gọi công cụ được kiểm tra đúng.

### -1- Tạo kiến trúc

Điều đầu tiên chúng ta cần giải quyết là một kiến trúc giúp mở rộng khi ta thêm nhiều tính năng hơn, nó trông như sau:

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

Bây giờ chúng ta đã thiết lập một kiến trúc đảm bảo ta có thể dễ dàng thêm công cụ mới trong thư mục tools. Bạn có thể thêm các thư mục con cho resources và prompts.

### -2- Tạo một công cụ

Hãy xem tạo một công cụ như thế nào tiếp theo. Đầu tiên, nó cần được tạo trong thư mục con *tool* như sau:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Xác thực đầu vào sử dụng mô hình Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: thêm Pydantic, để chúng ta có thể tạo một AddInputModel và xác thực các đối số

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Ở đây chúng ta thấy cách định nghĩa tên, mô tả, một schema đầu vào sử dụng Pydantic và một hàm xử lý sẽ được gọi khi công cụ này được gọi. Cuối cùng, chúng ta khai báo `tool_add` là một dictionary chứa tất cả các thuộc tính này.

Ngoài ra còn có *schema.py* dùng để định nghĩa schema đầu vào của công cụ:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Chúng ta cũng cần điền vào *__init__.py* để đảm bảo thư mục tools được coi là một module. Thêm vào đó chúng ta cần khai báo các module trong nó như sau:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Chúng ta có thể tiếp tục thêm vào file này khi thêm nhiều công cụ hơn.

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

Ở đây chúng ta tạo một dictionary gồm các thuộc tính:

- name, đây là tên công cụ.
- rawSchema, đây là schema Zod, sẽ dùng để xác thực các yêu cầu gọi công cụ này.
- inputSchema, schema này sẽ được trình xử lý sử dụng.
- callback, dùng để gọi công cụ.

Ngoài ra còn có `Tool` dùng để chuyển dictionary này thành kiểu mà trình xử lý của mcp server có thể chấp nhận, trông như sau:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Và có file *schema.ts* chứa các schema đầu vào cho mỗi công cụ như sau, hiện chỉ có một schema nhưng khi thêm công cụ ta có thể thêm nhiều mục hơn:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Tuyệt vời, hãy tiếp tục xử lý việc liệt kê công cụ tiếp theo.

### -3- Xử lý liệt kê công cụ

Tiếp theo, để xử lý liệt kê công cụ, ta cần thiết lập một trình xử lý yêu cầu cho điều đó. Đây là những gì ta cần thêm vào tập tin server của mình:

**Python**

```python
# mã đã bị loại bỏ để rút gọn
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

Ở đây ta thêm decorator `@server.list_tools` và hàm triển khai `handle_list_tools`. Trong hàm này, ta cần tạo danh sách các công cụ. Chú ý mỗi công cụ cần có tên, mô tả và inputSchema.

**TypeScript**

Để thiết lập trình xử lý yêu cầu liệt kê công cụ, ta cần gọi `setRequestHandler` trên server với một schema phù hợp với việc ta muốn làm, trong trường hợp này là `ListToolsRequestSchema`.

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
// mã đã bị bỏ qua cho ngắn gọn
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Trả về danh sách các công cụ đã đăng ký
  return {
    tools: tools
  };
});
```

Tuyệt vời, giờ ta đã giải quyết xong phần liệt kê công cụ, hãy xem cách ta gọi công cụ tiếp theo.

### -4- Xử lý gọi công cụ

Để gọi một công cụ, ta cần thiết lập một trình xử lý yêu cầu nữa, lần này tập trung vào xử lý yêu cầu xác định tính năng nào được gọi và với những tham số nào.

**Python**

Hãy sử dụng decorator `@server.call_tool` và triển khai nó với một hàm như `handle_call_tool`. Trong hàm đó, ta cần phân tích tên công cụ, tham số của nó và đảm bảo tham số hợp lệ với công cụ được gọi. Ta có thể xác thực tham số trong hàm này hoặc trong chính công cụ.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools là một từ điển với tên công cụ làm khóa
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # gọi công cụ
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Đây là những gì diễn ra:

- Tên công cụ đã có sẵn là tham số đầu vào `name` và tham số truyền vào ở dạng dictionary `arguments`.

- Công cụ được gọi với `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Việc xác thực tham số diễn ra trong thuộc tính `handler` trỏ tới một hàm, nếu không hợp lệ, sẽ ném ngoại lệ.

Vậy là, giờ chúng ta đã hiểu đầy đủ về cách liệt kê và gọi công cụ sử dụng máy chủ cấp thấp.

Xem [ví dụ đầy đủ](./code/README.md) ở đây

## Bài tập

Mở rộng mã code đã được cung cấp với một số công cụ, tài nguyên và prompt và suy ngẫm về việc bạn chỉ cần thêm các file trong thư mục tools mà không cần nơi nào khác.

*Không có giải pháp*

## Tóm tắt

Trong chương này, chúng ta đã thấy cách hoạt động của phương pháp máy chủ cấp thấp và cách nó giúp tạo ra một kiến trúc đẹp mà ta có thể tiếp tục xây dựng. Chúng ta cũng đã thảo luận về xác thực và bạn được hướng dẫn cách sử dụng các thư viện xác thực để tạo schema cho việc kiểm tra đầu vào.

## Tiếp theo

- Tiếp theo: [Xác thực đơn giản](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố từ chối trách nhiệm**:  
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc không chính xác. Tài liệu gốc bằng ngôn ngữ gốc được coi là nguồn chính xác và đáng tin cậy. Đối với các thông tin quan trọng, khuyến nghị sử dụng dịch vụ dịch thuật chuyên nghiệp do con người thực hiện. Chúng tôi không chịu trách nhiệm về bất kỳ sự hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# 고급 서버 사용법

MCP SDK에는 두 가지 유형의 서버가 있습니다. 일반 서버와 저수준 서버입니다. 일반적으로는 기능을 추가하기 위해 일반 서버를 사용합니다. 그러나 다음과 같은 경우에는 저수준 서버를 사용하는 것이 좋습니다:

- 더 나은 아키텍처. 일반 서버와 저수준 서버를 함께 사용하여 깨끗한 아키텍처를 만드는 것이 가능하지만, 저수준 서버 쪽이 조금 더 간편하다고 할 수 있습니다.
- 기능 가용성. 일부 고급 기능은 저수준 서버에서만 사용할 수 있습니다. 샘플링 및 엘리시테이션을 추가하는 후속 장에서 이를 보게 될 것입니다.

## 일반 서버와 저수준 서버

일반 서버로 MCP Server를 생성하는 예는 다음과 같습니다.

**Python**

```python
mcp = FastMCP("Demo")

# 덧셈 도구 추가
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

// 추가 도구를 추가하십시오
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

요점은 서버에 포함시키고자 하는 각 도구, 리소스, 프롬프트를 명시적으로 추가한다는 것입니다. 이것 자체는 문제가 없습니다.

### 저수준 서버 접근법

하지만 저수준 서버 접근법을 사용할 때는 다르게 생각해야 합니다. 즉, 각 도구를 등록하는 대신 기능 유형별(도구, 리소스, 프롬프트)로 두 개의 핸들러만 생성합니다. 예를 들어 도구의 경우 다음 두 가지 함수만 있습니다:

- 모든 도구를 나열하는 함수. 모든 도구 목록 조회 시 이 함수가 호출됩니다.
- 모든 도구 호출을 처리하는 함수. 도구 호출 요청에 대해 이 함수가 처리합니다.

이것은 잠재적으로 작업이 줄어드는 것처럼 들리죠? 도구를 등록하는 대신 모든 도구를 나열할 때 목록에 도구가 포함되어 있고, 도구 호출 요청이 있을 때 호출되도록 하면 됩니다.

코드는 다음과 같이 됩니다:

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
  // 등록된 도구 목록을 반환합니다
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

여기서는 기능 목록을 반환하는 함수를 생성했습니다. 도구 목록의 각 항목은 반환 타입을 준수하기 위해 `name`, `description`, `inputSchema` 같은 필드를 가집니다. 이렇게 하면 도구와 기능 정의를 다른 곳에 둘 수 있습니다. 이제 도구 폴더에 모든 도구를 만들 수 있고, 마찬가지로 모든 기능도 각각의 폴더로 분리할 수 있어 프로젝트는 다음과 같이 깔끔하게 구성될 수 있습니다.

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

훌륭합니다. 아키텍처가 상당히 깔끔해질 수 있습니다.

도구 호출은 어떨까요? 똑같은 아이디어인가요? 네, 어떤 도구든 호출할 수 있는 하나의 핸들러입니다. 다음은 그 코드입니다:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools는 도구 이름을 키로 사용하는 사전입니다
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
    // TODO 도구 호출,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

위 코드를 보면 호출할 도구와 인자를 파싱한 후, 도구를 호출하는 과정을 수행함을 알 수 있습니다.

## 검증을 통한 접근법 개선

지금까지 도구, 리소스, 프롬프트를 추가하는 모든 등록을 기능 유형별 두 개의 핸들러로 대체하는 방법을 보았습니다. 그 외에 해야 할 것은 뭘까요? 도구를 올바른 인자와 함께 호출하는지 검증하는 형태를 추가해야 합니다. 각 런타임에는 자체 솔루션이 있습니다. 예를 들면 Python은 Pydantic을, TypeScript는 Zod를 사용합니다. 아이디어는 다음과 같습니다:

- 기능(도구, 리소스, 프롬프트) 생성 로직을 전용 폴더로 이동합니다.
- 도구 호출 요청이 들어왔을 때 이를 검증할 수 있도록 합니다.

### 기능 생성하기

기능을 생성하려면 해당 기능의 파일을 만들고 그 기능에 필요한 필수 필드를 포함시켜야 합니다. 도구, 리소스, 프롬프트마다 일부 필드가 다를 수 있습니다.

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
        # Pydantic 모델을 사용하여 입력값 검증
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic 추가, AddInputModel을 생성하고 인수를 검증할 수 있도록 하기 위해

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

여기서는 다음을 수행합니다:

- *schema.py* 파일에 Pydantic `AddInputModel` 스키마를 만들어 `a`와 `b` 필드를 정의합니다.
- 들어오는 요청을 `AddInputModel` 형식으로 파싱을 시도하며, 인자 불일치시 오류가 발생합니다:

   ```python
   # add.py
    try:
        # Pydantic 모델을 사용하여 입력값을 검증합니다
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

이 파싱 로직을 도구 호출 함수 내에 두거나 핸들러 함수 내에 둘 수 있습니다.

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

- 모든 도구 호출을 처리하는 핸들러 내에서 도구의 정의된 스키마에 따라 들어오는 요청을 파싱하려고 시도합니다:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

성공하면 실제 도구를 호출합니다:

    ```typescript
    const result = await tool.callback(input);
    ```

이 접근법은 모든 것이 적절한 위치에 배치되어 훌륭한 아키텍처를 만듭니다. *server.ts* 파일은 요청 핸들러를 연결하는 아주 작은 파일이며, 각 기능은 각각의 폴더(도구, 리소스, 프롬프트)에 있습니다.

좋습니다, 이제 이것을 구현해 봅시다.

## 연습: 저수준 서버 만들기

이번 연습에서 다음을 수행할 것입니다:

1. 도구 목록 조회 및 도구 호출을 처리하는 저수준 서버를 만듭니다.
2. 확장 가능한 아키텍처를 구현합니다.
3. 도구 호출 검증을 추가합니다.

### -1- 아키텍처 만들기

먼저 확장성을 도와줄 아키텍처를 만들어야 합니다. 다음과 같습니다:

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

이제 tools 폴더에 새 도구를 쉽게 추가할 수 있도록 아키텍처를 구성했습니다. 리소스 및 프롬프트에 대해서도 하위 디렉터리를 추가해도 좋습니다.

### -2- 도구 만들기

다음은 도구를 만드는 모습입니다. 먼저 도구는 *tool* 하위 디렉터리에 만들어야 합니다:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic 모델을 사용하여 입력을 검증합니다
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic을 추가하여 AddInputModel을 만들고 인수를 검증할 수 있도록 합니다

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

여기서는 이름, 설명, Pydantic을 사용한 입력 스키마와 호출될 때 실행되는 핸들러를 정의합니다. 마지막으로, 이러한 속성을 모두 갖는 `tool_add` 딕셔너리를 노출합니다.

또한, 도구에서 사용할 입력 스키마를 정의하는 *schema.py*도 있습니다:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

*__init__.py* 파일도 작성해 tools 디렉터리를 모듈로 인식하게 하고, 다음과 같이 내부 모듈을 노출해야 합니다:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

도구가 늘어나면 이 파일에 계속 추가하면 됩니다.

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

여기에서는 다음 속성을 가진 딕셔너리를 만듭니다:

- name: 도구 이름
- rawSchema: Zod 스키마, 이 스키마로 도구 호출 요청을 검증합니다.
- inputSchema: 핸들러가 사용하는 스키마
- callback: 도구를 호출하는 함수

`Tool` 타입은 이 딕셔너리를 MCP 서버 핸들러가 처리할 수 있는 타입으로 변환하며, 다음과 같습니다:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

그리고 각 도구의 입력 스키마를 저장하는 *schema.ts*가 있습니다. 현재는 하나의 스키마만 포함하지만 도구가 많아지면 더 추가할 수 있습니다:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

좋습니다. 다음은 도구 목록 조회를 처리해 봅시다.

### -3- 도구 목록 조회 처리

도구 목록 조회를 처리하려면 요청 핸들러를 설정해야 합니다. 서버 파일에 다음을 추가하세요:

**Python**

```python
# 간결함을 위해 코드 생략됨
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

여기서 `@server.list_tools` 데코레이터와 `handle_list_tools` 구현 함수를 추가합니다. 후자는 도구 목록을 생성합니다. 각 도구에는 이름, 설명, inputSchema가 있어야 합니다.

**TypeScript**

도구 목록 조회 요청 핸들러 설정을 위해 서버에 `setRequestHandler`를 호출하고, 시도 중인 작업에 맞는 스키마로 `ListToolsRequestSchema`를 사용합니다.

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
// 간결함을 위해 코드 생략
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // 등록된 도구 목록을 반환합니다
  return {
    tools: tools
  };
});
```

이제 도구 목록 조회가 해결되었으니, 도구 호출은 어떻게 할지 살펴보겠습니다.

### -4- 도구 호출 처리

도구를 호출하려면 요청 핸들러를 설정해야 하며, 이 때 호출할 기능과 인자를 지정한 요청을 처리합니다.

**Python**

`@server.call_tool` 데코레이터를 사용하고 `handle_call_tool` 함수로 구현합시다. 이 함수 내에서 도구 이름과 인자를 파싱하고, 유효한 인자인지 확인합니다. 검증은 이 함수 내 또는 실제 도구 내에서 할 수 있습니다.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools는 도구 이름을 키로 가진 딕셔너리입니다
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # 도구를 호출합니다
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

일어나는 일:

- 도구 이름은 이미 입력 매개변수 `name`에 있고, 인자는 `arguments` 딕셔너리에 있습니다.
- 도구는 `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`를 통해 호출됩니다. 인자 검증은 이 `handler` 함수 내에서 수행되며 실패 시 예외가 발생합니다.

이제 저수준 서버를 사용하여 도구 목록을 조회하고 호출하는 과정을 완전히 이해했습니다.

전체 예제는 [여기](./code/README.md)에서 확인할 수 있습니다.

## 과제

제공된 코드를 여러 도구, 리소스, 프롬프트로 확장해 보세요. 그리고 도구 디렉터리에 파일만 추가하면 되고 다른 곳에서는 추가할 필요가 없음을 인식해 보세요.

*해답 없음*

## 요약

이번 장에서는 저수준 서버 접근법이 어떻게 동작하는지, 그리고 이를 통해 계속 확장 가능한 좋은 아키텍처를 만드는 방법을 보았습니다. 또한 검증에 대해 논의하였고, 입력 검증을 위한 스키마를 만드는 데 사용되는 검증 라이브러리들을 사용하는 방법도 보여드렸습니다.

## 다음 내용

- 다음: [간단한 인증](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:  
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 위해 최선을 다하고 있으나, 자동 번역에는 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서는 해당 언어의 공식적인 자료로 간주되어야 합니다. 중요한 정보의 경우 전문 인간 번역을 권장합니다. 이 번역의 사용으로 인해 발생하는 모든 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
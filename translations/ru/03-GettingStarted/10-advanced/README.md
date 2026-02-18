# Расширенное использование сервера

В MCP SDK представлены два разных типа серверов: обычный сервер и низкоуровневый сервер. Обычно вы используете обычный сервер, чтобы добавлять функции. Однако в некоторых случаях вам может понадобиться опираться на низкоуровневый сервер, например:

- Лучшее архитектурное решение. Можно создать чистую архитектуру и с помощью обычного сервера, и с помощью низкоуровневого, но можно утверждать, что с низкоуровневым сервером это сделать немного проще.
- Доступность функций. Некоторые продвинутые функции можно использовать только с низкоуровневым сервером. Вы увидите это в последующих главах, когда мы добавим выборку и элицитацию.

## Обычный сервер и низкоуровневый сервер

Вот как выглядит создание MCP Server с использованием обычного сервера

**Python**

```python
mcp = FastMCP("Demo")

# Добавить инструмент сложения
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

// Добавить инструмент сложения
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

Суть в том, что вы явно добавляете каждый инструмент, ресурс или подсказку, которые хотите видеть на сервере. В этом нет ничего плохого.

### Подход низкоуровневого сервера

Однако при использовании низкоуровневого сервера вам нужно мыслить иначе: вместо регистрации каждого инструмента создаются два обработчика на каждый тип функции (инструменты, ресурсы или подсказки). Например, для инструментов всего две функции:

- Перечисление всех инструментов. Одна функция отвечает за все попытки получить список инструментов.
- Обработка вызова инструментов. Здесь тоже только одна функция обрабатывает вызовы инструментов.

Звучит как потенциальное облегчение работы, верно? Вместо регистрации инструмента мне нужно просто удостовериться, что инструмент есть в списке при перечислении, и что он вызывается при поступлении запроса на вызов.

Посмотрим, как теперь выглядит код:

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
  // Вернуть список зарегистрированных инструментов
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

Теперь у нас есть функция, которая возвращает список функций. Каждый элемент в списке инструментов содержит поля `name`, `description` и `inputSchema`, чтобы соответствовать типу возвращаемого значения. Это позволяет размещать определения инструментов и функций в другом месте. Теперь мы можем создать все инструменты в папке tools, и то же самое касается всех функций, так что ваш проект может выглядеть примерно так:

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

Отлично, наша архитектура может выглядеть довольно чистой.

А как насчет вызова инструментов — это та же идея, один обработчик на вызов любого инструмента? Да, именно так, вот код:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools — это словарь с названиями инструментов в качестве ключей
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
    // TODO вызвать инструмент,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Как видите из кода выше, нам нужно разобрать, какой инструмент вызвать и с какими аргументами, после чего перейти к вызову инструмента.

## Улучшение подхода с валидацией

Пока вы видели, как все ваши регистрации для добавления инструментов, ресурсов и подсказок можно заменить двумя обработчиками на каждый тип функций. Что еще нужно сделать? Следует добавить некоторую форму валидации, чтобы гарантировать, что инструмент вызывается с правильными аргументами. В каждом рантайме есть свое решение для этого, например, Python использует Pydantic, а TypeScript — Zod. Суть в следующем:

- Переместить логику создания функции (инструмента, ресурса или подсказки) в её собственную папку.
- Добавить способ валидировать входящий запрос, например, при вызове инструмента.

### Создание функции

Для создания функции нам нужно создать файл для этой функции и убедиться, что она содержит обязательные поля, требуемые для этой функции. Набор полей немного отличается для инструментов, ресурсов и подсказок.

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
        # Проверить ввод с помощью модели Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: добавить Pydantic, чтобы мы могли создать AddInputModel и проверять аргументы

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Здесь показано следующее:

- Создание схемы с помощью Pydantic `AddInputModel` с полями `a` и `b` в файле *schema.py*.
- Попытка распарсить входящий запрос как `AddInputModel`; если параметры не совпадают, будет ошибка:

   ```python
   # add.py
    try:
        # Проверить ввод с использованием модели Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Вы можете выбрать, где разместить эту логику парсинга — непосредственно в вызове инструмента или в обработчике.

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

- В обработчике, который обрабатывает вызовы всех инструментов, мы пытаемся распарсить входящий запрос в схему инструмента:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    Если это сработает, мы продолжаем вызов самого инструмента:

    ```typescript
    const result = await tool.callback(input);
    ```

Как видите, такой подход создаёт хорошую архитектуру: всё на своих местах, *server.ts* — очень маленький файл, который лишь связывает обработчики запросов, а каждая функция находится в своей папке: tools/, resources/ или prompts/.

Отлично, давайте попробуем это реализовать дальше.

## Упражнение: Создание низкоуровневого сервера

В этом упражнении мы сделаем следующее:

1. Создадим низкоуровневый сервер, обрабатывающий перечисление и вызов инструментов.
2. Реализуем архитектуру, на которую можно будет опираться и развивать.
3. Добавим валидацию, чтобы гарантировать правильность вызовов инструментов.

### -1- Создание архитектуры

Первое, что нужно сделать — создать архитектуру, которая поможет нам масштабироваться при добавлении новых функций. Вот как она выглядит:

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

Теперь у нас настроена архитектура, которая позволяет легко добавлять новые инструменты в папку tools. Вы можете по аналогии добавить подпапки для ресурсов и подсказок.

### -2- Создание инструмента

Давайте посмотрим, как создать инструмент. Сначала его нужно создать в подкаталоге *tool* так:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Проверить ввод с помощью модели Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: добавить Pydantic, чтобы мы могли создать AddInputModel и проверить аргументы

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Здесь мы видим, как определяется имя, описание, входная схема с помощью Pydantic и обработчик, который будет вызываться при обращении к этому инструменту. В конце мы экспортируем `tool_add`, словарь, содержащий все эти свойства.

Также есть *schema.py*, где определена входная схема инструмента:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Нужно заполнить *__init__.py*, чтобы папка tools считалась модулем. Кроме того, нужно экспортировать модули внутри неё, так:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Мы можем добавлять новые инструменты в этот файл по мере необходимости.

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

Здесь создаётся словарь, содержащий свойства:

- name — имя инструмента.
- rawSchema — схема Zod, используемая для валидации входящих запросов на вызов инструмента.
- inputSchema — схема, используемая обработчиком.
- callback — функция вызова инструмента.

Есть также `Tool`, который преобразует этот словарь в тип, который может принять обработчик сервера MCP, и выглядит он так:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

В *schema.ts* хранятся входные схемы для каждого инструмента, сейчас там всего одна схема, но по мере добавления инструментов можно добавлять новые записи:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Отлично, теперь перейдем к обработке перечисления инструментов.

### -3- Обработка перечисления инструментов

Для обработки списка инструментов нужно настроить обработчик запроса. Вот что необходимо добавить в файл сервера:

**Python**

```python
# код опущен для краткости
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

Здесь добавляем декоратор `@server.list_tools` и функцию `handle_list_tools`. В ней нужно сформировать список инструментов. Обратите внимание, что каждый инструмент должен иметь имя, описание и inputSchema.

**TypeScript**

Для настройки обработчика запроса на перечисление инструментов вызываем `setRequestHandler` на сервере с подходящей схемой, в данном случае `ListToolsRequestSchema`.

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
// код опущен для краткости
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Возвращает список зарегистрированных инструментов
  return {
    tools: tools
  };
});
```

Отлично, теперь мы умеем перечислять инструменты, посмотрим, как обрабатывать вызов инструментов.

### -4- Обработка вызова инструмента

Для вызова инструмента нам нужен ещё один обработчик, который будет обрабатывать запросы с указанием, какую функцию вызывать и с какими аргументами.

**Python**

Используем декоратор `@server.call_tool` и реализуем функцию `handle_call_tool`. Внутри нужно распарсить имя инструмента, его аргументы и проверить корректность аргументов для выбранного инструмента. Валидацию можно сделать здесь или внутри самого инструмента.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools — словарь с названиями инструментов в качестве ключей
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # вызвать инструмент
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Вот что происходит:

- Имя инструмента уже передано через параметр `name`, а аргументы — в словаре `arguments`.

- Инструмент вызывается с помощью `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Валидация аргументов происходит в функции в свойстве `handler`: если она не пройдёт, будет выброшено исключение.

Теперь мы полностью понимаем, как перечислять и вызывать инструменты с помощью низкоуровневого сервера.

См. [полный пример](./code/README.md)

## Задание

Расширьте код, который вы получили, добавьте несколько инструментов, ресурсов и подсказок и понаблюдайте, что вам нужно добавлять файлы только в папку tools и нигде больше.

*Решение не предоставлено*

## Итоги

В этой главе мы узнали, как работает подход низкоуровневого сервера и как он помогает создавать удобную архитектуру, на которую можно опираться в дальнейшем. Также мы обсудили валидацию и показали, как работать с библиотеками валидации для создания схем для входных данных.

## Что дальше

- Далее: [Простая аутентификация](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:  
Этот документ был переведен с помощью сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, следует учитывать, что автоматические переводы могут содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
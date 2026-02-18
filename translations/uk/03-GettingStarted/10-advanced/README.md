# Просунутий сервісний сервер

У MCP SDK існують два різні типи серверів: ваш звичайний сервер і низькорівневий сервер. Зазвичай ви використовуєте звичайний сервер для додавання функцій. Однак у деяких випадках варто спиратися на низькорівневий сервер, наприклад:

- Краща архітектура. Можна створити чисту архітектуру із звичайним сервером і низькорівневим сервером, але можна стверджувати, що з низькорівневим сервером це трохи простіше.
- Доступність функцій. Деякі просунуті функції можна використовувати лише з низькорівневим сервером. Ви побачите це в наступних розділах, коли ми додаватимемо семплінг та елікитацію.

## Звичайний сервер проти низькорівневого сервера

Ось як виглядає створення MCP Server зі звичайним сервером

**Python**

```python
mcp = FastMCP("Demo")

# Додати інструмент додавання
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

// Додати інструмент додавання
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

Суть у тому, що ви явно додаєте кожен інструмент, ресурс або запит, які хочете мати на сервері. У цьому немає нічого поганого.

### Підхід низькорівневого сервера

Однак при використанні підходу низькорівневого сервера потрібно думати інакше, а саме замість реєстрації кожного інструмента ви створюєте по два обробники на тип функції (інструменти, ресурси або запити). Наприклад, інструменти мають лише дві функції, як-от:

- Перелік усіх інструментів. Одна функція відповідає за всі спроби отримати список інструментів.
- Обробка виклику інструментів. Тут також лише одна функція обробляє виклики інструментів.

Звучить так, ніби це потенційно менше роботи, правда? Тобто замість реєстрації інструмента потрібно лише переконатися, що він є у списку при переліку всієї колекції, і що його викликають, коли надходить запит на використання інструмента.

Давайте подивимося, як тепер виглядає код:

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
  // Повернути список зареєстрованих інструментів
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

Тут у нас функція, яка повертає список функцій. Кожен запис у списку інструментів тепер має поля, як-от `name`, `description` і `inputSchema`, згідно з типом повернення. Це дозволяє розміщувати визначення інструментів і функцій окремо. Тепер ми можемо створювати всі наші інструменти в папці tools, і так само для всіх функцій, тож ваш проект може організуватися так:

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

Це чудово, наша архітектура може стати доволі чистою.

Що з викликом інструментів? Це ж ідея, один обробник для виклику будь-якого інструмента? Так, саме так, ось код для цього:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools — це словник з назвами інструментів як ключі
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
    // TODO викликати інструмент,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Як видно з коду вище, нам потрібно розпарсити, який інструмент викликати, з якими аргументами, а потім викликати інструмент.

## Покращення підходу за допомогою валідації

До цього ви бачили, як всі ваші реєстрації для додавання інструментів, ресурсів і запитів можна замінити на два обробники на тип функції. Що ще потрібно зробити? Нам слід додати форму валідації, щоб гарантувати правильність аргументів інструмента при його виклику. Кожне середовище виконання має власне рішення, наприклад, Python використовує Pydantic, а TypeScript — Zod. Ідея полягає в наступному:

- Перемістити логіку створення функції (інструмент, ресурс або запит) у відповідну папку.
- Додати спосіб валідувати вхідні запити, які зокрема просять викликати інструмент.

### Створення функції

Щоб створити функцію, потрібно створити файл для неї і переконатися, що він має обов’язкові поля, потрібні для відповідної функції. Ці поля дещо відрізняються для інструментів, ресурсів і запитів.

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
        # Перевірити вхідні дані за допомогою моделі Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: додати Pydantic, щоб ми могли створити AddInputModel і перевіряти аргументи

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

тут ви бачите, що ми робимо наступне:

- Створюємо схему Pydantic `AddInputModel` з полями `a` та `b` у файлі *schema.py*.
- Прагнемо розпарсити вхідний запит як тип `AddInputModel`; якщо параметри не співпадають — це призведе до помилки:

   ```python
   # add.py
    try:
        # Перевірка вхідних даних за допомогою Pydantic моделі
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Ви можете вирішити, чи виконувати цю логіку розбору у самому виклику інструмента, чи в функції обробника.

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

- В обробнику для всіх викликів інструментів тепер намагаємося розпарсити вхідний запит згідно зі схемою інструмента:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    якщо це вдасться — переходимо до виклику самого інструмента:

    ```typescript
    const result = await tool.callback(input);
    ```

Як видно, цей підхід утворює чудову архітектуру, оскільки все має своє місце: *server.ts* — дуже маленький файл, який лише підключає обробники запитів, а кожна функція знаходиться у своїй папці, наприклад tools/, resources/ або prompts/.

Чудово, давайте спробуємо це реалізувати.

## Вправа: Створення низькорівневого сервера

У цій вправі ми виконаємо наступне:

1. Створимо низькорівневий сервер, що обробляє перелік інструментів і виклики інструментів.
2. Реалізуємо архітектуру, на якій можна базувати подальшу розробку.
3. Додамо валідацію, щоб переконатися, що виклики інструментів коректні.

### -1- Створюємо архітектуру

Перше, що потрібно зробити — це архітектура, яка допоможе нам масштабуватися при додаванні нових функцій, ось як це виглядає:

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

Тепер ми налаштували архітектуру, яка забезпечує легкість додавання нових інструментів у папці tools. За бажанням можете додати підпапки для resources і prompts.

### -2- Створення інструмента

Далі подивимося, як створити інструмент. Спершу він має бути створений у власній підпапці tools так:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Перевірте вхідні дані за допомогою моделі Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: додати Pydantic, щоб ми могли створити AddInputModel і перевірити аргументи

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Тут ми визначаємо ім’я, опис, схему вводу за допомогою Pydantic і обробник, який викликається, коли інструмент викликають. Насамкінець експортуємо `tool_add`, словник з цими властивостями.

Також є *schema.py* для визначення схеми вводу для інструмента:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Потрібно також заповнити *__init__.py*, щоб папку tools розглядали як модуль. Додатково потрібно експортувати модулі всередині так:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Можна додавати сюди нові інструменти за потреби.

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

Тут створюється словник із такими властивостями:

- name — назва інструмента.
- rawSchema — схема Zod, використовується для валідації запитів на виклик інструмента.
- inputSchema — ця схема використовується обробником.
- callback — функція, що викликає інструмент.

Є також `Tool`, що перетворює цей словник у тип, який MCP server handler може прийняти, і це виглядає так:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

У *schema.ts* зберігаються схеми вводу для кожного інструмента, наразі лише одна схема, але можна додавати більше за потреби:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Чудово, продовжуємо з обробкою переліку інструментів.

### -3- Обробка переліку інструментів

Щоб обробляти запити на перелік інструментів, потрібно налаштувати відповідний обробник запитів. Ось що треба додати у файл сервера:

**Python**

```python
# код опущено для стислості
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

Тут додаємо декоратор `@server.list_tools` і функцію `handle_list_tools`. В останній потрібно сформувати список інструментів. Зверніть увагу, що кожен інструмент має мати name, description і inputSchema.

**TypeScript**

Щоб налаштувати обробник запитів для переліку інструментів, викликаємо `setRequestHandler` на сервері із схемою, що відповідає нашій задачі, у цьому випадку з `ListToolsRequestSchema`.

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
// код опущено заради стислості
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Повернути список зареєстрованих інструментів
  return {
    tools: tools
  };
});
```

Чудово, ми розв’язали питання переліку інструментів, тепер подивимося, як обробляти виклики інструментів.

### -4- Обробка виклику інструмента

Щоб викликати інструмент, потрібно додати ще один обробник запитів, що працює з запитом, де вказано, яку функцію викликати і з якими аргументами.

**Python**

Використаємо декоратор `@server.call_tool` і реалізуємо функцію `handle_call_tool`. Всередині потрібно розпарсити ім’я інструмента, його аргументи і переконатися, що аргументи підходять для цього інструмента. Валідацію можна виконувати тут або у самому інструменті.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools є словником з назвами інструментів у якості ключів
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # викликати інструмент
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Ось як це працює:

- Ім’я інструмента уже є у вхідному параметрі `name`, а аргументи — у словнику `arguments`.
- Інструмент викликається так: `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Валідація аргументів відбувається у властивості `handler`, що посилається на функцію; у разі помилки вона викличе виключення.

Тепер ми повністю розуміємо, як перелічувати та викликати інструменти за допомогою низькорівневого сервера.

Дивіться [повний приклад](./code/README.md)

## Завдання

Розширте наданий вам код кількома інструментами, ресурсами і запитами, звернувши увагу, що вам потрібно лише додавати файли в каталог tools і ніде більше.

*Розв’язок не надається*

## Підсумок

У цьому розділі ми побачили, як працює підхід низькорівневого сервера та як він допомагає створити гарну архітектуру, на якій можна продовжувати будувати. Також ми поговорили про валідацію та показали, як працювати з бібліотеками валідації для створення схем перевірки вводу.

## Що далі

- Наступне: [Проста автентифікація](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу автоматичного перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоч ми прагнемо до точності, просимо враховувати, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критичної інформації рекомендується звертатися до професійного перекладу людиною. Ми не несемо відповідальність за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
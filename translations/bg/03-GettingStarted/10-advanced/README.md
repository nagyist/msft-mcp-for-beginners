# Разширено използване на сървъра

В MCP SDK има два различни типа сървъри, вашият обикновен сървър и ниско-нивото сървърно ниво. Обикновено използвате обикновения сървър, за да добавяте функции към него. В някои случаи обаче искате да разчитате на ниско-нивото сървърно ниво, като например:

- По-добра архитектура. Възможно е да се създаде чиста архитектура с обикновения сървър и ниско-нивото сървърно ниво, но може да се твърди, че е малко по-лесно с ниско-нивото сървърно ниво.
- Наличност на функции. Някои разширени функции могат да се използват само с ниско-нивото сървърно ниво. Това ще видите в по-късните глави, когато добавяме семплиране и извличане.

## Обикновен сървър срещу ниско-нивото сървърно ниво

Ето как изглежда създаването на MCP сървър с обикновения сървър

**Python**

```python
mcp = FastMCP("Demo")

# Добавете инструмент за събиране
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

// Добавете инструмент за събиране
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

Същината е, че изрично добавяте всеки инструмент, ресурс или подкана, който искате сървърът да има. Няма нищо лошо в това.

### Подход с ниско-нивото сървърно ниво

Обаче когато използвате подхода с ниско-нивото сървърно ниво, трябва да мислите по различен начин, а именно вместо да регистрирате всеки инструмент, вие създавате два обработващи метода за всеки тип функция (инструменти, ресурси или подкани). Например инструментите имат само две функции, както следва:

- Изброяване на всички инструменти. Една функция отговаря за всички опити да се изброят инструментите.
- Обработване на извиквания към всички инструменти. Тук също има само една функция, която обработва повикванията към инструмент.

Звучи като потенциално по-малко работа, нали? Така вместо да регистрирам инструмент, просто трябва да се уверя, че инструментът е изброен, когато изброявам всички инструменти, и че се извиква, когато има входяща заявка за извикване на инструмент.

Нека разгледаме как изглежда кода сега:

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
  // Върнете списъка с регистрирани инструменти
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

Тук вече имаме функция, която връща списък с функции. Всеки запис в списъка с инструменти има полета като `name`, `description` и `inputSchema`, за да отговаря на типа на връщане. Това ни позволява да поставим нашите инструменти и дефиниции на функции на друго място. Сега можем да създадем всички наши инструменти в папка tools и същото важи за всичките ви функции, така че проектът ви внезапно може да бъде организиран така:

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

Това е страхотно, нашата архитектура може да изглежда доста чиста.

Какво по отношение на извикването на инструменти, идеята ли е същата тогава — един обработващ метод за извикване на инструмент, който и да е инструмент? Да, точно така, ето кода за това:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools е речник с имена на инструменти като ключове
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
    
    // аргументи: request.params.arguments
    // TODO извикайте инструмента,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Както виждате от горния код, трябва да разпарсим инструмента, който да извикаме, и с какви аргументи, и след това да продължим към извикване на инструмента.

## Подобряване на подхода с валидиране

Досега видяхте как всички регистрации за добавяне на инструменти, ресурси и подкани могат да се заменят с тези два обработващи метода за всеки тип функция. Какво още трябва да направим? Трябва да добавим някаква форма на валидиране, за да сме сигурни, че инструментът се извиква с правилните аргументи. Всеки runtime има собствено решение за това, например Python използва Pydantic, а TypeScript използва Zod. Идеята е следната:

- Преместваме логиката за създаване на функция (инструмент, ресурс или подкана) в нейна собствена папка.
- Добавяме начин за валидиране на входяща заявка, която например иска да извика инструмент.

### Създаване на функция

За да създадем функция, ще трябва да създадем файл за тази функция и да се уверим, че има задължителните полета, изисквани за тази функция. Кои полета са различни в малка степен между инструментите, ресурсите и подкани.

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
        # Валидирайте входните данни с помощта на Pydantic модел
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: добавете Pydantic, за да можем да създадем AddInputModel и да валидираме аргументите

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Тук можете да видите как правим следното:

- Създаваме схема с Pydantic `AddInputModel` с полета `a` и `b` във файл *schema.py*.
- Опитваме се да разпарсим входящата заявка като тип `AddInputModel`, ако има несъответствие в параметрите, това ще доведе до срив:

   ```python
   # add.py
    try:
        # Валидирайте въвеждането с използване на Pydantic модел
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Можете да изберете дали да сложите тази логика за парсиране в самото извикване на инструмента или в обработващата функция.

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

- В обработващата функция, която се занимава с всички извиквания на инструменти, сега се опитваме да разпарсим входящата заявка в дефинираната от инструмента схема:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ако това работи, тогава продължаваме към извикване на самия инструмент:

    ```typescript
    const result = await tool.callback(input);
    ```

Както виждате, този подход създава отлична архитектура, тъй като всичко си има място — *server.ts* е много малък файл, който само свързва обработващите заявки, а всяка функция е в съответната папка, т.е. tools/, resources/ или prompts/.

Отлично, нека се опитаме да изградим това следващо.

## Упражнение: Създаване на ниско-ниво сървър

В това упражнение ще направим следното:

1. Ще създадем ниско-ниво сървър, който обработва изброяването на инструменти и извикването на инструменти.
1. Ще приложим архитектура, върху която можете да надграждате.
1. Ще добавим валидиране, за да сме сигурни, че извикванията на вашите инструменти са правилно валидирани.

### -1- Създаване на архитектура

Първото нещо, което трябва да решим, е архитектура, която ни помага да мащабираме, докато добавяме повече функции, ето как изглежда:

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

Сега имаме изградена архитектура, която гарантира, че лесно можем да добавяме нови инструменти в папка tools. Можете да добавите подобни подпапки за resources и prompts.

### -2- Създаване на инструмент

Нека видим как изглежда създаването на инструмент. Първо, той трябва да бъде създаден в своята поддиректория *tool* така:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Валидирайте входните данни с помощта на Pydantic модел
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: добавете Pydantic, за да можем да създадем AddInputModel и да валидираме аргументите

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Тук виждаме как дефинираме име, описание, входна схема с Pydantic и обработчик, който ще се извика, когато този инструмент бъде повикан. Накрая излагаме `tool_add`, което е речник, съдържащ тези свойства.

Има и *schema.py*, използван за дефиниране на входната схема, използвана от нашия инструмент:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Трябва също така да попълним *__init__.py*, за да гарантираме, че директорията tools се третира като модул. Освен това трябва да изложим модулите вътре в нея, както следва:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Можем да продължаваме да добавяме към този файл, докато добавяме повече инструменти.

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

Тук създаваме речник, състоящ се от свойства:

- name, това е името на инструмента.
- rawSchema, това е Zod схемата, която ще се използва за валидиране на входящи заявки за извикване на този инструмент.
- inputSchema, тази схема се използва от обработващия метод.
- callback, това се използва за извикване на инструмента.

Има и `Tool`, който се използва за преобразуване на този речник в тип, който обработващият метод на mcp сървъра може да приеме, и изглежда така:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

И има *schema.ts*, където съхраняваме схемите за вход за всеки инструмент. Понастоящем има само една схема, но с добавяне на инструменти можем да добавяме още записи:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Отлично, нека преминем към обработката на изброяването на инструментите.

### -3- Обработване на изброяване на инструменти

Следва, за да обработим изброяването на инструментите, трябва да настроим обработващ метод за заявка за това. Ето какво трябва да добавим във файла на сървъра:

**Python**

```python
# кодът е пропуснат за краткост
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

Тук добавяме декоратора `@server.list_tools` и реализиращата функция `handle_list_tools`. В последната трябва да създадем списък с инструменти. Обърнете внимание, че всеки инструмент трябва да има име, описание и inputSchema.

**TypeScript**

За да настроим обработващия метод за заявката за изброяване на инструменти, трябва да извикаме `setRequestHandler` на сървъра със схема, която отговаря на това, което искаме да направим - в този случай `ListToolsRequestSchema`.

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
// кодът е пропуснат за краткост
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Върнете списъка с регистрирани инструменти
  return {
    tools: tools
  };
});
```

Страхотно, сега сме решили частта с изброяването на инструменти, нека разгледаме как можем да извикваме инструменти.

### -4- Обработване на извикване на инструмент

За да извикаме инструмент, трябва да настроим друг обработващ метод за заявка, този път съсредоточен върху заявка, която указва коя функция да се извика и с какви аргументи.

**Python**

Нека използваме декоратора `@server.call_tool` и го реализираме с функция като `handle_call_tool`. В тази функция трябва да разпарсим името на инструмента, неговите аргументи и да гарантираме, че аргументите са валидни за конкретния инструмент. Можем да валидираме аргументите тук или по-надолу в самия инструмент.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools е речник с имена на инструменти като ключове
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # извикайте инструмента
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Ето какво се случва:

- Името на инструмента вече е налично като входен параметър `name`, което важи и за нашите аргументи като речник `arguments`.

- Инструментът се извиква с `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Валидирането на аргументите става в свойството `handler`, което сочи към функция, ако това се провали, ще бъде хвърлена грешка.

Ето, вече имаме пълно разбиране за изброяването и извикването на инструменти с помощта на ниско-ниво сървър.

Вижте [пълния пример](./code/README.md) тук

## Задача

Разширете дадения ви код с няколко инструмента, ресурси и подкани и наблюдавайте как забелязвате, че трябва да добавяте файлове само в директорията tools и никъде другаде.

*Решение не е предоставено*

## Резюме

В тази глава видяхме как работи подходът с ниско-нивото сървърно ниво и как това може да ни помогне да създадем добра архитектура, върху която можем да надграждаме. Обсъдихме и валидирането и ви беше показано как да работите с библиотеки за валидиране, за да създавате схеми за проверка на входните данни.

## Какво следва

- Следва: [Проста автентикация](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от отговорност**:  
Този документ е преведен с помощта на автоматична услуга за превод [Co-op Translator](https://github.com/Azure/co-op-translator). Въпреки че се стремим към точност, имайте предвид, че автоматичните преводи могат да съдържат грешки или неточности. Оригиналният документ на неговия език трябва да се счита за авторитетен източник. За критична информация се препоръчва професионален превод от човек. Не носим отговорност за разбирателни или тълкувателни грешки, произтичащи от използването на този превод.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
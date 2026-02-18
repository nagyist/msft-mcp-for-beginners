# Напредна употреба сервера

У MCP SDK-у постоје два различита типа сервера, ваш уобичајени сервер и ниско-ниво сервер. Обично бисте користили регуларни сервер да бисте му додали функционалности. Међутим, у неким случајевима желите да се ослоните на ниско-ниво сервер као што су:

- Боља архитектура. Могуће је креирати чисту архитектуру и са регуларним и са ниско-ниво сервером, али може се тврдити да је нешто лакше са ниско-ниво сервером.
- Доступност функција. Неке напредне функције могу се користити само уз ниско-ниво сервер. Видећете ово у каснијим поглављима када будемо додавали семплирање и еликитацију.

## Регуларни сервер вс ниско-ниво сервер

Ево како изгледа креирање MCP сервера са регуларним сервером

**Python**

```python
mcp = FastMCP("Demo")

# Додајте алат за сабирање
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

// Додај алатку за сабирање
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

Суштина је да експлицитно додате сваки алат, ресурс или упит који желите да сервер има. Нема ништа лоше у томе.

### Приступ ниско-ниво сервера

Међутим, када користите приступ ниско-ниво сервера, потребно је да размишљате другачије, наиме уместо да региструјете сваки алат, ви уместо тога креирате два обрађивача по типу функције (алати, ресурси или упити). На пример, алати ће онда имати само две функције као што су:

- Листинг свих алата. Једна функција би била одговорна за све покушаје да се алати прикажу.
- Обрада позива свих алата. Такође, постоји само једна функција која обрађује позиве алата.

То звучи као потенцијално мање посла, зар не? Дакле, уместо да региструјем алат, само морам да се уверим да је алат на листи када набрајам све алате и да се позове када дође захтев за позив алата.

Хајде да видимо како сада изгледа код:

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
  // Врати листу регистрованих алата
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

Овде сада имамо функцију која враћа листу функција. Свa ставка у листи алата сада има поља као што су `name`, `description` и `inputSchema` да одговара повратном типу. Ово нам омогућава да дефиницију наших алата и функција ставимо на друго место. Сада можемо креирати све алате у фасцикли tools, и исто важи за све ваше функције па ваш пројекат може одједном бити организован овако:

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

Одлично, наша архитектура може бити прилично чиста.

А шта је са позивом алата, да ли је иста идеја, један обрађивач да позове алат, било који алат? Да, управо тако, ево кода за то:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools је речник са именима алата као кључевима
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
    // TODO позвати алатку,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Као што видите из горе наведеног кода, потребно је да издвојимо који алат се позива, са којим аргументима и онда да наставимо са позивом алата.

## Побољшање приступа валидацијом

До сада сте видели како све ваше регистрације за додавање алата, ресурса и упита могу да се замене са ова два обрађивача по типу функције. Шта још треба да урадимо? Па, требало би да додамо неки облик валидације да бисмо осигурали да се алат позива са исправним аргументима. Сваки ратним има своје решење за ово, на пример Python користи Pydantic, а TypeScript користи Zod. Идеја је да урадимо следеће:

- Преселимо логику креирања функције (алат, ресурс или упит) у њен посебан фолдер.
- Додамо начин да валидирамо долазни захтев за позив алата, на пример.

### Креирање функције

Да бисмо креирали функцију, потребно је да креирамо фајл за ту функцију и да обезбедимо да има обавезна поља која су захтевана за ту функцију. Која поља се разликују између алата, ресурса и упита.

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
        # Валидација уноса коришћењем Пидантик модела
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: додај Пидантик, да бисмо могли да направимо AddInputModel и верификујемо аргументе

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

овде можете видети како урадимо следеће:

- Креирамо шему користећи Pydantic `AddInputModel` са пољима `a` и `b` у фајлу *schema.py*.
- Покушавамо да парсирамо долазни захтев да буде типа `AddInputModel`, ако се параметри не поклапају, ово ће избацити грешку:

   ```python
   # add.py
    try:
        # Валидација уноса користећи Пидантик модел
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Можете одабрати да ли ову логику парсирања ставите у позив алата или у функцију обрађивача.

**TypeScript**

```typescript
// сервер.ts
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

// шема.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// додај.ts
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

- У обрађивачу који обрађује све позиве алата, сада покушавамо да парсирамо долазни захтев у дефинисану шему алата:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ако то успе, настављамо да зовемо сам алат:

    ```typescript
    const result = await tool.callback(input);
    ```

Као што видите, овај приступ ствара добру архитектуру јер све има своје место, *server.ts* је врло мали фајл који само повезује обрађиваче позива, а свака функција је у свом одговарајућем фолдеру, нпр. tools/, resources/ или /prompts.

Одлично, хајде да сада то изградимо.

## Вежба: Креирање ниско-ниво сервера

У овој вежби ћемо урадити следеће:

1. Креирати ниско-ниво сервер који ће обрађивати листање алата и позив алата.
1. Имплементирати архитектуру на којој можете градити.
1. Додати валидацију да бисте осигурали да су позиви алата правилно валидациони.

### -1- Креирање архитектуре

Прва ствар коју треба да урадимо је архитектура која нам помаже да скалирамо док додајемо више функција, ево како изгледа:

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

Сада имамо постављену архитектуру која осигурава да можемо лако додавати нове алате у фолдер tools. Слободно пратите ово да бисте додали поддиректоријуме за ресурсе и упите.

### -2- Креирање алата

Хајде да видимо како изгледа креирање алата. Прво, мора да буде креиран у свом поддиректоријуму *tool* као овде:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Валидација уноса коришћењем Пајдантик модела
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # ЗАДАЦИ: додати Пајдантик, тако да можемо направити AddInputModel и валидацију аргумената

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Видимо овде како дефинишемо име, опис, уносну шему користећи Pydantic и обрађивач који ће се позвати када се овај алат позове. На крају излажемо `tool_add` који је речник са свим тим својствима.

Постоји и *schema.py* који служи за дефинисање уносне шеме коју користи наш алат:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Такође морамо попунити *__init__.py* да би се tools директоријум третирао као модул. Поред тога потребно је да излагамо модуле унутар њега овако:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Можемо наставити да додајемо овом фајлу како додајемо више алата.

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

Овде креирамо речник који се састоји од својстава:

- name, ово је име алата.
- rawSchema, ово је Zod шема која ће се користити за валидацију долазних захтева за позив овог алата.
- inputSchema, ове шеме ће користити обрађивач.
- callback, ово се користи за позив алата.

Постоји и `Tool` који се користи да конвертује овај речник у тип који MCP сервер обрађивач може да прихвати и изгледа овако:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Постоји и *schema.ts* где чувамо уносне шеме за сваки алат, тренутно са само једном шемом али како додамо алате можемо додавати и нове ставке:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Одлично, наставимо са обрадом листања алата.

### -3- Обрада листања алата

Затим, да бисмо обрадили листање наших алата, потребно је да подесимо обрађивач захтева за то. Ево шта треба да додамо у нашем серверу:

**Python**

```python
# код изостављен ради краткоће
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

Овде додавамо декоратор `@server.list_tools` и имплементирамо функцију `handle_list_tools`. У овој функцији треба да произведемо листу алата. Обратите пажњу како сваки алат мора да има име, опис и inputSchema.

**TypeScript**

Да бисте подесили обрађивач захтева за листање алата, треба да позовете `setRequestHandler` на серверу са шемом која одговара оно што радите, у овом случају `ListToolsRequestSchema`.

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
// код изостављен ради краткоће
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Врати листу регистрованих алата
  return {
    tools: tools
  };
});
```

Сјајно, сада смо решили део листања алата, хајде да видимо како се позивају алати.

### -4- Обрада позива алата

Да бисте позвали алат, потребно је да подесите још један обрађивач захтева, овог пута фокусиран на захтев који специфично одређује коју функцију позвати и са којим аргументима.

**Python**

Користимо декоратор `@server.call_tool` и имплементирамо га функцијом `handle_call_tool`. У тој функцији треба да издвојимо име алата, његов аргумент и осигурамо да су аргументи валидни за тај алат. Валидацију аргумената можемо урадити или у овој функцији или у самом алату.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools је речник са именима алата као кључевима
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # позови алат
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Ево шта се дешава:

- Име нашег алата је већ присутно као улазни параметар `name`, што важи и за наше аргументе у облику речника `arguments`.

- Алат се позива са `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Валидација аргумената се дешава у својству `handler` које указује на функцију, ако то не успе изазваће изузетак.

Ето, сада имамо пуно разумевање листања и позивања алата помоћу ниско-ниво сервера.

Погледајте [целокупан пример](./code/README.md) овде

## Задатак

Проширите код који сте добили са бројем алата, ресурса и упита и размислите како примећујете да треба да додате само фајлове у директоријум tools и нигде друго.

*Решење није дато*

## Резиме

У овом поглављу смо видели како приступ ниско-ниво сервера ради и како нам то може помоћи у креирању лепе архитектуре на којој можемо наставити да градимо. Такође смо разговарали о валидацији и показано вам је како да радите са библиотекама за валидацију да бисте креирали шеме за верификацију улаза.

## Следеће

- Следеће: [Једноставна аутентификација](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Одрицање од одговорности**:
Овај документ је преведен коришћењем вештачке интелигенције преко услуге за превођење [Co-op Translator](https://github.com/Azure/co-op-translator). Иако настојимо да превод буде што прецизнији, молимо имајте у виду да аутоматизовани преводи могу да садрже грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитетом. За критичне информације препоручује се професионални превод од стране стручног човека. Нисмо одговорни за било каква неспоразуме или погрешна тумачења која могу настати употребом овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
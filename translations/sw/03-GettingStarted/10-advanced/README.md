# Matumizi ya hali ya juu ya seva

Kuna aina mbili tofauti za seva zinazotolewa katika MCP SDK, seva yako ya kawaida na seva ya kiwango cha chini. Kawaida, ungetumia seva ya kawaida kuongeza vipengele ndani yake. Lakini kwa baadhi ya hali, unataka kutegemea seva ya kiwango cha chini kama vile:

- Muundo bora. Inawezekana kuunda muundo safi na seva ya kawaida pamoja na seva ya kiwango cha chini lakini inaweza kudaiwa kuwa ni rahisi kidogo na seva ya kiwango cha chini.
- Upatikanaji wa kipengele. Baadhi ya vipengele vya hali ya juu vinaweza kutumiwa tu na seva ya kiwango cha chini. Utayaona haya katika sura za baadaye tunapoongeza sampuli na elicitation.

## Seva ya kawaida dhidi ya seva ya kiwango cha chini

Hivi ndivyo uundaji wa MCP Server unavyoonekana na seva ya kawaida

**Python**

```python
mcp = FastMCP("Demo")

# Ongeza zana ya kuongeza
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

// Ongeza chombo cha kuongeza
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

Nukuu ni kwamba unaongeza kila chombo, rasilimali au onyo (prompt) unayotaka seva iwe nayo kwa wazi. Hakuna tatizo na hilo.

### Njia ya seva ya kiwango cha chini

Hata hivyo, unapochukua njia ya seva ya kiwango cha chini unahitaji kufikiria tofauti yaani badala ya kusajili kila chombo unaunda wasimamizi wawili kwa kila aina ya kipengele (vifaa, rasilimali au onyo). Kwa mfano, vifaa vina kazi mbili tu kama ifuatavyo:

- Orodhesha vifaa vyote. Kazi moja itahusika na kila jaribio la kuorodhesha vifaa.
- Shughulikia kupiga simu kwa vifaa vyote. Pia hapa, kuna kazi moja inayoshughulikia simu kwa kifaa.

Hiyo inaonekana kama kazi ndogo sivyo? Kwa hiyo badala ya kusajili kifaa, ninahitaji tu kuhakikisha kifaa kimeorodheshwa ninapoorodhesha vifaa vyote na kinapoitwa wakati kuna ombi linakuja la kuitisha kifaa.

Hebu tuangalie basi kifaa kinavyoonekana sasa:

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
  // Rejesha orodha ya zana zilizosajiliwa
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

Hapa sasa tuna kazi inayorudisha orodha ya vipengele. Kila kipengele kwenye orodha ya vifaa kina sehemu kama `name`, `description` na `inputSchema` kutii aina iliyorejeshwa. Hii inatuwezesha kuweka vifaa na ufafanuzi wa kipengele mahali pengine. Sasa tunaweza kuunda vifaa vyote katika folda ya vifaa na vivyo hivyo kwa vipengele vyote hivyo mradi wako unaweza kuandaliwa kama ifuatavyo:

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

Hiyo ni nzuri, muundo wetu unaweza kufanywa kuwa safi sana.

Je, kuhusu kuitisha vifaa, ni wazo lile lile? msimamizi mmoja wa kuitisha kifaa, kifaa chochote? Ndiyo, hasa, hapa ni msimbo wa hilo:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ni kamusi yenye majina ya zana kama funguo
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
    
    // hoja: request.params.arguments
    // KUFANYA kiraka chombo,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Kama unaona katika msimbo hapo juu, tunahitaji kugawanya kifaa cha kuitisha, na na hoja gani, kisha tunaendelea kuitisha kifaa.

## Kuboresha njia hii kwa uthibitishaji

Hadi sasa, umeona jinsi usajili wako wote wa kuongeza vifaa, rasilimali na onyo unavyoweza kubadilishwa na wasimamizi hawa wawili kwa kila aina ya kipengele. Je, tunachohitaji zaidi? Naam, tunapaswa kuongeza aina fulani ya uthibitishaji kuhakikisha kifaa kinaitwa na hoja sahihi. Kila runtime ina suluhisho lake kwa hili, kwa mfano Python hutumia Pydantic na TypeScript hutumia Zod. Wazo ni kufanya yafuatayo:

- Hamisha mantiki ya kuunda kipengele (kifaa, rasilimali au onyo) hadi folda yake maalum.
- Ongeza njia ya kuthibitisha ombi linalokuja kama mfano la kuitisha kifaa.

### Tengeneza kipengele

Kutama kipengele, tutahitaji kuunda faili kwa kipengele hicho na kuhakikisha kina sehemu za lazima zinazohitajika kwa kipengele hicho. Sehemu hizo hubadilika kidogo kati ya vifaa, rasilimali na onyo.

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
        # Thibitisha ingizo kwa kutumia mfano wa Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: ongeza Pydantic, ili tuweze kuunda AddInputModel na kuthibitisha vigezo

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

hapa unaweza kuona tunavyofanya yafuatayo:

- Tengeneza skima kwa kutumia Pydantic `AddInputModel` na sehemu `a` na `b` katika faili *schema.py*.
- Jaribu kuchambua ombi linalokuja kuwa ya aina `AddInputModel`, ikiwa kuna mkitano usio sawa wa vigezo hii itasababisha kuanguka:

   ```python
   # add.py
    try:
        # Thibitisha ingizo kwa kutumia mfano wa Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Unaweza kuchagua kuweka mantiki hii ya kuchambua katika wito wa kifaa yenyewe au katika kazi ya msimamizi.

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

- Katika msimamizi anayeshughulikia wito wote wa kifaa, sasa tunajaribu kuchambua ombi linalokuja katika skima iliyobainishwa ya kifaa:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ikiwa hiyo itafanya kazi basi tunaendelea kuitisha kifaa halisi:

    ```typescript
    const result = await tool.callback(input);
    ```

Kama unavyoona, njia hii huunda muundo mzuri kwani kila kitu kina mahali pake, *server.ts* ni faili ndogo sana inayotengeneza tu wasimamizi wa maombi na kila kipengele kiko katika folda zake husika yaani tools/, resources/ au /prompts.

Nzuri, hebu tujipange kuunda hii ifuatayo.

## Zoefu: Kuunda seva ya kiwango cha chini

Katika zoefu hili, tutafanya yafuatayo:

1. Tengeneza seva ya kiwango cha chini inayoshughulikia kuorodhesha vifaa na kuitisha vifaa.
2. Tekeleza muundo unaweza kujenga juu yake.
3. Ongeza uthibitishaji kuhakikisha miito yako ya kifaa inathibitishwa ipasavyo.

### -1- Tengeneza muundo

Jambo la kwanza la kushughulikia ni muundo unaotusaidia kupanua tunapoongeza vipengele zaidi, hivi ndivyo inaonekana:

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

Sasa tumesanidi muundo unaohakikisha tunaweza kuongeza vifaa vipya kwa urahisi katika folda ya tools. Hifadhi kutumia hii kuongeza saraka ndogo kwa resources na prompts.

### -2- Kuunda kifaa

Tuchunguze jinsi kuunda kifaa kinavyoonekana baadae. Kwanza, kinahitaji kuundwa katika saraka yake ndogo ya *tool* kama ifuatavyo:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Thibitisha ingizo kwa kutumia mfano wa Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: ongeza Pydantic, ili tuweze kuunda AddInputModel na kuthibitisha hoja

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Tunavyoona hapa ni jinsi tunavyofafanua jina, maelezo, skima ya ingizo kwa kutumia Pydantic na msimamizi atakaetengwa mara kifaa hiki kitapigiwa simu. Mwisho, tunaonyesha `tool_add` ambayo ni kamusi inayoshikilia sifa hizi zote.

Pia kuna *schema.py* inayotumika kufafanua skima ya ingizo inayotumika na kifaa chetu:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Tunahitaji pia kujaza *__init__.py* kuhakikisha saraka ya tools inachukuliwa kuwa moduli. Zaidi ya hayo, tunahitaji kuonyesha moduli zilizo ndani yake kama ifuatavyo:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Tunaweza kuendelea kuongeza kwa faili hii tunapoongeza vifaa zaidi.

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

Hapa tunaunda kamusi yenye sifa:

- name, hili ni jina la kifaa.
- rawSchema, hii ni skima ya Zod, itatumika kuthibitisha maombi yanayoingia ya kuitisha kifaa hiki.
- inputSchema, skima hii itatumiwa na msimamizi.
- callback, hii hutumiwa kuitisha kifaa.

Kuna pia `Tool` inayotumika kubadilisha kamusi hii kuwa aina inayoweza kukubaliwa na msimamizi wa seva ya mcp na inaonekana kama ifuatavyo:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Na kuna *schema.ts* ambapo tunahifadhi skima za ingizo kwa kila kifaa ambazo zinaonekana hivi kwa sasa na skima moja pekee lakini tunapoongeza vifaa tunaweza kuongeza maingizo zaidi:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Nzuri, twende tuendelee kushughulikia kuorodhesha vifaa vyetu ifuatayo.

### -3- Shughulikia orodha ya vifaa

Ifuatayo, kushughulikia orodha ya vifaa vyetu, tunahitaji kuweka msimamizi wa ombi kwa hilo. Hivi ndivyo tunavyopaswa kuongeza kwenye faili yetu ya seva:

**Python**

```python
# msimbo umeachwa kwa ufupi
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

Hapa tunaongeza dekoreta `@server.list_tools` na kazi inayotekeleza `handle_list_tools`. Katika hii ya mwisho, tunapaswa kuzalisha orodha ya vifaa. Angalia kila kifaa kinapaswa kuwa na jina, maelezo na inputSchema.

**TypeScript**

Ili kuweka msimamizi wa ombi kwa kuorodhesha vifaa, tunahitaji kuita `setRequestHandler` kwenye seva na skima inayofaa kwa kile tunachojaribu kufanya, katika kesi hii `ListToolsRequestSchema`.

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
// msimbo umekatwa kwa ufupi
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Rudisha orodha ya zana zilizosajiliwa
  return {
    tools: tools
  };
});
```

Nzuri, sasa tumetatua sehemu ya kuorodhesha vifaa, hebu tuangalie jinsi tunaweza kuitisha vifaa ifuatayo.

### -4- Shughulikia kuitisha kifaa

Kuitisha kifaa, tunahitaji kuweka msimamizi mwingine wa ombi, mara hii ikilenga kushughulikia ombi linalobainisha ni kipengele gani kitaitwa na na hoja gani.

**Python**

Tuchukue dekoreta `@server.call_tool` na tuiite katika kazi kama `handle_call_tool`. Ndani ya kazi hii, tunahitaji kuchambua jina la kifaa, hoja zake na kuhakikisha hoja ni sahihi kwa kifaa husika. Tunaweza kuthibitisha hoja katika kazi hii au chini huko kwenye kifaa halisi.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # zana ni kamusi yenye majina ya zana kama funguo
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # itumie zana hiyo
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Haya ndivyo yanavyotokea:

- Jina la kifaa chetu tayari lipo kama kipengele cha ingizo `name` ambacho ndivyo kwa hoja zetu katika kamusi ya `arguments`.

- Kifaa kinaitwa kwa `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Uthibitishaji wa hoja hutokea kwenye eneo la `handler` linaloelekeza kwa kazi, kama haifanyi kazi itatoa hitilafu.

Hapo sasa tumeelewa kabisa kuorodhesha na kuitisha vifaa kwa kutumia seva ya kiwango cha chini.

Tazama [mfano kamili](./code/README.md) hapa

## Kazi

Panua msimbo uliotolewa kwa idadi ya vifaa, rasilimali na onyo na tafakari jinsi unavyoona kuwa unahitaji tu kuongeza faili katika saraka ya tools na si mahali pengine.

*Hakuna suluhisho lililotolewa*

## Muhtasari

Katika sura hii, tuliona jinsi njia ya seva ya kiwango cha chini ilivyofanya kazi na jinsi inavyotusaidia kuunda muundo mzuri tunaweza kuendelea kuujenga. Pia tulijadili uthibitishaji na ulionyeshwa jinsi ya kutumia maktaba za uthibitishaji kuunda skima kwa uthibitishaji wa ingizo.

## Nini Kifuatacho

- Ifuatayo: [Uthibitishaji Rahisi](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kiarifu cha Msamaha**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kuwa sahihi, tafadhali fahamu kuwa tafsiri za kiotomatiki zinaweza kuwa na makosa au upotoshaji. Hati asili katika lugha yake ya asili inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu ya binadamu inapendekezwa. Hatubebeki dhamana kwa kutoelewana au upotoshaji unaotokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
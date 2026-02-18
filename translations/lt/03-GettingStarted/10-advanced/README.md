# Išplėstinis serverio naudojimas

MCP SDK yra du skirtingi serverių tipai, jūsų įprastasis serveris ir žemo lygio serveris. Paprastai naudotumėte įprastą serverį, kad jame pridėtumėte funkcijų. Tačiau kai kuriais atvejais norite pasikliauti žemo lygio serveriu, pavyzdžiui:

- Geresnė architektūra. Galima sukurti tvarkingą architektūrą tiek naudojant įprastą serverį, tiek žemo lygio serverį, bet galima teigti, kad žemo lygio serveriu tai padaryti šiek tiek lengviau.
- Funkcijų prieinamumas. Kai kurios pažangios funkcijos gali būti naudojamos tik su žemo lygio serveriu. Tai pamatysite vėlesniuose skyriuose, kai pridėsime mėginius ir iššaukimo mechanizmus.

## Įprastinis serveris prieš žemo lygio serverį

Štai kaip atrodo MCP serverio kūrimas naudojant įprastinį serverį

**Python**

```python
mcp = FastMCP("Demo")

# Pridėti papildymo įrankį
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

// Pridėti pridėjimo įrankį
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

Svarbiausia, kad jūs aiškiai pridedate kiekvieną įrankį, išteklių ar iššaukimo tekstą, kurį norite, kad serveris turėtų. Tai nėra klaida.  

### Žemo lygio serverio metodas

Tačiau naudojant žemo lygio serverio metodą reikia galvoti kitaip, t. y., vietoj kiekvieno įrankio registravimo jūs sukuriate du apdorotojus kiekvienam funkcijos tipui (įrankiai, ištekliai arba iššaukimo tekstai). Pavyzdžiui, įrankiams yra tik dvi funkcijos:

- Visų įrankių išvardijimas. Viena funkcija atsakinga už visas pastangas išvardyti įrankius.
- Visi įrankių kvietimai. Čia taip pat yra viena funkcija, apdorojanti kvietimus į bet kurį įrankį.

Skamba kaip mažiau darbo, tiesa? Taigi, vietoj įrankio registravimo, man reikia tik įsitikinti, kad įrankis yra išvardytas, kai išvardiju visus įrankius, ir kad jis kviečiamas, kai yra įeinantis užklausimas įrankio kvietimui. 

Pažiūrėkime, kaip dabar atrodo kodas:

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
  // Grąžina registruotų įrankių sąrašą
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

Čia turime funkciją, kuri grąžina funkcijų sąrašą. Kiekvienas įrankių sąrašo įrašas turi laukus kaip `name`, `description` ir `inputSchema`, atitinkančius grąžinimo tipą. Tai leidžia mums perkelti įrankių ir funkcijų apibrėžimus kitur. Galime dabar kurti visus įrankius į *tools* aplanką ir tas pats galioja visoms jūsų funkcijoms, tad jūsų projektas gali būti organizuotas taip:

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

Puiku, mūsų architektūra gali atrodyti labai tvarkingai.

O kaip su įrankių kvietimu – ar tai ta pati idėja, vienas apdorotojas kviečia bet kurį įrankį? Taip, tiksliai, štai tas kodas:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools yra žodynas, kuriame raktai yra įrankių pavadinimai
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
    // TODO iškviesti įrankį,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Kaip matote iš aukščiau pateikto kodo, reikia išanalizuoti, kurį įrankį kviečiame ir su kokiais argumentais, tada tęsti įrankio kvietimą.

## Metodo gerinimas su validacija

Iki šiol matėte, kaip visas įrankių, išteklių ir iššaukimo registracijas galima pakeisti šiais dviem apdorotojais kiekvienam funkcijos tipui. Ką dar turėtume daryti? Reikėtų pridėti tam tikrą validacijos būdą, kad užtikrintume, jog įrankis kviečiamas su teisingais argumentais. Kiekviena vykdymo aplinka turi savo sprendimą: Python naudoja Pydantic, o TypeScript – Zod. Idėja yra tokia:

- Perkelti logiką, kuri kuria funkciją (įrankį, išteklių arba iššaukimo tekstą), į savo dedikuotą aplanką.
- Pridėti būdą patikrinti įeinančią užklausą, pavyzdžiui kvietimą įrankiui.

### Kurti funkciją

Norint sukurti funkciją, turėsime sukurti failą šiai funkcijai ir užtikrinti, kad jame būtų privalomi laukai, reikalingi tai funkcijai. Šie laukai šiek tiek skiriasi tarp įrankių, išteklių ir iššaukimo tekstų.

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
        # Patikrinkite įvestį naudodami Pydantic modelį
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: pridėti Pydantic, kad galėtume sukurti AddInputModel ir patikrinti argumentus

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Čia matote, kaip darome šiuos veiksmus:

- Sukuriame schemą naudodami Pydantic `AddInputModel` su laukais `a` ir `b` faile *schema.py*.
- Bandoma išanalizuoti įeinančią užklausą kaip tipą `AddInputModel`, jei yra neatitikimų argumentuose, programa sugrius:

   ```python
   # add.py
    try:
        # Patikrinti įvestį naudojant Pydantic modelį
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Galite pasirinkti, ar šią analizės logiką dėti į patį įrankio kvietimą, ar į apdorotojo funkciją.

**TypeScript**

```typescript
// serveris.ts
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

// pridėti.ts
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

- Apdorotoje, tvarkančioje visus įrankių kvietimus, dabar bandoma išanalizuoti įeinančią užklausą pagal įrankio apibrėžtą schemą:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    jei tai pavyksta, tęsiame faktinį įrankio kvietimą:

    ```typescript
    const result = await tool.callback(input);
    ```

Kaip matote, šis metodas sukuria puikią architektūrą, nes viskas yra savo vietoje, *server.ts* yra labai mažas failas, kuris tiesiog susieja užklausų apdorotojus, o kiekviena funkcija yra savo aplanke, pvz. tools/, resources/ arba prompts/.

Puiku, pradėkime tai kurti.

## Užduotis: Žemo lygio serverio kūrimas

Šioje užduotyje padarysime šiuos dalykus:

1. Sukursime žemo lygio serverį, kuris apdoros įrankių išvardijimą ir įrankių kvietimą.
1. Įgyvendinsime architektūrą, kuria galėsite plėtoti.
1. Pridėsime validaciją, kad jūsų įrankių kvietimai būtų tinkamai patikrinti.

### -1- Kurti architektūrą

Pirmiausia turime sukurt architektūrą, kuri padės mums praplėsti sistemą pridedant daugiau funkcijų, štai kaip ji atrodo:

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

Dabar turime architektūrą, kuri užtikrina, kad galime lengvai pridėti naujų įrankių į *tools* aplanką. Galite sekti šiuo pavyzdžiu ir pridėti aplankų ištekliams bei iššaukimo tekstams.

### -2- Kurti įrankį

Pažiūrėkime, kaip atrodo įrankio kūrimas. Pirmiausia jis turi būti sukurtas atitinkamame *tool* poskirsnyje:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Patvirtinkite įvestį naudodami Pydantic modelį
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: pridėti Pydantic, kad galėtume sukurti AddInputModel ir patvirtinti argumentus

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Čia matome, kaip aprašome pavadinimą, aprašymą, įvedimo schemą naudodami Pydantic ir apdorotoją, kuris bus iškviečiamas, kai šis įrankis bus kviečiamas. Galiausiai eksponuojame `tool_add`, kuris yra žodynas, laikantis šias savybes.

Taip pat yra *schema.py*, kuri naudojama apibrėžti įvedimo schemą mūsų įrankiui:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Taip pat turime užpildyti *__init__.py*, kad *tools* katalogas būtų laikomas moduliu. Be to, turime eksportuoti jame esančius modulius taip:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Šį failą galime toliau papildyti pridėdami daugiau įrankių.

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

Čia kuriame žodyną su savybėmis:

- name – įrankio pavadinimas.
- rawSchema – Zod schema, kuri bus naudojama patikrinti įeinančias užklausas įrankio kvietimui.
- inputSchema – šią schemą naudos apdorotojas.
- callback – funkcija, kuri kvies įrankį.

Yra taip pat `Tool`, kuris paverčia žodyną į tipą, kurį MCP serverio apdorotojas gali priimti, ir jis atrodo taip:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Yra *schema.ts*, kur saugomos įvedimo schemos kiekvienam įrankiui. Šiuo metu yra tik viena schema, bet pridedant įrankių, galėsime pridėti daugiau:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Puiku, dabar pereikime prie įrankių išvardijimo apdorojimo.

### -3- Tvarkyti įrankių išvardijimą

Dabar, kad aptarnautume įrankių išvardijimą, turime sukurti užklausų apdorotoją tam. Štai ką reikia pridėti prie serverio failo:

**Python**

```python
# kodas sutrumpintas dėl glaustumo
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

Čia pridedame dekoratorių `@server.list_tools` ir apdorojančią funkciją `handle_list_tools`. Pastaroje turime sugeneruoti įrankių sąrašą. Atkreipkite dėmesį, kad kiekvienas įrankis turi turėti pavadinimą, aprašymą ir inputSchema.   

**TypeScript**

Norėdami nustatyti užklausų apdorotoją įrankių išvardijimui, kviečiame `setRequestHandler` serveryje su schema, atitinkančia mūsų siekį, šiuo atveju `ListToolsRequestSchema`.

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
// kodas sutrumpintas
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Grąžinti registruotų įrankių sąrašą
  return {
    tools: tools
  };
});
```

Puiku, dabar išsprendėme įrankių išvardijimo dalį, pažiūrėkime, kaip galėtume kvieti įrankius.

### -4- Tvarkyti įrankio kvietimą

Norint iškviesti įrankį, reikia sukurti kitą užklausų apdorotoją, kuris apdoros užklausą, nurodančią, kuri funkcija turi būti kviečiama ir su kokiais argumentais.

**Python**

Naudosime dekoratorių `@server.call_tool` ir įgyvendinsime jį funkcija `handle_call_tool`. Šioje funkcijoje turime išanalizuoti įrankio pavadinimą, argumentus ir užtikrinti, kad argumentai yra teisingi tam įrankiui. Galime validuoti argumentus tiek čia, tiek žemiau pačiame įrankyje.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools yra žodynas, kurio raktai yra įrankių pavadinimai
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # iškvieskite įrankį
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Čia vyksta šie veiksmai:

- Įrankio pavadinimas jau pateiktas įvesties parametru `name`, o mūsų argumentai yra `arguments` žodyne.

- Įrankis kviečiamas su `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Argumentų validacija vyksta `handler` funkcijoje; jei nepavyksta, kyla išimtis. 

Štai ir viskas – dabar turime pilną supratimą apie įrankių išvardijimą ir kvietimą žemo lygio serverio pagalba.

Pilną pavyzdį rasite [čia](./code/README.md).

## Užduotis

Išplėskite pateiktą kodą pridėdami kelis įrankius, išteklių ir iššaukimo tekstų ir pastebėkite, kaip reikia tik pridėti failus *tools* kataloge, ir niekur kitur.

*Nėra pateikto sprendimo*

## Santrauka

Šiame skyriuje pamatėme, kaip veikia žemo lygio serverio metodas ir kaip jis gali padėti sukurti tvarkingą architektūrą, ant kurios galima tęsti kūrimą. Taip pat aptarėme validaciją ir parodėme, kaip dirbti su validavimo bibliotekomis, kad būtų sukurtos schemos įvedimo validacijai.

## Kas toliau

- Toliau: [Paprastas autentifikavimas](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors stengiamės užtikrinti tikslumą, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų arba netikslumų. Originalus dokumentas gimtąja kalba turėtų būti laikomas autoritetingu šaltiniu. Esant svarbiai informacijai rekomenduojama kreiptis į profesionalų žmogaus vertėją. Mes neprisiimame atsakomybės už bet kokius nesusipratimus ar neteisingus interpretavimus, kilusius dėl šio vertimo naudojimo.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
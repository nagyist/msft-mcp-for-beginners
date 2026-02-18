# Täiustatud serveri kasutamine

MCP SDK-s on kaks erinevat serveri tüüpi, tavaline server ja madala taseme server. Tavaliselt kasutaksite tavalist serverit funktsioonide lisamiseks. Mõnel juhul aga soovite tugineda madala taseme serverile, näiteks:

- Parem arhitektuur. Puhas arhitektuur on võimalik luua nii tavalise serveri kui ka madala taseme serveriga, kuid võib väita, et madala taseme serveriga on see veidi lihtsam.
- Funktsioonide saadavus. Mõnda täiustatud funktsiooni saab kasutada ainult madala taseme serveriga. Seda näete hilisemas peatükis, kui lisame võtmist ja küsitlemist.

## Tavaline server vs madala taseme server

Nii näeb välja MCP serveri loomine tavalise serveriga

**Python**

```python
mcp = FastMCP("Demo")

# Lisa liitmisseade
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

// Lisa liitmiste tööriist
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

Oluline on see, et te lisate selgesõnaliselt iga tööriista, ressursi või käsu, mida soovite serverile lisada. Sellega pole midagi valesti.

### Madala taseme serveri lähenemine

Kui aga kasutate madala taseme serveri lähenemist, peate mõtlema teisiti, nimelt selle asemel, et registreerida iga tööriist, loote iga funktsiooni tüübi (tööriistad, ressursid või käsud) jaoks kaks käsitlejat. Näiteks tööriistadel on siis ainult kaks funktsiooni:

- Kõigi tööriistade loetelu. Üks funktsioon vastutab kõigi tööriistade loendamise katsete eest.
- Kõigi tööriistade kutsumise käsitlemine. Siin on samuti ainult üks funktsioon, mis käsitleb tööriista kutsumisi.

See kõlab nagu vähem tööd, eks? Nii et tööriista registreerimise asemel pean lihtsalt tagama, et tööriist oleks loendis, kui loendan kõiki tööriistu, ning et see kutsutakse, kui saabub päring tööriista kutsumiseks.

Vaatame, kuidas kood nüüd välja näeb:

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
  // Tagastage registreeritud tööriistade nimekiri
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

Siin on meil nüüd funktsioon, mis tagastab omaduste nimekirja. Iga tööriista kirjelduse välja nagu `name`, `description` ja `inputSchema` on nüüd olemas tagastustüübi järgimiseks. See võimaldab meil panna oma tööriistad ja omaduste definitsioonid mujale. Nüüd saame kõik tööriistad luua tööriistade kaustas ning sama kehtib ka kõigi teie omaduste kohta, nii et teie projekt võib korraga olla selline:

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

See on suurepärane, meie arhitektuur saab üsna puhtaks.

Mis saab tööriistade kutsumisest, kas see on sama idee — üks käsitleja kutsumiseks, ükskõik milline tööriist? Jah, täpselt, siin on selle kood:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tööriistad on sõnastik, kus võtmeteks on tööriistade nimed
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
    // TODO kutsuda tööriist,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Nagu ülaltoodud koodist näete, peame tuvastama, millist tööriista kutsutakse ja milliste argumentidega ning seejärel peame jätkama tööriista kutsumisega.

## Lähenemise parendamine valideerimisega

Siiani olete näinud, kuidas kõik teie tööriistade, ressursside ja käskude registreerimised saab asendada nende kahe käsitlejaga iga funktsiooni tüübi kohta. Mida veel peame tegema? Peame lisama mingisuguse valideerimise, et tagada, et tööriist kutsutakse õigete argumentidega. Igal käivitamisel on selleks oma lahendus, näiteks Python kasutab Pydantic'ut ja TypeScript kasutab Zodi. Idee on järgmine:

- Viia funktsiooni (tööriist, ressurss või käsk) loomise loogika selle pühendatud kausta.
- Lisada viis valideerida sissetulev päring, mis palub näiteks tööriista kutsuda.

### Funktsiooni loomine

Funktsiooni loomiseks peame looma selle funktsiooni faili ja veenduma, et selles on selle funktsiooni jaoks vajalikud nõutavad väljad. Millised väljad on olenevalt tööriistadest, ressurssidest ja käskudest veidi erinevad.

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
        # Sisendi valideerimine Pydantic mudeli abil
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: lisa Pydantic, et saaksime luua AddInputModeli ja valideerida argumente

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Siin näete, kuidas teeme järgmist:

- Loome skeemi Pydanticu `AddInputModel` abil, milles on väljad `a` ja `b` failis *schema.py*.
- Püüame sissetuleva päringu analüüsida kui `AddInputModel` tüüpi; kui parameetrites on vastuolu, kukub see kokku:

   ```python
   # add.py
    try:
        # Sisendi valideerimine Pydantic mudeli abil
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Võite valida, kas panna see analüüsi loogika tööriista kutsesse või käsitleja funktsiooni.

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

- Käsitlejas, mis tegeleb kõigi tööriistakutsetega, proovime nüüd analüüsida sissetuleva päringu tööriistale määratud skeemi järgi:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    kui see töötab, jätkame tegeliku tööriista kutsumisega:

    ```typescript
    const result = await tool.callback(input);
    ```

Nagu näha, loob see lähenemine suurepärase arhitektuuri, kuna kõigil on oma koht, *server.ts* on väga väike fail, mis lihtsalt ühendab päringukäsitlejad, ja iga funktsioon on oma kaustas, näiteks tools/, resources/ või /prompts.

Suurepärane, proovime seda nüüd ehitada.

## Harjutus: Madala taseme serveri loomine

Selles harjutuses teeme järgmised sammud:

1. Loome madala taseme serveri, mis haldab tööriistade loendamist ja kutset.
2. Rakendame arhitektuuri, millele saab ehitada edasi.
3. Lisame valideerimise, et tagada teie tööriistakutsed on korrektselt valideeritud.

### -1- Loome arhitektuuri

Esmalt peame looma arhitektuuri, mis aitab meil skaleerida, kui lisame rohkem funktsioone, see välja näeb järgmine:

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

Nüüd oleme seadistanud arhitektuuri, mis tagab, et saame hõlpsasti lisada uusi tööriistu tools-kaustas. Julgelt lisage alamkaustad ressursside ja käskude jaoks.

### -2- Tööriista loomine

Vaatame, milline näeb tööriista loomine välja. Esiteks peab see olema loodud oma *tool* alamkaustas nii:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Kontrolli sisendit Pydantic mudeli abil
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: lisa Pydantic, et saaksime luua AddInputModeli ja kontrollida argumente

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Siin näeme, kuidas määratleme nime, kirjelduse, sisendskeemi Pydanticuga ja käsitleja, mida kutsutakse, kui tööriista kutsutakse. Lõpuks ekspordime `tool_add`, mis on sõnastik, mis sisaldab neid kõiki omadusi.

Samuti on olemas *schema.py*, mis määratleb tööriista kasutatava sisendskeemi:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Peame täiendama ka *__init__.py*, et tagada tools-kausta käsitlemine moodulina. Lisaks peame sellest moodulid väljapoole ekspordima nii:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Seda faili võime täiendavalt laiendada, kui lisame rohkem tööriistu.

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

Siin loome sõnastiku, mis sisaldab omadusi:

- name, tööriista nimi.
- rawSchema, see on Zodi skeem, mida kasutatakse saabuvate tööriistakutsete valideerimiseks.
- inputSchema, seda skeemi kasutab käsitleja.
- callback, mida kasutatakse tööriista kutsumiseks.

Samuti on olemas `Tool`, mida kasutatakse selle sõnastiku teisendamiseks tüübiks, mida MCP serveri käsitleja saab vastu võtta, see näeb välja nii:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Ja on olemas *schema.ts*, kus hoitakse iga tööriista sisendskeeme, mis näeb välja nii; hetkel on vaid üks skeem, kuid uute tööriistade lisamisel saab lisada rohkem kirjeid:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Suurepärane, liigume edasi tööriistade loendamise käsitlemise juurde.

### -3- Tööriistade loendamise käsitlemine

Järgmiseks, tööriistade loendamise käsitlemiseks peame seadistama päringukäsitleja. Siin on, mida lisada serverifaili:

**Python**

```python
# kood on lühendatud
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

Siin lisame dekoratsiooni `@server.list_tools` ja rakendava funktsiooni `handle_list_tools`. Viimases peame tootma tööriistade nimekirja. Pange tähele, et igal tööriistal peab olema nimi, kirjeldus ja inputSchema.

**TypeScript**

Tööriistade loendamise päringukäsitleja seadistamiseks kutsume serveril välja `setRequestHandler` skeemiga, mis sobib tehtava ülesandega, antud juhul `ListToolsRequestSchema`.

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
// Kood on lühiduse mõttes välja jäetud
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Tagastab registreeritud tööriistade nimekirja
  return {
    tools: tools
  };
});
```

Suurepärane, nüüd on meil tööriistade loendamise osa lahendatud, vaatame, kuidas tööriistu kutsuda.

### -4- Tööriista kutsumise käsitlemine

Tööriista kutsumiseks peame seadistama teise päringukäsitleja, mis keskendub sellele, millist funktsiooni kutsutakse ja milliste argumentidega.

**Python**

Kasutame dekoratsiooni `@server.call_tool` ja rakendame selle funktsiooniga `handle_call_tool`. Selle funktsiooni sees peame välja lugema tööriista nime, selle argumendi ja tagama, et argumendid on tööriista jaoks kehtivad. Võime kas valideerida argumendid selles funktsioonis või edasi tööriistas.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tööriistad on sõnastik tööriistade nimedega võtmetena
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # kutsu tööriist esile
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Siin toimub järgnev:

- Meie tööriista nimi on sisendsuparametrina juba olemas kui `name`, ja argumendid on `arguments` sõnastikus.

- Tööriista kutsutakse `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Argumentide valideerimine toimub `handler` omaduses oleva funktsiooni sees, kui see ebaõnnestub, viskab vea.

Nüüd on meil täielik ülevaade tööriistade loendamisest ja kutsumisest madala taseme serveri kaudu.

Vaata [täielikku näidet](./code/README.md)

## Kodutöö

Laiendage teile antud koodi mitme tööriista, ressursi ja käsuga ning mõelge sellele, kuidas märkate, et peate kolmanda faile lisama ainult tööriistade kausta, mitte kusagile mujale.

*Mingit lahendust ei anta*

## Kokkuvõte

Selles peatükis nägime, kuidas madala taseme serveri lähenemine toimib ja kuidas see aitab luua kena arhitektuuri, millele saab edasi ehitada. Arutasime ka valideerimist ja teile näidati, kuidas kasutada valideerimisteeke sisendi valideerimiseks skeemide loomiseks.

## Mis järgmiseks

- Järgmiseks: [Lihtne autentimine](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastutusest loobumine**:
See dokument on tõlgitud AI tõlketeenuse [Co-op Translator](https://github.com/Azure/co-op-translator) abil. Kuigi püüame täpsust, palun arvestage, et automaatsed tõlked võivad sisaldada vigu või ebatäpsusi. Originaaldokument oma emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlke kasutamisest tingitud arusaamatuste või valesti tõlgendamise eest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
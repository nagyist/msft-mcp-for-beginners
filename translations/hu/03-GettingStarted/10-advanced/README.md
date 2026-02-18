# Fejlett szerverhasználat

Az MCP SDK két különböző szervertípust kínál, a normál szervert és az alacsony szintű szervert. Normál esetben a szokásos szervert használod, hogy hozzáadj funkciókat. Bizonyos esetekben azonban az alacsony szintű szerverre szeretnél támaszkodni, például:

- Jobb architektúra. Lehetséges tiszta architektúrát létrehozni mind a szokásos szerverrel, mind az alacsony szintű szerverrel, de azt lehet állítani, hogy az alacsony szintű szerverrel kissé könnyebb.
- Funkcióelérhetőség. Néhány fejlett funkció csak alacsony szintű szerverrel használható. Ezt a későbbi fejezetekben látni fogod, amikor mintavételezést és kiváltást adunk hozzá.

## Normál szerver vs alacsony szintű szerver

Így néz ki egy MCP szerver létrehozása a normál szerverrel

**Python**

```python
mcp = FastMCP("Demo")

# Adj hozzá egy összeadási eszközt
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

// Adj hozzá egy összeadó eszközt
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

A lényeg, hogy kifejezetten hozzáadod az összes eszközt, erőforrást vagy promptot, amit a szervernek adni szeretnél. Ebben nincs semmi rossz.

### Alacsony szintű szerver megközelítés

Azonban ha az alacsony szintű szerver megközelítést használod, máshogy kell gondolkodnod, nevezetesen hogy az egyes eszközök regisztrálása helyett két kezelőt hozol létre kategóriánként (eszközök, erőforrások vagy promptok). Például az eszközök esetén két funkció van:

- Az összes eszköz listázása. Egy függvény felel az összes eszköz listázási kísérletéért.
- Az eszközök hívásának kezelése. Itt is csak egy függvény kezeli az eszközök hívását.

Ez potenciálisan kevesebb munkának hangzik, igaz? Így az eszköz regisztrálása helyett csak biztosítani kell, hogy az eszköz szerepeljen az összes eszköz listázásánál, és hogy meghívják, ha bejövő kérés érkezik az eszköz meghívására.

Nézzük meg hogyan néz ki így a kód:

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
  // Adja vissza a regisztrált eszközök listáját
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

Most van egy függvényünk, amely egy listát ad vissza a funkciókról. Az eszközök listájának minden eleme most olyan mezőket tartalmaz, mint a `name`, `description` és `inputSchema`, hogy megfeleljen a visszatérési típusnak. Ez lehetővé teszi, hogy az eszközeinket és a funkciódefinícióinkat máshol tároljuk. Most már létrehozhatjuk az összes eszközt egy eszközök mappában, és ugyanez igaz az összes funkcióra is, így a projekted hirtelen így szerveződhet:

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

Ez nagyszerű, az architektúránk viszonylag tisztának tűnhet.

Mi a helyzet az eszközök hívásával, ugyanaz az ötlet, egy kezelő meghívni egy eszközt, bármelyik eszközt? Igen, pontosan, itt van ennek a kódja:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # a tools egy szótár, ahol az eszköznevek a kulcsok
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
    // TODO hívja meg az eszközt,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Ahogy a fenti kódból látható, ki kell bontanunk, hogy melyik eszközt kell meghívni, milyen argumentumokkal, majd folytatnunk kell az eszköz meghívását.

## A megközelítés javítása validációval

Eddig azt láttad, hogy az eszközök, erőforrások és promptok hozzáadására vonatkozó regisztrációidat helyettesíthetjük ezzel a két kezelővel kategóriánként. Mi mást kell még tennünk? Hozzá kell adnunk valamilyen validációt, hogy biztosítsuk, az eszközt a helyes argumentumokkal hívják meg. Minden futtatókörnyezetnek megvan a saját megoldása erre, például Pythonban a Pydantic, TypeScript-ben a Zod. Az ötlet a következő:

- Áthelyezni a funkció (eszköz, erőforrás vagy prompt) létrehozásának logikáját a dedikált mappájába.
- Hozzáadni egy módját annak, hogy egy bejövő kérés érvényességét ellenőrizzük, például egy eszköz hívásakor.

### Funkció létrehozása

Funkció létrehozásához létre kell hozni egy fájlt az adott funkció számára, és biztosítani kell, hogy tartalmazza a kötelező mezőket, amelyek az adott funkcióra vonatkoznak. A mezők egy kissé eltérnek az eszközök, erőforrások és promptok között.

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
        # Érvényesítsük a bemenetet Pydantic modell segítségével
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TEENDŐ: adjuk hozzá a Pydantic-et, hogy létrehozhassunk egy AddInputModelt és érvényesíthessük a paramétereket

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

itt láthatod, hogy a következőket tesszük:

- Egy Pydantic séma létrehozása `AddInputModel` néven mezőkkel `a` és `b` a *schema.py* fájlban.
- Megkíséreljük a bejövő kérést a `AddInputModel` típusúra parse-olni, ha az argumentumok nem egyeznek, az összeomlik:

   ```python
   # add.py
    try:
        # Érvényesítse a bemenetet Pydantic modell segítségével
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Választhatod, hogy ezt a parse-olási logikát az eszközhívásban vagy a kezelőfüggvényben helyezed el.

**TypeScript**

```typescript
// szerver.ts
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

       // @ts-figyelmen kívül hagyás
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

// séma.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// hozzáadás.ts
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

- A kezelőben, amely az összes eszközhívást kezeli, megpróbáljuk a bejövő kérést az adott eszköz által definiált sémára parse-olni:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ha ez sikerül, akkor folytatjuk az eszköz tényleges meghívását:

    ```typescript
    const result = await tool.callback(input);
    ```

Ahogy látható, ez a megközelítés egy nagyszerű architektúrát alkot, mivel mindennek megvan a maga helye, a *server.ts* egy nagyon kicsi fájl, amely csak összeköti a kéréskezelőket, és minden funkció a saját mappájában van, azaz tools/, resources/ vagy prompts/.

Szuper, nézzük meg a következő lépéseket.

## Gyakorlat: Alacsony szintű szerver létrehozása

Ebben a gyakorlatban a következőket fogjuk tenni:

1. Létrehozunk egy alacsony szintű szervert az eszközök listázásának és hívásának kezelésére.
2. Megvalósítunk egy olyan architektúrát, amelyre építeni tudsz.
3. Hozzáadunk validációt, hogy biztosítsuk az eszköz hívások helyes érvényesítését.

### -1- Architektúra létrehozása

Az első dolog, amit meg kell oldanunk, egy olyan architektúra, amely segít skálázni új funkciók hozzáadásakor, így néz ki:

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

Most egy olyan architektúrát hoztunk létre, amely biztosítja, hogy könnyen hozzá tudjuk adni az új eszközöket a tools mappában. Nyugodtan kövesd ezt az erőforrások és promptok almappáinak hozzáadásához is.

### -2- Eszköz létrehozása

Nézzük meg, hogyan néz ki egy eszköz létrehozása. Először is, létre kell hozni az eszközt az *tools* alkönyvtárában így:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Érvényesítsük a bemenetet Pydantic modellel
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TEENDŐ: adjuk hozzá a Pydantic-et, hogy létrehozhassunk egy AddInputModel-t és érvényesíthessük az argumentumokat

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Itt látjuk, hogy hogyan definiáljuk a nevet, leírást, a bemeneti sémát Pydantic segítségével, valamint egy kezelőt, amely meghívásra kerül, amikor ezt az eszközt hívják. Végül kiexponáljuk a `tool_add` nevű szótárt, amely tartalmazza ezeket a tulajdonságokat.

Van emellett egy *schema.py* fájl is, amely az eszköz input sémáját tartalmazza:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Szükséges továbbá a *__init__.py* fájlt is kitölteni, hogy az eszközök mappa modulnak legyen tekintve. Ezen kívül exponálni kell a benne lévő modulokat így:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Ezt a fájlt bővíthetjük, ahogy egyre több eszközt adunk hozzá.

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

Itt létrehozunk egy szótárt a következő tulajdonságokkal:

- name, az eszköz neve.
- rawSchema, ez a Zod séma, amelyet a bejövő hívások validálásához használunk.
- inputSchema, ezt a sémát használja a kezelő.
- callback, ez indítja el az eszközt.

Van még a `Tool` típus, amely e szótárt a MCP szerver kezelő által elfogadott típussá konvertálja, és így néz ki:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

És van egy *schema.ts* fájl is, ahol tároljuk az egyes eszközök input sémáit, jelenleg csak egy séma van benne, de ahogy eszközöket adunk hozzá, további bejegyzéseket vehetünk fel:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Nagyszerű, most térjünk rá az eszközök listázásának kezelésére.

### -3- Az eszközök listázásának kezelése

Az eszközök listázásának kezeléséhez request kezelőt kell beállítanunk. Ezt kell hozzáadni a szerver fájlhoz:

**Python**

```python
# a kód rövidítve van
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

Itt adjuk hozzá a `@server.list_tools` dekorátort és a megvalósító függvényt `handle_list_tools` néven. Ebben utóbbi kell előállítanunk az eszközök listáját. Ügyelj arra, hogy minden eszköznek rendelkeznie kell névvel, leírással és input sémával. 

**TypeScript**

Ahhoz, hogy beállítsuk a request kezelőt az eszközök listázására, a szerveren a `setRequestHandler` függvényt kell meghívni egy olyan sémával, ami megfelel a céljainknak, ez esetben a `ListToolsRequestSchema`.

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
// a kód rövidítve
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Visszaadja a regisztrált eszközök listáját
  return {
    tools: tools
  };
});
```

Szuper, most megoldottuk az eszközök listázásának részét, nézzük meg, hogyan hívhatunk meg eszközöket.

### -4- Eszköz hívásának kezelése

Az eszköz meghívásához egy másik kéréskezelőt kell beállítanunk, amely arra összpontosít, hogy meghatározza, melyik funkciót kell meghívni és milyen argumentumokkal.

**Python**

Használjuk a `@server.call_tool` dekorátort, és hajtsuk végre egy `handle_call_tool` függvénnyel. Ebben a függvényben ki kell elemezni az eszköz nevét, annak argumentumait, és ellenőrizni kell, hogy az argumentumok érvényesek-e az adott eszközhöz. Az argumentumokat validálhatjuk ebben a függvényben vagy később, magában az eszközben.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # a tools egy szótár, amelyben az eszközök nevei a kulcsok
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # hívja meg az eszközt
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

A történés:

- Az eszköz neve már jelen van, mint az input paraméter `name`, ahogyan az argumentumaink is, az `arguments` szótárban.

- Az eszközt a `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` hívással hajtjuk végre. Az argumentumok validációja a `handler` tulajdonságban történik, amely egy függvényre mutat; ha ez nem sikerül, kivételt dob.

Így most már teljes rálátásunk van arra, hogyan lehet felsorolni és meghívni eszközöket alacsony szintű szerver használatával.

Nézd meg a [teljes példát](./code/README.md) itt

## Feladat

Bővítsd a megadott kódot további eszközökkel, erőforrásokkal és promptokkal, és figyeld meg, hogy csak a tools könyvtárban kell fájlokat hozzáadnod, máshol nem.

*Nincs megoldás megadva*

## Összefoglalás

Ebben a fejezetben láttuk, hogyan működik az alacsony szintű szerver megközelítés, és hogyan segít ez egy használható architektúra létrehozásában, amelyre folyamatosan építhetünk. Beszéltünk a validációról is, és megmutattuk, hogyan dolgozhatsz validációs könyvtárakkal bemenet ellenőrzési sémák létrehozásához.

## Mi következik

- Következő: [Egyszerű hitelesítés](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Nyilatkozat**:  
Ezt a dokumentumot az AI fordító szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével fordítottuk le. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén profi emberi fordítást javaslunk. Nem vállalunk felelősséget az ebből az átiratból eredő félreértésekért vagy téves értelmezésekért.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
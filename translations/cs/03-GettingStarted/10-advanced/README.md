# Pokročilé použití serveru

V MCP SDK jsou vystaveny dva různé typy serverů, váš běžný server a nízkoúrovňový server. Obvykle byste použili běžný server k přidávání funkcí. V některých případech však chcete spoléhat na nízkoúrovňový server, například:

- Lepší architektura. Je možné vytvořit čistou architekturu jak s běžným serverem, tak s nízkoúrovňovým serverem, ale dá se říci, že je to o něco jednodušší s nízkoúrovňovým serverem.
- Dostupnost funkcí. Některé pokročilé funkce lze použít pouze s nízkoúrovňovým serverem. Uvidíte to v dalších kapitolách, když přidáme vzorkování a elicitační proces.

## Běžný server vs nízkoúrovňový server

Takto vypadá vytvoření MCP Serveru s běžným serverem:

**Python**

```python
mcp = FastMCP("Demo")

# Přidejte nástroj pro sčítání
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

// Přidejte nástroj pro sčítání
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

Podstata je v tom, že explicitně přidáváte každý nástroj, zdroj nebo prompt, který chcete, aby server měl. Na tom není nic špatného.

### Přístup nízkoúrovňového serveru

Když však použijete přístup nízkoúrovňového serveru, musíte přemýšlet trochu jinak, totiž že místo registrace každého nástroje vytvoříte dvě zpracovatelské funkce na každý typ funkce (nástroje, zdroje nebo prompty). Například nástroje pak mají pouze dvě funkce, například:

- Vypsání všech nástrojů. Jedna funkce bude zodpovědná za všechny pokusy o vypsání nástrojů.
- zpracování volání všech nástrojů. Opět je tady jen jedna funkce, která zpracovává volání nástroje.

To zní jako možná méně práce, že? Takže místo registrace nástroje stačí pouze zajistit, aby byl nástroj uveden při vypsání všech nástrojů a aby byl zavolán, když přijde požadavek na volání nástroje.

Podívejme se, jak kód nyní vypadá:

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
  // Vrátit seznam registrovaných nástrojů
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

Tady máme funkci, která vrací seznam funkcí. Každý záznam v seznamu nástrojů má nyní pole jako `name`, `description` a `inputSchema` odpovídající typu návratové hodnoty. To nám umožňuje umístit definice nástrojů a funkcí jinam. Nyní můžeme vytvořit všechny naše nástroje ve složce tools a to samé platí pro všechny vaše funkce, takže váš projekt může být najednou organizován takto:

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

To je skvělé, naše architekturu lze udělat docela čistou.

A co volání nástrojů, je to stejný princip, jedna funkce zpracuje volání nástroje, jakkoli nástroj? Ano, přesně tak, tady je kód pro to:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je slovník s názvy nástrojů jako klíči
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
    // TODO zavolat nástroj,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Jak vidíte z výše uvedeného kódu, potřebujeme analyzovat, který nástroj volat a s jakými argumenty, a poté volat tento nástroj.

## Vylepšení přístupu pomocí validace

Dosud jste viděli, jak lze všechna vaše registrace pro přidávání nástrojů, zdrojů a promptů nahradit těmito dvěma zpracovatelskými funkcemi na typ funkce. Co ještě je potřeba? Měli bychom přidat nějakou formu validace, abychom zajistili, že nástroj je volán se správnými argumenty. Každé runtime má vlastní řešení, například Python používá Pydantic a TypeScript používá Zod. Myšlenka je následující:

- Přesunout logiku pro vytvoření funkce (nástroj, zdroj nebo prompt) do jejího vlastního adresáře.
- Přidat způsob, jak validovat příchozí požadavek, např. na volání nástroje.

### Vytvoření funkce

Chceme-li vytvořit funkci, musíme vytvořit soubor pro tuto funkci a zajistit, že obsahuje povinná pole požadovaná pro danou funkci. Která pole se trochu liší mezi nástroji, zdroji a prompty.

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
        # Ověřit vstup pomocí Pydantic modelu
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: přidat Pydantic, abychom mohli vytvořit AddInputModel a ověřit argumenty

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Zde vidíte, jak děláme toto:

- Vytvoříme schéma pomocí Pydantic `AddInputModel` s poli `a` a `b` v souboru *schema.py*.
- Pokusíme se analyzovat příchozí požadavek jako typ `AddInputModel`, pokud jsou parametry nesprávné, dojde k chybě:

   ```python
   # add.py
    try:
        # Ověřte vstup pomocí modelu Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Můžete si vybrat, zda tuto logiku parsování vložíte přímo do volání nástroje nebo do zpracovatelské funkce.

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

- Ve zpracovateli volání všech nástrojů nyní zkoušíme analyzovat příchozí požadavek do definovaného schématu nástroje:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    Pokud to funguje, pokračujeme ve volání skutečného nástroje:

    ```typescript
    const result = await tool.callback(input);
    ```

Jak vidíte, tento přístup vytváří skvělou architekturu, protože vše má své místo, *server.ts* je velmi malý soubor, který jen propojil zpracovatele požadavků a každá funkce je ve svém vlastním adresáři, tj. tools/, resources/ nebo /prompts.

Skvělé, zkusme to teď postavit.

## Cvičení: Vytvoření nízkoúrovňového serveru

V tomto cvičení uděláme následující:

1. Vytvoříme nízkoúrovňový server, který bude zpracovávat výpis nástrojů a jejich volání.
1. Implementujeme architekturu, na které můžeme stavět.
1. Přidáme validaci, abychom zajistili správnou validaci volání nástrojů.

### -1- Vytvoření architektury

První věc, kterou potřebujeme řešit, je architektura, která nám pomůže škálovat se zvyšujícím se počtem funkcí. Takto vypadá:

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

Teď máme nastavenou architekturu, která zajistí, že můžeme snadno přidávat nové nástroje do složky tools. Klidně ji použijte i pro přidání podsložek resources a prompts.

### -2- Vytvoření nástroje

Podívejme se, jak vypadá vytvoření nástroje. Nejprve musí být vytvořen ve své podsložce *tool* takto:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Ověřit vstup pomocí Pydantic modelu
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: přidat Pydantic, abychom mohli vytvořit AddInputModel a ověřit argumenty

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Vidíme zde, jak definujeme název, popis, vstupní schéma pomocí Pydantic a zpracovatele, který bude vyvolán, jakmile je tento nástroj volán. Nakonec zpřístupníme `tool_add`, což je slovník, který obsahuje všechny tyto vlastnosti.

Je tu také *schema.py*, který definuje vstupní schéma používané naším nástrojem:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Musíme také naplnit *__init__.py*, aby byla složka tools považována za modul. Kromě toho musíme zpřístupnit moduly uvnitř takto:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Do tohoto souboru můžeme postupně přidávat další nástroje.

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

Zde vytváříme slovník skládající se z vlastností:

- name, to je název nástroje.
- rawSchema, to je Zod schéma, použije se k validaci příchozích požadavků na volání tohoto nástroje.
- inputSchema, toto schéma použije zpracovatel.
- callback, to je použito k vyvolání nástroje.

Je tu také `Tool`, které slouží ke konverzi tohoto slovníku na typ, který může přijmout handler MCP serveru, a vypadá takto:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

A je tu *schema.ts*, kde uchováváme vstupní schémata pro každý nástroj, které v současnosti obsahuje pouze jedno schéma, ale s přidáváním nástrojů přibudou další položky:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Skvělé, pojďme teď pokračovat s vyřizováním výpisu našich nástrojů.

### -3- Zpracování výpisu nástrojů

K vyřizování výpisu nástrojů potřebujeme nastavit zpracovatele požadavků. Tady je, co je potřeba přidat do našeho serverového souboru:

**Python**

```python
# kód vynechán pro stručnost
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

Přidáme dekorátor `@server.list_tools` a implementační funkci `handle_list_tools`. V ní potřebujeme vytvořit seznam nástrojů. Všimněte si, že každý nástroj musí mít název, popis a inputSchema.

**TypeScript**

Pro nastavení zpracovatele požadavku na výpis nástrojů voláme na serveru `setRequestHandler` se schématem, které odpovídá tomu, co chceme udělat, v tomto případě `ListToolsRequestSchema`.

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
// kód vynechán pro stručnost
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Vrátit seznam registrovaných nástrojů
  return {
    tools: tools
  };
});
```

Skvěle, tím jsme vyřešili část pro výpis nástrojů, pojďme se podívat, jak můžeme volat nástroje.

### -4- Zpracování volání nástroje

Pro volání nástroje potřebujeme nastavit další zpracovatele požadavků, tentokrát se zaměřením na požadavek, který specifikuje, jakou funkci volat a s jakými argumenty.

**Python**

Použijme dekorátor `@server.call_tool` a implementujme ho funkcí `handle_call_tool`. V této funkci je potřeba získat název nástroje, jeho argumenty a zajistit, že argumenty jsou platné pro daný nástroj. Můžeme argumenty validovat v této funkci nebo až v samotném nástroji.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je slovník s názvy nástrojů jako klíči
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # vyvolejte nástroj
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Takto to funguje:

- Název nástroje je již přítomen jako vstupní parametr `name`, což platí i pro argumenty formou slovníku `arguments`.

- Nástroj je zavolán pomocí `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Validace argumentů probíhá v `handler` vlastnosti, která ukazuje na funkci, a pokud selže, vyvolá se výjimka.

Takže, nyní plně rozumíme výpisu a volání nástrojů pomocí nízkoúrovňového serveru.

Viz [úplný příklad](./code/README.md) zde

## Zadání

Rozšiřte kód, který máte, o řadu nástrojů, zdrojů a promptů a pozorujte, jak si všimnete, že stačí přidávat jen soubory ve složce tools a nikde jinde.

*Řešení není poskytnuto*

## Shrnutí

V této kapitole jsme viděli, jak funguje přístup nízkoúrovňového serveru a jak nám může pomoci vytvořit dobrou architekturu, na kterou můžeme dále stavět. Diskutovali jsme také o validaci a ukázalo se, jak pracovat s knihovnami pro validaci k vytvoření schémat pro validaci vstupů.

## Co bude dál

- Další: [Jednoduchá autentizace](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o vyloučení odpovědnosti**:  
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). I když usilujeme o přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Původní dokument v jeho mateřském jazyce by měl být považován za rozhodující zdroj. Pro zásadní informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za žádná nedorozumění nebo chybné výklady vyplývající z použití tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
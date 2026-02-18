# Pokročilé využitie servera

V MCP SDK sú k dispozícii dva rôzne typy serverov, váš bežný server a nízkoúrovňový server. Normálne by ste používali bežný server na pridávanie funkcií. V niektorých prípadoch však chcete využiť nízkoúrovňový server, napríklad:

- Lepšia architektúra. Je možné vytvoriť čistú architektúru s bežným serverom aj nízkoúrovňovým serverom, ale dá sa tvrdiť, že je to o niečo jednoduchšie s nízkoúrovňovým serverom.
- Dostupnosť funkcií. Niektoré pokročilé funkcie je možné využiť iba s nízkoúrovňovým serverom. Uvidíte to v nasledujúcich kapitolách, keď pridáme sampling a elicitation.

## Bežný server vs nízkoúrovňový server

Takto vyzerá vytvorenie MCP servera s bežným serverom

**Python**

```python
mcp = FastMCP("Demo")

# Pridajte nástroj na sčítanie
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

// Pridajte nástroj na sčítanie
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

Pointa je v tom, že explicitne pridávate každý nástroj, zdroj alebo prompt, ktorý chcete, aby server mal. S tým nie je nič zlé.  

### Prístup nízkoúrovňového servera

Keď však použijete prístup nízkoúrovňového servera, musíte na to myslieť trochu inak, konkrétne namiesto registrovania každého nástroja vytvárate dva handleri na typ funkcie (nástroje, zdroje alebo prompty). Napríklad nástroje majú len dve funkcie takto:

- Zoznam všetkých nástrojov. Jedna funkcia zodpovedá za všetky pokusy o zoznam nástrojov.
- Spracovanie volaní všetkých nástrojov. Aj tu je len jedna funkcia, ktorá spracováva volania nástroja.

To znie ako potenciálne menej práce, však? Namiesto registrovania nástroja stačí len zabezpečiť, aby bol nástroj uvedený pri zozname všetkých nástrojov a aby bol volaný, keď príde požiadavka na jeho volanie.

Pozrime sa, ako teraz vyzerá kód:

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
  // Vrátiť zoznam registrovaných nástrojov
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

Tu máme funkciu, ktorá vracia zoznam funkcií. Každý záznam v zozname nástrojov má teraz polia ako `name`, `description` a `inputSchema` podľa návratového typu. To nám umožňuje umiestniť definície nástrojov a funkcií inde. Môžeme vytvoriť všetky naše nástroje v priečinku tools a to isté platí pre všetky vaše funkcie, takže váš projekt môže byť náhle organizovaný takto:

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

To je skvelé, naša architektúra môže byť veľmi čistá.

A čo volanie nástrojov, je to rovnaký princíp, jeden handler na volanie akéhokoľvek nástroja? Áno, presne tak, tu je kód na to:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je slovník s názvami nástrojov ako kľúčmi
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
    // TODO zavolať nástroj,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Ako môžete vidieť z uvedeného kódu, musíme rozparsovať, ktorý nástroj volať, a s akými argumentmi, potom pokračujeme v volaní nástroja.

## Vylepšenie prístupu pomocou validácie

Doteraz ste videli, ako všetky vaše registrácie na pridanie nástrojov, zdrojov a promptov možno nahradiť týmito dvoma handlermi na typ funkcie. Čo ešte musíme urobiť? Mali by sme pridať nejakú formu validácie, aby sme zabezpečili, že nástroj je volaný so správnymi argumentmi. Každé runtime má svoje riešenie, napríklad Python používa Pydantic a TypeScript používa Zod. Myšlienka je takáto:

- Presunúť logiku vytvorenia funkcie (nástroj, zdroj alebo prompt) do jej vyhradenej zložky.
- Pridať spôsob na validáciu prichádzajúcej požiadavky, napríklad na volanie nástroja.

### Vytvorenie funkcie

Na vytvorenie funkcie je potrebné vytvoriť súbor pre tú funkciu a zabezpečiť, aby mala povinné polia požadované pre tú funkciu. Ktoré polia sa líšia trochu medzi nástrojmi, zdrojmi a promptmi.

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
        # Overiť vstup pomocou Pydantic modelu
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: pridať Pydantic, aby sme mohli vytvoriť AddInputModel a overiť argumenty

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Tu vidíte, ako:

- Vytvoríme schému pomocou Pydantic `AddInputModel` s poliami `a` a `b` v súbore *schema.py*.
- Pokúsime sa rozparsovať prichádzajúcu požiadavku ako typ `AddInputModel`, ak dôjde k nezrovnalosti v parametroch, toto zlyhá:

   ```python
   # add.py
    try:
        # Overiť vstup pomocou modelu Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Môžete si vybrať, či túto logiku parsovania umiestnite priamo do volania nástroja alebo do handler funkcie.

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

- V handleri, ktorý spracováva všetky volania nástrojov, teraz skúšame rozparsovať prichádzajúcu požiadavku do definovanej schémy nástroja:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ak to vyjde úspešne, pokračujeme v volaní samotného nástroja:

    ```typescript
    const result = await tool.callback(input);
    ```

Ako vidíte, tento prístup vytvára skvelú architektúru, pretože všetko má svoje miesto, *server.ts* je veľmi malý súbor, ktorý iba prepája request handleri a každá funkcia je vo svojej príslušnej zložke, napríklad tools/, resources/ alebo /prompts.

Skvelé, poďme to teraz skúsiť zostrojiť.

## Cvičenie: Vytvorenie nízkoúrovňového servera

V tomto cvičení urobíme nasledovné:

1. Vytvoríme nízkoúrovňový server, ktorý bude spracovávať zoznam nástrojov a volanie nástrojov.
2. Implementujeme architektúru, na ktorej môžete stavať.
3. Pridáme validáciu, aby sme zabezpečili správnu validáciu volaní nástrojov.

### -1- Vytvorenie architektúry

Prvou vecou, ktorú musíme riešiť je architektúra, ktorá nám pomôže škálovať, keď pridávame viac funkcií, vyzerá takto:

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

Teraz máme nastavenú architektúru, ktorá zaručuje, že môžeme jednoducho pridávať nové nástroje v priečinku tools. Kľudne pokračujte a pridajte podadresáre pre resources a prompts.

### -2- Vytvorenie nástroja

Pozrime sa, ako vyzerá vytvorenie nástroja. Najprv musí byť vytvorený vo svojej podzložke *tool* takto:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Overiť vstup pomocou modelu Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: pridať Pydantic, aby sme mohli vytvoriť AddInputModel a overiť argumenty

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Tu vidíme, ako definujeme názov, popis, vstupnú schému pomocou Pydantic a handler, ktorý sa spustí, keď je tento nástroj volaný. Nakoniec sprístupníme `tool_add`, čo je slovník drží všetky tieto vlastnosti.

Je tu tiež *schema.py*, ktorý definuje vstupnú schému používanú naším nástrojom:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Potrebujeme tiež naplniť súbor *__init__.py*, aby sa priečinok tools správal ako modul. Navyše musíme vystaviť moduly v ňom takto:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Do tohto súboru môžeme pridávať ďalšie nástroje podľa potreby.

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

Tu vytvárame slovník pozostávajúci z vlastností:

- name, názov nástroja.
- rawSchema, toto je Zod schéma, ktorá sa používa na validáciu prichádzajúcich požiadaviek na volanie tohto nástroja.
- inputSchema, táto schéma sa použije v handleri.
- callback, používa sa na vyvolanie nástroja.

Je tu tiež `Tool`, ktorý slúži na konverziu tohto slovníka na typ, ktorý MCP server handler môže akceptovať, vyzerá takto:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Existuje tiež *schema.ts*, kde uchovávame vstupné schémy pre každý nástroj, vyzerá to takto, zatiaľ s jedinou schémou, ale postupne podľa pridávania nástrojov ich môžeme rozšíriť:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Skvelé, teraz pokračujme riešením zoznamu nástrojov.

### -3- Spracovanie zoznamu nástrojov

Ďalej, na spracovanie zoznamu nástrojov, musíme nastaviť request handler pre toto. Tu je, čo treba pridať do server súboru:

**Python**

```python
# kód vynechaný kvôli stručnosti
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

Tu pridávame dekorátor `@server.list_tools` a implementačnú funkciu `handle_list_tools`. V nej treba vytvoriť zoznam nástrojov. Všimnite si, že každý nástroj musí mať name, description a inputSchema.   

**TypeScript**

Na nastavenie request handlera pre zoznam nástrojov voláme `setRequestHandler` na serveri s vhodnou schémou pre našu požiadavku, v tomto prípade `ListToolsRequestSchema`. 

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
// kód vynechaný pre stručnosť
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Vráti zoznam zaregistrovaných nástrojov
  return {
    tools: tools
  };
});
```

Skvelé, teraz máme vyriešenú časť s výpisom nástrojov, poďme sa pozrieť na volanie nástrojov.

### -4- Spracovanie volania nástroja

Na volanie nástroja potrebujeme nastaviť ďalší request handler, tentokrát zameraný na spracovanie požiadavky, ktorá špecifikuje, ktorý nástroj volať a s akými argumentmi.

**Python**

Použijeme dekorátor `@server.call_tool` a implementujeme ho funkciou napríklad `handle_call_tool`. V tejto funkcii rozparsujeme názov nástroja, jeho argumenty a overíme, či sú argumenty platné pre daný nástroj. Validáciu môžeme spraviť tu alebo neskôr v samotnom nástroji.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je slovník s názvami nástrojov ako kľúčmi
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # zavolajte nástroj
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Tu sa deje toto:

- Názov nástroja je už prítomný ako vstupný parameter `name`, čo platí aj pre naše argumenty ako slovník `arguments`.
- Nástroj sa volá pomocou `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Validácia argumentov prebieha vo vlastnosti `handler`, ktorá ukazuje na funkciu, ak zlyhá, vyhodí výnimku.

Takže teraz máme plné pochopenie toho, ako sa zoznam a volanie nástrojov rieši pomocou nízkoúrovňového servera.

Pozrite si [úplný príklad](./code/README.md) tu

## Zadanie

Rozšírte poskytnutý kód o množstvo nástrojov, zdrojov a promptov a zamyslite sa, ako ste si všimli, že teda stačí pridávať iba súbory do priečinka tools a nikde inde.

*Riešenie nie je poskytnuté*

## Zhrnutie

V tejto kapitole sme videli, ako funguje prístup nízkoúrovňového servera a ako nám pomáha vytvoriť peknú architektúru, na ktorej môžeme stavať. Diskutovali sme tiež validáciu a ukázali sme si, ako pracovať s validačnými knižnicami na tvorbu schém pre validáciu vstupov.

## Čo bude ďalej

- Nasleduje: [Jednoduchá autentifikácia](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Upozornenie**:  
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, prosím vezmite na vedomie, že automatizované preklady môžu obsahovať chyby alebo nepresnosti. Originálny dokument v jeho pôvodnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
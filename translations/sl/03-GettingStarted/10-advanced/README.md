# Napredna uporaba strežnika

V MCP SDK so predstavljeni dve različni vrsti strežnikov, vaš običajni strežnik in nizkonivojski strežnik. Običajno bi uporabljali običajni strežnik za dodajanje funkcij. V nekaterih primerih pa želite zaupati nizkonivojskemu strežniku, na primer:

- Boljša arhitektura. Možno je ustvariti čisto arhitekturo z običajnim strežnikom in nizkonivojskim strežnikom, vendar lahko trdimo, da je nekoliko lažje z nizkonivojskim strežnikom.
- Razpoložljivost funkcij. Nekatere napredne funkcije so na voljo le pri nizkonivojskem strežniku. To boste videli v kasnejših poglavjih, ko bomo dodajali vzorčenje in pridobivanje.

## Običajni strežnik proti nizkonivojskemu strežniku

Tako izgleda ustvarjanje MCP strežnika z običajnim strežnikom

**Python**

```python
mcp = FastMCP("Demo")

# Dodaj orodje za seštevanje
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

// Dodajte orodje za seštevanje
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

Pomen je v tem, da eksplicitno dodate vsako orodje, vir ali poziv, ki ga želite, da ga ima strežnik. Ni nič narobe s tem.

### Pristop nizkonivojskega strežnika

Ko pa uporabite pristop nizkonivojskega strežnika, morate razmišljati drugače, in sicer, da namesto registracije vsakega orodja ustvarite dva upravljavca na vrsto funkcije (orodja, viri ali pozivi). Na primer, orodja imajo samo dve funkciji, kot sledi:

- Prikaz vseh orodij. Ena funkcija bi bila odgovorna za vse poskuse prikaza orodij.
- Upravljanje klica vseh orodij. Tudi tukaj obstaja le ena funkcija, ki upravlja klice orodja.

To se sliši kot potencialno manj dela, kajne? Namesto da registriram orodje, moram samo zagotoviti, da so orodja prikazana, ko prikažem vsa orodja, in da se kliče, ko pride zahteva za klic orodja.

Poglejmo, kako zdaj izgleda koda:

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
  // Vrni seznam registriranih orodij
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

Tukaj imamo funkcijo, ki vrne seznam funkcij. Vsak vnos na seznamu orodij zdaj vsebuje polja, kot so `name`, `description` in `inputSchema`, da ustreza tipu vrnitve. To nam omogoča, da orodja in definicije funkcij postavimo drugam. Zdaj lahko ustvarimo vsa orodja v mapi tools in enako velja za vse funkcije, tako da je vaš projekt lahko organiziran takole:

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

To je super, naša arhitektura je lahko zelo čista.

Kaj pa klic orodij, je potem ista ideja, en upravljalec za klic orodja, katerega koli orodja? Da, točno tako, tukaj je koda za to:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je slovar z imeni orodij kot ključih
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
    // TODO pokliči orodje,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Kot lahko vidite iz zgornje kode, moramo razčleniti, katero orodje pokličemo in s katerimi argumenti, nato pa nadaljujemo s klicem orodja.

## Izboljšava pristopa z validacijo

Doslej ste videli, kako lahko vse vaše registracije za dodajanje orodij, virov in pozivov nadomestimo s temi dvema upravljalcema na vrsto funkcije. Kaj še moramo storiti? Dodati moramo nekakšno validacijo, da zagotovimo, da je orodje poklicano z ustreznimi argumenti. Vsako izvajanje ima svojo rešitev za to, na primer Python uporablja Pydantic, TypeScript pa Zod. Ideja je, da naredimo naslednje:

- Prenesemo logiko ustvarjanja funkcije (orodje, vir ali poziv) v namensko mapo.
- Dodamo način za validacijo prihajajoče zahteve, na primer za klic orodja.

### Ustvarjanje funkcije

Za ustvarjanje funkcije bomo morali ustvariti datoteko za to funkcijo in zagotoviti, da vsebuje obvezna polja, ki jih zahteva ta funkcija. Polja se nekoliko razlikujejo med orodji, viri in pozivi.

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
        # Preveri vhod z uporabo Pydantic modela
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: dodaj Pydantic, da lahko ustvarimo AddInputModel in preverimo argumente

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Tukaj lahko vidite, kako naredimo naslednje:

- Ustvarimo shemo z uporabo Pydantic `AddInputModel` s polji `a` in `b` v datoteki *schema.py*.
- Poskušamo razčleniti prihajajočo zahtevo kot tip `AddInputModel`, če pride do neskladnosti v parametrih, se bo program zrušil:

   ```python
   # add.py
    try:
        # Preveri vhod z uporabo Pydantic modela
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Lahko izberete, ali to logiko razčlenjevanja položite neposredno v klic orodja ali v funkcijo upravljalca.

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

- V upravljalcu, ki obravnava vse klice orodij, zdaj poskušamo razčleniti prihajajočo zahtevo v shemo, določeno za orodje:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    Če to deluje, nadaljujemo s klicem dejanskega orodja:

    ```typescript
    const result = await tool.callback(input);
    ```

Kot vidite, ta pristop ustvarja odlično arhitekturo, saj ima vse svoj prostor, *server.ts* je zelo majhna datoteka, ki samo poveže upravljalce zahtev, vsaka funkcija pa je v svoji ustrezni mapi, npr. tools/, resources/ ali /prompts.

Super, poskusimo to zdaj zgraditi.

## Vaja: Ustvarjanje nizkonivojskega strežnika

V tej vaji bomo naredili naslednje:

1. Ustvarili nizkonivojski strežnik, ki obravnava prikazovanje orodij in klic orodij.
2. Implementirali arhitekturo, na kateri lahko gradite.
3. Dodali validacijo, da zagotovimo pravilno validacijo vaših klicev orodij.

### -1- Ustvarjanje arhitekture

Prva stvar, ki jo moramo rešiti, je arhitektura, ki nam pomaga skalirati, ko dodajamo več funkcij, tako izgleda:

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

Zdaj smo vzpostavili arhitekturo, ki zagotavlja, da lahko z lahkoto dodajamo nova orodja v mapo tools. Lahko dodate tudi podmape za vire in pozive.

### -2- Ustvarjanje orodja

Poglejmo, kako izgleda ustvarjanje orodja. Najprej ga je treba ustvariti v njegovi podmapi *tool*, tako:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validiraj vhod z uporabo Pydantic modela
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: dodaj Pydantic, da lahko ustvarimo AddInputModel in validiramo argumente

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Tukaj vidimo, kako definiramo ime, opis, vhodno shemo s Pydantic in upravljalca, ki bo klican, ko je orodje poklicano. Nazadnje eksponiramo `tool_add`, ki je slovar z vsemi temi lastnostmi.

Obstaja tudi *schema.py*, ki se uporablja za definiranje vhodne sheme, ki jo uporablja naše orodje:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Prav tako moramo napolniti *__init__.py*, da zagotovimo, da je mapa tools obravnavana kot modul. Poleg tega moramo izpostaviti module v njej, tako:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

To datoteko lahko še naprej dopolnjujemo, ko dodajamo več orodij.

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

Tukaj ustvarimo slovar, ki vsebuje lastnosti:

- name, to je ime orodja.
- rawSchema, to je Zod shema, ki se uporablja za validacijo prihajajočih zahtev za klic tega orodja.
- inputSchema, to shemo uporablja upravljalec.
- callback, to se uporablja za klic orodja.

Obstaja tudi `Tool`, ki se uporablja za pretvorbo tega slovarja v tip, ki ga lahko sprejme upravljalec MCP strežnika, in izgleda takole:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Obstaja tudi *schema.ts*, kamor shranjujemo vhodne sheme za vsako orodje, ki izgleda takole, trenutno s samo eno shemo, a ko dodajamo orodja, lahko dodamo več vnosov:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Super, nadaljujmo z upravljanjem prikaza našega orodja.

### -3- Upravljanje prikaza orodij

Nato, da upravljamo prikaz naših orodij, moramo nastaviti upravljalca zahtev za to. Tukaj je, kar moramo dodati v našo datoteko strežnika:

**Python**

```python
# koda izpuščena zaradi jedrnatosti
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

Tukaj dodamo dekorator `@server.list_tools` in implementacijsko funkcijo `handle_list_tools`. V slednji moramo proizvesti seznam orodij. Upoštevajte, da mora vsako orodje imeti `name`, `description` in `inputSchema`.

**TypeScript**

Da nastavimo upravljalca zahtev za prikaz orodij, moramo na strežniku poklicati `setRequestHandler` z ustrezno shemo za naš namen, v tem primeru `ListToolsRequestSchema`.

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
// koda izpuščena zaradi jedrnatosti
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Vrni seznam registriranih orodij
  return {
    tools: tools
  };
});
```

Super, zdaj smo rešili prikaz orodij, poglejmo, kako lahko nato kličemo orodja.

### -4- Upravljanje klica orodja

Za klic orodja moramo nastaviti še enega upravljalca zahtev, tokrat osredotočenega na zahtevo, ki določa, katero funkcijo poklicati in s kakšnimi argumenti.

**Python**

Uporabimo dekorator `@server.call_tool` in ga implementiramo s funkcijo, na primer `handle_call_tool`. V tej funkciji moramo razčleniti ime orodja, njegov argument in zagotoviti, da so argumenti veljavni za dano orodje. Argumente lahko validiramo v tej funkciji ali kasneje v samem orodju.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je slovar z imeni orodij kot ključi
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # pokliči orodje
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Tukaj se dogaja naslednje:

- Ime našega orodja je že prisotno kot vhodni parameter `name`, kar velja tudi za argumente v obliki slovarja `arguments`.

- Orodje se kliče z `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Validacija argumentov poteka v lastnosti `handler`, ki kaže na funkcijo; če to ne uspe, sproži izjemo.

Tako, zdaj imamo popolno razumevanje prikaza in klica orodij z uporabo nizkonivojskega strežnika.

Oglejte si [popoln primer](./code/README.md) tukaj

## Naloga

Razširite dano kodo z več orodji, viri in pozivi ter razmislite, kako opazite, da morate dodajati datoteke samo v mapo tools in nikjer drugje.

*Rešitev ni podana*

## Povzetek

V tem poglavju smo videli, kako deluje pristop nizkonivojskega strežnika in kako nam lahko pomaga ustvariti lepo arhitekturo, na kateri lahko nadaljujemo z gradnjo. Prav tako smo razpravljali o validaciji in pokazali, kako delati z validacijskimi knjižnicami za ustvarjanje shem za validacijo vhodnih podatkov.

## Kaj sledi

- Naprej: [Preprosta avtentikacija](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Izjava o omejitvi odgovornosti**:
Ta dokument je bil preveden s pomočjo umetne inteligence preko storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v prvotnem jeziku velja za avtoritativni vir. Za ključne informacije priporočamo strokovni prevod s strani človeka. Nismo odgovorni za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
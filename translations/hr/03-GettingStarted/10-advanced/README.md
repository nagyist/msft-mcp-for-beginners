# Napredno korištenje servera

U MCP SDK postoje dva različita tipa servera, vaš obični server i niskorazinski server. Obično biste koristili redovni server za dodavanje značajki. Međutim, u nekim slučajevima želite se osloniti na niskorazinski server kao što su:

- Bolja arhitektura. Moguće je kreirati čistu arhitekturu i s redovnim serverom i niskorazinskim serverom, ali može se tvrditi da je nešto lakše s niskorazinskim serverom.
- Dostupnost značajki. Neke napredne značajke mogu se koristiti samo s niskorazinskim serverom. To ćete vidjeti u kasnijim poglavljima kada dodajemo uzorkovanje i izzivanje.

## Redovni server vs niskorazinski server

Evo kako izgleda stvaranje MCP Servera s redovnim serverom

**Python**

```python
mcp = FastMCP("Demo")

# Dodajte alat za zbrajanje
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

// Dodajte alat za zbrajanje
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

Poanta je da eksplicitno dodajete svaki alat, resurs ili prompt koji želite da server ima. Nema ništa loše u tome.

### Pristup niskorazinskog servera

Međutim, kad koristite pristup niskorazinskog servera, trebate razmišljati drugačije, naime umjesto registracije svakog alata stvarate dva handlera po tipu značajke (alat, resurs ili prompt). Dakle, na primjer, alati tada imaju samo dvije funkcije ovako:

- Nabrajanje svih alata. Jedna funkcija je odgovorna za sve pokušaje nabrajanja alata.
- Obrada poziva svih alata. Također, postoji samo jedna funkcija koja obrađuje poziv alata.

Zvui kao potencijalno manje posla, zar ne? Dakle, umjesto registracije alata, samo trebam osigurati da alat bude naveden kad nabrajam sve alate i da se pozove kad postoji dolazni zahtjev za poziv alata.

Pogledajmo kako sada izgleda kod:

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
  // Vraća popis registriranih alata
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

Ovdje sada imamo funkciju koja vraća listu značajki. Svaki unos u popisu alata sada ima polja poput `name`, `description` i `inputSchema` da odgovara tipu povratka. To nam omogućuje da naše alate i definiciju značajki stavimo negdje drugdje. Sada možemo stvoriti sve naše alate u mapi tools i isto vrijedi za sve vaše značajke pa vaš projekt može odjednom biti organiziran ovako:

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

Super, naša arhitektura može biti vrlo uredna.

A što je s pozivanjem alata, je li to ista ideja, jedan handler za pozivanje bilo kojeg alata? Da, točno, evo koda za to:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je rječnik s imenima alata kao ključevima
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
    // TODO pozvati alat,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Kao što vidite iz gornjeg koda, trebamo izdvojiti alat za pozivati i s kojim argumentima, a zatim nastaviti s pozivanjem alata.

## Poboljšanje pristupa s validacijom

Dosad ste vidjeli kako se sve registracije za dodavanje alata, resursa i promptova mogu zamijeniti s ta dva handlera po tipu značajke. Što još trebamo napraviti? Pa, trebali bismo dodati neki oblik validacije kako bismo osigurali da je alat pozvan s ispravnim argumentima. Svaki runtime ima vlastito rješenje za to, na primjer Python koristi Pydantic, a TypeScript koristi Zod. Ideja je da napravimo sljedeće:

- Premjestimo logiku za kreiranje značajke (alat, resurs ili prompt) u njegovu namjensku mapu.
- Dodamo način za validaciju dolaznog zahtjeva koji traži, na primjer, pozivanje alata.

### Kreiranje značajke

Da bismo kreirali značajku, morat ćemo napraviti datoteku za tu značajku i osigurati da sadrži obavezna polja zahtijevana za tu značajku. Koja se polja razlikuju između alata, resursa i promptova.

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
        # Validiraj unos koristeći Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: dodaj Pydantic, kako bismo mogli kreirati AddInputModel i validirati argumente

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ovdje možete vidjeti kako radimo sljedeće:

- Kreiramo shemu koristeći Pydantic `AddInputModel` s poljima `a` i `b` u datoteci *schema.py*.
- Pokušavamo parsirati dolazni zahtjev kao tip `AddInputModel`; ako postoji neslaganje u parametrima, to će uzrokovati grešku:

   ```python
   # add.py
    try:
        # Provjerite unos pomoću Pydantic modela
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Možete odabrati hoćete li ovu logiku parsiranja staviti u sam poziv alata ili u handler funkciju.

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

- U handleru koji obrađuje sve pozive alata sad pokušavamo parsirati dolazni zahtjev u shemu definiranu za alat:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

  ako to uspije, nastavljamo s pozivanjem stvarnog alata:

    ```typescript
    const result = await tool.callback(input);
    ```

Kao što vidite, ovaj pristup stvara izvrsnu arhitekturu jer sve ima svoje mjesto; *server.ts* je vrlo mala datoteka koja samo povezuje zahvalnice zahtjeva, a svaka značajka je u svojoj odgovarajućoj mapi tj. tools/, resources/ ili /prompts.

Odlično, pokušajmo to sada izgraditi.

## Vježba: Kreiranje niskorazinskog servera

U ovoj vježbi ćemo napraviti sljedeće:

1. Kreirati niskorazinski server koji obrađuje nabrajanje alata i pozivanje alata.
1. Implementirati arhitekturu na kojoj možete graditi dalje.
1. Dodati validaciju da osiguramo ispravno validirane pozive alata.

### -1- Kreiranje arhitekture

Prvo što trebamo riješiti je arhitektura koja nam pomaže da lako skaliramo dok dodajemo nove značajke, evo kako izgleda:

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

Sada smo postavili arhitekturu koja osigurava da možemo lako dodavati nove alate u mapu tools. Slobodno koristite ovaj pristup za dodavanje poddirektorija za resurse i promptove.

### -2- Kreiranje alata

Pogledajmo kako izgleda kreiranje alata. Prvo, mora biti napravljen u njegovom *tool* poddirektoriju ovako:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validirajte unos koristeći Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: dodajte Pydantic, tako da možemo napraviti AddInputModel i validirati argumente

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Ovdje vidimo kako definiramo ime, opis, ulaznu shemu koristeći Pydantic i handler koji će se pozvati kad se alat pozove. Na kraju izlažemo `tool_add` što je rječnik koji drži sve te karakteristike.

Tu je i *schema.py* koji se koristi za definiranje ulazne sheme koju naš alat koristi:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Također, moramo popuniti *__init__.py* kako bismo osigurali da se mapa tools tretira kao modul. Dodatno, moramo izložiti module unutar nje ovako:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

U ovu datoteku možemo nastaviti dodavati kako dodajemo nove alate.

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

Ovdje stvaramo rječnik koji se sastoji od svojstava:

- name, ime alata.
- rawSchema, to je Zod shema koja će se koristiti za validaciju dolaznih zahtjeva za pozivanje ovog alata.
- inputSchema, ova shema će se koristiti u handleru.
- callback, služi za pozivanje alata.

Postoji i `Tool` koji služi za pretvaranje ovog rječnika u tip koji MCP server handler može prihvatiti i izgleda ovako:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

I tu je *schema.ts* gdje pohranjujemo ulazne sheme za svaki alat koje trenutno izgleda ovako s jednom shemom, ali kako dodajemo alate možemo dodati više unosa:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Super, krenimo sada s obradom nabrajanja alata.

### -3- Obrada nabrajanja alata

Dalje, da bismo obradili nabrajanje alata, trebamo postaviti handler zahtjeva za to. Evo što trebamo dodati u našu server datoteku:

**Python**

```python
# kod izostavljen radi sažetosti
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

Ovdje dodajemo dekorator `@server.list_tools` i implementirajuću funkciju `handle_list_tools`. U potonjoj trebamo proizvesti popis alata. Primijetite da svaki alat mora imati ime, opis i ulaznu shemu.

**TypeScript**

Za postavljanje handlera zahtjeva za nabrajanje alata, potrebno je pozvati `setRequestHandler` na serveru s shemom koja odgovara onome što želimo, u ovom slučaju `ListToolsRequestSchema`.

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
// kod izostavljen radi sažetosti
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Vraća popis registriranih alata
  return {
    tools: tools
  };
});
```

Super, sada smo riješili dio nabrajanja alata, pogledajmo kako pozivati alate.

### -4- Obrada poziva alata

Da pozovemo alat, trebamo postaviti još jednog handlera zahtjeva, ovaj put fokusiranog na obradu zahtjeva koji specificira koju značajku pozvati i s kojim argumentima.

**Python**

Koristimo dekorator `@server.call_tool` i implementiramo ga funkcijom poput `handle_call_tool`. Unutar te funkcije trebamo izvaditi ime alata, njegove argumente i osigurati da su argumenti valjani za dotični alat. Validaciju argumenata možemo napraviti u ovoj funkciji ili dalje u samom alatu.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools je rječnik s imenima alata kao ključevima
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # pozovi alat
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Evo što se događa:

- Ime našeg alata već je prisutno kao ulazni parametar `name`, dok su argumenti u obliku rječnika `arguments`.
- Alat se poziva s `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Validacija argumenata događa se u `handler` svojstvu koje ukazuje na funkciju, ako to ne uspije, bit će podignuta iznimka.

Tu imamo potpuno razumijevanje nabrajanja i pozivanja alata koristeći niskorazinski server.

Pogledajte [cjeloviti primjer](./code/README.md) ovdje

## Zadatak

Proširite postojeći kod s nekoliko alata, resursa i promptova i razmislite kako primjećujete da trebate dodavati datoteke samo u direktorij tools, a nigdje drugo.

*Nema dane riješene verzije*

## Sažetak

U ovom poglavlju vidjeli smo kako pristup niskorazinskog servera funkcionira i kako nam može pomoći stvoriti lijepu arhitekturu na kojoj možemo dalje graditi. Također smo razgovarali o validaciji i pokazano vam je kako raditi s knjižnicama za validaciju da biste stvorili sheme za validaciju ulaza.

## Što je sljedeće

- Sljedeće: [Jednostavna autentifikacija](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Odricanje od odgovornosti**:
Ovaj dokument preveden je pomoću AI usluge za prijevod [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, molimo imajte na umu da automatski prijevodi mogu sadržavati pogreške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporučuje se profesionalni ljudski prijevod. Ne snosimo odgovornost za bilo kakva nerazumijevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
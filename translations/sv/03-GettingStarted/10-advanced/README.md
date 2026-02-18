# Avancerad serveranvändning

Det finns två olika typer av servrar som exponeras i MCP SDK, din vanliga server och low-level servern. Normalt använder du den vanliga servern för att lägga till funktioner i den. För vissa fall vill du dock förlita dig på low-level servern såsom:

- Bättre arkitektur. Det är möjligt att skapa en ren arkitektur med både den vanliga servern och en low-level server men det kan hävdas att det är något enklare med en low-level server.
- Funktionstillgång. Vissa avancerade funktioner kan endast användas med en low-level server. Du kommer att se detta i senare kapitel när vi lägger till sampling och elicitation.

## Vanlig server vs low-level server

Så här ser skapandet av en MCP Server ut med den vanliga servern

**Python**

```python
mcp = FastMCP("Demo")

# Lägg till ett additionsverktyg
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

// Lägg till ett additionsverktyg
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

Poängen är att du explicit lägger till varje verktyg, resurs eller prompt som du vill att servern ska ha. Inget fel med det.

### Low-level server-ansatsen

När du använder low-level server-ansatsen måste du tänka annorlunda, nämligen att istället för att registrera varje verktyg skapar du istället två hanterare per funktionstyp (verktyg, resurser eller prompts). Så till exempel har verktyg då bara två funktioner som så här:

- Lista alla verktyg. En funktion skulle vara ansvarig för alla försök att lista verktyg.
- Hantera anrop till alla verktyg. Här finns också bara en funktion som hanterar anrop till ett verktyg.

Det låter som potentiellt mindre arbete, eller hur? Så istället för att registrera ett verktyg behöver jag bara säkerställa att verktyget listas när jag listar alla verktyg och att det anropas när det finns en inkommande förfrågan att anropa ett verktyg.

Låt oss titta på hur koden ser ut nu:

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
  // Returnera listan över registrerade verktyg
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

Här har vi nu en funktion som returnerar en lista av funktioner. Varje post i verktygslistan har nu fält som `name`, `description` och `inputSchema` för att följa returtypen. Detta gör att vi kan lägga våra verktyg och funktionsdefinitioner på annat håll. Vi kan nu skapa alla våra verktyg i en verktygsmapp och samma gäller alla dina funktioner så att ditt projekt plötsligt kan organiseras så här:

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

Det är toppen, vår arkitektur kan göras ganska ren.

Hur är det med att anropa verktyg, är det samma idé då, en hanterare för att anropa ett verktyg, vilket verktyg som helst? Ja, exakt, här är koden för det:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools är en ordlista med verktygsnamn som nycklar
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
    // TODO anropa verktyget,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Som du kan se från ovanstående kod måste vi parsa ut vilket verktyg som ska anropas, och med vilka argument, och sedan fortsätta med att anropa verktyget.

## Förbättra ansatsen med validering

Hittills har du sett hur alla dina registreringar för att lägga till verktyg, resurser och prompts kan ersättas med dessa två hanterare per funktionstyp. Vad mer behöver vi göra? Jo, vi bör lägga till någon form av validering för att säkerställa att verktyget anropas med rätt argument. Varje runtime har sin egen lösning för detta, exempelvis använder Python Pydantic och TypeScript använder Zod. Idén är att vi gör följande:

- Flytta logiken för att skapa en funktion (verktyg, resurs eller prompt) till dess dedikerade mapp.
- Lägg till ett sätt att validera en inkommande förfrågan som till exempel begär att anropa ett verktyg.

### Skapa en funktion

För att skapa en funktion behöver vi skapa en fil för den funktionen och säkerställa att den har de obligatoriska fälten som krävs för den funktionen. Vilka fält som krävs skiljer sig lite mellan verktyg, resurser och prompts.

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
        # Validera indata med hjälp av Pydantic-modell
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: lägg till Pydantic, så att vi kan skapa en AddInputModel och validera argumenten

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

här kan du se hur vi gör följande:

- Skapar ett schema med Pydantic `AddInputModel` med fälten `a` och `b` i filen *schema.py*.
- Försöker parsa den inkommande förfrågan till typen `AddInputModel`, om det finns en avvikelse i parametrarna kraschar detta:

   ```python
   # add.py
    try:
        # Validera indata med Pydantic-modell
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Du kan välja att lägga denna parsningslogik i själva verktygsanropet eller i hanteringsfunktionen.

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

- I hanteraren som hanterar alla verktygsanrop försöker vi nu parsa den inkommande förfrågan till verktygets definierade schema:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    om det lyckas fortsätter vi att anropa det faktiska verktyget:

    ```typescript
    const result = await tool.callback(input);
    ```

Som du ser skapar detta angreppssätt en bra arkitektur eftersom allt har sin plats, *server.ts* är en mycket liten fil som bara kopplar upp förfrågningshanterare och varje funktion finns i respektive mapp dvs verktyg/, resurser/ eller /prompts.

Bra, låt oss försöka bygga detta härnäst.

## Övning: Skapa en low-level server

I denna övning ska vi göra följande:

1. Skapa en low-level server som hanterar listning av verktyg och anrop av verktyg.
1. Implementera en arkitektur du kan bygga vidare på.
1. Lägg till validering för att säkerställa att dina verktygsanrop är korrekt validerade.

### -1- Skapa en arkitektur

Det första vi behöver ta itu med är en arkitektur som hjälper oss att skala när vi lägger till fler funktioner, så här ser den ut:

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

Nu har vi satt upp en arkitektur som säkerställer att vi enkelt kan lägga till nya verktyg i en verktygsmapp. Känn dig fri att följa detta för att lägga till undermappar för resurser och prompts.

### -2- Skapa ett verktyg

Låt oss se hur det ser ut att skapa ett verktyg härnäst. Först måste det skapas i dess *tool*-undermapp så här:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validera indata med Pydantic-modell
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: lägg till Pydantic, så att vi kan skapa en AddInputModel och validera argumenten

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Vad vi ser här är hur vi definierar namn, beskrivning, ett inmatningsschema med Pydantic och en hanterare som kommer att anropas när detta verktyg kallas. Slutligen exponerar vi `tool_add` som är en dictionary som innehåller alla dessa egenskaper.

Det finns också *schema.py* som används för att definiera inmatningsschemat som vårt verktyg använder:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Vi måste också fylla på *__init__.py* för att säkerställa att verktygsmappen behandlas som en modul. Dessutom måste vi exponera modulerna inom den så här:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Vi kan fortsätta lägga till i denna fil när vi lägger till fler verktyg.

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

Här skapar vi en dictionary bestående av egenskaper:

- name, detta är namnet på verktyget.
- rawSchema, detta är Zod-schemat, det kommer att användas för att validera inkommande förfrågningar att anropa detta verktyg.
- inputSchema, detta schema kommer att användas av hanteraren.
- callback, detta används för att anropa verktyget.

Det finns också `Tool` som används för att omvandla denna dictionary till en typ som mcp-serverhanteraren kan acceptera och det ser ut så här:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Och det finns *schema.ts* där vi lagrar inmatningsscheman för varje verktyg som ser ut så här med endast ett schema för tillfället men när vi lägger till verktyg kan vi lägga till fler poster:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Toppen, låt oss gå vidare till att hantera listning av våra verktyg härnäst.

### -3- Hantera listning av verktyg

Nästa steg, för att hantera listning av våra verktyg, behöver vi ställa in en förfrågningshanterare för det. Så här behöver vi lägga till i vår serverfil:

**Python**

```python
# kod utelämnad för korthet
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

Här lägger vi till dekoratorn `@server.list_tools` och implementerande funktionen `handle_list_tools`. I den senare måste vi producera en lista med verktyg. Observera hur varje verktyg måste ha namn, beskrivning och inputSchema.

**TypeScript**

För att ställa in förfrågningshanteraren för att lista verktyg behöver vi anropa `setRequestHandler` på servern med ett schema som passar det vi vill göra, i detta fall `ListToolsRequestSchema`.

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
// kod utelämnad för korthet
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Returnera listan över registrerade verktyg
  return {
    tools: tools
  };
});
```

Bra, nu har vi löst biten för att lista verktyg, låt oss titta på hur vi kan anropa verktyg härnäst.

### -4- Hantera anrop av ett verktyg

För att anropa ett verktyg behöver vi ställa in en annan förfrågningshanterare, den här gången med fokus på att hantera en förfrågan som specificerar vilken funktion som ska anropas och med vilka argument.

**Python**

Låt oss använda dekoratorn `@server.call_tool` och implementera den med en funktion som `handle_call_tool`. Inom den funktionen måste vi parsa ut verktygsnamnet, dess argument och säkerställa att argumenten är giltiga för det verktyg det gäller. Vi kan antingen validera argumenten i denna funktion eller längre ner i det faktiska verktyget.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools är en ordlista med verktygsnamn som nycklar
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # anropa verktyget
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Så här går det till:

- Vårt verktygsnamn finns redan som inparameter `name` vilket också gäller för våra argument i form av `arguments` dictionaryn.

- Verktyget anropas med `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Valideringen av argumenten sker i egenskapen `handler` som pekar på en funktion, om detta misslyckas kastas ett undantag.

Där har vi det, nu har vi en full förståelse för hur man listar och anropar verktyg med en low-level server.

Se det [fullständiga exemplet](./code/README.md) här

## Uppgift

Utöka den kod du fått med ett antal verktyg, resurser och prompts och reflektera över hur du märker att du bara behöver lägga till filer i verktygsmappen och ingen annanstans.

*Ingen lösning finns*

## Sammanfattning

I detta kapitel såg vi hur low-level server-ansatsen fungerade och hur den kan hjälpa oss skapa en trevlig arkitektur som vi kan fortsätta bygga på. Vi diskuterade också validering och du visades hur man arbetar med valideringsbibliotek för att skapa scheman för inmatningsvalidering.

## Vad kommer härnäst

- Nästa: [Enkel autentisering](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Trots att vi strävar efter noggrannhet kan automatiska översättningar innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För viktig information rekommenderas professionell översättning av människa. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår från användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# Geavanceerd servergebruik

Er zijn twee verschillende typen servers die worden aangeboden in de MCP SDK, je normale server en de low-level server. Normaal gebruik je de reguliere server om er functies aan toe te voegen. In sommige gevallen wil je echter vertrouwen op de low-level server, bijvoorbeeld:

- Betere architectuur. Het is mogelijk om een nette architectuur te creëren met zowel de reguliere server als een low-level server, maar er kan worden betoogd dat het iets makkelijker is met een low-level server.
- Beschikbaarheid van functies. Sommige geavanceerde functies kunnen alleen worden gebruikt met een low-level server. Je zult dit in latere hoofdstukken zien als we sampling en elicitation toevoegen.

## Reguliere server versus low-level server

Zo ziet het aanmaken van een MCP Server eruit met de reguliere server

**Python**

```python
mcp = FastMCP("Demo")

# Voeg een toevoegtool toe
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

// Voeg een toevoegingshulpmiddel toe
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

Het punt is dat je expliciet elk hulpmiddel, resource of prompt toevoegt die je wilt dat de server heeft. Daar is niets mis mee.

### Low-level server benadering

Echter, wanneer je de low-level server benadering gebruikt, moet je er anders over nadenken, namelijk dat je in plaats van elk hulpmiddel te registreren, je twee handlers maakt per functietype (tools, resources of prompts). Dus bijvoorbeeld tools hebben dan slechts twee functies zoals:

- Alle tools op een rij zetten. Eén functie is verantwoordelijk voor alle pogingen om tools op te sommen.
- Alle calls naar tools afhandelen. Ook hier is er slechts één functie die calls naar een tool afhandelt.

Dat klinkt als potentieel minder werk toch? Dus in plaats van een tool te registreren, hoef ik alleen maar te zorgen dat de tool vermeld staat als ik alle tools opsom, en dat hij wordt aangeroepen als er een inkomend verzoek is om een tool aan te roepen.

Laten we eens kijken hoe de code er nu uitziet:

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
  // Geef de lijst van geregistreerde hulpmiddelen terug
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

Hier hebben we nu een functie die een lijst met functies teruggeeft. Elke invoer in de tools-lijst heeft nu velden als `name`, `description` en `inputSchema` om te voldoen aan het teruggeeftype. Hierdoor kunnen we onze tools en functiedefinities ergens anders plaatsen. We kunnen nu al onze tools aanmaken in een map tools en hetzelfde geldt voor al je features, zodat je project opeens zo georganiseerd kan worden:

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

Dat is geweldig, onze architectuur kan er nu erg clean uitzien.

Wat betreft het aanroepen van tools, is het dan hetzelfde idee, één handler die een tool aanroept, welke tool dan ook? Ja, precies, hier is de code daarvoor:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools is een woordenboek met toolnamen als sleutels
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
    // TODO roep het hulpmiddel aan,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Zoals je kunt zien aan de bovenstaande code, moeten we de tool die we willen aanroepen en de argumenten ervan parsen, en vervolgens de tool aanroepen.

## Verbeteren van de aanpak met validatie

Tot nu toe heb je gezien hoe al je registraties om tools, resources en prompts toe te voegen kunnen worden vervangen door deze twee handlers per functietype. Wat moeten we verder nog doen? Nou, we zouden een vorm van validatie moeten toevoegen om er zeker van te zijn dat een tool wordt aangeroepen met de juiste argumenten. Elke runtime heeft hiervoor zijn eigen oplossing, bijvoorbeeld Python gebruikt Pydantic en TypeScript gebruikt Zod. Het idee is dat we het volgende doen:

- Verplaats de logica voor het maken van een feature (tool, resource of prompt) naar een eigen map.
- Voeg een manier toe om een inkomend verzoek te valideren, bijvoorbeeld om een tool aan te roepen.

### Creëer een feature

Om een feature aan te maken, moeten we een bestand maken voor die feature en ervoor zorgen dat het de verplichte velden voor die feature heeft. Welke velden verschillen enigszins tussen tools, resources en prompts.

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
        # Valideer invoer met behulp van Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: voeg Pydantic toe, zodat we een AddInputModel kunnen maken en args kunnen valideren

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

hier zie je hoe we het volgende doen:

- Een schema maken met Pydantic `AddInputModel` met velden `a` en `b` in het bestand *schema.py*.
- Proberen het inkomende verzoek te parsen naar het type `AddInputModel`, als er een mismatch in parameters is zal dit crashen:

   ```python
   # add.py
    try:
        # Valideer invoer met behulp van Pydantic-model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Je kunt ervoor kiezen deze parsing-logica in de tool call zelf te plaatsen of in de handler functie.

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

- In de handler die alle tool calls afhandelt, proberen we nu het inkomende verzoek te parsen naar het door de tool gedefinieerde schema:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    als dat lukt gaan we door om de daadwerkelijke tool aan te roepen:

    ```typescript
    const result = await tool.callback(input);
    ```

Zoals je ziet, creëert deze aanpak een mooie architectuur omdat alles zijn plek heeft, het *server.ts* is een heel klein bestand dat alleen request handlers aansluit en elke feature zit in zijn respectievelijke map, d.w.z tools/, resources/ of /prompts.

Geweldig, laten we dit nu gaan bouwen.

## Oefening: Een low-level server maken

In deze oefening doen we het volgende:

1. Creëer een low-level server die het opsommen van tools en het aanroepen van tools afhandelt.
1. Implementeer een architectuur waarop je verder kunt bouwen.
1. Voeg validatie toe om ervoor te zorgen dat je tool calls correct gevalideerd worden.

### -1- Creëer een architectuur

Het eerste wat we moeten doen is een architectuur opzetten die ons helpt op te schalen als we meer features toevoegen, dit is hoe het eruitziet:

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

Nu hebben we een architectuur opgezet die ervoor zorgt dat we gemakkelijk nieuwe tools kunnen toevoegen in een tools-map. Voel je vrij om dit te volgen om submappen voor resources en prompts toe te voegen.

### -2- Een tool maken

Laten we nu zien hoe het maken van een tool eruitziet. Eerst moet deze worden aangemaakt in de submap *tool* zo:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Valideer invoer met behulp van Pydantic-model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: voeg Pydantic toe, zodat we een AddInputModel kunnen maken en args kunnen valideren

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Wat we hier zien is hoe we naam, beschrijving, een input schema maken met Pydantic en een handler die wordt aangeroepen zodra deze tool wordt aangeroepen. Tot slot exposen we `tool_add` die een dictionary is met al deze eigenschappen.

Er is ook *schema.py* dat gebruikt wordt om het input schema te definiëren dat door onze tool wordt gebruikt:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

We moeten ook *__init__.py* vullen om ervoor te zorgen dat de tools-map als een module wordt behandeld. Daarnaast moeten we de modules erin exposen zoals:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

We kunnen dit bestand blijven uitbreiden naarmate we meer tools toevoegen.

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

Hier maken we een dictionary bestaande uit eigenschappen:

- name, dit is de naam van de tool.
- rawSchema, dit is het Zod-schema, het wordt gebruikt voor validatie van inkomende verzoeken om deze tool aan te roepen.
- inputSchema, dit schema wordt gebruikt door de handler.
- callback, dit wordt gebruikt om de tool aan te roepen.

Er is ook `Tool` dat wordt gebruikt om deze dictionary om te zetten naar een type dat de mcp server handler kan accepteren en het ziet er zo uit:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

En er is *schema.ts* waar we de input schema's voor elke tool opslaan, er is er nu één maar als we meer tools toevoegen kunnen we meer items toevoegen:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Geweldig, laten we verder gaan met het afhandelen van het opsommen van onze tools.

### -3- Tools opsommen afhandelen

Vervolgens, om het opsommen van onze tools af te handelen, moeten we een request handler instellen hiervoor. Dit moeten we aan ons serverbestand toevoegen:

**Python**

```python
# code weggelaten voor de overzichtelijkheid
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

Hier voegen we de decorator `@server.list_tools` toe en de implementerende functie `handle_list_tools`. In die functie moeten we een lijst met tools produceren. Let op dat elke tool een naam, beschrijving en inputSchema moet hebben.

**TypeScript**

Om de request handler voor tools opsommen in te stellen, moeten we `setRequestHandler` aanroepen op de server met een schema dat past bij wat we willen doen, in dit geval `ListToolsRequestSchema`.

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
// code weggelaten voor beknoptheid
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Geef de lijst met geregistreerde tools terug
  return {
    tools: tools
  };
});
```

Geweldig, nu hebben we het stuk voor tools opsommen opgelost, laten we kijken hoe we tools kunnen aanroepen.

### -4- Een tool aanroepen afhandelen

Om een tool aan te roepen, moeten we een andere request handler instellen, gericht op een verzoek dat specificeert welke feature moet worden aangeroepen en met welke argumenten.

**Python**

Laten we de decorator `@server.call_tool` gebruiken en implementeren met een functie als `handle_call_tool`. In die functie moeten we de toolnaam en argumenten parsen en zorgen dat de argumenten geldig zijn voor de betreffende tool. We kunnen de validatie hier doen of downstream in de tool zelf.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools is een woordenboek met toolnamen als sleutels
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # roep de tool aan
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Dit gebeurt er:

- Onze toolnaam is al aanwezig als inputparameter `name` wat ook geldt voor onze argumenten in de vorm van de `arguments` dictionary.

- De tool wordt aangeroepen met `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. De validatie van de argumenten gebeurt in de `handler` property die naar een functie wijst, als die faalt wordt een uitzondering opgegooid.

Daarmee hebben we een volledig begrip van hoe tools worden opgesomd en aangeroepen met een low-level server.

Zie het [volledige voorbeeld](./code/README.md) hier

## Opdracht

Breid de code die je hebt gekregen uit met een aantal tools, resources en prompts en reflecteer op hoe je merkt dat je alleen bestanden hoeft toe te voegen in de tools-directory en nergens anders.

*Geen oplossing gegeven*

## Samenvatting

In dit hoofdstuk hebben we gezien hoe de low-level server aanpak werkt en hoe dat ons kan helpen een mooie architectuur te creëren waar we op voort kunnen bouwen. We hebben ook validatie besproken en je hebt gezien hoe je met validatiebibliotheken schemas maakt voor inputvalidatie.

## Wat volgt

- Volgende: [Eenvoudige authenticatie](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:  
Dit document is vertaald met behulp van de AI-vertalingsdienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onjuistheden kunnen bevatten. De oorspronkelijke versie van het document in de oorspronkelijke taal wordt beschouwd als de gezaghebbende bron. Voor belangrijke informatie wordt een professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor enige misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
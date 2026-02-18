# Utilizare avansată a serverului

Există două tipuri diferite de servere expuse în MCP SDK, serverul normal și serverul de nivel jos. În mod normal, ai folosi serverul obișnuit pentru a adăuga funcționalități. Totuși, în unele cazuri, dorești să te bazezi pe serverul de nivel jos, cum ar fi:

- Arhitectură mai bună. Este posibil să creezi o arhitectură curată atât cu serverul obișnuit, cât și cu un server de nivel jos, însă se poate susține că este puțin mai ușor cu un server de nivel jos.
- Disponibilitate a funcționalităților. Unele funcționalități avansate pot fi folosite doar cu un server de nivel jos. Vei vedea asta în capitolele următoare când vom adăuga eșantionare și elicitație.

## Server obișnuit vs server de nivel jos

Iată cum arată crearea unui MCP Server cu serverul obișnuit

**Python**

```python
mcp = FastMCP("Demo")

# Adăugați un instrument de adunare
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

// Adaugă un instrument de adunare
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

Punctul este că adaugi în mod explicit fiecare instrument, resursă sau prompt pe care vrei să îl aibă serverul. Nu e nimic greșit în asta.  

### Abordarea serverului de nivel jos

Totuși, când folosești abordarea serverului de nivel jos, trebuie să gândești diferit, anume că în loc să înregistrezi fiecare instrument creezi două handler-e pentru fiecare tip de funcționalitate (instrumente, resurse sau prompturi). De exemplu, instrumentele au doar două funcții, astfel:

- Listarea tuturor instrumentelor. O funcție este responsabilă pentru toate încercările de a lista instrumentele.
- gestionarea apelării tuturor instrumentelor. Și aici, este o singură funcție care gestionează apelurile către un instrument.

Sună a muncă mai puțină, nu? Deci în loc să înregistrezi un instrument, trebuie doar să te asiguri că instrumentul este listat când listezi toate instrumentele și că este apelat când vine o cerere pentru a apela un instrument.

Să vedem cum arată acum codul:

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
  // Returnează lista instrumentelor înregistrate
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

Aici avem acum o funcție care returnează o listă de funcționalități. Fiecare intrare din lista de instrumente are acum câmpuri ca `name`, `description` și `inputSchema` pentru a respecta tipul returnat. Acest lucru ne permite să punem definițiile instrumentelor și funcționalităților în altă parte. Putem crea toate instrumentele într-un dosar tools și la fel pentru toate funcționalitățile, astfel încât proiectul poate fi organizat astfel:

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

E grozav, arhitectura noastră poate fi făcută să arate destul de curat.

Cât despre apelarea instrumentelor, este aceeași idee, un handler pentru a apela un instrument, oricare ar fi acela? Da, exact, iată codul pentru asta:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools este un dicționar cu numele instrumentelor ca chei
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
    // TODO apelează instrumentul,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Așa cum vezi în codul de mai sus, trebuie să extragem instrumentul de apelat și cu ce argumente, apoi trebuie să continuăm apelarea instrumentului.

## Îmbunătățirea abordării cu validare

Până acum, ai văzut cum toate înregistrările tale pentru a adăuga instrumente, resurse și prompturi pot fi înlocuite cu aceste două handler-e pentru fiecare tip de funcționalitate. Ce altceva trebuie să facem? Ei bine, ar trebui să adăugăm o formă de validare pentru a ne asigura că instrumentul este apelat cu argumentele corecte. Fiecare runtime are propria soluție pentru asta, de exemplu Python folosește Pydantic și TypeScript folosește Zod. Ideea este că facem următoarele:

- Mutăm logica pentru crearea unei funcționalități (instrument, resursă sau prompt) în folderul dedicat.
- Adăugăm o metodă de validare a unei cereri de intrare care solicită, spre exemplu, apelarea unui instrument.

### Crearea unei funcționalități

Pentru a crea o funcționalitate, va trebui să creezi un fișier pentru acea funcționalitate și să te asiguri că are câmpurile obligatorii cerute de acea funcționalitate. Câmpurile diferă puțin între instrumente, resurse și prompturi.

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
        # Validează intrarea folosind modelul Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: adaugă Pydantic, astfel încât să putem crea un AddInputModel și să validăm argumentele

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

aici poți vedea cum facem următoarele:

- Creăm un schema folosind Pydantic `AddInputModel` cu câmpurile `a` și `b` în fișierul *schema.py*.
- Încercăm să parse-ăm cererea de intrare să fie de tip `AddInputModel`; dacă există o nepotrivire în parametri, se va produce o eroare:

   ```python
   # add.py
    try:
        # Validează intrarea folosind modelul Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Poți alege să pui această logică de parcurgere fie în apelul instrumentului însuși, fie în funcția handler.

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

- În handlerul care se ocupă de toate apelurile instrumentelor, încercăm să parse-ăm cererea de intrare în schema definită a instrumentului:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    Dacă merge, atunci trecem la apelarea instrumentului efectiv:

    ```typescript
    const result = await tool.callback(input);
    ```

După cum vezi, această abordare creează o arhitectură grozavă deoarece totul are locul său, fișierul *server.ts* este foarte mic și se ocupă doar de conectarea handler-elor de cereri, iar fiecare funcționalitate este în propriul său folder, adică tools/, resources/ sau /prompts.

Groaznic, hai să încercăm să construim asta în continuare.

## Exercițiu: Crearea unui server de nivel jos

În acest exercițiu, vom face următoarele:

1. Creăm un server de nivel jos care se ocupă de listarea instrumentelor și apelarea instrumentelor.
1. Implementăm o arhitectură pe care o poți extinde.
1. Adăugăm validare pentru a ne asigura că apelurile la instrumentele tale sunt validate corect.

### -1- Crearea unei arhitecturi

Primul lucru la care trebuie să ne gândim este o arhitectură care să ne ajute să scalăm pe măsură ce adăugăm mai multe funcționalități, iată cum arată:

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

Acum am configurat o arhitectură care ne permite să adăugăm cu ușurință noi instrumente într-un folder tools. Te poți simți liber să adaugi subdirectoare pentru resources și prompts.

### -2- Crearea unui instrument

Să vedem cum arată crearea unui instrument mai departe. În primul rând, trebuie creat în subdirectorul *tool* astfel:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validează intrarea folosind modelul Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: adaugă Pydantic, astfel încât să putem crea un AddInputModel și să validăm argumentele

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Ce vedem aici este cum definim numele, descrierea, un schema de intrare folosind Pydantic și un handler care va fi invocat când acest instrument este apelat. În final, expunem `tool_add` care este un dicționar ce conține toate aceste proprietăți.

Mai există și *schema.py* folosit pentru a defini schema de intrare folosită de instrumentul nostru:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

De asemenea, trebuie să populăm fișierul *__init__.py* pentru a ne asigura că directorul tools este tratat ca un modul. În plus, trebuie să expunem modulele din el astfel:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Putem continua să adăugăm în acest fișier pe măsură ce adăugăm mai multe instrumente.

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

Aici creăm un dicționar format din proprietăți:

- name, acesta este numele instrumentului.
- rawSchema, acesta este schema Zod, care va fi folosită pentru a valida cererile de intrare pentru apelarea acestui instrument.
- inputSchema, această schemă va fi folosită de handler.
- callback, acesta este folosit pentru a invoca instrumentul.

Există și `Tool` care este folosit pentru a converti acest dicționar într-un tip acceptabil de handler-ul mcp server și arată astfel:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Și există *schema.ts* unde stocăm schemele de intrare pentru fiecare instrument care arată astfel, având momentan o singură schemă, dar pe măsură ce adăugăm instrumente vom adăuga mai multe intrări:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Groaznic, să trecem să gestionăm listarea instrumentelor noastre mai departe.

### -3- Gestionarea listării instrumentelor

Următorul pas, pentru a gestiona listarea instrumentelor, trebuie să configurăm un handler de cereri pentru asta. Iată ce trebuie să adăugăm în fișierul nostru server:

**Python**

```python
# cod omis pentru concizie
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

Aici adăugăm decoratorul `@server.list_tools` și funcția de implementare `handle_list_tools`. În aceasta, trebuie să producem o listă de instrumente. Observă cum fiecare instrument trebuie să aibă un nume, descriere și inputSchema.   

**TypeScript**

Pentru a configura handler-ul de cereri pentru listarea instrumentelor, trebuie să apelăm `setRequestHandler` pe server cu o schemă care să corespundă ceea ce încercăm să facem, în acest caz `ListToolsRequestSchema`. 

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
// cod omis pentru concizie
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Returnează lista uneltelor înregistrate
  return {
    tools: tools
  };
});
```

Groaznic, acum am rezolvat partea de listare a instrumentelor, să vedem cum am putea apela instrumentele mai departe.

### -4- Gestionarea apelării unui instrument

Pentru a apela un instrument, trebuie să configurăm un alt handler de cereri, de data aceasta concentrat pe gestionarea unei cereri care specifică ce funcționalitate să fie apelată și cu ce argumente.

**Python**

Să folosim decoratorul `@server.call_tool` și să îl implementăm cu o funcție ca `handle_call_tool`. În interiorul acelei funcții trebuie să extragem numele instrumentului, argumentul său și să asigurăm că argumentele sunt valide pentru instrumentul respectiv. Putem valida argumentele în această funcție sau în aval, în instrumentul real.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools este un dicționar cu numele uneltelor ca chei
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # apelează unealta
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Iată ce se întâmplă:

- Numele instrumentului este deja prezent ca și parametru de intrare `name`, iar argumentele noastre sunt în forma dicționarului `arguments`.

- Instrumentul este apelat cu `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Validarea argumentelor are loc în proprietatea `handler` care indică o funcție; dacă aceasta eșuează se aruncă o excepție. 

Gata, acum avem o înțelegere completă despre listarea și apelarea instrumentelor folosind un server de nivel jos.

Vezi [exemplul complet](./code/README.md) aici

## Tema

Extinde codul pe care l-ai primit cu un număr de instrumente, resurse și prompturi și reflectează asupra modului în care observi că trebuie să adaugi doar fișiere în directorul tools și nicăieri altundeva.

*Nu se oferă soluție*

## Rezumat

În acest capitol, am văzut cum funcționează abordarea serverului de nivel jos și cum aceasta ne poate ajuta să creăm o arhitectură frumoasă pe care putem continua să construim. Am discutat și despre validare și ți s-a arătat cum să lucrezi cu biblioteci de validare pentru a crea scheme pentru validarea intrării.

## Ce urmează

- Următorul: [Autentificare simplă](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare de responsabilitate**:  
Acest document a fost tradus folosind serviciul de traducere automată AI [Co-op Translator](https://github.com/Azure/co-op-translator). Deși ne străduim pentru acuratețe, vă rugăm să țineți cont că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa oficială. Pentru informații critice, se recomandă traducerea profesională efectuată de un specialist uman. Nu ne asumăm răspunderea pentru eventualele neînțelegeri sau interpretări greșite care pot rezulta din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
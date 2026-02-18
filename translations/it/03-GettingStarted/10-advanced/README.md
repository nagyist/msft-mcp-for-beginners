# Uso avanzato del server

Ci sono due diversi tipi di server esposti nell'MCP SDK, il tuo server normale e il server a basso livello. Normalmente, useresti il server regolare per aggiungere funzionalità. Tuttavia, in alcuni casi, vuoi fare affidamento sul server a basso livello come:

- Architettura migliore. È possibile creare un'architettura pulita sia con il server regolare che con un server a basso livello, ma si può sostenere che sia leggermente più facile con un server a basso livello.
- Disponibilità delle funzionalità. Alcune funzionalità avanzate possono essere utilizzate solo con un server a basso livello. Lo vedrai nei capitoli successivi mentre aggiungiamo campionamento e elicitazione.

## Server regolare vs server a basso livello

Ecco cosa sembra la creazione di un MCP Server con il server regolare

**Python**

```python
mcp = FastMCP("Demo")

# Aggiungi uno strumento di addizione
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

// Aggiungi uno strumento di addizione
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

Il punto è che aggiungi esplicitamente ogni strumento, risorsa o prompt che vuoi che il server abbia. Non c'è niente di male in questo.

### Approccio server a basso livello

Tuttavia, quando usi l'approccio del server a basso livello devi pensare in modo diverso, cioè invece di registrare ogni strumento, crei due gestori per tipo di funzionalità (strumenti, risorse o prompt). Quindi, ad esempio, gli strumenti avranno solo due funzioni così:

- Elencare tutti gli strumenti. Una funzione sarebbe responsabile di tutti i tentativi di elencare gli strumenti.
- Gestire le chiamate a tutti gli strumenti. Qui, inoltre, c'è solo una funzione che gestisce le chiamate a uno strumento.

Sembra potenzialmente meno lavoro, giusto? Quindi invece di registrare uno strumento, devo solo assicurarmi che lo strumento sia elencato quando elenco tutti gli strumenti e che sia chiamato quando arriva una richiesta per chiamare uno strumento.

Diamo un'occhiata a come appare ora il codice:

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
  // Restituisce la lista degli strumenti registrati
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

Qui abbiamo ora una funzione che restituisce una lista di funzionalità. Ogni voce nell'elenco degli strumenti ora ha campi come `name`, `description` e `inputSchema` per aderire al tipo di ritorno. Questo ci permette di mettere i nostri strumenti e la definizione delle funzionalità altrove. Ora possiamo creare tutti i nostri strumenti in una cartella tools e la stessa cosa vale per tutte le tue funzionalità, così il tuo progetto può improvvisamente essere organizzato così:

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

Ottimo, la nostra architettura può essere resa abbastanza pulita.

E per quanto riguarda la chiamata degli strumenti, è la stessa idea allora, un gestore per chiamare uno strumento, qualunque strumento? Sì, esatto, ecco il codice per questo:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools è un dizionario con i nomi degli strumenti come chiavi
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
    // TODO chiamare lo strumento,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Come puoi vedere dal codice sopra, dobbiamo analizzare lo strumento da chiamare, e con quali argomenti, quindi dobbiamo procedere a chiamare lo strumento.

## Migliorare l'approccio con la validazione

Finora, hai visto come tutte le tue registrazioni per aggiungere strumenti, risorse e prompt possono essere sostituite con questi due gestori per tipo di funzionalità. Cos'altro dobbiamo fare? Beh, dovremmo aggiungere una forma di validazione per assicurarci che lo strumento venga chiamato con gli argomenti corretti. Ogni runtime ha la propria soluzione per questo, ad esempio Python usa Pydantic e TypeScript usa Zod. L'idea è che facciamo quanto segue:

- Spostare la logica per creare una funzionalità (strumento, risorsa o prompt) nella sua cartella dedicata.
- Aggiungere un modo per convalidare una richiesta in arrivo che chiede ad esempio di chiamare uno strumento.

### Creare una funzionalità

Per creare una funzionalità, dobbiamo creare un file per quella funzionalità e assicurarci che abbia i campi obbligatori richiesti da quella funzionalità. I campi differiscono un po' tra strumenti, risorse e prompt.

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
        # Valida l'input usando il modello Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: aggiungere Pydantic, così possiamo creare un AddInputModel e validare gli argomenti

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Qui puoi vedere come facciamo quanto segue:

- Creiamo uno schema usando Pydantic `AddInputModel` con campi `a` e `b` nel file *schema.py*.
- Tentiamo di analizzare la richiesta in arrivo per essere di tipo `AddInputModel`; se c'è una discrepanza nei parametri questo fallirà:

   ```python
   # add.py
    try:
        # Valida l'input usando il modello Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Puoi scegliere se mettere questa logica di parsing nella chiamata dello strumento stessa o nella funzione gestore.

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

- Nel gestore che si occupa di tutte le chiamate agli strumenti, ora cerchiamo di analizzare la richiesta in arrivo nello schema definito dallo strumento:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    se questo funziona allora procediamo a chiamare lo strumento reale:

    ```typescript
    const result = await tool.callback(input);
    ```

Come puoi vedere, questo approccio crea una grande architettura poiché tutto ha il suo posto, il *server.ts* è un file molto piccolo che si occupa solo di collegare i gestori di richieste e ogni funzionalità è nella rispettiva cartella cioè tools/, resources/ o /prompts.

Ottimo, proviamo a costruirlo adesso.

## Esercizio: Creare un server a basso livello

In questo esercizio, faremo quanto segue:

1. Creare un server a basso livello che gestisca l'elenco degli strumenti e la chiamata degli strumenti.
1. Implementare un'architettura su cui puoi costruire.
1. Aggiungere la validazione per garantire che le chiamate agli strumenti siano correttamente validate.

### -1- Creare un'architettura

La prima cosa da affrontare è un'architettura che ci aiuti a scalare man mano che aggiungiamo più funzionalità, ecco come appare:

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

Ora abbiamo impostato un'architettura che garantisce che possiamo facilmente aggiungere nuovi strumenti in una cartella tools. Sentiti libero di seguire questa per aggiungere sottodirectory per risorse e prompt.

### -2- Creare uno strumento

Vediamo com'è creare uno strumento. Per prima cosa, deve essere creato nella sua sottodirectory *tool* così:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Valida l'input utilizzando il modello Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: aggiungere Pydantic, così possiamo creare un AddInputModel e validare gli argomenti

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Quello che vediamo qui è come definiamo nome, descrizione, uno schema di input usando Pydantic e un gestore che verrà invocato una volta che questo strumento viene chiamato. Infine, esponiamo `tool_add` che è un dizionario che contiene tutte queste proprietà.

C'è anche *schema.py* usato per definire lo schema di input usato dal nostro strumento:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Dobbiamo anche popolare *__init__.py* per garantire che la directory tools sia trattata come un modulo. Inoltre, dobbiamo esporre i moduli al suo interno così:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Possiamo continuare ad aggiungere a questo file man mano che aggiungiamo più strumenti.

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

Qui creiamo un dizionario costituito da proprietà:

- name, questo è il nome dello strumento.
- rawSchema, questo è lo schema Zod, verrà usato per convalidare le richieste in arrivo per chiamare questo strumento.
- inputSchema, questo schema sarà usato dal gestore.
- callback, questo viene usato per invocare lo strumento.

C'è anche `Tool` che viene usato per convertire questo dizionario in un tipo che il gestore del server mcp può accettare e appare così:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

E c'è *schema.ts* dove memorizziamo gli schemi di input per ogni strumento e che appare così con un solo schema al momento ma man mano che aggiungiamo strumenti possiamo aggiungere altre voci:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Ottimo, passiamo a gestire l'elenco dei nostri strumenti.

### -3- Gestire l'elenco degli strumenti

Per gestire l'elenco dei nostri strumenti, dobbiamo impostare un gestore di richieste per questo. Ecco cosa dobbiamo aggiungere al nostro file server:

**Python**

```python
# codice omesso per brevità
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

Qui aggiungiamo il decoratore `@server.list_tools` e la funzione di implementazione `handle_list_tools`. In quest'ultima, dobbiamo produrre una lista di strumenti. Nota come ogni strumento deve avere un nome, descrizione e inputSchema.

**TypeScript**

Per impostare il gestore di richieste per l'elenco degli strumenti, dobbiamo chiamare `setRequestHandler` sul server con uno schema adatto a quello che vogliamo fare, in questo caso `ListToolsRequestSchema`.

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
// codice omesso per brevità
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Restituisci la lista degli strumenti registrati
  return {
    tools: tools
  };
});
```

Ottimo, ora abbiamo risolto la parte dell'elenco degli strumenti, guardiamo come potremmo chiamare gli strumenti.

### -4- Gestire la chiamata a uno strumento

Per chiamare uno strumento, dobbiamo impostare un altro gestore di richieste, questa volta focalizzato a gestire una richiesta che specifica quale funzionalità chiamare e con quali argomenti.

**Python**

Usiamo il decoratore `@server.call_tool` e lo implementiamo con una funzione come `handle_call_tool`. All'interno di questa funzione, dobbiamo analizzare il nome dello strumento, il suo argomento e assicurarci che gli argomenti siano validi per lo strumento in questione. Possiamo validare gli argomenti in questa funzione o a valle nello strumento effettivo.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools è un dizionario con i nomi degli strumenti come chiavi
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # invoca lo strumento
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Ecco cosa succede:

- Il nostro nome dello strumento è già presente come parametro di input `name` che è vero per i nostri argomenti sotto forma del dizionario `arguments`.

- Lo strumento viene chiamato con `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. La validazione degli argomenti avviene nella proprietà `handler` che punta a una funzione, se fallisce verrà sollevata un'eccezione.

Ecco, ora abbiamo una comprensione completa dell'elenco e della chiamata degli strumenti utilizzando un server a basso livello.

Vedi il [esempio completo](./code/README.md) qui

## Compito

Estendi il codice che ti è stato dato con un numero di strumenti, risorse e prompt e rifletti su come noti che devi solo aggiungere file nella directory tools e da nessun'altra parte.

*Nessuna soluzione fornita*

## Sommario

In questo capitolo, abbiamo visto come funziona l'approccio del server a basso livello e come questo può aiutarci a creare una bella architettura su cui continuare a costruire. Abbiamo anche discusso la validazione e ti è stato mostrato come lavorare con librerie di validazione per creare schemi per la validazione degli input.

## Cosa c'è dopo

- Il prossimo: [Autenticazione semplice](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:  
Questo documento è stato tradotto utilizzando il servizio di traduzione automatica AI [Co-op Translator](https://github.com/Azure/co-op-translator). Pur impegnandoci per garantire accuratezza, si prega di considerare che le traduzioni automatiche possono contenere errori o imprecisioni. Il documento originale nella sua lingua originale deve essere considerato la fonte autorevole. Per informazioni critiche, si consiglia una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali incomprensioni o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
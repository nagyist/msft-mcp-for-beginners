# Erweiterte Servernutzung

Es gibt zwei verschiedene Servertypen, die im MCP SDK bereitgestellt werden: Ihren normalen Server und den Low-Level-Server. Normalerweise würden Sie den regulären Server verwenden, um Funktionen hinzuzufügen. In manchen Fällen möchten Sie jedoch auf den Low-Level-Server zurückgreifen, wie zum Beispiel:

- Bessere Architektur. Es ist möglich, eine saubere Architektur sowohl mit dem regulären Server als auch mit einem Low-Level-Server zu erstellen, aber es kann argumentiert werden, dass es mit einem Low-Level-Server etwas einfacher ist.
- Verfügbarkeit von Funktionen. Einige fortgeschrittene Funktionen können nur mit einem Low-Level-Server verwendet werden. Dies werden Sie in späteren Kapiteln sehen, wenn wir Sampling und Elicitation hinzufügen.

## Regulärer Server vs. Low-Level-Server

So sieht die Erstellung eines MCP-Servers mit dem regulären Server aus

**Python**

```python
mcp = FastMCP("Demo")

# Ein Hinzufügungswerkzeug hinzufügen
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

// Ein Additionstool hinzufügen
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

Dabei geht es darum, dass Sie explizit jedes Werkzeug, jede Ressource oder Eingabeaufforderung hinzufügen, die der Server haben soll. Daran ist nichts falsch.  

### Low-Level-Server-Ansatz

Wenn Sie jedoch den Low-Level-Server-Ansatz verwenden, müssen Sie anders denken, nämlich dass Sie anstatt jedes Werkzeug zu registrieren, stattdessen zwei Handler pro Feature-Typ (Werkzeuge, Ressourcen oder Eingabeaufforderungen) erstellen. Zum Beispiel haben Werkzeuge dann nur zwei Funktionen, wie folgt:

- Auflisten aller Werkzeuge. Eine Funktion wäre für alle Versuche verantwortlich, Werkzeuge aufzulisten.
- Aufrufe aller Werkzeuge bearbeiten. Hier gibt es ebenfalls nur eine Funktion, die Aufrufe eines Werkzeugs bearbeitet.

Das klingt nach möglicherweise weniger Arbeit, oder? Also anstatt ein Werkzeug zu registrieren, muss ich nur sicherstellen, dass das Werkzeug beim Auflisten aller Werkzeuge aufgelistet wird und es aufgerufen wird, wenn eine eingehende Anfrage vorliegt, ein Werkzeug zu verwenden.

Sehen wir uns an, wie der Code jetzt aussieht:

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
  // Gibt die Liste der registrierten Werkzeuge zurück
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

Hier haben wir nun eine Funktion, die eine Liste von Features zurückgibt. Jeder Eintrag in der Werkzeugliste hat nun Felder wie `name`, `description` und `inputSchema`, um dem Rückgabetyp zu entsprechen. Das ermöglicht uns, unsere Werkzeuge und Feature-Definitionen anderswo zu platzieren. Wir können jetzt alle unsere Werkzeuge in einem tools-Ordner erstellen, und das gleiche gilt für alle Ihre Features, sodass Ihr Projekt plötzlich so organisiert sein kann:

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

Das ist großartig, unsere Architektur kann recht sauber gestaltet werden.

Wie sieht es mit dem Aufrufen von Werkzeugen aus, ist es dann das gleiche Prinzip, ein Handler zum Aufrufen eines Werkzeugs, egal welches? Ja, genau, hier ist der Code dafür:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ist ein Wörterbuch mit Toolnamen als Schlüssel
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
    // TODO das Tool aufrufen,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Wie Sie im obigen Code sehen können, müssen wir das zu rufende Werkzeug und die Argumente herausfiltern, und dann müssen wir mit dem Aufruf des Werkzeugs fortfahren.

## Verbesserung des Ansatzes mit Validierung

Bis jetzt haben Sie gesehen, wie alle Ihre Registrierungen zum Hinzufügen von Werkzeugen, Ressourcen und Eingabeaufforderungen durch diese beiden Handler pro Feature-Typ ersetzt werden können. Was müssen wir noch tun? Nun, wir sollten eine Form der Validierung hinzufügen, um sicherzustellen, dass das Werkzeug mit den richtigen Argumenten aufgerufen wird. Jede Laufzeitumgebung hat dafür ihre eigene Lösung, zum Beispiel verwendet Python Pydantic und TypeScript verwendet Zod. Die Idee ist, dass wir Folgendes tun:

- Die Logik zur Erstellung eines Features (Werkzeug, Ressource oder Eingabeaufforderung) in seinen eigenen Ordner verschieben.
- Eine Möglichkeit hinzufügen, eine eingehende Anfrage z. B. zum Aufruf eines Werkzeugs zu validieren.

### Ein Feature erstellen

Um ein Feature zu erstellen, müssen wir eine Datei für dieses Feature erstellen und sicherstellen, dass sie die für dieses Feature erforderlichen obligatorischen Felder enthält. Welche Felder das sind, unterscheidet sich ein wenig zwischen Werkzeugen, Ressourcen und Eingabeaufforderungen.

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
        # Eingabe mit Pydantic-Modell validieren
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic hinzufügen, damit wir ein AddInputModel erstellen und Argumente validieren können

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Hier sehen Sie, wie wir Folgendes tun:

- Ein Schema mit Pydantic `AddInputModel` mit den Feldern `a` und `b` in der Datei *schema.py* erstellen.
- Versuchen, die eingehende Anfrage als Typ `AddInputModel` zu parsen, bei einer Parameterabweichung wird dies abstürzen:

   ```python
   # add.py
    try:
        # Eingaben mit Pydantic-Modell validieren
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Sie können wählen, ob Sie diese Parsing-Logik im Tool-Aufruf selbst oder in der Handler-Funktion platzieren.

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

- Im Handler, der alle Werkzeugaufrufe bearbeitet, versuchen wir nun, die eingehende Anfrage in das definierte Schema des Werkzeugs zu parsen:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    Wenn das funktioniert, rufen wir das eigentliche Werkzeug auf:

    ```typescript
    const result = await tool.callback(input);
    ```

Wie Sie sehen, schafft dieser Ansatz eine großartige Architektur, da alles seinen Platz hat. Die *server.ts* ist eine sehr kleine Datei, die nur die Anforderungs-Handler verbindet, und jedes Feature befindet sich in seinem jeweiligen Ordner, z.B. tools/, resources/ oder /prompts.

Großartig, lassen Sie uns das als Nächstes aufbauen.

## Übung: Einen Low-Level-Server erstellen

In dieser Übung werden wir Folgendes tun:

1. Einen Low-Level-Server erstellen, der das Auflisten von Werkzeugen und das Aufrufen von Werkzeugen behandelt.
1. Eine Architektur implementieren, auf der Sie aufbauen können.
1. Eine Validierung hinzufügen, um sicherzustellen, dass Ihre Werkzeugaufrufe richtig validiert werden.

### -1- Eine Architektur erstellen

Das Erste, womit wir uns befassen müssen, ist eine Architektur, die uns beim Skalieren unterstützt, wenn wir weitere Features hinzufügen. So sieht sie aus:

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

Jetzt haben wir eine Architektur eingerichtet, die sicherstellt, dass wir problemlos neue Werkzeuge in einem tools-Ordner hinzufügen können. Fühlen Sie sich frei, dieser zu folgen und Unterverzeichnisse für resources und prompts hinzuzufügen.

### -2- Ein Werkzeug erstellen

Sehen wir uns als Nächstes an, wie ein Werkzeug erstellt wird. Zuerst muss es in seinem *tool*-Unterverzeichnis wie folgt erstellt werden:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Eingabe mit Pydantic-Modell validieren
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic hinzufügen, damit wir ein AddInputModel erstellen und args validieren können

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Hier sehen wir, wie wir Namen, Beschreibung, ein Eingabeschema mit Pydantic definieren und einen Handler, der aufgerufen wird, wenn dieses Werkzeug ausgeführt wird. Zuletzt exposen wir `tool_add`, das ein Dictionary mit all diesen Eigenschaften hält.

Es gibt auch die Datei *schema.py*, die verwendet wird, um das Eingabeschema für unser Werkzeug zu definieren:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Wir müssen auch *__init__.py* füllen, damit das tools-Verzeichnis als Modul behandelt wird. Zusätzlich müssen wir die Module innerhalb des Verzeichnisses so exposen:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Wir können diese Datei erweitern, wenn wir mehr Werkzeuge hinzufügen.

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

Hier erstellen wir ein Dictionary mit Eigenschaften:

- name, das ist der Name des Werkzeugs.
- rawSchema, das ist das Zod-Schema, das verwendet wird, um eingehende Anfragen zur Nutzung dieses Werkzeugs zu validieren.
- inputSchema, dieses Schema wird vom Handler verwendet.
- callback, das wird verwendet, um das Werkzeug aufzurufen.

Es gibt auch `Tool`, das verwendet wird, um dieses Dictionary in einen Typ umzuwandeln, den der MCP-Server-Handler akzeptieren kann, und sieht so aus:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Und es gibt *schema.ts*, wo wir die Eingabeschemas für jedes Werkzeug speichern, die derzeit nur ein Schema enthalten, aber wenn wir mehr Werkzeuge hinzufügen, können wir weitere Einträge ergänzen:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Großartig, dann machen wir als Nächstes mit der Handhabung des Auflistens unserer Werkzeuge weiter.

### -3- Werkzeugauflistung handhaben

Als Nächstes, um das Auflisten unserer Werkzeuge zu bearbeiten, müssen wir einen Anforderungs-Handler dafür einrichten. Das müssen wir unserer Server-Datei hinzufügen:

**Python**

```python
# Code aus Platzgründen ausgelassen
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

Hier fügen wir den Decorator `@server.list_tools` und die zugehörige Funktion `handle_list_tools` hinzu. In letzterer müssen wir eine Liste von Werkzeugen erzeugen. Beachten Sie, dass jedes Werkzeug einen Namen, eine Beschreibung und ein inputSchema haben muss.   

**TypeScript**

Um den Anforderungs-Handler für das Auflisten von Werkzeugen einzurichten, müssen wir `setRequestHandler` auf dem Server mit einem Schema aufrufen, das zu dem passt, was wir tun wollen, in diesem Fall `ListToolsRequestSchema`. 

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
// Code aus Platzgründen weggelassen
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Gibt die Liste der registrierten Werkzeuge zurück
  return {
    tools: tools
  };
});
```

Großartig, jetzt haben wir den Teil zum Auflisten von Werkzeugen gelöst, sehen wir uns an, wie wir Werkzeuge aufrufen könnten.

### -4- Einen Werkzeugaufruf handhaben

Um ein Werkzeug aufzurufen, müssen wir einen weiteren Anforderungs-Handler einrichten, diesmal fokussiert darauf, mit einer Anfrage umzugehen, die angibt, welches Feature mit welchen Argumenten aufgerufen werden soll.

**Python**

Verwenden wir den Decorator `@server.call_tool` und implementieren ihn mit einer Funktion wie `handle_call_tool`. In dieser Funktion müssen wir den Werkzeugnamen und dessen Argument herausfiltern und sicherstellen, dass die Argumente für das betreffende Werkzeug gültig sind. Wir können die Argumente entweder in dieser Funktion oder im eigentlichen Werkzeug validieren.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools ist ein Wörterbuch mit Werkzeugnamen als Schlüsseln
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # das Werkzeug aufrufen
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Folgendes passiert hier:

- Unser Werkzeugname ist bereits als Eingabeparameter `name` vorhanden, ebenso die Argumente in Form des `arguments`-Dictionarys.

- Das Werkzeug wird aufgerufen mit `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Die Validierung der Argumente erfolgt in der `handler`-Eigenschaft, die auf eine Funktion zeigt; wenn die Validierung fehlschlägt, wird eine Ausnahme ausgelöst.

Damit haben wir nun ein vollständiges Verständnis davon, wie man Werkzeuge mit einem Low-Level-Server auflistet und aufruft.

Sehen Sie sich das [vollständige Beispiel](./code/README.md) hier an

## Aufgabe

Erweitern Sie den Ihnen gegebenen Code mit mehreren Werkzeugen, Ressourcen und Eingabeaufforderungen und reflektieren Sie, wie Sie feststellen, dass Sie nur Dateien im tools-Verzeichnis hinzufügen müssen und nirgendwo sonst. 

*Keine Lösung vorgegeben*

## Zusammenfassung

In diesem Kapitel haben wir gesehen, wie der Low-Level-Server-Ansatz funktioniert und wie er uns hilft, eine schöne Architektur zu schaffen, auf der wir weiter aufbauen können. Wir haben außerdem über Validierung gesprochen und Ihnen gezeigt, wie man mit Validierungsbibliotheken arbeitet, um Schemata zur Eingabevalidierung zu erstellen.

## Was kommt als Nächstes

- Nächstes: [Einfache Authentifizierung](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:  
Dieses Dokument wurde mithilfe des KI-Übersetzungsdienstes [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, weisen wir darauf hin, dass automatische Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Für wichtige Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Nutzung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
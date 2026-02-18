# Zaawansowane użycie serwera

W MCP SDK dostępne są dwa różne typy serwerów: zwykły serwer oraz serwer niskiego poziomu. Zazwyczaj używa się zwykłego serwera do dodawania funkcji. W niektórych jednak przypadkach warto polegać na serwerze niskiego poziomu, na przykład:

- Lepsza architektura. Możliwe jest stworzenie czystej architektury zarówno z zwykłym serwerem, jak i serwerem niskiego poziomu, ale można argumentować, że jest to nieco łatwiejsze z serwerem niskiego poziomu.
- Dostępność funkcji. Niektóre zaawansowane funkcje można wykorzystać tylko z serwerem niskiego poziomu. Zobaczysz to w późniejszych rozdziałach, gdy dodamy próbkowanie i elicytację.

## Zwykły serwer a serwer niskiego poziomu

Tak wygląda tworzenie serwera MCP za pomocą zwykłego serwera

**Python**

```python
mcp = FastMCP("Demo")

# Dodaj narzędzie do dodawania
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

// Dodaj narzędzie do dodawania
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

Chodzi o to, że jawnie dodajesz każde narzędzie, zasób lub prompt, które chcesz, aby serwer posiadał. W tym nie ma nic złego.  

### Podejście serwera niskiego poziomu

Natomiast w podejściu z serwerem niskiego poziomu musisz podejść do tego inaczej, mianowicie zamiast rejestrować każde narzędzie, tworzysz dwa handlery na typ funkcji (narzędzia, zasoby lub prompt). Na przykład dla narzędzi są tylko dwie funkcje:

- Lista wszystkich narzędzi. Jedna funkcja odpowiada za wszystkie próby listowania narzędzi.
- obsługa wywoływania wszystkich narzędzi. Tutaj również jest tylko jedna funkcja obsługująca wywołania narzędzi.

Brzmi jak potencjalnie mniej pracy, prawda? Więc zamiast rejestrować narzędzie, wystarczy upewnić się, że narzędzie jest na liście podczas listowania wszystkich narzędzi oraz że jest wywoływane, gdy pojawi się żądanie jego wywołania. 

Zobaczmy jak teraz wygląda kod:

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
  // Zwróć listę zarejestrowanych narzędzi
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

Teraz mamy funkcję, która zwraca listę funkcji. Każdy wpis na liście narzędzi ma pola takie jak `name`, `description` i `inputSchema`, aby odpowiadać typowi zwracanemu. Pozwala to na umieszczenie definicji narzędzi i funkcji gdzie indziej. Możemy teraz tworzyć wszystkie nasze narzędzia w folderze tools, to samo tyczy się wszystkich funkcji, dzięki czemu projekt może być zorganizowany tak:

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

Świetnie, nasza architektura może wyglądać całkiem czyście.

A co z wywoływaniem narzędzi, czy to ta sama idea, jeden handler do wywołania dowolnego narzędzia? Tak, dokładnie, oto kod do tego:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools to słownik z nazwami narzędzi jako kluczami
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
    // TODO wywołaj narzędzie,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Jak widać w powyższym kodzie, musimy wyodrębnić narzędzie do wywołania oraz argumenty, a następnie kontynuować wywołanie narzędzia.

## Ulepszanie podejścia za pomocą walidacji

Jak dotąd widziałeś, że wszystkie rejestracje dodające narzędzia, zasoby i prompt można zastąpić tymi dwoma handlerami na typ funkcji. Co jeszcze powinniśmy zrobić? Powinniśmy dodać jakąś formę walidacji, aby upewnić się, że narzędzie jest wywoływane z właściwymi argumentami. Każde środowisko wykonawcze ma swoje rozwiązanie, na przykład Python używa Pydantic, a TypeScript używa Zod. Idea jest następująca:

- Przenieść logikę tworzenia funkcji (narzędzia, zasobu lub prompta) do dedykowanego folderu.
- Dodać sposób walidacji przychodzącego żądania, np. wywołania narzędzia.

### Tworzenie funkcji

Aby stworzyć funkcję, musimy utworzyć dla niej plik i upewnić się, że zawiera obowiązkowe pola wymagane przez daną funkcję. Które pola się różnią nieco między narzędziami, zasobami i promptami.

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
        # Waliduj dane wejściowe za pomocą modelu Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: dodaj Pydantic, abyśmy mogli stworzyć AddInputModel i zwalidować argumenty

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

tutaj widać, jak robimy:

- Tworzymy schemat za pomocą Pydantic `AddInputModel` z polami `a` i `b` w pliku *schema.py*.
- Próbujemy sparsować przychodzące żądanie na typ `AddInputModel`, jeśli parametry nie pasują, nastąpi awaria:

   ```python
   # add.py
    try:
        # Waliduj dane wejściowe za pomocą modelu Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Możesz wybrać, czy tę logikę parsowania umieścić w samym wywołaniu narzędzia, czy w funkcji handlera.

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

- W handlerze obsługującym wszystkie wywołania narzędzi próbujemy teraz sparsować przychodzące żądanie do zdefiniowanego schematu narzędzia:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    jeśli to się uda, przechodzimy do wywołania właściwego narzędzia:

    ```typescript
    const result = await tool.callback(input);
    ```

Jak widzisz, to podejście tworzy świetną architekturę, bo wszystko ma swoje miejsce, *server.ts* jest bardzo małym plikiem, który tylko łączy request handlery, a każda funkcja znajduje się w swoim folderze, tj. tools/, resources/ lub prompts/.

Świetnie, spróbujmy teraz to zbudować.

## Ćwiczenie: Tworzenie serwera niskiego poziomu

W tym ćwiczeniu zrobimy następujące rzeczy:

1. Stworzymy serwer niskiego poziomu, obsługujący listowanie narzędzi i wywoływanie narzędzi.
1. Zaimportujemy architekturę, na której można dalej budować.
1. Dodamy walidację, aby zapewnić poprawną walidację wywołań narzędzi.

### -1- Stworzenie architektury

Pierwsza rzecz, którą musimy rozwiązać, to architektura pomagająca nam skalować się wraz z dodawaniem kolejnych funkcji, wygląda to tak:

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

Teraz mamy ustawioną architekturę, która pozwala łatwo dodawać narzędzia w folderze tools. Możesz też dodać podkatalogi dla zasobów i promptów wg uznania.

### -2- Tworzenie narzędzia

Zobaczmy, jak wygląda tworzenie narzędzia. Najpierw musi ono powstać w podkatalogu *tool* tak:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Waliduj dane wejściowe za pomocą modelu Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: dodaj Pydantic, abyśmy mogli stworzyć AddInputModel i zwalidować argumenty

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Widzimy tu, jak definiujemy nazwę, opis, schemat wejściowy za pomocą Pydantic oraz handler, który będzie wywołany po wywołaniu narzędzia. Na końcu udostępniamy `tool_add`, który jest słownikiem zawierającym wszystkie te właściwości.

Jest też *schema.py* używany do definiowania schematu wejściowego używanego przez nasze narzędzie:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Musimy także wypełnić *__init__.py*, aby folder tools był traktowany jako moduł. Dodatkowo musimy takie moduły w nim udostępnić tak:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Możemy dalej dodawać wpisy w tym pliku, gdy dodajemy kolejne narzędzia.

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

Tworzymy tu słownik składający się z właściwości:

- name, to nazwa narzędzia,
- rawSchema, to schemat Zod używany do walidacji przychodzących żądań wywołania tego narzędzia,
- inputSchema, ten schemat będzie używany przez handler,
- callback, służy do wywołania narzędzia.

Jest też `Tool`, który konwertuje ten słownik na typ akceptowany przez handler serwera MCP, wygląda tak:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Jest też *schema.ts*, gdzie trzymamy schematy wejściowe dla poszczególnych narzędzi, obecnie z jednym schematem, ale możemy dodawać kolejne wpisy:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Świetnie, przejdźmy teraz do obsługi listowania naszych narzędzi.

### -3- Obsługa listowania narzędzi

Aby obsłużyć listowanie narzędzi, musimy dodać request handler obsługujący to żądanie. Dodajemy to do naszego pliku serwera:

**Python**

```python
# kod pominięty dla zwięzłości
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

Dodajemy dekorator `@server.list_tools` oraz implementującą funkcję `handle_list_tools`. W tej funkcji musimy zwrócić listę narzędzi. Zwróć uwagę, że każde narzędzie musi mieć nazwę, opis i inputSchema.   

**TypeScript**

Aby dodać handler do żądania listowania narzędzi, wywołujemy `setRequestHandler` na serwerze z odpowiednim schematem, czyli `ListToolsRequestSchema`.

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
// kod pominięty dla zwięzłości
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Zwróć listę zarejestrowanych narzędzi
  return {
    tools: tools
  };
});
```

Dobrze, mamy już rozwiązany fragment listowania narzędzi, zobaczmy teraz, jak wywołujemy narzędzia.

### -4- Obsługa wywołania narzędzia

Aby wywołać narzędzie, musimy dodać kolejny request handler, tym razem obsługujący żądanie określające, którą funkcję wywołać oraz z jakimi argumentami.

**Python**

Użyjmy dekoratora `@server.call_tool` i zaimplementujmy go funkcją `handle_call_tool`. W tej funkcji musimy wydobyć nazwę narzędzia, jego argumenty oraz upewnić się, że argumenty są poprawne dla danego narzędzia. Walidację możemy zrobić tutaj albo dalej, w samym narzędziu.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools to słownik z nazwami narzędzi jako kluczami
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # wywołaj narzędzie
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Co się tutaj dzieje:

- Nazwa narzędzia jest dostępna jako parametr wejściowy `name`, a argumenty jako słownik `arguments`.

- Narzędzie jest wywoływane przez `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Walidacja argumentów odbywa się w polu `handler`, które wskazuje na funkcję, jeśli walidacja się nie powiedzie, zostanie rzucony wyjątek.

Otóż to, teraz mamy pełne zrozumienie listowania i wywoływania narzędzi za pomocą serwera niskiego poziomu.

Zobacz [pełny przykład](./code/README.md)

## Zadanie

Rozszerz podany kod o kilka narzędzi, zasobów i promptów i zaobserwuj, że musisz dodawać jedynie pliki w katalogu tools i nigdzie indziej.

*Brak podanego rozwiązania*

## Podsumowanie

W tym rozdziale zobaczyliśmy, jak działa podejście serwera niskiego poziomu i jak może pomóc stworzyć ładną architekturę, na której można budować kolejne funkcje. Omówiliśmy też walidację oraz pokazano, jak pracować z bibliotekami walidacyjnymi do tworzenia schematów walidacji wejścia.

## Co dalej

- Następny: [Prosta autoryzacja](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:  
Niniejszy dokument został przetłumaczony przy użyciu usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dokładamy starań, aby tłumaczenie było jak najbardziej poprawne, prosimy mieć na uwadze, że automatyczne tłumaczenia mogą zawierać błędy lub nieścisłości. Oryginalny dokument w języku źródłowym powinien być traktowany jako ostateczne i autorytatywne źródło. W przypadku istotnych informacji zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z korzystania z tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
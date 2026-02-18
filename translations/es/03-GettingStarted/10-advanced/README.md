# Uso avanzado del servidor

Hay dos tipos diferentes de servidores expuestos en el SDK de MCP, tu servidor normal y el servidor de bajo nivel. Normalmente, usarías el servidor regular para agregarle funciones. Sin embargo, en algunos casos, querrás depender del servidor de bajo nivel, como por ejemplo:

- Mejor arquitectura. Es posible crear una arquitectura limpia con ambos, el servidor regular y el servidor de bajo nivel, pero se podría argumentar que es un poco más fácil con un servidor de bajo nivel.
- Disponibilidad de funciones. Algunas funciones avanzadas sólo pueden usarse con un servidor de bajo nivel. Verás esto en capítulos posteriores cuando añadamos muestreo y elicitación.

## Servidor regular vs servidor de bajo nivel

Así es como se ve la creación de un servidor MCP con el servidor regular:

**Python**

```python
mcp = FastMCP("Demo")

# Añadir una herramienta de suma
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

// Añadir una herramienta de suma
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

Lo importante es que tú agregas explícitamente cada herramienta, recurso o indicación que quieres que tenga el servidor. No hay nada de malo en eso.

### Enfoque del servidor de bajo nivel

Sin embargo, cuando usas el enfoque del servidor de bajo nivel, debes pensar diferente, a saber, que en lugar de registrar cada herramienta, creas dos manejadores por tipo de función (herramientas, recursos o indicaciones). Por ejemplo, las herramientas solo tienen dos funciones así:

- Listar todas las herramientas. Una función sería responsable de todos los intentos de listar herramientas.
- manejar llamadas a todas las herramientas. Aquí también, hay una sola función manejando las llamadas a una herramienta.

Eso suena a potencialmente menos trabajo ¿verdad? Entonces, en lugar de registrar una herramienta, solo necesito asegurarme de que la herramienta esté listada cuando liste todas las herramientas y que se llame cuando haya una solicitud entrante para llamar a una herramienta.

Veamos cómo se ve el código ahora:

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
  // Devuelve la lista de herramientas registradas
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

Aquí ahora tenemos una función que devuelve una lista de funciones. Cada entrada en la lista de herramientas ahora tiene campos como `name`, `description` y `inputSchema` para ajustarse al tipo de retorno. Esto nos permite poner nuestras herramientas y definición de funciones en otro lugar. Ahora podemos crear todas nuestras herramientas en una carpeta tools y lo mismo para todas tus funciones, así tu proyecto puede organizarse repentinamente así:

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

Eso es genial, nuestra arquitectura puede ser hecha para lucir bastante limpia.

¿Qué tal llamar a herramientas, es la misma idea entonces, un manejador para llamar a una herramienta, cualquiera que sea? Sí, exactamente, aquí está el código para eso:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools es un diccionario con nombres de herramientas como claves
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
    // TODO llamar a la herramienta,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Como puedes ver en el código de arriba, necesitamos analizar qué herramienta llamar, y con qué argumentos, y luego proceder a llamar a la herramienta.

## Mejorando el enfoque con validación

Hasta ahora, has visto cómo todos tus registros para añadir herramientas, recursos e indicaciones pueden ser reemplazados con estos dos manejadores por tipo de función. ¿Qué más necesitamos hacer? Bueno, deberíamos agregar alguna forma de validación para asegurar que la herramienta se llame con los argumentos correctos. Cada entorno de ejecución tiene su propia solución para esto, por ejemplo Python usa Pydantic y TypeScript usa Zod. La idea es que hagamos lo siguiente:

- Mover la lógica para crear una función (herramienta, recurso o indicación) a su carpeta dedicada.
- Agregar una forma de validar una solicitud entrante que pida por ejemplo llamar a una herramienta.

### Crear una función

Para crear una función, necesitaremos crear un archivo para esa función y asegurarnos de que tenga los campos obligatorios requeridos por esa función. Qué campos difieren un poco entre herramientas, recursos e indicaciones.

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
        # Validar la entrada usando un modelo Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: agregar Pydantic, para que podamos crear un AddInputModel y validar los argumentos

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Aquí puedes ver cómo hacemos lo siguiente:

- Crear un esquema usando Pydantic `AddInputModel` con los campos `a` y `b` en el archivo *schema.py*.
- Intentar analizar la solicitud entrante para que sea del tipo `AddInputModel`, si hay una discordancia en los parámetros, esto fallará:

   ```python
   # add.py
    try:
        # Validar la entrada usando el modelo Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Puedes elegir si poner esta lógica de análisis en la llamada a la herramienta misma o en la función manejadora.

**TypeScript**

```typescript
// servidor.ts
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

// esquema.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// agregar.ts
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

- En el manejador que se ocupa de todas las llamadas a herramientas, ahora intentamos analizar la solicitud entrante dentro del esquema definido por la herramienta:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    si eso funciona, entonces procedemos a llamar la herramienta real:

    ```typescript
    const result = await tool.callback(input);
    ```

Como puedes ver, este enfoque crea una gran arquitectura ya que todo tiene su lugar, el *server.ts* es un archivo muy pequeño que sólo conecta los manejadores de solicitud y cada función está en su respectiva carpeta i.e tools/, resources/ o /prompts.

Genial, intentemos construir esto a continuación.

## Ejercicio: Creando un servidor de bajo nivel

En este ejercicio, haremos lo siguiente:

1. Crear un servidor de bajo nivel que maneje la lista de herramientas y la llamada a herramientas.
1. Implementar una arquitectura sobre la que puedas construir.
1. Añadir validación para asegurar que las llamadas a tus herramientas estén correctamente validadas.

### -1- Crear una arquitectura

Lo primero que necesitamos abordar es una arquitectura que nos ayude a escalar mientras añadimos más funciones, así es como se ve:

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

Ahora hemos montado una arquitectura que asegura que podemos añadir fácilmente nuevas herramientas en una carpeta tools. Siéntete libre de seguir esto para añadir subdirectorios para recursos e indicaciones.

### -2- Crear una herramienta

Veamos cómo se ve crear una herramienta. Primero, necesita ser creada en su subdirectorio *tool* así:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validar entrada usando el modelo Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: agregar Pydantic, para que podamos crear un AddInputModel y validar los argumentos

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Lo que vemos aquí es cómo definimos name, description, un esquema de entrada usando Pydantic y un manejador que se invocará una vez que esta herramienta sea llamada. Por último, exponemos `tool_add` que es un diccionario que contiene todas estas propiedades.

También está *schema.py* que se usa para definir el esquema de entrada usado por nuestra herramienta:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

También necesitamos poblar *__init__.py* para asegurar que el directorio tools sea tratado como un módulo. Además, necesitamos exponer los módulos dentro de él así:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Podemos seguir añadiendo a este archivo conforme agregamos más herramientas.

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

Aquí creamos un diccionario que consiste en propiedades:

- name, este es el nombre de la herramienta.
- rawSchema, este es el esquema Zod, se usará para validar las solicitudes entrantes para llamar a esta herramienta.
- inputSchema, este esquema será usado por el manejador.
- callback, esto se usa para invocar la herramienta.

También está `Tool` que se usa para convertir este diccionario en un tipo que el manejador del servidor mcp puede aceptar y se ve así:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Y hay un *schema.ts* donde almacenamos los esquemas de entrada para cada herramienta que se ve así, con sólo un esquema por ahora, pero conforme añadamos herramientas podemos añadir más entradas:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Genial, procedamos a manejar la lista de nuestras herramientas a continuación.

### -3- Manejar la lista de herramientas

A continuación, para manejar la lista de nuestras herramientas, necesitamos configurar un manejador de solicitud para eso. Esto es lo que necesitamos añadir a nuestro archivo del servidor:

**Python**

```python
# código omitido por brevedad
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

Aquí agregamos el decorador `@server.list_tools` y la función que lo implementa `handle_list_tools`. En esta última, necesitamos producir una lista de herramientas. Observa cómo cada herramienta necesita tener un name, description e inputSchema.

**TypeScript**

Para configurar el manejador de solicitud para listar herramientas, necesitamos llamar a `setRequestHandler` en el servidor con un esquema que encaje con lo que estamos tratando de hacer, en este caso `ListToolsRequestSchema`.

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
// código omitido por brevedad
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Devuelve la lista de herramientas registradas
  return {
    tools: tools
  };
});
```

Genial, ahora hemos resuelto la parte de listar herramientas, veamos cómo podríamos llamar a las herramientas.

### -4- Manejar la llamada a una herramienta

Para llamar a una herramienta, necesitamos configurar otro manejador de solicitud, esta vez enfocado en tratar una solicitud que especifique qué función llamar y con qué argumentos.

**Python**

Usemos el decorador `@server.call_tool` e implementémoslo con una función como `handle_call_tool`. Dentro de esa función, necesitamos analizar el nombre de la herramienta, su argumento y asegurar que los argumentos sean válidos para la herramienta en cuestión. Podemos validar los argumentos en esta función o más abajo en la herramienta real.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools es un diccionario con nombres de herramientas como claves
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # invocar la herramienta
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Esto es lo que sucede:

- El nombre de nuestra herramienta ya está presente como el parámetro de entrada `name`, lo cual es cierto para nuestros argumentos en forma del diccionario `arguments`.

- La herramienta se llama con `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. La validación de los argumentos sucede en la propiedad `handler` que apunta a una función, si eso falla lanzará una excepción.

Ahí tienes, ahora tenemos una comprensión completa de listar y llamar herramientas usando un servidor de bajo nivel.

Consulta el [ejemplo completo](./code/README.md) aquí

## Tarea

Extiende el código que te han dado con varias herramientas, recursos e indicaciones y reflexiona sobre cómo notas que sólo necesitas añadir archivos en el directorio tools y en ningún otro lugar.

*No se da solución*

## Resumen

En este capítulo, vimos cómo funciona el enfoque del servidor de bajo nivel y cómo eso puede ayudarnos a crear una arquitectura agradable en la que podemos seguir construyendo. También discutimos la validación y viste cómo trabajar con librerías de validación para crear esquemas para validar entradas.

## Qué sigue

- Siguiente: [Autenticación simple](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:  
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automáticas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional realizada por humanos. No somos responsables de malentendidos o interpretaciones erróneas que puedan surgir del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
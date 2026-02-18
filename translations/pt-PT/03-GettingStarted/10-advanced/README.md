# Utilização avançada do servidor

Existem dois tipos diferentes de servidores expostos no SDK MCP, o seu servidor normal e o servidor de baixo nível. Normalmente, usaria o servidor regular para adicionar funcionalidades. No entanto, em alguns casos, quer confiar no servidor de baixo nível, como por exemplo:

- Melhor arquitetura. É possível criar uma arquitetura limpa tanto com o servidor regular como com o servidor de baixo nível, mas pode-se argumentar que é ligeiramente mais fácil com um servidor de baixo nível.
- Disponibilidade de funcionalidades. Algumas funcionalidades avançadas só podem ser usadas com um servidor de baixo nível. Irá ver isto em capítulos posteriores, quando adicionarmos amostragem e elicitação.

## Servidor regular vs servidor de baixo nível

Aqui está como a criação de um Servidor MCP parece com o servidor regular

**Python**

```python
mcp = FastMCP("Demo")

# Adicionar uma ferramenta de adição
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

// Adicionar uma ferramenta de adição
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

A ideia é que adiciona explicitamente cada ferramenta, recurso ou prompt que quer que o servidor tenha. Não há nada de errado nisso.

### Abordagem do servidor de baixo nível

No entanto, quando usa a abordagem do servidor de baixo nível, precisa pensar de forma diferente, nomeadamente que em vez de registar cada ferramenta, cria dois handlers por tipo de funcionalidade (ferramentas, recursos ou prompts). Por exemplo, as ferramentas têm apenas duas funções deste género:

- Listar todas as ferramentas. Uma função seria responsável por todas as tentativas de listar ferramentas.
- Tratar chamadas a todas as ferramentas. Aqui também, só existe uma função que trata chamadas a uma ferramenta.

Parece potencialmente menos trabalho, certo? Então em vez de registar uma ferramenta, só preciso de garantir que a ferramenta está listada quando listamos todas as ferramentas e que é chamada quando há um pedido para chamar uma ferramenta.

Vamos ver como o código fica agora:

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
  // Retorna a lista de ferramentas registadas
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

Aqui temos uma função que retorna uma lista de funcionalidades. Cada entrada na lista de ferramentas tem agora campos como `name`, `description` e `inputSchema` para cumprir o tipo de retorno. Isto permite-nos colocar as nossas ferramentas e definição de funcionalidades noutro local. Podemos agora criar todas as nossas ferramentas numa pasta tools e o mesmo se aplica a todas as suas funcionalidades, pelo que o seu projeto pode subitamente estar organizado assim:

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

Isso é ótimo, a nossa arquitetura pode ficar bastante limpa.

E quanto a chamar ferramentas, é a mesma ideia, um handler para chamar uma ferramenta, qualquer que seja? Sim, exatamente, aqui está o código para isso:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools é um dicionário com nomes de ferramentas como chaves
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
    // TODO chamar a ferramenta,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Como pode ver no código acima, precisamos analisar qual a ferramenta a chamar, e com que argumentos, e depois avançar para chamar a ferramenta.

## Melhorar a abordagem com validação

Até agora, viu como todos os registos para adicionar ferramentas, recursos e prompts podem ser substituídos por estes dois handlers por tipo de funcionalidade. O que mais precisamos fazer? Bem, devemos adicionar alguma forma de validação para garantir que a ferramenta é chamada com os argumentos certos. Cada runtime tem a sua própria solução para isto, por exemplo Python usa Pydantic e TypeScript usa Zod. A ideia é que façamos o seguinte:

- Mover a lógica para criar uma funcionalidade (ferramenta, recurso ou prompt) para a sua pasta dedicada.
- Adicionar uma forma de validar um pedido de entrada pedindo, por exemplo, para chamar uma ferramenta.

### Criar uma funcionalidade

Para criar uma funcionalidade, precisamos criar um ficheiro para essa funcionalidade e garantir que contém os campos obrigatórios requeridos para essa funcionalidade. Os campos diferem um pouco entre ferramentas, recursos e prompts.

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
        # Validar entrada usando modelo Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: adicionar Pydantic, para podermos criar um AddInputModel e validar argumentos

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

aqui pode ver como fazemos o seguinte:

- Criar um esquema usando Pydantic `AddInputModel` com os campos `a` e `b` no ficheiro *schema.py*.
- Tentar analisar o pedido de entrada para o tipo `AddInputModel`, se houver uma incompatibilidade nos parâmetros, isto irá falhar:

   ```python
   # add.py
    try:
        # Validar a entrada usando modelo Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Pode escolher se coloca esta lógica de parsing na chamada da ferramenta em si ou na função handler.

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

- No handler que lida com todas as chamadas de ferramentas, agora tentamos converter o pedido de entrada para o esquema definido pela ferramenta:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    se isso funcionar, prosseguimos para chamar a ferramenta propriamente dita:

    ```typescript
    const result = await tool.callback(input);
    ```

Como pode ver, esta abordagem cria uma ótima arquitetura porque tudo tem o seu lugar, o *server.ts* é um ficheiro muito pequeno que só liga os handlers de pedidos e cada funcionalidade está na sua respetiva pasta, ou seja, tools/, resources/ ou /prompts.

Ótimo, vamos tentar construir isto a seguir.

## Exercício: Criar um servidor de baixo nível

Neste exercício, faremos o seguinte:

1. Criar um servidor de baixo nível que trate a listagem de ferramentas e chamadas de ferramentas.
1. Implementar uma arquitetura sobre a qual possa construir.
1. Adicionar validação para garantir que as chamadas às suas ferramentas são devidamente validadas.

### -1- Criar uma arquitetura

A primeira coisa que precisamos tratar é uma arquitetura que nos ajuda a escalar à medida que adicionamos mais funcionalidades, veja como é:

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

Agora temos uma arquitetura que assegura que podemos adicionar facilmente novas ferramentas numa pasta tools. Sinta-se livre para seguir este exemplo para adicionar subdiretórios para recursos e prompts.

### -2- Criar uma ferramenta

Vamos ver como criar uma ferramenta. Primeiro, precisa ser criada no seu subdiretório *tool* assim:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validar a entrada usando o modelo Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: adicionar Pydantic, para podermos criar um AddInputModel e validar os args

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

O que vemos aqui é como definimos o nome, descrição, um esquema de entrada usando Pydantic e um handler que será invocado quando esta ferramenta for chamada. Por último, expomos `tool_add` que é um dicionário que contém todas essas propriedades.

Também existe o *schema.py* usado para definir o esquema de entrada usado pela nossa ferramenta:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Também precisamos preencher o *__init__.py* para garantir que o diretório tools é tratado como um módulo. Além disso, temos de expor os módulos dentro dele assim:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Podemos continuar a adicionar a este ficheiro à medida que adicionamos mais ferramentas.

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

Aqui criamos um dicionário composto pelas propriedades:

- name, que é o nome da ferramenta.
- rawSchema, que é o esquema Zod, usado para validar pedidos de entrada para chamar esta ferramenta.
- inputSchema, este esquema será usado pelo handler.
- callback, usado para invocar a ferramenta.

Existe também `Tool` que é usado para converter este dicionário num tipo que o handler do servidor mcp pode aceitar e que é assim:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

E há *schema.ts* onde guardamos os esquemas de entrada para cada ferramenta que é assim, com apenas um esquema presente, mas que podemos adicionar mais à medida que criamos mais ferramentas:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Ótimo, vamos prosseguir para tratar a listagem das nossas ferramentas a seguir.

### -3- Tratar a listagem de ferramentas

A seguir, para tratar a listagem das nossas ferramentas, precisamos configurar um handler de pedido para isso. Aqui está o que precisamos adicionar ao nosso ficheiro do servidor:

**Python**

```python
# código omitido por brevidade
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

Aqui adicionamos o décorator `@server.list_tools` e a função de implementação `handle_list_tools`. Nesta última, precisamos produzir uma lista de ferramentas. Note como cada ferramenta precisa ter um nome, descrição e inputSchema.

**TypeScript**

Para configurar o handler de pedido para listar ferramentas, precisamos chamar `setRequestHandler` no servidor com um esquema adequado ao que estamos a tentar fazer, neste caso `ListToolsRequestSchema`.

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
// código omitido para brevidade
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Retorna a lista de ferramentas registadas
  return {
    tools: tools
  };
});
```

Ótimo, agora resolvemos a parte da listagem das ferramentas, vamos ver como podemos chamar ferramentas a seguir.

### -4- Tratar chamada a ferramenta

Para chamar uma ferramenta, precisamos configurar outro handler de pedido, desta vez focado em lidar com um pedido que especifica qual funcionalidade chamar e com que argumentos.

**Python**

Vamos usar o décorator `@server.call_tool` e implementar com uma função como `handle_call_tool`. Dentro dessa função, precisamos desestruturar o nome da ferramenta, seu argumento e garantir que os argumentos são válidos para a ferramenta específica. Podemos validar os argumentos nesta função ou mais abaixo na ferramenta real.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools é um dicionário com nomes de ferramentas como chaves
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # invocar a ferramenta
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Aqui está o que acontece:

- O nome da nossa ferramenta já está presente como parâmetro de entrada `name` e os nossos argumentos estão na forma do dicionário `arguments`.

- A ferramenta é chamada com `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. A validação dos argumentos acontece na propriedade `handler` que aponta para uma função e se falhar vai lançar uma exceção.

Pronto, agora temos uma compreensão completa de como listar e chamar ferramentas usando um servidor de baixo nível.

Veja o [exemplo completo](./code/README.md) aqui

## Tarefa

Estenda o código que recebeu com várias ferramentas, recursos e prompts e reflita sobre como vai notar que só precisa de adicionar ficheiros na pasta tools e em mais lado nenhum.

*Sem solução fornecida*

## Resumo

Neste capítulo, vimos como funciona a abordagem do servidor de baixo nível e como isso pode ajudar-nos a criar uma boa arquitetura em que podemos continuar a construir. Também discutimos validação e foi-lhe mostrado como trabalhar com bibliotecas de validação para criar esquemas para validação de entrada.

## O que vem a seguir

- A seguir: [Autenticação simples](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, deve ter em atenção que as traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte oficial. Para informação crítica, recomenda-se a tradução profissional por um humano. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
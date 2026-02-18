# Advanced server usage

Mayroong dalawang magkaibang uri ng mga server na ipinapakita sa MCP SDK, ang iyong normal na server at ang low-level server. Karaniwan, gagamitin mo ang regular na server upang magdagdag ng mga tampok dito. Sa ilang mga kaso naman, nais mong umasa sa low-level server tulad ng:

- Mas magandang arkitektura. Posible na gumawa ng malinis na arkitektura gamit ang parehong regular na server at isang low-level server ngunit maaaring sabihin na mas madali ito gamit ang low-level server.
- Pagkakaroon ng mga tampok. Ang ilang mga advanced na tampok ay maaari lamang gamitin sa low-level server. Makikita mo ito sa mga susunod na kabanata habang nagdadagdag tayo ng sampling at elicitation.

## Regular server vs low-level server

Ganito ang hitsura ng paggawa ng isang MCP Server gamit ang regular na server

**Python**

```python
mcp = FastMCP("Demo")

# Magdagdag ng isang karagdagang kasangkapan
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

// Magdagdag ng kasangkapan para sa pagdaragdag
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

Ang punto ay malinaw na idinaragdag mo ang bawat tool, resource o prompt na nais mo na magkaroon ang server. Walang masama dito.  

### Low-level server approach

Gayunpaman, kapag ginamit mo ang low-level server approach kailangan mong pag-isipan ito nang iba, ang ibig sabihin ay sa halip na irehistro ang bawat tool ay gumagawa ka ng dalawang handler para sa bawat uri ng tampok (tools, resources o prompts). Halimbawa para sa mga tools ay merong dalawang functions lang tulad ng:

- Paglista ng lahat ng tools. Isang function ang responsable sa lahat ng pagtatangkang maglista ng tools.
- Pamahalaan ang pagtawag sa lahat ng tools. Dito rin, isang function lang ang humahawak ng mga tawag sa isang tool.

Mukhang mas konti ang trabaho, 'di ba? Kaya imbes na irehistro ang tool, kailangan ko lang siguraduhin na nakalista ang tool kapag nilista ko lahat ng tools at tatawagin ito kapag may papasok na request na tumawag sa tool. 

Tingnan natin ang hitsura ng code ngayon:

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
  // Ibalik ang listahan ng mga nakarehistrong kagamitan
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

Dito mayroon tayong function na nagbabalik ng isang listahan ng mga tampok. Ang bawat entry sa listahan ng tools ay may mga field tulad ng `name`, `description` at `inputSchema` upang sumunod sa return type. Pinapayagan tayo nitong ilagay ang ating mga tools at mga definition ng tampok kahit saan pa. Maaari na nating likhain lahat ng tools sa isang folder na tools at ganoon din sa lahat ng iyong mga tampok kaya bigla na lang naayos ang iyong proyekto tulad nito:

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

Ang ganda, maaaring gawing malinis ang ating arkitektura.

Paano naman ang pagtawag sa mga tools, pareho ba ang ideya, isang handler lang na tumatawag sa isang tool, alin mang tool? Oo, eksakto, ito ang code para doon:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # ang tools ay isang diksyunaryo na may mga pangalan ng tool bilang mga susi
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
    // TODO tawagin ang kasangkapan,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Makikita mo mula sa code sa itaas, kailangan nating kunin ang tool na tatawagin, at kung ano ang mga argument, pagkatapos ay kailangan nating isagawa ang pagtawag sa tool.

## Pagpapabuti ng paraan gamit ang validation

Sa ngayon, nakita mo na kung paano ang lahat ng iyong mga rehistrasyon para magdagdag ng tools, resources, at prompts ay maaaring palitan ng dalawang handlers na ito para sa bawat uri ng tampok. Ano pa ang kailangan nating gawin? Dapat tayong magdagdag ng ilang uri ng validation upang matiyak na ang tool ay tatawagin gamit ang tamang mga argumento. Ang bawat runtime ay may sarili nitong solusyon para dito, halimbawang gumagamit ang Python ng Pydantic at ang TypeScript ay gumagamit ng Zod. Ang ideya ay gawin natin ang sumusunod:

- Ilipat ang lohika para sa paggawa ng isang tampok (tool, resource o prompt) sa dedikadong folder nito.
- Magdagdag ng paraan upang i-validate ang papasok na request na nagsasabing halimbawang tumawag ng isang tool.

### Gumawa ng isang tampok

Upang gumawa ng tampok, kailangan nating gumawa ng isang file para sa tampok na iyon at siguraduhing mayroon itong mga kinakailangang field na kailangan ng tampok na iyon. Ang mga fields ay bahagyang nagkakaiba sa pagitan ng tools, resources at prompts.

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
        # Suriin ang input gamit ang Pydantic model
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: idagdag ang Pydantic, upang makagawa tayo ng AddInputModel at masuri ang mga argumento

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Dito makikita kung paano natin ginagawa ang mga sumusunod:

- Gumawa ng schema gamit ang Pydantic na `AddInputModel` na may mga field na `a` at `b` sa file na *schema.py*.
- Subukang i-parse ang papasok na request na maging uri ng `AddInputModel`, kung may hindi tugma sa mga parameter ito ay magreresulta sa crash:

   ```python
   # add.py
    try:
        # I-validate ang input gamit ang Pydantic na modelo
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Maaari mong piliin kung ilalagay ang parsing logic na ito sa mismong pagtawag ng tool o sa handler function.

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

- Sa handler na humahawak ng lahat ng mga tawag sa tool, sinusubukan na nating i-parse ang papasok na request sa schema na tinukoy ng tool:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    kung matagumpay iyon saka natin ipagpapatuloy ang pagtawag sa aktwal na tool:

    ```typescript
    const result = await tool.callback(input);
    ```

Makikita mo, ang paraang ito ay lumilikha ng magandang arkitektura dahil ang lahat ay may kanya-kanyang lugar, ang *server.ts* ay napakaliit na file na nag-uugnay lang sa mga request handler at ang bawat tampok ay nasa kani-kanilang folder gaya ng tools/, resources/ o /prompts.

Maganda, subukan nating itayo ito.

## Pagsasanay: Paglikha ng low-level server

Sa pagsasanay na ito, gagawin natin ang mga sumusunod:

1. Gumawa ng low-level server na humahawak sa paglista ng mga tools at pagtawag ng mga tools.
1. Magpatupad ng isang arkitektura na maaari mong pagtagumpayan.
1. Magdagdag ng validation upang matiyak ang wastong pagkakakilanlan ng iyong mga tawag sa tool.

### -1- Gumawa ng arkitektura

Ang unang kailangan nating ayusin ay ang arkitektura na tutulong sa atin na mag-scale habang nagdadagdag tayo ng mas maraming tampok, ganito ang hitsura nito:

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

Ngayon ay na-set up na natin ang arkitektura na nagsisiguro na madali tayong makadagdag ng mga bago pang tools sa loob ng tools folder. Malaya kang sundan ito upang magdagdag ng mga subdirectory para sa resources at prompts.

### -2- Paglikha ng isang tool

Tingnan natin kung paano gumawa ng isang tool. Una, kailangang ito ay malikha sa sariling *tool* subdirectory nito tulad nito:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Patunayan ang input gamit ang Pydantic modelo
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: idagdag ang Pydantic, para makagawa tayo ng AddInputModel at mapatunayan ang mga argumento

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Makikita natin dito kung paano itinakda ang name, description, isang input schema gamit ang Pydantic at isang handler na tatawagin kapag tinawag ang tool na ito. Sa huli, inilalantad ang `tool_add` na isang diksyunaryo na may hawak ng mga ari-ariang ito.

Mayroon ding *schema.py* na ginagamit para tukuyin ang input schema na ginagamit ng ating tool:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Kailangan din nating punan ang *__init__.py* upang matiyak na ang tools directory ay ituturing bilang isang module. Bukod dito, kailangan nating ilantad ang mga module dito tulad nito:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Maaari nating patuloy na dagdagan ang file na ito habang nagdadagdag pa tayo ng mga tools.

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

Dito gumawa tayo ng isang diksyunaryo na binubuo ng mga ari-arian:

- name, ito ang pangalan ng tool.
- rawSchema, ito ang schema ng Zod, gagamitin ito upang i-validate ang mga papasok na request para tawagin ang tool na ito.
- inputSchema, ang schema na ito ang gagamitin ng handler.
- callback, ito ang ginagamit upang tawagin ang tool.

Mayroon ding `Tool` na ginagamit para i-convert ang diksyunaryong ito sa isang uri na maaaring tanggapin ng mcp server handler at ganito ang itsura nito:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

At may *schema.ts* kung saan itinatago natin ang mga input schemas para sa bawat tool na ganito ang hitsura ngayon na isa lamang ang naka-schema ngunit habang nagdadagdag tayo ng mga tool maaari tayong magdagdag pa ng mga entry:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Maganda, magpatuloy tayo sa paghawak ng paglista ng ating mga tools.

### -3- Hawakan ang paglista ng mga tool

Sunod, para hawakan ang paglista ng ating mga tools, kailangan nating mag-set up ng request handler para dito. Ganito ang mga kailangan nating idagdag sa ating server file:

**Python**

```python
# nilaktawan ang code para sa maigsi
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

Dito nagdagdag tayo ng dekorator na `@server.list_tools` at ang implementing function na `handle_list_tools`. Sa huli, kailangan nating bumuo ng listahan ng tools. Pansinin na ang bawat tool ay kailangang may pangalan, deskripsyon at inputSchema.   

**TypeScript**

Upang mag-set up ng request handler para sa paglista ng mga tools, kailangan nating tawagin ang `setRequestHandler` sa server na may schema na angkop sa ginagawa natin, sa kasong ito ay `ListToolsRequestSchema`. 

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
// inalis ang code para sa ikinaikling bersyon
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Ibalik ang listahan ng mga nakarehistrong gamit
  return {
    tools: tools
  };
});
```

Maganda, ngayon ay nasolusyunan na natin ang bahagi ng paglista ng tools, tingnan naman natin kung paano tayo tatawag ng mga tools.

### -4- Hawakan ang pagtawag sa isang tool

Para tumawag ng tool, kailangan nating mag-set up ng isa pang request handler, ngayon nakatuon sa pagtanggap ng request na nagsasaad kung aling tampok ang tatawagin at anong mga argumento.

**Python**

Gamitin natin ang dekorator na `@server.call_tool` at ipatupad ito gamit ang isang function tulad ng `handle_call_tool`. Sa function na iyon, kailangan nating kunin ang pangalan ng tool, ang mga argumento at tiyakin na ang mga argumento ay wasto para sa tool na iyon. Maaari nating i-validate ang mga argumento dito o sa mismong tool.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # ang tools ay isang diksyunaryo na may mga pangalan ng kagamitan bilang mga susi
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # tawagin ang kagamitan
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Ganito ang nangyayari:

- Ang pangalan ng ating tool ay nasa input parameter na `name` na totoo rin para sa ating mga argumento sa anyo ng diksyunaryong `arguments`.

- Ang tool ay tinatawag gamit ang `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Ang pag-validate ng mga argumento ay nangyayari sa `handler` na property na tumuturo sa function, kung ito ay mabigo magreresulta ito sa exception. 

Ayan, ngayon may buong-unawa na tayo kung paano maglista at tumawag ng mga tools gamit ang low-level server.

Tingnan ang [kumpletong halimbawa](./code/README.md) dito

## Takdang-aralin

Palawakin ang code na ibinigay sa iyo gamit ang ilang mga tools, resources, at prompt at pagnilayan kung paano mo napapansin na kailangan mo lang magdagdag ng mga file sa tools na direktoryo at hindi na sa iba pa. 

*Walang ibinigay na solusyon*

## Buod

Sa kabanatang ito, nakita natin kung paano gumagana ang low-level server approach at kung paano tayo makakagawa ng magandang arkitektura na maaari nating patuloy na pagyamanin. Tinalakay din natin ang validation at ipinakita kung paano gumamit ng mga validation library para gumawa ng mga schema para sa input validation.

## Ano ang Susunod

- Susunod: [Simple Authentication](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Paunawa**:
Ang dokumentong ito ay isinalin gamit ang serbisyong AI na pagsasalin [Co-op Translator](https://github.com/Azure/co-op-translator). Bagamat aming pinagsisikapang maging tumpak, pakatandaan na ang mga awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o di-tumpak na impormasyon. Ang orihinal na dokumento sa kanyang orihinal na wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang mga maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
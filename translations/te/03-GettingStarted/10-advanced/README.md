# అధునాతన సర్వర్ ఉపయోగం

MCP SDKలో రెండు వేర్వేరు రకాల సర్వర్లు ఉన్నాయి, మీ సాధారణ సర్వర్ మరియు లో-లెవల్ సర్వర్. సాధారణంగా, మీరు సాధారణ సర్వర్‌ను ఉపయోగించి దీనిపై ఫీచర్లను జోడిస్తారు. కొన్ని సందర్భాల్లో, మీరు లో-లెవల్ సర్వర్‌పై ఆధారపడాలనుకుంటారు, ఉదాహరణకు:

- మెరుగైన ఆర్కిటెక్చర్. సాధారణ సర్వర్ మరియు లో-లెవల్ సర్వర్ రెండింటితో కలిపి శుభ్రంగా ఆర్కిటెక్చర్ సృష్టించడం సాధ్యమే కానీ  లో-లెవల్ సర్వర్‌తో ఇది కొంచెం సులభమని వాదించవచ్చు.
- ఫీచర్ అందుబాటు. కొన్ని అధునాతన ఫీచర్‌లు కేవలం లో-లెవల్ సర్వర్‌తో మాత్రమే ఉపయోగించవచ్చు. మీరు తర్వాతి అధ్యాయాల్లో శాంప్లింగ్ మరియు elicitation ను జోడిస్తున్నప్పుడు దీన్ని చూడగలుగుతారు.

## సాధారణ సర్వర్ vs లో-లెవల్ సర్వర్

ఇక్కడ సాధారణ సర్వర్‌తో MCP Server సృష్టి ఎలా ఉండాలి అనేది ఉంది

**Python**

```python
mcp = FastMCP("Demo")

# ఒక జోడింపు సాధనాన్ని జోడించండి
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

// ఒక అదనపు సాధనాన్ని జోడించండి
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

పాయింట్ ఏమిటంటే మీరు సర్వర్‌కు కావలసిన ప్రతి టూల్, వనరు లేదా ప్రాంప్ట్‌ను స్పష్టంగా జోడిస్తారు. ఇందులో దోషం ఏమీ లేదు.

### లో-లెవల్ సర్వర్ పరిష్కారం

కానీ, మీరు లో-లెవల్ సర్వర్ పద్ధతిని ఉపయోగించినప్పుడు వేరుగా ఆలోచించాల్సి ఉంటుంది, అంటే ప్రతి ఫీచర్ రకం (టూల్స్, వనరులు లేదా ప్రాంప్ట్‌లు) కోసం రెండు హ్యాండ్లర్లను సృష్టించాలి. ఉదాహరణకు, టూల్స్ కోసం కేవలం ఈ విధంగా రెండు ఫంక్షన్స్ ఉంటాయి:

- అన్ని టూల్స్‌ను లిస్ట్ చేయడం. ఒక ఫంక్షన్ టూల్స్‌ను లిస్ట్ చేయడానికి అన్ని ప్రయత్నాలు నిర్వహిస్తుంది.
- అన్ని టూల్స్‌ను కోల్ చేయడం. ఇక్కడ కూడా, ఒకే ఒక్క ఫంక్షన్ టూల్ పిలుపులను నిర్వహిస్తుంది.

ఇది కొంత పని తగ్గినట్టే అనిపిస్తోందా? కాబట్టి టూల్ రిజిస్టర్ చేయడం కాకుండా, నేను టూల్స్‌ను అన్నిటినీ లిస్ట్ చేసేటప్పుడు ఆ టూల్ చూపించబడడం మరియు టూల్ పిలుపు కోసం వచ్చిన అభ్యర్థనకు టూల్ పిలవబడటం మాత్రమే చూసుకోవాలి.

ఇప్పుడు కోడ్ ఎలా ఉన్నదో చూద్దాం:

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
  // నమోదు చేయబడిన సాధనాల జాబితాను తిరిగిస్తాయి
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

ఇక్కడ ఇప్పుడు ఒక ఫంక్షన్ ఉంది ఇది ఫీచర్ల జాబితాను తిరిగి ఇస్తుంది. టూల్స్ జాబితాలో ప్రతి ఎంట్రీకు `name`, `description` మరియు `inputSchema` వంటి ఫీల్డ్స్ ఉన్నాయి, ఇవి రిటర్న్ టైపు కోసం ఉండాలి. ఇది మన టూల్స్ మరియు ఫీచర్ నిర్వచనాన్ని వేరే చోట ఉంచుకోవడానికి వీలు కలిగి ఉంటుంది. ఇప్పుడు మనం అన్ని టూల్స్ ని tools ఫోల్డర్‌లో సృష్టించవచ్చు మరియు అన్ని ఫీచర్ల కోసం కూడా అదే విధంగా ఉంటే ప్రాజెక్ట్ ఇప్పుడు ఈ విధంగా సజావుగా ఏర్పడుతుంది:

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

అది చాలా బాగుంది, మన ఆర్కిటెక్చర్ చాలా శుభ్రముగా ఉండేలా చేయగలుగుతుంది.

టూల్స్ పిలవడం తేడా ఉందా, అదే ఆలోచననా? ఒక హ్యాండ్లర్ ఏ టూల్ అయినా పిలవడం కోసం? అవును, ఖచ్చితంగా, కోడ్ ఇలా ఉంటుంది:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools అనేది యంత్రపరికరాలతో కీలుగా ఉన్న ఒక నిఘంటువు
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
    
    // ఆర్గ్స్: request.params.arguments
    // TODO టూల్‌ను కాల్ చేయండి,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

పై కోడ్ నుండి మీరు చూడగలరుగా, మనం పిలవవలసిన టూల్ మరియు దానికి అవసరమైన ఆర్గ్యుమెంట్స్‌ను పార్స్ చేయాలి, ఆ తర్వాత టూల్ పిలవడం కొనసాగించాలి.

## ప్రమాణీకరణతో పద్ధతి మెరుగుపరచడం

ఇప్పటివరకు, మీరు చూసిన విధంగా మీ టూల్స్, వనరులు మరియు ప్రాంప్ట్‌లకు సంబంధించిన అన్ని రిజిస్ట్రేషన్లు ఈ రెండు హ్యాండ్లర్లతో భర్తీ చేయవచ్చు. మరేమి చేయాలి? మనం టూల్ సరిగ్గా పిలవబడుతున్నట్టు నిర్ధారించడానికి ప్రమాణీకరణను జోడించాలి. ప్రతి రన్‌టైమ్‌కు దీని కోసం తమ స్వంత పరిష్కారం ఉంటుంది, ఉదాహరణకు Python Pydantic ఉపయోగిస్తుంది మరియు TypeScript Zod వాడుతుంది. ఆలోచన ఇది:

- ఫీచర్ (టూల్, వనరు లేదా ప్రాంప్ట్) సృష్టించే లాజిక్‌ను దాని ప్రత్యేక ఫోల్డర్‌కి తరలించడం.
- టూల్ పిలవడం వంటి అభ్యర్థనకు ప్రమాణీకరణ చేస్తూ ఉండే ప్రఱియకుడు ఏర్పాటుచేయడం.

### ఫీచర్ సృష్టించండి

ఫీచర్‌ను సృష్టించడానికి, ఆ ఫీచర్ కు సంబంధించిన ఫైల్‌ని సృష్టించి, ఆ ఫీచర్‌కు అవసరమైన ఫీల్డ్స్ కలిగి ఉన్నదని నిర్ధారించాలి. టూల్స్, వనరులు, ప్రాంప్ట్‌లు మధ్యలో కొన్ని ఫీల్డ్స్ తేడా ఉంటాయి.

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
        # Pydantic మోడల్ ఉపయోగించి ఇన్‌పుట్‌ని ధృవీకరించండి
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic జోడించండి, తద్వారా మేము AddInputModel సృష్టించి ఆర్గిలను ధృవీకరించగలం

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ఇక్కడ మీరు ఇవి ఎలా చేయాలో చూడవచ్చు:

- Pydantic `AddInputModel` ఉపయోగించి *schema.py* ఫైల్‌లో `a` మరియు `b` అనే ఫీల్డ్స్ తో స్కీమాను సృష్టించడం.
- వస్తున్న అభ్యర్థనను `AddInputModel` టైపు గా పార్స్ చేయడానికి ప్రయత్నించడం, పామిటీలు సరిపోలకపోతే ఇది క్రాష్ అవుతుంది:

   ```python
   # add.py
    try:
        # Pydantic మోడల్ ఉపయోగించి ఇన్‌పుట్‌ను ధృవీకరించండి
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

ఈ పార్సింగ్ లాజిక్‌ని మీరు టూల్ కాల్‌లోనే లేదా హ్యాండ్లర్ ఫంక్షన్‌లో ఉంచుకోవచ్చు.

**TypeScript**

```typescript
// సర్వర్.ts
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

       // @ts-గమనించవద్దు
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

// స్కీమా.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// చేర్చు.ts
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

- టూల్ పిలుపులను నిర్వహించే హ్యాండ్లర్‌లో, వస్తున్న అభ్యర్థనను టూల్ నిర్వచించిన స్కీమాలోకి పార్స్ చేయడానికి ప్రయత్నించవచ్చు:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    ఇది సఫలమైతే అసలైన టూల్‌ను పిలవడానికి ముందుకు వెళ్ళుతాము:

    ```typescript
    const result = await tool.callback(input);
    ```

ఈ విధానం మంచి ఆర్కిటెక్చర్‌ను రూపొందిస్తుంది, ఎందుకంటే *server.ts* ఒక చిన్న ఫైల్ మాత్రమే, అది కేవలం రిక్వెస్ట్ హ్యాండ్లర్లను వయిర్ చేస్తుంది, మరియు ప్రతి ఫీచర్ తానే తన ఫోల్డర్ లో ఉంటుంది: tools/, resources/ లేదా /prompts/.

చాలా బాగుంది, ఇప్పుడు దీన్ని నిర్మించడం ప్రారంభిద్దాం.

## వ్యాయామం: లో-లెవల్ సర్వర్ సృష్టించడం

ఈ వ్యాయామంలో, మనం క్రింద ఉన్నవి చేస్తాం:

1. టూల్స్ లిస్ట్ మరియు టూల్స్ కాల్లింగ్ హ్యాండిల్ చేసే లో-లెవల్ సర్వర్ సృష్టించండి.
1. మీరు నిర్మించగల ఒక ఆర్కిటెక్చర్ అమలు చేయండి.
1. మీ టూల్ పిలుపులు సరైనవిగా ప్రమాణీకరించబడుతున్నాయని నిర్ధారించడానికి వాలిడేషన్ జోడించండి.

### -1- ఆర్కిటెక్చర్ సృష్టించండి

ముందుగా మనం శ్రేణి పెరగడానికి సహాయపడే ఆర్కిటెక్చర్ గురించి దృష్టి పెట్టాలి, ఇది ఇలా ఉంటుంది:

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

ఇప్పుడు మనం ఒక ఆర్కిటెక్చర్ ఏర్పాటు చేసాము, ఇది tools ఫోల్డర్‌లో కొత్త టూల్స్ సులభంగా జోడించడానికి అనుమతిస్తుంది. resources మరియు prompts కోసం ఉపఫోల్డర్లను జోడించడానికి ఈ పద్ధతిని అనుసరించండి.

### -2- టూల్ సృష్టించడం

తర్వాత టూల్ సృష్టించడం ఎలా ఉన్నదో చూద్దాం. మొదట, ఇది దాని *tool* ఉపఫోల్డర్‌లో ఇలా సృష్టించాలి:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic మోడల్‌ను ఉపయోగించి ఇన్‌పుట్‌ను త<|vq_image_2521|><|vq_image_6234|><|vq_image_9061|><|vq_image_6393|><|vq_image_10598|><|vq_image_12520|><|vq_image_6393|><|vq_image_16304|><|vq_image_6393|><|vq_image_6393|><|vq_image_8263|><|vq_image_6393|><|vq_image_8027|><|vq_image_6718|><|vq_image_1877|><|vq_image_10598|><|vq_image_10598|><|vq_image_14797|><|vq_image_7192|><|vq_image_6035|><|vq_image_6234|><|vq_image_9061|><|vq_image_9061|><|vq_image_11206|><|vq_image_8971|><|vq_image_11870|><|vq_image_11326|><|vq_image_12438|><|vq_image_12600|><|vq_image_4015|><|vq_image_14081|><|vq_image_13557|><|vq_image_3114|><|vq_image_5465|><|vq_image_12554|><|vq_image_6609|><|vq_image_6417|><|vq_image_9276|><|vq_image_12060|><|image_border_927|><|vq_image_12683|><|vq_image_6173|><|vq_image_6173|><|vq_image_1432|><|vq_image_12991|><|vq_image_1432|><|vq_image_12326|><|vq_image_12326|><|vq_image_12326|><|vq_image_12326|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_12326|><|vq_image_12326|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_175|><|vq_image_8895|><|vq_image_5971|><|vq_image_4687|><|vq_image_1635|><|vq_image_6530|><|vq_image_1384|><|vq_image_9360|><|vq_image_2593|><|vq_image_13495|><|vq_image_12500|><|vq_image_4915|><|vq_image_13552|><|vq_image_13023|><|vq_image_9276|><|vq_image_1083|><|vq_image_1635|>
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: add Pydantic, so we can create an AddInputModel and validate args

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ఇక్కడ టూల్ పేరు, వివరణ, Pydantic ఉపయోగించి ఇన్‌పుట్ స్కీਮਾ మరియు టూల్ పిలబడినప్పుడు పిలయ్యే హ్యాండ్లర్ నిర్వచించడం ఎలా జరుగుతుందో చూపిస్తుంది. చివరగా, `tool_add` అనే డిక్షనరీ అందుబాటులో ఉంచడం జరిగింది, ఇందులో అన్ని ప్రాపర్టీలూ ఉంటాయి.

ఇంకా *schema.py* ఉంది, ఇది మన టూల్ ఉపయోగించే ఇన్‌పుట్ స్కీమాను నిర్వచిస్తుంది:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

మరియు *__init__.py* ని tools డైరెక్టరీ మాడ్యూల్ గా పరిగణింపచేసుకునేందుకు పూరింపుచేయాలి. అదనంగా, అందులోని మాడ్యూల్స్‌ను ఇలా ఎక్స్‌పోర్స్ చేయాలి:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

పలుమార్లు టూల్స్ జోడించగానే ఈ ఫైల్‌ను కొనసాగించి అప్డేట్ చేశారు.

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

ఇక్కడ మనం క్రింది ప్రాపర్టీస్ కలిగిన డిక్షనరీ తయారు చేస్తాం:

- name, ఇది టూల్ యొక్క పేరు.
- rawSchema, ఇది Zod స్కీమా, ఇది టూల్ పిలవడానికి వస్తున్న అభ్యర్థనలను నిర్ధారించడానికి ఉపయోగిస్తారు.
- inputSchema, ఇది హ్యాండ్లర్ ఉపయోగించే స్కీమా.
- callback, ఇది టూల్‌ను పిలవడానికి ఉపయోగిస్తారు.

ఇంకా `Tool` ఉంది, ఇది ఈ డిక్షనరీని mcp సర్వర్ హ్యాండ్లర్ ఆమోదించే టైప్‌గా మార్చుతుంది, ఇది ఇలా ఉంటుంది:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

మరియు *schema.ts* కూడా ఉంది, మనం ఇక్కడ ప్రతి టూల్ కోసం ఇన్‌పుట్ స్కీమాలను నిల్వ చేస్తాం, ప్రస్తుతం కేవలం ఒక్క స్కీమా ఉన్నా, టూల్స్ పెరిగినప్పుడు entries పెరుగుతాయి:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

మంచిది, ఇప్పుడు మన టూల్స్‌ని లిస్ట్ చేయడం ఎలా నిర్వహించాలో చూద్దాం.

### -3- టూల్స్ లిస్టింగ్ హ్యాండిల్ చేయండి

టూల్స్ లిస్ట్ చేయడానికి, ఒక రిక్వెస్ట్ హ్యాండ్లర్ సెట్ చేయాలి. ఇది మన సర్వర్ ఫైల్‌లో ఇలా జోడించాలి:

**Python**

```python
# సారాంశం కోసం కోడ్ తీసివేయబడింది
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

మనం ఇక్కడ `@server.list_tools` డెకొరేటర్ మరియు `handle_list_tools` సంకలింపుని జోడిస్తున్నాము. ఇందులో, టూల్స్ జాబితా ఉత్పత్తి చేయాలి. ప్రతి టూల్ కు name, description, మరియు inputSchema ఉండాలి.

**TypeScript**

టూల్స్ లిస్ట్ చేయడానికి రిక్వెస్ట్ హ్యాండ్లర్ సెట్ చేయడం అంటే సర్వర్ మీద `setRequestHandler` కాల్ చేయడం, ఇది మనం చేయబోయే ప్రక్రియకు అనుగుణమైన స్కీమాతో, ఈ కేసులో `ListToolsRequestSchema`.

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
// సంక్షిప్తత కోసం కోడ్ మినహాయించబడింది
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // నమోదు చేసిన పరికరాల జాబితాను վերադարձించండి
  return {
    tools: tools
  };
});
```

బాగుంది, ఇప్పుడు మనం టూల్స్ లిస్ట్ అంశాన్ని పరిష్కరించాము, తర్వాత టూల్స్ పిలవడం ఎలా ఉంటుంది చూడండి.

### -4- టూల్ పిలవడం హ్యాండిల్ చేయండి

టూల్ పిలవడానికి మరో రిక్వెస్ట్ హ్యాండ్లర్ సెట్ చేయాలి, ఇది పిలవవలసిన ఫీచర్ మరియు దానికి అవసరమైన ఆర్గ్యుమెంట్స్ ని సంకేతం చేసే అభ్యర్థనపై దృష్టి పెట్టాలి.

**Python**

`@server.call_tool` డెకొరేటర్ ఉపయోగించి `handle_call_tool` వంటి ఫంక్షన్‌ని అమలు చేయండి. ఆ ఫంక్షన్లో టూల్ పేరు, ఆర్గ్యుమెంట్స్‌ను పార్స్ చేసి వాటి సరైనత నిర్ధారించాలి. ఆర్గ్యుమెంట్స్ నిర్ధారణ ఈ ఫంక్షన్‌లో లేదా టూల్ లోపల నేను చేయవచ్చు.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools అనేది సాధన పేర్లను కీలు గా కలిగి ఉన్న ఒక డిక్షనరీ
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # సాధనాన్ని ఆహ్వానించండి
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

ఇక్కడ జరుగుతుందంటే:

- మన టూల్ పేరు `name` అనే ఇన్‌పుట్ పరామితి ద్వారా ఇప్పటికే ఉంది, మరియు `arguments` అనే డిక్షనరీలో ఆర్గ్యుమెంట్లు ఉన్నాయి.

- టూల్ `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` తో పిలవబడుతుంది. ఆర్గ్యుమెంట్స్ నిర్ధారణ `handler` ప్రాపర్టీలో జరుగుతుంది ఇది ఒక ఫంక్షన్‌ని సూచిస్తుంది; ఫెయిలైతే ఎక్సెప్షన్ తేవడంతో ఆగుతుంది.

ఇది ప్రదర్శిస్తుంది, లో-లెవల్ సర్వర్ ఉపయోగించి టూల్స్ లిస్ట్ మరియు కాల్ చేయడంలో పూర్తి అవగాహన.

[పూర్తి ఉదాహరణ](./code/README.md) ఇక్కడ చూడండి.

## అసైన్‌మెంట్

మీకు ఇచ్చిన కోడ్‌ను అనేక టూల్స్, వనరులు మరియు ప్రాంప్ట్‌లతో విస్తరించి, మీరు మాత్రమే tools డైరెక్టరీలో ఫైల్స్ జోడించాల్సి ఉంటుందని గమనించి దానిపై ప్రతిబింబించండి.

*ఏ పరిష్కారం ఇవ్వలేదు*

## సారాంశం

ఈ అధ్యాయంలో, లో-లెవల్ సర్వర్ పద్ధతి ఎలా పనిచేస్తుందో మరియు అదే మనం నిర్మించగల మంచి ఆర్కిటెక్చర్ ఎలా సృష్టించగలమో చూశాము. అలాగే, ప్రమాణీకరణ గురించి చర్చించి, మనం ఇన్‌పుట్ వాలిడేషన్ కోసం పద్ధతులు సిద్ధం చేసే విధానాన్ని వర్ణించాము.

## తర్వాత ఏమిటి

- తర్వాత: [సాధారణ ధృవీకరణ](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అस्वీకరణ**:
ఈ డాక్యుమెంట్‌ను AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మనం ఖచ్చితత్వానికి ప్రయత్నించినప్పటికీ, ఆటోమేటిగ్గా రూపొందించిన అనువాదాలలో లోపాలు లేదా తప్పులొ ఉండే అవకాశం ఉందని దయచేసి గుర్తుంచుకోండి. మౌలిక భాషలో ఉన్న అసలైన డాక్యుమెంట్‌ను అధికారిక మూలంగా పరిగణించాలి. ముఖ్యమైన సమాచారానికి, వృత్తిపరమైన మానవ అనువాదాన్ని సూచిస్తాము. ఈ అనువాదం వలన కలిగిన ఏవైనా అపార్థాలు లేదా తప్పైన అర్థవ్యాఖ్యల కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
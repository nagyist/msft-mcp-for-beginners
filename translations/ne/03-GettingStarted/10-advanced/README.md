# उच्च स्तरको सर्भर प्रयोग

MCP SDK मा दुई फरक प्रकारका सर्भरहरू प्रदान गरिन्छ, तपाईंको सामान्य सर्भर र लो-लेवल सर्भर। सामान्य रूपमा, तपाईंले यसको सुविधाहरू थप्न सामान्य सर्भर प्रयोग गर्नुहुनेछ। तर केही अवस्थामा, तपाईंले लो-लेवल सर्भरमा निर्भर रहन चाहनुहुन्छ जस्तै:

- राम्रो संरचना। दुवै सामान्य सर्भर र लो-लेवल सर्भरसँग सफा संरचना बनाउन सकिन्छ तर यो बहस गर्न सकिन्छ कि लो-लेवल सर्भरसँग यो अलि सजिलो हुन्छ।
- सुविधा उपलब्धता। केही उन्नत सुविधाहरू केवल लो-लेवल सर्भरसँग मात्र प्रयोग गर्न सकिन्छ। तपाईंले यो पछि अध्यायहरूमा देख्नुहुनेछ जस्तै नमुना संकलन र प्रवृत्ति निर्धारण थप्दा।

## सामान्य सर्भर बनाम लो-लेवल सर्भर

यहाँ सामान्य सर्भरसँग MCP Server कसरी तयार गरिन्छ:

**Python**

```python
mcp = FastMCP("Demo")

# एक योग उपकरण थप्नुहोस्
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

// थप्ने उपकरण थप्नुहोस्
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

मुख्य कुरा यो हो कि तपाईंले सर्भरमा आवश्यक प्रत्येक उपकरण, स्रोत वा प्रॉम्प्टलाई स्पष्ट रूपमा थप्नुहुन्छ। त्यसमा कुनै समस्या छैन।  

### लो-लेवल सर्भर दृष्टिकोण

तर, जब तपाईं लो-लेवल सर्भर दृष्टिकोण प्रयोग गर्नुहुन्छ भने तपाईंले फरक तरीकाले सोच्न आवश्यक हुन्छ, अर्थात् प्रत्येक उपकरण दर्ता गर्ने बजाय तपाईंले प्रत्येक सुविधा प्रकार (उपकरण, स्रोत वा प्रॉम्प्ट) को लागि दुई ह्यान्डलरहरू बनाउनु पर्छ। उदाहरणका लागि उपकरणहरूका लागि केवल दुई कार्य हुन्छन्:

- सबै उपकरणहरूको सूची बनाउने। एउटा कार्यले सबै उपकरणहरूको सूची बनाउने प्रयासहरूको जिम्मेवारी लिन्छ।
- सबै उपकरणहरूको कल ह्यान्डल गर्ने। यहाँ पनि, केवल एउटा कार्यले उपकरणलाई कल गर्ने कार्यको ह्यान्डलिंग गर्छ।

यो सम्भावित रूपमा कम काम जस्तो सुनिन्छ, होइन र? त्यसैले उपकरण दर्ता गर्ने सट्टा, म केवल सुनिश्चित गर्नुपर्छ कि उपकरण सूचीमा समावेश छ र आउँदै गरेको अनुरोधमा उपकरण कल गर्न सकिन्छ।

अब कोड कस्तो देखिन्छ हेर्नुहोस्:

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
  // दर्ता गरिएका उपकरणहरूको सूची फर्काउनुहोस्
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

यहाँ हामीसँग अब एउटा फङ्क्शन छ जसले सुविधा (features) को सूची फर्काउँछ। उपकरण सूचीमा हरेक प्रविष्टिमा `name`, `description` र `inputSchema` जस्ता क्षेत्रहरू छन् जुन रिटर्न प्रकारको पालना गर्दछ। यसले हामीलाई उपकरणहरू र सुविधा परिभाषाहरू अन्यत्र राख्न सक्षम बनाउँछ। अब हामी सबै उपकरणहरू उपकरणहरू फोल्डरमा बनाउन सक्छौं र सबै सुविधा लागि पनि सोही हुन्छ जसले तपाईंको परियोजनालाई यसरी व्यवस्थित बनाउन सक्छ:

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

यो राम्रो छ, हाम्रो संरचनालाई सफा देखिने बनाउन सकिन्छ।

उपकरणहरू कल गर्ने कुरा के हो, के यो एउटै विचार हो, एउटै ह्यान्डलरले कुनै पनि उपकरण कल गर्ने? हो, बिल्कुल, यहाँ त्यो कोड छ:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools एउटा डिक्सनरी हो जसमा उपकरण नामहरू कुञ्जीहरूका रूपमा छन्
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
    // TODO उपकरण कल गर्नुहोस्,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

माथिको कोडबाट देख्न सकिन्छ, हामीलाई कल गर्नुपर्ने उपकरण र त्यसका कस्ता तर्कहरू छन् भनेर पार्स गर्नुपर्छ, अनि उपकरणलाई कल गर्न अघि बढ्नुपर्छ।

## प्रमाणीकरणसहित दृष्टिकोण सुधार

अहिलेसम्म, तपाईंले देख्नुभयो कि कसरी तपाईंले उपकरण, स्रोत र प्रॉम्प्टहरू थप्न गरेको सबै दर्ता यो दुई ह्यान्डलरहरूमा प्रतिस्थापन गर्न सकिन्छ। अब अरू के गर्नु पर्छ? हामीले केही प्रकारको प्रमाणीकरण थप्नुपर्छ जसले सुनिश्चित गर्छ कि उपकरण सही तर्कहरूका साथ कल गरिएको छ। प्रत्येक रनटाइमले यसको आफ्नै समाधान प्रयोग गर्छ, उदाहरणका लागि Python ले Pydantic प्रयोग गर्छ र TypeScript ले Zod। विचार यो हो:

- सुविधा (उपकरण, स्रोत वा प्रॉम्प्ट) बनाउने तर्कलाई यसको निर्दिष्ट फोल्डरमा सार्नु।
- उदाहरणका लागि उपकरण कल गर्न आउँदै गरेको अनुरोध प्रमाणीकरण गर्ने तरिका थप्नु।

### सुविधा सिर्जना गर्नुहोस्

सुविधा बनाउन, हामीलाई त्यो सुविधाको लागि फाइल बनाउनु पर्छ र सुनिश्चित गर्नु पर्छ कि त्यहाँ त्यो सुविधाका लागि अनिवार्य क्षेत्रहरू छन्। यी क्षेत्रहरू उपकरण, स्रोत र प्रॉम्प्टहरूमा थोरै फरक पर्छन्।

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
        # Pydantic मोडल प्रयोग गरी इनपुट मान्य गर्नुहोस्
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic थप्नुहोस्, ताकि हामी AddInputModel सिर्जना गर्न र args मान्य गर्न सकौं

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

यहाँ हामीले यसरी गर्छौं:

- Pydantic `AddInputModel` प्रयोग गरी `schema.py` फाइलमा `a` र `b` क्षेत्रहरू सहित स्किमा बनाउँछौं।
- आउँदै गरेको अनुरोधलाई `AddInputModel` को रूपमा पार्स गर्न प्रयास गर्छौं, यदि प्यारामिटरहरूमा मिल्दैन भने यो क्र्यास हुनेछ:

   ```python
   # add.py
    try:
        # Pydantic मोडेल प्रयोग गरी इनपुट मान्य गर्नुहोस्
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

तपाईंले यो पार्सिङ लॉजिक उपकरण कलमै राख्न वा ह्यान्डलर फङ्क्शनमा राख्ने निर्णय गर्न सक्नुहुन्छ।

**TypeScript**

```typescript
// सर्भर.ts
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

// स्कीमा.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// थप्नुहोस्.ts
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

- सबै उपकरण कलसँग डिल गर्ने ह्यान्डलरमा, हामी अब उपकरणले परिभाषित गरेको स्किमा अनुसार आउँदै गरेको अनुरोध पार्स गर्न प्रयास गर्छौं:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    यदि ठीक भयो भने हामी वास्तविक उपकरणलाई कल गर्छौं:

    ```typescript
    const result = await tool.callback(input);
    ```

जस्तै देख्न सकिन्छ, यस दृष्टिकोणले राम्रो संरचना सिर्जना गर्छ किनभने सबै कुरा आफ्नो ठाउँमा छ, *server.ts* एकदम सानो फाइल हो जसले अनुरोध ह्यान्डलरहरू जोड्छ र प्रत्येक सुविधा आफ्नो निर्दिष्ट फोल्डरमा हुन्छ जस्तै tools/, resources/ वा /prompts।

ठीक छ, अब यो बनाउने प्रयास गरौं। 

## अभ्यास: लो-लेवल सर्भर बनाउने

यस अभ्यासमा, हामीले यसरी गर्नेछौं:

1. उपकरणहरूको सूची र उपकरण कललाई हेर्ने लो-लेवल सर्भर बनाउने।
2. यस्तो संरचना कार्यान्वयन गर्ने जसमा तपाईंले बिस्तार गर्न सक्नुहुन्छ।
3. तपाईंको उपकरण कलहरू ठीकसँग प्रमाणीकरण भएको सुनिश्चित गर्न प्रमाणीकरण थप्ने।

### -1- संरचना तयार पार्नुहोस्

पहिलो कुरा जुन हामीलाई ध्यान दिनु छ त्यो संरचना हो जसले हामीलाई सुविधाहरू थप्दै बढ्न मद्दत गर्छ, यसरी देखिन्छ:

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

अब हामीले यस्तो संरचना सेटअप गरेका छौं जसले सजिलैसँग उपकरणहरू tools फोल्डरमा थप्न सकिन्छ। स्रोत र प्रॉम्प्टका लागि पनि उपफोल्डरहरू थप्न स्वतन्त्र हुनुहोस्।

### -2- उपकरण बनाउने

अर्को, उपकरण बनाउन कस्तो देखिन्छ हेर्नुहोस्। पहिलो, यो आफ्नो *tool* उपडाइरेक्टरीमा बन्नुपर्छ जस्तै:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic मोडेल प्रयोग गरी इनपुट प्रमाणित गर्नुहोस्
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic थप्नुहोस्, त्यसैले हामी AddInputModel बनाउन र args प्रमाणित गर्न सक्छौं

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

यहाँ हामीले कसरी नाम, विवरण, Pydantic प्रयोग गरेर इनपुट स्किमा र एक ह्यान्डलर परिभाषित गर्ने देख्न सकिन्छ जुन उपकरण कल गर्दा प्रयोग हुने छ। अन्ततः, हामी `tool_add` नामक डिक्सनरी बाहिर पठाउँछौं जसले यी सबै गुणहरू समावेश गर्दछ।

त्यसैगरी *schema.py* छ जुन उपकरणले प्रयोग गर्ने इनपुट स्किमा परिभाषित गर्न प्रयोग हुन्छ:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

हामीले *__init__.py* पनि भर्नु पर्छ ताकि tools डाइरेक्टरीलाई मोड्युलको रूपमा मान्न सकियोस्। थप रूपमा, हामी भित्रका मोड्युलहरू यसरी बाहिर पठाउनुपर्छ:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

हामी यस फाइलमा नयाँ उपकरणहरू थप्दै जान सक्छौं।

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

यहाँ हामी गुणहरूको डिक्सनरी बनाउँछौं:

- name, उपकरणको नाम।
- rawSchema, Zod स्किमा, यो उपकरण कल गर्ने इनकमिङ अनुरोधहरू प्रमाणीकरण गर्न प्रयोग हुन्छ।
- inputSchema, यो स्किमा ह्यान्डलरद्वारा प्रयोग हुन्छ।
- callback, यो उपकरणलाई invoke गर्न प्रयोग हुन्छ।

त्यहाँ `Tool` पनि छ जुन यो डिक्सनरीलाई मcp सर्भर ह्यान्डलरले स्वीकार्ने प्रकारमा रूपान्तरण गर्छ र यसरी देखिन्छ:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

र *schema.ts* जहाँ हामी प्रत्येक उपकरणका इनपुट स्किमा भण्डारण गर्छौं जुन हाल एक मात्रै स्किमा छ तर जब उपकरण थपिन्छन् थप प्रविष्टिहरू थप्न सक्छौं:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

राम्रो, अब उपकरणहरूको सूची हेर्ने कार्य ह्यान्डल गर्ने प्रयास गरौं।

### -3- उपकरण सूची ह्यान्डल गर्ने

अर्को, उपकरणहरूको सूची ह्यान्डल गर्न हामीलाई अनुरोध ह्यान्डलर सेटअप गर्नुपर्छ। सर्भर फाइलमा यसो थप्नुहोस्:

**Python**

```python
# संक्षिप्तताको लागि कोड हटाइएको
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

यहाँ हामी `@server.list_tools` डेकोरेटर र कार्यान्वयन गर्ने फङ्क्शन `handle_list_tools` थप्छौं। यसले उपकरणहरूको सूची प्रदान गर्नुपर्छ। हरेक उपकरणको नाम, विवरण र इनपुटस्किमा हुनुपर्ने ध्यान दिनुहोस्।   

**TypeScript**

उपकरण सूची हेर्ने अनुरोध ह्यान्डलर सेटअप गर्न हामीले सर्भरसँग `setRequestHandler` बोलाएर यहाँ `ListToolsRequestSchema` जस्तो स्किमालाई फिट पार्ने काम गर्छौं। 

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
// संक्षिप्तताका लागि कोड हटाइएको
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // दर्ता भएका उपकरणहरूको सूची फिर्ता गर्नुहोस्
  return {
    tools: tools
  };
});
```

राम्रो, अब हामीले उपकरणहरूको सूची बनाउने टुक्रा समाधान गर्यौं, अब उपकरणहरू कसरी कल गर्ने कुरा हेरौं।

### -4- उपकरण कल ह्यान्डल गर्ने

उपकरण कल गर्न हामी अर्को अनुरोध ह्यान्डलर सेटअप गर्नुपर्छ, यो पटक कुन सुविधा कल गर्ने र कस्ता तर्कसहित भन्ने कुरामा केन्द्रित।

**Python**

हामी `@server.call_tool` डेकोरेटर प्रयोग गरेर `handle_call_tool` फङ्क्शनमार्फत कार्यान्वयन गरौं। यस फङ्क्शनमा, हामी उपकरणको नाम, यसको तर्कहरू पार्स गर्नुपर्छ र उपकरणलाई लागि तर्कहरू मान्य छन् कि छैनन् सुनिश्चित गर्नुपर्छ। तर्कहरूको प्रमाणीकरण यस फङ्क्शनमा वा उपकरणभित्र गर्न सकिन्छ।

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # उपकरणहरू कुञ्जीहरूका रूपमा उपकरण नामहरूसँगको शब्दकोश हो
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # उपकरणलाई सक्रिय पार्नुहोस्
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

यहाँ के भइरहेको छ:

- हाम्रो उपकरण नाम पहिले नै `name` नामक इनपुट प्यारामिटरमा छ जुन `arguments` डिक्सनरीमा तर्कहरूको लागि सत्य हो।

- उपकरणलाई `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` प्रयोग गरी कल गरिन्छ। तर्कहरूको प्रमाणीकरण ह्यान्डलर गुणस्तर मा हुन्छ जुन कुनै फङ्क्शनलाई जनाउँछ, यदि असफल भयो भने अपवाद उठ्छ। 

त्यसैले, अब हामीसँग लो-लेवल सर्भर प्रयोग गरी उपकरणहरूको सूची बनाउने र कल गर्ने पूर्ण बुझाइ छ।

यहाँ [पूर्ण उदाहरण](./code/README.md) हेर्नुहोस्

## असाइनमेन्ट

तपाईंले पाएको कोडमा धेरै उपकरणहरू, स्रोतहरू र प्रॉम्प्टहरू थप्नुहोस् र यसो गर्नाले तपाईंले केवल tools डाइरेक्टरीमा फाइलहरू थप्नुपर्ने मात्र देख्नुहुन्छ र अरु कुनै ठाउँमा होइन।

*कोई समाधान प्रदान गरिएको छैन*

## सारांश

यस अध्यायमा, हामीले देख्यौं कि लो-लेवल सर्भर दृष्टिकोण कसरी काम गर्छ र कसरी यसले राम्रो संरचना सिर्जना गर्न मद्दत गर्दछ जसमा हामीले निरन्तर विकास गर्नसक्नेछौं। हामीले प्रमाणीकरण पनि छलफल गर्यौं र कसरी प्रमाणीकरण पुस्तकालयहरू प्रयोग गरी इनपुट प्रमाणीकरणका लागि स्किमा बनाउन सकिन्छ भनेर देखियो।

## के आउनेछ

- अर्को: [साधारण प्रमाणीकरण](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरी अनूदित गरिएको हो। हामी शुद्धताका लागि प्रयासरत छौं, तर कृपया बुझ्नुस् कि स्वचालित अनुवादहरूमा त्रुटिहरू वा अशुद्धता हुनसक्छ। मूल भाषा मा रहेको दस्तावेजलाई अधिकारिक स्रोतको रूपमा लिनुहोला। महत्वपूर्ण जानकारीको लागि पेशेवर मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलतफहमी वा गलत व्याख्या को लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
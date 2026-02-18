# उन्नत सर्वर उपयोग

MCP SDK में दो अलग-अलग प्रकार के सर्वर होते हैं, आपका सामान्य सर्वर और लो-लेवल सर्वर। सामान्यतः, आप फीचर्स जोड़ने के लिए नियमित सर्वर का उपयोग करेंगे। हालांकि कुछ मामलों में, आप लो-लेवल सर्वर पर निर्भर रहना चाहेंगे जैसे कि:

- बेहतर आर्किटेक्चर। यह दोनों नियमित सर्वर और लो-लेवल सर्वर के साथ एक स्वच्छ आर्किटेक्चर बनाने के लिए संभव है, लेकिन यह तर्क दिया जा सकता है कि यह लो-लेवल सर्वर के साथ थोड़ा आसान है।
- फीचर उपलब्धता। कुछ उन्नत फीचर केवल लो-लेवल सर्वर के साथ ही उपयोग किए जा सकते हैं। आप इसे बाद के अध्यायों में देखेंगे जब हम सैंपलिंग और इलिसिटेशन जोड़ेंगे।

## नियमित सर्वर बनाम लो-लेवल सर्वर

यहाँ एक MCP सर्वर के निर्माण का उदाहरण है सामान्य सर्वर के साथ

**Python**

```python
mcp = FastMCP("Demo")

# एक जोड़ उपकरण जोड़ें
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

// एक जोड़ने का उपकरण जोड़ें
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

मूल बात यह है कि आप स्पष्ट रूप से प्रत्येक टूल, संसाधन या प्रॉम्प्ट जोड़ते हैं जिसे आप सर्वर में चाहते हैं। इसमें कोई गलत बात नहीं है।

### लो-लेवल सर्वर दृष्टिकोण

हालांकि, जब आप लो-लेवल सर्वर दृष्टिकोण का उपयोग करते हैं तो आपको अलग तरह से सोचना होगा, अर्थात् कि प्रत्येक टूल को रजिस्टर करने के बजाय आप फीचर प्रकार (टूल्स, संसाधन या प्रॉम्प्ट) के लिए दो हैंडलर बनाते हैं। उदाहरण के लिए, टूल्स के लिए केवल दो फंक्शन होंगे:

- सभी टूल सूचीबद्ध करना। एक फंक्शन सभी टूल सूचीबद्ध करने के प्रयासों के लिए जिम्मेदार होगा।
- सभी टूल कॉल को हैंडल करना। यहाँ भी, केवल एक फंक्शन टूल कॉल को संभाल रहा होगा।

यह शायद कम काम लगता है, है ना? तो टूल रजिस्टर करने के बजाय, मुझे केवल यह सुनिश्चित करना है कि टूल सूची में शामिल हो जब मैं सभी टूल सूचीबद्ध करता हूँ और वह कॉल हो जब कोई टूल कॉल करने का अनुरोध करे।

आइए देखें कि अब कोड कैसा दिखता है:

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
  // पंजीकृत उपकरणों की सूची लौटाएं
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

यहाँ अब हमारे पास एक फंक्शन है जो फीचर्स की सूची लौटाता है। टूल सूची में प्रत्येक प्रविष्टि में अब `name`, `description` और `inputSchema` जैसे फ़ील्ड हैं ताकि वापसी प्रकार का पालन किया जा सके। इससे हम अपने टूल्स और फीचर परिभाषा अन्यत्र रख सकते हैं। अब हम अपने सभी टूल्स को tools फ़ोल्डर में बना सकते हैं और इसी तरह आपके सभी फीचर्स के लिए भी, जिससे आपका प्रोजेक्ट अचानक इस तरह व्यवस्थित हो सकता है:

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

बहुत अच्छा है, हमारी आर्किटेक्चर काफी साफ़ दिख सकती है।

टूल्स कॉल करने के बारे में क्या, क्या वह भी समान विचार है, एक हैंडलर किसी भी टूल को कॉल करने के लिए? हाँ, बिल्कुल, यहाँ उसका कोड है:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools एक शब्दकोश है जिसमें टूल के नाम कुंजियों के रूप में हैं
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
    
    // आर्ग्स: request.params.arguments
    // TODO टूल कॉल करें,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

जैसा कि आप ऊपर के कोड से देख सकते हैं, हमें टूल कॉल करना है, और किस तर्क के साथ, इसे पार्स करना होगा, और फिर टूल को कॉल करना होगा।

## मान्यकरण के साथ दृष्टिकोण सुधारना

अब तक, आपने देखा कि आपके सभी रजिस्ट्रेशन टूल्स, संसाधन और प्रॉम्प्ट जोड़ने के लिए इन दो हैंडलरों के साथ प्रतिस्थापित किए जा सकते हैं। हमें और क्या करना चाहिए? खैर, हमें कुछ प्रकार का मान्यकरण जोड़ना चाहिए ताकि यह सुनिश्चित किया जा सके कि टूल सही तर्क के साथ कॉल हो। प्रत्येक रनटाइम का इसका अपना समाधान होता है, उदाहरण के लिए Python Pydantic का उपयोग करता है और TypeScript Zod का। विचार यह है कि हम निम्न करें:

- किसी फीचर (टूल, संसाधन या प्रॉम्प्ट) को बनाने की लॉजिक को उसके समर्पित फ़ोल्डर में स्थानांतरित करें।
- एक ऐसा तरीका जोड़ें जो आने वाले अनुरोध को मान्य कर सके, उदाहरण के लिए किसी टूल को कॉल करने के लिए।

### एक फीचर बनाना

फीचर बनाने के लिए, हमें उस फीचर के लिए एक फ़ाइल बनानी होगी और यह सुनिश्चित करना होगा कि उसमें उस फीचर के लिए आवश्यक अनिवार्य फ़ील्ड हों। ये फ़ील्ड टूल्स, संसाधन और प्रॉम्प्ट के बीच थोड़ा भिन्न होते हैं।

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
        # Pydantic मॉडल का उपयोग करके इनपुट को मान्य करें
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic जोड़ें, ताकि हम एक AddInputModel बना सकें और आर्ग्स को मान्य कर सकें

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

यहाँ आप देख सकते हैं कि हम निम्न कर रहे हैं:

- Pydantic के साथ एक स्कीमा बनाना `AddInputModel` जिसमें फ़ील्ड `a` और `b` हैं, फ़ाइल *schema.py* में।
- आने वाले अनुरोध को `AddInputModel` प्रकार का पार्स करने का प्रयास करना, अगर पैरामीटर में असंगति है तो यह क्रैश हो जाएगा:

   ```python
   # add.py
    try:
        # Pydantic मॉडल का उपयोग करके इनपुट मान्य करें
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

आप चुन सकते हैं कि इस पार्सिंग लॉजिक को टूल कॉल में रखें या हैंडलर फंक्शन में।

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

- सभी टूल कॉल को हैंडल करने वाले हैंडलर में, अब हम आने वाले अनुरोध को टूल द्वारा परिभाषित स्कीमा में पार्स करने का प्रयास करते हैं:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    यदि यह काम करता है तो हम असली टूल को कॉल करना जारी रखते हैं:

    ```typescript
    const result = await tool.callback(input);
    ```

जैसा कि आप देख सकते हैं, यह दृष्टिकोण एक शानदार आर्किटेक्चर बनाता है क्योंकि सब कुछ अपनी जगह पर है, *server.ts* एक बहुत ही छोटी फ़ाइल है जो केवल रिक्वेस्ट हैंडलर्स को कनेक्ट करती है और प्रत्येक फीचर अपने संबंधित फ़ोल्डर में होता है जैसे tools/, resources/ या /prompts।

बहुत अच्छा, आइए इसे बनाना शुरू करें।

## अभ्यास: लो-लेवल सर्वर बनाना

इस अभ्यास में, हम निम्न करेंगे:

1. टूल्स की सूची बनाना और टूल कॉल करने वाला एक लो-लेवल सर्वर बनाएं।
2. एक ऐसी आर्किटेक्चर लागू करें जिस पर आप निर्माण कर सकें।
3. यह सुनिश्चित करने के लिए मान्यकरण जोड़ें कि आपके टूल कॉल उचित रूप से मान्य हों।

### -1- आर्किटेक्चर बनाएं

सबसे पहले हमें एक आर्किटेक्चर को संबोधित करने की जरूरत है जो हमें फीचर्स जोड़ने पर स्केल करने में मदद करे, यह इस प्रकार दिखता है:

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

अब हमने एक ऐसी आर्किटेक्चर सेट की है जो सुनिश्चित करती है कि हम आसानी से tools फ़ोल्डर में नए टूल जोड़ सकें। आप संसाधन और प्रॉम्प्ट के लिए उपनिर्देशिका जोड़ने के लिए स्वतंत्र हैं।

### -2- एक टूल बनाना

आइए देखें कि टूल बनाना कैसा दिखता है। सबसे पहले, इसे अपने *tool* उपनिर्देशिका में बनाया जाना चाहिए:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic मॉडल का उपयोग करके इनपुट मान्य करें
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic जोड़ें, ताकि हम एक AddInputModel बना सकें और args को मान्य कर सकें

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

यहाँ हम देखते हैं कि हम नाम, विवरण, इनपुट स्कीमा Pydantic का उपयोग करके और एक हैंडलर को परिभाषित करते हैं जो इस टूल को कॉल किए जाने पर सक्रिय होगा। अंत में, हम `tool_add` एक्सपोज़ करते हैं जो इन सभी प्रॉपर्टीज को रखता है।

यहाँ *schema.py* भी है जो हमारे टूल के लिए इनपुट स्कीमा को परिभाषित करने के लिए उपयोग होता है:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

हमें *__init__.py* को भी भरना होगा ताकि tools डायरेक्टरी को एक मॉड्यूल माना जाए। इसके अलावा, हमें इसमें मौजूद मॉड्यूल्स को एक्सपोज़ भी करना होगा, इस तरह:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

जैसे-जैसे हम और टूल जोड़ते जाएंगे, हम इस फ़ाइल में जोड़ सकते हैं।

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

यहाँ हम प्रॉपर्टीज वाला एक डिक्शनरी बनाते हैं:

- name, यह टूल का नाम है।
- rawSchema, यह Zod स्कीमा है, इसका उपयोग इस टूल को कॉल करने के लिए आने वाले अनुरोधों को मान्य करने के लिए किया जाएगा।
- inputSchema, यह स्कीमा हैंडलर द्वारा उपयोग की जाएगी।
- callback, यह टूल को इनवोक करने के लिए उपयोग किया जाता है।

यहाँ `Tool` भी है जिसका उपयोग इस डिक्शनरी को उस प्रकार में परिवर्तित करने के लिए किया जाता है जिसे mcp सर्वर हैंडलर स्वीकार करता है और यह इस प्रकार दिखता है:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

और *schema.ts* है जहाँ हम प्रत्येक टूल के लिए इनपुट स्कीमा रखते हैं, फिलहाल केवल एक स्कीमा है लेकिन जैसे-जैसे हम टूल जोड़ेंगे, हम और प्रविष्टियाँ जोड़ सकते हैं:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

बहुत अच्छा, अब आइए हमारे टूल्स की सूची को हैंडल करना शुरू करें।

### -3- टूल सूची को हैंडल करें

अब, टूल्स की सूची को हैंडल करने के लिए, हमें उसके लिए एक रिक्वेस्ट हैंडलर सेटअप करना होगा। हमें अपने सर्वर फ़ाइल में यह जोड़ना होगा:

**Python**

```python
# संक्षेप के लिए कोड छोड़ दिया गया है
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

यहाँ हम `@server.list_tools` डेकोरेटर और `handle_list_tools` नामक कार्यान्वयन फ़ंक्शन जोड़ते हैं। बाद में, हमें टूल्स की एक सूची बनानी होगी। ध्यान दें कि प्रत्येक टूल में नाम, विवरण और inputSchema होना आवश्यक है।

**TypeScript**

टूल्स सूची बनाने के लिए रिक्वेस्ट हैंडलर सेट करने हेतु, हमें सर्वर पर `setRequestHandler` कॉल करना होगा एक स्कीमा के साथ जो हमारे प्रयास को फिट करता है, इस मामले में `ListToolsRequestSchema`।

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
// संक्षिप्तता के लिए कोड छोड़ा गया
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // पंजीकृत टूल की सूची लौटाएं
  return {
    tools: tools
  };
});
```

बहुत बढ़िया, अब हमने टूल्स की सूची की समस्या हल कर दी है, आइए देखें कि हम टूल कॉल कैसे कर सकते हैं।

### -4- टूल कॉल को हैंडल करें

टूल कॉल करने के लिए, हमें एक और रिक्वेस्ट हैंडलर सेट करना होगा, जो यह निर्धारित करता है कि किस फीचर को कॉल करना है और किन तर्कों के साथ।

**Python**

आइए `@server.call_tool` डेकोरेटर का उपयोग करें और इसे `handle_call_tool` जैसी फ़ंक्शन के साथ लागू करें। उस फ़ंक्शन में, हमें टूल नाम, इसके तर्क को पार्स करना होगा और सुनिश्चित करना होगा कि तर्क टूल के लिए वैध हैं। हम या तो इस फ़ंक्शन में तर्कों को मान्य कर सकते हैं या असली टूल में।

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools एक शब्दकोश है जिसमें टूल के नाम कुंजियों के रूप में होते हैं
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # टूल को कॉल करें
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

यहाँ यह होता है:

- हमारा टूल नाम इनपुट पैरामीटर `name` के रूप में पहले से मौजूद है जो कि `arguments` डिक्शनरी के रूप में हमारे तर्क हैं।

- टूल को `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` के साथ कॉल किया जाता है। तर्कों का मान्यकरण `handler` प्रॉपर्टी में होता है जो एक फ़ंक्शन को दर्शाता है, यदि वहाँ असफलता होती है तो यह अपवाद उठाएगा।

यहाँ, अब हमारे पास एक पूर्ण समझ है कि लो-लेवल सर्वर का उपयोग करके टूल सूचीबद्ध करना और कॉल करना कैसा होता है।

पूरा उदाहरण यहाँ देखें: [full example](./code/README.md)

## असाइनमेंट

जिस कोड को आपको दिया गया है, उसे कई टूल्स, संसाधनों और प्रॉम्प्ट्स के साथ विस्तार करें और सोचें कि आपको केवल tools डायरेक्टरी में फ़ाइलें जोड़ने की जरूरत होती है, कहीं और नहीं।

*कोई समाधान नहीं दिया गया*

## सारांश

इस अध्याय में, हमने देखा कि लो-लेवल सर्वर दृष्टिकोण कैसे काम करता है और यह कैसे एक अच्छी आर्किटेक्चर बनाने में मदद करता है जिस पर हम आगे निर्माण कर सकते हैं। हमने मान्यकरण पर भी चर्चा की और आपको दिखाया गया कि कैसे मान्यकरण पुस्तकालयों के साथ काम करके इनपुट स्कीमा बनाएं।

## आगे क्या है

- अगला: [सरल प्रमाणीकरण](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:  
यह दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके अनुवादित किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में अधिकारिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सलाह दी जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम जिम्मेदार नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# प्रगत सर्व्हर वापर

MCP SDK मध्ये दोन वेगवेगळ्या प्रकारचे सर्व्हर उपलब्ध आहेत, तुमचा सामान्य सर्व्हर आणि कम-कायम सर्व्हर. सामान्यतः, तुम्ही त्यात वैशिष्ट्ये जोडण्यासाठी सामान्य सर्व्हर वापराल. काही प्रकरणांमध्ये, तुम्हाला कम-कायम सर्व्हरवर विश्वास ठेवायचा असतो जसे की:

- चांगली आर्किटेक्चर. सामान्य सर्व्हर आणि कम-कायम सर्व्हरसह स्वच्छ आर्किटेक्चर तयार करणे शक्य आहे पण हे असा म्हणता येईल की कम-कायम सर्व्हरसह ते थोडे सोपे आहे.
- वैशिष्ट्ये उपलब्धता. काही प्रगत वैशिष्ट्ये फक्त कम-कायम सर्व्हरसह वापरता येतात. पुढील प्रकरणांमध्ये तुम्ही हे पाहाल जसे की आम्ही सॅम्पलिंग आणि एलिसिटेशन जोडतो.

## सामान्य सर्व्हर विरुद्ध कम-कायम सर्व्हर

सामान्य सर्व्हर वापरून MCP सर्व्हर तयार करणे कसे दिसते हे येथे आहे

**Python**

```python
mcp = FastMCP("Demo")

# एक संयोग उपकरण जोडा
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

// एक बेरीज साधन जोडा
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

महत्त्वाचे म्हणजे तुम्ही स्पष्टपणे प्रत्येक टूल, रिसोर्स किंवा प्रॉम्प्ट जो तुम्हाला सर्व्हरमध्ये हवा आहे तो जोडत आहात. यात काही चूक नाही.

### कम-कायम सर्व्हर दृष्टिकोन

परंतु, जेव्हा तुम्ही कम-कायम सर्व्हर दृष्टिकोन वापरता तेव्हा तुम्हाला वेगळ्या प्रकारे विचार करावा लागतो, म्हणजे प्रत्येक टूल नोंदविण्याऐवजी तुम्ही प्रत्येक वैशिष्ट्य प्रकारासाठी (टूल्स, रिसोर्सेस किंवा प्रॉम्प्ट्स) दोन हँडलर्स तयार करता. उदाहरणार्थ टूल्ससाठी फक्त दोन फंक्शन्स असतात:

- सर्व टूल्सची यादी करणे. एक फंक्शन सर्व टूल्सची यादी करण्याच्या सर्व प्रयत्नांसाठी जबाबदार असेल.
- सर्व टूल्स कॉल करण्याचे हाताळणे. येथेही टूल कॉल करण्यासाठी फक्त एक फंक्शन असते.

हे कदाचित कमी काम वाटतंय, नाही का? तर टूल नोंदवण्याऐवजी, मला फक्त खात्री करावी लागते की टूल सर्व टूल्सच्या यादीत आहे आणि जेव्हा टूल कॉल करण्यासाठी विनंती येते तेव्हा तो कॉल होतो.

आता कोड कसा दिसतो ते पाहूया:

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
  // नोंदणी केलेल्या साधनांची यादी परत करा
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

आता आमच्याकडे एक फंक्शन आहे जे वैशिष्ट्यांची यादी परत करते. टूल्स यादीतील प्रत्येक नोंदीत आता `name`, `description` आणि `inputSchema` सारखे फील्ड्स असतात जे परतावा प्रकाराला अनुरूप आहेत. यामुळे आम्ही आमची टूल्स आणि वैशिष्ट्ये परिभाषा दुसऱ्या ठिकाणी ठेवू शकतो. आम्ही आता सर्व टूल्स tools फोल्डरमध्ये तयार करू शकतो आणि तुमच्या सर्व वैशिष्ट्यांसाठीही तसेच करू शकतो, त्यामुळे तुमचा प्रकल्प अचानक याप्रमाणे संघटित होऊ शकतो:

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

हे छान आहे, आमची आर्किटेक्चर प्रामाणिक दिसू शकते.

टूल्स कॉल करण्याबाबत काय, तेच विचार आहे का, एक हँडलर सर्व टूल्स कॉल करण्यासाठी, कोणतेही टूल? हो, अगदी, त्याचा कोड असा आहे:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools हा टूल नावांसह की घेतलेला शब्दकोश आहे
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
    // TODO टूलला कॉल करा,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

वरील कोडमधून तुम्हाला दिसते की, आपल्याला कॉल करायचा टूल आणि कोणत्या अर्ग्युमेंट्ससह याची पार्सिंग करावी लागते, आणि नंतर टूल कॉल करायचे.

## व्हॅलिडेशनसह दृष्टिकोन सुधारणा

आतापर्यंत, तुम्ही पाहिले की तुमचे सर्व टूल्स, रिसोर्सेस आणि प्रॉम्प्ट नोंदणी या दोन हँडलर्सने काढून टाकू शकता. आपण आणखी काय करायला हवे? आपण काही प्रकारचे व्हॅलिडेशन जोडले पाहिजे जे सुनिश्चित करेल की टूल योग्य अर्ग्युमेंट्ससह कॉल होत आहे. प्रत्येक रनटाइमसाठी वेगवेगळे सोल्यूशन्स आहेत, उदाहरणार्थ Python मध्ये Pydantic वापरला जातो आणि TypeScript मध्ये Zod वापरला जातो. संकल्पना अशी आहे की आपण खालील करू:

- वैशिष्ट्य (टूल, रिसोर्स किंवा प्रॉम्प्ट) तयार करण्याची लॉजिक त्याच्या समर्पित फोल्डरमध्ये हलवा.
- येणाऱ्या विनंतीसाठी उदाहरणार्थ टूल कॉल करताना व्हॅलिडेशन करण्याचा मार्ग जोडा.

### वैशिष्ट्य तयार करा

वैशिष्ट्य तयार करण्यासाठी, आपल्याला त्या वैशिष्ट्यासाठी एक फाइल तयार करावी लागेल आणि त्या वैशिष्ट्यासाठी आवश्यक फील्ड्स आहेत याची खात्री करावी लागेल. टूल्स, रिसोर्सेस आणि प्रॉम्प्ट्समध्ये काही फील्ड्स वेगळे असतात.

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
        # Pydantic मॉडेल वापरून इनपुटचे प्रमाणीकरण करा
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic जोडा, जेणेकरून आपण AddInputModel तयार करू शकू आणि args चे प्रमाणीकरण करू शकू

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

येथे तुम्हाला कसे पुढे जाते ते दिसते:

- Pydantic वापरून `AddInputModel` स्कीमा तयार करणे ज्यात `a` आणि `b` फील्ड्स असतात, *schema.py* फाईलमध्ये.
- येणारी विनंती `AddInputModel` प्रकारात पार्स करण्याचा प्रयत्न करणे, जर पॅरामीटर्समध्ये असमतोल असेल तर हा क्रॅश होईल:

   ```python
   # add.py
    try:
        # Pydantic मॉडेल वापरून इनपुटचे प्रमाणीकरण करा
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

तुम्ही हा पार्सिंग लॉजिक टूल कॉलमध्ये किंवा हँडलर फंक्शनमध्ये ठेवू शकता.

**TypeScript**

```typescript
// सर्व्हर.ts
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

// जोड.ts
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

- सर्व टूल कॉल्ससाठी हाताळताना हँडलरमध्ये, आता आम्ही येणाऱ्या विनंतीला टूलच्या परिभाषित स्कीमामध्ये पार्स करण्याचा प्रयत्न करतो:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    जर ते यशस्वी झाले, तर आम्ही प्रत्यक्ष टूल कॉल करतो:

    ```typescript
    const result = await tool.callback(input);
    ```

जास्त स्पष्टपणे, हा दृष्टिकोन एक छान आर्किटेक्चर तयार करतो कारण प्रत्येक गोष्टीस आपले ठिकाण आहे, *server.ts* खूप छोटी फाइल आहे जी फक्त रीक्वेस्ट हँडलर्स वायर्स अप करते आणि प्रत्येक वैशिष्ट्य त्याच्या संबंधित फोल्डर मध्ये आहे उदा. tools/, resources/ किंवा /prompts/.

 छान, चला आता हे तयार करूया.

## व्यायाम: कम-कायम सर्व्हर तयार करणे

या व्यायामात, आम्ही हे करू:

1. टूल्सची यादी प्रदर्शित करणारे आणि टूल्स कॉल करणारे कम-कायम सर्व्हर तयार करा.
2. एक आर्किटेक्चर अंमलात आणा ज्यावर तुम्ही पुढे निर्माण करू शकता.
3. तुमच्या टूल कॉल्स योग्य रीतीने व्हॅलिडेट होत असल्याची खात्री करण्यासाठी व्हॅलिडेशन जोडा.

### -1- आर्किटेक्चर तयार करा

आम्हाला प्रथम एका आर्किटेक्चरची आवश्यकता आहे जी आम्हाला अधिक वैशिष्ट्ये जोडतानाच वाढण्यास मदत करेल, हे कसे दिसते ते खालीलप्रमाणे:

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

आता आम्ही एक अशा प्रकारे आर्किटेक्चर सेटअप केले आहे जेणेकरून तुम्ही tools फोल्डरमध्ये नवीन टूल्स सहजपणे जोडू शकता. रिसोर्सेस आणि प्रॉम्प्ट्ससाठी उपडिरेक्टरी तयार करण्यासाठी तुम्ही हे अनुसरू शकता.

### -2- टूल तयार करणे

आता टूल तयार करणे कसे दिसते ते पाहूया. प्रथम, ते त्याच्या *tool* उपडिरेक्टरीमध्ये तयार करावे लागेल असा:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydantic मॉडेल वापरून इनपुटची पडताळणी करा
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic जोडा, जेणेकरून आपण AddInputModel तयार करू शकू आणि args ची पडताळणी करू शकू

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

येथे आम्ही कसे name, description, Pydantic वापरून input schema परिभाषित करतो आणि एक हँडलर जो या टूलला कॉल करताना इव्होक केला जातो हे पाहिले. शेवटी, `tool_add` नावाचा एक dictionary एक्सपोज केला जातो ज्यात ही सर्व वैशिष्ट्ये आहेत.

*schema.py* फाइल देखील आहे जी आमच्या टूलसाठी वापरल्या जाणार्‍या input schema ची व्याख्या करते:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

आम्हाला *__init__.py* मध्येही भर घालावी लागेल जेणेकरून tools निर्देशिका मॉड्यूल म्हणून वापरली जाईल. शिवाय, त्यामधील मॉड्यूल्स असे एक्सपोज करावेत:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

जसे आम्ही अधिक टूल्स जोडतो तशाप्रकारे आम्ही या फाइलमध्ये वारंवार भर घालू शकतो.

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

येथे आम्ही खालीलप्रमाणे गुणधर्म असलेला dictionary तयार करतो:

- name, हा टूलचा नाव आहे.
- rawSchema, हा Zod schema आहे, येणाऱ्या विनंत्यांचे व्हॅलिडेशन करण्यासाठी वापरला जातो.
- inputSchema, हा हँडलरद्वारे वापरला जाणारा स्कीमा आहे.
- callback, हा टूलला इव्होक करण्यासाठी वापरला जातो.

`Tool` देखील आहे ज्याचा उपयोग हा dictionary mcp सर्व्हर हँडलर स्वीकार करू शकणाऱ्या प्रकारात रूपांतरित करण्यासाठी होतो आणि तो असा दिसतो:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

आणि *schema.ts* जिथे आम्ही प्रत्येक टूलसाठी इनपुट स्कीमाज संग्रहित करतो, सध्या एकाच स्कीमा असून तुम्ही टूल्स वाढविल्यास आणखी एन्ट्रीज जोडू शकता:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

छान, आता टूल्सची यादी हाताळण्याकडे वळूया.

### -3- टूल्सची यादी हाताळा

पुढे, टूल्सची यादी करण्यासाठी विनंती हँडलर सेट करणे आवश्यक आहे. हे सर्व्हर फाइलमध्ये कसे जोडायचे ते खालीलप्रमाणे:

**Python**

```python
# संक्षिप्ततेसाठी कोड वगळला आहे
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

इथे आम्ही डेकोरेटर `@server.list_tools` आणि त्याची अंमलबजावणी करणारे फंक्शन `handle_list_tools` जोडतो. यामध्ये प्रत्येक टूलला नाव, वर्णन आणि inputSchema असणे आवश्यक आहे.

**TypeScript**

टूल्सची यादी करण्यासाठी विनंती हँडलर सेट करण्यासाठी, आम्हाला सर्व्हरवर `setRequestHandler` कॉल करावा लागेल ज्यामध्ये आमच्या हेतूस अनुरूप स्कीमा म्हणजे `ListToolsRequestSchema` असावा.

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
// brevity साठी कोड वगळले
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // नोंदणीकृत टूल्सची यादी परत करा
  return {
    tools: tools
  };
});
```

छान, आता आपण टूल्सची यादी करणे पूर्ण केले, पाहूया पुढे आपण टूल्स कसे कॉल करू शकतो.

### -4- टूल कॉलिंग हाताळा

टूल कॉल करण्यासाठी, आणखी एक विनंती हँडलर सेट करावा लागेल जो कोणता वैशिष्ट्य कॉल करायचा आहे आणि कशा अर्ग्युमेंटसह हे हाताळतो.

**Python**

डेकोरेटर `@server.call_tool` वापरून आणि `handle_call_tool` सारख्या फंक्शनने त्याची अंमलबजावणी करूया. या फंक्शनमध्ये टूलचे नाव, त्याचा अर्ग्युमेंट्स पार्स करणे आणि अर्ग्युमेंट्स त्या टूलसाठी वैध आहेत का ते सुनिश्चित करणे आवश्यक आहे. आम्ही अर्ग्युमेंट्सची पडताळणी या फंक्शनमध्ये करू शकतो किंवा प्रत्यक्ष टूलमध्ये हळूहळू करू शकतो.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools हा टूल नावे की म्हणून असलेला शब्दसंग्रह आहे
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # टूलला कॉल करा
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

इथे काय होते:

- आमचे टूलचे नाव आधीच इनपुट पॅरामीटर `name` मध्ये आहे जे `arguments` dictionary मधील अर्ग्युमेंट्ससाठी खरे आहे.
- टूल कॉल केली जाते `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` अशा प्रकारे. अर्ग्युमेंट्सची पडताळणी `handler` प्रॉपर्टीत होते जी फंक्शनकडे निर्देशित करते, जर ती अयशस्वी झाली तर अपवाद निर्माण होतो.

अशा प्रकारे, आपल्याकडे कम-कायम सर्व्हर वापरून टूल्सची यादी करणे आणि कॉल करणे याचे संपूर्ण समज आहे.

पूर्ण उदाहरण पाहा: [full example](./code/README.md)

## गृहकार्य

तुम्हाला दिलेला कोड अनेक टूल्स, रिसोर्सेस आणि प्रॉम्प्टसह विस्तृत करा आणि तुम्हाला कशी समजते ते प्रतिबिंबित करा की तुम्हाला फक्त tools निर्देशिकेत फाइल्स जोडायच्या आहेत आणि कुठेही इतर ठिकाणी नाही.

*कोणतेही उत्तर दिलेले नाही*

## सारांश

या प्रकरणात, आपण पाहिले की कम-कायम सर्व्हर दृष्टिकोन कसा काम करतो आणि तो आपल्याला कसा एक छान आर्किटेक्चर तयार करण्यात मदत करतो ज्यावर आपण पुढे बांधू शकतो. आपण व्हॅलिडेशनबद्दल चर्चा केली आणि बसल्याने इनपुट व्हॅलिडेशनसाठी स्कीमा तयार करण्यासाठी व्हॅलिडेशन लायब्ररी कशी वापरायची ते दर्शवले.

## पुढे काय आहे

- पुढे: [Simple Authentication](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**सर्व्हनामाः**  
हा दस्तऐवज AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) वापरून अनुवादित केला आहे. आम्ही अचूकतेसाठी प्रयत्नशील असलो तरी, कृपया लक्षात घ्या की स्वयंचलित अनुवादांमध्ये त्रुटी किंवा चुकीची माहिती असू शकते. मूळ दस्तऐवज त्याच्या स्थानिक भाषेत अधिकृत स्रोत समजावयास हवा. महत्त्वाच्या माहितीसाठी व्यावसायिक मानवी अनुवादशिवाय वापर टाळावा. या अनुवादाचा वापर केल्यामुळे उद्भवणाऱ्या कोणत्याही गैरसमजुती किंवा चुकीच्या अर्थलागीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
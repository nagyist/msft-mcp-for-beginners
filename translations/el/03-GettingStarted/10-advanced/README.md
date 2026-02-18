# Προχωρημένη χρήση διακομιστή

Υπάρχουν δύο διαφορετικοί τύποι διακομιστών που εκτίθενται στο MCP SDK, ο κανονικός διακομιστής σας και ο διακομιστής χαμηλού επιπέδου. Κανονικά, θα χρησιμοποιούσατε τον κανονικό διακομιστή για να προσθέσετε λειτουργίες σε αυτόν. Ωστόσο, σε μερικές περιπτώσεις, θέλετε να βασιστείτε στον διακομιστή χαμηλού επιπέδου όπως:

- Καλύτερη αρχιτεκτονική. Είναι δυνατό να δημιουργηθεί μια καθαρή αρχιτεκτονική με τον κανονικό διακομιστή και έναν διακομιστή χαμηλού επιπέδου, αλλά μπορεί να λεχθεί ότι είναι λίγο πιο εύκολο με έναν διακομιστή χαμηλού επιπέδου.
- Διαθεσιμότητα λειτουργιών. Ορισμένες προχωρημένες λειτουργίες μπορούν να χρησιμοποιηθούν μόνο με διακομιστή χαμηλού επιπέδου. Θα το δείτε αυτό σε επόμενα κεφάλαια καθώς προσθέτουμε δειγματοληψία και εκλογή.

## Κανονικός διακομιστής έναντι διακομιστή χαμηλού επιπέδου

Να πώς μοιάζει η δημιουργία ενός MCP Server με τον κανονικό διακομιστή

**Python**

```python
mcp = FastMCP("Demo")

# Προσθέστε ένα εργαλείο πρόσθεσης
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

// Προσθέστε ένα εργαλείο πρόσθεσης
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

Ο σκοπός είναι ότι προσθέτετε ρητά κάθε εργαλείο, πόρο ή προτροπή που θέλετε να έχει ο διακομιστής. Δεν υπάρχει κανένα πρόβλημα με αυτό.

### Προσέγγιση διακομιστή χαμηλού επιπέδου

Ωστόσο, όταν χρησιμοποιείτε την προσέγγιση του διακομιστή χαμηλού επιπέδου, πρέπει να σκεφτείτε διαφορετικά, δηλαδή ότι αντί να καταχωρήσετε κάθε εργαλείο, δημιουργείτε δύο χειριστές ανά τύπο λειτουργίας (εργαλεία, πόροι ή προτροπές). Για παράδειγμα, τα εργαλεία έχουν μόνο δύο λειτουργίες όπως:

- Λίστα όλων των εργαλείων. Μία λειτουργία θα είναι υπεύθυνη για όλες τις προσπάθειες λίστας εργαλείων.
- Ανάκτηση κλήσης όλων των εργαλείων. Εδώ επίσης, υπάρχει μόνο μία λειτουργία που χειρίζεται τις κλήσεις σε ένα εργαλείο.

Ακούγεται σαν λιγότερη δουλειά, σωστά; Αντί να καταχωρήσω ένα εργαλείο, απλώς πρέπει να βεβαιωθώ ότι το εργαλείο είναι στη λίστα όταν κάνω λίστα με όλα τα εργαλεία και ότι καλείται όταν υπάρχει αίτημα για κλήση εργαλείου.

Ας δούμε πώς μοιάζει τώρα ο κώδικας:

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
  // Επιστρέψτε τη λίστα των εγγεγραμμένων εργαλείων
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

Εδώ έχουμε τώρα μια λειτουργία που επιστρέφει μια λίστα λειτουργιών. Κάθε καταχώριση στη λίστα εργαλείων έχει πεδία όπως `name`, `description` και `inputSchema` για να τηρεί τον τύπο επιστροφής. Αυτό μας επιτρέπει να τοποθετήσουμε τα εργαλεία μας και τον ορισμό λειτουργιών αλλού. Τώρα μπορούμε να δημιουργήσουμε όλα μας τα εργαλεία σε έναν φάκελο tools και το ίδιο ισχύει και για όλες τις λειτουργίες σας ώστε το έργο σας να μπορεί ξαφνικά να οργανωθεί ως εξής:

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

Αυτό είναι εξαιρετικό, η αρχιτεκτονική μας μπορεί να είναι αρκετά καθαρή.

Τι γίνεται με την κλήση εργαλείων, είναι η ίδια ιδέα, ένας χειριστής να καλεί οποιοδήποτε εργαλείο; Ναι, ακριβώς, εδώ είναι ο κώδικας για αυτό:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # τα εργαλεία είναι ένα λεξικό με ονόματα εργαλείων ως κλειδιά
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
    // TODO καλέστε το εργαλείο,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Όπως βλέπετε από τον παραπάνω κώδικα, πρέπει να αναλύσουμε ποιο εργαλείο θα κληθεί, και με ποια επιχειρήματα, και μετά προχωράμε στην κλήση του εργαλείου.

## Βελτίωση της προσέγγισης με επικύρωση

Μέχρι τώρα, είδατε πώς όλες οι καταχωρήσεις σας για να προσθέσετε εργαλεία, πόρους και προτροπές μπορούν να αντικατασταθούν με αυτούς τους δύο χειριστές ανά τύπο λειτουργίας. Τι άλλο πρέπει να κάνουμε; Λοιπόν, πρέπει να προσθέσουμε κάποια μορφή επικύρωσης για να διασφαλίσουμε ότι το εργαλείο καλείται με τα σωστά επιχειρήματα. Κάθε πλαίσιο εκτέλεσης έχει τη δική του λύση γι' αυτό, για παράδειγμα, η Python χρησιμοποιεί Pydantic και η TypeScript χρησιμοποιεί Zod. Η ιδέα είναι να κάνουμε τα εξής:

- Να μεταφέρουμε τη λογική δημιουργίας μιας λειτουργίας (εργαλείο, πόρος ή προτροπή) στο αφιερωμένο φάκελό της.
- Να προσθέσουμε έναν τρόπο επικύρωσης ενός εισερχόμενου αιτήματος π.χ. για κλήση ενός εργαλείου.

### Δημιουργία λειτουργίας

Για να δημιουργήσουμε μια λειτουργία, θα χρειαστεί να δημιουργήσουμε ένα αρχείο για αυτήν τη λειτουργία και να βεβαιωθούμε ότι έχει τα υποχρεωτικά πεδία που απαιτούνται για αυτήν. Τα πεδία διαφέρουν λίγο μεταξύ εργαλείων, πόρων και προτροπών.

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
        # Επικύρωση εισόδου χρησιμοποιώντας το μοντέλο Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: πρόσθεσε Pydantic, ώστε να μπορούμε να δημιουργήσουμε ένα AddInputModel και να επικυρώσουμε τα επιχειρήματα

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Εδώ μπορείτε να δείτε πώς κάνουμε τα εξής:

- Δημιουργούμε ένα σχήμα χρησιμοποιώντας το Pydantic `AddInputModel` με πεδία `a` και `b` στο αρχείο *schema.py*.
- Προσπαθούμε να αναλύσουμε το εισερχόμενο αίτημα ώστε να είναι τύπου `AddInputModel`, αν υπάρχει ασυμφωνία στα παραμέτρους, αυτό θα προκαλέσει σφάλμα:

   ```python
   # add.py
    try:
        # Επικυρώστε την είσοδο χρησιμοποιώντας το μοντέλο Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Μπορείτε να επιλέξετε αν θα βάλετε αυτή τη λογική ανάλυσης στην ίδια κλήση του εργαλείου ή στη λειτουργία χειριστή.

**TypeScript**

```typescript
// διακομιστής.ts
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

// σχήμα.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// πρόσθεσε.ts
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

- Στον χειριστή που διαχειρίζεται όλες τις κλήσεις εργαλείων, τώρα προσπαθούμε να αναλύσουμε το εισερχόμενο αίτημα στο ορισμένο σχήμα του εργαλείου:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    αν αυτό πετύχει προχωράμε στην κλήση του ίδιου του εργαλείου:

    ```typescript
    const result = await tool.callback(input);
    ```

Όπως βλέπετε, αυτή η προσέγγιση δημιουργεί μια εξαιρετική αρχιτεκτονική καθώς όλα έχουν τη θέση τους, το *server.ts* είναι ένα πολύ μικρό αρχείο που απλώς συνδέει τους χειριστές αιτήσεων και κάθε λειτουργία είναι στον αντίστοιχο φάκελό της π.χ tools/, resources/ ή /prompts.

Εξαιρετικά, ας προσπαθήσουμε να το κατασκευάσουμε αυτό στη συνέχεια.

## Άσκηση: Δημιουργία διακομιστή χαμηλού επιπέδου

Σε αυτή την άσκηση, θα κάνουμε τα εξής:

1. Δημιουργία διακομιστή χαμηλού επιπέδου που χειρίζεται τη λίστα εργαλείων και κλήση εργαλείων.
1. Υλοποίηση μιας αρχιτεκτονικής πάνω στην οποία μπορείτε να βασιστείτε.
1. Προσθήκη επικύρωσης για να διασφαλίσετε ότι οι κλήσεις εργαλείων σας επικυρώνονται σωστά.

### -1- Δημιουργία αρχιτεκτονικής

Το πρώτο που πρέπει να αντιμετωπίσουμε είναι μια αρχιτεκτονική που μας βοηθά να κλιμακώσουμε καθώς προσθέτουμε περισσότερες λειτουργίες, ιδού πώς μοιάζει:

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

Τώρα έχουμε ρυθμίσει μια αρχιτεκτονική που διασφαλίζει ότι μπορούμε να προσθέτουμε εύκολα νέα εργαλεία σε έναν φάκελο tools. Μη διστάσετε να ακολουθήσετε αυτό και να προσθέσετε υποφακέλους για πόρους και προτροπές.

### -2- Δημιουργία εργαλείου

Ας δούμε πώς φαίνεται το επόμενο βήμα της δημιουργίας εργαλείου. Πρώτον, πρέπει να δημιουργηθεί στον υποφάκελο *tool* όπως:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Επαλήθευση εισόδου χρησιμοποιώντας το μοντέλο Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: πρόσθεσε Pydantic, ώστε να μπορούμε να δημιουργούμε ένα AddInputModel και να επαληθεύουμε τα ορίσματα

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Εδώ βλέπουμε πώς ορίζουμε το όνομα, την περιγραφή, ένα σχήμα εισόδου χρησιμοποιώντας το Pydantic και έναν χειριστή που θα κληθεί όταν αυτό το εργαλείο καλείται. Τέλος, εκθέτουμε το `tool_add` το οποίο είναι ένα λεξικό που περιλαμβάνει όλες αυτές τις ιδιότητες.

Υπάρχει επίσης το *schema.py* που χρησιμοποιείται για να ορίσουμε το σχήμα εισόδου που χρησιμοποιεί το εργαλείο μας:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Πρέπει επίσης να συμπληρώσουμε το *__init__.py* για να βεβαιωθούμε ότι ο φάκελος tools αντιμετωπίζεται ως module. Επιπλέον, πρέπει να εκθέσουμε τα modules μέσα σε αυτόν όπως:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Μπορούμε να συνεχίσουμε να προσθέτουμε σε αυτό το αρχείο καθώς προσθέτουμε περισσότερα εργαλεία.

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

Εδώ δημιουργούμε ένα λεξικό που αποτελείται από ιδιότητες:

- name, αυτό είναι το όνομα του εργαλείου.
- rawSchema, αυτό είναι το σχήμα Zod, το οποίο θα χρησιμοποιηθεί για την επικύρωση εισερχόμενων αιτημάτων για κλήση του εργαλείου.
- inputSchema, αυτό το σχήμα θα χρησιμοποιηθεί από τον χειριστή.
- callback, αυτό χρησιμοποιείται για να εκτελέσει το εργαλείο.

Υπάρχει επίσης το `Tool` που χρησιμοποιείται για να μετατρέψει αυτό το λεξικό σε τύπο που ο χειριστής mcp server μπορεί να αποδεχτεί και έχει την εξής μορφή:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Και υπάρχει το *schema.ts* όπου αποθηκεύουμε τα σχήματα εισόδου για κάθε εργαλείο που μοιάζει ως εξής, προς το παρόν μόνο με ένα σχήμα, αλλά καθώς προσθέτουμε εργαλεία μπορούμε να προσθέσουμε περισσότερες καταχωρήσεις:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Τέλεια, ας συνεχίσουμε τώρα να χειριστούμε τη λίστα των εργαλείων.

### -3- Χειρισμός λίστας εργαλείων

Επόμενο βήμα, για να χειριστούμε τη λίστα εργαλείων, πρέπει να ρυθμίσουμε έναν χειριστή αιτήσεων γι' αυτό. Δείτε τι πρέπει να προσθέσουμε στο αρχείο του διακομιστή:

**Python**

```python
# ο κώδικας παραλείπεται για συντομία
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

Εδώ προσθέτουμε το διακοσμητή `@server.list_tools` και τη λειτουργία `handle_list_tools`. Σε αυτήν τη λειτουργία πρέπει να παράγουμε μια λίστα εργαλείων. Σημειώστε πώς κάθε εργαλείο χρειάζεται να έχει όνομα, περιγραφή και inputSchema.

**TypeScript**

Για να ρυθμίσουμε τον χειριστή αιτήσεων για τη λίστα εργαλείων, πρέπει να καλέσουμε το `setRequestHandler` στον διακομιστή με ένα σχήμα που ταιριάζει με αυτό που προσπαθούμε να κάνουμε, στην προκειμένη περίπτωση το `ListToolsRequestSchema`.

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
// κώδικας παραλειπόμενος για συντομία
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Επιστρέφει τη λίστα των καταχωρημένων εργαλείων
  return {
    tools: tools
  };
});
```

Τέλεια, τώρα έχουμε λύσει το κομμάτι της λίστας εργαλείων, ας δούμε πώς μπορούμε στη συνέχεια να καλούμε εργαλεία.

### -4- Χειρισμός κλήσης εργαλείου

Για να καλέσουμε ένα εργαλείο, πρέπει να ρυθμίσουμε έναν άλλο χειριστή αιτήσεων, αυτή τη φορά με στόχο να χειρίζεται ένα αίτημα που καθορίζει ποια λειτουργία να καλέσει και με ποια επιχειρήματα.

**Python**

Ας χρησιμοποιήσουμε το διακοσμητή `@server.call_tool` και να τον υλοποιήσουμε με μια λειτουργία όπως `handle_call_tool`. Μέσα σε αυτή τη λειτουργία, πρέπει να αναλύσουμε το όνομα του εργαλείου, τα επιχειρήματά του και να διασφαλίσουμε ότι τα επιχειρήματα είναι έγκυρα για το εκάστοτε εργαλείο. Μπορούμε είτε να επικυρώσουμε τα επιχειρήματα εδώ είτε στην ίδια τη λειτουργία του εργαλείου.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # το tools είναι ένα λεξικό με ονόματα εργαλείων ως κλειδιά
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # εκτέλεσε το εργαλείο
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Αυτό συμβαίνει:

- Το όνομα του εργαλείου είναι ήδη παρόν ως η παράμετρος εισόδου `name`, το ίδιο ισχύει για τα επιχειρήματά μας υπό μορφή λεξικού `arguments`.

- Το εργαλείο καλείται με `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Η επικύρωση των επιχειρημάτων γίνεται στην ιδιότητα `handler` που δείχνει σε μια λειτουργία, αν αποτύχει θα ρίξει εξαίρεση.

Έτσι, τώρα έχουμε πλήρη κατανόηση της λίστας και της κλήσης εργαλείων χρησιμοποιώντας διακομιστή χαμηλού επιπέδου.

Δείτε το [πλήρες παράδειγμα](./code/README.md) εδώ

## Ανάθεση εργασίας

Επέκτεινε τον κώδικα που σου έχει δοθεί με πολλά εργαλεία, πόρους και προτροπές και αναλογίσου πώς παρατηρείς ότι χρειάζεται να προσθέτεις αρχεία μόνο στον φάκελο tools και πουθενά αλλού.

*Δεν παρέχεται λύση*

## Περίληψη

Σε αυτό το κεφάλαιο, είδαμε πώς λειτουργεί η προσέγγιση του διακομιστή χαμηλού επιπέδου και πώς αυτή μπορεί να μας βοηθήσει να δημιουργήσουμε μια καλή αρχιτεκτονική που μπορούμε να συνεχίσουμε να χτίζουμε. Συζητήσαμε επίσης την επικύρωση και σας δείξαμε πώς να δουλέψετε με βιβλιοθήκες επικύρωσης για να δημιουργήσετε σχήματα για την επικύρωση εισόδου.

## Τι ακολουθεί

- Επόμενο: [Απλή αυθεντικοποίηση](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:  
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία αυτόματης μετάφρασης AI [Co-op Translator](https://github.com/Azure/co-op-translator). Παρότι καταβάλλουμε προσπάθεια για ακρίβεια, παρακαλούμε να λάβετε υπόψη ότι οι αυτόματες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη γλώσσα του αποτελεί την επίσημη και αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρανοήσεις ή εσφαλμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
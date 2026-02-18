# שימוש מתקדם בשרת

ישנם שני סוגים שונים של שרתים המוצגים ב-MCP SDK, השרת הרגיל שלך והשרת ברמה נמוכה. בדרך כלל, תשתמש בשרת הרגיל כדי להוסיף לו תכונות. עם זאת, במקרים מסוימים תרצה להסתמך על השרת ברמה נמוכה כמו:

- ארכיטקטורה טובה יותר. אפשר ליצור ארכיטקטורה נקייה עם שני השרתים, הרגיל והנמוך, אך ניתן לטעון שכלי השרת ברמה נמוכה מעט יותר נוח.
- זמינות תכונות. חלק מהתכונות המתקדמות ניתן להשתמש רק עם השרת ברמה נמוכה. תראה זאת בפרקים הבאים כאשר נוסיף דגימה ו-\<elicitation\>אילוסון\</elicitation\>.

## שרת רגיל לעומת שרת ברמה נמוכה

כך נראה יצירת שרת MCP עם השרת הרגיל

**Python**

```python
mcp = FastMCP("Demo")

# הוסף כלי חיבור
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

// הוסף כלי חיבור
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

הכוונה היא שאתה מוסיף במפורש כל כלי, משאב או בקשה שברצונך שהשרת יכיל. אין בזה שום דבר לא בסדר.  

### גישת השרת ברמה נמוכה

עם זאת, כאשר אתה משתמש בגישת השרת ברמה נמוכה עליך לחשוב שונה, כלומר במקום לרשום כל כלי בנפרד, אתה יוצר שני מטפלים לכל סוג תכונה (כלים, משאבים או בקשות). לדוגמה, לכלים יהיו רק שתי פונקציות כאלה:

- רשימת כל הכלים. פונקציה אחת תהיה אחראית על כל הניסיונות לרשום כלים.
- טיפול בקריאה לכל הכלים. כאן גם, יש רק פונקציה אחת המטפלת בקריאות לכלי.

נשמע שזה עשוי לחסוך עבודה, נכון? אז במקום לרשום כלי, אני רק צריך לוודא שהכלי מופיע ברשימה כשאני מציג את כל הכלים ושהוא נקרא כשרואים בקשה שתתקשר לכלי. 

בוא נבחן כיצד הקוד נראה עכשיו:

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
  // החזר את רשימת הכלים הרשומים
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

כאן יש לנו פונקציה שמחזירה רשימת תכונות. כל רשומה ברשימת הכלים כוללת כעת שדות כמו `name`, `description` ו-`inputSchema` כדי לעמוד בסוג ההחזרה. זה מאפשר לנו למקם את הכלים והגדרת התכונות במקום אחר. אנחנו יכולים כעת ליצור את כל הכלים בתיקיית כלים וכן לגבי כל התכונות כך שהפרויקט שלך יכול להיות מאורגן כך פתאום:

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

זה נהדר, הארכיטקטורה שלנו יכולה להיראות נקייה מאוד.

ומה לגבי קריאת כלים, האם זה אותו רעיון? מטפל אחד לקריאת כלי, כל כלי שתרצה? כן, בדיוק, הנה הקוד לכך:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # כלי הוא מילון עם שמות כלים כמפתחות
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
    
    // ארגומנטים: request.params.arguments
    // TODO לקרוא לכלי,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

כפי שנראה מהקוד למעלה, אנחנו צריכים לפרש איזה כלי לקרוא ובאיזה פרמטרים, ואז להמשיך לקרוא לכלי.

## שיפור הגישה עם אימות

עד כה, ראית כיצד כל הרישומים להוספת כלים, משאבים ופרומפטים יכולים להיות מוחלפים עם שני המטפלים האלה לכל סוג תכונה. מה עוד צריך לעשות? טוב, כדאי להוסיף איזשהו סוג של אימות כדי לוודא שהקריאה לכלי נעשית עם הפרמטרים הנכונים. בכל זמן ריצה יש פתרון משלו לכך, לדוגמה ב-Python משתמשים ב-Pydantic וב-TypeScript ב-Zod. הרעיון הוא שנעשה את הדברים הבאים:

- להעביר את הלוגיקה ליצירת תכונה (כלי, משאב או פרומפט) לתיקייה ייעודית שלה.
- להוסיף דרך לאמת בקשה שמגיעה, למשל קריאה לכלי.

### יצירת תכונה

על מנת ליצור תכונה, נצטרך ליצור קובץ עבור התכונה הזו ולוודא שיש בו את השדות ההכרחיים שהתכונה דורשת. אילו שדות משתנים מעט בין כלים, משאבים ופרומפטים.

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
        # אימות קלט באמצעות מודל Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: להוסיף Pydantic, כדי שנוכל ליצור AddInputModel ולאמת ארגומנטים

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

כאן ניתן לראות כיצד אנו עושים את הדברים הבאים:

- יוצרים סכימה בעזרת Pydantic בשם `AddInputModel` עם השדות `a` ו-`b` בקובץ *schema.py*.
- מנסים לפרש את הבקשה המתקבלת כסוג `AddInputModel`, אם יש חוסר התאמה בפרמטרים זה יתקל בשגיאה:

   ```python
   # add.py
    try:
        # אימות קלט באמצעות מודל Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

אתה יכול לבחור לאן למקם את לוגיקת הפרשנות - בקריאה לכלי עצמו או בפונקציית המטפל.

**TypeScript**

```typescript
// שרת.ts
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

       // @ts-התעלם
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

// סכימה.ts
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });

// הוסף.ts
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

- במטפל שמתעסק עם כל קריאות הכלים, אנו כעת מנסים לפרש את הבקשה המגיעה לפי הסכימה שהוגדרה לכלי:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    אם זה מצליח, אנו ממשיכים לקרוא לכלי בפועל:

    ```typescript
    const result = await tool.callback(input);
    ```

כפי שניתן לראות, גישה זו יוצרת ארכיטקטורה מצוינת כי לכל דבר יש את מקומו, *server.ts* הוא קובץ קטן שמקשר רק את מטפלי הבקשות ולכל תכונה יש תיקייה משלה כמו tools/, resources/ או /prompts.

מצוין, בוא ננסה לבנות את זה בהמשך. 

## תרגיל: יצירת שרת ברמה נמוכה

בתרגיל זה נבצע את הדברים הבאים:

1. ליצור שרת ברמה נמוכה שמטפל ברשימת כלים ובקריאת כלים.
1. ליישם ארכיטקטורה שניתן להרחיב.
1. להוסיף אימות כדי לוודא שהקריאות שלך לכלים מאומתות כראוי.

### -1- יצירת ארכיטקטורה

הדבר הראשון שצריך לטפל בו הוא ארכיטקטורה שתעזור לנו להגדיל כאשר נוסיף עוד תכונות, כך היא נראית:

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

כעת הגדרנו ארכיטקטורה שמבטיחה שניתן להוסיף כלים בקלות בתיקיית tools. הרגש חופשי לעקוב אחר זה גם להוספת תיקיות משנה למשאבים ופרומפטים.

### -2- יצירת כלי

נסקור כיצד נראה יצירת כלי כעת. קודם כל, הוא צריך להיווצר בתיקייה המשנה *tool* כך:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # לאמת קלט באמצעות מודל Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: להוסיף Pydantic, כדי שנוכל ליצור AddInputModel ולאמת פרמטרים

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

מה שרואים כאן הוא כיצד אנו מגדירים שם, תיאור, סכימת קלט בעזרת Pydantic ומטפל שיפעיל כשיקראו לכלי. לבסוף, אנו חושפים את `tool_add` שהוא מילון המכיל את כל התכונות האלו.

קיים גם *schema.py* בו מוגדרת סכימת הקלט שהכלי משתמש בו:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

כמו כן יש למלא את *__init__.py* כדי לוודא שתיקיית הכלים תטופל כמודול. בנוסף, נחשוף את המודולים שבתוכה כך:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

אפשר להמשיך להוסיף לקובץ זה ככל שנוסיף כלים.

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

כאן אנו יוצרים מילון המורכב מתכונות:

- name, הוא שם הכלי.
- rawSchema, זוהי סכימת Zod, תשמש כדי לאמת בקשות שקוראות לכלי.
- inputSchema, סכימה זו תשמש את המטפל.
- callback, משמש לע invokes את הכלי.

קיים גם `Tool` שמשמש להמרת מילון זה לסוג שניתן לקבל ע"י מטפל השרת של MCP והוא נראה כך:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

וקיים *schema.ts* בו מאוחסנות סכימות הקלט לכל כלי כך עם סכימה אחת בלבד לעת עתה אך ככל שנוסיף כלים נוסיף עוד כניסות:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

מצוין, נמשיך לטפל ברשימת הכלים שלנו כעת.

### -3- טיפול ברשימת הכלים

כדי לטפל ברשימת הכלים, עלינו להגדיר מטפל בקשה לכך. כך נוסיף זאת לקובץ השרת שלנו:

**Python**

```python
# הקוד הושמט לקבלת תמצית
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

כאן אנו מוסיפים את ה-דקורטור `@server.list_tools` ואת הפונקציה המיישמת `handle_list_tools`. בפונקציה הזו אנו צריכים להפיק רשימה של כל הכלים. שים לב שכל כלי צריך לכלול שם, תיאור ו-inputSchema.  

**TypeScript**

כדי להגדיר את מטפל הבקשה לרשימת כלים, עלינו לקרוא ל-`setRequestHandler` על השרת עם סכימה המתאימה למה שאנו מנסים לעשות, במקרה זה `ListToolsRequestSchema`.

```typescript
// אינדקס.ts
import addTool from "./add.js";
import subtractTool from "./subtract.js";
import {server} from "../server.js";
import { Tool } from "./tool.js";

export let tools: Array<Tool> = [];
tools.push(addTool);
tools.push(subtractTool);

// שרת.ts
// הקוד הושמט בקיצור
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // החזר את רשימת הכלים הרשומים
  return {
    tools: tools
  };
});
```

מעולה, עכשיו פתרנו את נושא רשימת הכלים, בוא נבחן כיצד ניתן לקרוא לכלים.

### -4- טיפול בקריאה לכלי

כדי לקרוא לכלי, עלינו להגדיר מטפל בקשות נוסף, הפעם שמתמקד בהתמודדות עם בקשה שמציינת איזו תכונה לקרוא ובאיזה פרמטרים.

**Python**

נשתמש בדקורטור `@server.call_tool` ונממש אותו בפונקציה כמו `handle_call_tool`. בתוך פונקציה זו עלינו לפרש את שם הכלי, הפרמטרים ולוודא שהפרמטרים תקינים עבור הכלי הנבחר. ניתן לאמת את הפרמטרים בפונקציה זו או במורד הזרם בתוך הכלי עצמו.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools הוא מילון עם שמות כלים כמפתחות
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # להפעיל את הכלי
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

הנה מה שקורה:

- שם הכלי שלנו כבר קיים כפרמטר הקלט `name` שהוא נכון לגבי הפרמטרים שלנו במילון `arguments`.

- הכלי נקרא עם `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. אימות הפרמטרים מתבצע בתכונת `handler` שמצביעה על פונקציה, אם זה נכשל תושלך חריגה.

כעת יש לנו הבנה מלאה של הרשימה והקריאה לכלים באמצעות שרת ברמה נמוכה.

ראה את ה[דוגמה המלאה](./code/README.md) כאן

## מטלה

הרחב את הקוד שניתן לך עם מספר כלים, משאבים ופרומפטים והרהר כיצד אתה מבחין בכך שאתה צריך להוסיף קבצים רק בתיקיית tools ולא במקום אחר.

*לא ניתנה פתרון*

## סיכום

בפרק זה ראינו כיצד עובדת גישת השרת ברמה נמוכה וכיצד היא יכולה לעזור לנו ליצור ארכיטקטורה טובה שנוכל להמשיך לבנות עליה. דיברנו גם על אימות והוצג בפניך כיצד לעבוד עם ספריות אימות ליצירת סכימות לאימות קלט.

## מה הלאה

- הבא: [אימות פשוט](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:  
מסמך זה תורגם באמצעות שירות תרגום מבוסס בינה מלאכותית [Co-op Translator](https://github.com/Azure/co-op-translator). על אף שאנו שואפים לדיוק, יש לקחת בחשבון כי תרגומים אוטומטיים עלולים להכיל שגיאות או אי דיוקים. יש להתייחס למסמך המקורי בשפתו המקורית כמקור הסמכות. עבור מידע קריטי, מומלץ לבצע תרגום מקצועי על ידי אדם. אנו לא נושאים באחריות לכל הבנה שגויה או פרשנות מוטעית הנובעות משימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
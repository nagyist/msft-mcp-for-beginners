# Penggunaan pelayan lanjutan

Terdapat dua jenis pelayan yang didedahkan dalam MCP SDK, pelayan biasa anda dan pelayan aras rendah. Biasanya, anda akan menggunakan pelayan biasa untuk menambah ciri kepadanya. Walau bagaimanapun untuk beberapa kes, anda mahu bergantung pada pelayan aras rendah seperti:

- Seni bina yang lebih baik. Adalah mungkin untuk mencipta seni bina yang kemas dengan kedua-dua pelayan biasa dan pelayan aras rendah tetapi boleh dikatakan ia sedikit lebih mudah dengan pelayan aras rendah.
- Ketersediaan ciri. Sesetengah ciri lanjutan hanya boleh digunakan dengan pelayan aras rendah. Anda akan melihat ini dalam bab-bab kemudian apabila kami menambah pensampelan dan elicitation.

## Pelayan biasa vs pelayan aras rendah

Ini adalah bagaimana penciptaan MCP Server kelihatan dengan pelayan biasa

**Python**

```python
mcp = FastMCP("Demo")

# Tambah alat penambahan
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

// Tambah alat tambah
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

Maksudnya ialah anda secara jelas menambah setiap alat, sumber atau prompt yang anda mahu pelayan itu miliki. Tiada apa yang salah dengan itu.  

### Pendekatan pelayan aras rendah

Walau bagaimanapun, apabila anda menggunakan pendekatan pelayan aras rendah anda perlu berfikir dengan cara yang berbeza iaitu bahawa sebagai ganti mendaftar setiap alat, anda sebaliknya mencipta dua pengendali bagi setiap jenis ciri (alat, sumber atau prompt). Jadi contohnya alat hanya mempunyai dua fungsi seperti berikut:

- Menyenaraikan semua alat. Satu fungsi yang bertanggungjawab untuk semua usaha menyenaraikan alat.
- Mengendalikan panggilan semua alat. Di sini juga, hanya ada satu fungsi yang mengendalikan panggilan ke alat

Kedengaran seperti kurang kerja kan? Jadi sebagai ganti mendaftarkan sebuah alat, saya hanya perlu memastikan alat disenaraikan apabila saya menyenaraikan semua alat dan ia dipanggil apabila terdapat permintaan masuk untuk memanggil alat itu. 

Mari kita lihat bagaimana kod sekarang kelihatan:

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
  // Pulangkan senarai alat yang didaftarkan
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

Di sini kita sekarang mempunyai fungsi yang mengembalikan senarai ciri. Setiap entri dalam senarai alat kini mempunyai medan seperti `name`, `description` dan `inputSchema` untuk mematuhi jenis pengembalian. Ini membolehkan kita meletakkan alat dan definisi ciri kita di tempat lain. Kita kini boleh mencipta semua alat kita dalam folder tools dan perkara yang sama untuk semua ciri anda supaya projek anda boleh tiba-tiba disusun seperti berikut:

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

Itu hebat, seni bina kita boleh dibuat kelihatan cukup kemas.

Bagaimana pula dengan memanggil alat, adakah ia idea yang sama, satu pengendali untuk memanggil alat, mana-mana alat? Ya, tepat sekali, ini adalah kod untuk itu:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools adalah kamus dengan nama alat sebagai kunci
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
    // TODO panggil alat,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Seperti yang anda boleh lihat daripada kod di atas, kita perlu menguraikan alat untuk dipanggil, dan dengan argumen apa, dan kemudian kita perlu meneruskan untuk memanggil alat tersebut.

## Memperbaiki pendekatan dengan pengesahan

Setakat ini, anda telah melihat bagaimana semua pendaftaran anda untuk menambah alat, sumber dan prompt boleh digantikan dengan dua pengendali ini bagi setiap jenis ciri. Apa lagi yang perlu kita lakukan? Baik, kita harus menambah beberapa bentuk pengesahan untuk memastikan alat dipanggil dengan argumen yang betul. Setiap runtime mempunyai penyelesaian mereka sendiri untuk ini, contohnya Python menggunakan Pydantic dan TypeScript menggunakan Zod. Idenya adalah bahawa kita melakukan yang berikut:

- Memindahkan logik untuk mencipta ciri (alat, sumber atau prompt) ke folder khususnya.
- Menambah cara untuk mengesahkan permintaan masuk yang meminta contohnya untuk memanggil alat.

### Mencipta ciri

Untuk mencipta ciri, kita perlu membuat fail bagi ciri itu dan memastikan ia mempunyai medan wajib yang dikehendaki oleh ciri itu. Medan mana berbeza sedikit antara alat, sumber dan prompt.

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
        # Sahkan input menggunakan model Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: tambah Pydantic, supaya kita boleh buat AddInputModel dan sahkan args

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

di sini anda boleh lihat bagaimana kita melakukan yang berikut:

- Mencipta skema menggunakan Pydantic `AddInputModel` dengan medan `a` dan `b` dalam fail *schema.py*.
- Cuba menguraikan permintaan masuk menjadi jenis `AddInputModel`, jika terdapat ketidakpadanan parameter ini akan menyebabkan kerosakan:

   ```python
   # add.py
    try:
        # Sahkan input menggunakan model Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Anda boleh memilih sama ada meletakkan logik penguraian ini dalam panggilan alat itu sendiri atau dalam fungsi pengendali.

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

- Dalam pengendali yang mengendalikan semua panggilan alat, kita sekarang cuba menguraikan permintaan masuk ke dalam skema yang ditakrifkan oleh alat:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    jika itu berjaya maka kita teruskan untuk memanggil alat sebenar:

    ```typescript
    const result = await tool.callback(input);
    ```

Seperti yang anda lihat, pendekatan ini mewujudkan seni bina yang sangat baik kerana segala-galanya mempunyai tempatnya, *server.ts* adalah fail yang sangat kecil yang hanya menyambungkan pengendali permintaan dan setiap ciri berada dalam folder masing-masing iaitu tools/, resources/ atau /prompts.

Hebat, mari cuba bina ini seterusnya. 

## Latihan: Mencipta pelayan aras rendah

Dalam latihan ini, kita akan melakukan yang berikut:

1. Mencipta pelayan aras rendah yang mengendalikan penyenaraian alat dan pemanggilan alat.
1. Melaksanakan seni bina yang anda boleh bina dengannya.
1. Menambah pengesahan untuk memastikan panggilan alat anda disahkan dengan betul.

### -1- Mencipta seni bina

Perkara pertama yang perlu kita tangani ialah seni bina yang membantu kita berkembang apabila kita menambah lebih banyak ciri, berikut adalah bagaimana ia kelihatan:

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

Sekarang kita telah menyiapkan seni bina yang memastikan kita boleh dengan mudah menambah alat baru dalam folder tools. Sila ikut ini untuk menambah subdirektori untuk resources dan prompts.

### -2- Mencipta alat

Mari kita lihat bagaimana mencipta alat kelihatan pula. Pertama, ia perlu dicipta dalam subdirektori *tool* seperti berikut:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Sahkan input menggunakan model Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: tambah Pydantic, supaya kita boleh mencipta AddInputModel dan sahkan args

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Apa yang kita lihat di sini ialah bagaimana kita mentakrifkan nama, perihalan, skema input menggunakan Pydantic dan pengendali yang akan dipanggil sekali alat ini dipanggil. Akhir sekali, kita dedahkan `tool_add` yang merupakan kamus yang memegang semua sifat ini.

Terdapat juga *schema.py* yang digunakan untuk mentakrifkan skema input yang digunakan oleh alat kita:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Kita juga perlu mengisi *__init__.py* untuk memastikan direktori tools dianggap sebagai modul. Selain itu, kita perlu dedahkan modul di dalamnya seperti berikut:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Kita boleh terus menambah ke fail ini apabila kita menambah lebih banyak alat.

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

Di sini kita mencipta kamus yang terdiri daripada sifat-sifat:

- name, ini adalah nama alat.
- rawSchema, ini adalah skema Zod, ia akan digunakan untuk mengesahkan permintaan masuk untuk memanggil alat ini.
- inputSchema, skema ini akan digunakan oleh pengendali.
- callback, ini digunakan untuk memanggil alat.

Terdapat juga `Tool` yang digunakan untuk menukar kamus ini ke dalam jenis yang boleh diterima oleh pengendali pelayan mcp dan ia kelihatan seperti berikut:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Dan terdapat *schema.ts* di mana kita menyimpan skema input untuk setiap alat yang kelihatan seperti berikut dengan hanya satu skema pada masa ini tetapi apabila kita menambah alat kita boleh menambah lebih banyak entri:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Hebat, mari teruskan untuk mengendalikan penyenaraian alat kita seterusnya.

### -3- Mengendalikan penyenaraian alat

Seterusnya, untuk mengendalikan penyenaraian alat kita, kita perlu menyiapkan pengendali permintaan untuk itu. Berikut adalah apa yang perlu kita tambah ke fail pelayan kita:

**Python**

```python
# kod disunting untuk ringkasan
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

Di sini kita menambah pemangkai `@server.list_tools` dan fungsi pelaksanaan `handle_list_tools`. Dalam fungsi terakhir, kita perlu menghasilkan senarai alat. Perhatikan bahawa setiap alat perlu mempunyai nama, perihalan dan inputSchema.   

**TypeScript**

Untuk menyediakan pengendali permintaan bagi menyenaraikan alat, kita perlu memanggil `setRequestHandler` pada pelayan dengan skema yang sesuai dengan apa yang kita cuba lakukan, dalam kes ini `ListToolsRequestSchema`. 

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
// kod dipendekkan untuk ringkasan
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Kembalikan senarai alat berdaftar
  return {
    tools: tools
  };
});
```

Hebat, kini kita telah menyelesaikan bahagian penyenaraian alat, mari kita lihat bagaimana kita boleh memanggil alat seterusnya.

### -4- Mengendalikan pemanggilan alat

Untuk memanggil alat, kita perlu menyiapkan satu lagi pengendali permintaan, kali ini memfokuskan pada mengendalikan permintaan yang menentukan ciri mana yang hendak dipanggil dan dengan argumen apa.

**Python**

Mari gunakan pemangkai `@server.call_tool` dan melaksanakannya dengan fungsi seperti `handle_call_tool`. Dalam fungsi itu, kita perlu menguraikan nama alat, argumennya dan memastikan argumen adalah sah untuk alat yang dimaksudkan. Kita boleh sama ada mengesahkan argumen dalam fungsi ini atau di hiliran dalam alat sebenar.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools adalah kamus dengan nama alat sebagai kunci
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # panggil alat tersebut
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Ini yang berlaku:

- Nama alat kita sudah hadir sebagai parameter input `name` yang benar bagi argumen kita dalam bentuk kamus `arguments`.

- Alat ini dipanggil dengan `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Pengesahan argumen berlaku di dalam sifat `handler` yang merujuk kepada fungsi, jika itu gagal ia akan menimbulkan pengecualian. 

Itu sahaja, sekarang kita sudah memahami sepenuhnya tentang penyenaraian dan pemanggilan alat menggunakan pelayan aras rendah.

Lihat [contoh penuh](./code/README.md) di sini

## Tugasan

Perluaskan kod yang telah diberikan dengan beberapa alat, sumber dan prompt dan fikirkan bagaimana anda perasan bahawa anda hanya perlu menambah fail dalam direktori tools dan tiada tempat lain. 

*Tiada penyelesaian diberikan*

## Ringkasan

Dalam bab ini, kita melihat bagaimana pendekatan pelayan aras rendah berfungsi dan bagaimana ia boleh membantu kita mencipta seni bina yang kemas yang boleh kita terus bina. Kita juga membincangkan pengesahan dan anda telah ditunjukkan bagaimana bekerja dengan perpustakaan pengesahan untuk mencipta skema bagi pengesahan input.

## Apa Seterusnya

- Seterusnya: [Pengesahan Mudah](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:  
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan profesional oleh manusia adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
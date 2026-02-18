# Penggunaan server tingkat lanjut

Ada dua jenis server yang tersedia dalam MCP SDK, server biasa dan server tingkat rendah. Biasanya, Anda akan menggunakan server biasa untuk menambahkan fitur ke dalamnya. Namun dalam beberapa kasus, Anda ingin mengandalkan server tingkat rendah seperti:

- Arsitektur yang lebih baik. Memungkinkan untuk membuat arsitektur yang bersih dengan server biasa dan server tingkat rendah, tetapi bisa dikatakan sedikit lebih mudah dengan server tingkat rendah.
- Ketersediaan fitur. Beberapa fitur lanjutan hanya dapat digunakan dengan server tingkat rendah. Anda akan melihat ini di bab-bab berikutnya saat kami menambahkan sampling dan elicitation.

## Server biasa vs server tingkat rendah

Berikut adalah contoh pembuatan MCP Server dengan server biasa

**Python**

```python
mcp = FastMCP("Demo")

# Tambahkan alat penjumlahan
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

// Tambahkan alat penjumlahan
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

Intinya adalah bahwa Anda secara eksplisit menambahkan setiap alat, sumber daya atau prompt yang Anda inginkan pada server. Tidak ada yang salah dengan itu.

### Pendekatan server tingkat rendah

Namun, ketika menggunakan pendekatan server tingkat rendah Anda perlu berpikir berbeda, yaitu alih-alih mendaftarkan setiap alat, Anda justru membuat dua handler per tipe fitur (alat, sumber daya, atau prompt). Jadi, misalnya alat hanya memiliki dua fungsi seperti berikut:

- Mendaftar semua alat. Satu fungsi bertanggung jawab untuk semua upaya mendaftar alat.
- Menangani pemanggilan semua alat. Di sini juga, hanya ada satu fungsi yang menangani pemanggilan suatu alat.

Kedengarannya seperti pekerjaan yang lebih sedikit bukan? Jadi alih-alih mendaftarkan alat, saya hanya perlu memastikan alat tersebut tercantum saat saya mendaftar semua alat dan bahwa alat tersebut dipanggil saat ada permintaan masuk untuk memanggil alat tersebut.

Mari kita lihat seperti apa kode tersebut sekarang:

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
  // Kembalikan daftar alat yang terdaftar
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

Di sini kita sekarang memiliki fungsi yang mengembalikan daftar fitur. Setiap entri dalam daftar alat sekarang memiliki field seperti `name`, `description` dan `inputSchema` untuk mematuhi tipe pengembalian. Ini memungkinkan kita menempatkan definisi alat dan fitur kita di tempat lain. Kita sekarang dapat membuat semua alat kita dalam folder tools dan hal yang sama berlaku untuk semua fitur Anda sehingga proyek Anda bisa diatur seperti ini:

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

Bagus, arsitektur kita bisa dibuat terlihat cukup bersih.

Bagaimana dengan pemanggilan alat, apakah idenya sama, satu handler untuk memanggil alat, alat mana pun? Ya, tepat sekali, berikut adalah kodenya:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools adalah sebuah kamus dengan nama alat sebagai kunci
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

Seperti yang Anda lihat dari kode di atas, kita perlu memparsing alat yang akan dipanggil, dan dengan argumen apa, kemudian kita melanjutkan untuk memanggil alat tersebut.

## Meningkatkan pendekatan dengan validasi

Sejauh ini, Anda telah melihat bagaimana semua pendaftaran untuk menambahkan alat, sumber daya dan prompt dapat digantikan dengan dua handler per tipe fitur ini. Apa lagi yang perlu kita lakukan? Nah, kita harus menambahkan semacam validasi untuk memastikan bahwa alat dipanggil dengan argumen yang benar. Setiap runtime memiliki solusinya sendiri, misalnya Python menggunakan Pydantic dan TypeScript menggunakan Zod. Idemnya adalah kita melakukan hal berikut:

- Memindahkan logika untuk membuat fitur (alat, sumber daya atau prompt) ke folder khususnya.
- Menambahkan cara untuk memvalidasi permintaan masuk yang misalnya meminta memanggil alat.

### Membuat fitur

Untuk membuat fitur, kita perlu membuat file untuk fitur tersebut dan memastikan file itu memiliki field wajib yang dibutuhkan oleh fitur tersebut. Field mana yang dibutuhkan sedikit berbeda antara alat, sumber daya, dan prompt.

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
        # Validasi input menggunakan model Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: tambahkan Pydantic, sehingga kita dapat membuat AddInputModel dan memvalidasi argumen

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Di sini Anda dapat melihat bagaimana kita melakukan hal berikut:

- Membuat schema menggunakan Pydantic `AddInputModel` dengan field `a` dan `b` di file *schema.py*.
- Mencoba memparsing permintaan masuk agar menjadi tipe `AddInputModel`, jika ada ketidaksesuaian parameter ini akan crash:

   ```python
   # add.py
    try:
        # Validasi input menggunakan model Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Anda dapat memilih apakah meletakkan logika parsing ini di pemanggilan alat itu sendiri atau pada fungsi handler.

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

// tambah.ts
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

- Dalam handler yang menangani semua pemanggilan alat, kita sekarang mencoba memparsing permintaan masuk ke dalam schema yang didefinisikan oleh alat:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    jika itu berhasil, kita melanjutkan untuk memanggil alat sebenarnya:

    ```typescript
    const result = await tool.callback(input);
    ```

Seperti yang Anda lihat, pendekatan ini menciptakan arsitektur yang hebat karena semuanya memiliki tempatnya, *server.ts* adalah file yang sangat kecil yang hanya menghubungkan request handler dan setiap fitur ada di folder masing-masing yaitu tools/, resources/ atau /prompts.

Bagus, mari kita coba bangun ini selanjutnya.

## Latihan: Membuat server tingkat rendah

Dalam latihan ini, kita akan melakukan hal berikut:

1. Membuat server tingkat rendah yang menangani listing alat dan pemanggilan alat.
1. Mengimplementasikan arsitektur yang dapat Anda bangun.
1. Menambahkan validasi untuk memastikan pemanggilan alat Anda tervalidasi dengan benar.

### -1- Membuat arsitektur

Hal pertama yang perlu kita lakukan adalah membuat arsitektur yang membantu kita melakukan scale saat kita menambahkan lebih banyak fitur, berikut tampilannya:

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

Sekarang kita telah mengatur arsitektur yang memastikan kita dapat dengan mudah menambahkan alat baru dalam folder tools. Silakan ikuti ini untuk menambahkan subdirektori untuk resources dan prompts.

### -2- Membuat alat

Mari kita lihat seperti apa membuat alat berikutnya. Pertama, alat tersebut harus dibuat di subdirektori *tool* seperti berikut:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Validasi input menggunakan model Pydantic
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: tambahkan Pydantic, sehingga kita bisa membuat AddInputModel dan memvalidasi argumen

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Yang kita lihat di sini adalah bagaimana kita mendefinisikan nama, deskripsi, schema input menggunakan Pydantic dan handler yang akan dipanggil saat alat ini dipanggil. Terakhir, kita mengekspose `tool_add` yang merupakan dictionary yang memegang semua properti tersebut.

Ada juga *schema.py* yang digunakan untuk mendefinisikan schema input yang digunakan oleh alat kita:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Kita juga perlu mengisi *__init__.py* untuk memastikan direktori tools diperlakukan sebagai modul. Selain itu, kita perlu mengekspose modul di dalamnya seperti ini:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Kita dapat terus menambahkan ke file ini saat kita menambahkan lebih banyak alat.

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

Di sini kita membuat dictionary yang terdiri dari properti:

- name, ini adalah nama alat.
- rawSchema, ini adalah schema Zod, akan digunakan untuk memvalidasi permintaan masuk untuk memanggil alat ini.
- inputSchema, schema ini akan digunakan oleh handler.
- callback, ini digunakan untuk memanggil alat.

Ada juga `Tool` yang digunakan untuk mengonversi dictionary ini menjadi tipe yang dapat diterima handler server mcp dan terlihat seperti ini:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Dan ada *schema.ts* tempat kita menyimpan schema input untuk setiap alat yang tampak seperti ini dengan hanya satu schema saat ini tapi saat kita menambahkan alat kita bisa menambah entri lebih banyak:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Bagus, mari lanjutkan untuk menangani listing alat kita berikutnya.

### -3- Menangani listing alat

Selanjutnya, untuk menangani listing alat kita, kita perlu membuat request handler untuk itu. Berikut yang perlu kita tambahkan ke file server kita:

**Python**

```python
# kode dihilangkan untuk singkat
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

Di sini kita menambahkan dekorator `@server.list_tools` dan fungsi implementasi `handle_list_tools`. Dalam fungsi tersebut, kita perlu mengeluarkan daftar alat. Perhatikan bagaimana setiap alat harus memiliki nama, deskripsi, dan inputSchema.

**TypeScript**

Untuk mengatur request handler listing alat, kita perlu memanggil `setRequestHandler` pada server dengan schema yang sesuai dengan apa yang kita coba lakukan, dalam hal ini `ListToolsRequestSchema`.

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
// kode dihilangkan untuk singkatnya
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Mengembalikan daftar alat yang terdaftar
  return {
    tools: tools
  };
});
```

Bagus, sekarang kita telah menyelesaikan bagian listing alat, mari kita lihat bagaimana kita bisa memanggil alat berikutnya.

### -4- Menangani pemanggilan alat

Untuk memanggil alat, kita perlu mengatur request handler lain, kali ini fokus pada menangani permintaan yang menentukan fitur mana yang akan dipanggil dan dengan argumen apa.

**Python**

Mari gunakan dekorator `@server.call_tool` dan implementasikan dengan fungsi seperti `handle_call_tool`. Dalam fungsi itu, kita perlu memparsing nama alat, argumennya dan memastikan argumen valid untuk alat tersebut. Kita bisa memvalidasi argumen dalam fungsi ini atau di alat aktual.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools adalah sebuah dictionary dengan nama alat sebagai kunci
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

Berikut yang terjadi:

- Nama alat kita sudah ada sebagai parameter input `name` yang juga berlaku untuk argumen kita dalam bentuk dictionary `arguments`.

- Alat dipanggil dengan `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Validasi argumen terjadi di properti `handler` yang menunjuk ke fungsi, jika gagal akan melempar pengecualian.

Nah, sekarang kita memiliki pemahaman penuh tentang listing dan pemanggilan alat menggunakan server tingkat rendah.

Lihat [contoh lengkap](./code/README.md) di sini

## Tugas

Perluas kode yang diberikan dengan sejumlah alat, sumber daya dan prompt dan refleksikan bagaimana Anda hanya perlu menambahkan file di direktori tools dan tidak di tempat lain.

*Tidak ada solusi diberikan*

## Ringkasan

Di bab ini, kita telah melihat bagaimana pendekatan server tingkat rendah bekerja dan bagaimana itu dapat membantu kita membuat arsitektur yang bagus yang bisa terus kita bangun. Kita juga membahas validasi dan Anda diperlihatkan cara bekerja dengan pustaka validasi untuk membuat schema validasi input.

## Selanjutnya

- Selanjutnya: [Simple Authentication](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:  
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diingat bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang berwenang. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas setiap kesalahpahaman atau salah tafsir yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# Gelişmiş sunucu kullanımı

MCP SDK'da iki farklı sunucu türü vardır, normal sunucunuz ve düşük seviyeli sunucu. Normalde, özellik eklemek için düzenli sunucuyu kullanırsınız. Ancak bazı durumlarda, şu gibi nedenlerle düşük seviyeli sunucuya dayanmak isteyebilirsiniz:

- Daha iyi mimari. Hem düzenli sunucu hem de düşük seviyeli sunucu ile temiz bir mimari oluşturmak mümkündür ancak düşük seviyeli sunucu ile biraz daha kolay olduğu iddia edilebilir.
- Özellik kullanılabilirliği. Bazı gelişmiş özellikler yalnızca düşük seviyeli sunucu ile kullanılabilir. Örnek olarak örnekleme ve soru yöneltmeyi eklediğimiz sonraki bölümlerde bunu göreceksiniz.

## Normal sunucu ve düşük seviyeli sunucu karşılaştırması

Bir MCP Sunucusu oluşturmanın düzenli sunucu ile nasıl göründüğüne bakalım

**Python**

```python
mcp = FastMCP("Demo")

# Bir toplama aracı ekleyin
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

// Bir toplama aracı ekle
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

Buradaki nokta, sunucunun sahip olmasını istediğiniz her araç, kaynak veya istemi açıkça eklemenizdir. Bununla ilgili hiçbir yanlış yoktur.

### Düşük seviyeli sunucu yaklaşımı

Ancak, düşük seviyeli sunucu yaklaşımını kullandığınızda, her aracı kaydetmek yerine her özellik türü için (araçlar, kaynaklar veya istemler) iki tane işleyici oluşturmanız gerektiğini farklı olarak düşünmeniz gerekir. Örneğin araçlar için sadece şu iki işlev olur:

- Tüm araçları listeleme. Bir işlev araçları listelemek için yapılan tüm denemelerden sorumlu olur.
- Tüm araç çağrılarını işleme. Burada da bir işlev araç çağrılarını işler.

Bu kulağa daha az iş gibi geliyor değil mi? Yani araç kaydetmek yerine, yalnızca araçların listelendiğinden ve çağrı isteği geldiğinde çağrıldığından emin olmam yeterli olur.

Şimdi kodun nasıl göründüğüne bakalım:

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
  // Kayıtlı araçların listesini döndür
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

Burada artık, bir özellikler listesini döndüren bir işlev var. Araçlar listesindeki her giriş şimdi `name`, `description` ve `inputSchema` gibi alanlara sahip olup dönüş tipine uyuyor. Bu, araçlarımızı ve özellik tanımlarımızı başka bir yerde tutabilmemizi sağlar. Artık tüm araçlarımızı tools klasöründe oluşturabiliriz ve aynı şekilde tüm özellikler için de geçerlidir, böylece projeniz aniden şu şekilde organize olabilir:

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

Harika, mimarimiz oldukça temiz görünebilir.

Peki araç çağırmak, aynı fikir mi yani, bir işleyici ile istediğimiz aracı çağırmak? Evet, tam olarak, işte kodu:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools, anahtarları araç adları olan bir sözlüktür
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
    // TODO aracı çağır,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Yukarıdaki koddaki gibi, çağrılacak aracı ve hangi argümanlarla çağrılacağını ayrıştırmamız gerekiyor, sonra aracı çağırmamız gerekiyor.

## Validasyon ile yaklaşımı geliştirmek

Şimdiye kadar, araç, kaynak ve istem eklemek için yaptığınız tüm kayıtların her özellik türü için bu iki işleyici ile değiştirilebileceğini gördünüz. Başka ne yapmamız gerekiyor? Elbette, aracın doğru argümanlarla çağrıldığından emin olmak için bir tür doğrulama eklemeliyiz. Her çalışma zamanı bunun için kendi çözümüne sahiptir, örneğin Python Pydantic, TypeScript ise Zod kullanır. Fikir şudur:

- Bir özelliğin (araç, kaynak veya istem) oluşturma mantığını kendi klasörüne taşımak.
- Örneğin bir aracın çağrılması istenen gelen isteği doğrulamak için bir yol eklemek.

### Bir özellik oluşturmak

Bir özellik oluşturmak için, o özellik için bir dosya oluşturmanız ve ilgili zorunlu alanların olduğundan emin olmanız gerekiyor. Bu alanlar araçlar, kaynaklar ve istemlerde biraz farklıdır.

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
        # Girdiyi Pydantic modeli kullanarak doğrula
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ekle, böylece bir AddInputModel oluşturabilir ve argümanları doğrulayabiliriz

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Burada şu adımları nasıl yaptığımızı görebilirsiniz:

- *schema.py* dosyasında `AddInputModel` adlı Pydantic şemasını `a` ve `b` alanları ile oluşturmak.
- Gelen isteği `AddInputModel` olarak ayrıştırmaya çalışmak, parametrelerde bir uyuşmazlık varsa hata verecektir:

   ```python
   # add.py
    try:
        # Girişi Pydantic modeli kullanarak doğrula
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Bu ayrıştırma mantığını araç çağrısının kendisinde veya işleyici işlevinde yapmayı seçebilirsiniz.

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

- Tüm araç çağrılarını ele alan işleyicide, gelen isteği aracın tanımlı şemasına ayrıştırmaya çalışıyoruz:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    Bu başarılı olursa, aracı çağırmaya devam ediyoruz:

    ```typescript
    const result = await tool.callback(input);
    ```

Gördüğünüz gibi bu yaklaşım harika bir mimari yaratır çünkü her şey yerli yerindedir, *server.ts* sadece istek işleyicilerini bağlayan çok küçük bir dosyadır ve her özellik kendi klasöründedir, örneğin tools/, resources/ veya /prompts.

Harika, bunu şimdi oluşturmaya çalışalım.

## Alıştırma: Düşük seviyeli sunucu oluşturmak

Bu alıştırmada şunları yapacağız:

1. Araçları listeleyen ve araçları çağıran düşük seviyeli bir sunucu oluşturmak.
1. Üzerine inşa edebileceğiniz bir mimari uygulamak.
1. Araç çağrılarınızın doğru şekilde doğrulandığından emin olmak için doğrulama eklemek.

### -1- Bir mimari oluşturmak

İlk ele almamız gereken, daha fazla özellik ekledikçe bize ölçeklendirme yardımcı olacak bir mimari, işte nasıl görünüyor:

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

Artık tools klasörüne yeni araçlar eklemeyi kolaylaştıran bir mimari kurduk. Kaynaklar ve istemler için alt dizinler eklemek için istediğiniz gibi bunu takip edebilirsiniz.

### -2- Bir araç oluşturmak

Şimdi bir araç oluşturmanın nasıl göründüğüne bakalım. Öncelikle, şu şekilde kendi *tool* alt dizininde oluşturulması gerekir:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Girdiyi Pydantic modeli kullanarak doğrulayın
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydantic ekle, böylece bir AddInputModel oluşturabilir ve argümanları doğrulayabiliriz

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Burada ismi, açıklaması, Pydantic ile bir input şeması ve bir çağrı yapıldığında tetiklenecek işleyici tanımladığımızı görüyoruz. Son olarak tüm bu özelliklere sahip bir sözlük olan `tool_add`'i dışarıya açıyoruz.

Ayrıca aracımız tarafından kullanılan input şemasını tanımlayan *schema.py* dosyasına sahibiz:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Araçlar dizinini modül olarak kabul ettirmek için *__init__.py* dosyasını da doldurmamız gerekiyor. Ayrıca şu şekilde içindekileri dışarıya açmalıyız:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Yeni araçlar ekledikçe bu dosyayı büyütebiliriz.

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

Burada şu özelliklerden oluşan bir sözlük oluşturuyoruz:

- name, aracın adı.
- rawSchema, Zod şeması, aracın çağrılması için gelen istekleri doğrulamada kullanılır.
- inputSchema, bu şema işleyici tarafından kullanılır.
- callback, aracı çağırmak için kullanılır.

`Tool` adlı bir yapı da vardır, bu sözlüğü mcp sunucu işleyicisinin kabul edebileceği bir tipe dönüştürür, şöyle görünür:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Ve her araç için şemaları depoladığımız *schema.ts* dosyası vardır, şu anda sadece bir tane şema var ama araç ekledikçe daha fazla girdi ekleyebiliriz:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Harika, şimdi araçlarımızın listesini yönetmeye geçelim.

### -3- Araç listesini yönetmek

Araç listesini yönetmek için bir istek işleyici kurmamız gerekiyor. Sunucu dosyamıza eklememiz gerekenler şunlar:

**Python**

```python
# kısaltma için kod atlandı
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

Burada `@server.list_tools` dekoratörünü ve uygulayıcı işlev `handle_list_tools`'u ekliyoruz. Bu işlevde araçların listesini üretmeliyiz. Her aracın bir adı, açıklaması ve inputSchema'sı olmalı.

**TypeScript**

Araç listesini almak için istek işleyici ayarlamak üzere, yapılmak istenen işle uyumlu bir şema ile sunucuda `setRequestHandler` çağrılır, bu durumda `ListToolsRequestSchema`:

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
// Kısalık için kod atlandı
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Kayıtlı araçların listesini döndürür
  return {
    tools: tools
  };
});
```

Harika, şimdi araçları listeleme kısmını çözdük, gelelim araç çağırmaya.

### -4- Bir aracı çağırmayı yönetmek

Bir aracı çağırmak için başka bir istek işleyici kurmamız gerekiyor, bu sefer hangi özelliğin çağrılacağını ve hangi argümanlarla çağrılacağını belirten istekleri ele alacak.

**Python**

`@server.call_tool` dekoratörünü kullanalım ve `handle_call_tool` şeklinde uygulayalım. Bu işlev içinde araç adını, argümanlarını ayrıştırmalı ve argümanların ilgili araç için geçerli olup olmadığını doğrulamalıyız. Argümanları burada veya gerçek araçta doğrulayabiliriz.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools, araç isimlerini anahtar olarak kullanan bir sözlüktür
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # aracı çağır
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Şöyle oluyor:

- Araç adı zaten giriş parametresi `name` olarak mevcut ve argümanlar `arguments` isimli sözlük şeklinde.

- Araç şu şekilde çağrılır: `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Argümanların doğrulaması `handler` özelliğinde, bir işlevi gösterir. Başarısız olursa bir hata fırlatır.

İşte, düşük seviyeli sunucu kullanarak araçları listeleme ve çağırmayı tamamen anladık.

[Burada tam örneğe](./code/README.md) bakabilirsiniz.

## Ödev

Size verilen kodu bir dizi araç, kaynak ve istemle genişletin ve sadece tools dizinine dosya eklemeniz gerektiğini ve başka yere dosya eklemenize gerek olmadığını görün.

*Hiçbir çözüm verilmedi*

## Özet

Bu bölümde, düşük seviyeli sunucu yaklaşımının nasıl çalıştığını ve bunun üzerine kurabileceğimiz temiz bir mimari yaratmaya nasıl yardımcı olduğunu gördük. Ayrıca doğrulamadan bahsettik ve giriş doğrulama şemaları oluşturmak için doğrulama kütüphanelerinin nasıl kullanılacağını gösterdik.

## Sonraki Bölüm

- Devam: [Basit Kimlik Doğrulama](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri servisi [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba göstersek de, otomatik çevirilerin hata veya yanlışlıklar içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek herhangi bir yanlış anlaşılma veya yorum hatasından sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
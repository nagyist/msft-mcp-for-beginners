# 高度なサーバー使用法

MCP SDKには、通常のサーバーと低レベルサーバーの2種類のサーバーが公開されています。通常は、通常のサーバーを使用して機能を追加します。ただし、場合によっては低レベルサーバーに依存したいことがあります。例えば：

- より良いアーキテクチャ。通常のサーバーと低レベルサーバーの両方でクリーンなアーキテクチャを作成することは可能ですが、低レベルサーバーの方がわずかに簡単と言える場合があります。
- 機能の可用性。高度な機能の一部は低レベルサーバーでのみ使用可能です。サンプリングやエリシテーションを追加する後の章でこれが分かります。

## 通常のサーバー vs 低レベルサーバー

通常のサーバーでMCPサーバーを作成する例は以下の通りです。

**Python**

```python
mcp = FastMCP("Demo")

# 加算ツールを追加する
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

// 追加ツールを追加する
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

ポイントは、サーバーに持たせたい各ツール、リソース、またはプロンプトを明示的に追加することです。それ自体は問題ありません。

### 低レベルサーバーのアプローチ

しかし、低レベルサーバーのアプローチを使う場合は、各ツールを登録する代わりに、機能タイプごと（ツール、リソース、プロンプト）に2つのハンドラを作成するという風に考える必要があります。例えば、ツールには以下のような2つの関数だけがあります：

- すべてのツールを一覧表示する。1つの関数がすべてのツール一覧取得の試行を担当します。
- ツールの呼び出しを処理する。同様に、1つの関数がツール呼び出しの処理を担当します。

これは作業量が少なそうに聞こえますよね？ツールを登録する代わりに、ツール一覧取得時にそれをリストに含め、ツール呼び出し要求が来た際にはその呼び出しを処理すればよいのです。

コードがどのようになるか見てみましょう。

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
  // 登録されているツールのリストを返します
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

ここでは、機能のリストを返す関数があります。toolsリスト内の各エントリーは `name`、`description`、`inputSchema` といったフィールドを持ち、返り値の型に準拠しています。これによりツールや機能定義を別の場所に置くことができます。すべてのツールを tools フォルダに作成し、他のすべての機能も同様に整理できるため、プロジェクトは次のように構成されます。

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

素晴らしいですね。アーキテクチャがかなりクリーンに見えます。

ツールの呼び出しはどうでしょうか。同じ考え方で良いのでしょうか？ツールを呼び出す1つのハンドラがあり、どのツールでも呼び出すという形でしょうか？そうです、そのとおりです。以下がそのコード例です。

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools はツール名をキーとする辞書です
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
    // TODO ツールを呼び出す,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

上記のコードでわかるように、呼び出すツールとその引数を解析し、ツール呼び出し処理へ進みます。

## バリデーションを用いたアプローチの改良

これまで、ツール、リソース、プロンプトを追加するためのすべての登録を各機能タイプごとに2つのハンドラーで代替できることを示しました。ほかに何が必要でしょうか？それはツールが正しい引数で呼ばれていることを検証するための何らかのバリデーションを追加することです。各ランタイムには独自のソリューションがあります。例えば、PythonはPydanticを使い、TypeScriptはZodを使います。考えとしては以下の通りです：

- 機能（ツール、リソース、プロンプト）を作成するロジックをそれぞれの専用フォルダに移動する。
- ツール呼び出しなどの受け入れリクエストのバリデーション手段を用意する。

### 機能の作成

機能を作成するには、その機能用のファイルを作り、必要な必須フィールドが含まれていることを確認します。どのフィールドが必須かはツール、リソース、プロンプトで少し異なります。

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
        # Pydanticモデルを使って入力を検証する
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydanticを追加して、AddInputModelを作成し、引数を検証できるようにする

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ここで以下の処理を行います：

- Pydanticを使い `AddInputModel` というスキーマを *schema.py* ファイルで作成し、`a` と `b` フィールドを持たせる。
- 受信リクエストを `AddInputModel` として解析し、パラメータに不一致があればクラッシュさせる：

   ```python
   # add.py
    try:
        # Pydanticモデルを使用して入力を検証する
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

この解析ロジックはツール呼び出し自体に置くことも、ハンドラ関数に置くことも自由です。

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

- すべてのツール呼び出しを扱うハンドラでは、受信リクエストをツール定義のスキーマへ解析を試みます：

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    解析が成功したら、続いて実際にツールを呼び出します：

    ```typescript
    const result = await tool.callback(input);
    ```

ご覧の通り、このアプローチではすべてが所定の場所にあり、*server.ts* はリクエストハンドラを結線するだけの非常に小さいファイルとなり、各機能がそれぞれのフォルダ（tools/、resources/、prompts/）にあるので素晴らしいアーキテクチャになります。

それでは、次にこれを構築してみましょう。

## 演習：低レベルサーバーの作成

この演習では以下を行います：

1. ツール一覧の取得とツール呼び出しを扱う低レベルサーバーを作成する。
1. 拡張可能なアーキテクチャを実装する。
1. ツール呼び出しが適切にバリデートされていることを保証するバリデーションを追加する。

### -1- アーキテクチャの作成

まず、機能追加の際にスケール可能になるアーキテクチャを整えます。以下のようになります：

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

これでtoolsフォルダ内に新しいツールを簡単に追加できるアーキテクチャが構築できました。必要に応じてresourcesやpromptsのサブディレクトリを追加してください。

### -2- ツールの作成

次に実際にツールを作る例を見てみましょう。まず、対象の *tool* サブディレクトリでツールを作成します。

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Pydanticモデルを使用して入力を検証する
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: Pydanticを追加して、AddInputModelを作成し、引数を検証できるようにする

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

ここでは、Pydanticを使いname、descriptionと入力スキーマを定義し、このツール呼び出し時に実行されるハンドラを定義しています。最後に `tool_add` を公開してこれらのプロパティをまとめた辞書を提供します。

また、ツールの入力スキーマを定義するための *schema.py* もあります：

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

toolsディレクトリをモジュールとして扱うために *__init__.py* にも内容を入れる必要があります。さらに内部モジュールを公開するのも同様です：

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

ツールを追加するごとにこのファイルにも追記していけます。

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

ここでは以下のプロパティをもつ辞書を作成します：

- name：ツール名
- rawSchema：Zodスキーマで、ツール呼び出しリクエストのバリデーションに使う
- inputSchema：ハンドラーで利用するスキーマ
- callback：ツールを実行するためのコールバック

また、mcpサーバーハンドラーが受け入れ可能な型に変換する `Tool` も次のように定義しています：

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

さらに、各ツールの入力スキーマを格納する *schema.ts* は以下のように初期状態は1つだけのスキーマですが、ツール追加時にエントリを追加します：

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

素晴らしいですね。次にツール一覧取得を処理しましょう。

### -3- ツール一覧の処理

続いてツール一覧取得を処理するリクエストハンドラを設定します。サーバーファイルに以下を追加します。

**Python**

```python
# 簡潔にするためコードを省略しました
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

ここでは `@server.list_tools` デコレータと実装関数 `handle_list_tools` を追加しています。後者ではツール一覧を生成します。各ツールは名前、説明、inputSchemaを持つ必要があることに注意。

**TypeScript**

ツール一覧取得のために `setRequestHandler` をサーバー上で呼び出し、扱うリクエストのスキーマには `ListToolsRequestSchema` を使います。

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
// 簡潔にするためコードを省略
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // 登録されたツールのリストを返す
  return {
    tools: tools
  };
});
```

これでツール一覧取得は解決しました。次にツール呼び出しをどう扱うか見てみましょう。

### -4- ツール呼び出しの処理

ツール呼び出しにはもう一つのリクエストハンドラを設定します。呼び出す機能を指定し、どんな引数で呼び出すかに対応します。

**Python**

`@server.call_tool` デコレータを使い、`handle_call_tool` という関数を実装します。この中でツール名、引数を解析し、ツールへの引数が妥当か確認します。バリデーションはこの関数内またはツール内部で行えます。

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # toolsはツール名をキーとする辞書です
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # ツールを呼び出す
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

処理内容は以下の通り：

- ツール名は入力パラメータ `name` として既に存在し、引数は `arguments` 辞書として受け取る。
- ツールは `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)` で呼び出される。バリデーションは`handler`プロパティが指す関数内で行われ、失敗すると例外が発生する。

これで低レベルサーバーを使ったツール一覧と呼び出しが完全に理解できました。

[完全な例](./code/README.md)はこちら

## 課題

与えられたコードに複数のツール、リソース、プロンプトを追加し、toolsディレクトリにファイルを追加するだけでよく、他の場所を触らなくてよいことを実感してください。

*解答はありません*

## まとめ

この章では低レベルサーバーアプローチの仕組みと、それが拡張しやすいアーキテクチャを作るのに役立つことを学びました。またバリデーションについても説明し、バリデーションライブラリを用いて入力検証用のスキーマを作成する方法を示しました。

## 次に進む

- 次へ: [シンプル認証](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類はAI翻訳サービス「Co-op Translator」（https://github.com/Azure/co-op-translator）を使用して翻訳されています。正確性の確保に努めておりますが、自動翻訳には誤りや不正確な箇所が含まれる可能性があります。正確な情報の確認には、原文の言語で書かれたオリジナル文書を権威ある情報源としてご参照ください。重要な情報については、専門の人間翻訳をご利用いただくことを推奨します。本翻訳の利用に起因するいかなる誤解や解釈の相違についても、一切の責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
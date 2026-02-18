# Kehittynyt palvelimen käyttö

MCP SDK:ssa on kaksi erilaista palvelintyyppiä, tavallinen palvelin ja matalan tason palvelin. Tavallisesti käytät tavallista palvelinta lisätäksesi siihen ominaisuuksia. Joissain tapauksissa kuitenkin haluat tukeutua matalan tason palvelimeen, kuten esimerkiksi:

- Parempi arkkitehtuuri. On mahdollista luoda siisti arkkitehtuuri sekä tavallisella palvelimella että matalan tason palvelimella, mutta voidaan sanoa, että se on hieman helpompaa matalan tason palvelimella.
- Ominaisuuksien saatavuus. Joitain kehittyneitä ominaisuuksia voi käyttää vain matalan tason palvelimella. Näet tämän myöhemmissä luvuissa, kun lisäämme otantaa ja tiedonkeruuta.

## Tavallinen palvelin vs matalan tason palvelin

Näin MCP-palvelimen luominen näyttää tavallisella palvelimella

**Python**

```python
mcp = FastMCP("Demo")

# Lisää lisäystyökalu
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

// Lisää lisäystyökalu
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

Tarkoituksena on, että lisäät nimenomaisesti jokaisen työkalun, resurssin tai kehotteen, jonka haluat palvelimessa olevan. Siinä ei ole mitään vikaa.

### Matalan tason palvelinlähestymistapa

Kun käytät matalan tason palvelimen lähestymistapaa, sinun tulee ajatella asiaa eri tavalla, nimittäin sen sijaan, että rekisteröisit jokaisen työkalun erikseen, luot kaksi käsittelijää jokaista ominaisuustyyppiä (työkalut, resurssit tai kehotteet) kohden. Esimerkiksi työkaluilla on tällöin vain kaksi funktiota:

- Kaikkien työkalujen listaaminen. Yksi funktio vastaa kaikista yrityksistä listata työkaluja.
- Kaikkien työkalujen kutsujen käsittely. Täälläkin on vain yksi funktio, joka käsittelee kutsut työkaluille.

Tämä kuulostaa vähemmän työltä, eikö? Eli rekisteröinnin sijaan minun tarvitsee vain varmistaa, että työkalu näkyy kaikkien työkalujen listassa ja että se kutsutaan, kun saapuu pyyntö kutsua työkalua.

Katsotaanpa, miltä koodi näyttää nyt:

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
  // Palauta rekisteröityjen työkalujen luettelo
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

Tässä meillä nyt on funktio, joka palauttaa listan ominaisuuksista. Jokaisella työkalulistauksen merkinnällä on nyt kentät kuten `name`, `description` ja `inputSchema` vastaamaan paluuarvon tyyppiä. Tämä mahdollistaa työkalujen ja ominaisuusmääritelmien sijoittamisen muualle. Voimme nyt luoda kaikki työkalumme tools-kansioon ja samoin kaikki ominaisuutesi, joten projektisi voi olla järjestetty seuraavasti:

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

Hienoa, arkkitehtuuristamme voi tulla varsin siisti.

Entä työkalujen kutsuminen, onko ajatus sama, yksi käsittelijä kutsumaan työkalua, mitä tahansa työkalu? Kyllä, juuri niin, tässä koodi siihen:

**Python**

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools on sanakirja, jonka avaimina ovat työkalujen nimet
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
    // TODO kutsu työkalua,

    return {
       content: [{ type: "text", text: `Tool ${name} called with arguments: ${JSON.stringify(input)}, result: ${JSON.stringify(result)}` }]
    };
});
```

Kuten yllä olevasta koodista näet, meidän täytyy purkaa kutsuttava työkalu ja sen argumentit ja sitten siirtyä kutsumaan työkalua.

## Lähestymistavan parantaminen validoinnilla

Tähän mennessä olet nähnyt, kuinka kaikki rekisteröintisi työkalujen, resurssien ja kehotteiden lisäämiseksi voidaan korvata kahdella käsittelijällä jokaista ominaisuustyyppiä kohti. Mitä muuta tarvitsemme? No, meidän tulisi lisätä jonkinlaista validointia varmistaaksemme, että työkalua kutsutaan oikeilla argumenteilla. Jokaisella ajonaikaisella ympäristöllä on oma ratkaisunsa tähän, esimerkiksi Python käyttää Pydanticia ja TypeScript käyttää Zodia. Ajatuksena on tehdä seuraavaa:

- Siirtää logiikka ominaisuuden (työkalu, resurssi tai kehotte) luomiseksi erilliseen kansioon.
- Lisätä tapa validoida sisään tuleva pyyntö, joka esimerkiksi pyytää kutsumaan työkalua.

### Ominaisuuden luominen

Luodaksemme ominaisuuden, meidän täytyy luoda tiedosto kyseiselle ominaisuudelle ja varmistaa, että siinä on ominaisuudelle vaaditut pakolliset kentät. Kentät eroavat hieman työkalujen, resurssien ja kehotteiden välillä.

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
        # Vahvista syöte käyttämällä Pydantic-mallia
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: lisää Pydantic, jotta voimme luoda AddInputModelin ja vahvistaa argumentit

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

tässä näet, kuinka teemme seuraavaa:

- Luomme Pydantic-pohjaisen skeeman `AddInputModel`, jossa kenttinä `a` ja `b` tiedostossa *schema.py*.
- Yritämme jäsentää sisään tulevan pyynnön tyypiksi `AddInputModel`, jos parametrit eivät täsmää, tämä kaataa ohjelman:

   ```python
   # add.py
    try:
        # Vahvista syöte käyttämällä Pydantic-mallia
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")
   ```

Voit valita, laitatko tämän jäsentämislogiikan suoraan työkalukutsuun vai käsittelijäfunktioon.

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

- Kaikkia työkalukutsuja käsittelevässä käsittelijässä yritämme nyt jäsentää sisään tulevan pyynnön työkalun määriteltyyn skeemaan:

    ```typescript
    const Schema = tool.rawSchema;

    try {
       const input = Schema.parse(request.params.arguments);
    ```

    jos se onnistuu, jatkamme varsinaisen työkalun kutsumista:

    ```typescript
    const result = await tool.callback(input);
    ```

Kuten näet, tämä lähestymistapa luo erinomaisen arkkitehtuurin, sillä kaikella on paikkansa, ja *server.ts* on hyvin pieni tiedosto, joka vain yhdistää pyyntökäsittelijät, ja jokainen ominaisuus sijaitsee omassa kansiossaan eli tools/, resources/ tai /prompts.

Hienoa, kokeillaan rakentaa tämä seuraavaksi.

## Harjoitus: Matala-tason palvelimen luominen

Tässä harjoituksessa teemme seuraavaa:

1. Luomme matalan tason palvelimen, joka käsittelee työkalujen listauksen ja kutsun.
2. Toteutamme arkkitehtuurin, jonka päälle voi rakentaa.
3. Lisäämme validoinnin, jotta työkalukutsut validoidaan oikein.

### -1- Arkkitehtuurin luominen

Ensimmäinen asia, johon meidän täytyy kiinnittää huomiota, on arkkitehtuuri, joka auttaa meitä skaalaamaan lisätessämme ominaisuuksia, näin se näyttää:

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

Nyt olemme asettaneet arkkitehtuurin, joka varmistaa, että voimme helposti lisätä uusia työkaluja tools-kansioon. Voit halutessasi tehdä alikansioita myös resourcesille ja promptsille.

### -2- Työkalun luominen

Seuraavaksi katsotaan, miltä työkalun luominen näyttää. Ensin se pitää luoda *tool*-alikansioon seuraavasti:

**Python**

```python
from .schema import AddInputModel

async def add_handler(args) -> float:
    try:
        # Vahvista syöte Pydantic-mallia käyttäen
        input_model = AddInputModel(**args)
    except Exception as e:
        raise ValueError(f"Invalid input: {str(e)}")

    # TODO: lisää Pydantic, jotta voimme luoda AddInputModelin ja vahvistaa argumentit

    """Handler function for the add tool."""
    return float(input_model.a) + float(input_model.b)

tool_add = {
    "name": "add",
    "description": "Adds two numbers",
    "input_schema": AddInputModel,
    "handler": add_handler 
}
```

Tässä näemme, miten määrittelemme nimen, kuvauksen, syötteen skeeman Pydanticilla ja käsittelijän, joka kutsutaan, kun tätä työkalua käytetään. Lopuksi paljastamme `tool_add`, joka on sanakirja, jossa on kaikki nämä ominaisuudet.

On myös *schema.py*, jota käytetään määrittelemään työkalun syöttöskeema:

```python
from pydantic import BaseModel

class AddInputModel(BaseModel):
    a: float
    b: float
```

Meidän täytyy myös täyttää *__init__.py*, jotta tools-kansiota käsitellään moduulina. Lisäksi meidän pitää paljastaa moduulit siellä seuraavasti:

```python
from .add import tool_add

tools = {
  tool_add["name"] : tool_add
}
```

Voimme jatkaa tämän tiedoston täydentämistä, kun lisäämme uusia työkaluja.

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

Tässä luomme sanakirjan (dictionary), joka sisältää ominaisuudet:

- name, tämä on työkalun nimi.
- rawSchema, tämä on Zod-skeema, jota käytetään sisään tulevien työkalukutsujen validointiin.
- inputSchema, tätä skeemaa käyttää käsittelijä.
- callback, tätä käytetään työkalun kutsumiseen.

On myös `Tool`, jota käytetään muuntamaan tämä sanakirja tyypiksi, jonka MCP-palvelimen käsittelijä voi hyväksyä. Se näyttää tältä:

```typescript
import { z } from 'zod';

export interface Tool {
    name: string;
    inputSchema: any;
    rawSchema: z.ZodTypeAny;
    callback: (args: z.infer<z.ZodTypeAny>) => Promise<{ content: { type: string; text: string }[] }>;
}
```

Ja on *schema.ts*, johon tallennamme jokaisen työkalun syöttöskeemat, tällä hetkellä siellä on vain yksi skeema, mutta kun lisäämme työkaluja, lisäämme myös merkintöjä:

```typescript
import { z } from 'zod';

export const MathInputSchema = z.object({ a: z.number(), b: z.number() });
```

Hienoa, jatketaan työkalujen listauksen käsittelyllä seuraavaksi.

### -3- Työkalulistauksen käsittely

Seuraavaksi, käsitelläksemme työkalujen listausta, meidän täytyy asettaa pyyntökäsittelijä sitä varten. Tässä, mitä meidän pitää lisätä palvelintiedostoomme:

**Python**

```python
# koodi jätetty pois lyhyyden vuoksi
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

Lisäämme koristeen `@server.list_tools` ja toteutamme funktion `handle_list_tools`. Viimeksi mainitussa meidän täytyy tuottaa lista työkaluista. Huomaa, että jokaisella työkalulla tulee olla nimi, kuvaus ja inputSchema.

**TypeScript**

Työkalujen listauksen pyyntökäsittelijän asettamiseksi meidän täytyy kutsua `setRequestHandler` palvelimella skeeman kanssa, joka sopii haluamaamme toimintaan, tässä tapauksessa `ListToolsRequestSchema`.

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
// koodi on jätetty pois tiiviyden vuoksi
import { tools } from './tools/index.js';

server.setRequestHandler(ListToolsRequestSchema, async (request) => {
  // Palauta rekisteröityjen työkalujen lista
  return {
    tools: tools
  };
});
```

Hienoa, nyt olemme ratkaisseet työkalujen listauksen osan, katsotaan seuraavaksi, miten voisimme kutsua työkaluja.

### -4- Työkalun kutsun käsittely

Työkalun kutsumista varten meidän täytyy asettaa toinen pyyntökäsittelijä, joka keskittyy käsittelemään pyyntöä, jossa määritellään, mitä ominaisuutta kutsutaan ja millä argumenteilla.

**Python**

Käytetään koristetta `@server.call_tool` ja toteutetaan se funktiolla `handle_call_tool`. Tämän funktion sisällä meidän täytyy purkaa työkalun nimi, sen argumentit ja varmistaa, että argumentit ovat kelvollisia kyseiselle työkalulle. Voimme validoida argumentit joko tässä funktiossa tai varsinaisessa työkalussa.

```python
@server.call_tool()
async def handle_call_tool(
    name: str, arguments: dict[str, str] | None
) -> list[types.TextContent]:
    
    # tools on sanakirja, jossa työkalujen nimet ovat avaimina
    if name not in tools.tools:
        raise ValueError(f"Unknown tool: {name}")
    
    tool = tools.tools[name]

    result = "default"
    try:
        # kutsu työkalua
        result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)
    except Exception as e:
        raise ValueError(f"Error calling tool {name}: {str(e)}")

    return [
        types.TextContent(type="text", text=str(result))
    ] 
```

Näin se toimii:

- Työkalun nimi on jo saatavilla syöteparametrina `name`, kuten myös argumentit `arguments`-sanakirjan muodossa.

- Työkalu kutsutaan rivillä `result = await tool["handler"](../../../../03-GettingStarted/10-advanced/arguments)`. Argumenttien validointi tapahtuu `handler`-ominaisuudessa, joka osoittaa funktioon, ja jos validointi epäonnistuu, se nostaa poikkeuksen.

Nyt meillä on täydellinen ymmärrys työkalujen listaamisesta ja kutsumisesta matalan tason palvelimen avulla.

Katso [täysi esimerkki](./code/README.md) täältä

## Tehtävä

Laajenna sinulle annettua koodia useilla työkaluilla, resursseilla ja kehotteilla ja pohdi, miten huomaat, että sinun tarvitsee lisätä tiedostoja vain tools-kansioon eikä muualle.

*Ratkaisua ei anneta*

## Yhteenveto

Tässä luvussa näimme, miten matalan tason palvelin toimii ja miten se voi auttaa meitä luomaan siistin arkkitehtuurin, johon voimme jatkossakin rakentaa. Kävimme myös läpi validointia ja näytettiin, miten voit työskennellä validointikirjastojen kanssa luodaksesi syötteen validointiskeemat.

## Mitä seuraavaksi

- Seuraava: [Yksinkertainen todennus](../11-simple-auth/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:  
Tämä asiakirja on käännetty tekoälypohjaisella käännöspalvelulla [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, ole hyvä ja huomioi, että automaattikäännöksissä saattaa esiintyä virheitä tai epätarkkuuksia. Alkuperäistä asiakirjaa sen alkuperäiskielellä tulee pitää virallisena lähteenä. Tärkeiden tietojen kohdalla suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä mahdollisesti aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
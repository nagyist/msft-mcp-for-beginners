## စတင်ရန်  

[![Build Your First MCP Server](../../../translated_images/my/04.0ea920069efd979a.webp)](https://youtu.be/sNDZO9N4m9Y)

_(ဤသင်ခန်းစာ၏ ဗီဒီယိုကိုကြည့်ရန် အထက်ပါပုံကို နှိပ်ပါ)_

ဤအပိုင်းတွင် သင်ခန်းစာအတော်များများ ပါဝင်သည်-

- **1 သင့်ပထမဆုံး ဆာဗာ**၊ ဤပထမဆုံးသင်ခန်းစာတွင် သင့်ပထမဆုံးဆာဗာ ဖန်တီးခြင်းနှင့် inspector tool ဖြင့် စစ်ဆေးသုံးသပ်ခြင်းကို သင်ယူမည်ဖြစ်သည်၊  ဆာဗာကို စမ်းသပ်ပြီး ပြင်ဆင်ရန် အရေးပါသော နည်းလမ်းဖြစ်သည်၊ [သင်ခန်းစာသို့](01-first-server/README.md)

- **2 Client**၊ ဤသင်ခန်းစာတွင် သင့်ဆာဗာနှင့် ချိတ်ဆက်နိုင်သည့် client များရေးသားနည်း သင်ယူမည်၊ [သင်ခန်းစာသို့](02-client/README.md)

- **3 Client with LLM**၊ client ရေးသားရာ၌ ပိုမိုကောင်းမွန်သော နည်းလမ်းတစ်ခုမှာ LLM ကို ထည့်သွင်းခြင်းဖြင့် ဆာဗာနှင့် "ညှိနှိုင်း" ဆွေးနွေးစေခြင်းဖြစ်သည်၊ [သင်ခန်းစာသို့](03-llm-client/README.md)

- **4 Consuming a server GitHub Copilot Agent mode in Visual Studio Code**။ ဤနေရာတွင် Visual Studio Code ထဲမှ MCP Server ကို အသုံးပြုခြင်းကို ကြည့်ရှုမည်၊ [သင်ခန်းစာသို့](04-vscode/README.md)

- **5 stdio Transport Server** stdio ကောင်းမွန်သော MCP server-to-client တွဲဖက်စကားပြောဆက်ဆံမှု အခြေပြုပါသည်၊ လုံခြုံသော subprocess-based ဆက်သွယ်မှုနှင့် process isolation ပါရှိသည် [သင်ခန်းစာသို့](05-stdio-server/README.md)

- **6 HTTP Streaming with MCP (Streamable HTTP)** ။ ခေတ်မီ HTTP streaming ကူးပြောင်းနည်းများနှင့် MCP servers နှင့် clients များကို ပုံမှန်ပြုသူတွေအတွက် Streamable HTTP အသုံးပြုပြီး စက်ရုပ်တင်မည့် real-time နှင့် scalable servers များ ဖန်တီးနည်းကို သင်ယူမည်၊ [သင်ခန်းစာသို့](06-http-streaming/README.md)

- **7 Utilising AI Toolkit for VSCode** MCP Clients နှင့် Servers ကို စမ်းသပ်အသုံးပြုရန် [သင်ခန်းစာသို့](07-aitk/README.md)

- **8 Testing** ။ ဆာဗာနှင့် client များကို ကွဲပြားသောနည်းလမ်းဖြင့် စမ်းသပ်သုံးသပ်ခြင်းအပေါ် အာရုံစိုက်မည်၊ [သင်ခန်းစာသို့](08-testing/README.md)

- **9 Deployment** ။ MCP ဖြေရှင်းချက်များ တပ်ဆင်သည့်နည်းလမ်းများကို ပြသမည်၊ [သင်ခန်းစာသို့](09-deployment/README.md)

- **10 Advanced server usage** ။ ဆာဗာအသုံးပြုမှု အဆင့်မြင့်အကြောင်းများကို ဖော်ပြမည်၊ [သင်ခန်းစာသို့](./10-advanced/README.md)

- **11 Auth** ။ ကိုယ်ပိုင် auth ဖြည့်စွက်နည်း၊ Basic Auth မှ JWT နှင့် RBAC အသုံးပြုခြင်းအထိ ဖော်ပြထားသည်။ နောက်ထပ်လုံခြုံရေးအဖွဲ့အစည်းများဆိုင်ရာ အကြံပြုချက်များနှင့် အဆင့်မြင့်အကြောင်းများအတွက် အစပိုင်းတွင် စတင်ရန် ညွှန်ကြားသည်၊ [သင်ခန်းစာသို့](./11-simple-auth/README.md)

- **12 MCP Hosts**။ Claude Desktop, Cursor, Cline နှင့် Windsurf စသည့် လူကြိုက်များသော MCP host clients များကို ဆက်တင် ပြုလုပ်၍ အသုံးပြုရန်၊ ဆက်သွယ်မှုအမျိုးအစားများနှင့် အမှားဖော်ပြချက်များလေ့လာရန်၊ [သင်ခန်းစာသို့](./12-mcp-hosts/README.md)

- **13 MCP Inspector**။ MCP Inspector tool ဖြင့် MCP servers များကို ကိုင်တွယ်စမ်းသပ်ခြင်း၊ troubleshooting tools, resources, protocol မက်ဆေ့ခ်ျများ သင်ယူရန်၊ [သင်ခန်းစာသို့](./13-mcp-inspector/README.md)

Model Context Protocol (MCP) သည် LLM များအတွက် context ပေးသည့် application များ မလိုက်လျောညီထွေဖြစ်စေသော ဖွင့်လှစ်ထားသော protocol ဖြစ်သည်။ MCP ကို USB-C port တစ်ခုအဖြစ်စဉ်းစားနိုင်ပြီး AI applications များအတွက် မတူညီသော ဒေတာရင်းမြစ်နှင့် စက်မှုကိရိယာများသို့ ချိတ်ဆက်ပေးသော စံနမူနာဖြစ်သည်။

## သင်ယူရမည့် ရည်ရွယ်ချက်များ

ဤသင်ခန်းစာအဆုံးတွင် သင်သည်:

- C#, Java, Python, TypeScript နှင့် JavaScript များအတွက် MCP ဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင်များကို တပ်ဆင်နိင်မည်
- စိတ်ကြိုက် features (resources, prompts, tools) ဖြင့် အခြေခံ MCP servers များကို ဖန်တီးတည်ဆောက်၍ တပ်ဆင်နိုင်မည်
- MCP servers များနှင့် ချိတ်ဆက်သည့် host applications များကို ဖန်တီးနိုင်မည်
- MCP implementation များကို စမ်းသပ်ပြီး ပြင်ဆင်နိုင်မည်
- setup ပြဿနာများနှင့် ဖြေရှင်းနည်းများကို နားလည်နိုင်မည်
- MCP implementation များကို လူကြိုက်အများဆုံး LLM ဝန်ဆောင်မှုများနှင့် ချိတ်ဆက်နိုင်မည်

## MCP ပတ်ဝန်းကျင် ကို ပြင်ဆင်ခြင်း

MCP ဖြင့် လုပ်ကိုင်မတိုင်မီ သင်တင်ဆင်ရန် တိုးတက်ပတ်ဝန်းကျင်ကို ပြင်ဆင်ပြီး အခြေခံ workflow ကို နားလည်ထားဖို့ အရေးကြီးသည်။ ဤအပိုင်းတွင် MCP ဖြင့် လွယ်ကူစွာ စတင်နိုင်ရန် ကူညီညွှန်ပြထားသည်။

### လိုအပ်ချက်များ

MCP ဖွံ့ဖြိုးရေးစတင်မတိုင်မီ အောက်ပါများရှိမည်ဟု သေချာစေပါ-

- **ဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင်** C#, Java, Python, TypeScript သို့မဟုတ် JavaScript အတွက်
- **IDE/Editor** Visual Studio, Visual Studio Code, IntelliJ, Eclipse, PyCharm သို့မဟုတ် ထူးခြားခေတ်မီ code editor များ
- **Package Managers** NuGet, Maven/Gradle, pip, သို့မဟုတ် npm/yarn
- **API Keys** သင်၏ host applications တွင် အသုံးပြုသည့် AI ဝန်ဆောင်မှုများအတွက်

### တရားဝင် SDK များ

လာမည့်အခန်းများတွင် Python, TypeScript, Java, .NET အသုံးပြု၍ ဖြေရှင်းချက်များကို မြင်တွေ့မည်ဖြစ်သည်။ အောက်တွင် တရားဝင်ထောက်ပံ့သော SDK များပါရှိသည်။

MCP သည် ဘာသာစကားများစွာအတွက် တရားဝင် SDK များ ပံ့ပိုးပေးသည် ([MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/) နှင့် ကိုက်ညီသည်)-
- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk) - Microsoft နှင့် ပူးပေါင်း ထိန်းသိမ်းနေသည်
- [Java SDK](https://github.com/modelcontextprotocol/java-sdk) - Spring AI နှင့် ပူးပေါင်း ထိန်းသိမ်းနေသည်
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) - တရားဝင် TypeScript ပုံစံ
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk) - တရားဝင် Python ပုံစံ (FastMCP)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk) - တရားဝင် Kotlin ပုံစံ
- [Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - Loopwork AI နှင့် ပူးပေါင်း ထိန်းသိမ်းနေသည်
- [Rust SDK](https://github.com/modelcontextprotocol/rust-sdk) - တရားဝင် Rust ပုံစံ
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk) - တရားဝင် Go ပုံစံ

## အဓိကအချက်များ

- MCP ဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင် တပ်ဆင်ခြင်းမှာ ဘာသာစကားအလိုက် SDK များဖြင့် လွယ်ကူသည်
- MCP servers ကို tools များဖန်တီးပြီး အတည်ပြုထားသော schema များဖြင့် မှတ်ပုံတင်ခြင်းဖြင့် တည်ဆောက်သည်
- MCP clients များသည် servers နှင့် model များကို ချိတ်ဆက်ကာ ဆန်းသစ်ပြီးမယူနိုင်စွမ်းများကို အသုံးချသည်
- စမ်းသပ်ခြင်းနှင့် ပြင်ဆင်ခြင်းသည် MCP implementation များ ရေစီးကြောင်းတည်ငြိမ်မှုအတွက် အရေးကြီးသည်
- တပ်ဆင်ခြင်းနည်းလမ်းများမှာ ဒေသတွင်း ဖွံ့ဖြိုးမှုပတ်ဝန်းကျင်မှ cloud အခြေခံ ဖြေရှင်းချက်ထိ ဖြစ်နိုင်သည်

## လေ့ကျင့်သုံးသပ်ခြင်း

ဤအပိုင်းရှိ အခန်းများရှိ စမ်းသပ်များကို ဖြည့်ဆည်းပေးသော နမူနာများရှိသည်။ ထို့ပြင် အခန်းတစ်ခုချင်းစီတွင် မိမိတို့၏ လေ့ကျင့်ခန်းများနှင့်တာဝန်များ ပါရှိသည်-

- [Java Calculator](./samples/java/calculator/README.md)
- [.Net Calculator](../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](./samples/javascript/README.md)
- [TypeScript Calculator](./samples/typescript/README.md)
- [Python Calculator](../../../03-GettingStarted/samples/python)

## ထပ်ဆောင်း ရင်းမြစ်များ

- [Build Agents using Model Context Protocol on Azure](https://learn.microsoft.com/azure/developer/ai/intro-agents-mcp)
- [Remote MCP with Azure Container Apps (Node.js/TypeScript/JavaScript)](https://learn.microsoft.com/samples/azure-samples/mcp-container-ts/mcp-container-ts/)
- [.NET OpenAI MCP Agent](https://learn.microsoft.com/samples/azure-samples/openai-mcp-agent-dotnet/openai-mcp-agent-dotnet/)

## နောက်တစ်ခု

ပထမဆုံးသင်ခန်းစာဖြင့် စတင်ပါ- [သင့်ပထမ စသော MCP Server ပြုလုပ်ခြင်း](01-first-server/README.md)

ဤ module ပြီးဆုံးလျှင် ဆက်လက်သင့်သည်- [Module 4: Practical Implementation](../04-PracticalImplementation/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**  
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါတယ်။ မှန်ကန်မှုကို ကြိုးစားကြပေမယ့်၊ စက်ဖြင့် ပြန်ဆိုမှုတွင် အမှားများ သို့မဟုတ် မမှန်ကန်မှုများ ရှိနိုင်ကြောင်း သိရှိထားဖို့ လိုအပ်ပါသည်။ မူရင်းစာတမ်းကို မူလဘာသာစကားဖြင့်သာ တရားဝင်အချက်အလက်အေနဖြင့် ယူဆသင့်သည်။ အရေးကြီးသော အချက်အလက်များအတွက် အသက်မွေးဝမ်းကြောင်းပညာရှင်များမှ ပြန်ဆိုမှုကို ဦးစားပေးရန် အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်အတွက် မွားယွင်းချက်များ သို့မဟုတ် ဖော်ပြချက်အနည်းလေးများကြောင့် ဖြစ်ပေါ်လာသော အခက်အခဲများအတွက် ကျွန်ုပ်တို့ တာဝန်မကျေပွန်ပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
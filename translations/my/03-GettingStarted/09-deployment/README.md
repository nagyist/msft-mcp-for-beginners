# MCP ဆာဗာများ တပ်ဆင်ခြင်း

သင်၏ MCP ဆာဗာကို တပ်ဆင်ခြင်းဖြင့် တခြားသူများသည် မိမိ၏ tools နှင့် အရင်းအမြစ်များကို လက်ရှိ ပတ်ဝန်းကျင်ထက် ကျော်၍ လက်လှမ်းမီနိုင်ပါသည်။ သင်၏ စွမ်းဆောင်ရည် မြှင့်တင်မှု၊ ယုံကြည်စိတ်ချရမှု နှင့် စီမံခန့်ခွဲရလွယ်ကူမှုလိုအပ်ချက်များကို မူတည်၍ တပ်ဆင်နိုင်စေသော လမ်းကြောင်းပုံစံ များစွာ ရှိပါသည်။ အောက်တွင် သင်အနေဖြင့် MCP ဆာဗာများကို ဒေသခံတွင်၊ ကွန်တိနာတွင် နှင့် cloud တွင် တပ်ဆင်နည်း အကြံဉာဏ်များကို တွေ့ရှိနိုင်ပါသည်။

## အနှစ်ချုပ်

ဤသင်ခန်းစာသည် သင်၏ MCP Server အက်ပ်ကို ကောင်းစွာ တပ်ဆင်နည်းကို ဖော်ပြပါသည်။

## သင်ယူရန် ရည်ရွယ်ချက်များ

ဤသင်ခန်းစာအဆုံးတွင် သင်အနေဖြင့် -

- တပ်ဆင်နိုင်မှု လမ်းလျှောက်နည်း များကို သုံးသပ်နိုင်မည်။
- သင်၏ အက်ပ်ကို တပ်ဆင်နိုင်မည်။

## ဒေသခံ ဖွံ့ဖြိုးတိုးတက်မှု နှင့် တပ်ဆင်မှု

သင်၏ ဆာဗာသည် အသုံးပြုသူ၏ စက်ပေါ်တွင် လည်ပတ်ရန် ရည်ရွယ်ပါက အောက်ပါအဆင့်များလိုက်နာနိုင်ပါသည်။

1. **ဆာဗာကို ဒေါင်းလုတ်လုပ်ပါ**။ သင် ဆာဗာကို မရေးသားခဲ့ပါက အရင်ဆုံးစက်ထဲ ထည့်ပါ။
1. **ဆာဗာ လည်ပတ်မှု စတင်ပါ**: သင်၏ MCP ဆာဗာ အက်ပ်ကို လည်ပတ်ပါ။

SSE (stdio type server မလိုအပ်ပါ)

1. **ကွန်ယက်ဆက်သွယ်မှု ပြင်ဆင်ပါ**: ဆာဗာသည် မျှော်မှန်းထားသော port မှာ လက်လှမ်းမီနိုင်စေရန် ဆောင်ရွက်ပါ။
1. **အသုံးပြုသူများ ချိတ်ဆက်ပါ**: `http://localhost:3000` ကဲ့သို့သော ဒေသခံ ချိတ်ဆက် URL များကို အသုံးပြုပါ။

## Cloud တပ်ဆင်ခြင်း

MCP ဆာဗာများကို မတူညီသော cloud ပလက်ဖောင်းများတွင် တပ်ဆင်နိုင်ပါသည်-

- **Serverless Functions**: ပေါ့ပါးသော MCP ဆာဗာများကို serverless functions အဖြစ် တပ်ဆင်ခြင်း
- **Container Services**: Azure Container Apps, AWS ECS, သို့မဟုတ် Google Cloud Run ကဲ့သို့သော ဝန်ဆောင်မှုများကို အသုံးပြုခြင်း
- **Kubernetes**: မြင့်မားသော ရရှိနိုင်မှုအတွက် MCP ဆာဗာများကို Kubernetes clusters တွင် တပ်ဆင်ပြီး စီမံခန့်ခွဲခြင်း

### ဥပမာ- Azure Container Apps

Azure Container Apps သည် MCP Server များ၏ တပ်ဆင်မှုကို ထောက်ပံ့ပေးသည်။ ယခုအချိန်တွင် အလုပ်များ ဆက်လက်လုပ်ဆောင်နေပြီး အခုအချိန်တွင် SSE ဆာဗာများကို ထောက်ပံ့သည်။

အောက်ပါအတိုင်း ဆက်လက်လုပ်ဆောင်နိုင်ပါသည်-

1. Repo တစ်ခုကို clone လုပ်ပါ-

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. ဒေသခံတွင် စမ်းသပ်ရန် လည်ပတ်ပါ-

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. ဒေသခံတွင် စမ်းသပ်ရန်၊ *.vscode* ဖိုဒါထဲတွင် *mcp.json* ဖိုင်တစ်ဖိုင် ဖန်တီးပြီး အောက်ပါ အကြောင်းအရာ ထည့်ပါ-

  ```json
  {
      "inputs": [
          {
              "type": "promptString",
              "id": "weather-api-key",
              "description": "Weather API Key",
              "password": true
          }
      ],
      "servers": {
          "weather-sse": {
              "type": "sse",
              "url": "http://localhost:8000/sse",
              "headers": {
                  "x-api-key": "${input:weather-api-key}"
              }
          }
      }
  }
  ```

  SSE server လည်ပတ်သည်နှင့်အမျှ JSON ဖိုင်ရှိ play အိုင်ကွန်ကို နှိပ်ပါက GitHub Copilot မှ ဆာဗာ၏ tools များကို ရယူနေသည်ကို တွေ့ရမည်၊ Tool အိုင်ကွန်ကို ကြည့်ပါ။

1. တပ်ဆင်ရန် အောက်ပါ command ကို အလုပ်လုပ်ပါ-

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

ဒါဆိုပြီး၊ ဒေသခံတွင် တပ်ဆင်ပြီး Azure သို့ ဒီလမ်းညွှန်များအားဖြင့် တပ်ဆင်နိုင်ပါပြီ။

## အပိုဆောင်း အရင်းအမြစ်များ

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps article](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP repo](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## နောက်တစ်ဆင့်

- နောက်တစ်ဆင့်: [Advanced Server Topics](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ကန့်သတ်ချက်**  
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) ကိုအသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ တိကျမှုအတွက်ကြိုးစားပေမယ့် အလိုအလျောက်ဘာသာပြန်ချက်များတွင် အမှားများ သို့မဟုတ် တိကျမှုမရှိနိုင်ခြင်းများရှိနိုင်ကြောင်း သတိပြုပါ။ မူလစာတမ်းသည် မူရင်းဘာသာဖြင့်သာ တရားဝင်အရင်းအမြစ်အဖြစ် ဖတ်ရှုသင့်ပါသည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ကျွမ်းကျင်သော လူ့ဘာသာပြန်ခြင်းကို ထုတ်ပြန်ရန် အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုမှုကြောင့် ဖြစ်ပေါ်မည့် နားလည်မှုပျက်ခြင်း သို့မဟုတ် အဓိပ္ပါယ်ခွဲခြမ်းမှုများအတွက် ကျွန်တော်တို့ဖြစ်သော တာဝန်မရှိပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
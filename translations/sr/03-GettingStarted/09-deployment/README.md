# Деплојмент MCP сервера

Деплојмент вашег MCP сервера омогућава другима приступ његовим алатима и ресурсима ван вашег локалног окружења. Постоји неколико стратегија деплојмента које треба размотрити, у зависности од ваших захтева за скалабилношћу, поузданошћу и лакоћом управљања. Испод ћете пронаћи смернице за деплојмент MCP сервера локално, у контејнерима и у облаку.

## Преглед

Ова лекција покрива како да деплојујете вашу MCP сервер апликацију.

## Циљеви учења

До краја ове лекције, моћи ћете да:

- Процените различите приступе деплојменту.
- Деплојујете вашу апликацију.

## Локални развој и деплојмент

Ако је ваш сервер намењен да се користи покретањем на корисничкој машини, можете пратити следеће кораке:

1. **Преузмите сервер**. Ако нисте написали сервер, преузмите га прво на вашу машину.  
1. **Покрените серверски процес**: Покрените вашу MCP сервер апликацију

За SSE (није потребно за stdio тип сервера)

1. **Конфигуришите мрежу**: Обезбедите приступ серверу на очекиваном порту  
1. **Повежите клијенте**: Користите локалне URL-ове за повезивање као што је `http://localhost:3000`

## Деплојмент у облаку

MCP сервери могу бити деплојовани на разним облачним платформама:

- **Serverless функциије**: Деплојујте лагане MCP сервере као serverless функциије
- **Услуге контејнера**: Користите услуге као што су Azure Container Apps, AWS ECS или Google Cloud Run
- **Kubernetes**: Деплојујте и управљајте MCP серверима у Kubernetes кластерима за високу доступност

### Пример: Azure Container Apps

Azure Container Apps подржава деплојмент MCP сервера. Још увек је у развоју и тренутно подржава SSE сервере.

Ево како то можете урадити:

1. Клонирајте репо:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Покрените га локално да тестирате:

  ```sh
  uv venv
  uv sync

  # линукс/macOS
  export API_KEYS=<AN_API_KEY>
  # виндоус
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Да бисте га пробали локално, направите *mcp.json* фајл у *.vscode* директоријуму и додајте следећи садржај:

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

  Када се SSE сервер покрене, можете кликнути на икону за пуштање у JSON фајлу, сада би алати на серверу требали бити препознати од стране GitHub Copilot-а, погледајте икону алата.

1. За деплојмент покрените следећу команду:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Ето га, деплојтујте га локално или у Azure следећи ове кораке.

## Додатни ресурси

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Чланак о Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP репо](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Шта следи

- Следеће: [Напредне теме о серверима](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Одрицање од одговорности**:  
Овај документ је преведен коришћењем АИ сервиса за превођење [Co-op Translator](https://github.com/Azure/co-op-translator). Иако настављамо да тежимо прецизности, молимо вас да имате у виду да аутоматизовани преводи могу да садрже грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитетним извором. За кључне информације препоручује се професионални људски превод. Не сносимо одговорност за било каква неспоразума или погрешне тумачења настала коришћењем овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->
# MCP ਸਰਵਰ ਤੈਅ ਕਰਨਾ

ਆਪਣਾ MCP ਸਰਵਰ ਤੈਅ ਕਰਨ ਨਾਲ ਹੋਰ ਲੋਕ ਇਸਦੇ ਉਪਕਰਣਾਂ ਅਤੇ ਸਾਧਨਾਂ ਤੱਕ ਤੁਹਾਡੇ ਸਥਾਨਕ ਵਾਤਾਵਰਣ ਤੋਂ ਪਰੇ ਪਹੁੰਚ ਸਕਦੇ ਹਨ। ਤੁਹਾਡੀਆਂ ਮੁੱਖ ਲੋੜਾਂ ਜਿਵੇਂ ਕਿ ਪੈਮਾਨਾ, ਵਿਸ਼ਵਸਨੀਯਤਾ ਅਤੇ ਪਰਬੰਧਨ ਦੀ ਸੌਖਿਆ ਦੇਅਨੁਸਾਰ ਕਈ ਤਰੀਕੇ ਹਨ। ਹੇਠਾਂ ਤੁਹਾਨੂੰ MCP ਸਰਵਰਾਂ ਨੂੰ ਸਥਾਨਕ, ਕੰਟੇਨਰਾਂ ਵਿੱਚ ਅਤੇ ਕਲਾਉਡ ਵਿੱਚ ਤੈਅ ਕਰਨ ਲਈ ਰਾਹਨੁਮਾ ਮਿਲੇਗਾ।

## ਸੁਝਾਅ

ਇਸ ਪਾਠ ਵਿੱਚ ਤੁਸੀਂ ਆਪਣੇ MCP ਸਰਵਰ ਐਪ ਨੂੰ ਕਿਵੇਂ ਤੈਅ ਕਰਨਾ ਹੈ, ਬਾਰੇ ਸਿੱਖੋਗੇ।

## ਸਿੱਖਣ ਦੇ ਉਦੇਸ਼

ਇਸ ਪਾਠ ਦੇ ਅੰਤ ਵਿੱਚ, ਤੁਸੀਂ ਸਮਰੱਥ ਹੋਵੋਗੇ:

- ਵੱਖ-ਵੱਖ ਤੈਅ ਕਰਨ ਦੇ ਤਰੀਕਿਆਂ ਦਾ ਮੁਲਾਂਕਣ ਕਰਨ ਲਈ।
- ਆਪਣਾ ਐਪ ਤੈਅ ਕਰਨ ਲਈ।

## ਸਥਾਨਕ ਵਿਕਾਸ ਅਤੇ ਤੈਅ

ਜੇ ਤੁਹਾਡਾ ਸਰਵਰ ਉਪਭੋਗਤਾ ਦੀ ਮਸ਼ੀਨ 'ਤੇ ਚਲ ਕੇ ਵਰਤੋਂ ਲਈ ਬਣਾਇਆ ਗਿਆ ਹੈ, ਤਾਂ ਤੁਸੀਂ ਹੇਠਾਂ ਦਿੱਤੀਆਂ ਵੀਹਕੀਆਂ ਫਾਲੋ ਕਰ ਸਕਦੇ ਹੋ:

1. **ਸਰਵਰ ਡਾਊਨਲੋਡ ਕਰੋ**। ਜੇ ਤੁਸੀਂ ਸਰਵਰ ਨਹੀਂ ਲਿਖਿਆ, ਤਾਂ ਪਹਿਲਾਂ ਇਸਨੂੰ ਆਪਣੇ ਮਸ਼ੀਨ 'ਤੇ ਡਾਊਨਲੋਡ ਕਰੋ।  
1. **ਸਰਵਰ ਪ੍ਰਕਿਰਿਆ ਚਾਲੂ ਕਰੋ**: ਆਪਣੇ MCP ਸਰਵਰ ਐਪਲੀਕੇਸ਼ਨ ਨੂੰ ਚਲਾ ਕੇ ਸਟਾਰਟ ਕਰੋ।

SSE ਲਈ (stdio ਪ੍ਰਕਾਰ ਦੇ ਸਰਵਰ ਲਈ ਜ਼ਰੂਰੀ ਨਹੀਂ)

1. **ਨੈੱਟਵਰਕਿੰਗ ਸੰਰਚਨਾ ਕਰੋ**: ਇਹ ਯਕੀਨੀ ਬਣਾਉ ਕਿ ਸਰਵਰ ਉਮੀਦ ਕੀਤੇ ਗਏ ਪੋਰਟ 'ਤੇ ਪਹੁੰਚਯੋਗ ਹੈ।  
1. **ਕਲਾਇੰਟ ਕਨੈਕਟ ਕਰੋ**: ਸਥਾਨਕ ਕਨੈਕਸ਼ਨ URLs ਵਰਤੋ ਜਿਵੇਂ `http://localhost:3000`

## ਕਲਾਉਡ ਤੈਅ

MCP ਸਰਵਰ ਵੱਖ-ਵੱਖ ਕਲਾਉਡ ਪਲੇਟਫਾਰਮਾਂ ਤੇ ਤੈਅ ਕੀਤੇ ਜਾ ਸਕਦੇ ਹਨ:

- **ਸਰਵਰਲੈੱਸ ਫੰਕਸ਼ਨਜ਼**: MCP ਸਰਵਰਾਂ ਨੂੰ ਹਲਕੇ ਸਰਵਰਲੈੱਸ ਫੰਕਸ਼ਨਜ਼ ਵਜੋਂ ਤੈਅ ਕਰੋ।  
- **ਕੰਟੇਨਰ ਸੇਵਾਵਾਂ**: Azure Container Apps, AWS ECS ਜਾਂ Google Cloud Run ਵਰਗੀਆਂ ਸੇਵਾਵਾਂ ਵਰਤੋਂ।  
- **ਕੁਬਰਨੇਟਿਜ਼**: MCP ਸਰਵਰਾਂ ਨੂੰ ਕੁਬਰਨੇਟਿਜ਼ ਕੁਲੱਸਟਰਾਂ ਵਿੱਚ ਉੱਚ ਉਪਲਬਧਤਾ ਲਈ ਪ੍ਰਬੰਧਿਤ ਕਰੋ।

### ਉਦਾਹਰਨ: Azure Container Apps

Azure Container Apps MCP ਸਰਵਰਾਂ ਦੀ ਤੈਅ ਦਾ ਸਮਰਥਨ ਕਰਦੇ ਹਨ। ਇਹ ਅਜੇ ਵੀ ਵਿਕਾਸ ਵਿੱਚ ਹੈ ਅਤੇ ਅਜੇ SSE ਸਰਵਰਾਂ ਦਾ ਸਮਰਥਨ ਕਰਦਾ ਹੈ।

ਇਸਨੂੰ ਕਿਵੇਂ ਕਰਨਾ ਹੈ:

1. ਇਕ ਰਿਪੋ ਕਲੋਨ ਕਰੋ:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. ਇਸਨੂੰ ਟੈਸਟ ਕਰਨ ਲਈ ਸਥਾਨਕ ਚਲਾਓ:

  ```sh
  uv venv
  uv sync

  # ਲਿਨਕਸ/ਮੈਕਓਐਸ
  export API_KEYS=<AN_API_KEY>
  # ਵਿੰਡੋਜ਼
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. ਇਸਨੂੰ ਸਥਾਨਕ ਪਰੀਖਣ ਲਈ, *.vscode* ਡਾਇਰੈਕਟਰੀ ਵਿੱਚ *mcp.json* ਫਾਇਲ ਬਣਾਓ ਅਤੇ ਹੇਠਾਂ ਦਿੱਤੀਆਂ ਸਮੱਗਰੀ ਸ਼ਾਮਿਲ ਕਰੋ:

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

  ਜਦੋਂ SSE ਸਰਵਰ ਚਾਲੂ ਹੁੰਦਾ ਹੈ, ਤੁਸੀਂ JSON ਫਾਇਲ ਵਿੱਚ ਪਲੇਅ ਆਈਕਨ ’ਤੇ ਕਲਿੱਕ ਕਰ ਸਕਦੇ ਹੋ, ਹੁਣ ਤੁਹਾਨੂੰ ਸਰਵਰ ਦੇ ਸਾਧਨਾਂ ਨੂੰ GitHub Copilot ਦੁਆਰਾ ਪਛਾਣਿਆ ਜਾਵੇਗਾ, Tool ਆਈਕਨ ਵੇਖੋ।

1. ਤੈਅ ਕਰਨ ਲਈ, ਹੇਠਾਂ ਦਿੱਤਾ ਆਦੇਸ਼ ਚਲਾਓ:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

ਹੋ ਗਿਆ, ਇਸ ਤਰ੍ਹਾਂ ਇਸਨੂੰ ਸਥਾਨਕ ਤੌਰ 'ਤੇ ਅਤੇ Azure 'ਤੇ ਤੈਅ ਕਰੋ।

## ਵਾਧੂ ਸਰੋਤ

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps ਲੇਖ](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP ਰਿਪੋ](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## ਅਗਲਾ ਕੀ ਹੈ

- ਅਗਲਾ: [ਅਡਵਾਂਸਡ ਸਰਵਰ ਟੌਪਿਕਸ](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋ:**
ਇਹ ਦਸਤਾਵੇਜ਼ AI ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਅਨੁਵਾਦ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸ਼ੁੱਧਤਾ ਲਈ ਯਤਨ ਕਰਦੇ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਜਾਣੂ ਹੋਵੋ ਕਿ ਸਵੈਚਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਤੀਕਤਾ ਹੋ ਸਕਦੀ ਹੈ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੇ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਹੀ ਅਧਿਕਾਰਿਕ ਸਰੋਤ ਵਜੋਂ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਨਿੱਤਮੁੱਖ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਸਿਫਾਰਸ਼ੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੀ ਵਰਤੋਂ ਤੋਂ ਉਤਪੰਨ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀ ਜਾਂ ਠਹਿਰਾਏ ਜਾਣ ਵਾਲੇ ਨਤੀਜੇ ਲਈ ਜ਼ਿੰਮੇਵਾਰ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->